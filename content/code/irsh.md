---
title: irsh
description: Internet Relay SHell
---

Internet Relay SHell

Rationale
---------

Both IRC and Unix shells share a line-oriented, text-based interface. Many IRC
bots follow the pattern of invoking commands and passing arguments, but do not
allow for composition of commands. The Unix shell (through pipes and
redirection) makes composition of commands and filters simple.

Thus irsh hopes to achieve the same, but in the restricted context of an IRC
channel, and with the reuse of as many Unix utilities as possible (with only
slight interface modifications).

Overview
--------

The bot's core is written in Python3 and requires no non-standard
libraries.

The rest of the bot is intended to be written in Unix shell, specifically
[fish](http://fishshell.com/), and in general the core commands require
only core utilities.

Use
---

The bot identifies commands with a prefix (typically `$`), and then
uses the `shlex` python module to tokenize the input. The rest works
as you expect from any Unix shell, with piping, and output redirection
(to simplify permissions only append is allowed).

Filenames are restricted to alphanumeric characters, and IRC channels are used
as "directories" (with the channel where the command originated being
considered the current working directory), allowing for each channel to have
its own namespace, while still allowing use of files across channels through
absolute pathnames.

For example, in a network with two channels, `#one` and `#two`,
if we append to a file in `#one`:

```
<stringy> $echo this is in #one >> one
<stringy> $cat one
   <fish> this is in #one
```

And then we want to access it from `#two`:

```
<stringy> $cat one
   <fish> cat: var/root/#two/one: No such file or directory
<stringy> $cat /#one/one
   <fish> this is in #one
```

We can simply use an absolute path.

You will also notice that the error message fish gave for a missing file
includes `var/root`, which is where fish stores files, relative to where it is
started. The most important component in the resolution of filenames is the
script `lib/path` which takes the context of the pathname (i.e. the current
channel) and crafts a real path to the actual file in `var/root`. This prevents
users from escaping the `var/root` "chroot".
