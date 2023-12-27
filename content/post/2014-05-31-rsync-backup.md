---
title: Rsync Backup
date: 2014-05-31
---

You should run backups. The more backups the better. The more frequent the
better. The more isolated the better.

## The Setup ##

With just $150 and some spare desk space you can set up a backup server. Here's
an example setup:

* Raspberry Pi ($40)
* Powered USB 2.0 Hub ($10)
* SATA to USB 2.0 Adapter ($8)
* Any SATA HDD ($80)

## The Backup ##

If you want a simple backup, you can just run an rsync one-liner:

```sh
rsync --progress -amzh $HOME rpi-host:$(date "+%F:%T")
```

(If you want to know more about what this line does, the rsync man page
is very helpful.)

This is wasteful, however, because it copies files regardless of whether they
have been modified since the last backup. Luckily this problem is easily
ameliorated with the help of [hard links][hardlinks] and the `--link-dest` flag:

```sh
CURRENT=$(date "+%F:%T")
rsync --progress -amzh --delete \
    --link-dest=latest $HOME rpi-host:$CURRENT
ssh rpi-host << EOI
if [ -e $CURRENT ]
    # an alternative is to pass -f to ln
    [ -L latest ] && rm latest
    ln -s $CURRENT latest
else
    echo "[ERR] Failed to create link for backup $CURRENT"
EOI
```

This method is slightly more complicated: once we complete a backup we need
to create a symbolic link to remember the root of the last backup, since we
need to be able to tell rsync about it the next time around.

This script behaves much like the "Time Machine" in Mac OS X: backups will only
allocate new inodes if a file is new or has been modified. This means you can
run backups while only using a minimal amount of storage.

One caveat to this method is that even though a file may be "backed up" multiple
times, if it does not change _it is only truley backed up once_. Each time the
backup runs the same inode is referenced again. If something should go wrong
with that file, it will effect every backup which references it. I mention this
because there is no such thing as a free lunch: the incremental backup scheme
saves space, but is arguably a less effective backup scheme.

## No Excuses ##

It's cheap and easy to set up a backup server that will be invaluable when you
experience data loss. There are many nice features that could be added to this
script (uniquely identifying backups per-host, excluding files via rsync's
`--filter` flag, etc.), but I'll leave these as exercises to the reader.

[hardlinks]: https://en.wikipedia.org/wiki/Hard_link
