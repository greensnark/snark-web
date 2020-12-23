---
title: "Quoting Strings in Postgres"
date: 2020-12-22T16:45:08-05:00
---

PostgreSQL offers multiple ways of quoting string literals. Most programmers
are familiar with the first, but not the other forms. 

<!--more-->

Let's look at them:

## [Single-quoted string constants](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-STRINGS):

```
$ psql
# SELECT 'What''s up?';
  ?column?
------------
 What's up?
(1 row)
```

Single-quoted literals are what you'll see most often in SQL examples and in
Postgres' own documentation.

If you want a literal quote (`'`) in a single-quoted literal string, doubling the
quote does the trick. This can be slightly eye-watering, though:

```
$ psql
# SELECT '''';
 ?column?
----------
 '
(1 row)
```

## [C-style string constants](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-STRINGS-ESCAPE)

Postgres also supports C-style string quoting using `E'<text>'` (or
`e'<text>'`), where you escape characters by prefixing them with `\`:

```
$ psql
# SELECT e'How\'s my escaping?\n  Not terrible';
      ?column?
--------------------
 How's my escaping?+
   Not terrible
(1 row)
```

This is particularly handy when you need to mix in tabs and newlines into your
strings, and for tricky Unicode characters, like non-breaking spaces, where
writing them literally can confuse readers. Common Unicode characters in the
[basic multilingual plane (BMP)](https://en.wikipedia.org/wiki/Plane_(Unicode))
can be referenced as `\uXXXX`, and characters outside the BMP (such as Emoji)
as `\UXXXXXXXX`, where `X` are hex digits:

```
$ psql
# SELECT e'\U0001F60A\u00A0\U0001F60A';
 ?column?
----------
 😊 😊
(1 row)
```

## [Unicode escape string constants](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-STRINGS-UESCAPE)

Unicode escapes use slightly simpler escape notation for Unicode characters:

```
$ psql
# SELECT U&'\+01F60A\00A0\+01F60A';
 ?column?
----------
 😊 😊
(1 row)
```

With `U&''` strings, use `\XXXX` four-hex-digit escapes for
[BMP](https://en.wikipedia.org/wiki/Plane_(Unicode)) characters, and six-digit
`\+XXXXXX` for characters outside the BMP.

Unicode escapes also support [customizing the escape character with the
`UESCAPE` keyword](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-STRINGS-UESCAPE)
which is occasionally handy if, say, you're trying to feed `psql` text via a
command-line pipeline, and escaping `\` is giving you a migraine.

## [Dollar-quoted string constants](https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-DOLLAR-QUOTING)

Dollar-quoted strings are extremely useful when you must put SQL or PL/pgSQL in
a string, because quoting SQL gets insane fast:

```
$ psql
# SELECT 'SELECT ''foo''';
   ?column?
--------------
 SELECT 'foo'
(1 row)
```

Dollar-strings solve this problem comprehensively. A simple dollar string
extends from `$$` to its paired closing `$$`. This is perfectly suited to defining functions:

```
$ psql
$ CREATE OR REPLACE FUNCTION foo() RETURNS text IMMUTABLE LANGUAGE SQL AS $$ SELECT 'foo' $$;
CREATE FUNCTION
$ SELECT foo();
 foo
-----
 foo
(1 row)
```

Dollar-strings also handle the case where your quoted string itself uses
dollar-strings, by adding a *tag* as `$tag$`. A *tagged* dollar string runs
from the starting `$tag$` to its closing `$tag$`:

```
$ psql
$ CREATE OR REPLACE FUNCTION bar() RETURNS text IMMUTABLE LANGUAGE SQL AS $body$ SELECT $$bar$$ $body$;
CREATE FUNCTION
$ SELECT bar();
 bar
-----
 bar
(1 row)
```

Dollar-strings are also handy outside of defining functions, but less necessary there:

```
$ psql
$ SELECT $quote$He said, 'Give me lots of $$$!'$quote$;
            ?column?
---------------------------------
 He said, 'Give me lots of $$$!'
(1 row)
```
