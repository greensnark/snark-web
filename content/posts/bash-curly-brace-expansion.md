---
title: "Bash Curly Brace Expansion"
date: 2020-12-08T17:10:23-05:00
author: "Darshan Shaligram"
---

Bash/Zsh [curly-brace expansion](https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html)
is simple but powerful: each permutation enclosed in braces is expanded,
recursively, *before* [glob expansion](https://tldp.org/LDP/abs/html/globbingref.html):

```sh
$ echo a{b,c,d{e,f}}g
abg acg adeg adfg
```

This is handy if you're renaming files far from your working directory:

```sh
$ mv some/long/path/Xyz.{foo,bar} 
```

Which is equivalent to the longer, and decidedly less pleasant-to-type: 

```sh
$ mv some/long/path/Xyz.foo some/long/path/Xyz.bar
```

Another good use for brace expansion is to create a complete directory tree in
one command:

```sh
$ mkdir -p src/{main,test}/java/org/foo/bar
```

To play with and understand complex brace expansions, use echo:

```sh
$ echo mkdir -p src/{main,test}/java/org/foo/bar
mkdir -p src/main/java/org/foo/bar src/test/java/org/foo/bar
```
