<p align="center">
<img src="https://github.com/luna/luna-studio/raw/master/resources/logo.ico"
    style="margin: 0 auto;">
</p>
<h1 align="center">Luna RFCs</h1>
<h3 align="center">
Visual and textual functional programming language with a focus on productivity,
collaboration and development ergonomics.
</h3>

## Introduction
Like any Open Source language, Luna welcomes interaction from the community, as
we believe that only through the community's suggestions and feedback can we
make Luna the best it can be.

To this end, this repository defines a process to be followed for those people
that want to make 'substantial' changes to the Luna language, `luna` the
compiler or even this RFC process itself. What we define as a 'substantial'
change is in flux based on community norms and in relation to the portion of
the ecosystem being changed.

Substantial changes may thus include the following:

- Semantic or syntactic changes to Luna that aren't related to fixing a
  pre-existing bug.
- Adding or removing language features.
- Changes to the interface between the language and its standard library.
- Additions to the standard library.

The following are examples of changes that do not need to follow the RFC
process:

- Refactoring of the compiler or libraries that does not change any semantics.
- Additions or alterations that improve numerical metrics (e.g. performance
  improvements, error detection, ergonomics improvements).
- Additions invisible to the users of the language (e.g. new APIs internal to
  the compiler that do not fall under the categories for 'substantial' changes
  listed above).

