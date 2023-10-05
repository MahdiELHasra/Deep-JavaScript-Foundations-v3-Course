# Coercion

We've thoroughly covered all of the different _types_ of values in JS. And along the way, more than a few times, we mentioned the notion of converting -- actually, coercing -- from one type of value to another.

## Coercion: Explicit vs Implicit

Some developers assert that when you explicitly indicate a type change in an operation, this doesn't qualify as a _coercion_ but just a type-cast or type-conversion. In other words, the claim is that coercion is only implicit.

I disagree with this characterization. I use _coercion_ to label any type conversion in a dynamically-typed language, whether it's plainly obvious in the code or not. Here's why: the line between _explicit_ and _implicit_ is not clear and objective, it's fairly subjective. If you think a type conversion is implicit (and thus _coercion_), but I think it's explicit (and thus not a _coercion_), the distinction becomes irrelevant.

Keep that subjectivity in mind as we explore various _explicit_ and _implicit_ forms of coercion. In fact, here's a spoiler: most of the coercions could be argued as either, so we'll be looking at them with such balanced perspective.

### Implicit: Bad or ...?

An extremely common opinion among JS developers is that _coercion is bad_, specifically, that _implicit coercion is bad_; the rise in popularity of type-aware tooling like TypeScript speaks loudly to this sentiment.

But that feeling is not new. 14+ years ago, Douglas Crockford's book "The Good Parts" also famously decried _implicit coercion_ as one of the _bad parts_. Even Brendan Eich, creator of JS, regularly claims that _implicit coercion_ was a mistake[^EichCoercion] in the early design of the language that he now regrets.

If you've been around JS for more than a few months, you've almost certainly heard these opinions voiced strongly and predominantly. And if you've been around JS for years or more, you probably have your mind already made up.

In fact, I think you'd be hard pressed to name hardly any other well-known source of JS teaching that strongly endorses coercion (in virtually all its forms); I do -- and this book definitely does! -- but I feel mostly like a lone voice shouting futilely in the wilderness.

However, here's an observation I've made over the years: most of the folks who publicly condemn _implicit coercion_, actually use _implicit coercion_ in their own code. Hmmmm...

Douglas Crockford says to avoid the mistake of _implicit coercion_[^CrockfordCoercion], but his code uses `if (..)` statements with non-boolean values evaluated. [^CrockfordIfs] Many have dismissed my pointing that out in the past, with the claim that conversion-to-boolean isn't _really_ coercion. Ummm... ok?

