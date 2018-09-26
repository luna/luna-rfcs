___
- **Feature Name:** Futureproof Runtime
- **Start Date:** 2018-08-24
- **Change Type:** Breaking
- **RFC Dependencies:**
- **RFC PR:** Leave Empty
- **Luna Issue:** Leave Empty
- **Implemented:** Leave blank, this will be filled with the first version of
  the relevant tool where it is implemented.

# Summary
A one-paragraph, high-level summary of the proposal.

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

# Notes

## TCO
The current interpreter/runtime performs no tail-call optimisation (TCO),
allocating a new stack frame for each recursive call where the recursive call is
in tail position. The new runtime should definitely support TCO, though the
exact details of this is unknown.

- Should TCO be an optimisation pass enabled with `-O` flags (or equivalent,
  e.g. `+Pass.Optimisation.TCO`)? This is more likely to be useful when we have
  a native target.
- It should likely be enabled by default for the interpreter, especially while
  we aren't doing many other optimisations.
- Needs to allow for _guarded recursion_ (tail recursion modulo constructors)
  to maintain lazy semantics.
- Any such optimisation should not interfere with the semantics of thunk
  evaluation.

## Runtime Design
The following designs should account for both interactive use in Luna Studio, as
well as interactive use in a future Luna REPL.

#### Runtime Design

- Luna's future as a compiled and interpreted language, with the need to support
  multiple backends.
- An effective division of the user experience into two groups of 'layers'. The
  first layer is the portion of the graph being actively manipulated by the
  user, and this should be interpreted. The second layer is the 'rest' of the
  graph, which can be compiled so as to offer the best performance.
- This needs to be more sophisticated than a traditional JIT-style method cache,
  as the nature of Luna's type-system means that we can dynamically compute
  types (dependent typing).
- This native compilation can take time, so need the ability to hot-swap code.
- Need to provide the ability for callers to control opt and deoptimisation
  levels as they see fit.
- The difficulty is what cache-invalidation strategy to use, as types can change
  dynamically in a way not possible with traditional languages (e.g. a change to
  a number can alter types).
- This interacts poorly with specialisation-based optimisation strategies.
- Need to think about how this plugs into the NCG, and think of the interpreter
  as just _one_ level of the compilation stack.
- One of the current issues with the runtime is doing (slow) lookups during
  execution. We want to avoid this as much as possible.
- The machinery around `GHC.Exts.Any` may not be the best way to implement the
  runtime data. There is some overhead associated with it, as well as
  complexity. It may be sensible to use a C-level data model, as this would
  potentially provide better interop with other languages and compilation
  stages.

#### Caching

- Lazy eviction of layers from cache, but connected to the interactive layer.
- Needs to cache the results of computations on the graph. The only things that
  should be recomputed are those _downstream_ from the change.
- However, this cache must also be type-aware.
- The type must also be effect-aware (e.g. IO - watching files on disk, or
  MVars), but default to active evaluation.
- Freeze caching: prevents recomputation of nodes even when not up to date.
- Frozen values should save with the project.
- Frozen values have the ability to reload.
- Multi-layer caches (memory, disk, cache eviction) to control memory usage.
  Size control set in project `config.yaml`.
- Eviction strategies by age, computation time, usage, etc.
- Caching of lazy / infinite structures.
- Dirty flag propagation and invalidation, but with saving of parameters.
- It is possible that the method cache and the value cache can be one and the
  same.
