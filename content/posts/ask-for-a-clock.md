---
title: "Ask for a Clock"
date: 2020-12-11T16:43:36-05:00
---

When writing code that needs the current time, don't hardcode a reference to
the current time. Don't do this:

```kt
class Appointments {
    ...

    fun makeAppointment(forPerson: Person): Appointment =
        // Hard-coded current time reference:
        dbAppointments.create(forPerson, at = Instant.now())
}
```

This applies to database queries as well, avoid:

```sql
SELECT
    *
FROM
    appointment
WHERE
    -- Hard-coded current time reference:
    created_at > current_date - interval '10 days'
```

Why is directly using the current time a problem? It makes testing this code
much harder. Tests are usually reduced to gymnastics involving the current time
in assertions, and often flake when those time gymnastics meet a slow CI build
server.

There are two good alternatives to hard-coded references to the current time:

1. Ask for the current time as a parameter OR
2. Depend on a time-source that testers can control

These options have trade-offs. Asking the caller to give you the time is simple
and convenient:

```kt
class Appointments {
    ...

    fun makeAppointment(forPerson: Person, at: Instant): Appointment =
        dbAppointments.create(forPerson, at)
}

...

class AppointmentsTest {
    fun testCreateAppointment() {
        val currentTime =
            ZonedDateTime
                .parse("2019-10-15T17:00:00-04:00[America/New_York]")
                .toInstant()
        assertEquals(
            "The appointment createdAt is the currentTime",
            currentTime,
            appointments
                .makeAppointment(
                    forPerson = liseMeitner, 
                    at = currentTime)
                .createdAt)
    }
}
```

This gets trickier the farther down the call stack the use of the wall-clock
time goes, because then you end up threading the current-time up and down the
call stack, damaging the signature of all the intervening methods.

To avoid this, make code that uses the current time _depend_ on a time source:

```kt
class Appointments(private val clock: Clock)  {
    fun makeAppointment(forPerson: Person): Appointment =
        appointments.create(forPerson = forPerson, at = clock.instant())
}

...


class AppointmentsTest {
    fun testCreateAppointment() {
        val currentTime =
            ZonedDateTime
                .parse("2019-10-15T17:00:00-04:00[America/New_York]")
                .toInstant()
        assertEquals(
            "The appointment createdAt is the currentTime",
            currentTime,
            Appointments(clock = FixedClock(currentTime))
                .makeAppointment(liseMeitner)
                .createdAt)
    }

    class FixedClock(
        val at: Instant,
        private val zone: ZoneId = ZoneId.systemDefault()
    ) : java.time.Clock() {
        override fun getZone(): ZoneId = zone
        override fun withZone(zone: ZoneId): Clock =
            FixedClock(at, zone)
        override fun instant(): Instant = at
    }
}
```

If you're writing an integration test in a larger application
dependency-injection framework like Spring, this approach still works: in
production configs you'd inject a `Clock` that returns the current time, while
in test contexts, you'd inject a clock that tests can control as needed.

## When to use time-as-parameter vs clock-as-dependency

So how to decide whether to use time-as-parameter or inject a clock as a
dependency? The key is whether the API *feels natural to the caller with the
time as a parameter*. (This is obviously somewhat subjective.)

The rule of thumb I use is to start with the time as a parameter, but switch to
clock-as-dependency if I have to thread a time through too many layers of code,
especially in cases where the intervening code really doesn't care about the
time it's passing down.
