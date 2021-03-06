---
tags: [learning, ecosystem]
---

# A Guide to PreProcessor eXtensions

Author: Perry Metzger @pmetzger

> This is a _very very_ rough work in progress. Please help the author
  improving it.

OCaml provides a syntactic extension facility known as "PreProcessor
eXtensions", or **PPX**s. A PPX allows you to add entirely new syntax and
features to the OCaml language, something that is not otherwise possible.

For example, the [ppx_regexp](https://github.com/paurkedal/ppx_regexp)
package adds a **match**-like construct to OCaml that matches strings
with regular expressions, like:

```ocaml
match%pcre somestring with
  | "^foo"       -> some_expression1
  | "(bar){2,3}" -> some_expression2
  | _            -> some_default
```

PPXs are implemented as OCaml programs which are plugged into the
compiler. The extensions look for small syntactic "hooks" that signal
that an extension should do its work. (In the example above, the
signal to `ppx_regexp` extension is the `%pcre` appended to the
`match` keyword.)

The syntactic signals that extensions look for all follow patterns that
are defined by the OCaml language specification, but that mean nothing
to the compiler itself without a plugin.

This document is presented in three parts. The first part gives a
brief informal explanation of how PPXs are used. The second part gives
some detail on how a PPX works and the syntactic hooks OCaml provides
for use by PPXs.  The third section provides some detail on how you
can write a new PPX, and gives pointers to other documentation that may
be of interest to extension writers.

## How PPXs are used

We have already seen an example of the `ppx_regexp` extension, which
creates a version of the `match` construct that checks strings for
regular expression matches.

Another typical PPX is `ppx_deriving`, a family of
extensions that automatically generate code from OCaml data
structures. For example, if you add the annotation:

```ocaml
[@@deriving show]
```

to a data type, as in:

```ocaml
type point3d = float * float * float
[@@deriving show]
```

the `deriving show` plugin to `ppx_deriving` will construct a `show`
function for the `point3d` type that will pretty print it.

The standard OCaml compiler knows nothing about the `ppx_deriving`
extension, of course. The extension exists as a plug-in for the
compiler, and the compiler must be invoked in such a way as to run
that extension over your code.

The documentation for an extension will typically explain how to tell
the compiler how to invoke the plugin. (For example, for
`ppx_deriving` and its `show` component, one usually adds the flags
`-package ppx_deriving.show` when invoking `ocamlfind`.)

## How does a PPX work behind the scenes?

The OCaml compiler operates in several steps. When it reads your
program, the first thing it does is convert the OCaml code into an
internal data structure called an _Abstract Syntax Tree_ (AST).

When you add

```ocaml
[@@deriving show]
```

or syntax from the `lwt` extension

```ocaml
let%lwt foo = ...
```

or the like to your code, the OCaml compiler records the presence of
the `@@` or `%` or similar constructs in your code in the AST.

Software implementing the PPX then walks the Abstract Syntax Tree
looking for constructs it is intended to respond to. (For example,
the `ppx_deriving` framework will look for attribute nodes labeled
with `@@deriving`, and the `ppx_regexp` extension will look for
extension nodes with the name `%pcre`.)

When the PPX finds such a construct, the extension can take fairly
arbitrary actions in response, though typically the extension will
rewrite a section of the the OCaml AST. (For example, `ppx_deriving`'s
`show` plugin will examine the AST of the type that has been annotated
`[@@deriving show]` and will then insert an appropriate `show`
function customized for the type into the AST.)

A PPX converts a valid AST into a valid AST.  This makes it possible to
compose multiple PPX rewriters by piping the output from one as the input to
the next, which happens when multiple PPXes are loaded by the build system.

Since the AST is constructed before type-checking, optimizations, and code
generation, all transformations are purely syntactic. (A PPX can,
however, access compiled modules, including types, though the compiler
library.)

### Syntactic hooks for PPXs

Since the original input must be syntactically valid for the
compiler to parse it, the OCaml language has been systematically
enriched with syntax intended for use by PPX rewriters:

1. Operator names starting with a `#` character and containing more than
   one `#` character are reserved for extensions.
2. `int` and `float` literals followed by an one-letter identifier in
   the range `[g..z|G..Z]` are reserved for extensions.
3. _Attributes_ are named attachments to AST nodes which will be ignored by
   the compiler if uninterpreted.
4. _Extension nodes_ are dedicated AST nodes which will be rejected by the
   compiler if uninterpreted.

For algebraic terms (value, type, pattern, module, module type, class and
class type expressions), extension nodes and attributes take the form

- `[%extension-name expression]`
- `expression [@attribute-name optional-arguments]`

For module and signature items, extension nodes and attributes take the form

- `[%%extension-name module-item]`
- `module-item [@@attribute-name optional-arguments]`

Lastly, we have _floating attributes_:

- `[@@@attribute-name optional-arguments]`

which, unlike other kind of attributes, are independent AST nodes
rather than attachments to existing node types.  They serve as
stand-alone module and module type items.

There are also a number of shortcuts for extension nodes, like `let%ext ...
= ...` for `[%ext let ... = ...]`, which are indistinguishable from their
canonical form after parsing (apart from location information).

In addition, "Quoted" strings of the form `{|string|}` or
`{foo|string|foo}` are not per se reserved for extensions, but are
often of use in them because they allow for strings containing
unescaped `\`, `"`, and similar characters.

(Note that in a string like `{foo|string|foo}`, it is _**not**_
acceptable for an extension to key off of `foo`, the purpose of a
`{foo|` construct is to enable quoting of a literal `|}` inside a
string. See the OCaml manual for details.)

## Writing PPXs

_This section is woefully incomplete. Please help me turn
it into something useful! Send me specific suggestions for things it
should say._

A PPX is implemented as an executable that reads a binary serialized
form of the OCaml AST and writes out a transformed binary serialized
AST.

The OCaml compiler simply runs each of the PPX executables it has
been asked to invoke, feeding each one the serialized AST and getting
it back again.

Thus, a PPX simply manipulates the OCaml AST, nothing more and nothing
less, but it can manipulate the AST in completely arbitrary ways,
subject to the constraint that the output AST must be valid.

The OCaml AST format is quite complicated, so the use of libraries
intended to make the writing PPXs easier is nearly mandatory.

Another reason libraries are important is that the OCaml AST format is
not guaranteed to remain the same between versions of the
compiler. Indeed, the OCaml AST does, in practice, change
significantly between compiler versions. Without the use of libraries
that deal with changes in the AST format, PPXs would break with every
compiler update.

It is therefore necessary to understand a bit both about the OCaml AST
and about the available PPX-writing libraries in order to successfully
write a PPX.

_Insert some explanation of suggested tools here, potentially focusing
on [ppxlib](https://github.com/ocaml-ppx/ppxlib)._
