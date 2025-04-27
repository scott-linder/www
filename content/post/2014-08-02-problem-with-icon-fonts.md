---
title: The Problem with Icon Fonts
date: 2014-08-02
---

## The Good ##

Everyone loves icon fonts like [Font Awesome][font-awesome]. What's not to
love? You can scale, color, shadow, etc. your icons right in CSS, all without
sacrificing a bit of browser compatibility.

## The Bad ##

With all these great benefits, why would there be a problem?

The issue lies in semantics: the code points which icon fonts use are
meaningless. The regions used are defined as [Private Use
Areas][private-use-areas] by the Unicode Consortium. This means the code points
are not assigned any meaning, but instead this is left to be determined by
third-parties which wish to extend the standard.

This makes sense, but it is analogous to the [Extended ASCII][ext-ascii]
encodings which caused the incompatibilities between systems which Unicode was
designed the resolve. Unicode is intended to unify and standardize character
encodings, but icon fonts exploit a region which subverts this purpose.

## The Ugly ##

Why does this matter, though? The icons look fine and they don't do any harm,
right? Except they pose a problem for anyone who wants to use one font-family
to view all of their content.

Web designers tend to fancy themselves typographers. They style their pages
with various web-fonts. This makes the web an inconsistent, heterogeneous mess
of fonts, some legible ([Droid Sans][droid-sans]), and some not so much ([Comic
Sans][comic-sans]).

While different fonts can be used to convey different moods, they tend only to
make the entire experience less enjoyable and productive.

Some browsers, such as [Firefox][firefox-fonts], ameliorate this headache by
allowing a user to override all custom fonts with a specific set of
user-defined ones (it even allows the user to specify a minimum point size for
fonts, because if it isn't important enough to be in at least 12pt, it's not
important enough to include on the page).

This works well. That is, until a site requires the use of an icon font to
provide meaning to parts of their site which utilize Private Use Areas of the
Unicode standard. Without their custom font, the page suddenly becomes a mess
of Unicode code points (boxes with hex numbers), and a sparse collection of
unrelated glyphs which happen to be defined by your font of choice (note
that these do not usually match the semantics of the desired icon).

## Is this actually a problem? ##

No.

Icon fonts are a perfectly reasonable solution for providing custom glyphs. The
only reason to take issue with them is if you would prefer to always use one
font-family consistently (seriously, check out the [Droid family of
fonts][droid-ttf], they're free and you get consistent Serif, Sans-Serif, and
Monospaced fonts with great Unicode coverage). If you want to do this while
still benefiting from icons you would have to maintain the equivalent to a
whitelist for fonts you are willing to load from sites, which is tedious.

## Is there a solution to this non-problem? ##

Use [SVG][svg] icons.

This may not be an option (e.g. if compatability with every browser since the
dawn of NetScape is required), but it is supported in every modern browser, and
you can still scale and style to your hearts content.


[font-awesome]: https://fontawesome.com/
[private-use-areas]: https://en.wikipedia.org/wiki/Private_Use_Areas
[ext-ascii]: https://en.wikipedia.org/wiki/Upper_ASCII
[droid-sans]: https://en.wikipedia.org/wiki/Droid_fonts
[comic-sans]: https://en.wikipedia.org/wiki/Comic_sans
[firefox-fonts]: https://support.mozilla.org/en-US/kb/settings-fonts-languages-and-pop-ups?redirectlocale=en-US&redirectslug=Options+window+-+Content+panel#w_fonts-dialog
[svg]: https://developer.mozilla.org/en-US/docs/Web/SVG?redirectlocale=en-US&redirectslug=SVG
[droid-ttf]: http://www.google.com/fonts#ChoosePlace:select/Collection:Droid+Sans|Droid+Sans+Mono|Droid+Serif