Brendan Eich says he regrets _implicit coercion_, but yet he openly endorses[^BrendanToString] idioms like `x + ""` (and others!) to coerce the value in `x` to a string (we'll cover this later); and that's most definitely an _implicit coercion_.

So what do we make of this dissonance? Is it merely a, "do as I say, not as I do" minor self-contradiction? Or is there more to it?

## Abstracts

Now that I've challenged you to examine coercion in more depth than you may have ever previously indulged, let's first look at the foundations of how coercion occurs, according to the JS specification.

The specification details a number of _abstract operations_[^AbstractOperations] that dictate internal conversion from one value-type to another. It's important to be aware of these operations, as coercive mechanics in the language mix and match them in various ways.

These operations _look_ as if they're real functions that could be called, such as `ToString(..)` or `ToNumber(..)`. But by _abstract_, we mean they only exist conceptually by these names; they aren't functions we can _directly_ invoke in our programs. Instead, we activate them implicitly/indirectly depending on the statements/expressions in our programs.

### ToPrimitive

Any value that's not already a primitive can be reduced to a primitive using the `ToPrimitive()` (specifically, `OrdinaryToPrimitive()`[^OrdinaryToPrimitive]) abstract operation. Generally, the `ToPrimitive()` is given a _hint_ to tell it whether a `number` or `string` is preferred.

```
// ToPrimitive() is abstract

ToPrimitive({ a: 1 },"string");          // "[object Object]"

ToPrimitive({ a: 1 },"number");          // NaN
```

The `ToPrimitive()` operation will look on the object provided, for either a `toString()` method or a `valueOf()` method; the order it looks for those is controlled by the _hint_. `"string"` means check in `toString()` / `valueOf()` order, whereas `"number"` (or no _hint_) means check in `valueOf()` / `toString()` order.

If the method returns a value matching the _hinted_ type, the operation is finished. But if the method doesn't return a value of the _hinted_ type, `ToPrimitive()` will then look for and invoke the other method (if found).

If the attempts at method invocation fail to produce a value of the _hinted_ type, the final return value is forcibly coerced via the corresponding abstract operation: `ToString()` or `ToNumber()`.

### ToString

Pretty much any value that's not already a string can be coerced to a string representation, via `ToString()`. [^ToString] This is usually quite intuitive, especially with primitive values:

```
// ToString() is abstract

ToString(42.0);                 // "42"
ToString(-3);                   // "-3"
ToString(Infinity);             // "Infinity"
ToString(NaN);                  // "NaN"
ToString(42n);                  // "42"

ToString(true);                 // "true"
ToString(false);                // "false"

ToString(null);                 // "null"
ToString(undefined);            // "undefined"
```

There are _some_ results that may vary from common intuition. Very large or very small numbers will be represented using scientific notation:

```
ToString(Number.MAX_VALUE);     // "1.7976931348623157e+308"
ToString(Math.EPSILON);         // "2.220446049250313e-16"
```

Another counter-intuitive result comes from `-0`:

```
ToString(-0);                   // "0" -- wtf?
```

This isn't a bug, it's just an intentional behavior from the earliest days of JS, based on the assumption that developers generally wouldn't want to ever see a negative-zero output.

One primitive value-type that is _not allowed_ to be coerced (implicitly, at least) to string is `symbol`:

```
ToString(Symbol("ok"));         // TypeError exception thrown
```

| WARNING:                                                                                                                                                                                                                                                                                                                   |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Calling the `String()`[^StringFunction] concrete function (without `new` operator) is generally thought of as _merely_ invoking the `ToString()` abstract operation. While that's mostly true, it's not entirely so. `String(Symbol("ok"))` works, whereas the abstract `ToString(Symbol(..))` itself throws an exception. |

#### Default `toString()`

When `ToString()` is activated with an object value-type, it delegates to the `ToPrimitive()` operation (as explained earlier), with `"string"` as its _hinted_ type:

```
ToString(new String("abc"));        // "abc"
ToString(new Number(42));           // "42"

ToString({ a: 1 });                 // "[object Object]"
ToString([ 1, 2, 3 ]);              // "1,2,3"
```

By virtue of `ToPrimitive(..,"string")` delegation, these objects all have their default `toString()` method (inherited via `[[Prototype]]`) invoked.

### ToNumber

Non-number values _that resemble_ numbers, such as numeric strings, can generally be coerced to a numeric representation, using `ToNumber()`: [^ToNumber]

```
// ToNumber() is abstract

ToNumber("42");                     // 42
ToNumber("-3");                     // -3
ToNumber("1.2300");                 // 1.23
ToNumber("   8.0    ");             // 8
```

If the full value doesn't _completely_ (other than whitespace) resemble a valid number, the result will be `NaN`:

```
ToNumber("123px");                  // NaN
ToNumber("hello");                  // NaN
```

Other primitive values have certain designated numeric equivalents:

```
ToNumber(true);                     // 1
ToNumber(false);                    // 0

ToNumber(null);                     // 0
ToNumber(undefined);                // NaN
```

There are some rather surprising designations for `ToNumber()`:

```
ToNumber("");                       // 0
ToNumber("       ");                // 0
```

| NOTE:                                                                                                                               |
| :---------------------------------------------------------------------------------------------------------------------------------- |
| I call these "surprising" because I think it would have made much more sense for them to coerce to `NaN`, the way `undefined` does. |

Some primitive values are _not allowed_ to be coerced to numbers, and result in exceptions rather than `NaN`:

```
ToNumber(42n);                      // TypeError exception thrown
ToNumber(Symbol("42"));             // TypeError exception thrown
```

| WARNING:                                                                                                                                                                                                                                                                                                                                 |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Calling the `Number()`[^NumberFunction] concrete function (without `new` operator) is generally thought of as _merely_ invoking the `ToNumber()` abstract operation to coerce a value to a number. While that's mostly true, it's not entirely so. `Number(42n)` works, whereas the abstract `ToNumber(42n)` itself throws an exception. |

### ToBoolean

Decision making (conditional branching) always requires a boolean `true` or `false` value. But it's extremely common to want to make these decisions based on non-boolean value conditions, such as whether a string is empty or has anything in it.

When non-boolean values are encountered in a context that requires a boolean -- such as the condition clause of an `if` statement or `for` loop -- the `ToBoolean(..)`[^ToBoolean] abstract operation is activated to facilitate the coercion.

All values in JS are in one of two buckets: _truthy_ or _falsy_. Truthy values coerce via the `ToBoolean()` operation to `true`, whereas falsy values coerce to `false`:

```
// ToBoolean() is abstract

ToBoolean(undefined);               // false
ToBoolean(null);                    // false
ToBoolean("");                      // false
ToBoolean(0);                       // false
ToBoolean(-0);                      // false
ToBoolean(0n);                      // false
ToBoolean(NaN);                     // false
```

Simple rule: _any other value_ that's not in the above list is truthy and coerces via `ToBoolean()` to `true`:

```
ToBoolean("hello");                 // true
ToBoolean(42);                      // true
ToBoolean([ 1, 2, 3 ]);             // true
ToBoolean({ a: 1 });                // true
```

Even values like `"   "` (string with only whitespace), `[]` (empty array), and `{}` (empty object), which may seem intuitively like they're more "false" than "true", nevertheless coerce to `true`.

| WARNING:                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| There _are_ narrow, tricky exceptions to this truthy rule. For example, the web platform has deprecated the long-standing `document.all` collection/array feature, though it cannot be removed entirely -- that would break too many sites. Even where `document.all` is still defined, it behaves as a "falsy object"[^ExoticFalsyObjects] -- `undefined` which then coerces to `false`; this means legacy conditional checks like `if (document.all) { .. }` no longer pass. |

The `ToBoolean()` coercion operation is basically a lookup table rather than an algorithm of steps to use in coercions a non-boolean to a boolean. Thus, some developers assert that this isn't _really_ coercion the way other abstract coercion operations are. I think that's bogus. `ToBoolean()` converts from non-boolean value-types to a boolean, and that's clear cut type coercion (even if it's a very simple lookup instead of an algorithm).

