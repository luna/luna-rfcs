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

## Table of Contents

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
'Active', and may then be implemented with the goal of eventual inclusion into
Luna.

The process for creating an RFC can be outlined as follows:

1. Fork the [RFC Repository](https://github.com/luna/luna-rfcs).
2. Copy [`0000-rfc-template.md`](https://github.com/luna/luna-rfcs/blob/master/0000-rfc-template.md)
   to `text/0000-my-proposal.md` (where `my-proposal` is descriptive for the 
   feature).


## The RFC Life-Cycle

## Label Descriptions

C - Language
C - Compiler
C - Syntax
C - Type System
C - Semantics
C - Libraries
C - RFC Process
C - GUI
C - Ecosystem
C - Tooling

T - Breaking Change
T - Non-Breaking Change

P - Low
P - Medium
P - High

S - Needs Summary
S - Waiting on Author
S - Waiting on Community Feedback
S - Waiting on Shepherd
S - Waiting on Team

Open
Accepted
Discussion 
Final Comment Period
Awaiting Final Decision
Postponed
Active
Shepherded

## Credits
This repository and process takes significant inspiration from the 
[Rust RFCs](https://github.com/rust-lang/rfcs) process, and the 
[GHC Proposals](https://github.com/ghc-proposals/ghc-proposals) process, so 
thank you to both communities for inspiring us! 

## License
This repository is licensed under the 
[MIT License](http://opensource.org/licenses/MIT), as specified in the 
[LICENSE](https://github.com/luna/luna-rfcs/blob/master/LICENSE) file.

### Contributions
Unless explicitly stated otherwise by the contributors, any work intentionally
submitted for inclusion in the RFCs process shall be licensed under the MIT 
license.
