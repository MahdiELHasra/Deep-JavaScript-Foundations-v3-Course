# Classy Objects

The class-design pattern generally entails defining a _type of thing_ (class), including data (members) and behaviors (methods), and then creating one or more concrete _instances_ of this class definition as actual objects that can interact and perform tasks. Moreover, class-orientation allows declaring a relationship between two or more classes, through what's called "inheritance", to derive new and augmented "subclasses" that mix-n-match and even re-define behaviors.

Prior to ES6 (2015), JS developers mimicked aspects of class-oriented (aka "object-oriented") design using plain functions and objects, along with the `[[Prototype]]` mechanism -- so called "prototypal classes".

But to many developers joy and relief, ES6 introduced dedicated syntax, including the `class` and `extends` keywords, to express class-oriented design more declaratively.

At the time of ES6's `class` being introduced, this new dedicated syntax was almost entirely _just syntactic sugar_ to make class definitions more convenient and readable. However, in the many years since ES6, `class` has matured and grown into its own first class feature mechanism, accruing a significant amount of dedicated syntax and complex behaviors that far surpass the pre-ES6 "prototypal class" capabilities.

Even though `class` now bears almost no resemblance to older "prototypal class" code style, the JS engine is still _just_ wiring up objects to each other through the existing `[[Prototype]]` mechanism. In other words, `class` is not its own separate pillar of the language (as `[[Prototype]]` is), but more like the fancy, decorative _Capital_ that tops the pillar/column.

That said, since `class` style code has now replaced virtually all previous "prototypal class" coding, the main text here focuses only on `class` and its various particulars. For historical purposes, we'll briefly cover the old "prototypal class" style in an appendix.

## When Should I Class-Orient My Code?

Class-orientation is a design pattern, which means it's a choice for how you organize the information and behavior in your program. It has pros and cons. It's not a universal solution for all tasks.

So how do you know when you should use classes?

In a theoretical sense, class-orientation is a way of dividing up the business domain of a program into one or more pieces that can each be defined by an "is-a" classification: grouping a thing into the set (or sets) of characteristics that thing shares with other similar things. You would say "X is a Y", meaning X has (at least) all the characteristics of a thing of kind Y.

For example, consider computers. We could say a computer is electrical, since it uses electrical current (voltage, amps, etc) as power. It's furthermore electronic, because it manipulates the electrical current beyond simply routing electrons around (electrical/magnetic fields), creating a meaningful circuit to manipulate the current into performing more complex tasks. By contrast, a basic desk lamp is electrical, but not really electronic.

We could thus define a class `Electrical` to describe what electrical devices need and can do. We could then define a further class `Electronic`, and define that in addition to being electrical, `Electronic` things manipulate electricity to create more specialized outcomes.

Here's where class-orientation starts to shine. Rather than re-define all the `Electrical` characteristics in the `Electronic` class, we can define `Electronic` in such a way that it "shares" or "inherits" those characteristics from `Electrical`, and then augments/redefines the unique behaviors that make a device electronic. This relationship between the two classes -- called "inheritance" -- is a key aspect of class-orientation.

So class-orientation is a way of thinking about the entities our program needs, and classifying them into groupings based on their characteristics (what information they hold, what operations can be performed on that data), and defining the relationships between the different grouping of characteristics.

But moving from the theoretical into in a bit more pragmatic perspective: if your program needs to hold and use multiple collections (instances) of alike data/behavior at once, you _may_ benefit from class-orientation.

### Single vs Multiple

I mentioned above that a pragmatic way of deciding if you need class-orientation is if your program is going to have multiple instances of a single kind/type of behavior (aka, "class"). In the timesheet example, we had 4 classes: Timesheet, Week, Day, and Task. But for each class, we had multiple instances of each at once.