Keep in mind: these rules of boolean coercion only apply when `ToBoolean()` is actually activated. There are constructs/idioms in the JS language that may appear to involve boolean coercion but which don't actually do so. More on these later.

## Coercion Corner Cases

I've been clear in expressing my pro-coercion opinion thus far. And it _is_ just an opinion, though it's based on interpreting facts gleaned from studying the language specification and observable JS behaviors.

That's not to say that coercion is perfect. There's several frustrating corner cases we need to be aware of, so we avoid tripping into those potholes. In case it's not clear, my following characterizations of these corner cases are just more of my opinions. Your mileage may vary.

### Strings

We already saw that the string coercion of an array looks like this:

```js
String([1, 2, 3]); // "1,2,3"
```

I personally find that super annoying, that it doesn't include the surrounding `[ ]`. In particular, that leads to this absurdity:

```js
String([]); // ""
```

So we can't tell that it's even an array, because all we get is an empty string? Great, JS. That's just stupid. Sorry, but it is. And it gets worse:

```js
String([null, undefined]); // ","
```

WAT!? We know that `null` coerces to the string `"null"`, and `undefined` coerces to the string `"undefined"`. But if those values are in an array, they magically just _disappear_ as empty strings in the array-to-string coercion. Only the `","` remains to even hint to us there was anything at all in the array! That's just silly town, right there.

