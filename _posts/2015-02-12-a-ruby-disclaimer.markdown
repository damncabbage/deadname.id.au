---
layout: post
title: "A Ruby Disclaimer"
date: "2015-02-12 10:43"
published: true
categories:
- Ruby
- Safety
- Types
- Possibly Being Overly Defensive
---

I have a disclaimer to make.

When I write about Ruby (or PHP, or JavaScript), I'm writing with the implicit assumption that:

* I know these languages are untyped / unityped; every value can be any other kind of value, or `nil` / `undefined` / `null`, and
* Abstractions like Functors or Monads are *possible* in these languages, but these languages also make them awkward to build and intractable to use on a regular basis without making mistakes.

Any advice I give in these posts about making Ruby projects more maintainable, for example, are with the above in mind. There will always be `nil`. There will not likely be a compiler for the language in its current form. It's difficult to check things in advance without just running the code over and over with different inputs and hoping you caught everything.

So with that anything I can say or write will be incremental improvements at best. I accept that. I can't just stand on the stage at [#rorosyd](http://www.meetup.com/Ruby-On-Rails-Oceania-Sydney/) and yell that we can't fix all those problems that we keep trying to fix with service-class / hexagonal-design / SRP / thin-fat-dieting-model-controller-view / self-flagellating workarounds.

And though I *do* find tools with stronger type systems very useful and very interesting, there are a lot companies using tools *without* them doing things that I find compelling, and I enjoy making improvements there. By inches, if necessary.
