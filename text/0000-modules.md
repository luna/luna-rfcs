___
- **Feature Name:** Modules
- **Start Date:** 2018-06-26
- **Change Type:** Breaking 
- **RFC Dependencies:** Syntax Overhaul
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:** 

# Summary
Any sensible programming language must provide its users with some way to 
modularise their code, splitting it up into well-defined parts. The ML family of
languages, in particular 1ML and OCaml take the concept of a module further and
Luna, with its unique category-based type system, is in a position to provide 
the most flexible implementation of modules yet, unifying the concepts of
modules, classes and interfaces. 

# Motivation
Why should we make this change? What use-cases does it support? What benefits
does it bring to the ecosystem? Explain why the status quo is insufficient or
not ideal.

# Guide-Level Explanation
Explain the proposal as if teaching the feature(s) to a newcomer to Luna. This
should usually include:

- Introduction of any new named concepts.
- Explaining the feature using motivating examples.
- Explaining how Luna programmers should _think_ about the feature, and how it
  should impact the way they use the language. This should aim to make the 
  impact of the feature as concrete as possible.
- If applicable, provide sample error messages, deprecation warnings, migration
  guidance, etc.
- If applicable, describe the differences in teaching this to new Luna
  programmers and experienced Luna programmers.

For implementation-oriented RFCs (e.g. compiler internals or luna-studio 
internals), this section should focus on how contributors to the project should
think about the change, and give examples of its concrete impact. For 
policy-level RFCs, this section should provide an example-driven introduction to
the policy, and explain its impact in concrete terms.

# Reference-Level Explanation
This is the technical portion of the RFC and should be written as a 
specification of the new feature(s). It should provide a syntax-agnostic 
description of the planned feature, and include sufficient detail to address the
following points:

- Impact on existing feature(s) or functions.
- Interactions with other features.
- A clear description of how it should be implemented, though this need not 
  contain precise references to internals where relevant.
- An explanation of all corner cases for the feature, with examples.

This section should be written in comprehensive and concise language, and may
include where relevant: 

- The grammar and semantics of any new constructs. 
- The types and semantics of any new library interfaces.

## Textual Syntax Explanation
This should explain how the new feature interacts with the textual syntax 
representation of Luna. It should touch on the same categories as above.

## Graph Syntax Explanation
This should explain how the new feature interacts with the graphical syntax for
Luna. It should also touch on the same categories as above.

# Drawbacks
A description of why we _should not_ do this. Write this section as if you are
picking apart your proposal.

# Rationale and Alternatives
A few paragraphs addressing the rationale for why this design is the best 
possible design for this feature in its design space. It should address how the
proposed design resolves the motivating factors. 

A few paragraphs addressing alternative designs for the feature, and the reasons
for not choosing them.

A paragraph or two addressing the impact of not including this feature. 

# Unresolved Questions
This section should address any unresolved questions you have with the RFC at 
the current time. Some examples include:

- What parts of the design will require further elaboration before the RFC is 
  complete?
- Which portions of the design will need resolution during the implementation
  process before the feature is made stable.
- Are there any issues related to this RFC but outside its scope that could be 
  addressed in future independently of this RFC? If so, what are they?
