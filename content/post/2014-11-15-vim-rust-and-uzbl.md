---
title: Vim, Rust, and Uzbl
date: 2014-11-15
---

Vim has a convenient feature called `keywordprg` which allows you to easily
access documentation for the keyword currently under the cursor. This is
typically set to `man`, meaning you can typically only get documentation for
shell scripts, system calls and the C standard libraries.

Rust, however, has [wonderful
documentation](http://doc.rust-lang.org/std/index.html) for their standard
library as well as a few other crates shipped along with it.

To marry these two seems like a great idea, and I must admit that [someone beat
me to the punch there](http://damienradtke.com/vim-view-rustdoc/), but I still
don't find this solution complete for most common browsers (e.g. Firefox,
Chrome, etc).

The problem is that a new browser window or tab will be opened for each search,
but this usually just gets in the way. You typically want to see just the
documentation you have most recently requested, and you always want it to show
up in the same place. This predictability and seamless integration with your
editor allows you to forget about the process of viewing documentation
altogether, which is the ultimate goal.

# Enter Uzbl #

[Uzbl](http://www.uzbl.org/) is a browser based around the UNIX philosophy of
doing one thing and doing it well, and being easily composable with other tools
via text streams. This means that a `uzbl-core` instance is just the pure
essence of a browser: it can navigate to a page and render it. All other
aspects of what one might consider a modern browser are added to this core via
(typically) python scripts.

What makes this perfect for our application is that a single uzbl instance can
be controlled by writing simple text commands to a named pipe. The script for
`keywordprg` then becomes as simple as creating a uzbl instance if it does not
exist, and then sending it a command to browse to the Rust documentation for
the given keyword.

This means the uzbl window will always reflect the latest request for
documentation, without the need for opening more windows or tabs (which aren't
supported by `uzbl-core` anyway: those are a matter of window management).

# uzbl-browser #

uzbl-core would work for our purposes, but since it does not support navigation
it would be more complicated for the user to explore the documentation.

Luckily a set of wrapper scripts around `uzbl-core`, called `uzbl-browser`, have
already been developed. Even nicer, their default configuration very closely
mimics Vim, with a status bar and modal approach to browsing, closing the gap
between editing and browsing documentation even further.

# keywordprg #

The final script looks something like this:
```fish
#!/bin/fish

set SEARCH_TERM $argv[1]
set SEARCH_URI "http://doc.rust-lang.org/std/?search=$SEARCH_TERM"
set UZBL_FIFO_VIM "/tmp/uzbl_fifo_vim"

01234567890123456789012345678901234567890123456789012345678901234567890123456789
# Direct existing uzbl instance to search uri, or spawn one if none exists.
if test -p $UZBL_FIFO_VIM
    echo "uri $SEARCH_URI" > $UZBL_FIFO_VIM
else
    uzbl-browser -u $SEARCH_URI -n vim >/dev/null ^&1 &
end
```

This is written in [fish shell](http://fishshell.com/), but it could easily be
written in any shell or scripting language. I've pushed it to a [GitHub
repo](https://github.com/scott-linder/vim-rusth), and would welcome pull
requests with versions written in other languages.