Before contributing an RFC, please ensure that you have read and agree with the
[licensing terms](#license) under which this repository operates.

## Table of Contents

- [Introduction](#introduction)
- [Table of Contents](#table-of-contents)
- [Creating an RFC](#creating-an-rfc)
    + [Before Creating an RFC](#before-creating-an-rfc)
    + [The RFC Process](#the-rfc-process)
    + [Reviewing RFCs](#reviewing-rfcs)
- [The RFC Life-Cycle](#the-rfc-life-cycle)
    + [Active RFCs](#active-rfcs)
    + [RFC Implementation](#rfc-implementation)
    + [RFC Postponement](#rfc-postponement)
    + [Label Descriptions](#label-descriptions)
- [Credits](#credits)
- [License](#license)
    + [Contributions](#contributions)
    + [The Contributor Agreement](#the-contributor-agreement)

## Creating an RFC
While informal, we do aim to provide a basic outline for the process of creating
an RFC. The intention of this process is that RFCs arrive in a state with enough
detail to spark discussion, and then evolve through discussion with all
interested parties.

### Before Creating an RFC
Keep in mind that a hastily-created RFC can hurt discussion and its chances of
being accepted, even if the idea proposed is excellent! A half-baked proposal,
a previously-rejected idea, or a feature that doesn't align with our vision for
Luna could be rejected swiftly, making the unprepared contributor feel
demotivated. Doing a little bit of preparatory legwork can make the RFC process
that much more smooth.

While there is no specific process for submitting an RFC, we do recommend that
you talk to other members of the community to find out whether your proposal
would have support, and fits with the broader vision for Luna as a whole. We do
recommend discussing the proposed RFC in the
[Luna Discord](https://discord.gg/RNrbSux) beforehand to see what the potential
community reception would be.

If the idea receives encouraging feedback from the core team, as well as the
community, it is an indication that the RFC is worth pursuing.

### The RFC Process
To summarise the end goal of the RFC process, the aim is to merge the proposal
(as a markdown file) into the RFC Repository. At this point, the RFC becomes
'Active', and may then be implemented.

If this is your first contribution to the Luna RFC process, please ensure that
you have followed the instructions found in 
[the Contributor License Agreement](#the-contributor-license-agreement).

The process for creating an RFC can be outlined as follows:

1. Fork the [RFC Repository](https://github.com/luna/luna-rfcs).
2. Copy [`0000-rfc-template.md`](0000-rfc-template.md) to
   `text/0000-my-proposal.md` (where `my-proposal` is descriptive for the
   feature).
3. Fill in the RFC Template for your feature. Take care with the details as RFCs
   that don't put thought into the design or its impacts, or are disingenuous
   about the drawbacks or alternatives are likely to meet a poor reception. One
   of the key points for considering RFCs is how they fit with the vision for
   Luna as a whole.
4. Submit a [Pull Request](https://github.com/luna/luna-rfcs/pulls). Please give
   the PR a descriptive title (`RFC: My Feature`). The Pull Request will be
   open for feedback and discussion from the community and core team, and the
   author should be open to revising it in response to this feedback. The RFC
   will be assigned a shepherd from the core team who will be responsible for
   managing the RFC alongside the author.
5. Evolve the RFC. Build consensus around the feature through your revisions.
   Feel free to reach out to the shepherd assigned to your proposal for
   discussion, as well as help identifying the key stakeholders and obstacles.
6. The core team will participate in the discussion on the PR thread, and will
   summarise any offline discussion in the same thread.
7. Once the RFC has reached a stable point, the shepherd will propose that the
   RFC enters the Final Comment Period (FCP), as well as a disposition for the
   RFC: _Accept_, _Close_ or _Postpone_. It should be noted that taking this
   step does not require consensus, and should include a summary of the previous
   discussion and changes.
8. The FCP lasts for 10 calendar days. This gives all stakeholders an ample
   chance to raise any final objections or comments before a decision is made.
   If substantial new arguments emerge during the FCP, the FCP may be cancelled.
9. The RFC has a final decision reached, and is either Accepted (where it is
   merged into the repository after having an RFC number assigned to it), 
   Postponed, where the PR is tagged and closed, or Rejected and closed.

### Reviewing RFCs
While the RFC pull request is under discussion, the shepherd may organise
meetings with the author and stakeholders to discuss the issue(s) in more
detail. A summary from this meeting will be compiled by the shepherd and posted
back to the pull request.

## The RFC Life-Cycle
RFCs proceed through their life-cycle in this repository as follows:

1. The Pull Request is submitted. The repository manager assigns appropriate
   [tags](#label-descriptions) to the RFC so that the relevant stakeholders can
   find it. It is also assigned a shepherd and tagged for discussion.
2. Once discussion has stabilised, the RFC is proposed for the Final Comment
   Period, and tagged as such.
3. At the end of the FCP, the RFC will be tagged with the decision made.
4. If the RFC is accepted, it is then marked as [Active](#active-rfcs). Being
   active means it can be implemented, and submitted as a
   [Pull Request](https://github.com/luna/luna/pulls) to the Luna repository.

### Active RFCs
An RFC being 'Active' should not be viewed as a rubber stamp, and in particular
it does _not_ mean that the feature will be ultimately merged into Luna. It does
mean that, in principle, stakeholders are in agreement on the feature and are
amenable to it being merged.

Being accepted doesn't mean that there is a priority assigned to its
implementation, or a developer dedicated to implementing it. While it is _not_
necessary (and in some cases not recommended) for the RFC author to write the
implementation, it is likely to result in the RFC being seen to completion more
swiftly than it otherwise would be.

Once accepted and active, small modifications to the RFC can be made via
follow-on pull requests. While we usually intend for RFCs to reflect the final
design, interim changes may necessitate small alterations to keep the RFC in
sync with the final implementation.

Please note that the RFC should not be substantially changed at this point. If
substantial changes are required, they should take place as a new RFC, linked to
and from the old.

### RFC Implementation
Some accepted RFCs are vital features that need to be implemented as soon as
possible, while others can wait until a so-inclined developer comes along. Every
accepted RFC is assigned a tracking issue in the
[Luna](https://github.com/luna/luna/) repository, and is then assigned a
priority via the Luna repository triage process.

The author of an RFC is not obligated to implement it. Of course, the RFC author
(like any other developer) is welcome to post an implementation for review after
the RFC has been accepted.

If you are interested in working on the implementation for an "active" RFC, but
cannot determine if someone else is already working on it, feel free to ask
(e.g. by leaving a comment on the associated issue).

### RFC Postponement
Some RFC pull requests are tagged with the 'Postponed' label when they are
closed (as part of the rejection process). This label implies that we neither
want to think about evaluating the proposal or implementing it until some time
in the future. These pull requests may be re-opened when the time is right.

RFCs are usually postponed after some evaluation has taken place, and the label
is usually given because the time is not right to consider or implement such a
feature for now.

### Label Descriptions
As part of the RFC process, the RFC pull request will have some of the following
labels assigned to it. This table provides an overview of what they mean.

| Label                             | Description                                                                       |
|:----------------------------------|:----------------------------------------------------------------------------------|
| B - Needs Summary                 | The RFC pull request is awaiting a meeting summary.                               |
| B - Waiting on Author             | The RFC pull request is waiting for author input.                                 |
| B - Waiting on Community Feedback | The RFC pull request is waiting for community feedback.                           |
| B - Waiting on Shepherd           | The RFC pull request is waiting for input from the shepherd.                      |
| B - Waiting on Team               | The RFC pull request is waiting on input from the Luna core team.                 |
| C - Compiler                      | The RFC deals with the luna compiler.                                             |
| C - Ecosystem                     | The RFC proposes changes to the ecosystem as a whole.                             |
| C - GUI                           | The RFC proposes changes or additions to the Luna Studio GUI.                     |
| C - Language                      | The RFC deals with the Luna language specification.                               |
| C - Libraries                     | The RFC proposes changes to the Luna standard library.                            |
| C - RFC Process                   | The RFC proposes changes to this RFC process.                                     |
| C - Semantics                     | The RFC proposes changes to language semantics.                                   |
| C - Syntax                        | The RFC proposes syntactic changes or additions to Luna.                          |
| C - Tooling                       | The RFC proposes changes to tooling surrounding Luna.                             |
| C - Type System                   | The RFC proposes changes or additions to the type system.                         |
| S - Accepted                      | The RFC has been accepted.                                                        |
| S - Active                        | The RFC is active as described in [Active RFCs](#active-rfcs).                    |
| S - Discussion                    | The RFC is currently under discussion.                                            |
| S - Final Comment Period          | The RFC is in its Final Comment Period.                                           |
| S - Postponed                     | The RFC has been postponed as described in [RFC Postponement](#rfc-postponement). |
| S - Rejected                      | The RFC has been rejected.                                                        |
| T - Breaking Change               | The changes proposed by the RFC are breaking.                                     |
| T - Non-Breaking Change           | The changed proposed by the RFC are non-breaking.                                 |

## Credits
This repository and process takes significant inspiration from the
[Rust RFCs](https://github.com/rust-lang/rfcs) process, and the
[GHC Proposals](https://github.com/ghc-proposals/ghc-proposals) process, so
thank you to both communities for inspiring us!

## License
This repository is licensed under the
[MIT License](http://opensource.org/licenses/MIT), as specified in the
[LICENSE](https://github.com/luna/luna-rfcs/blob/master/LICENSE) file. All
contributions to this repository are accepted under these terms.

Please be aware that, as the commercial backing for Luna, 
**New Byte Order Sp. z o. o.** reserves the right under the MIT license to 
implement any RFCs contained in this repository as part of commercially 
available Luna products. This means that any contributions here _may not_ appear
as part of an open-source portion of the Luna ecosystem as a result of work by
the aforementioned company.

However, this places _no restriction_ on the implementation of such RFCs into 
the open-source portions of the Luna ecosystem by other contributors, as allowed 
under the MIT license. 

If these terms are unacceptable to you, please do not contribute to the 
repository.

### Contributions
Any work intentionally submitted for inclusion in the RFCs process shall be
licensed under the MIT license.

### The Contributor License Agreement
As part of your first contribution to this repository, you need to accept the 
Contributor License Agreement. You will automatically be asked to sign the CLA 
when you make your first pull request. 

The CLA you sign applies to all repositories associated with the Luna project 
([Luna](https://github.com/luna/luna-rfcs), 
[Luna Studio](https://github.com/luna/luna-studio), etc), so you will only have 
to sign it once at the start of your contributions. 
