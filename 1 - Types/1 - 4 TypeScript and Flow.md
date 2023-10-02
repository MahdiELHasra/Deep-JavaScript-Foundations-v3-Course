# Type Awareness

We've now sliced and diced and examined coercion from every conceivable angle, starting from the abstract internals of the specification, then moving to the concrete expressions and statements that actually trigger the coercions.

But what's the point of all this? Is the detail in this chapter, and indeed this whole book up to this point, mostly just trivia? Eh, I don't think so.

Let's return to the observations/questions I posed way back at the beginning of this long chapter.

There's no shortage of opinions (especially negative) about coercion. The nearly universally held position is that coercion is mostly/entirely a _bad part_ of JS's language design. But inspite of that reality, most every developer, in most every JS program ever written, faces the reality that coercion cannot be avoided.

In other words, no matter what you do, you won't be able to get away from the need to be aware of, understand, and manage JS's value-types and the conversions them. Contrary to common assumptions, embracing a dynamically-typed (or even a weakly-typed) language, does _not_ mean being careless or unaware of types.

Type-aware programming is always, always better than type ignorant/agnostic programming.

### Uhh... TypeScript?

Surely you're thinking at this moment: "Why can't I just use TypeScript and declare all my types statically, avoiding all the confusion of dynamic typing and coercion?"

| NOTE:                                                                                                                                                                 |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| I have many more detailed thoughts on TypeScript and the larger role it plays in our ecosystem; I'll save those opinions for the appendix ("Thoughts on TypeScript"). |

Let's start by addressing head on the ways TypeScript does, and does not, aid in type-aware programming, as I'm advocating.

TypeScript is both **statically-typed** (meaning types are declared at author time and checked at compile-time) and **strongly-typed** (meaning variables/containers are typed, and these associations are enforced; strongly-typed systems also disallow _implicit_ coercion). The greatest strength of TypeScript is that it typically forces both the author of the code, and the reader of the code, to confront the types comprising most (ideally, all!) of a program. That's definitely a good thing.

