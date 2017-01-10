---
layout: post
title: "Effective Types: Parameterised Types, a Friendly Primer (Flow)"
date: "2017-01-01 18:57"
published: true
categories:
- Effective Types
- Type Safety
- Flow
- JavaScript
---

_"Parameterised Types"_ are a feature of Flow, TypeScript, and a growing collection of other languages. Like functions have parameters (parameterised functions), parameterised types are a way of punching holes in type definitions, and leaving it to the user to provide specific types. They're a tool for letting you create type definitions that are both generic and reusable, and also as a tool for us to work on functions while excluding details we don't care about.

This is a little hard to relate in a sentence or two, so let's get straight to it. Here's a few type declarations:

```ts
// @flow

type JustAString = string;
type StringOrNumber = string | number;
type Optional<X> = X | null;
```

We have:

* A type `JustAString` that represents a value that could be any string, but nothing else.
* A type `StringOrNumber` that represents a value that could be any number, or any string, but nothing else.
* A more general type, `Optional<X>`. It represents either something (the `X`; we haven't specified what yet), or `null`, but nothing else.

We call this `X` a "type parameter". The `X` isn't a special name: it could be `T`, or `Item`, or `Thing` for all we care. We'll dig into what a "type parameter" actually means a little bit later, but first let's switch gears and think about functions for a moment.

Say we have this:

```ts
function doubleIt(x) {
  return x + x;
}
```

This is a function, `doubleIt`, which has a "parameter", `x`. We **call** a function by providing values for its function parameters:

```ts
const result = doubleIt(5);

// ... is like taking this:
function doubleIt(x) {
  return x + x;
}

// ... is like subbing in the variables like this:
function doubleIt(5) {
  return 5 + 5;
}

// ... to get a value: 10;
```

In the same way, we can **specialise** a type, by providing types for its type parameters:

```ts
type OptionalNumber = Optional<number>;

// ... is like taking this:
type Optional<X> = X | null;

// ... and subbing in the types like this:
type Optional<number> = number | null;

// ... to get a type representing "a number, or null".
const someNumber: Optional<number> = 10;
const isJustNull: Optional<number> = null;

const gonnaBreak: Optional<number> = "💥"; // Kaboom; doesn't pass Flow's type-check.
```

The `X` isn't a special name: it's just what we called the type parameter, just like `x` is what we called the function parameter; it could be `T`, or `Item`, or `Thing` for all we care.

And like function parameters, `X` (for example) refers to **the one type**. A type that is `type Point<T> = { x: T, y: T }` refers to an object with two fields, `x` and `y`, each with a value of the same type (`T`, whatever it is later specialised to).

Multiple parameters are supported too, eg. `type Post<ID,Title> = { id: ID, title: Title }`. They could be the same type (eg. string) or completely different ones (eg. number and a string), but we don't know until they're specialised with each use, so we can't assume they overlap.

<hr />

Moving on. These type parameters work with a variety of things, eg. functions:

```ts
// A function that takes a T, and returns a value of that same type.
function justReturn<T>(x: T): T {
  return x;
}

const hello = justReturn("Hello");

// ... is like subbing in the type like this:
function justReturn<string>(x: string): string {
  return x;
}
```

Or:

```ts
function first<X>(items: Array<X>): Optional<X> {
  if (0 in items) {
    return items[0];
  } else {
    return null;
  }
}

const nine = first([9, 8, 7]);

// ... specialising to a type like:
function first<number>(items: Array<number>): Optional<number> {
  // ...
}

// Which in turn expands to:
function first<number>(items: Array<number>): (number | null) {
  // ...
}
```

Notice something about `first`? We never looked at the contents of the items, only at the array itself. The function, as it's written, **doesn't know** what the type of the items are; it's only when used can we specialise to `items` being an array of numbers, strings, objects, etc.

Take this last example:

```ts
function first<X>(items: Array<X>): Optional<X> {
  if (0 in items) {
    // Add 100 to each array item. But what is the type of item? We don't
    // know, and/ if we don't know, we can't tell if this is correct,
    // so... Kaboom. 💥
    return items[0] + 100;
  } else {
    return null;
  }
}

// ... This fails to type-check with:
//
// 15: function first<T>(items: Array<T>): Optional<T> {
//                    ^ T. This type cannot be added to
// 17:     return items[0] + 100; // Add 100 to each item
//                           ^ number
```

Our `first` function **can't**<sup>[1](#note-1)</sup> know what the items are. We already use Flow to restrict *data* to types like numbers and strings. By preventing `first` from mucking around with the items themselves, we've restricted *the function*, and having the type of the function (written out in terms of the type parameter `T`) reflect that restriction.

As is the goal with much of what we do with Flow, the more restricted we can make the types of our data and functions, the fewer possibilities we need to think about when we change them, letting us focus in what we're immediately working on with fewer mistakes. Parameterised types let us do this by having us only worry about the specifics we care about (our input is an array), and not about the things we don't (it's an array of numbers).

<hr />

I'm going to leave you with some examples of parameterised types that I've found useful. I'm [interested to hear](https://twitter.com/damncabbage) what other combos you come up with. 😊

```ts
// This is Flow's ? symbol, eg. Q<string> is ?string.
type Q<X> = X | null | undefined;


// This is a function that returns a Promise; we've used a type
// parameter to specify that the returned value is going to
// be a string.
function fetchExampleDotCom(): Promise<string> {
  return fetch('http://example.com')
           .then(response => response.text())
}


// Inspired by Rust; this is either a failure or a success.
// Coming up with a generic success/failure type like this means
// you can write helper functions for them, and reuse them for
// many cases of success or failure.
type Result<Error,Value> =
    { error: Error }
  | { value: Value }


// An AJAX request state, inspired by @krisjenkin's post here:
// http://blog.jenkster.com/2016/06/how-elm-slays-a-ui-antipattern.html
type RequestData<Error,Value> =
    { type: "notRequested" }
  | { type: "requesting" }
  | { type: "failure", error: Error }
  | { type: "success", value: Value }

// eg.
// type RemoteTodoItems = RequestData<string, Array<TodoItem>>;


// Something from the next post. :)
// This lets us create a named wrapper around string (for User IDs, in
// this case) that prevents us from mixing up with other strings.
class UserID extends TypeWrapper<string> {}
```

PS: One last note, on terminology. In case you run into this in future, the concept of having these parameterised types is sometimes called "Generics" (see Java, C#, the Go "Generics" debate, etc), and is *mostly* like something called "Parametricity".<sup>[2](#note-2)</sup>


<small id="note-1">1. Flow has an (unfortunate? intentionally designed?) backdoor that lets you figure out what the items are; I'll be covering this in the next post.</small>

<small id="note-2">2. Flow and TypeScript both let through code that breaks parametricity, which is why I'm reluctant to use the term; there's some back and forth on the topic on [this Gist](https://gist.github.com/raichoo/b5d2534c18eadbf9da8b).</small>
