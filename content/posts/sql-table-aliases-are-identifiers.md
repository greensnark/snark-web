---
title: "SQL Table Aliases are Identifiers"
date: 2021-04-18T13:49:31-04:00
---

A surprising thing to me working day-to-day is how often I still see queries
that treat SQL table aliases purely as typing-saving shortcuts.

<!--more-->

I often see code (whether it's in a programming language like Go or Javascript,
or in a database view definition or stored procedure) using queries that
treat table aliases purely as a mechanism to reduce typing. For instance:

```sql
SELECT
    u.*
FROM
    users u
JOIN
    memberships m
    ON u.id = m.user_id
JOIN
    groups g
    ON g.id = m.group_id
WHERE
    u.created_at < now() - INTERVAL '300 days'
    AND g.features && array['billing_suspended] 
```

Leaving aside *other* problems in this query[^problems], the use of
short table aliases makes this query much harder to read, particularly as the
query gets longer. The short aliases make the query _easier to write_, at the
cost of making the query _harder to read_.

Optimizing for ease-of-writing over ease-of-reading is always, 100% the wrong
tradeoff, and is a huge part of why codebases decay over time. Code is read
much more frequently than it is written, so the writer _must work harder_ to
make the code easy to read.

In this case you may think the example query is so simple that the short
aliases don't hurt readability much. While this is to an extent true, the
problem is that developers are often in a hurry, and when they want to make a
small change (or what the developer _hopes fervently_ is a small change), they
prefer _following the existing code pattern_ and prefer not to go back and
clean up code.

How would this example query look like with meaningful table aliases? It might
look like this:

```sql
SELECT
    billing_suspended_users.*
FROM
    users billing_suspended_users
WHERE
    billing_suspended_users.created_at < :now - INTERVAL '300 days'
    EXISTS (
        SELECT FROM
            memberships
        JOIN
            groups
            ON
                memberships.group_id = groups.id
                AND groups.features && array['billing_suspended']
        WHERE
            memberships.user_id = billing_suspended_users.id
    )
```

Notice that I haven't bothered giving `memberships` and `groups` table aliases:
table aliases are valuable only when they help clarify _how the table is being
used_, and not merely to shorten the name.

[^problems]: Some other obvious problems in the original query are:

    - The use of `now()` hardcoded in the SQL. Current time references make it
      quite annoying to construct test data. Test data insertion for queries
      referencing the wallclock time directly is also forced to reference the
      wallclock time directly, and is prone to flake. The query should [ask for
      a clock](ask-for-a-clock.md) instead.

    - The joins on `membership` and `groups` are being used not because those
      tables have data that's interesting to the consumer, but rather as *filters*
      on the set of users to return. For readability, it's best to join tables
      only if they have data the consumer wants, and to prefer `EXISTS` clauses
      for filtering.