What about objects? Almost as aggravating, though in the opposite direction:

```js
String({}); // "[object Object]"

String({ a: 1 }); // "[object Object]"
```

Umm... OK. Sure, thanks JS for no help at all in understanding what the object value is.

### Numbers

I'm about to reveal what I think is _the_ worst root of all coercion corner case evil. Are you ready for it?!?

```js
Number(""); // 0
Number("       "); // 0
```

I'm still shaking my head at this one, and I've known about it for nearly 20 years. I still don't get what Brendan was thinking with this one.

The empty string is devoid of any contents; it has nothing in it with which to determine a numeric representation. `0` is absolutely **_NOT_** the numeric equivalent of missing/invalid numeric value. You know what number value we have that is well-suited to communicate that? `NaN`. Don't even get me started on how whitespace is stripped from strings when coercing to a number, so the very-much-not-empty `"       "` string is still treated the same as `""` for numeric coercion purposes.

Even worse, recall how `[]` coerces to the string `""`? By extension:

```js
Number([]); // 0
```

Doh! If `""` didn't coerce to `0` -- remember, this is the root of all coercion evil! --, then `[]` wouldn't coerce to `0` either.

This is just absurd, upside-down universe territory.

Much more tame, but still mildly annoying:

```js
Number("NaN"); // NaN  <--- accidental!

Number("Infinity"); // Infinity
Number("infinity"); // NaN  <--- oops, watch case!
```

The string `"NaN"` is not parsed as a recognizable numeric value, so the coercion fails, producing (accidentally!) the `NaN` value. `"Infinity"` is explicitly parseable for the coercion, but any other casing, including `"infinity"`, will fail, again producing `NaN`.

This next example, you may not think is a corner case at all:

```js
Number(false); // 0
Number(true); // 1
```

It's merely programmer convention, legacy from languages that didn't originally have boolean `true` and `false` values, that we treat `0` as `false`, and `1` as `true`. But does it _really_ make sense to go the other direction?

Think about it this way:

```js
false + true + false + false + true; // 2
```

Really? I don't think there's any case where treating a `boolean` as its `number` equivalent makes any rational sense in a program. I can understand the reverse, for historical reasons: `Boolean(0)` and `Boolean(1)`.

But I genuniely feel that `Number(false)` and `Number(true)` (as well as any implicit coercion forms) should produce `NaN`, not `0` / `1`.

### Coercion Absurdity

To prove my point, let's take the absurdity up to level 11:

```js
[] == ![]; // true
```

How!? That seems beyond credibility that a value could be coercively equal to its negation, right!?

But follow down the coercion rabbit hole:

1. `[] == ![]`
2. `[] == false`
3. `"" == false`
4. `0 == false`
5. `0 == 0`
6. `0 === 0` -> `true`

We've got three different absurdities conspiring against us: `String([])`, `Number("")`, and `Number(false)`; if any of these weren't true, this nonsense corner case outcome wouldn't occur.

Let me make something absolutely clear, though: none of this is `==`'s fault. It gets the blame here, of course. But the real culprits are the underlying `string` and `number` corner cases.

#### Other Abstract Numeric Conversions

In addition to `ToNumber()`, the specification defines `ToNumeric()`, which activates `ToPrimitive()` on a value, then conditionally delegates to `ToNumber()` if the value is _not_ already a `bigint` value-type.

There are also a wide variety of abstract operations related to converting values to very specific subsets of the general `number` type:

- `ToIntegerOrInfinity()`
- `ToInt32()`
- `ToUint32()`
- `ToInt16()`
- `ToUint16()`
- `ToInt8()`
- `ToUint8()`
- `ToUint8Clamp()`

Other operations related to `bigint`:

- `ToBigInt()`
- `StringToBigInt()`
- `ToBigInt64()`
- `ToBigUint64()`

You can probably infer the purpose of these operations from their names, and/or from consulting their algorithms in the specification. For most JS operations, it's more likely that a higher-level operation like `ToNumber()` is activated, rather than these specific ones.
