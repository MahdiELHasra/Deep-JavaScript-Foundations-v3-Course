# 1 : Primitive Values

we confronted the common misconception that **_"everything in JS is an object"_**. We now circle back to that topic, and again dispel that myth.

Here, we'll look at the core value types of JS, specifically the non-object types called _primitives_.

## Value Types

JS doesn't apply types to variables or properties -- what I call, "container types" -- but rather, values themselves have types -- what I call, "value types".

The language provides seven built-in, primitive (non-object) value types: [^PrimitiveValues]

- `undefined`
- `null`
- `boolean`
- `number`
- `bigint`
- `symbol`
- `string`

These value-types define collections of one or more concrete values, each with a set of shared behaviors for all values of each type.

### Type-Of

Any value's value-type can be inspected via the `typeof` operator, which always returns a `string` value representing the underlying JS value-type:

```1,2
var v;
typeof v;                   // "undefined"
v = "1";
typeof v;                   // "string"
v = 2;
typeof v;                   // "number"
v = true;
typeof v;                   // "boolean"
v = {};
typeof v;                   // "object"
v = symbol();
typeof v;                   // "symbol"
typeof doesntExist;         // "undefined"
v = null;
typeof v;                   // "object"    OOPS!
v = function();
typeof v;                   // "function"  hmmm?
v = [1,2,3];
typeof v;                   // "object"    hmmm?
typeof 42n;                 // "bigint"
```

The `typeof` operator, when used against a variable instead of a value, is reporting the value-type of _the value in the variable_:

JS variables themselves don't have types. They hold any arbitrary value, which itself has a value-type.

### BigInteger Values

As the maximum safe integer in JS `number`s is `9007199254740991` (see above), such a relatively low limit can present a problem if a JS program needs to perform larger integer math, or even just hold values like 64-bit integer IDs (e.g., Twitter Tweet IDs).

For that reason, JS provides the alternate `bigint` type (BigInteger), which can store arbitrarily large (theoretically not limited, except by finite machine memory and/or JS implementation) integers.

To distinguish a `bigint` from a whole (integer) `number` value, which would otherwise both look the same (`42`), JS requires an `n` suffix on `bigint` values:

```js
myAge = 42n; // this is a bigint, not a number

myKidsAge = 11; // this is a number, not a bigint
```

Let's illustrate the upper un-boundedness of `bigint`:

```js
Number.MAX_SAFE_INTEGER; // 9007199254740991

Number.MAX_SAFE_INTEGER + 2; // 9007199254740992 -- oops!

myBigInt = 9007199254740991n;

myBigInt + 2n; // 9007199254740993n -- phew!

myBigInt ** 2n; // 81129638414606663681390495662081n
```

As you can see, the `bigint` value-type is able to do precise arithmetic above the integer limit of the `number` value-type.

| WARNING:                                                                                                                                                                                                                                                                                           |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Notice that the `+` operator required `.. + 2n` instead of just `.. + 2`? You cannot mix `number` and `bigint` value-types in the same expression. This restriction is annoying, but it protects your program from invalid mathematical operations that would give non-obvious unexpected results. |

A `bigint` value can also be created with the `BigInt(..)` function; for example, to convert a whole (integer) `number` value to a `bigint`:

```js
myAge = 42n;

inc = 1;

myAge += BigInt(inc);

myAge; // 43n
```

| WARNING:                                                                                                                                                        |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Though it may seem counter-intuitive to some readers, `BigInt(..)` is _always_ called without the `new` keyword. If `new` is used, an exception will be thrown. |

That's definitely one of the most common usages of the `BigInt(..)` function: to convert `number`s to `bigint`s, for mathematical operation purposes.

But it's not that uncommon to represent large integer values as strings, especially if those values are coming to the JS environment from other language environments, or via certain exchange formats, which themselves do not support `bigint`-style values.

As such, `BigInt(..)` is useful to coerce those string values to `bigint`s:

```js
myBigInt = BigInt("12345678901234567890");

myBigInt; // 12345678901234567890n
```

Unlike `parseInt(..)`, if any character in the string is non-numeric (`0-9` digits or `-`), including `.` or even a trailing `n` suffix character, an exception will be thrown. In other words, `BigInt(..)` is an all-or-nothing coercion-conversion, not a parsing-conversion.

| NOTE:                                                                                                                                                                                                                                                                                                          |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| I think it's absurd that `BigInt(..)` won't accept the trailing `n` character while string coercing (and thus effectively ignore it). I lobbied vehemently for that behavior, in the TC39 process, but was ultimately denied. In my opinion, it's now a tiny little gotcha wart on JS, but a wart nonetheless. |

## Special Values :

### Invalid Number

Mathematical operations can sometimes produce an invalid result. For example:

```js
42 / "Kyle"; // NaN
```

It's probably obvious, but if you try to divide a number by a string, that's an invalid mathematical operation.

Another type of invalid numeric operation is trying to coercively-convert a non-numeric resembling value to a `number`. As discussed earlier, we can do so with either the `Number(..)` function or the unary `+` operator:

```js
myAge = Number("just a number");

myAge; // NaN

+undefined; // NaN
```

