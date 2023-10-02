# Equality

Thus far, the coercions we've seen have been focused on single values. We turn out attention now to equality comparisons, which inherently involve two values, either or both of which may be subject to coercion.

Earlier in this chapter, we talked about several abstract operations for value equality comparison.

For example, the `SameValue()` operation[^SameValue] is the strictest of the equality comparisons, with absolutely no coercion. The most obvious JS operation that relies on `SameValue()` is:

```js
Object.is(42,42);                   // true
Object.is(-0,-0);                   // true
Object.is(NaN,NaN);                 // true

Object.is(0,-0);                    // false
```

The `SameValueZero()` operation -- recall, it only differs from `SameValue()` by treating `-0` and `0` as indistinguishable -- is used in quite a few more places, including:

```js
[ 1, 2, NaN ].includes(NaN);        // true
```

We can see the `0` / `-0` misdirection of `SameValueZero()` here:

```js
[ 1, 2, -0 ].includes(0);           // true  <--- oops!

(new Set([ 1, 2, 0 ])).has(-0);     // true  <--- ugh

(new Map([[ 0, "ok" ]])).has(-0);   // true  <--- :(
```

In these cases, there's a *coercion* (of sorts!) that treats `-0` and `0` as indistinguishable. No, that's not technically a "coercion" in that the type is not being changed, but I'm sort of fudging the definition to *include* this case in our broader discussion of coercion here.

Contrast the `includes()` / `has()` methods here, which activate `SameValueZero()`, with the good ol' `indexOf(..)` array utility, which instead activates `IsStrictlyEqual()` instead. This algorithm is slightly more "coercive" than `SameValueZero()`, in that it prevents `NaN` values from ever being treated as equal to each other:

```js
[ 1, 2, NaN ].indexOf(NaN);         // -1  <--- not found
```

If these nuanced quirks of `includes(..)` and `indexOf(..)` bother you, when searching -- looking for an equality match within -- for a value in an array, you can avoid any "coercive" quicks and *force* the strictest `SameValue()` equality matching, via `Object.is(..)`:

```js
vals = [ 0, 1, 2, -0, NaN ];

vals.find(v => Object.is(v,-0));            // -0
vals.find(v => Object.is(v,NaN));           // NaN

vals.findIndex(v => Object.is(v,-0));       // 3
vals.findIndex(v => Object.is(v,NaN));      // 4
```

#### Equality Operators: `==` vs `===`

The most obvious place where *coercion* is involved in equality checks is with the `==` operator. Despite any pre-conceived notions you may have about `==`, it behaves extremely predictably, ensuring that both operands match types before performing its equality check.

To state something that may or may not be super obvious: the `==` (and `===`) operators always return a `boolean` (`true` or `false`), indicating the result of the equality check; they never return anything else, regardless of what coercion may happen.

Now, recall and review the steps discussed earlier in the chapter for the `IsLooselyEqual()` operation. [^LooseEquality] Its behavior, and thus how `==` acts, can be pragmatically intuited with just these two facts in mind:

1. If the types of both operands are the same, `==` has the exact same behavior as `===` -- `IsLooselyEqual()` immediately delegates to `IsStrictlyEqual()`. [^StrictEquality]

    For example, when both operands are object references:

    ```js
    myObj = { a: 1 };
    anotherObj = myObj;

    myObj == anotherObj;                // true
    myObj === anotherObj;               // true
    ```

    Here, `==` and `===` determine that both of their respective operands are of the `object` reference type, so both equality checks behave identically; they compare the object references for equality.

2. But if the operand types differ, `==` allows coercion until they match, and prefers numeric comparison; it attempts to coerce both operands to numbers, if possible:

    ```js
    42 == "42";                         // true
    ```

    Here, the `"42"` string is coerced to a `42` number (not vice versa), and thus the comparison is then `42 == 42`, and must clearly return `true`.


Armed with this knowledge, we'll now dispel the common myth that only `===` checks the type and value, while `==` checks only the value. Not true!

In fact, `==` and `===` are both type-sensitive, each checking the types of their operands. The `==` operator allows coercion of mismatched types, whereas `===` disallows any coercion.

It's a nearly universally held opinion that `==` should be avoided in favor of `===`. I may be one of the only developers who publicly advocates a clear and straight-faced case for the opposite. I think the main reason people instead prefer `===`, beyond simply conforming to the status quo, is a lack of taking the time to actually understand `==`.

I'll be revisiting this topic to make the case for preferring `==` over `===`, later in this chapter, in "Type Aware Equality". All I ask is, no matter how strongly you currently disagree with me, try to keep an open mindset.
