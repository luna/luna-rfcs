___
- **Feature Name:** Comments Overhaul
- **Start Date:** 2018-02-01
- **Change Type:** Breaking
- **RFC Dependencies:** 
- **RFC PR:** 
- **Luna Issue:** 
- **Implemented:** 

# Summary
Comments are an oft-overlooked part of a language's design, but they serve many 
purposes to the programmer. From disabling code to documenting the codebase,
comments are an incredibly useful tool. Luna's dual-syntax representation, 
however, means that the design of comments requires special consideration. This 
proposal provides a fully-specified set of comments for the language, filling
all of the niches that comments fulfil.

# Motivation
With comments serving such a multi-faceted role in today's programming 
languages, it is particularly important that Luna gets their design right. This
means that Luna must have support for all common types of comments, including
marking tasks to be finished or fixed, documenting the code, disabling code for
debugging purposes, and even the luna-specific freezing of code execution for
certain portions of the code. 

Luna is a powerful, general-purpose language, and getting the commenting syntax
correct is hence of great importance. This means considering the usability of 
the different kinds of comments, even to the point of breaking existing code
while in beta. As a result, this RFC proposes a total overhaul of the current
commenting system, and fully specifies the behaviour of all the comment types 
that Luna aims to support. 

In doing so, this RFC proposes breaking changes with regards to the current
syntax of comments, but in doing so produces significant usability enhancements
for any of the individuals working in Luna. 

# Guide-Level Explanation
So you've been learning Luna and you're wondering exactly how to comment your 
code. Now Luna, as you've probably come to see, is a fairly opinionated language
and this extends to the commenting system. While you can easily perform all of 
the standard commenting functions that you're used to, the way you do them will
be slightly different. 

## Commenting Out Code
You're working on a feature and something isn't going quite right. You've tried
to isolate the problem using the graph's visual debugging, but that hasn't 
helped, so you're going back to that old standby of commenting out code until
you find the problem.

To do this, Luna provides the 'disable comment' in both the visual and textual
syntaxes. Disable comments in Luna are very similar to commenting out code in 
any other language. Just prefix a line with `##` and it'll stop the code from 
executing. If you're in the graph view, select some nodes, and hit `Ctrl+/`
(or `Cmd + /` on MacOS), and they'll be commented out!

```
def main:
    ## hello = "Hello, World!"
```

