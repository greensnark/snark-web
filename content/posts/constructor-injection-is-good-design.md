---
title: "Constructor Injection is Good Design"
date: 2020-12-17T16:38:50-05:00
---

A key aspect of writing good code is asking for your dependencies as inputs,
instead of hard-wiring references to your dependencies in the code, viz.
[dependency injection](https://en.wikipedia.org/wiki/Dependency_injection)

<!--more-->

As an example, if you have a class that needs to make a query, you could
directly reach out and grab a connection:

```kt
class DatabaseIntrospector {
    fun allTables(): List<Table> =
        DatabaseConnection.statement { statement ->
            statement.execute("SELECT * FROM information_schema.tables").resultSet.use {
                // ...
            }
        }
}
```

Hard-wiring access to the database connection like this is convenient, but it
makes the code less reusable. For instance, if you want to query two different
databases, the code with the hard-wired reference to a database connection
won't work.

Dependency injection is the answer: instead of a component *knowing* where its
dependencies are, the component *asks* for its user to give it what it needs:

```kt
class DatabaseIntrospector(private val statementSupplier: () -> Statement) {
    fun allTables(): List<Table> =
        statementSupplier().use { statement ->
            statement.execute("SELECT * FROM information_schema.tables").resultSet.use {
                // ...
            }
        }
}
```

Now if we want to introspect two different databases, we can create two
different database introspectors:

```kt
DatabaseIntrospector(databaseConnectionA::createStatement).allTables()
...
DatabaseIntrospector(databaseConnectionB::createStatement).allTables()
```

That's dependency injection in short: by *asking* for what you need, you gain
reuse. You also gain testability, which is often reuse in disguise. If code is
hard to test, it's usually because it's hard to reuse.

Since `DatabaseIntrospector` asks for its dependency as a constructor
parameter, this is an example of constructor dependency injection.

There's another dependency injection pattern that's common in the Java world,
viz. setter/field injection:

```kt
class DatabaseIntrospector {
    lateinit var statementSupplier: () -> Statement

    fun allTables(): List<Table> =
        statementSupplier().use { statement ->
            statement.execute("SELECT * FROM information_schema.tables").resultSet.use {
                // ...
            }
        }
}
```

With setter injection, a `DatabaseIntrospector` instance is now used as:

```kt
DatabaseIntrospector()
    .apply { statementSupplier = (databaseConnectionA::createStatement) }
    .allTables()
```

This is obviously clumsier than constructor injection, but it has a more
subtle, yet serious drawback: it introduces *mutable state*. A
`DatabaseIntrospector` instance can now be in two possible states:

1. A `DatabaseIntrospector` without a `statementSupplier`, aka. "this instance
   is a ticking time-bomb"

2. A `DatabaseIntrospector` with a `statementSupplier`, aka. "this instance is
   safe to use, as long as no other thread is trying to swap in a new
   statementSupplier"

These are *serious* drawbacks, yet setter/field injection is stubbornly
popular in production code. *Why* is setter injection popular, when developers
widely acknowledge its drawbacks?

Setter/field injection remains popular because poor design choices in Java (and
early versions of Spring) make setter injection easy. Java's verbosity actively
encourages poor code design. The *correct* behavior (constructor injection) is
clumsy and painful in Java:

```java
@Component
public class DatabaseIntrospector {
    private final Supplier<Statement> statementSupplier;

    public DatabaseIntrospector(Supplier<Statement> statementSupplier) {
        this.statementSupplier = statementSupplier;
    }

    ...
}
```

But the *bad* behavior (field injection) is trivial in Java:

```java
@Component
public class DatabaseIntrospector {
    @Autowired private Supplier<Statement> statementSupplier;

    ...
}
```

When given a choice between correct and easy, inertia will push programmers
toward easy (and you can't blame them).

In constrast with Java, Kotlin makes the correct design choice easy.

If you're stuck with Java, there are some mitigations to the verbose
constructor problem, like [Lombok](https://projectlombok.org/).

Setter injection is only evil in production code (or, more broadly, in reusable
code). Using setter injection for test dependencies is perfectly fine, since
test cases aren't reusable code. This is fine:

```kt
@RunWith(SpringRunner::class)
@ContextConfiguration(...)
class DatabaseIntrospectorTest {
    @Autowired lateinit var testDatabaseConnection: Connection

    ...
}
```

Since nobody will ever manually new up `DatabaseIntrospectorTest`, or reuse it
in a different context, the setter injection here is perfectly okay.
