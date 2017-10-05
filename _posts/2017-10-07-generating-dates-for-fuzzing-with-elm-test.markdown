---
layout: post
title: "Fuzzing Dates with Elm-Test"
date: "2017-10-05 22:29"
published: true
listed: true
categories:
- Elm
- Property-Based Testing
- Time
---

Elm, with its [elm-test](https://github.com/elm-community/elm-test) library, allows for a [property-based style of testing](/talks/2016/06/property-based-testing) with its [Fuzz](package.elm-lang.org/packages/elm-community/elm-test/latest/Fuzz) test runner and suite of helpers.

Generating dates is something that I've found to be a bit of a pain. For the examples below I'll be using the [elm-community/elm-time](https://github.com/elm-community/elm-time) library.


## Broken

Generating the individual parts (year, month, day) with a bunch of `Int`s, eg.

```haskell
import Time.Date exposing (Date, date)
import Fuzz exposing (Fuzzer, int)

sillyDate : Fuzzer Date
sillyDate =
  Fuzz.map3 (\y m d -> date y m d)
    int -- year
    int -- month
    int -- day

  -- For reference:
  --   date : Int -> Int -> Int -> Date
```

... means you can end up with nonsensical dates, like `2000/99/06` or `-5/-9/44`.

And coming up with "sensible" numbers, eg.

```haskell
conservativeDate : Fuzzer Date
conservativeDate =
  Fuzz.map3 (\y m d -> date y m d)
    (intRange 1900 2100) -- year
    (intRange 1 12) -- month
    (intRange 1 28) -- just in case month is February
```

... means you're not going to cover the leap-year date of February 29, or even the 30/31-day months the rest of the year.

You could generate some potentially-failing cases and then [`Fuzz.conditional`](http://package.elm-lang.org/packages/elm-community/elm-test/latest/Fuzz#conditional)-filter the bad ones out, but we're already heading down a bad road. Let's see what else we can do instead.


## Better

Instead we're going to start with a year we care about (we'll pick one later), taking it in as input to a `dateForYear : Int -> Fuzzer Date` fuzzer we can stash in a helper module for later reuse.

`Time.Date` has an [`addDays: Int -> Date -> Date`](package.elm-lang.org/packages/elm-community/elm-time/latest/Time-Date#addDays) function that, when given a date, will let you add (or subtract) an arbitrary number of days to it; with the library handling the transition as you progress through months and years, the date will stay valid the entire time. If we were to pick the start of the year, eg. `date 2017 1 1` for Jan 1st 2017, we could then `addDays` to that.

So using this function, we only need two things now: a generated Int for the year, and a generated Int for the number of days we want. They'll always work together even if generated independently. Solved!

Let's have a look at how we might define some helpers:

```haskell
module Test.Helpers.Dates exposing (..)

import Time.Date exposing (Date, date, addDays, isLeapYear)
import Fuzz exposing (Fuzzer, int, intRange)

dateForYear : Int -> Fuzzer Date
dateForYear year =
  let
    daysUpper = if isLeapYear year then 365 else 364
  in
    intRange 0 daysUpper
      |> Fuzz.map (\days -> addDays days (date year 1 1))

dateWithinYearRange : Int -> Int -> Fuzzer Date
dateWithinYearRange lower upper =
  intRange lower upper
    |> Fuzz.andThen (\year -> dateForYear year)
```

... And how we might use them in some tests. They're silly examples, but show passing and failing cases:

```haskell
import Test.Helpers.Dates exposing (dateWithinYearRange)
import Date.Extra.Facts

import Expect
import Fuzz exposing (Fuzzer)
import Test exposing (..)

suite : Test
suite =
  describe "Random Dates"
    [ fuzz (dateNear 2017)
        "Always passes"
        <| \date ->
          date |> Expect.equal date

    , fuzz2
        (dateForYear 2017)
        (dateForYear 2017)
        "Fails"
        <| \date1 date2 ->
          date1 |> Expect.equal date2

    , fuzz2 (dateNear 2017) (dateNear 2017)
        "Fails a lot more"
        <| \date1 date2 ->
          date1 |> Expect.equal date2
    ]

-- A quick convenience:
dateNear : Int -> Fuzzer Date
dateNear y =
  dateWithinYearRange (y - 2) (y + 2)
```

`Always passes` does what it says on the tin, and `Fails` and `Fails a lot more` fall over while preserving some sensible-looking failure output for debugging:

```
✗ Fails

Given (Date { year = 2017, month = 1, day = 2 },Date { year = 2017, month = 1, day = 1 })

    Date { year = 2017, month = 1, day = 2 }
    ╷
    │ Expect.equal
    ╵
    Date { year = 2017, month = 1, day = 1 }


Given (Date { year = 2017, month = 1, day = 1 },Date { year = 2017, month = 1, day = 2 })

    Date { year = 2017, month = 1, day = 1 }
    ╷
    │ Expect.equal
    ╵
    Date { year = 2017, month = 1, day = 2 }


↓ Random Dates
✗ Fails a lot more

Given (Date { year = 2019, month = 1, day = 1 },Date { year = 2019, month = 1, day = 2 })

    Date { year = 2019, month = 1, day = 1 }
    ╷
    │ Expect.equal
    ╵
    Date { year = 2019, month = 1, day = 2 }

...
```

# Extras

If you want to make sure you hit the leap years, you can either test them separately, or use the [`Fuzz.frequency`](http://package.elm-lang.org/packages/elm-community/elm-test/latest/Fuzz#frequency) function to make sure they show up sometimes whenever you reach for the date generator:

```haskell
module Test.Helpers.Dates exposing (..)

import Time.Date exposing (Date, date, isLeapYear)
import Fuzz exposing (Fuzzer, constant, frequency)
import List

dateNear : Int -> Fuzzer Date
dateNear year =
  let
    knownLeap = 2016
    nearestLeap =
      List.range -2 2
        |> List.map (\n -> year + n)
        |> List.foldl (\y rest -> if isLeapYear y then y else rest) knownLeap
  in
    Fuzz.frequency
      [ (1, constant (date nearestLeap 2 29))          -- chosen 25% of the time
      , (3, dateWithinYearRange (year - 2) (year + 2)) -- chosen 75% of the time
      ]
```



## Final Words of Warning

1) The `Date` type that the `elm-community/elm-time` library exports is a _different_ one to the one that the core `Date` module exposes, but the latter [appears to be going away](https://github.com/elm-lang/core/commit/a892fdf705f83523752c5469384e9880fbdfe3b1#diff-25d902c24283ab8cfbac54dfa101ad31). You may need to convert between the two, so just in case you do:  

```haskell
import Time.Date as Time
import Date as Core

coreDateToTimeDate : Core.Date -> Time.Date
coreDateToTimeDate d =
  Time.date (Core.year d) (Core.month d) (Core.day d)
```

2) It looks like [`Fuzz.andThen` is going away](https://github.com/elm-community/elm-test/pull/183); that means `dateWithYearRange` isn't going to work. I'm not sure what they intend to replace it with.
