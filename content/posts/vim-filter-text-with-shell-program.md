---
title: "Vim: filter text with external programs"
date: 2021-01-02T15:30:01-05:00
---

One of the reasons Vim[^not-just-vim] is powerful is that it gives you easy
access to your shell. If you've invested in developing your skills with the
Unix shell, and in writing shell-scripts, that work also pays off in Vim.

Here's a simple example that I use frequently: let's say I find a slow SQL
statement in my Postgres logs, and I'd like to rewrite it. To rewrite it, I
first want to read it, so I'd like to turn this:

```sql
select person.first_name, person.last_name, person.email, person.phone, person.active, perm.name, perm.created_at from person join person_permission on person.id = person_permission.person_id join permission on person_permission.permission_id = permission.id where person.last_name like 'Fl%one';
```

into something I can read.

Happily there's a command-line tool called
[`sqlformat`](https://github.com/andialbrecht/sqlparse) that formats SQL
queries tidily.  I don't want to leave Vim to use this command-line tool, and I
don't have to: Vim has [`!` filter
commands](http://vimdoc.sourceforge.net/htmldoc/change.html#filter). I can run
`sqlformat` on my query by selecting the line with `!!` (a special case of the
more general `!` to run an external program on the current line) and running
this command:

```
sqlformat -r -
```

This asks `sqlformat` to reformat the SQL query supplied on standard input. Vim
will pipe the selected text to `sqlformat`'s stdin, and replace the selected
text with the *output* of `sqlformat -r -` (on stdout), which is:

```sql
select person.first_name,
       person.last_name,
       person.email,
       person.phone,
       person.active,
       perm.name,
       perm.created_at
from person
join person_permission on person.id = person_permission.person_id
join permission on person_permission.permission_id = permission.id
where person.last_name like 'Fl%one';
```

This may not seem like a big deal at first: you could also do this directly
from a terminal. If you [copy the SQL query into your clipboard, you can
run](../command-line-clipboard-manip):

```sh
$ pbpaste | sqlformat -r - | pbcopy
```

and paste that back into your editor. If you do it often, you could write a
shell script to automate it and perhaps even bind it to a keyboard shortcut.
Even if you do all that though, you still don't get the immediate *in-place
replacement* that Vim affords with `!`.

If you find that your `!` maneuver went badly (perhaps you typoed your
command-line), you can always `u`ndo the error and try again.

You may have a query that spans multiple lines. In such cases I use `!ip` to
run the filter on the entire "paragraph" (`!`: filter, `ip`: inside paragraph).

What if you have multiple queries in one text file? You can run `!ip` once,
and use [`.` repeat](http://vimdoc.sourceforge.net/htmldoc/repeat.html) to
repeat it for other occurrences.

While I've talked about `sqlformat` here, that's just one example. Here are
some other examples:

### Reformat JSON in Vim

If you have machine-output JSON that you'd like to pretty-print in-place, you
can use `jq`.

For instance, given

```
{"a":"b"}
``` 

`!a}jq` (`!`: filter, `a}` [around the JSON's
curly-brackets](http://vimdoc.sourceforge.net/htmldoc/motion.html#object-select),
run: `jq`) will reformat it.

If you work with JSON a lot, consider Tim Pope's [vim-jdaddy
plugin](https://github.com/tpope/vim-jdaddy)

### Run SQL queries

If you have a SQL query in your editor that you'd like to run on a Postgres
database, you can use `psql`

```
SELECT 90 ^ 3
```

`!!psql` will evaluate it. (The obvious caveats apply: `psql` on the command-line
must take you to your preferred database.)

### Run code

If you have some Python code you'd like to evaluate:

```
print(33 ** 5)
```

You can use `!!python` to run it.

## Use visual mode when unsure

When you're new to Vim, you may not be confident in what text you're electing
to run the external program on. Visual mode is handy when you're building your
intuition of Vim's motions and text objects: `vip` will select the query as
before, but visually highlight it so you can confirm you've selected what you
want before you hit `!`. 

An important catch with visual mode is that it doesn't play well with repeating
commands with `.`, and it encourages you to read the visual selection to
cross-check what you're doing. For things you do often, avoid visual mode:
you'll work faster.

[^not-just-vim]: I'll write "Vim" here, but this applies exactly as much to other
  powerful text editors like Emacs. If you use
  [evil-mode](https://github.com/emacs-evil/evil) in Emacs, you can use the
  same syntax as you would in Vim.
