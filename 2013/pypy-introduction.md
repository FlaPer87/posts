<!---
$"metadata"$
{
  "md": true,
  "title": "PyPy from scratch (1st)",
  "draft": false,
  "slug": "pypy-from-scratch",
  "tags": [
    "python",
    "pypy"
  ]
}
$"metadata"$
-->

I'm pretty sure many of you have already heard about PyPy and how cool it is, other's most may already know how it works from a high-level POV and others may already know every single detail about it.

I belonged to the second group until a couple of weeks ago when I started contributing to this project - I haven't done much, hopefully I will - and, since I went through the getting to know process I know would like to share knowledge with y'all.

First, lets review what PyPy is.

# Definition

PyPy, is a python implementation written in Restricted Python (rpython), which is a slightly modified version of python with some constraints. For instance, it infers types at compile time.

Other features are:

* JIT
* Different garbage collectors
* Stackless mode
* Memory usage improvement


# Tree Distribution

PyPy working tree might be confusing at the beginning but, after some digging, reading and chatting it becomes clearer. This is what the tree looks like:

    ├── lib-python
    │   ├── 2.7
    ├── lib_pypy
    │   ├── _ctypes
    │   ├── _tkinter
    │   ├── cffi
    │   ├── ctypes_config_cache
    │   ├── numpypy
    │   └── pyrepl
    ├── pypy
    │   ├── bin
    │   ├── config
    │   ├── doc
    │   ├── goal
    │   ├── interpreter
    │   ├── module
    │   ├── objspace
    │   ├── sandbox
    │   └── tool
    ├── rpython
    │   ├── annotator
    │   ├── bin
    │   ├── config
    │   ├── flowspace
    │   ├── jit
    │   ├── memory
    │   ├── rlib
    │   ├── rtyper
    │   ├── tool
    │   └── translator
    └── pytest.py


The ones we care about right now are:

## pypy


This is where the interpreter code resides. The interpreter is the Python interpreter written in RPython. Here we'll find other packages like:

### interpreter:

It contains the parser code, interpreter code and some basic types

### objspace:

It contains basic types (like bools, lists, basestring, ints) and also defines the ObjectSpace (a.k.a space) which is used in pypy to encapsulate the whole Python universe - will talk more about it in a separate post.

### module

It contains modules from the standard library written in RPython. This modules reside here as they need to access interpreter-level objects.

## lib_pypy

As the ones under pypy/module, modules under lib_pypy are modules from the standard library but they were ported to Python instead of RPython.

## lib-python

This is where CPython's standard library modules reside. Some of them may be modified in order to improve performance or for better supporting PyPy itself.

## rpython

This package instead, contains all packages and modules that belong to PyPy's restricted Python version. The package itself contains key parts of this project such as the translator, GC, type manager, etc. All this packages are used for translating and compiling PyPy's code and its modules.

One cool thing about RPython is that it can be used for meta-programing and with a few hints to an interpreter it can also generate a new JIT it.

## pytest.py

This is not exactly a package but it is important as well. Since PyPy's development is mostly test-driven, all implementations must be tested. This script is used for such task. In order to use it, it is necessary to step into the test directory and execute it. For example:

    $ cd pypy/module/itertools/test/
    $ ../../../pytest.py [options]

I'll address PyPy's space, its interpreter and other modules in future posts.