- The ideas presented by [skip](http://skiplang.com/) may have some very useful
  implications for the runtime caching mechanism. 

#### Graph Reduction

- The interpreter is the major part of the typechecker, allowing for the
  evaluation of type level expressions.
- Structural pattern matching for optimisation (e.g. a pure function with
  literal arguments).
- Such graph reductions are optimisation passes.
- How these reductions are propagated through the graph (as they may introduce
  new patterns) is an important point of this design.
- Reduction may elide useful things (e.g. sliders), so could be disabled or
  (hot-pathed).

## Parallelism
As part of its usage in Luna Studio, Luna wants to offer the ability for
automatic parallelism of certain function calls. Furthermore, its use as a
programming language will provide explicit programmer control over parallelism.
The runtime needs to support both of these use-cases.

#### Runtime Parallelism

- The runtime needs to be thread-safe and support both OS-threads and
  lightweight green threading that multiplexes onto OS threads. For now this can
  be provided by the GHC RTS, but the need for this should inform any future
  designs.
- Some functions should be able to be automatically parallelised based on the
  size of their inputs (or other measures).
- This automatic parallelism should be able to be controlled by the user,
  whether through a configuration option or configuration parameter passed to
  the runtime (e.g. `+Runtime.Parallelism.Automatic`).
- Automated parallelism, however, should defer to manually-controlled
  parallelism where relevant (e.g. any situation where the two may conflict).
- Manual parallelism should provide access to OS-threads (e.g. POSIX threads),
  as well as a lightweight green-threading solution.
- Automatic splitting at branches in pure cases or cases with provably separate
  evaluations. This should be a graph-level annotation set by a pass.

#### Design Considerations for Dealing with Parallelism

- Any stateful and side-effecting operations in the runtime need to be carefully
  managed so as not to introduce race conditions.
- Synchronisation for MVars so we can allow for some cross-thread communication.
- Even if they aren't exposed to the users, standard synchronisation primitives
  must _work_ in the new runtime (barrier, mutex, spinlock), as they allow for
  fine-grained concurrency control. Such control would be important for big-data
  applications where parallel data structures can be implemented using them
  (e.g. persistent tries).
- The concurrency should be implemented at as low a level as possible,
  potentially using the GHC RTS directly. The more layers we pile up, the more
  performance we lose.
- The concurrency needs to work with _both_ interpreted code and compiled code
  so as to provide a seamless experience. This means the runtime has to have
  direct access to ABI hooks in compiled code where necessary.
- Whatever is done, it mustn't cripple performance like the Python GIL does.

## RTTI
While Luna is currently simply typed (no pi-types), we fully intend for it to
evolve into a truly dependently-typed language over time as we upgrade the type
system. While we already have plans for this, the runtime needs to support it
too.

#### Types at Runtime

- The issue with dependent-types is that you can't erase _all_ type information
  at runtime. Unless this is designed sensibly, it is very possible that keeping
  extraneous type information around could bloat the runtime.
- If we can afford the memory cost, it may be better to retain _all_ type
  information at runtime, even in the compiled code. This would speed up the
  flow for optimisation and deoptimisation (and potentially yield improvements
  for the typechecker).
- If we do this, type information should be retained inline in the expression
  context as much as possible. The more indirections done to look up type info,
  the slower it is to work with type-level computations (bearing in mind that
  type and value-level computations are executed by the same interpreter).
- Any runtime type support shouldn't make any assumptions regarding the type
  universe of the types on which it is operating. Doing so would preclude future
  introductions of additional type universes such as
  [Affine Types](http://asaj.org/talks/lola16/?full#cover),
  [Linear Types](https://ghc.haskell.org/trac/ghc/wiki/LinearTypes) or
  [Uniqueness Types](http://docs.idris-lang.org/en/latest/reference/uniqueness-types.html)
  ([Substructural Types](https://en.wikipedia.org/wiki/Substructural_type_system)
  in general).
- While Luna is statically typed, the runtime manipulation of types provides
  less opportunities for usage-analysis based erasure than languages like Idris
  or Agda. However, it is likely still possible that we can apply a
  usage-analysis pass to the Luna Core graph.
- The analysis of _relevance_ of type information is interesting, and
  potentially we can learn some lessons from the progress of
  [Dependent Haskell](https://ghc.haskell.org/trac/ghc/wiki/DependentHaskell)
  in GHC.

## ESA
Future plans for Luna also include embedding other languages within Luna code to
allow for seamless interoperability with said languages. Designing this
properly, and making it easy to write new bindings, will require some level of
support in the runtime. This system is known as the Embedded Syntax Architecture
(ESA).

#### Runtime Design for ESA

- Ideally, we want zero-cost data access between the languages participating in
  ESA linkages.
- This means that the runtime's data model needs to be low-level enough that any
  language can reasonably interact with it.
- It also needs to interact with ESA in a thread-safe manner.

## Evaluation Strategies
While Luna is currently a lazy language, we have plans to make it strict (with
optional laziness) in the future.

#### Laziness and Strictness

- Should be able to handle a mixture of lazy and strict data, with support for
  'viral' laziness annotations.
- Should be able to handle functions with a mixture of lazy and strict
  arguments.
- It should have well-defined evaluation strategies for both lazy and strict
  code (e.g. call-by-need for lazy code, call-by-name for strict code).
- The interpreter's design needs to be agnostic to the evaluation strategy.

## FFI
Luna's FFI currently only supports calling C ABI functions via the use of
libffi. The runtime can likely support dynamic function calls in a slightly
better fashion, and potentially extend this to different languages.

#### FFI in the Luna Runtime

- The FFI should be tied to the current back-end. As we want to support multiple
  backends, the JS backend should allow for JavaScript FFI, while the ASM
  backend should allow for C FFI.
- This flexibility should extend to the interpreted runtime.
- FFI calls should be as close to zero-overhead in the dynamic runtime as
  possible, while in compiled code they should be zero-cost.