By contrast, JS is **dynamically-typed** (meaning types are discovered and managed purely at runtime) and **weakly-typed** (meaning variables/containers are not typed, so there's no associations to enforce, and variables can thus hold any value-types; weakly-typed systems allow any form of coercion).

| NOTE:                                                                                                                                                                                                                                                                                        |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| I'm hand-waving at a pretty high level here, and intentionally not diving deeply into lots of nuance on the static/dynamic and strong/weak typing spectrums. If you're feeling the urge to "Well, actually..." me at this moment, please just hold on a bit and let me lay out my arguments. |

### Type-Awareness _Without_ TypeScript

Does a dynamically-typed system automatically mean you're programming with less type-awareness? Many would argue that, but I disagree.

I do not at all think that declaring static types (annotations, as in TypeScript) is the only way to accomplish effective type-awareness. Clearly, though, proponents of static-typing believe that is the _best_ way.

Let me illustrate type-awareness without TypeScript's static typing. Consider this variable declaration:

```js
let API_BASE_URL = "https://some.tld/api/2";
```

Is that statement in any way _type-aware_? Sure, there's no `: string` annotation after `API_BASE_URL`. But I definitely think it _is_ still type-aware! We clearly see the value-type (`string`) of the value being assigned to `API_BASE_URL`.

| WARNING:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Don't get distracted by the `let` declaration being re-assignable (as opposed to a `const`). JS's `const` is _not_ a first-class feature of its type system. We don't really gain additional type-awareness simply because we know that reassignment of a `const` variable is disallowed by the JS engine. If the code is structured well -- ahem, structured with type-awareness as a priority -- we can just read the code and see clearly that `API_BASE_URL` is _not_ reassigned and is thus still the value-type it was previously assigned. From a type-awareness perspective, that's effectively the same thing as if it _couldn't_ be reassigned. |

If I later want to do something like:

```js
// are we using the secure API URL?
isSecureAPI = /^https/.test(API_BASE_URL);
```

I know the regular-expression `test(..)` method expects a string, and since I know `API_BASE_URL` is holding a string, I know that operation is type-safe.

Similarly, since I know the simple rules of `ToBoolean()` coercion as it relates to string values, I know this kind of statement is also type-safe:

```js
// do we have an API URL determined yet?
if (API_BASE_URL) {
  // ..
}
```

But if later, I start to type something like this:

```js
APIVersion = Number(API_BASE_URL);
```

A warning siren triggers in my head. Since I know there's some very specific rules about how string values coerce to numbers, I recognize that this operation is **not** type-safe. So I instead approach it differently:

```js
// pull out the version number from API URL
versionDigit = API_BASE_URL.match(/\/api\/(\d+)$/)[1];

// make sure the version is actually a number
APIVersion = Number(versionDigit);
```

I know that `API_BASE_URL` is a string, and I further know the format of its contents includes `".../api/{digits}"` at the end. That lets me know that the regular expression match will succeed, so the `[1]` array access is type-safe.

I also know that `versionDigit` will hold a string, because that's what regular-expression matches return. Now, I know it's safe to coerce that numeric-digit string into a number with `Number(..)`.

By my definition, that kind of thinking, and that style of coding, is type-aware. Type-awareness in coding means thinking carefully about whether or not such things will be _clear_ and _obvious_ to the reader of the code.

### Type-Awareness _With_ TypeScript

TypeScript fans will point out that TypeScript can, via type inference, do static typing (enforcement) without ever needing a single type annotation in the program. So all the code examples I shared in the previous section, TypeScript can also handle, and provide its flavor of compile-time static type enforcement.

In other words, TypeScript will give us the same kind of benefit in type checking, whichever of these two we write:

```ts
let API_BASE_URL: string = "https://some.tld/api/2";

// vs:

let API_BASE_URL = "https://some.tld/api/2";
```

But there's no free-lunch. We have some issues we need to confront. First of all, TypeScript does _not_ trigger an error here:

```js
API_BASE_URL = "https://some.tld/api/2";

APIVersion = Number(API_BASE_URL);
// NaN
```

Intuitively, _I_ want a type-aware system to understand why that's unsafe. But maybe that's just too much to ask. Or perhaps if we actually define a more narrow/specific type for that `API_BASE_URL` variable, than simply `string`, it might help? We can use a TypeScript trick called "Template Literal Types": [^TSLiteralTypes]

```ts
type VersionedURL = `https://some.tld/api/${number}`;

API_BASE_URL: VersionedURL = "https://some.tld/api/2";

APIVersion = Number(API_BASE_URL);
// NaN
```

Nope, TypeScript still doesn't see any problem with that. Yes, I know there's an explanation for why (how `Number(..)` itself is typed).

| NOTE:                                                                                                                                                                                                                                                                |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| I imagine the really smart folks who _know_ TypeScript well have creative ideas on how we can contort ourselves into raising an error there. Maybe there's even a dozen different ways to force TypeScript to trigger on that code. But that's not really the point. |

My point is, we cannot fully rely on TypeScript types to solve all our problems, letting us check out and remain blissfully unaware of the nuances of types and, in this case, coercion behaviors.

But! You're surely objecting to this line of argument, desperate to assert that even if TypeScript can't understand some specific situation, surely using TypeScript doesn't make it _worse_! Right!?

Let's look at what TypeScript has to say[^TSExample1] about this line:

```ts
type VersionedURL = `https://some.tld/api/${number}`;

let API_BASE_URL: VersionedURL = "https://some.tld/api/2";

let versionDigit = API_BASE_URL.match(/\/api\/(\d+)$/)[1];
// Object is possibly 'null'.
```

The error indicates that the `[1]` access isn't type-safe, because if the regular expression fails to find any match on the string, `match(..)` returns `null`.

You see, even though _I_ can reason about the contents of the string compared to how the regular expression is written, and even if _I_ went to the trouble to make it super clear to TypeScript exactly what those specific string contents are, it's not quite smart enough to line those two up to see that it's actually fully type-safe to assume the match happens.

| TIP:                                                                                                                                                                                                                                             |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Is it really the job of, and best use of, a type-aware tool to be contorted to express every single possible nuance of type-safety? We don't need perfect and universal tools to derive immense amounts of benefit from the stuff they _can_ do. |

Moreover, comparing the code style in the previous section to the code in this section (with or without the annotations), is TypeScript actually making our coding more type-aware?

Like, does that `type VersionedURL = ..` and `API_BASE_URL: VersionedURL` stuff _actually_ make our code more clearly type-aware? I don't necessarily think so.

### TypeScript Intelligence

Yes, I hear you screaming at me through the computer screen. Yes, I know that TypeScript provides what type information it discovers (or infers) to your code editor, which comes through in the form of intelligent autocompletes, helpful inline warning markers, etc.

But I'm arguing that even _those_ don't, in and of themselves, make you more type-aware as a developer

Why? Because type-awareness is _not_ just about the authoring experience. It's also about the reading experience, maybe even more so. And not all places/mechanisms where code is read, have access to benefit from all the extra intelligence.

Look, the magic of a language-server pumping intelligence into your code editor is unquestionably amazing. It's cool and super helpful.

And I don't begrudge TypeScript as a tool inferring things about my **JS code** and giving me hints and suggestions through delightful code editor integrations. I just don't necessarily want to _have_ to annotate type information in some extremely specific way just to silence the tool's complaints.

### The Bar Above TypeScript

But even if I did/had all that, it's still not **_sufficient_** for me to be fully type-aware, both as a code-author and as a code-reader.

These tools don't catch every type error that can happen, no matter how much we want to tell ourselves they can, and no matter how many hoops and contortions we endure to wish it so. All the efforts to coax and _coerce_ a tool into catching those nuanced errors, through endlessly increasing complexity of type syntax tricks, is... at best, misplaced effort.

Moreover, no such tool is immune to false positives, complaining about things which aren't actually errors; these tools will never be as smart as we are as humans. You're really wasting your time in chasing down some quirky syntax trick to quite down the tool's complaints.

There's just no substitute, if you want to truly be a type-aware code-author and code-reader, from learning how the language's built-in type systems work. And yes, that means every single developer on your team needs to spend the efforts to learn it. You can't water this stuff down just to be more attainable for less experienced developers on the project/team.

Even if we granted that you could avoid 100% of all _implicit_ coercions -- you can't -- you are absolutely going to face the need to _explicit_ coercions -- all programs do!

And if your response to that fact is to suggest that you'll just offload the mental burden of understanding them to a tool like TypeScript... then I'm sorry to tell you, but you're plainly and painfully falling short of the _type-aware_ bar that I'm challenging all developers to strive towards.

I'm not advocating, here, for you to ditch TypeScript. If you like it, fine. But I am very explicitly and passionately challenging you: stop using TypeScript as a crutch. Stop prostrating yourself to appease the TypeScript engine overlords. Stop foolishly chasing every type rabbit down every syntactic hole.

From my observation, there's a tragic, inverse relationship between usage of type-aware tooling (like TypeScript) and the desire/effort to pursue actual type-awareness as a code-author and code-reader. The more you rely on TypeScript, the more it seems you're tempted and encouraged to shift your attention away from JS's type system (and especially, from coercion) to the alternate TypeScript type system.

Unfortunately, TypeScript can never fully escape JS's type system, because TypeScript's types are _erased_ by the compiler, and what's left is just JS that the JS engine has to contend with.

| TIP:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Imagine if someone handed you a cup of filtered water to drink. And just before you took a sip, they said, "We extracted that water from the ground near a waste dump. But don't worry, we used a perfectly great filter, and that water is totally safe!" How much do you trust that filter? More to my overall point, wouldn't you feel more comfortable drinking that water if you understood everything about the source of the water, all the processes of filtration, and everything that was _in_ the water of the glass in your hand!? Or is trusting that filter good enough? |

### Type Aware Equality

I'll close this long, winding chapter with one final illustration, modeling how I think developers should -- armed with more critical thinking than bandwagon conformism -- approach type-aware coding, whether you use a tool like TypeScript or not.

We'll yet again revisit equality comparisons (`==` vs `===`), from the perspective of type-awareness. Earlier in this chapter, I promised that I would make the case for `==` over `===`, so here it goes.

Let's restate/summarize what we know about `==` and `===` so far:

1. If the types of the operands for `==` match, it behaves _exactly the same_ as `===`.

2. If the types of the operands for `===` do not match, it will always return `false`.

3. If the types of the operands for `==` do not match, it will allow coercion of either operand (generally preferring numeric type-values), until the types finally match; once they match, see (1).

OK, so let's take those facts and analyze how they might interact in our program.

If you are making an equality comparison of `x` and `y` like this:

```js
if ( /* are x and y equal */ ) {
    // ..
}
```

What are the possible conditions we may be in, with respect to the types of `x` and `y`?

1. We might know exactly what type(s) `x` and `y` could be, because we know how those variables are getting assigned.

2. Or we might not be able to tell what those types could be. It could be that `x` or `y` could be any type, or at least any of several different types, such that the possible combinations of types in the comparison are too complex to understand/predict.

Can we agree that (1) is far preferable to (2)? Can we further agree that (1) represents having written our code in a type-aware fashion, whereas (2) represents code that is decidedly type-_unaware_?

If you're using TypeScript, you're very likely to be aware of the types of `x` and `y`, right? Even if you're not using TypeScript, we've already shown that you can take intentional steps to write your code in such a way that the types of `x` and `y` are known and obvious.

#### (2) Unknown Types

If you're in scenario (2), I'm going to assert that your code is in a problem state. Your code is less-than-ideal. Your code needs to be refactored. The best thing to do, if you find code in this state, is... fix it!

Change the code so it's type-aware. If that means using TypeScript, and even inserting some type annotations, do so. Or if you feel you can get to the type-aware state with _just JS_, do that. Either way, do whatever you can to get to scenario (1).

If you cannot ensure the code doing this equality comparison between `x` and `y` is type-aware, and you have no other options, then you absolutely _must_ use the `===` strict-equality operator. Not doing so would be supremely irresponsible.

```js
if (x === y) {
  // ..
}
```

If you don't know anything about the types, how could you (or any other future reader of your code) have any idea how the coercive steps in `==` are going to behave!? You can't.

The only responsible thing to do is, avoid coercion and use `===`.

But don't lose sight of this fact: you're only picking `===` as a last resort, when your code is so type-unaware -- ahem, type-broken! -- as to have no other choice.

#### (1) Known Types

OK, let's instead assume you're in scenario (1). You know the types of `x` and `y`. It's very clear in the code what this narrow set of types participating in the equality check can be.

Great!

But there's still two possible sub-conditions you may be in:

- (1a): `x` and `y` might already be of the same type, whether that be both are `string`s, `number`s, etc.

- (1b): `x` and `y` might be of different types.

Let's consider each of these cases individually.

##### (1a) Known Matching Types

If the types in the equality comparison match (whatever they are), we already know for certain that `==` and `===` do exactly the same thing. There's absolutely no difference.

Except, `==` _is_ shorter by one character. Most developers feel instinctively that the most terse but equivalent version of something is often most preferable. That's not universal, of course, but it's a general preference at least.

```js
// this is best
if (x == y) {
  // ..
}
```

In this particular case, an extra `=` would do nothing for us to make the code more clear. In fact, it actually would make the comparison worse!

```js
// this is strictly worse here!
if (x === y) {
  // ..
}
```

Why is it worse?

Because in scenario (2), we already established that `===` is used for the last-resort when we don't know enough/anything about the types to be able to predict the outcome. We use `===` when we want to make sure we're avoiding coercion when we know coercion could occur.

But that doesn't apply here! We already know that no coercion would occur. There's no reason to confuse the reader with a `===` here. If you use `===` in a place where you already _know_ the types -- and moreover, they're matched! -- that actually might send a mixed signal to the reader. They might have assumed they knew what would happen in the equality check, but then they see the `===` and they second guess themselves!

Again, to state it plainly, if you know the types of an equality comparison, and you know they match, there's only one right choice: `==`.

```js
// stick to this option
if (x == y) {
  // ..
}
```

##### (1b) Known Mismatched Types

OK, we're in our final scenario. We need to compare `x` and `y`, and we know their types, but we also know their types are **NOT** the same.

Which operator should we use here?

If you pick `===`, you've made a huge mistake. Why!? Because `===` used with known-mismatched types will never, ever, ever return `true`. It will always fail.

```js
// `x` and `y` have different types?
if (x === y) {
  // congratulations, this code in here will NEVER run
}
```

OK. So, `===` is out when the types are known and mismatched. What's our only other choice?

Well, actually, we again have two options. We _could_ decide:

- (1b-1): Let's change the code so we're not trying to do an equality check with known mismatched types; that could involve explicitly coercing one or both values so they types now match, in which case pop back up to scenario (1a).

- (1b-2): If we're going to compare known mismatched types for equality, and we want any hope of that check ever passing, we _must_ used `==`, because it's the only one of the equality operators which can coerce one or both operands until the types match.

```js
// `x` and `y` have different types,
// so let's allow JS to coerce them
// for equality comparison
if (x == y) {
  // .. (so, you're saying there's a chance?)
}
```

That's it. We're done. We've looked at every possible type-sensitive equality comparison condition (between `x` and `y`).

ondition will always return 'false' since
// the types 'number' and 'string' have no overlap.

I am at a loss for words to describe how aggravating that is to me. If you've paid attention to this long, heavy chapter, you know that TypeScript is basically telling a lie here. Of course `42 == "42"` will produce `true` in JS.

Well, it's not a lie, but it's exposing a fundamental truth that so many still don't fully appreciate: TypeScript completely tosses out the normal rules of JS's type system, because TypeScript's position is that JS's type system -- and especially, implicit coercion -- are bad, and need to be replaced.

In TypeScript's world, `42` and `"42"` can never be equal to each other. Hence the error message. But in JS land, `42` and `"42"` are absolutely coercively equal to each other. And I believe I've made a strong case here that they _should be_ assumed to be safely coercively equivalent.

What bothers me even more is, TypeScript has a variety of inconsistencies in this respect. TypeScript is perfectly fine with the _implicit_ coercion in this code:

```js
irony = `The value '42' and ${42} are coercively equal.`;
```

The `42` gets implicitly coerced to a string when interpolating it into the sentence. Why is TypeScript ok with this implicit coercion, but not the `42 == "42"` implicit coercion?

TypeScript has no complaints about this code, either:

```js
API_BASE_URL = "https://some.tld/api/2";
if (API_BASE_URL) {
  // ..
}
```

Why is `ToBoolean()` an OK implicit coercion, but `ToNumber()` in the `==` algorithm is not?

I will leave you to ponder this: do you really think it's a good idea to write code that will ultimately run in a JS engine, but use a tool and style of code that has intentionally ejected most of an entire pillar of the JS language? Moreover, is it fine that it's also flip-flopped with a variety of inconsistent exceptions, simply to cater to the old habits of JS developers?