Had we instead only needed a single instance of a class, like just one `Computer` thing that was an instance of the `Electronic` class, which was a subclass of the `Electrical` class, then class-orientation may not offer quite as much benefit. In particular, if the program doesn't need to create an instance of the `Electrical` class, then there's no particular benefit to separating `Electrical` from `Electronic`, so we aren't really getting any help from the inheritance aspect of class-orientation.

So, if you find yourself designing a program by dividing up a business problem domain into different "classes" of entities, but in the actual code of the program you are only ever need one concrete _thing_ of one kind/definition of behavior (aka, "class"), you might very well not actually need class-orientation. There are other design patterns which may be a more efficient match to your effort.

But if you find yourself wanting to define classes, and subclasses which inherit from them, and if you're going to be instantiating one or more of those classes multiple times, then class-orientation is a good candidate. And to do class-orientation in JS, you're going to need the `class` keyword.

## Keep It `class`y

`class` defines either a declaration or expression for a class. As a declaration, a class definition appears in a statement position and looks like this:

```js
class Point2d {
  // ..
}
```

As an expression, a class definition appears in a value position and can either have a name or be anonymous:

```js
// named class expression
const pointClass = class Point2d {
  // ..
};

// anonymous class expression
const anotherClass = class {
  // ..
};
```

The contents of a `class` body typically include one or more method definitions:

```js
class Point2d {
  setX(x) {
    // ..
  }
  setY(y) {
    // ..
  }
}
```

Inside a `class` body, methods are defined without the `function` keyword, and there's no `,` or `;` separators between the method definitions.

| NOTE:                                                                                                                                                                                                 |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Inside a `class` block, all code runs in strict-mode even without the `"use strict"` pragma present in the file or its functions. In particular, this impacts the `this` behavior for function calls. |

### The Constructor

One special method that all classes have is called a "constructor". If omitted, there's a default empty constructor assumed in the definition.

The constructor is invoked any time a `new` instance of the class is created:

```js
class Point2d {
  constructor() {
    console.log("Here's your new instance!");
  }
}

var point = new Point2d();
// Here's your new instance!
```

Even though the syntax implies a function actually named `constructor` exists, JS defines a function as specified, but with the name of the class (`Point2d` above):

```js
typeof Point2d; // "function"
```

It's not _just_ a regular function, though; this special kind of function behaves a bit differently:

```js
Point2d.toString();
// class Point2d {
//   ..
// }

Point2d();
// TypeError: Class constructor Point2d cannot
// be invoked without 'new'

Point2d.call({});
// TypeError: Class constructor Point2d cannot
// be invoked without 'new'
```

You can construct as many different instances of a class as you need:

```js
var one = new Point2d();
var two = new Point2d();
var three = new Point2d();
```

Each of `one`, `two`, and `three` here are objects that are independent instances of the `Point2d` class.

| NOTE:                                                                                                                                                                                                               |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Each of the `one`, `two`, and `three` objects have a `[[Prototype]]` linkage to the `Point2d.prototype` object. In this code, `Point2d` is both a `class` definition and the constructor function of the same name. |

If you add a property to the object `one`:

```js
one.value = 42;
```

That property now exists only on `one`, and does not exist in any way that the independent `two` or `three` objects can access:

```js
two.value; // undefined
three.value; // undefined
```

### Class Methods

As shown above, a class definition can include one or more method definitions:

```js
class Point2d {
  constructor() {
    console.log("Here's your new instance!");
  }
  setX(x) {
    console.log(`Setting x to: ${x}`);
    // ..
  }
}

var point = new Point2d();

point.setX(3);
// Setting x to: 3
```

The `setX` property (method) _looks like_ it exists on (is owned by) the `point` object here. But that's a mirage. Each class method is added to the `prototype`object, a property of the constructor function.

So, `setX(..)` only exists as `Point2d.prototype.setX`. Since `point` is `[[Prototype]]` linked to `Point2d.prototype` via the `new` keyword instantiation, the `point.setX(..)` reference traverses the `[[Prototype]]` chain and finds the method to execute.

