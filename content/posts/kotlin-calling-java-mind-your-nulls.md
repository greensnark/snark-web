---
title: "Kotlin Calling Java: Mind Your nulls"
date: 2020-12-09T15:05:21-05:00
author: "Darshan Shaligram"
---

If you're introducing Kotlin to an existing Java codebase, you'll probably have
places in your code where new Kotlin code has to call existing Java code, or
vice-versa. Kotlin's default behavior for Java interop poses a subtle, but
significant safety problem with nullability.

If you have Kotlin code calling Java code with this signature:

```java
    public void addPerson(Person newPerson);
```

Kotlin doesn't know if `newPerson` means `Person?` or `Person` in Kotlin, so
[type-inference treats it as `Person!`, a platform type of unknown
nullability](https://kotlinlang.org/docs/reference/java-interop.html#null-safety-and-platform-types).

What does that mean? It means Kotlin lets you treat `newPerson` as _either_
nullable _or_ non-null, assuming you know what you're doing, and making no
effort to safety-check your assumptions. This is obviously bad, because if the
Java code expects `newPerson` to always be non-null, this will blow up at
runtime, with no warnings from Kotlin.

There's worse. What happens when Java code calls Kotlin? Given a Kotlin class:

```kt
data class Person(val name: String)
```

Java code can then try to new up a `Person` as:

```java
    final var person = new Person(name);
    // What happens when name is null?
```

Which will provoke a `NullPointerException` from the Kotlin code. Not exactly
type-safe, and quite entertaining when it happens in production.

## The Workaround

There's no great fix for this problem. The best workaround I've found is to
carefully [add JSR-305 nullability annotations to Java code that faces
Kotlin](https://kotlinlang.org/docs/reference/java-interop.html#jsr-305-support) as
the Kotlin-Java interop page suggests, and to be vigilant when reviewing code
with Kotlin-Java boundaries.

```java
import javax.annotation.Nonnull;

...

    public void addPerson(@Nonnull Person newPerson);
```
