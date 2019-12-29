---
title: rustpn
description: A stack-based scripting language
---

RustPN is a stack-based scripting language designed to extend Rust programs
with runtime programmability.

Overview
--------

All execution in RustPN shares one common data stack. It contains typed data,
referred to as "items". Item types include integers (of variable precision,
decided at the creation of the interpreter), 64-bit IEEE floating point
numbers, utf-8 encoded strings, booleans, symbols (think of ruby atoms, but
prefixed with ':'), and blocks. A block is an anonymous function body, which
can contain item literals and calls to functions.

Whenever an item is evaluated that is not a function call, the value
is pushed onto the data stack. Whenever a function call is encountered,
control is immediately transferred to either the native code implementing
that function, or the RustPN block defining it.

Comments are delimited by parenthesis and do not nest.

Example
-------

Here is an example of an iterative Fibonacci function written in RustPN:

```forth
:fib ( n -- n' ) {
    0 1 rot
    {
        over
        +
        swap
    } times
    pop
} fn

1000 fib
```
