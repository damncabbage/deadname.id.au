---
layout: post
title: "Flow and Refined Type Parameters"
date: "2017-01-10 03:05"
published: true
categories:
- Type Safety
- Flow
- JavaScript
---

This is a follow-up to my earlier [Parameterised Types](/blog/2017/01/effective-types-a-parameterised-type-primer-flow/) post, specifically focusing on this code example:

```ts
type Optional<X> = X | null;

function first<X>(items: Array<X>): Optional<X> {
  if (0 in items) {
    return items[0];
  } else {
    return null;
  }
}
```

And this footnote:

> 1. Flow has an (unfortunate? intentionally designed?) backdoor that lets you figure out what the items are; I'll be covering this in the next post.

In short: Flow's parameterised types unfortunately don't give us [Parametricity](https://www.schoolofhaskell.com/school/starting-with-haskell/introduction-to-haskell/5-type-classes). Both because of thing like being able to [compare anything](https://gist.github.com/raichoo/b5d2534c18eadbf9da8b) (a property of JavaScript, something that's arguably not Flow's fault), but because of Flow's own "refinements" concept.


Let's dig into an example.

```ts
function justReturn<X>(thing: X): X {
  return thing;
}

const x = justReturn("hi"); // x is "hi"
const y = justReturn(1234); // y is 1234
```

The type signature of this function, `justReturn<X>(thing: X): X`, means to some people: "justReturn is a function, that when called with an argument of type `X` (`thing`), will return an value of type `X`". And without the ability to know what type `X` is, we can't do anything with it (call any methods, use it in any functions) or make version of `X`.

Except unfortunately we can:

```ts
function justReturn<X>(thing: X): X {
  if (typeof thing === "number") {
    return thing * 2;
  }
  return thing;
}

const x = justReturn("hi"); // x is "hi"
const y = justReturn(1234); // y is 2468... Uh oh.
```

The type signature of this function basically becomes `justReturn<X>(thing: (X | number)): (X | number)`. Using `flow type-at-pos` on `thing` inside the `if (typeof thing === "number")` branch, we see it's type is `number`. Outside that, in the other branch, it's still `X`.

Back to the very first example:

```ts
type Optional<X> = X | null;

function first<X>(items: Array<X>): Optional<X> {
  if (0 in items) {
    return items[0];
  } else {
    return null;
  }
}
```

We can't just replace `return items[0];` with `return items[0] + 100;` – we'd have to _refine_ the value first. Then it'd quite happily work, thanks to refinements:

```ts
function first<X>(items: Array<X>): Optional<X> {
  if (0 in items) {
    if (typeof items[0] === "number") {
      return items[0] + 100; // refined to "number"; can add 100.
    }
    return items[0];
  } else {
    return null;
  }
}
```

So suddenly `first<X>(items: Array<X>): Optional<X>

This `typeof` check is a way of doing something called "Reflection", and basically means you can't trust your type signatures as much anymore.

What can we do about it? I'm not sure.

* Linting it is hard. Say we write a [eslint-flow rule](https://www.npmjs.com/package/eslint-plugin-flowtype#eslint-plugin-flowtype-rules-define-flow-type) for some particular cases of it: we could easily make the `justReturn` example throw up warnings. But it gets hairy with the second example; how does it know that `items[0]` means getting the `X` from `Array<T>`? Or any other function that interrogates and changes this or other parameterised types.

* Alter the compiler to throw errors when refining these bare ([unbounded](https://flowtype.org/blog/2015/03/12/Bounded-Polymorphism.html)) type parameters. I don't know how commonly in use this is, though.

* Leave it as it is, and trust that people will not really make use of this, or think of type signatures as advisory at most. Bummer. 😟

In any case, I'm [interested to hear](https://twitter.com/damncabbage) what people think on this topic.

Until next time. 👋
