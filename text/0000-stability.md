___
- **Feature Name:** Ecosystem Stability
- **Start Date:** 2018-02-14
- **Change Type:** Breaking 
- **RFC Dependencies:** 
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:** 

# Summary
This RFC provides a mechanism for managing the evolution of Luna the language as
time goes on. This includes management of breaking changes, as well as the 
notion of a _Rallying Point_ for providing users with coherent releases of 
functionality, even on a rolling release schedule.

# Motivation
As is, Luna currently has no well-specified mechanism for evolving the language.
Releases are ad-hoc, the stability policy is unspecified, and the milestone 
structure of the language unclear.

This RFC aims to resolve the tensions between the competing interests in Luna's
evolution by providing a coherent mechanism for managing language changes and 
stability going forward from the moment the language becomes stable. This 
involves mechanisms for constraining the scope of breaking changes, providing
_rallying points_ for promoting the language, as well as maintaining backwards
compatibility while not leaving people on older compiler versions. 

The current state of Luna's stability is, in a word, _undiscussed_. There has 
been little conversation about what guarantees the language makes and will stick
to as it evolves. This fact is likely to put off many users from trying Luna as
the language's future is an unknown quantity in many respects. 

# The Luna Release Policy
This RFC proposes the following release policy for Luna:

1. Six-week releases using semantic versioning. This implies that Luna will 
   forever be in a `2.x` state as it will aim to maintain backwards 
   compatibility for breaking changes from the `2.0` release.
2. Biannual _Rallying Points_ for the language (e.g. 'Luna 2018 (2.1)'). These
   rallying points provide Luna with the concept of a 'major release', allowing
   for:
   	- The promotion of all major language upgrades since the previous rallying
   	  point.
   	- A focus around documentation and introducing new users to the language.
   	- Highlighting ergonomics improvements in the language 
   	- Opt-in to breaking changes.

Much of the inspiration for this comes from the
[Rust Epoch](https://github.com/rust-lang/rfcs/blob/master/text/2052-epochs.md)
system, and I would recommend reading that before commenting on this RFC.

# Managing Breaking Changes
Now the above may seem a bit of a counterpoint to the management of breaking 
changes. To that end, this RFC proposes the following process.

- Breaking changes in Luna are kept to an absolute minimum.
- The compiler remains compatible with all changes, increasing the burden of 
  making a breaking change (and hence such changes have a high bar, which is
  beneficial for language stability).
- Rather than pinning _compiler_ versions in their projects, users pin _Luna_ 
  versions (the rally points). This means that they can ensure the compiler 
  remains compatible with their code, while being able to upgrade to benefit 
  from ergonomics enhancements to the tooling and ecosystem.
- As packages will pin language versions, there will be a mechanism for allowing
  packages pinned to different rallying points to coexist.  

Such a system will ensure that older code works without issue, meaning that the
Luna ecosystem remains coherent as the language and tooling evolves. 

# Drawbacks
Managing breakage in such a fashion _does_ inherently impose some additional 
cost on the Luna compiler and ecosystem. While breaking changes become opt-in 
with each new compiler version, the compiler version must still be able to 
compile older code at the switch of a flag. Without proper management this is 
liable to introduce unmanageable levels of complexity in the compiler over time,
even if the number of breaking changes is kept to a minimum. 

# Rationale and Alternatives
You may wonder why go through all this effort when so many languages get on fine
with making breaking releases every so often, but Luna exists at a confluence of
use-cases where it is likely that breakage imposes too high a cost on 
maintainers, forever leaving them on old Luna versions. 

Instead, this mechanism provides a way to manage breaking changes without 
sacrificing language stability, and beyond that to still have the ability to 
collate and highlight _major_ releases of Luna as the language evolves.

The obvious alternative is, as mentioned above, to follow the standard model, 
making breaking changes at intervals throughout the language's evolution. These
days, however, this model is becoming more and more dated, with languages able 
to iterate quickly and evolve almost in response to the demands of their users.
Even GHC, known for having a longer-term release model, has started shortening
the release cadence. While this RFC does not necessarily recommend that Luna 
goes with a six-week release cadence or a six-monthly release cadence, there are
numerous benefits around language evolution and improvement brought by not using
such a long release cycle. 

If we do not make decisions on this, or break with what we say, it is very 
likely that we risk abusing the trust of users of the Luna ecosystem. Rust's 
fast evolution, combined with its stability guarantee (see 
[epochs](https://github.com/rust-lang/rfcs/blob/master/text/2052-epochs.md)), 
has been a huge boon to the adoption of Rust as a language, but also the 
stability of its long-term evolution.

# Unresolved Questions
This section should address any unresolved questions you have with the RFC at 
the current time. Some examples include:

- Exactly what stability and evolution guarantees are we happy to make?
- Exactly how do we want to name the major releases?
