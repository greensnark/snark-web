---
title: "Kotlin: Calling Extension Functions from Java"
date: 2021-02-19T16:40:07-05:00
---

Kotlin's [extension functions](https://kotlinlang.org/docs/extensions.html) are
a convenient way of attaching behavior to the most natural place it belongs.
Extensions are incredibly handy to extend the behavior of classes you don't
control, or to define behavior that may only be useful to a subset of users of
the class.

<!--more-->

Here's an example extension (I'd use this mainly in tests, not in production
code) to convert an IANA timezone name (like "America/Los_Angeles") into a Java
[`ZoneId`](https://docs.oracle.com/javase/8/docs/api/java/time/ZoneId.html)

```kt
/**
 * The IANA [ZoneId] for this string.
 *
 * For instance:
 *
 *     "America/Los_Angeles".zone == ZoneId.of("America/Los_Angeles")
 *
 */
val String.zone: ZoneId get() = ZoneId.of(this)
```

In Kotlin, using this extension is easy. The question is, if we're working in a
mixed Java+Kotlin codebase, how do we use a Kotlin extension in Java code?

These are the key concepts to using a Kotlin extension function/val from Java:

- An extension is effectively equivalent to a Java static method, with the
  extension receiver passed as the first parameter.

- Top-level Kotlin functions and vals can be treated as Java static methods if
  the file is annotated with `@file:JvmName`

- A Kotlin `val` named `x` is [usable from Java as
  `getX`](https://kotlinlang.org/docs/java-to-kotlin-interop.html#properties)

Thus if we annotate the file as:

```kt
@file:JvmName("ZoneExtensions")

package org.foobar

...

/**
 * The IANA [ZoneId] for this string.
 *
 * For instance:
 *
 *     "America/Los_Angeles".zone == ZoneId.of("America/Los_Angeles")
 *
 */
val String.zone: ZoneId get() = ZoneId.of(this)
```

We can then use the extension from Java as:

```java
package org.foobar;

import static org.foobar.ZoneExtensions.getZone;

...

        getZone("Pacific/Honolulu")
```

Granted, with this particular example extension, we'd rather use
`ZoneId.of(...)` from Java, but this is a useful option to have when your
Kotlin extension function is non-trivial.
