---
layout: post
title: "The Testing ↔ Monitoring Continuum"
date: "2015-01-13 22:08"
published: false
categories:
- Safety
- Monitoring
- Testing
---

The dot-points version:

* Ops: Who uses Nagios? Serverspec?
* Dev: Who writes unit tests? Cool. Integration tests? Capybara, Selenium, Browser-based tests, curl + bash

* What is monitoring?
  * Well, say we have a bunch of machines running an app. It's got load balancers, app servers, databases, whatever.
  * Let's make sure they're still up
  * Well, is Postgres running on the database?
  * Is it accepting connections?
  * Well, is it the right username + password? Maybe we broke something.
  * Okay, but does it have the extensions installed that we need?
  * Is the database right?
  * Can we see the tables that the app needs?
  * Can we write to the tables that the app needs?
  * Do we need to do this for every service that the app relies on?
  * Where do we stop?

* We have a stopping point. We should arguably be running the app itself.
  * Okay, so how much?

* Back to our earlier test mentions.
  * We could write a smoke test that uses curl or something and tests, like, login
  * But why not go the whole hog?
  * Our app has a test suite. (And if it doesn't, it bloody well should. ;) )

* Previous company; multi-tenant, hosted Software-as-a-Service app that users bought instances of.
  * We'd have unit tests, etc. but then these big browser-driven tests that would smoke test large parts of the app.
  * Spin up a new instance, run tests on it, throw it away.
  * In our case it was a custom app, but could easily be Jenkins.
* The nature of that system means that the system-under-test was distant from the testing agent. That's inconvenient, only use that when needed.
* Instead, use a set of helpers to set up test data, and then have those helpers either
    * Use an API behind the scenes, or
    * Muck around with the database like you usually would.

* My thesis of this talk is:
  * There is no Continuum.
  * Good Monitoring includes Testing.

...

* Two buckets:
  * Preventative (may it fail soon)
  * Right Now (is it working)
* Environmental (eg. RAM and Disk) is preventative.


* ServerSpec