All such invalid operations (mathematical or coercive/numeric) produce the special `number` value called `NaN`.

The historical root of "NaN" (from the IEEE-754[^IEEE754] specification) is as an acronym for "Not a Number". Technically, there are about 9 quadrillion values in the 64-bit IEEE-754 number space designated as "NaN", but JS treats all of them indistinguishably as the single `NaN` value.

Unfortunately, that _not a number_ meaning produces confusion, since `NaN` is _absolutely_ a `number`.

| TIP:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Why is `NaN` a `number`?!? Think of the opposite: what if a mathematical/numeric operation, like `+` or `/`, produced a non-`number` value (like `null`, `undefined`, etc)? Wouldn't that be really strange and unexpected? What if they threw exceptions, so that you had to `try..catch` all your math? The only sensible behavior is, numeric/mathematical operations should _always_ produce a `number`, even if that value is invalid because it came from an invalid operation. |

To avoid such confusion, I strongly prefer to define "NaN" as any of the following instead:

- "iNvalid Number"
- "Not actual Number"
- "Not available Number"
- "Not applicable Number"

`NaN` is a special value in JS, in that it's the only value in the language that lacks the _identity property_ -- it's never equal to itself.

```js
NaN === NaN; // false
```

So unfortunately, the `===` operator cannot check a value to see if it's `NaN`. But there are some ways to do so:

```js
politicianIQ = "nothing" / Infinity;

Number.isNaN(politicianIQ); // true

Object.is(NaN, politicianIQ); // true
[NaN].includes(politicianIQ); // true
```

Here's a fact of virtually all JS programs, whether you realize it or not: `NaN` happens. Seriously, almost all programs that do any math or numeric conversions are subject to `NaN` showing up.

If you're not properly checking for `NaN` in your programs where you do math or numeric conversions, I can say with some degree of certainty: you probably have a number bug in your program somewhere, and it just hasn't bitten you yet (that you know of!).

| WARNING:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| JS originally provided a global function called `isNaN(..)` for `NaN` checking, but it unfortunately has a long-standing coercion bug. `isNaN("Kyle")` returns `true`, even though the string value `"Kyle"` is most definitely _not_ the `NaN` value. This is because the global `isNaN(..)` function forces any non-`number` argument to coerce to a `number` first, before checking for `NaN`. Coercing `"Kyle"` to a `number` produces `NaN`, so now the function sees a `NaN` and returns `true`! This buggy global `isNaN(..)` still exists in JS, but should never be used. When `NaN` checking, always use `Number.isNaN(..)`, `Object.is(..)`, etc. |

### Double Zeros

It may surprise you to learn that JS has two zeros: `0`, and `-0` (negative zero). But what on earth is a "negative zero"? [^SignedZero] A mathematician would surely balk at such a notion.

This isn't just a funny JS quirk; it's mandated by the IEEE-754[^IEEE754] specification. All floating point numbers are signed, including zero. And though JS does kind of hide the existence of `-0`, it's entirely possible to produce it and to detect it:

```js
function isNegZero(v) {
  return v == 0 && 1 / v == -Infinity;
}

regZero = 0 / 1;
negZero = 0 / -1;

regZero === negZero; // true -- oops!
Object.is(-0, regZero); // false -- phew!
Object.is(-0, negZero); // true

isNegZero(regZero); // false
isNegZero(negZero); // true
```

You may wonder why we'd ever need such a thing as `-0`. It can be useful when using numbers to represent both the magnitude of movement (speed) of some item (like a game character or an animation) and also its direction (e.g., negative = left, positive = right).

Without having a signed zero value, you couldn't tell which direction such an item was pointing at the moment it came to rest.

| NOTE:                                                                                                                                                                                                  |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| While JS defines a signed zero in the `number` type, there is no corresponding signed zero in the `bigint` number type. As such, `-0n` is just interpreted as `0n`, and the two are indistinguishable. |

### Object or Primitive?

Unlike other primitives like `42`, where you can create multiple copies of the same value, symbols _do_ act more like specific object references in that they're always completely unique (for purposes of value assignment and equality comparison). The specification also categorizes the `Symbol()` function under the "Fundamental Objects" section, calling the function a "constructor", and even defining its `prototype` property.

However, as mentioned earlier, `new` cannot be used with `Symbol(..)`; this is similar to the `BigInt()` "constructor". We clearly know `bigint` values are primitives, so `symbol` values seem to be of the same _kind_.

And in the specification's "Terms and Definitions", it lists symbol as a primitive value. [^PrimitiveValues] Moreover, the values themselves are used in JS programs as primitives rather than objects. For example, symbols are primarily used as keys in objects -- we know objects cannot use other object values as keys! -- along with strings, which are also primitives.

As mentioned earlier, some JS engines even internally implement symbols as unique, monotonically incrementing integers (primitives!).

Finally, as explained at the top of this chapter, we know primitive values are _not allowed_ to have properties set on them, but are _auto-boxed_ (see "Automatic Objects" in Chapter 3) internally to the corresponding object-wrapper type to facilitate property/method access. Symbols follow all these exact behaviors, the same as all the other primitives.

All this considered, I think symbols are _much more_ like primitives than objects, so that's how I present them in this book.