Class methods should only be invoked via an instance; `Point2d.setX(..)` doesn't work because there _is no_ such property. You _could_ invoke `Point2d.prototype.setX(..)`, but that's not generally proper/advised in standard class-oriented coding. Always access class methods via the instances.

## Class Instance `this`

We will cover the `this` keyword in much more detail. But as it relates to class-oriented code, the `this` keyword generally refers to the current instance that is the context of any method invocation.

In the constructor, as well as any methods, you can use `this.` to either add or access properties on the current instance:

```js
class Point2d {
  constructor(x, y) {
    // add properties to the current instance
    this.x = x;
    this.y = y;
  }
  toString() {
    // access the properties from the current instance
    console.log(`(${this.x},${this.y})`);
  }
}

var point = new Point2d(3, 4);

point.x; // 3
point.y; // 4

point.toString(); // (3,4)
```

Any properties not holding function values, which are added to a class instance (usually via the constructor), are referred to as _members_, as opposed to the term _methods_ for executable functions.

While the `point.toString()` method is running, its `this` reference is pointing at the same object that `point` references. That's why both `point.x` and `this.x` reveal the same `3` value that the constructor set with its `this.x = x` operation.

### Public Fields

Instead of defining a class instance member imperatively via `this.` in the constructor or a method, classes can declaratively define _fields_ in the `class` body, which correspond directly to members that will be created on each instance:

```js
class Point2d {
  // these are public fields
  x = 0;
  y = 0;

  constructor(x, y) {
    // set properties (fields) on the current instance
    this.x = x;
    this.y = y;
  }
  toString() {
    // access the properties from the current instance
    console.log(`(${this.x},${this.y})`);
  }
}
```

Public fields can have a value initialization, as shown above, but that's not required. If you don't initialize a field in the class definition, you almost always should initialize it in the constructor.

Fields can also reference each other, via natural `this.` access syntax:

```js
class Point3d {
  // these are public fields
  x;
  y = 4;
  z = this.y * 5;

  // ..
}
```

| TIP:                                                                                                                                                                                                                                                                                     |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| You can mostly think of public field declarations as if they appear at the top of the `constructor(..)`, each prefixed with an implied `this.` that you get to omit in the declarative `class` body form. But, there's a catch! See "That's Super!" later for more information about it. |

Just like computed property names, field names can be computed:

```js
var coordName = "x";

class Point2d {
  // computed public field
  [coordName.toUpperCase()] = 42;

  // ..
}

var point = new Point2d(3, 4);

point.x; // 3
point.y; // 4

point.X; // 42
```

## Class Extension

The way to unlock the power of class inheritance is through the `extends` keyword, which defines a relationship between two classes:

```js
class Point2d {
  x = 3;
  y = 4;

  getX() {
    return this.x;
  }
}

class Point3d extends Point2d {
  x = 21;
  y = 10;
  z = 5;

  printDoubleX() {
    console.log(`double x: ${this.getX() * 2}`);
  }
}

var point = new Point2d();

point.getX(); // 3

var anotherPoint = new Point3d();

anotherPoint.getX(); // 21
anotherPoint.printDoubleX(); // double x: 42
```

Take a few moments to re-read that code snippet and make sure you fully understand what's happening.

The base class `Point2d` defines fields (members) called `x` and `y`, and gives them the initial values `3` and `4`, respectively. It also defines a `getX()` method that accesses this `x` instance member and returns it. We see that behavior illustrated in the `point.getX()` method call.

But the `Point3d` class extends `Point2d`, making `Point3d` a derived-class, child-class, or (most commonly) subclass. In `Point3d`, the same `x` property that's inherited from `Point2d` is re-initialized with a different `21` value, as is the `y` overridden to value from `4`, to `10`.

It also adds a new `z` field/member method, as well as a `printDoubleX()` method, which itself calls `this.getX()`.