There are, however, two major differences with how they work in Luna. If you
comment out a line, it is still parsed. This means that the graph can display
corresponding nodes, just greyed out. This is great, as it means you can still 
see the structure of your code no matter which syntax you prefer. Furthermore,
there are certain cases where Luna can make 'implicit connections' between your
nodes to keep execution running. For a full description of when this applies, 
see [Disable Comments](#disable-comments) later in this document.

The other major difference, is what happens when you comment out a block. If you
disable a line that introduces a block scope, this is sufficient to disable 
_all_ of the expressions contained in this scope. In the following example, the
single disable comment actually comments out the entire block down to 
`print "hi!"`, so the only code that will run from `func` is that line.

```
def func a:
    ## if a then foo else:
        bar 
        baz
        quux

    print "hi!"
```

## Reminders in Your Code
It is not uncommon to be working on a new feature and wanting to get out an MVP
as quickly as possible. As always, this means that something gets left out, and
you want to add something to document that fact so you can come back later.

Luna features `TODO` comments for this purpose, allowing you to annotate lines
of code. A todo comment looks like `# TODO`, followed by whatever string you 
want, and can even span multiple lines (as they get concatenated together up to
the first blank line). Todo comments are known as 'tagging comments' in Luna, 
and are part of a documentation comment. They have specific visual styles
associated with them in debug documentation, and are not included in the 
rendered documentation in release mode. 

You can use these comments on the graph as well. Each node has a 'comment' 
button, and pressing this will open up a text field. Anything you type in here
is automatically prefixed with the `#` for translation into the textual syntax,
so you don't need to do that!

Along similar lines as todo comments, Luna also supports `FIXME` and `BUG` 
annotation comments that are used in the same way, each with a distinctive 
visual style:

```
def broken a b:
    # FIXME Currently broken due to incorrect logic in `frob`.
    frob a b
```

## Documenting Your Code
Like any good programmer, you know that documentation for your code is just as
important as the code you write, and unlike many languages, Luna provides a
versatile integrated system for producing doc-comments. 

These are large, multiline comments that support a variety of easily-readable
text-formatting measures to allow the documentation to be as pretty in the 
source code as it is when rendered. The documentation comments work in both
textual (as expected) and graph syntaxes, using the 'comment box' mentioned 
above on the visual graph. 

Doc comments in Luna can apply to _any_ line of code, not just declarations of
larger constructs as seen in many other programming languages. This means that 
you can carefully document anything you see fit!

There are a few things to keep in mind when writing your documentation comments:

- The first line should be a short explanation of whatever you're documenting, 
  as this line will be used when space is limited. This line is defined up until
  the first blank line in the comment.
- As part of the rendering process, a function signature, with types, will be
  included in the documentation, even if you don't explicitly specify the type
  signature. 
- Keep your doc comments short and to the point as, in Luna, we value code that
  is self-documenting.

Now the documentation comment syntax is best explained by example, rather than
a list. If you really want a fully-specified list of all the syntax, you can 
look at the section on 
[Documentation Comment Syntax](#documentation-comment-syntax). A basic example 
of a documentation comment can be seen below:

```
# A brief summary of what my function does.
# 
# Some more detail on what the function does. This can span multiple lines, and
# these will be automatically concatenated as long as I don't leave a
# 
# blank line.
# 
# Block Heading
#   This is a block, and here I can talk about whatever is important to the 
#   block. In here I have a numbered list:
#     - First Number
#     - Second Number
#
#   But within the same block I can also have a bulleted list:
#     * First bullet
#     * Second Bullet
#       * Bulleted sub-list, but you can mix numbers and bullets.
# 
# Sometimes I might want to embed a code block in my documentation as a usage
# example. I can do this as `frobnicate foo bar`, but for bigger blocks I can:
#
#   > import Math
#   > frobnicate "a" (Math.sqrt b)
# 
# `a`: Here I'm making an item reference to the first function argument.
# `b`: Here I describe the second. 
#
# `return`: Here I can talk about the return value of the function.
# 
# You can find more information about doc comments in Luna by following this 
# [link -> https://luna-lang.gitbooks.io/docs/].

def frobnicate :: a -> Int -> Text
def frobnicate a b:
    ...
```

# Reference-Level Explanation
This RFC proposes the removal of the syntax and semantics for all current 
commenting functionality in Luna, and in their place proposes the following new
comment types:

- **Documentation Comments:** These comments allow for the documentation of Luna
  code, and are associated with the line they are situated above.
- **Tagging Comments:** These are the often-seen `TODO`, `FIXME` and `BUG` 
  comments, and consist of the tag and a description. They are associated with
  the line below the comment.
- **Disable Comments:** These comments disable a line or block of code, meaning
  that the code in question is not executed by the Luna runtime.

The visual and textual syntax, and semantics for such comment types are 
described in the following subsections. 

## Documentation Comments
Documentation comments in Luna support a simple, human-readable syntax that
allows for the creation of nicely-formatted function declarations. A doc
comment can be associated with any line of Luna code, and can be used to 
document it. Such comments are associated with the line of code that they appear
above (e.g. with the function if appearing above a function definition), and 
hence become associated with a node in an identical fashion to the tagging 
comments mentioned above. 

Documentation comments are very similar to tagging comments, but the syntax is
far more freeform. They conform to the following grammar:

```
doc-comment = "# ", {doc-comment-char};
```

The parsing of documentation comments is a little more complex than for the 
other comment types, using a custom syntax. They conform to the following rules:

- Documentation comments are associated with the line above which they sit.
- Documentation comments may be associated with any line of code. 
- Subsequent document comment lines are combined into a single block. Lines in
  this block that conform to the specification of a tagging comment (see above)
  are not considered to be part of the documentation. Such lines are listed 
  separately.
- Once the extent of the comment block is determined, the leading `# ` is 
  stripped from each line, and the resulting block of text is parsed according
  to the documentation comment syntax listed in the below section.

### Documentation Comment Syntax
The documentation comment style is intended to ensure that everyone writing doc
comments will write using the same style. This is to ensure that any Luna code
is familiar to anyone writing Luna, and that there are not significant 
variations in style between codebases. It supports the following features:

- **Line Concatenation:** Multiple Lines at the same indentation are 
  concatenated unless separated by a blank line. 
- **Paragraphing:** Paragraphs are separated by one or more entirely blank 
  lines.
- **Sectioning:** Sections are created using indentation. Each indentation of
  two spaces creates a new level of section. The line immediately above the
  indented block is rendered as the section heading.
- **Lists:** Luna documentation comments support both bulleted and numbered 
  lists. To create a bulleted list, indent the line by two spaces, and use an
  asterisk: `*`. For numbered lists, indent the line by two spaces, and use a 
  standard dash: `-`. Further levels of indentation will create sub-lists. 
- **Item References:** Item references obey the following grammar. An item 
  reference allows the documentation to reference a name in the code (e.g. an
  argument to a function), and provide associated documentation. They must refer
  to valid identifiers, otherwise a parse error is raised. There is a special,
  reserved reference `return`, for documenting the return value of a function.
  These item references are automatically parsed within code snippets (see 
  below), and are linked to the references. 

```
item-reference = "`", identifier-name, "`: ", {" "}, {text-char};
```

- **Code Snippets:** You can embed inline code snippets by using backticks to
  begin and end the block. These characters may be escaped using `\` to occur 
  within the block. You may also create a multiline code-block using a two-space
  indent, and beginning each line of the block with `> `. 
- **External Links:** External links can be embedded using the following syntax:
  `[link name -> link]`.
- **Images:** Images can be embedded using a similar syntax 
  `[# image name -> image-path]`
- **Text Formatting:** Luna documentation comments also support basic textual
  formatting, using the `_` to denote italics, and the `*` to denote bold.

### Documentation Comment Rendering
Upon rendering documentation comments, the parser will display the full, typed
function signature, even if the signature is not provided by the user. 
Furthermore, if items are tagged using item references, a table will be created
that displays the item name, type, and associated description.

Finally, the first line of the doc-comment block is used as the summary for the 
documentation comment. This means it will be displayed anywhere the full
documentation cannot be. 

## Tagging Comments
Tagging comments are associated with the line of code that they appear above. 
They have no special syntax, and are more defined by convention than anything
else. Tagging comments occur within 
[Documentation Comments](#documentation-comments), and produce particular visual
styling in the generated documentation (in debug mode). In release mode, they 
are not included in the generated documentation.


They conform to the following grammar (using EBNF syntax).

```
tag-comment = "# ", ( "TODO" | "FIXME" | "BUG" ), {comment-char};
```

Successive lines beginning with a `#` are considered to be part of the same
comment unless the line is otherwise blank. Any subsequent lines are then 
considered to be part of the rest of the documentation comment. In the visual 
syntax, each node has a button to reveal any associated comments, and tagging 
comments will be included in this area. 

In the following example, the `TODO` comment is associated with the function
definition, and would appear on the function's node. It is also possible that,
within the function's scope on the graph view, that there will also be some way
to display associated comments.

```
# TODO: Frobnicate the arguments
def myFunc a b:
    ...
```

Should a tagging comment fail to parse, it will just be treated as a standard
doc comment description without any special visual style.

## Disable Comments
Disable comments are used to prevent execution of a given line of code. They do
not prevent parsing of the code, as comments are not stripped by the Luna 
parser. As a result, nodes corresponding to disabled code can still be 
displayed, though no execution takes place. These comments conform to the 
following grammar (using EBNF syntax):

```
disable-comment = "##", [" "], {code-char};
```

The line following the `##` is parsed by Luna but not executed. If this line 
corresponds to the introduction of a block, the entire block is considered to be
disabled by the application of the disable comment. 

An example of such a comment can be seen below. In this case, the expression 
assigned to `hello` is not evaluated, but a node corresponding to this 
expression is still displayed on the graph.

```
def main:
    ## hello = "Hello, World!"
```

As part of making this more functional, Luna performs some automatic connection
of nodes from input to output where possible. It operates based on the following
guidelines:

1. Luna is an Object-Oriented language, and hence each node has a special 'self'
   port. This self port is used as part of the 'main-line' of processing, often
   made visually distinct in the graph. In text, this is a group of variables:

    ```
    a = foo1 ...
    b = a.foo2 ...
    c = b.foo3 ...
    d = c.foo4 ...
    ```

   In this case, disabling a node means 'take no further action', and hence we 
   can safely connect the node chain. If `## c = b.foo ...` is disabled, it has
   the effect of `c = b`.

2. If we disable a line that does not bind variables, it does not have any 
   special meaning. `## print "foo"`, for example, just makes this node 
   non-executable.
3. When disabling a line that does not use the 'self' port, we connect the 
   first-used variable through if the types would match. In other cases, we 
   cannot make it work, and execution is halted. For example, if we have 
   `## x = foo a b`, and the types of `x` and `a` are the same, there is an 
   implicit creation of `x = a` so execution can continue. 

# Drawbacks
The main drawback to this proposal is that it is a breaking change for Luna's
existing commenting system. It entirely overhauls the syntax for, and behaviour
of, comments in Luna, and hence should be given due consideration before the
change is approved. However, as Luna is currently in Beta and hence not 
considered to be stable, now is the best time to make a breaking change to 
ensure that comments function as we want.

# Rationale and Alternatives
While this does proposal does suggest a major breaking change, it is making sure
that the beta period is used to ensure that commenting in Luna is up to scratch.
This RFC presents a coherent specification of how the different comment types in
Luna should behave, as well as how they interact with each other. 

While much of this functionality could have been retrofitted onto the existing
commenting system, it is exceedingly likely that this would've led to some
significant and unanticipated edge cases, due to the lack of a proper spec for
the old system. 

The other main alternative would have been to leave the commenting system as it 
is, but this would have been a mistake due to its current lack of functionality.

# Unresolved Questions

- How should we visually indicate disable comments in the textual syntax?
