---
tags: [ecosystem]
---

# Foreign Function Interface

Interop with C is the most natural for OCaml, since the runtime is partially written in C.

## C

* [ctypes](https://github.com/ocamllabs/ocaml-ctypes): Modern library for binding to C libraries using pure OCaml.
You should always try to bind to C code using ctypes before trying the older method.
* [cstruct](https://github.com/mirage/ocaml-cstruct): Library to access C-style structures from OCaml.
It comes with a ppx preprocessor and variants for lwt and async.
* The older way of binding to C code is to create stubs in C which call functions in the OCaml runtime.
This method is error prone, and you're advised to stay away from it unless it's absolutely necessary.
* [ocaml-main-program-in-c](https://github.com/johnwhitington/ocaml-main-program-in-c) – Example build system for making mixed C/Ocaml binaries where the main program is in C.

## Python

* [Py](https://github.com/zshipko/ocaml-py): Library for interacting with Python 3.5 using Ctypes.
* [Pyml](https://github.com/thierry-martinez/pyml): Library for interacting with python. Uses C stubs.
* [lymp](https://github.com/dbousque/lymp): Another library for interacting with Python, this one using a Python process.

## Articles

* [Modular foreign function bindings](http://openmirage.org/blog/modular-foreign-function-bindings) with ctypes.
