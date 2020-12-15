---
title: "Commmand Line Clipboard Manipulation"
date: 2020-12-08T18:33:59-05:00
author: "Darshan Shaligram"
---

When working on the command-line, it's often useful to paste from the system
clipboard to a command-line tool, or to copy output from the command-line into
your system clipboard.

On macOS, `pbcopy` and `pbpaste` do this. On Linux, you can use `xclip` or
`xsel`.

<!--more-->

If you have text in your clipboard you'd like to uppercase, a low-budget way of
doing that is:

```sh
$ pbpaste | tr a-z A-Z
```

If you'd like to uppercase your clipboard in place, so you can paste the
uppercased text into some other app, piping it back to `pbcopy` does the trick:

```sh
$ pbpaste | tr a-z A-Z | pbcopy
```

On Linux, I use the [`xsel`](https://linux.die.net/man/1/xsel) tool instead of
`pbcopy`/`pbpaste`:

```sh
$ xsel --output --clipboard | tr a-z A-Z | xsel --input --clipboard
```

`xsel -ob` is a synonym for `xsel --output --clipboard`, and `xsel -ib` is
short for `xsel --input --clipboard`, if you're using these commands
frequently.

I use both macOS and Linux, so my personal scripts include a `clip-copy` script
that's platform independent:

```sh
#! /bin/bash

if [[ $OSTYPE == linux-gnu ]]; then
    xsel --input --clipboard
else
    pbcopy
fi
```

and a `clip-paste` script:

```sh
#! /bin/bash

if [[ $OSTYPE == linux-gnu ]]; then
    xsel --output --clipboard
else
    pbpaste
fi
```