When `anotherPoint.printDoubleX()` is invoked, the inherited `this.getX()` is thus invoked, and that method makes reference to `this.x`. Since `this` is pointing at the class instance (aka, `anotherPoint`), the value it finds is now `21` (instead of `3` from the `point` object's `x` member).

### That's Super!

In addition to a subclass method accessing an inherited method definition (even if overriden on the subclass) via `super.` reference, a subclass constructor must manually invoke the inherited base class constructor via `super(..)` function invocation:

```js
class Point2d {
  x;
  y;
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class Point3d extends Point2d {
  z;
  constructor(x, y, z) {
    super(x, y);
    this.z = z;
  }
  toString() {
    console.log(`(${this.x},${this.y},${this.z})`);
  }
}

var point = new Point3d(3, 4, 5);

point.toString(); // (3,4,5)
```

| WARNING:                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| An explicitly defined subclass constructor _must_ call `super(..)` to run the inherited class's initialization, and that must occur before the subclass constructor makes any references to `this` or finishes/returns. Otherwise, a runtime exception will be thrown when that subclass constructor is invoked (via `new`). If you omit the subclass constructor, the default constructor automatically -- thankfully! -- invokes `super()` for you. |

One nuance to be aware of: if you define a field (public or private) inside a subclass, and explicitly define a `constructor(..)` for this subclass, the field initializations will be processed not at the top of the constructor, but _between_ the `super(..)` call and any subsequent code in the constructor.

Pay close attention to the order of console messages here:

```js
class Point2d {
  x;
  y;
  constructor(x, y) {
    console.log("Running Point2d(..) constructor");
    this.x = x;
    this.y = y;
  }
}

class Point3d extends Point2d {
  z = console.log("Initializing field 'z'");

  constructor(x, y, z) {
    console.log("Running Point3d(..) constructor");
    super(x, y);

    console.log(`Setting instance property 'z' to ${z}`);
    this.z = z;
  }
  toString() {
    console.log(`(${this.x},${this.y},${this.z})`);
  }
}

var point = new Point3d(3, 4, 5);
// Running Point3d(..) constructor
// Running Point2d(..) constructor
// Initializing field 'z'
// Setting instance property 'z' to 5
```

As the console messages illustrate, the `z = ..` field initialization happens _immediately after_ the `super(x,y)` call, _before_ the `` console.log(`Setting instance...`) `` is executed. Perhaps think of it like the field initializations attached to the end of the `super(..)` call, so they run before anything else in the constructor does.

### "Inheritance" Is Sharing, Not Copying

It may seem as if `Point3d`, when it `extends` the `Point2d` class, is in essence getting a _copy_ of all the behavior defined in `Point2d`. Moreover, it may seem as if the concrete object instance `anotherPoint` receives, _copied down_ to it, all the methods from `Point3d` (and by extension, also from `Point2d`).

However, that's not the correct mental model to use for JS's implementation of class-orientation. Recall this base class and subclass definition, as well as instantiation of `anotherPoint`:

```js
class Point2d {
  x;
  y;
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class Point3d extends Point2d {
  z;
  constructor(x, y, z) {
    super(x, y);
    this.z = z;
  }
  toString() {
    console.log(`(${this.x},${this.y},${this.z})`);
  }
}

var anotherPoint = new Point3d(3, 4, 5);
```

If you inspect the `anotherPoint` object, you'll see it only has the `x`, `y`, and `z` properties (instance members) on it, but not the `toString()` method:

```js
Object.hasOwn(anotherPoint, "x"); // true
Object.hasOwn(anotherPoint, "y"); // true
Object.hasOwn(anotherPoint, "z"); // true

Object.hasOwn(anotherPoint, "toString"); // false
```

Where is that `toString()` method located? On the prototype object:

```js
Object.hasOwn(Point3d.prototype, "toString"); // true
```

And `anotherPoint` has access to that method via its `[[Prototype]]` linkage . In other words, the prototype objects **share access** to their method(s) with the subclass(es) and instance(s). The method(s) stay in place, and are not copied down the inheritance chain.

As nice as the `class` syntax is, don't forget what's really happening under the syntax: JS is _just_ wiring up objects to each other along a `[[Prototype]]` chain.

## Static Class Behavior

We've so far emphasized two different locations for data or behavior (methods) to reside: on the constructor's prototype, or on the instance. But there's a third option: on the constructor (function object) itself.

In a traditional class-oriented system, methods defined on a class are not concrete things you could ever invoke or interact with. You have to instantiate a class to have a concrete object to invoke those methods with. Prototypal languages like JS blur this line a bit: all class-defined methods are "real" functions residing on the constructor's prototype, and you could therefore invoke them. But as I asserted earlier, you really _should not_ do so, as this is not how JS assumes you will write your `class`es, and there are some weird corner-case behaviors you may run into. Best to stay on the narrow path that `class` lays out for you.

Not all behavior that we define and want to associate/organize with a class _needs_ to be aware of an instance. Moreover, sometimes a class needs to publicly define data (like constants) that developers using that class need to access, independent of any instance they may or may not have created.

So, how does a class system enable defining such data and behavior that should be available with a class but independent of (unaware of) instantiated objects? **Static properties and functions**.

| NOTE:                                                                                                                                                                                                                                                 |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| I'll use "static property" / "static function", rather than "member" / "method", just so it's clearer that there's a distinction between instance-bound members / instance-aware methods, and non-instance properties and instance-unaware functions. |

We use the `static` keyword in our `class` bodies to distinguish these definitions:

```js
class Point2d {
  // class statics
  static origin = new Point2d(0, 0);
  static distance(point1, point2) {
    return Math.sqrt((point2.x - point1.x) ** 2 + (point2.y - point1.y) ** 2);
  }

  // instance members and methods
  x;
  y;
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  toString() {
    return `(${this.x},${this.y})`;
  }
}

console.log(`Starting point: ${Point2d.origin}`);
// Starting point: (0,0)

var next = new Point2d(3, 4);
console.log(`Next point: ${next}`);
// Next point: (3,4)

console.log(`Distance: ${Point2d.distance(Point2d.origin, next)}`);
// Distance: 5
```

The `Point2d.origin` is a static property, which just so happens to hold a constructed instance of our class. And `Point2d.distance(..)` is a static function that computes the 2-dimensional cartesian distance between two points.

Of course, we could have put these two somewhere other than as `static`s on the class definition. But since they're directly related to the `Point2d` class, it makes _most sense_ to organize them there.

| NOTE:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Don't forget that when you use the `class` syntax, the name `Point2d` is actually the name of a constructor function that JS defines. So `Point2d.origin` is just a regular property access on that function object. That's what I meant at the top of this section when I referred to a third location for storing _things_ related to classes; in JS, `static`s are stored as properties on the constructor function. Take care not to confuse those with properties stored on the constructor's `prototype` (methods) and properties stored on the instance (members). |

### Static Property Initializations

The value in a static initialization (`static whatever = ..`) can include `this` references, which refers to the class itself (actually, the constructor) rather than to an instance:

```js
class Point2d {
  // class statics
  static originX = 0;
  static originY = 0;
  static origin = new this(this.originX, this.originY);

  // ..
}
```

| WARNING:                                                                                                                                                                                                                         |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| I don't recommend actually doing the `new this(..)` trick I've illustrated here. That's just for illustration purposes. The code would read more cleanly with `new Point2d(this.originX,this.originY)`, so prefer that approach. |

An important detail not to gloss over: unlike public field initializations, which only happen once an instantiation (with `new`) occurs, class static initializations always run _immediately_ after the `class` has been defined. Moreover, the order of static initializations matters; you can think of the statements as if they're being evaluated one at a time.

Also like class members, static properties do not have to be initialized (default: `undefined`), but it's much more common to do so. There's not much utility in declaring a static property with no initialized value (`static whatever`); Accessing either `Point2d.whatever` or `Point2d.nonExistent` would both result in `undefined`.

Recently (in ES2022), the `static` keyword was extended so it can now define a block inside the `class` body for more sophisticated initialization of `static`s:

```js
class Point2d {
  // class statics
  static origin = new Point2d(0, 0);
  static distance(point1, point2) {
    return Math.sqrt((point2.x - point1.x) ** 2 + (point2.y - point1.y) ** 2);
  }

  // static initialization block (as of ES2022)
  static {
    let outerPoint = new Point2d(6, 8);
    this.maxDistance = this.distance(this.origin, outerPoint);
  }

  // ..
}

Point2d.maxDistance; // 10
```

The `let outerPoint = ..` here is not a special `class` feature; it's exactly like a normal `let` declaration in any normal block of scope (see the "Scope & Closures" book of this series). We're merely declaring a localized instance of `Point2d` assigned to `outerPoint`, then using that value to derive the assignment to the `maxDistance` static property.

Static initialization blocks are also useful for things like `try..catch` statements around expression computations.

### Static Inheritance

Class statics are inherited by subclasses (obviously, as statics!), can be overriden, and `super` can be used for base class references (and static function polymorphism), all in much the same way as inheritance works with instance members/methods:

```js
class Point2d {
    static origin = /* .. */
    static distance(x,y) { /* .. */ }

    static {
        // ..
        this.maxDistance = /* .. */;
    }

    // ..
}

class Point3d extends Point2d {
    // class statics
    static origin = new Point3d(
        // here, `this.origin` references wouldn't
        // work (self-referential), so we use
        // `super.origin` references instead
        super.origin.x, super.origin.y, 0
    )
    static distance(point1,point2) {
        // here, super.distance(..) is Point2d.distance(..),
        // if we needed to invoke it

        return Math.sqrt(
            ((point2.x - point1.x) ** 2) +
            ((point2.y - point1.y) ** 2) +
            ((point2.z - point1.z) ** 2)
        );
    }

    // instance members/methods
    z
    constructor(x,y,z) {
        super(x,y);     // <-- don't forget this line!
        this.z = z;
    }
    toString() {
        return `(${this.x},${this.y},${this.z})`;
    }
}

Point2d.maxDistance;        // 10
Point3d.maxDistance;        // 10
```

As you can see, the static property `maxDistance` we defined on `Point2d` was inherited as a static property on `Point3d`.

| TIP:                                                                                                                                                             |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Remember: any time you define a subclass constructor, you'll need to call `super(..)` in it, usually as the first statement. I find that all too easy to forget. |

Don't skip over the underlying JS behavior here. Just like method inheritance discussed earlier, the static "inheritance" is _not_ a copying of these static properties/functions from base class to subclass; it's sharing via the `[[Prototype]]` chain. Specifically, the constructor function `Point3d()` has its `[[Prototype]]` linkage changed by JS (from the default of `Function.prototype`) to `Point2d`, which is what allows `Point3d.maxDistance` to delegate to `Point2d.maxDistance`.

It's also interesting, perhaps only historically now, to note that static inheritance -- which was part of the original ES6 `class` mechanism feature set! -- was one specific feature that went beyond "just syntax sugar". Static inheritance, as we see it illustrated here, was _not_ possible to achieve/emulate in JS prior to ES6, in the old prototypal-class style of code. It's a special new behavior introduced only as of ES6.

## Private Class Behavior

Everything we've discussed so far as part of a `class` definition is publicly visible/accessible, either as static properties/functions on the class, methods on the constructor's `prototype`, or member properties on the instance.

But how do you store information that cannot be seen from outside the class? This was one of the most asked for features, and biggest complaints with JS's `class`, up until it was finally addressed in ES2022.

`class` now supports new syntax for declaring private fields (instance members) and private methods. In addition, private static properties/functions are possible.
