---
title: db
description: A simple x86_64 debugger written with ptrace and Udis86
---

A simple x86_64 debugger written with ptrace and
[Udis86](http://udis86.sourceforge.net/)

Rationale
---------

This was an exercise in learning ptrace for an assignment that lead to a
curiosity about how modern debuggers trace code.

Features
--------

db supports single-stepping through assembly, disassembling single
instructions, reading and writing arbitrary registers, and reading and writing
arbitrary memory locations. It also supports intercepting and squashing signals
before they reach the traced process.
