---
layout: post
title: "Effective Types: Phantom Types with Flow"
date: "2017-02-02 21:46"
listed: true
published: true
categories:
- Effective Types
- Flow
- JavaScript
- Type Safety
- Phantom Types
---

> This is part of a short series of Flow/TypeScript posts I'm calling "Effective Types". Posts so far:
>
> * [Parameterised Types, a Friendly Primer](/blog/2017/01/effective-types-a-parameterised-type-primer-flow/) (Flow, TypeScript)
> * [Type Wrappers with Flow](/blog/2017/01/effective-types-type-wrappers-with-flow/) (Flow)
> * [Phantom Types](/blog/2017/02/effective-types-phantom-types-with-flow/) (Flow)
> * Type Design (Flow; coming soonish)

Phantom Types are a way to add extra information to types, eg. to differentiate them, in such a way so that the extra information goes away when type-checking is complete.

This post relies pretty heavily on understanding [type parameters](/blog/2017/01/effective-types-a-parameterised-type-primer-flow/) and ["type wrappers" / new-types](/blog/2017/01/effective-types-type-wrappers-with-flow/), and I recommend you understand both of those posts before digging in to this one.


## Ghost Stories

### Phantom Types

Let's start with a useless class with a type parameter:

```ts
class Useless<T> {
  // Absolutely nothing.
}
```

We have a type parameter that's completely spare; nothing is using it. But look what happens when we create a couple of values, and fill in that parameter with a type:

```ts
import { isEqual } form 'lodash'; // We'll use this later.

// Create two values.
const uString: Useless<string> = new Useless();
const uNumber: Useless<number> = new Useless();

console.log(isEqual(uString, uNumber)); // => true; effectively the same, value-wise.

// But...
function doNothing(x: Useless<string>) { return; }

doNothing(uString); // <-- Compiles!
doNothing(uNumber); // <-- Kaboom! 💥
                    // Expected a Useless<string>, but
                    // got a Useless<number> instead.
```

We have two values (instances of `Useless`) that are completely identical when we run the code ("at run-time"), but have different types. This is the essence of Phantom Types.

Here's one more example, this time with something that isn't a blank class:

```ts
class Temperature<T> {
  degrees: number;
  constructor(n: number) {
    this.degrees = n;
  }
}

// Just define a couple of types; they're not actually used for real values.
// It's like we're defining our own "string" or "number" types to sub in, like
// how we used them in the previous example.
class Fahrenheit {}
class Celsius {}

const boilingFahr: Temperature<Fahrenheit> = new Temperature(212);
const boilingCels: Temperature<Celsius> = new Temperature(100);

console.log(
  boilingFahr.degrees === boilingCels.degrees
); // => true; the same value.

// But...
function convertFtoC(t: Temperature<Fahrenheit>): Temperature<Celsius> {
  return (t.degrees - 32) / 1.8;
}

convertFtoC(boilingFahr); // <-- Compiles!
convertFtoC(boilingCels); // <-- Kaboom! 💥
                          // Expected a Temperature<Fahrenheit>, but
                          // got a Temperature<Celsius> instead.

```


### Phantom Types with Type Wrappers

If you remember the [the last post](/blog/2017/01/effective-types-type-wrappers-with-flow/), you'll quickly see that this `Temperature` class is basically the same as an `class Temperature<T> extends TypeWrapper<number>`. You could do the manual version, but the TypeWrapper version is just a quicker way to define the type, and gives you standardised methods for wrapping up and extracting the value.

Last example, using a TypeWrapper:

```ts
import { TypeWrapper } from 'flow-classy-type-wrapper';

class Temperature<T> extends TypeWrapper<number> {}
class Fahrenheit {}
class Celsius {}

function convertFtoC(f: Temperature<Fahrenheit>): Temperature<Celsius> {
  return Temperature.wrap(
    (Temperature.unwrap(n) - 32) / 1.8
  );
}

const boilingFahr: Temperature<Fahrenheit> = new Temperature(212);

convertFtoC(boilingFahr); // <-- Compiles!
```


## Examples

### Validation

(This example was partly drawn from [https://wiki.haskell.org/Phantom_type](https://wiki.haskell.org/Phantom_type))

Besides the aforementioned Temperature example (which is valid enough in itself), it's also useful for tagging a value as having gone through some process:

```ts
class FormEmail<T> extends TypeWrapper<string> {}
class Validated {}
class Unvalidated {}

function validateEmail(x: FormEmail<Unvalidated>): (FormEmail<Validated> | null) {
  const email = FormEmail.unwrap(x);
  if (email.match(/@/)) {
    return FormEmail.wrap(email);
  }
  return null;
}

function createFormEmail(x: string): FormEmail<Unvalidated> {
  return FormEmail.wrap(x);
}

// Later:
const email = createFormEmail("alice@example.com"); 
const goodEmail = validateEmail(email); // <-- Compiles!
if (goodEmail) {
  // goodEmail is known to be a FormEmail<Validated> now.
  console.log("Validated!");

  const gooderEmail = validateEmail(goodEmail);
  // Kaboom! 💥  ------^
  // We gave it a FormEmail<Validated>, but it was expecting
  // a FormEmail<Unvalidated>. No double-validation allowed.
}
```

(The unwrapping and rewrapping in `validateEmail` is necessary to convince Flow that we've intentionally made the jump from `Unvalidated` to `Validated`. We could bypass this with an `(... : any)`, but that's too easy to hold incorrectly; I'll take the safe-but-boring method.)

... Or even:

```ts
// Now FormData is completely generic:
class FormData<Value,Phantom> extends TypeWrapper<Value> {}
class Validated {}
class Unvalidated {}

function createFormData<T>(x: T): FormData<T,Unvalidated> {
  return FormData.wrap(x);
}

// With some value-specific validation functions:

function validateEmail(
  x: FormData<Email,Unvalidated>
): (FormData<Email,Validated> | null) {
  // ...
}

function validateDropdownSelection<T>(
  item: FormData<T,Unvalidated>,
  options: Array<T>
): (FormData<T,Validated> | null) {
  // ...
}
```

Combining this with the ["Gatekeeper" functions trick](/blog/2017/01/effective-types-type-wrappers-with-flow/#gatekeeper-functions), only exporting the `FormData` type, `createFormData`, `unwrap` and validation functions, means you have a practically airtight way of making sure new data passes through your validation layer before being used.


### State Transitions

This is the more generic version of the Validation example above; previously, we had a simplistic [State Machine](https://www.youtube.com/watch?v=hJIST1cEf6A) that started at `Unvalidated` and (via `validate...()`) transitioned to `Validated`.

Say we have a plane that is always represented by a value (an object with its `flightCode`, for example). The plane can go through a number of different states (being on the ground, taxiing, etc), but only some of the transitions are valid.

As a final whimsical example rounding off this post, we can use Phantom Types to encode this:

```ts
class Plane<State> extends TypeWrapper<{flightCode: string}> {}

class OnGround {}
class Taxiing {}
class InAir {}
class OnApproach {}

function taxi(p: Plane<OnGround>): Plane<Takiing> { return Plane.wrap(Plane.unwrap(p)); }
function takeOff(p: Plane<Taxiing>): Plane<InAir> { /* ... */ }
function descend(p: Plane<InAir>): Plane<OnApproach> { /* ... */ }
function land(p: Plane<OnApproach>): Plane<OnGround> { /* ... */ }
function crash<T>(p: Plane<T>): Plane<OnGround> { /* ... ouch. */ }
```


