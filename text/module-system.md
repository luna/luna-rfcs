# Modules
Modules mean a lot of different things to a lot of different people. Some people think of them as just units of code, while in the ML style they can become first-class language constructs. In an ideal world, modules provide groupings of code, and methods for abstraction. The ML module system is probably best in class so far. As for _why_ modules are important, this [presentation](http://macqueenfest.cs.uchicago.edu/slides/harper.pdf) has more. The key elements are:

- **Signatures:** Provide an interface to which code can conform for a given set of types. ML signatures are more flexible than Haskell typeclasses in that they are structurally typed. If a signature `T` contains the same methods as `Eq`, then a type that implements `T` will implement `Eq`. Signatures can _also_ be predicated on other signatures for a given type, much like typeclass restrictions in Haskell. Haskell typeclasses are thus a _use-case_ of ML modules. 
- **Structures:** Code containers (basic modules) that are canonical implementations of a signature. 
- **Families of Modules:** Inherently, these too are modules, threading type parameters through. A family of modules is defined as the family of types that implement a signature. This is a very flexible notion. Hierarchies of modules provide sum-types, and parametrised modules are product-types. 
- **Subtyping:** Allowing mediation of composition. 
- **Sharing Constraints:** Allowing the expression of coherence conditions.
- **Abstraction:** Genericity over types allowing higher-order modules, but coming with some difficulties on separate module compilation due to dependencies between modules and resolution order. The interaction of this with dependent types is particularly interesting. 

## The Discussion
The discussion here revolves around _what_ we want the Luna module system to be. Do we want first-class modules? If so, what roles should they fill? Can they replace the need for interfaces (typeclasses) through being parametric? The key questions to consider:
 
- **Modules and Imports:** What should the syntax be? How do we do imports using whatever kind of module we decide on?
- **Module Methods:** Effectively free functions in module scope, how do we want these to interact with Luna? Should we allow them, or should we relegate them to object methods. 
- **Extension Methods:** How do we want these to interact with the module system? Uniform Function-Call Syntax?

## Resources for Discussion
This section aims to provide a number of resources with summaries to aid in the discussion of Luna Modules.

### OCaml Modules
Modules in OCaml, unlike Haskell, are [first-class language constructs](https://realworldocaml.org/v1/en/html/first-class-modules.html). The module language is able to work with types and values, while the term language cannot work with modules. This means, that while the modules are first-class language constructs, there is an artificial barrier to expression in OCaml's module system. 

The main features of this module system are:

- Separation of `signatures` and `modules`.
- Modules in OCaml are implementations that satisfy a signature. Bundled together, they produce a first-class model. 
- These first-class modules can be used as values in OCaml. 
- Anonymous modules (useful when working with them as values). 
- Equality for module types is nominal rather than structural, which seems like a downside. Two modules with the same signature cannot be used identically.
- Modules can be pattern matched on like any other first-class construct. 
- Annoyingly, modules parametrised on some type `a` do not, by default, have knowledge of `a`. This is a huge oversight in my view. How would this interact with the notion of a _Locally Abstract Type_.

However, the design of OCaml modules leaves much to be desired, with the separation between module and term languages that doesn't need to exist. This separation also has concerns when it comes to dependent types, as modules being first-class means that they should be able to be computed, much like `Type`s.

### 1ML and First-Class Modules
Andreas Rossberg's [1ML](https://people.mpi-sws.org/~rossberg/1ml/) is a reimagining of ML starting from the concept of first-class modules, thereby avoiding the level mismatch found in OCaml between the module and term languages. It can be thought of as a thin syntactic sugar over System-F-Omega, but it provides a number of useful and interesting points that can inform our modules effort:

- A refinement of the ML type system where _everything_ in the language is a use of modules, allowing for niceties like _runtime_ module selection (e.g. `module Table = if size > threshold then HashMap else TreeMap`).
- The key observation of 1ML is that dependent types are _not needed_ to explain module systems. It has the insight that first-class modules do _not_ have to make typechecking undecideable, and type inference impractical! This may be even more the case in Luna, whose type system is not based on HM-style inference. This is performed through the small and large type distinction, but while 1ML restricts large types (read: abstract types) from being instantiated with other large types, it is possible this restriction could be relaxed in Luna's type system. 
- 1ML finds, through desugaring first-class modules to System-F-Omega, that such modules are syntactic sugar for concepts already able to be expressed in the core language (assuming said language's types can be translated to System-F). 
- Through this refinement of module semantics, it allows for a _minimal_ amount of type annotation. 
- I wonder if the variant of System-F Omega used in [the 1ML paper](https://people.mpi-sws.org/~rossberg/1ml/1ml-extended.pdf#9) is applicable to Luna. 

### Idris Interfaces and Module System
Idris, arguably, is a language that didn't get modules right. Instead of providing a unified concept to handle abstraction and namespacing, it instead provides separate module and interface systems. This is not to say, however, that it has nothing interesting to say about a first-class module system:

- Interfaces in Idris are like Haskell typeclasses, providing collections of functions that can be overridden to provide polymorphism. 
- Idris allows for interface implementation disambiguation. This means that, unlike in Haskell, it is possible to implement an interface (typeclass) multiple times for a type (e.g. multiply and add monoids for `Integer`). 
- This is done via [Named Implementations](http://docs.idris-lang.org/en/latest/tutorial/interfaces.html#named-implementations). While the syntax for naming implementations of interfaces is relatively ugly, the actual functionality is super useful. Consider multiple implementations of `Ord` to trivially allow different sorts. 
- These named implementations can also be used in the constraints of other named implementations (e.g. ensuring the `addNatMonoid` monoid implementation uses the `addNatSemi` semigroup implementation). 
- Idris allows for multi-parameter interfaces, and provides a mechanism to allow the types of some parameters to determine other parameters (`-XMultiParamTypeClasses` + `-XFunctionalDependencies`). We can maybe do better due to the different nature of Luna's type system. 
- Idris modules are simplistic namespaces for code, with the addition of access control and visibility modifiers. We probably want to do better. It does allow changing of the default access mode, however, but that seems awkward to me. 

### Rust Modules
Rust, much like Idris, has a distinct separation between its functionality for abstraction and namespacing. It handles abstraction and provides polymorphism via Traits, and namespacing via modules. Some thoughts.

- Rust allows named implementations of traits, making it possible in the context of genericity to select the correct trait implementation at compile time. 
- The implementation syntax for traits is nasty (_requiring_ duplication of the type signature), so let's not do that. It might be beneficial, however, to _allow_ the implementation of a typeclass/interface/whatever to have its signature provided (e.g. implementing in a different file). 
- Rust enforces the orphan restriction in that one cannot implement an _external_ trait on an _external_ type (e.g. if `pub trait Foo` and `type Bar` both exist in separate crates (packages), you cannot `impl Foo for Bar` in your crate). I don't know enough about Luna's type system to know if the orphan rule would be an issue in Luna. 
- Default implementations are great. 
- One cool thing that Rust does is let you specify a default type for a generic parameter in a trait. This becomes more useful in multi-parameter cases, which allows implementors of the trait to avoid implementing the trait for an _extra_ type as long as the default type works for them. 
- Rust also allows for explicit disambiguation in the case where trait `A` and trait `B` both implement a method `foo`. This means that a type can implement traits with identical names, and the user is able to specify the correct one at the call site (e.g. `<Dog as Animal>::baby_name()`, or where it is less ambiguous `Animal::baby_name()`. It may be possible, though rust does not, to deduce the correct implementation by expected return type. 
- Much like in Haskell you can use the newtype pattern to get around the orphan rule. 
- Modules in Rust, on the other hand, are just simplistic containers of code with some access control. 

## Uses of the Module System
In Luna, the module system can fulfil the roles of:

- Namespacing (`import Foo.Bar ... Foo.Bar.baz`): This means it has a way to store collections of identifiers in a fashion that restricts access to them. These identifiers can be values, types, functions, metafunctions (in the future), etc.
- Abstraction (`foo : (Foo.Quux a) => a -> a`): Much like a Haskell Typeclass, Rust Trait or Idris Interface, these allow for (parametric) polymorphism over a range of types. 
- Generic Implementations: It should be possible to parametrise a module over a type, allowing one to swap the underlying implementation of a module (much akin to Backpack with GHC letting you use `String` and `Text` interchangeably via a signature).  

## Thoughts
This section is just a dumping ground for points of discussion:

- We don't want to have the same restriction as Haskell where only one implementation of a typeclass can exist for a type. We should allow specialisation at the call site if ambiguous. For example, two implementations of `monoid` for `Integer` (`IntegerAdd` and `IntegerMult`) can be chosen between as follows `foldl (+) mempty@IntegerAdd foo` (using a faux-type-applications syntax here to illustrate the point). 
- It could be useful to allow imports into scopes other than global. In such cases, the modules would only be available in the scope of the import declaration:
    
    ```
    class Foo:
        import Bar
         
        Bar.baz
    ```
- Default definitions are important to whatever the Luna equivalent to Interfaces/Typeclasses/Signatures is. 
- How do we want module extensions to work? Similar to typeclasses being restricted to types that match a given typeclass, `module Foo.Bar a` could be restricted to `module Foo.Bar (Eq a) => a` instead.  
- We want to be able to parametrise modules over (a) type(s) such that an appropriate implementation can be selected at runtime (e.g. A module `String` with implementations `FastString`, `Text`, etc).
- Perhaps it is a dramatic change, but classes could be unified with modules, and, providing nested modules, we could get the same functionality. Is there a benefit to having `class` and `module` as separate concepts and constructs? 
- Interactions between module functionality and `Dynamic` (structurally-typed dynamic type). 
- I feel like the Trait + Module or Interface + Module combination is a local maxima. It works well enough for most use cases, but isn't the ideal of what a module can do. In particular, it omits the ability to (backpack-style) parametrise code over a type made concrete at compile time. 

## Notes from the Meeting (2018-04-04)
What follows is a summary of the items discussed during the above meeting with regards to the shape of the Luna module system:

- Merging the concepts of `class`, `module` and `interface`. A properly implemented module system can do this, and in Luna we have the ability to get it right. 
- The final name for this unified concept is undecided, as `module` and `class` both have too much baggage. In the absence of any better options, however, `class` is the best so far.
- The final syntactic form is also undecided, but should work with the syntactic changes coming to Luna.
- Modules are _first-class values_. This means that they can me manipulated at runtime (e.g. selecting an implementation of `Map` based on the expected number of elements), and computed via dependent types. 
- The unification of these concepts makes visualisation in the graphical syntax significantly easier, treating the module as a visual scope. 
- A file must contain only a single module at the top level, disallowing multiple modules and top-level functions. We must, however, consider how this proscription interacts with the main function. The module should be named as for the file (e.g. `Containers/src/Data/Map.luna` can be imported as `import Containers.Data.Map`, making the top level module `Map` available). 
- Import syntax for modules should be able to determine what is imported and where to, as well as any qualification. Some syntactic ideas follow:

    ```
    import Foo
    import Foo as F
    import Foo: <imports>
    import Foo as F: <imports>
    ```

- Modules can be arbitrarily nested, accessed as expected from scopes. 
- Modules can be declared with an arbitrary number of type parameters, and dependent types can be used to determine the correct types in complex cases where certain types determine other types. 
- When used as a class modules work simply as expected:

    ```
    module Map k v = 
        ...

    val = Map.empty
    ```

- When used as an interface, they can either be directly implemented on the type `module Foo : Bar, Baz = ...`, or separately. They can be used to constrain behaviour. 

    ```
    foo : (a ~ Num b) => a -> Int
    bar : (a ~ Foo) => a -> Text
    baz : (a : Num b) -> a -> Int
    ```

    The equation `(a ~ Num b)` states that the type in `a` must behave like an instance of `Num b`, whether it is implemented the normal way or conforms in another way. It is also, of course, possible to constrain the type variable `b`: `a ~ Num Double`. When used as an interface, default implementations of methods can be provided, which can be overriden by implementing types, but the module can also just provide type signatures:

    ```
    module (a ~ Eq) => Foo a b = 
        bar : a -> a -> b
        baz : a -> b
        quux : b -> b
    ```

- Anonymous modules are supported, though the syntax is currently undetermined: `foo = (module = x, y, x).init a b c`.
- As first-class values, modules can be pattern matched on and destructured as expected for a class. 
- While modules should include some visibility specification, this is being left until later, with the current default being set to public. 

There are still some unresolved (at least moreso than the above) points that require further thought and discussion: 

- Will the support for extension methods require orphan instances to be allowed? I suspect that it will, but I am not convinced yet. There are other potential ways to handle this as the issues with orphan instances are large. However, is a newtype any different from defining an orphan instance in the presence of Luna's categorical type system? Not really...
- While it might be useful to allow defaulted types in interface implementations, this would potentially introduce different semantics between the 'type' and 'value' level in Luna. This can't happen as the language is dependently-typed and such a separation does not exist. This will tie into defaulting at the syntax level and, with types as first-class values, should allow a unification of the two defaulting approaches. 
- Should Luna allow for multiple implementations of a module (e.g. `MonoidAdd` and `MonoidMult` for `Integer`)? If so, this would require disambiguation at the call-site. Syntax currently undetermined, but a _bad_ suggestion follows:

    ```
    module Int = ...

    instance MonoidAdd for Monoid Int = 
        mappend = +
        mempty = 0

    instance MonoidMult for Monoid Int = 
        mappend = *
        mempty = 1

    myFunc : [Int] -> [Int]
    myFunc xs = foldl Int@MonoidMult.mappend mempty xs
    ```

- Additionally, if a type implements two modules with clashing names, the same disambiguation machinery could be used to select an implementation to run. 
- Re-exporting from modules should definitely be allowed, but the syntax for doing so is still up in the air.
- What name do we want to use for the unified Module/Class/Interface?
- Think very carefully about how the proposed design features will work in the presence of the category-based typing system and dependent types.
- Luna syntax examples as part of the proposal.
- Idris examples of multiple interface implementations.
- Runtime module selection
- Imports into any scope
- Anonymous modules
- Modules unifying classes and interfaces 
- Export lists + unification with classes gives us a hiding mechanism
- Exports and re-exports
- Differentiate between IS a type and IMPLEMENTS that type



- The syntax is inherently indentation-based, so this means that all code would
  necessarily be indented in the file. This is UGLY.
- Why don't we keep haskell syntax for interface implementation?

- How can we use classes as ADTs and Records using this syntax?

- Things a constraint needs to say:
    + A tyvar has a given concrete type
    + A tyvar conforms to an interface
    + Equality of tyvars
    + Type must be independent from name

- How do we write the quantifiers?  
- How do we define GADT-style dependent data types?

- Wrapping and unwrapping of computations.

```
main =
    foo = @ print "hi"
    out1 = foo.eval
    @ out2 = foo 

    @ foo
```

- Must be able to name and default function parameters.

- Potential Keywords:
    + `module`: Has a lot of baggage.
    + `class`: Also has a lot of baggage, but the baggage is closer to the truth
    + `type`: Quite good if you ask me, as if `module ~ class ~ interface` then
      `module : Type`, `class : Type` and `interface : Type` - all first-class.
      People may expect all types to be constructable though.

- Curried type application in imports
- Modules not matching the filename are private. Imports into module scope are
  exported.
- Top level imports are not exported.
- No computations in the constraints, only in the body.
- Parameter names only below the type signature, but the signature can refer to 
  them (bidirectionally). Defaulting here (partial application applies all the
  defaulted arguments, but can be accessed by name).

    ```
    mkVec : Nat -> Vector n a
    mkVec (n = 1) = ...
    ```

```
topLevel1 : (Num a) => a -> a

topLevel2 : 

topLevel3 : 

module (key : Ord) => Map key val = 
    fromList : [(key, val)] -> Map key val

module (n : Num) => Vector n a =
    head : (forall a, pi n) . (n > 0) => Vector n a -> a

module EfficientMap k v : Map k v = 

    import Data.Num
    import Data.Ord

    threshold : Int
    threshold = 1000

    f : (forall v) . (k : Ord, b : Map k v) => k -> b -> v
    f key map = undefined

    module TreeMap k v : Map k v =
        module Node k v = 
            left  : v
            right : v
            key   : k

        fromList = ...

    module HashMap k v : Map k v = 
        import Util.Hash

        fromList = ...

    EfficientMap : Type
    EfficientMap = if expectedBuckets > threshold then HashMap else TreeMap
```

```
module TreeMap k v : Map k v =
    fromList = ...

module Node k v = 
    left  : v
    right : v
    key   : k
```

- Needs some syntax for anonymous types (as they are first-class values). Do you
  need to have the `type` keyword? Want the ability to make it multiple lined.
  Anonymous records in types.

```
{ x : Int, y : Int, z : Int } . print

# These get progressively more specific, as one mandates names.
foo : { Int, Int, Int } -> Int
foo : { x : Int, y : Int, z : Int } -> Int

# You can also provide function signatures as part of the anon type
# Self is a special identifier.
foo : { Int, Int, Self -> Int } -> Int
foo rec{x, y, fn} = fn rec

foo { x = 0, y = 0, z = 0 }
foo type = 
    x = 0
    y = 0
    z = 0
```

- Is this a general layout rule? How does the non-layout case parse with 
  multiple anon types?

fromList : [(key, val)] -> Map key val
fromList lst = ...

- Need to explain how this ties into Luna's type system in detail. Typing Rules

- Import Syntax (explain the scoping rules for these and their combinations):
    + `import Data.Map`: This imports `Map` into scope.
    + `import Data.Map: fromList, empty`: This imports `Map`, `fromList` and 
      `empty` into scope.
    + `import Data.Containers.Map as MapInterface`: This imports `Map` into 
      scope named `MapInterface`.
    + `import Data.Map String`: Imports `Map` with its first type argument
      specialised to string.
    + `import Data.Map String as StringMap`: Imports `Map` with its first type
      argument specialised to string and names it `StringMap`.
    + `import Data.Map String as StringMap: fromList`: Imports `Map` named as
      `StringMap` and `fromList` with one of its type arguments already 
      specialised to String.

- Named arguments, default arguments and semantics thereof
- You can take either the _behaviour_ and _values_ of a type via use as an 
  interface. If you use it as a _type_ then it determines a category.

PrettyPrinter.luna

```
import Utils.Show: baseShow

# An interface for printing with a default impl.
type (a : Textual) => PrettyPrinter a b =
    prettyPrint : b -> a
    prettyPrint item = baseShow item
```

ToString.luna

```
type ToString = 
    toString : String
```

Point.luna

```
import Util.PrettyPrinter

# This type implements the PrettyPrinter interface directly. It should be noted
# that the annotation is documentation only and could be omitted. Style says 
# that it is a warning to omit this.
type Point : PrettyPrinter Text Point, PrettyPrinter String Point =
    x : int
    y : int

    prettyPrint : Point -> Text
    prettyPrint self = ...

    prettyPrint : Point -> String
    prettyPrint self = ...

# A raw type without any annotations, the compiler warns here if TestBuffer is
# used in a context expecting PrettyPrinter even though the default impl would
# suffice.
type TextBuffer : ToString = 
    encoding : Encoding

    # Buf is a heterogenous list
    buf : [Size, CWord8..] 

    printBuf : TextBuffer -> Text

# This function expects printable things, so both TextBuffer and Point work.
someLogFn : (a : PrettyPrinter b) => String -> a -> Text
someLogFn msg thing = msg <> prettyPrint @Text thing

# This function requires anything with two ints as its value domain
someValFunction : { Int, Int } -> Int
someValFunction { x, y } = x + y

# Example Calls of someValFunction. Both are valid.
someValFunction { 0, 0 }
someValFunction (Point 0 0)

# This function is restricted to the category of Point, which includes values
# and behaviour, so `dot {1, 1} {2, 2}` is not valid.
dot : Point -> Point -> Int
dot p1{x1, y1} p2{x2, y2} = x1 * x2 + y1 * y2 
```

```
# Explicit constructors without named parameters.
type Class1 = 
    Con1 T
    Con2 T2
    Con3 T3

# Autogenerated constructor
type Class2 = 
    x : Double
    y : Double
    z : Double

type Class3 = Class1 | Class2

type XVector a =
    Vector (x : a) (y : a) (z : a)
    
    Scalar (w : a) 
```


- How do we deal with extension methods?
- Unification of Lists and Tuples as part of syntax RFC. How should pattern 
  matching work on these?
- Unification of namespaces, types
- Semantics of nested types:
    + Instance of type held by instance of type?
    + Or separate type held by each instance of type? (`foo.A != bar.A`, where
      `foo : Foo` and `bar : Foo`)

```
type TreeMap k v : Map k v =
    type Node = 
        left  : v
        right : v
        key   : k

    fromList = ...
```

- `prim` keyword? Constructors as primitive entities.
- Could you unify functions with types? Function definitions as shorthand.

```
type Int = 
    1
    2
    3
    4
    5
    ...
```

- If you truly want to represent constructors as values, then all constructors
  should obey the same principle. Constructors describe values and, like any 
  function, should be able to constrain the sets of values they represent.
- Do the type's constructors come into scope on import? No, probably, as this
  causes conflicts.
- Instead, for a `type Vector a = { x y z : a }`, we get `Vector a` in scope and
  an alias `vector == Vector.Vector` so all constructors are lower case. 
  However, this introduces issues with pattern-matching on zero-argument
  constructors. We could also make constructors lowercase in this case. 
- We could just require parentheses around all patterns.
- How does this work with pattern matching on groups?

1. We can define types like this:

   >> type Vector a =
   >>     x y z :: a


2. Value level syntax = type level syntax, so the following is correct:

   >> v = Vector.Vector 1 2 3 :: Vector.Vector 1 2 3
   >> v = Vector.Vector 1 2 3 :: Vector.Vector Int Int Int
   >> v = Vector.Vector 1 2 3 :: Vector Int
   >> foo :: Vector Int -> Int
   >> foo :: Vector.Vector 1 2 3 -> Int
   >> foo :: Vector.Vector Int Int Int -> Int


3. If (2) is correct then we've got a type `Vector.Vector` and we can use it
   everywhere other types are usable. In particular I can use it like

   >> type Foo = Vector.Vector Int Int Int 

   So I can also use it this way:
   
   >> type Foo a b = Vector.Vector a b a
   
   
4. If (3) is correct, then there is a type `Vector.Vector` which behaves in a
   very specific way - you can pass types as its values. It IS a constructor
   but instead of concrete values you pass "group names" (like `Int`). 
