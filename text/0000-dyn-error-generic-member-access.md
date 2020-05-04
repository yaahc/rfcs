- Feature Name: Add fns for generic member access to dyn Error and the Error trait
- Start Date: 2020-04-01
- RFC PR: [rust-lang/rfcs#0000](https://github.com/rust-lang/rfcs/pull/2895)
- Rust Issue: [rust-lang/rust#0000](https://github.com/rust-lang/rust/issues/0000)

# Summary
[summary]: #summary

This RFC proposes additions to the `Error` trait to support accessing generic
forms of context from `dyn Error` trait objects. This generalizes the pattern
used in `backtrace` and `source` and allows ecosystem iteration on error
reporting infrastructure outside of the standard library. The two proposed
additions are a new trait method `Error::get_context`, which offers
`TypeId`-based member lookup, and a new inherent fn `<dyn Error>::context`,
which makes use of an implementor's `get_context` to return a typed reference
directly. These additions would primarily be useful in "error reporting"
contexts, where we typically no longer have type information and may be
composing errors from many sources.

The names here are just placeholders. The specifics of the `Request` type are a
suggested starting point. And the `Request` style api could be replaced with a
much simpler API based on  `TypeId` + `dyn Any` at the cost of being
incompatible with dynamically sized types. The basic proposal is this:

Add this method to the `Error` trait

```rust
pub trait Error {
    // ...

    /// Provides an object of type `T` in response to this request.
    fn get_context<'r, 'a>(&'a self, request: Request<'r, 'a>) -> ProvideResult<'r, 'a> {
        Ok(request)
    }
}
```

Where an example implementation of this method would look like:

```rust
fn get_context<'r, 'a>(&'a self, request: Request<'r, 'a>) -> ProvideResult<'r, 'a> {
    request
        .provide::<Backtrace>(&self.backtrace)?
        .provide::<SpanTrace>(&self.span_trace)?
        .provide::<dyn Error>(&self.source)?
        .provide::<Vec<&'static Location<'static>>>(&self.locations)?
        .provide::<[&'static Location<'static>>(&self.locations)
}
```

And usage would then look like this:

```rust
let e: &dyn Error = &concrete_error;

if let Some(bt) = e.context::<Backtrace>() {
    println!("{}", bt);
}
```

# Motivation
[motivation]: #motivation

In Rust, errors typically gather two forms of context when they are created:
context for the *current error message* and context for the *final* *error
report*. The `Error` trait exists to provide a interface to context intended
for error reports. This context includes the error message, the source error,
and, more recently, backtraces.

However, the current approach of promoting each form of context to a method on
the `Error` trait doesn't leave room for forms of context that are not commonly
used, or forms of context that are defined outside of the standard library.
Adding a generic equivalent to these member access functions would leave room
for many more forms of context in error reports.

## Example use cases this enables

* using alternatives to `std::backtrace::Backtrace` such as
  `backtrace::Backtrace` or [`SpanTrace`]
* zig-like Error Return Traces by extracting `Location` types from errors
  gathered via `#[track_caller]` or similar.
* error source trees instead of chains by accessing the source of an error as a
  slice of errors rather than as a single error, such as a set of errors caused
  when parsing a file
* Help text such as suggestions or warnings attached to an error report

To support these use cases without ecosystem fragmentation, we would extend the
Error trait with a dynamic context API that allows implementors and clients to
enrich errors in an opt-in fashion.

## Moving `Error` into `libcore`

Adding a generic member access function to the Error trait and removing the
`backtrace` fn would make it possible to move the `Error` trait to libcore
without losing support for backtraces on std. The only difference would be that
in places where you can currently write `error.backtrace()` on nightly you
would instead need to write `error.context::<Backtrace>()`.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

Error handling in Rust consists of three steps: creation/propagation, handling,
and reporting. The `std::error::Error` trait exists to bridge the gap between
creation and reporting. It does so by acting as a interface that all error
types can implement that defines how to access context intended for error
reports, such as the error message, source, or location it was created. This
allows error reporting types to handle errors in a consistent manner when
constructing reports for end users while still retaining control over the
format of the full report.

The error trait accomplishes this by providing a set of methods for accessing
members of `dyn Error` trait objects. It requires types implement the display
trait, which acts as the interface to the main member, the error message
itself.  It provides the `source` function for accessing `dyn Error` members,
which typically represent the current error's cause. It provides the
`backtrace` function, for accessing a `Backtrace` of the state of the stack
when an error was created. For all other forms of context relevant to an error
report, the error trait provides the `context` and `get_context` functions.

As an example of how to use this interface to construct an error report, let’s
explore how one could implement an error reporting type. In this example, our
error reporting type will retrieve the source code location where each error in
the chain was created (if it exists) and render it as part of the chain of
errors. Our end goal is to get an error report that looks something like this:

```
Error:
    0: Failed to read instrs from ./path/to/instrs.json
        at instrs.rs:42
    1: No such file or directory (os error 2)
```

The first step is to define or use a type to represent a source location. In
this example, we will define our own:

```rust
struct Location {
    file: &'static str,
    line: usize,
}
```

Next, we need to gather the location when creating our error types.

```rust
struct ExampleError {
    source: std::io::Error,
    location: Location,
    path: PathBuf,
}

impl fmt::Display for ExampleError {
    fn fmt(&self, fmt: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(fmt, "Failed to read instrs from {}", path.display())
    }
}

fn read_instrs(path: &Path) -> Result<String, ExampleError> {
    std::fs::read_to_string(path).map_err(|source| {
        ExampleError {
            source,
            path: path.to_owned(),
            location: Location {
                file: file!(),
                line: line!(),
            },
        }
    })
}
```

Then, we need to implement the `Error` trait to expose these members to the error reporter.

```rust
impl std::error::Error for ExampleError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        Some(&self.source)
    }

    fn get_context<'r, 'a>(&'a self, request: Request<'r, 'a>) -> ProvideResult<'r, 'a> {
        request.provide::<Location>(&self.location)
    }
}
```

And, finally, we create an error reporter that prints the error and its source
recursively, along with any location data that was gathered

```rust
struct ErrorReporter(Box<dyn Error + Send + Sync + 'static>);

impl fmt::Debug for ErrorReporter {
    fn fmt(&self, fmt: &mut fmt::Formatter<'_>) -> fmt::Result {
        let mut current_error = Some(self.0.as_ref());
        let mut ind = 0;

        while let Some(error) = current_error {
            writeln!(fmt, "    {}: {}", ind, error)?;

            if let Some(location) = error.context::<Location>() {
                writeln!(fmt, "        at {}:{}", location.file, location.line)?;
            }

            ind += 1;
            current_error = error.source();
        }

        Ok(())
    }
}
```

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

The following changes need to be made to implement this proposal:

## Add a type like [`ObjectProvider::Request`] to std

This type fills the same role as `&dyn Any` except that it supports other trait
objects as the requested type.

Here is the implementation for the proof of concept:

```rust
/// A dynamic request for an object based on its type.
///
/// `'r` is the lifetime of request, and `'out` is the lifetime of the requested
/// reference.
pub struct Request<'r, 'out> {
    buf: NonNull<TypeId>,
    _marker: PhantomData<&'r mut &'out Cell<()>>,
}

impl<'r, 'out> Request<'r, 'out> {
    /// Provides an object of type `T` in response to this request.
    ///
    /// Returns `Err(FulfilledRequest)` if the value was successfully provided,
    /// and `Ok(self)` if `T` was not the type being requested.
    ///
    /// This method can be chained within `provide` implementations using the
    /// `?` operator to concisely provide multiple objects.
    pub fn provide<T: ?Sized + 'static>(self, value: &'out T) -> ProvideResult<'r, 'out> {
        self.provide_with(|| value)
    }

    /// Lazily provides an object of type `T` in response to this request.
    ///
    /// Returns `Err(FulfilledRequest)` if the value was successfully provided,
    /// and `Ok(self)` if `T` was not the type being requested.
    ///
    /// The passed closure is only called if the value will be successfully
    /// provided.
    ///
    /// This method can be chained within `provide` implementations using the
    /// `?` operator to concisely provide multiple objects.
    pub fn provide_with<T: ?Sized + 'static, F>(mut self, cb: F) -> ProvideResult<'r, 'out>
    where
        F: FnOnce() -> &'out T,
    {
        match self.downcast_buf::<T>() {
            Some(this) => {
                debug_assert!(
                    this.value.is_none(),
                    "Multiple requests to a `RequestBuf` were acquired?"
                );
                this.value = Some(cb());
                Err(FulfilledRequest(PhantomData))
            }
            None => Ok(self),
        }
    }

    /// Get the `TypeId` of the requested type.
    pub fn type_id(&self) -> TypeId {
        unsafe { *self.buf.as_ref() }
    }

    /// Returns `true` if the requested type is the same as `T`
    pub fn is<T: ?Sized + 'static>(&self) -> bool {
        self.type_id() == TypeId::of::<T>()
    }

    /// Try to downcast this `Request` into a reference to the typed
    /// `RequestBuf` object.
    ///
    /// This method will return `None` if `self` was not derived from a
    /// `RequestBuf<'_, T>`.
    fn downcast_buf<T: ?Sized + 'static>(&mut self) -> Option<&mut RequestBuf<'out, T>> {
        if self.is::<T>() {
            unsafe { Some(&mut *(self.buf.as_ptr() as *mut RequestBuf<'out, T>)) }
        } else {
            None
        }
    }

    /// Calls the provided closure with a request for the the type `T`, returning
    /// `Some(&T)` if the request was fulfilled, and `None` otherwise.
    ///
    /// The `ObjectProviderExt` trait provides helper methods specifically for
    /// types implementing `ObjectProvider`.
    pub fn with<T: ?Sized + 'static, F>(f: F) -> Option<&'out T>
    where
        for<'a> F: FnOnce(Request<'a, 'out>) -> ProvideResult<'a, 'out>,
    {
        let mut buf = RequestBuf {
            type_id: TypeId::of::<T>(),
            value: None,
        };
        let _ = f(Request {
            buf: unsafe {
                NonNull::new_unchecked(&mut buf as *mut RequestBuf<'out, T> as *mut TypeId)
            },
            _marker: PhantomData,
        });
        buf.value
    }
}

// Needs to have a known layout so we can do unsafe pointer shenanigans.
#[repr(C)]
struct RequestBuf<'a, T: ?Sized> {
    type_id: TypeId,
    value: Option<&'a T>,
}

/// Marker type indicating a request has been fulfilled.
pub struct FulfilledRequest(PhantomData<&'static Cell<()>>);

/// Provider method return type.
///
/// Either `Ok(Request)` for an unfulfilled request, or `Err(FulfilledRequest)`
/// if the request was fulfilled.
pub type ProvideResult<'r, 'a> = Result<Request<'r, 'a>, FulfilledRequest>;
```

## Define a generic accessor on the `Error` trait

```rust
pub trait Error {
    // ...

    /// Provides an object of type `T` in response to this request.
    fn get_context<'r, 'a>(&'a self, request: Request<'r, 'a>) -> ProvideResult<'r, 'a> {
        Ok(request)
    }
}
```

## Use this `Request` type to handle passing generic types out of the trait object

```rust
impl dyn Error {
    pub fn context<T: ?Sized + 'static>(&self) -> Option<&T> {
        Request::with::<T, _>(|req| self.get_context(req))
    }
}
```

# Drawbacks
[drawbacks]: #drawbacks

* The `Request` api is being added purely to help with this fn, there may be
  some design iteration here that could be done to make this more generally
  applicable, it seems very similar to `dyn Any`.
* The `context` function name is currently widely used throughout the rust
  error handling ecosystem in libraries like `anyhow` and `snafu` as an
  ergonomic version of `map_err`. If we settle on `context` as the final name
  it will possibly break existing libraries.


# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

The two alternatives I can think of are:

## Do Nothing

We could not do this, and continue to add accessor functions to the `Error`
trait whenever a new type reaches critical levels of popularity in error
reporting.

If we choose to do nothing we will continue to see hacks around the current
limitations on the error trait such as the `Fail` trait, which added the
missing function access methods that didn't previously exist on the `Error`
trait and type erasure / unnecessary boxing of errors to enable downcasting to
extract members.
[[1]](https://docs.rs/tracing-error/0.1.2/src/tracing_error/error.rs.html#269-274).


## Use an alternative proposal that relies on the `Any` trait for downcasting

This approach is much simpler, but critically doesn't support providing
dynamically sized types, and it is more error prone because it cannot provide
compile time errors when the type you provide does not match the type_id you
were given.

```rust
pub trait Error {
    /// Provide an untyped reference to a member whose type matches the provided `TypeId`.
    ///
    /// Returns `None` by default, implementors are encouraged to override.
    fn provide(&self, ty: TypeId) -> Option<&dyn Any> {
        None
    }
}

impl dyn Error {
    /// Retrieve a reference to `T`-typed context from the error if it is available.
    pub fn request<T: Any>(&self) -> Option<&T> {
        self.get_context(TypeId::of::<T>())?.downcast_ref::<T>()
    }
}
```

# Prior art
[prior-art]: #prior-art

I do not know of any other languages whose error handling has similar
facilities for accessing members when reporting errors. For the most part prior
art exists within rust itself in the form of previous additions to the `Error`
trait.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

* What should the names of these functions be?
    * `context`/`context_ref`/`get_context`/`provide_context`
    * `member`/`member_ref`
    * `provide`/`request`
* Should there be a by value version for accessing temporaries?
    * We bring this up specifically for the case where you want to use this
      function to get an `Option<&[&dyn Error]>` out of an error, in this case
      its unlikely that the error behind the trait object is actually storing
      the errors as `dyn Errors`, and theres no easy way to allocate storage to
      store the trait objects.
* How should context handle failed downcasts?
    * suggestion: panic, as providing a type that doesn't match the typeid
      requested is a program error

# Future possibilities
[future-possibilities]: #future-possibilities

Libraries like `thiserror` could add support for making members exportable as
context for reporters.

This opens the door to supporting `Error Return Traces`, similar to zigs, where
if each return location is stored in a `Vec<&'static Location<'static>>` a full
return trace could be built up with:

```rust
let mut locations = e
    .chain()
    .filter_map(|e| e.context::<[&'static Location<'static>]>())
    .flat_map(|locs| locs.iter());
```

[`SpanTrace`]: https://docs.rs/tracing-error/0.1.2/tracing_error/struct.SpanTrace.html
[`ObjectProvider::Request`]: https://github.com/yaahc/nostd-error-poc/blob/master/fakecore/src/any.rs