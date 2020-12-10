---
title: "Timezones and Calendars on the Command Line"
date: 2020-12-10T17:12:07-05:00
---

When programming, you may have to work with dates and times in different
time-zones, or look at events in calendars in the past or future. You can do all
of these using GUI tools, but your Unix command-line is there for you (and
easier to script):

```sh
$ date
Thu Dec 10 05:14:10 PM EST 2020
```

We often need to look at UTC times. `date` can give us the UTC time:

```sh
$ date -u
Thu Dec 10 10:14:10 PM UTC 2020
```

Perhaps we want the time in a different time-zone, like Hawaii?

```sh
$ TZ=Pacific/Honolulu date
Thu Dec 10 12:14:10 PM HST 2020
```

Or India?

```sh
$ TZ=Asia/Kolkata date
Fri Dec 11 03:44:10 AM IST 2020
```

Maybe we're working with dates for display in some other locale, and we'd like
to see the date, in the language of that locale:

```sh
$ LC_ALL=ru_RU.UTF-8 TZ=Europe/Moscow date
Пт 11 дек 2020 01:14:10 MSK
```

(That _may_ not use the Russian date format for you if you don't have a Russian
locale installed.)

If you're working with calendars, `cal` shows the calendar for the current
month by default:

```sh
$ cal
    December 2020
Su Mo Tu We Th Fr Sa
       1  2  3  4  5
 6  7  8  9 10 11 12
13 14 15 16 17 18 19
20 21 22 23 24 25 26
27 28 29 30 31
```

If you'd like to see the whole year's calendar, give `cal` the year:

```sh
$ cal 1980
                               1980

       January               February                 March
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
       1  2  3  4  5                   1  2                      1
 6  7  8  9 10 11 12    3  4  5  6  7  8  9    2  3  4  5  6  7  8
13 14 15 16 17 18 19   10 11 12 13 14 15 16    9 10 11 12 13 14 15
20 21 22 23 24 25 26   17 18 19 20 21 22 23   16 17 18 19 20 21 22
27 28 29 30 31         24 25 26 27 28 29      23 24 25 26 27 28 29
                                              30 31
        April                   May                   June
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
       1  2  3  4  5                1  2  3    1  2  3  4  5  6  7
 6  7  8  9 10 11 12    4  5  6  7  8  9 10    8  9 10 11 12 13 14
13 14 15 16 17 18 19   11 12 13 14 15 16 17   15 16 17 18 19 20 21
20 21 22 23 24 25 26   18 19 20 21 22 23 24   22 23 24 25 26 27 28
27 28 29 30            25 26 27 28 29 30 31   29 30

        July                  August                September
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
       1  2  3  4  5                   1  2       1  2  3  4  5  6
 6  7  8  9 10 11 12    3  4  5  6  7  8  9    7  8  9 10 11 12 13
13 14 15 16 17 18 19   10 11 12 13 14 15 16   14 15 16 17 18 19 20
20 21 22 23 24 25 26   17 18 19 20 21 22 23   21 22 23 24 25 26 27
27 28 29 30 31         24 25 26 27 28 29 30   28 29 30
                       31
       October               November               December
Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa   Su Mo Tu We Th Fr Sa
          1  2  3  4                      1       1  2  3  4  5  6
 5  6  7  8  9 10 11    2  3  4  5  6  7  8    7  8  9 10 11 12 13
12 13 14 15 16 17 18    9 10 11 12 13 14 15   14 15 16 17 18 19 20
19 20 21 22 23 24 25   16 17 18 19 20 21 22   21 22 23 24 25 26 27
26 27 28 29 30 31      23 24 25 26 27 28 29   28 29 30 31
                       30
```

Maybe you'd like the calendar, localized in Chinese?

```sh
$  LC_ALL=zh_CN.UTF-8 cal 1980
                               1980

        一月                   二月                   三月
一 二 三 四 五 六 日   一 二 三 四 五 六 日   一 二 三 四 五 六 日
    1  2  3  4  5  6                1  2  3                   1  2
 7  8  9 10 11 12 13    4  5  6  7  8  9 10    3  4  5  6  7  8  9
14 15 16 17 18 19 20   11 12 13 14 15 16 17   10 11 12 13 14 15 16
21 22 23 24 25 26 27   18 19 20 21 22 23 24   17 18 19 20 21 22 23
28 29 30 31            25 26 27 28 29         24 25 26 27 28 29 30
                                              31
        四月                   五月                   六月
一 二 三 四 五 六 日   一 二 三 四 五 六 日   一 二 三 四 五 六 日
    1  2  3  4  5  6             1  2  3  4                      1
 7  8  9 10 11 12 13    5  6  7  8  9 10 11    2  3  4  5  6  7  8
14 15 16 17 18 19 20   12 13 14 15 16 17 18    9 10 11 12 13 14 15
21 22 23 24 25 26 27   19 20 21 22 23 24 25   16 17 18 19 20 21 22
28 29 30               26 27 28 29 30 31      23 24 25 26 27 28 29
                                              30
        七月                   八月                   九月
一 二 三 四 五 六 日   一 二 三 四 五 六 日   一 二 三 四 五 六 日
    1  2  3  4  5  6                1  2  3    1  2  3  4  5  6  7
 7  8  9 10 11 12 13    4  5  6  7  8  9 10    8  9 10 11 12 13 14
14 15 16 17 18 19 20   11 12 13 14 15 16 17   15 16 17 18 19 20 21
21 22 23 24 25 26 27   18 19 20 21 22 23 24   22 23 24 25 26 27 28
28 29 30 31            25 26 27 28 29 30 31   29 30

        十月                  十一月                 十二月
一 二 三 四 五 六 日   一 二 三 四 五 六 日   一 二 三 四 五 六 日
       1  2  3  4  5                   1  2    1  2  3  4  5  6  7
 6  7  8  9 10 11 12    3  4  5  6  7  8  9    8  9 10 11 12 13 14
13 14 15 16 17 18 19   10 11 12 13 14 15 16   15 16 17 18 19 20 21
20 21 22 23 24 25 26   17 18 19 20 21 22 23   22 23 24 25 26 27 28
27 28 29 30 31         24 25 26 27 28 29 30   29 30 31
```

(Again, this works correctly only if you have the locale for simplified Chinese
available on your computer.)
