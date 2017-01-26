---
layout: post
title: 'Revisiting "Ruby is Doomed"'
date: "2016-02-28 16:41"
listed: false
published: false
categories:
- Ruby
---

A couple of years ago I gave a talk at Railscamp Tasmania 2014 titled "Ruby is Doomed". Out of
embarrassment, a couple of days after giving it a renamed it to ["Ruby's Outer
Limits"](/talks/2014/11/rubys-outer-limits) and included
an apology in the second slide for any readers coming across it subsequently.

In short, the talk was about the problems we run into when we use Ruby from a technical angle. It
covered type-checking and immutability, stepping through some basic examples and constrasting them
with approaches some other languages take.

I still feel silly about the talk title. I don't feel silly about the content anymore.

These last couple of years, other languages with the same problems have started to find ways to make
it better:

* JavaScript, in the form of Flow and Typescript, have added gradual type-checking. JavaScript's
  libraries, in the form of React, Mithril, Cycle, etc. have tackled UI problems in a more
  functional way. JavaScript has added `const` to lock down some variables (versus our "constants"
  that aren't), and libraries like Immutable.js and Mori to provide an immutable interface for them.

* Python has PEP 484 and MyPy to add Flow-like type annotations and checking to its own language.

* Elixir is somewhat Ruby-like in style, is functional and with immutability by default, but with
  the tooling niceties you've come to expect from Ruby.


There's been some encouraging moves. Ruby has added auto-freezing to strings, and in time, Ruby's
standard library will be fixed up to make use of it. Transproc has an introduced a general-purpose
functional-transform pipeline.

But I fear there's stagnation. I've only first-hand experiences to go on, but from what I'm
seeing there's an evaporation happening. Devs who have used Ruby are seriously looking into different
languages outside-of-hours, or bringing them to work. Devs wanting immutability and a more
functional interface are using Elixir. Devs wanting a type-checker are moving to Go. Devs who
previously would've dabbled in JavaScript for a portion of their Rails apps are now writing their entire apps in it.

And while there's a cycle of devs moving "through" languages, I don't think the level I've seen with
Ruby is normal. New devs are coming in, via General Assembly, and the valuable efforts with
Rails Girls and the reInteractive Dev Hubs and Installfests. But even General Assembly is now moving
to a more JavaScript-heavy curriculum.

So what do we do? I'm not sure. Here's some ideas:

* Follow Python's lead and introduce type-hinting. In the same way that having it as a PEP means
  multiple things can target it, I'd argue this would need to be a core-Ruby-issue-tracker thing.

* Look into a `Object#deep_freeze`, `Object#deep_dup`, and/or ownership transfer, as
  [Tony Arcieri suggests in this post](https://tonyarcieri.com/2012-the-year-rubyists-learned-to-stop-worrying-and-love-the-threads).

* The DRY libraries
