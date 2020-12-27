---
title: "Vim, Orthogonality, and Software Design"
date: 2020-12-27T12:43:10-05:00
---

I learned Vim[^vim] in my first programming job, when a senior engineer who loved Vim
gave my group of new hires a few Vim basics intro sessions. I floundered at
first, then got to the point where I could navigate Vim fairly fast. I could
comfortably use Vim, but I didn't *really get Vim*: I only got good enough at
using it to make it work as well as any other (normal) text editor. 

<!--more-->

Even then, when I didn't really grok Vim, one thing stood out for me.
Vim's s[ubstitute] command[^search-command]:

```
:%s/<search>/<replacement>/[flags]
```

Beyond regular expressions, there was an aspect of `:substitute` that intrigued
me: the search term, the replacement text, and the modifier flags all *lived on
a single command-line* with no UI walls separating the parts.

While graphical editors also allow you to go back and forth between the search
text and the replacement, they need finger-twisting keystrokes like Shift+Tab
(or worse, the mouse) to go back and change your search. Graphical editors
often represent search+replace options as checkboxes or other GUI elements, and
while keyboard shortcuts let you select options fairly fast, typing single
characters is still faster than finger-stretching shortcuts.

With Vim `:substitute`, if you're editing the replacement text and realize your
search needs to be fixed too, you can back up easily to fix
it[^edit-command-line]: it's on the same line.  This "more easily" seems like
an advantage almost too small to mention, but in any interface that you use a
lot, small frictions add up, and change your behavior, often without you even
noticing that your behavior is changing.

Vim's command-line makes commands a *single line of text*, unbroken by
graphical elements. This makes Vim scriptable, and allows commands to
compose. [`:global`](http://vimdoc.sourceforge.net/htmldoc/usr_10.html#10.4), in
particular, made my jaw drop when I understood how powerful it is, and how
useful the idea of composable command-lines is, in general.

While Vim's command-line is powerful, it's not what newcomers to Vim first run
into. Newcomers run straight into Vim's modal editing wall, where they fall
into "normal" mode and find that pressing innocent-looking keys like 'c' and
'p' randomly deletes or duplicates text, and then spend their first few days in
Vim living like hunted creatures in insert mode. Insert mode feels *safe*:
pressing a key does what it says it does on the key-cap, and no hideous
disappearing magic befalls them if they accidentally try to type a 'u'.

Vim newcomers often live in insert mode all the time, only taking brief
excursions into what feels like the cruel hostile "normal" mode when it's
necessary to do things like save a file with `:w`. 

I know how appealing insert mode is for beginners, because this was exactly how
*I* used Vim when I first learned it, in 2000, and how I've seen every other
beginner to Vim behave. It took me quite a while to get comfortable *staying*
in Vim's normal mode.

This is why learning Vim is hard. It's tempting to live in insert mode, but
it's vital to *stay in normal mode when editing text*, because normal mode is
where Vim's power lies. Normal mode commands *compose*.

### Normal mode composition

Vim normal mode has *motions* that move the cursor and *commands* that operate
on a motion[^not-all-commands-do-motions].

Motions like `w` (moving forward by a word) and `F{char}` (move back to the
previous occurrence of `{char}`) move the cursor forward or back. 

Vim commands usually *act on motions*, so, for instance, `cw`[^cw] changes the
text between the current cursor position, and the end of the current word. `yw`
copies ("yanks" in Vim jargon) that same text into Vim's copy/paste registers,
while `dw` deletes that same text.

The way Vim commands combine *orthogonally* with motions makes them really
powerful, because *what a command does* is clearly separated from *what text
the command operates on*.

Why is this powerful?

- Users need only remember a small set of commands (O(c)) and motions (O(m)),
  while getting the O(m * n) benefit of the *combination*.

  Each new command or motion that a user learns is *multiplied* in effective
  power by the number of existing motions or commands they already know.

- Developers of Vim and Vim extensions can add a new command, and it'll
  immediately be able to use the full galaxy of available motions, including
  custom motions that other developers have added.

  This means that two Vim developers who are not even aware of each other can
  write extensions, one providing a new command, and another a new motion, that
  then work together perfectly.

### Vim's UI orthogonality and programming API design

To close, I'd like to touch on how Vim's core design philosophy is similar to
the essence of good programming API design. When software components (I'm
talking here of functions, classes, interfaces, command-line programs, web
services, etc.) are designed *orthogonally*, in the same sense that Vim's user
interface is designed orthogonally, they can be powerfully, easily combined
with other components in ways that the author of the component may not have
envisioned.

Let Vim's clear separation of *what's being done* from *what it's being done
on* inspire you to write programs that have that same separation of concerns. 

The components that you build to write such programs will let you easily build
*similar* programs (viz. when requirements change, you can meet them easily
with your toolkit of components). 

You'll need to write much less code, because a lot of glue code disappears when
components fit together with this clear separation of concerns.

Finally, the code that you write will be beautiful in the simplicity, clarity,
and elegance of its compositions.

[^vim]: I'll write `Vim` everywhere, although what I talk about applies just as
well to vi, nvi, and to newer variants like neovim.

[^search-command]: I was *aware* that there were other ways to run `:s` beyond
  `:%s`, but `:%s` was the only command I used at the time (beyond the staples
  like `:e[dit]` and `:w[rite]`). `:s` introduced me to regular expressions as a
  day-to-day tool. This was in itself a great thing, because I still meet many
  programmers who struggle to write and understand regular expressions, largely
  because they use them so rarely day-to-day.

[^edit-command-line]: It's clumsy to move the cursor around on Vim's
  command-line, but there's no need to: `CTRL-F` gives you the full power of
  Vim to edit the command-line.

[^not-all-commands-do-motions]: Not all commands act on a motion. For instance
[`x` deletes a character, and it can be repeated, but it doesn't use a motion](https://vimhelp.org/change.txt.html#x)

[^cw]: As a historical quirk [`cw` does the same thing as `ce` by
default](https://vimhelp.org/motion.txt.html#WORD)
