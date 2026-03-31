- Start Date: 2026-03-27
- RFC PR: [rust-lang/rfcs#0000](https://github.com/rust-lang/rfcs/pull/0000)
- Rust Issue: [rust-lang/rust#0000](https://github.com/rust-lang/rust/issues/0000)

## Summary
[summary]: #summary

Add policy documenting the de-facto treatment and expectations around LLM tool
usage by and for contributors to the Rust project.

- We will make sure that contribution guidelines, including but not limited to
  CONTRITBUTING.md, Forge's how to start contributing page, rustc-dev-guide
  have obvious guidance and cite this policy explicitly.
- We will backlink to contribution etiquette and this policy in the PR template.

## Motivation
[motivation]: #motivation

This RFC aims to document the de-facto policy that is already practically in
effect and acted upon by the project's various reviewers and moderators to help
provide clarity to our contributors and clear up any differing interpretations
or gray areas in said de-facto policy.

This policy seeks to help address the burden of LLM generated contributions
upon the Rust Project Maintainers.

Other goals adopted by the author that influence the shaping of this policy:

* Building a community of deep experts in our collective projects.
* Building an inclusive community where all feel welcome and respected.

## Rust LLM Tool Use Policy
[ai-tool-use-policy]: #ai-tool-use-policy

### Disclaimer

There is not yet a full consensus within the Rust org about when/how/where it
is acceptable to use LLM-based tools. Many members of the Rust community find
value in LLMs; many others feel that its negative impact on society and the
climate are severe enough that no use is acceptable. Still others are working
out their opinion. For more detailed insight into perspectives on LLMs within
the Rust Project, please refer to [Rust Project Perspectives on
AI][rp-perspectives-ai].

[rp-perspectives-ai]: https://nikomatsakis.github.io/rust-project-perspectives-on-ai/all-comments.html

This policy does not seek to set the final word on LLM use within the Rust
Project. We expect this policy to evolve over time as the Rust Project members
work through the difficult discussions surrounding this topic.

### Policy

The Rust Project's policy is that contributors can use whatever tools they
would like to craft their contributions, but there must be a **human in the
loop**. **Contributors must read and review all LLM-generated code before they
ask other project members to review it.** The contributor is always the author
and is fully accountable for their contributions. Contributors should be
sufficiently confident that the contribution is high enough quality that asking
for a review is a good use of scarce maintainer time, and they should be **able
to answer questions about their work, without using an LLM** during review.

Contributors must not present LLM generated text as if it communication from a
human, such as in PR descriptions or issue comments. LLM generated text may be
included in these venues when clearly identified as such above the text in
question. This includes asking an LLM to draft a comment for you and presenting
it as if you wrote it. Such comments can contain signs of being written by LLMs
and may be interpreted as such. For usages such as LLM generated translations,
contributors are encouraged to include the original text in the source language
alongside the translation, which may include additional context lost in
translation that other project members who understand the source language are
able to catch.

We expect that new contributors will be less confident in their contributions,
and our guidance to them is to **start with small contributions** that they can
fully understand to build confidence. We aspire to be a welcoming community
that helps new contributors grow their expertise, but learning involves taking
small steps, getting feedback, and iterating. Passing maintainer feedback to an
LLM pasting the generated text back doesn't help anyone grow, and does not
sustain our community.

Contributors are expected to **be transparent and explain how and when they've
used LLM generated content during their contribution**. This can for example
include sharing prompts, context, or conversations with chat-style LLMs.

This policy includes, but is not limited to, the following kinds of
contributions:

- Code, usually in the form of a pull request
- RFCs or design proposals
- Issues or security vulnerabilities
- Comments and feedback on pull requests
- Communication on Zulip

### Details

To ensure sufficient self review and understanding of the work, it is strongly
recommended that contributors write PR descriptions themselves. The description
should explain the motivation, implementation approach, expected impact, and
any open questions or uncertainties to the same extent as a contribution made
without tool assistance.

An important implication of this policy is that it bans agents that take action
in our digital spaces without human approval, such as the GitHub [`@claude`
agent](https://github.com/claude/). Similarly, automated review tools that
publish comments without human review are not allowed. However, an opt-in
review tool that **keeps a human in the loop** is acceptable under this policy.
As another example, using an LLM to generate documentation, which a contributor
manually reviews for correctness, edits, and then posts as a PR, is an approved
use of tools under this policy.

LLM tools must not be used to fix GitHub issues labelled [`E-easy`][E-easy].
These issues are generally not urgent, and are intended to be learning
opportunities for new contributors to get familiar with the codebase. Whether
you are a newcomer or not, fully automating the process of fixing this issue
squanders the learning opportunity and doesn't add much value to the project.
**Using LLM tools to fix issues labelled as "E-easy" is forbidden**.

[E-easy]: https://github.com/rust-lang/rust/issues?q=is%3Aissue%20state%3Aopen%20label%3AE-easy

TODO additional prohibited uses of LLMs to fit into this section

* writing code if you are a first-time contributors or are not capable of
  reviewing your own PR -- don't take away your own chance to learn :)
* communicating "as you" on Zulip, Github issues, etc -- do not take messages
  that people have sent you and just forward them to a bot!

### Extractive Contributions

The reason for our "human-in-the-loop" contribution policy is that processing
patches, PRs, RFCs, and comments to rust-lang/rust is not free -- it takes a lot of
maintainer time and energy to review those contributions! Sending the
unreviewed output of an LLM to open source project maintainers *extracts* work
from them in the form of design and code review, so we call this kind of
contribution an "extractive contribution".

Our **golden rule** is that a contribution should be worth more to the project
than the time it takes to review it. These ideas are captured by this quote
from the book [Working in Public][public] by Nadia Eghbal:

[public]: https://press.stripe.com/working-in-public

> \"When attention is being appropriated, producers need to weigh the costs and
> benefits of the transaction. To assess whether the appropriation of attention
> is net-positive, it's useful to distinguish between *extractive* and
> *non-extractive* contributions. Extractive contributions are those where the
> marginal cost of reviewing and merging that contribution is greater than the
> marginal benefit to the project's producers. In the case of a code
> contribution, it might be a pull request that's too complex or unwieldy to
> review, given the potential upside.\" \-- Nadia Eghbal

Prior to the advent of LLMs, open source project maintainers would often review
any and all changes sent to the project simply because posting a change for
review was a sign of interest from a potential long-term contributor. While new
tools enable more development, it shifts effort from the implementor to the
reviewer, and our policy exists to ensure that we value and do not squander
maintainer time.

Reviewing changes from new contributors is part of growing the next generation
of contributors and sustaining the project. We want the Rust project to be
welcoming and open to aspiring compiler engineers who are willing to invest
time and effort to learn and grow, because growing our contributor base and
recruiting new maintainers helps sustain the project over the long term.

TODO customize below to be more specific to the rust project

Being open to contributions and [liberally granting commit access][commit-access]
is a big part of how Rust has grown and successfully been adopted all across
the industry.  We therefore automatically post a greeting comment to pull
requests from new contributors and encourage maintainers to spend their time to
help new contributors learn.

[commit-access]: https://llvm.org/docs/DeveloperPolicy.html#obtaining-commit-access

### Handling Violations

- If rust-lang/rust contributors suspects an extractive contribution, they can report
  the instance to the moderation team.
- If the moderation team agrees with the assessment:
    - The first instance of such extractive contribution will receive a moderation
      warning. Repeated offense(s) will result in a moderation ban.
- If rust-lang/rust reviewers or the moderation team have reasonable grounds to
  suspect that the account submitting the extractive contribution is most likely
  fully automated (such as spamming contributions fully automated via
  OpenClaw), the contributor may receive a moderation ban immediately even if
  it is an account's first instance.

Contributors who received a moderation warning or ban as a consequence of this policy can contact the moderation team to discuss (and possibly rescind) the moderation action as usual.

TODO how much of this llvm text would we want to adopt vs jieyouxu's proposal
above?

> If a maintainer judges that a contribution doesn't comply with this policy,
> they should paste the following response to request changes:
>
>     This PR doesn't appear to comply with our policy on tool-generated content,
>     and requires additional justification for why it is valuable enough to the
>     project for us to review it. Please see our developer policy on
>     LLM-generated contributions: TODO link
>
> The best ways to make a change less extractive and more valuable are to reduce
> its size or complexity or to increase its usefulness to the community. These
> factors are impossible to weigh objectively, and our project policy leaves this
> determination up to the maintainers of the project, i.e. those who are doing
> the work of sustaining the project.
>
> If or when it becomes clear that a GitHub issue or PR is off-track and not
> moving in the right direction, maintainers should apply the `extractive` label
> to help other reviewers prioritize their review time.
>
> If a contributor fails to make their change meaningfully less extractive,
> maintainers should escalate to the relevant moderation or admin team for the
> space (GitHub, Discourse, Discord, etc) to lock the conversation.

### Copyright

TODO integrate and update language here based on https://rust-lang.zulipchat.com/#narrow/channel/244344-t-compiler.2Fcontrib-private/topic/forge.20guidelines.20on.20use.20of.20AI/near/582187540

> On this topic, the Rust project directors consulted the Rust Foundation’s
> legal counsel and they did not have significant concerns about Rust accepting
> LLM-generated code from a legal perspective. Some courts have found that
> AI-generated code is not subject to copyright and it’s expected that others
> will follow suit. Any human-contributed original expression would be owned by
> the human author, but if that author is the contributor (or the modifications
> are licensed under an open source license), the situation is no different
> from any human-origin contribution. However, this does not present a legal
> obstacle to us redistributing the code, because, as this code is not
> copyrighted, it can be freely redistributed. Further, while it is possible
> for LLMs to generate code (especially small portions) that is identical to
> code in the training data, outstanding litigation has not revealed that this
> is a significant issue, and often such portions are too small or contain such
> limited originality that they may not qualify for copyright protection.

Artificial intelligence systems raise many questions around copyright that have
yet to be answered. Our policy on LLM tools is similar to our copyright policy:
Contributors are responsible for ensuring that they have the right to
contribute code under the terms of our license, typically meaning that either
they, their employer, or their collaborators hold the copyright. Using LLM tools
to regenerate copyrighted material does not remove the copyright, and
contributors are responsible for ensuring that such material does not appear in
their contributions. Contributions found to violate this policy will be removed
just like any other offending contribution.

### Exceptions

### Examples

## Drawbacks
[drawbacks]: #drawbacks

Why should we *not* do this?

## Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

<!--
- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not choosing them?
- What is the impact of not doing this?
- If this is a language proposal, could this be done in a library or macro instead? Does the proposed change make Rust code easier or harder to read, understand, and maintain?
-->

- https://github.com/rust-lang/rfcs/pull/3936
- Review of open source LLM tool use policies, ordered strictness, top is most strict, bottom least. Copied from [leadership-council#273 comment](https://github.com/rust-lang/leadership-council/issues/273#issuecomment-4051188890)
    - full ban
        - [postmarketOS](https://docs.postmarketos.org/policies-and-processes/development/ai-policy.html) also explicitly bans encouraging others to use AI for solving problems related to postmarketOS
            - multi point ethics based rational with citations included
        - [zig](https://ziglang.org/code-of-conduct/)
            - philosophical, cites https://en.wikipedia.org/wiki/Profession_(novella)
            - afaict rooted in concerns around the construction and origins of original thought
        - [servo](https://book.servo.org/contributing/getting-started.html#ai-contributions)
            - more pragmatic, directly lists concerns around ai, fairly concise
        - [qemu](https://www.qemu.org/docs/master/devel/code-provenance.html#use-of-ai-content-generators)
            - pragmatic, focuses on copyright and licensing concerns
            - explicitly allows ai for exploring api, debugging, and other non generative assistance, other policies do not explicitly disallow this or mention it in any way
    - allowed with supervision, human is ultimately responsible
        - [scipy](https://github.com/scipy/scipy/pull/24583/changes)
            - strict attribution policy including name of model
        - [llvm](https://llvm.org/docs/AIToolPolicy.html)
        - [blender](https://devtalk.blender.org/t/ai-contributions-policy/44202)
        - [linux kernel](https://kernel.org/doc/html/next/process/coding-assistants.html)
            - quite concise but otherwise seems the same as many in this category
        - [mesa](https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/docs/submittingpatches.rst)
            - framed as a contribution policy not an AI policy, AI is listed as a tool that can be used but emphasizes same requirements that author must understand the code they contribute, seems to leave room for partial understanding from new contributors.
                  > Understand the code you write at least well enough to be able
                  to explain why your changes are beneficial to the project.
        - [forgejo](https://codeberg.org/forgejo/governance/src/branch/main/AIAgreement.md)
            - bans ai for review, does not explicitly require contributors to understand code generated by ai. One could interpret the "accountability for contribution lies with contributor even if AI is used" line as implying this requirement, though their version seems poorly worded imo.
        - [firefox](https://firefox-source-docs.mozilla.org/contributing/ai-coding.html)
        - [ghostty](https://github.com/ghostty-org/ghostty/blob/main/AI_POLICY.md)
            - pro-AI but views "bad users" as the source of issues with it and the only reason for what ghostty considers a "strict AI policy"
        - [fedora](https://communityblog.fedoraproject.org/council-policy-proposal-policy-on-ai-assisted-contributions/)
            - clearly inspired and is cited by many of the above, but is definitely framed more pro-ai than the derived policies tend to be
    - [curl](https://curl.se/dev/contribute.html#on-ai-use-in-curl)
        - does not explicitly require humans understand contributions, otherwise policy is similar to above policies
    - [linux foundation](https://www.linuxfoundation.org/legal/generative-ai)
        - encourages usage, focuses on legal liability, mentions that tooling exists to help automate managing legal liability, does not mention specific tools
    - In progress
        - NixOS
            - https://github.com/NixOS/nixpkgs/issues/410741
        - Rust
            - https://github.com/rust-lang/compiler-team/issues/893
            - https://github.com/rust-lang/leadership-council/issues/273

## Prior art
[prior-art]: #prior-art

- https://llvm.org/docs/AIToolPolicy.html
- rust-lang.zulipchat.com/#narrow/channel/392734-council/topic/Project.20perspectives.20on.20AI/near/580301943
- https://github.com/rust-lang/leadership-council/issues/273
- https://github.com/rust-lang/compiler-team/issues/893
- https://github.com/rust-lang/moderation-team/blob/main/policies/spam.md/#fully-or-partially-automated-contribs
- https://forge.rust-lang.org/how-to-start-contributing.html#etiquette

## Unresolved questions
[unresolved-questions]: #unresolved-questions

Grey areas raised by niko that haven't been integrated into the draft policy yet:

```md
## Grey areas for experienced contributors and team members

For experienced contributors and team members, the use of AI is a grey zone. We
encourage people in these areas to talk to one another and be open to feedback
as we try to figure out how to navigate this time.

* *Use of AI by experience contributors to edit code*: but keep in mind that
  most reviewers do not want to review AI-generated code until you have given
  it a thorough review, not just a "quick glance".
* *Use of AI to generate "first drafts" of text, PR descriptions etc*: but keep
  in mind that AI-generated text tends to lack character and many people will
  have negative reactions when they read it unless you've given it a good read.

- What parts of the design do you expect to resolve through the RFC process
  before this gets merged?
- What related issues do you consider out of scope for this RFC that could be
  addressed in the future independently of the solution that comes out of this
  RFC?
```

## Future possibilities
[future-possibilities]: #future-possibilities

<!--
Think about what the natural extension and evolution of your proposal would
be and how it would affect the project as a whole in a holistic
way. Try to use this section as a tool to more fully consider all possible
interactions with the project in your proposal.
Also consider how this all fits into the roadmap for the project.

This is also a good place to "dump ideas", if they are out of scope for the
RFC you are writing but otherwise related.

If you have tried and cannot think of any future possibilities,
you may simply state that you cannot think of anything.

Note that having something written down in the future-possibilities section
is not a reason to accept the current or a future RFC; such notes should be
in the section on motivation or rationale in this or subsequent RFCs.
The section merely provides additional information.
-->
