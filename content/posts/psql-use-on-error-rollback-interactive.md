---
title: "psql: Use ON_ERROR_ROLLBACK=interactive"
date: 2021-01-26T17:38:51-05:00
---

`psql` is a powerful command-line Postgres client that's useful for both
scripting, and to work with databases interactively.

When working interactively with a database, typos in transactions can be
extremely annoying.

<!--more-->

Let's say you make a mistake on statement 2 of a transaction:

```
$ psql
psql (13.1)
Type "help" for help.

snark=# create table date_activity (at date default current_date, activity_count int4 default 0);
snark=# begin;
BEGIN
snark=*# insert into date_activity (activity_count) values (15);
INSERT 0 1
snark=*# insert into date_activity (activity_count) values (17.a);
ERROR:  syntax error at or near "a"
LINE 1: insert into date_activity (activity_count) values (17.a);
snark=!# commit;
ROLLBACK
```

When working interactively, a mistake in a transaction forces you to roll back
that entire transaction, even if the mistake was a simple typo. This is
quite aggravating if you're several steps into a transaction and then have to
redo everything.

Conveniently, `psql` has a solution: run it with
`ON_ERROR_ROLLBACK=interactive`, and errors in transactions are handled more
reasonably:

```
$ psql --set=ON_ERROR_ROLLBACK=interactive
Type "help" for help.

snark=# begin;
BEGIN
snark=*# insert into date_activity (activity_count) values (15);
INSERT 0 1
snark=*# insert into date_activity (activity_count) values (17.a);
ERROR:  syntax error at or near "a"
LINE 1: insert into date_activity (activity_count) values (17.a);
snark=*# commit;
COMMIT
snark=# select * from date_activity;
     at     | activity_count
------------+----------------
 2021-01-26 |             15
(1 row)
```

Behind the scenes `psql` with `ON_ERROR_ROLLBACK=interactive` wraps every
statement you execute in an implicit
[savepoint](https://www.postgresql.org/docs/current/sql-savepoint.html). If any
statement fails, `psql` rolls back to the savepoint that it created before the
statement, thus preserving the overall transaction.

Setting `ON_ERROR_ROLLBACK` to `interactive` only affects interactive `psql`
sessions where you type in queries from the command-line, leaving `psql`
behavior for scripts unchanged. (This is usually what you want.)

`ON_ERROR_ROLLBACK` is so convenient for interactive use that I have it
permanently set in my `~/.psqlrc`:

```
\set ON_ERROR_ROLLBACK interactive
```
