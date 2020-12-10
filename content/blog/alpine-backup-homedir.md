---
title: "Alpine backups: Home directory"
date: 2020-12-10T10:38:37-08:00
categories: [software]
---

I set up a cheap NAS-alike using a [Raspberry
Pi](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/). After a
first attempt using the default OS resulted in an unusable SD card, I did some
digging and decided to use [Alpine Linux](https://alpinelinux.org/), mostly
because it runs by default in Diskless Mode.

In Diskless Mode, the SD card is consulted at boot time, when the entire file
tree is loaded into memory (using `tmpfs`), and then it is left alone. All
operations that write to disk (logging, etc.) go to memory. The next time the
device boots, it comes up the same way. This effectively turns the server into
an appliance, and makes the `reboot` operation return to a known good state.

However, we do want to change _some_ things from the default, and for this,
Alpine provides [Alpine Local
Backup](https://wiki.alpinelinux.org/wiki/Alpine_local_backup), which lets
users save modifications to an overlay file on the SD card, so they "survive"
the next reboot.

By default, the only changes it "notices" are inside `/etc`, but you can
include more directories to "watch".

This largely works as advertised, but one of the hitches I ran into was how to
include a user's home directory, without all the files in it.

Here's how I did it.

## The story so far

Running everything as `root` is a bad idea. Instead, I've created a shell user
to do my day-to-day operations. (It's called `huevo`, 'cause eggs have shells
😬).

```sh
$ adduser huevo2
```

Now we explicitly include the home directory, so that it survives reboot (with
the appropriate permissions):

```sh
$ lbu include /home/huevo
```

## Cleaning up

So far, so good. But now the home directory picks up any changes we make, which
will include things like `.ash_history`, `.viminfo`, and other cruft. To
prevent inadvertent bloat of the overlay file, we don't want to pick up those
things by accident. Instead, we want include only those files that we've
explicitly chosen to stick around.

This is where the documents aren't directly helpful: Running `lbu exclude
/home/huevo/*` actually excludes the home directory as well.

## The solution

It turns out that `lbu` stores its list of added paths in
`/etc/apk/protected_paths.d/lbu.list`. After playing around with the file (and
running `lbu ls` a bunch), it turns out that the order matters! Switching
around the exclude and include statements makes the problem go away:

```sh
$ cat /etc/apk/protected_paths.d/lbu.list
-home/huevo/*
+home/huevo
```

This pattern also works for other paths, like persisting the `~/.cache` folder
without any of the cached files.
