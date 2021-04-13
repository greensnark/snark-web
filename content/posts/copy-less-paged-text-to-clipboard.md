---
title: "Copy less-Paged Text to Clipboard"
date: 2021-04-12T21:18:56-04:00
tags: ["shell", "less", "clipboard"]
---

When viewing text in the pager
[`less`](https://en.wikipedia.org/wiki/Less_(Unix)), it's often handy to copy
it to your clipboard. You can do this by typing (at the less prompt):

`|$ [clipboard-copy-program]`

You must do this from the *first line of the output you want to copy*, and this
will copy all output to the end (`$` is the automatic less mark for the
end-of-file)

For instance, on macOS you'd type:

```
|$ pbcopy
```

On Linux you'd use `xclip` or `xsel`:

```
|$ xclip -selection clipboard
```

or

```
|$ xsel --input --clipboard
```

Or if you switch frequently between macOS and Linux, use a script to choose
between `pbcopy` and `xclip`/`xsel`:

```sh
#! /bin/bash

if [[ $OSTYPE == linux-gnu ]]; then
    exec xsel --input --clipboard
else
    exec pbcopy
fi
```

Needing to copy something from less is common. For instance, you may be running
or `EXPLAIN`ing a query in `psql`, and want to copy the output for use
elsewhere.
