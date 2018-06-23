---
title: Notes on "You Don't Know JS" by Kyle Simpson
subtitle: A great book for learning JS internals.
date: 2018-06-07
image: /books/you-dont-know-js.png
tags:
  - javascript
  - books
---

# Notes on "You Don't Know JS" by Kyle Simpson

The best way to understand and avoid language "gotchas" is by understanding what
causes them. It's also a great book to learn how to optimize your JS programs
with very little effort.

I have written a good amount of notes over all the six books. Some may find
these too details, some might find them too shallow. Either way, those notes
cover the whole book series and are meant as a review, so I can learn things
more effectively and quickly review some details.

<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Notes on "You Don't Know JS" by Kyle Simpson](#notes-on-you-dont-know-js-by-kyle-simpson)
	- [Up & Going](#up-going)
	- [Scope & Closures](#scope-closures)
		- [Scope and Lexical Scope](#scope-and-lexical-scope)
		- [Function and Block Scopes](#function-and-block-scopes)
		- [Closures](#closures)
		- [Arrow Functions (ES6)](#arrow-functions-es6)
	- [`this` & Object Prototypes](#this-object-prototypes)
		- [`this`](#this)
		- [Objects](#objects)
		- [Mixins](#mixins)
		- [Prototypes](#prototypes)
		- [Behavior Delegation/OLOO (objects-linked-to-other-objects)](#behavior-delegationoloo-objects-linked-to-other-objects)
		- [ES6 `class`](#es6-class)
	- [Types & Grammar](#types-grammar)
		- [Types](#types)
		- [Values](#values)
			- [Arrays](#arrays)
			- [Strings](#strings)
			- [Numbers](#numbers)
			- [`void` operator](#void-operator)
			- [Value vs. Reference](#value-vs-reference)
		- [Natives](#natives)
		- [Coercion](#coercion)
			- [Explicit Coercion](#explicit-coercion)
			- [Implicit Coercion](#implicit-coercion)
			- [Loose/Strict Equals](#loosestrict-equals)
		- [Grammar](#grammar)
	- [Async & Performance](#async-performance)
		- [Async](#async)
		- [Performance](#performance)
	- [ES6 & Beyond](#es6-beyond)

<!-- /TOC -->

## Up & Going

This first book is great for getting started into the series, specially if you
have a shallow understanding of JS internals. That is because each of the other
books will assume you have some basic understanding of the language mechanisms
explained deeply in the other books of the series.

But if you already have a good understanding of the language, maybe it's best to
just skip to the other books.

Always use the `"use strict";` in your code. Keeps code safer and optimized.

## Scope & Closures

### Scope and Lexical Scope

- JavaScript compiles statements kind-of individually and shortly before they
  are executed. JS engines also use JIT, lazy compilation and hot compilation.
- We can devide JavaScript compilation in three "characters": Engine, Compiler
  and Scope.
- Types of Scope look-ups:
  - *LHS*: when the variable is the target of the assignment. If it doesn't
    exist, the `Scope` creates a new global variable (when strict mode is not
    enabled). If strict mode is enabled, it will throw a `ReferenceError`.
  - *RHS*: when the variable is the source of the assigment. `Engine` wants the
    value of the variable. If it doesn't exist, it will throw a
    `ReferenceError`.
- JavaScript scopes by using *Lexical Scope*, as opposed to *Dynamic Scope*.
  This means that the scopes are nested and are solely determined by the code
  structure, rather than by the execution stack.
- Lexical scope can be cheated at runtime through `eval` and `with`, but they
  are almost completely disabled in strict mode and the simple fact you are
  using them will disable several optimizations. So, *don't use them*.

### Function and Block Scopes

- Author recommends using named functions all the time, even in expressions.
  Improves code readability and also stack traces.
- JavaScript only restricts variable scope inside functions:
  ```javascript
  if (true) {
    var a = 42;
  }

  (function () { var b = 42; })();

  console.log(a); // I can access a.
  console.log(b); // Error.
  ```
- You should follow the "Principle of Least ~~Privilege~~ Exposure".
- *IIFEs* (Immediately Invoked Function Expressions):
  - `(function() { ... })();`
  - `(function() { ... }());`
  - `(function(foo) { foo("bar"); })(function(str) { ... });`
- Use the ES6 keyword `let` to do block-scoping:
  ```javascript
  {
    let foo = "bar";
  }

  console.log(foo); // ReferenceError
  ```
- Variables declared with `let` are never hoisted.
- This also improves performance, as values are garbage collected earlier.
- The `const` keyword works just like `let`, but is used for constants.
- This ES6 behaviour is transpilled using `try/catch` blocks.
- JavaScript does hoisting with functions and variables. Functions first:
  ```javascript
  foo();

  var a = 1; // Declaration will go up, but not assignment.

  function foo() { // This will go above everything.
    console.log(a); // undefined -- only "a" declaration was hoisted.
  }
  ```

### Closures

- Loops + Closure:
  This code prints `6` five times:
  ```javascript
  for (var i = 1; i <= 5; i++) {
    setTimeout(function() { console.log(i); }, i*100);
  }
  ```
  You can solve this by using a closure:
  ```javascript
  for (var i = 1; i <= 5; i++) {
    (function() {
      var j = i;
      setTimeout(function() { console.log(j); }, j*100);
    })();
  }
  ```
  You can solve this using `let`:
  ```javascript
  for (let i = 1; i <= 5; i++) {
    // you could use the `let j = i;` trick here, but...
    setTimeout(function() { console.log(i); }, i*100);
  }
  ```
  There's a *special behaviour* defined for `let` declarations used in the head
  of a for-loop: the variable is declared *each iteration* and it's initialized
  with the value from the previous iteration.
- *Closures*: when a function can remember and access its lexical scope even
  when it's invoked outside its lexical scope.
- *Modules* require two characteristics:
  - An outer wrapper function being invoked, to create the enclosing scope.
  - The return value of the function should include a reference to at least one
    inner function that then has closure over the private inner scope.
  ```javascript
  var foo = (function MyCoolModule() {
    var a, b, c;

    function doSomething() { ... };

    function doAnother() { ... };

    return {
      doSomething: doSomething,
      doAnother: doAnother
    };
  })();
  ```
- You can use ES6 file modules through `export`, `import` and `module`.

### Arrow Functions (ES6)

- They bind `this` to the lexical context they are in:
  ```javascript
  var id = "not awesome";

  var bad = {
    id: "awesome",
    cool: function coolFn() {
      console.log(this.id);
    }
  };

  var good1 = {
    count: 0,
    cool: function coolFn() {
      if (this.count < 1) {
        setTimeout(() => { // arrow-function ftw?
          this.count++;
          console.log("awesome?");
        }, 100 );
      }
    }
  };

  var good2 = {
    count: 0,
    cool: function coolFn() {
      if (this.count < 1) {
        setTimeout(function timer(){
          this.count++; // `this` is safe because of `bind(..)`
          console.log("more awesome");
        }.bind(this), 100 ); // look, `bind()`!
      }
    }
  };

  bad.cool(); // awesome
  setTimeout(bad.cool, 100); // not awesome
  good1.cool(); // awesome?
  good2.cool(); // more awesome
  ```

## `this` & Object Prototypes

### `this`

- `this` *isn't* a reference to the function itself.
- `this` *isn't* a reference to the function's lexical scope.
- `this` *is* a binding that is made when a function is *invoked*, and what it
  references is determined by the call-site.

- There are four rules for how `this` gets set:
  ```javascript
  function foo() {
    console.log(this.bar);
  }

  var bar = "global";

  var obj1 = {
    bar: "obj1",
    foo: foo
  };

  var obj2 = {
    bar: "obj2"
  };

  foo();            // "global"
  obj1.foo();       // "obj1"
  foo.call( obj2 ); // "obj2"
  new foo();        // undefined
  ```
  1. *Default Binding*: Standalone function invocation. In the example above,
     `foo()` ends up setting this to the global object in non-strict mode. In
     strict mode, `this` would be undefined.
  2. *Implicit Binding*: The call-site has a context object. In the example
     above, `obj1.foo()` sets `this` to the `obj1` object.
  3. *Explicit Binding*: Using `fn.call()`, `fn.apply()` or ES5's `fn.bind()`.
     In the example above, `foo.call(obj2)` sets `this` to the `obj2` object.
     Some API functions have a parameter "context" to be bound in callbacks.
  4. *`new` Binding*: When a function is called with `new`, a brand new object
     is created and set as `this`. Unless the function returns something, the
     new object is returned. In the example above, `new foo()` sets `this` to a
     brand new empty object.
- *Binding exceptions*:
  - Passing `null` or `undefined` to `call`, `apply` or `bind`. Use instead:
    ```javascript
    var ø = Object.create(null); // a "DMZ" empty object
    fn.bind(ø);
    ```
  - Indirection: `(p.foo = o.foo)();`
- *"Soft Binding"*:
  Allows overriding with Implicit/Explicit Binding.
  ```javascript
  if (!Function.prototype.softBind) {
    Function.prototype.softBind = function(obj) {
      var fn = this,
        curried = [].slice.call( arguments, 1 ),
        bound = function bound() {
          return fn.apply(
            (!this ||
              (typeof window !== "undefined" &&
                this === window) ||
              (typeof global !== "undefined" &&
                this === global)
            ) ? obj : this,
            curried.concat.apply( curried, arguments )
          );
        };
      bound.prototype = Object.create( fn.prototype );
      return bound;
    };
  }
  ```
- It is possible to do lexical binding through arrow functions, but the author
  discourages it.

### Objects

- In objects, property names are *always strings*:
  ```javscript
  myObject[myObject] = 42;
  myObject["[object Object]"]; // 42
  ```
- ES6 computed property names:
  ```javascript
  var prefix = "foo";

  var myObject = {
    [prefix + "bar"]: "hello",
    [prefix + "baz"]: "world"
  };

  myObject["foobar"]; // hello
  myObject["foobaz"]; // world
  ```
- Methods "don't exist" in JS, only references to functions.
- Duplicating objects.
  - JSON-safe objects:
    ```javascript
    var newObj = JSON.parse(JSON.stringify(obj));
    ```
  - Shallow copy (ES6):
    ```javascript
    var newObj = Object.assign({}, obj);
    ```
- Property Descriptors: `Object.getOwnPropertyDescriptor`,
  `Object.defineProperty`. The descriptors are: `writable`, `configurable` and
  `enumberable`.
- `Object.preventExtensions`, `Object.seal`, `Object.freeze` to make "immutable"
  objects.
- `[[Get]]` and `[[Set]]` characteristics of a property:
  ```javascript
  var obj = {
    get a() { return this._a_; },
    set a(val) { this._a_ = val * 2; }
  };

  Object.defineProperty(obj, "b", {
    get: function(){ return this.a * 2 },
    enumerable: true
  });

  obj.a = 1;
  obj.a; // 2;
  obj.b; // 4;
  ```
- Test for existence: `("a" in obj);` (includes prototypes),
  `obj.hasOwnProperty("a")`.
- Array of keys: `Object.keys` (enumerable only), `Object.getOwnPropertyNames`.
- Iterate over keys: `for..in` loop (includes prototypes). Use in objects, not
  arrays. Also: `obj.forEach`, `obj.every` and `obj.some`.
- Iterate over values (ES6): `for..of`.
  - Works by accessing the built-in `@@iterator`:
    ```javascript
    var it = myArray[Symbol.iterator]();
    it.next(). // {value: 42, done: false}
    ```
  - It is possible to define custom iterators for objects:
    ```javascript
    Object.defineProperty( myObject, Symbol.iterator, {
      enumerable: false,
      writable: false,
      configurable: true,
      value: function() {
        var o = this, i = 0, k = Object.keys(o);
        return {
          next: function() {
            return {;
              value: o[k[i++]],
              done: (i > k.length)
            };
          }
        };
      }
    });
    ```

### Mixins

- JS has no notion of classes/inheritance/polymorphism.
- Developers use mixings instead:
  ```javascript
  function mixin(sourceObj, targetObj) {
    for (var key in sourceObj) {
      if (!(key in targetObj)) {
        targetObj[key] = sourceObj[key];
      }
    }

    return targetObj;
  }
  ```
- This leads to "explicit pseudo-polymorphism" (which the author discourages):
  ```javascript
  sourceObj.methodName.call(this, ...);
  ```
- Explicit mixings are not exactly the same as class *copy*.
- "Parasitic Inheritance": calling `new Parent()` inside `new Child()`.
- "Implicit Mixings": "borrowing" methods through "implicit binding".

### Prototypes

- `Object.create(obj)` (ES5) creates a object with `obj` in `[[Prototype]]`
  linkage.
- `Object.prototype` is at the top of every "*normal*" `[[Prototype]]` chain.
- Shadowing only occurs if the property found in the `[[Prototype]]` chain is
  writable and it is not a setter.
- A function's `Foo.prototype` gets linked to the `[[Prototype]]` chain of an
  object created with `new Foo();`:
  ```javascript
  Object.getPrototypeOf(new Foo()) === Foo.prototype; // true
  ```
- `Foo.prototype` and the created object (through `[[Prototype]]`) get a
  `.constructor` property:
  ```javascript
  Foo.prototype.constructor === Foo; // true
  (new Foo()).constructor === Foo; // true
  ```
- However, `.constructor` is unreliable and should be avoided where possible.
- We're are not actually *copying* but *linking* "classes".
- How to simulate class-orientation then? Example from the book:
  ```javascript
  function Foo(name) {
    this.name = name;
  }

  Foo.prototype.myName = function() {
    return this.name;
  };

  function Bar(name,label) {
    Foo.call(this, name);
    this.label = label;
  }

  // here, we make a new `Bar.prototype`
  // linked to `Foo.prototype`
  Bar.prototype = Object.create(Foo.prototype);

  // Beware! Now `Bar.prototype.constructor` is gone,
  // and might need to be manually "fixed" if you're
  // in the habit of relying on such properties!

  Bar.prototype.myLabel = function() {
    return this.label;
  };

  var a = new Bar("a", "obj a");

  a.myName(); // "a"
  a.myLabel(); // "obj a"
  ```
- Alternatives to `Bar.prototype = Object.create(Foo.prototype);`:
  - `Bar.prototype = Foo.prototype;`: not a copy, will modify `Foo.prototype`.
  - `Bar.prototype = new Foo();`: if `Foo()` has side-effects, they will happen.
  - ES6's `Object.setPrototypeOf(Bar.prototype, Foo.prototype);`: works well.
- `a instanceof Foo` tests whether `Foo.prototype` appears in the
  `[[Prototype]]` chain of `a`.
- Functions hard-bound with `.bind(..)` loose their `.prototype` property.
- A much cleaner approach is to use `Foo.prototype.isPrototypeOf(a)`.
- Retrieve `[[Prototype]]` of an object: `Object.getPrototypeOf(a);` (ES5) or
  `a.__proto__`.
- You *should not* change the `[[Prototype]]` of an existing object.

### Behavior Delegation/OLOO (objects-linked-to-other-objects)

- An alternative way of thinking/designing software in JS, which the author
  thinks is better than Object Orientation.
- Objects *delegate* common behaviour to a common prototype through `Object.create(..)`.
- You want *state* to be on the *delegators*.
- *We avoid if at all possible naming things the same* at different levels of the `[[Prototype]]` chain.
- Code gets much nicer when using `Object.setPrototypeOf(obj, prototypeObj)`.
- We often can eliminate base classes, because behavior delegation suggests
  objects as peer of each other.

### ES6 `class`

- No need for `.prototype` references in the code.
- `extends` keyword provides inheritance. It is also possible to extend built-in
  objects.
- `constructor` method provides "instantiation".
- `super(..)` and `super.fn(..)` provide "*relative polmorphism*".
- No need for commas.
- *However*, it works mostly the same way as the `[[Prototype]]` mechanism.
- *But* it makes some things "static" where otherwise they would be dynamic.

## Types & Grammar

### Types

- JavaScript has only seven built-in types: `string`, `number`, `boolean`,
  `null`, `undefined`, `object` and `symbol` (introduced in ES6).
- All are called "primitives", except for `object`.
- `typeof` has a long-standing bug, but there is a workaround.
  ```javascript
  typeof null // "object"`

  var a = null;
  (!a && typeof a === "object"); // true
  ```
- `function` is a (special) subtype of `object`:
  ```javascript
  typeof function foo() { return 42; } // "function"

  // but:
  typeof [1, 2, 3] // "object"
  ```
- "In JS, variables don't have types -- values have types."
- We can use `typeof` to prevent errors with undeclared variables, without
  resorting to the global `window` object:
  ```javascript
  if (typeof FLAG !== "undefined") {}
  // or:
  if (window.FLAG) {}
  ```

### Values

#### Arrays

- `delete` removes slots but does not update `.length`.
- Adding named properties to arrays doesn't affect the `length` property, except
  if the property "looks like a number": `myArray["42"] = "foo";`.
- Empty slots are left in this case. They return `undefined` when accessed.
- Build arrays from "array-likes":
  ```javascript
  Array.prototype.slice.call([1, 2, 3]); // [1, 2, 3]
  // or (ES6):
  Array.from([1, 2, 3]); // [1, 2, 3]
  ```

#### Strings

- JS `string`s are immutable.
- There are some array-like methods, but other array methods are not available.
- It is possible to "borrow" some `array` methods:
  ```javascript
  Array.prototype.join.call("foo", "-"); // "f-o-o"
  Array.prototype.map.call("foo", v => v.toUpperCase() + ".").join(""); // "F.O.O."
  ```

#### Numbers

- JS `number`s use the "IEEE 754" standard ("double precision"), often called
  "floating-point."
- Comparisons can fail due to imprecision. Use the `Number.EPSILON` (ES6) or
  `2^-52` to test equality.
- Safe integers (no errors): `Number.MAX_SAFE_INTEGER` or `2^53 -1`.
- Force a `number` into a 32-bit signed integer: `x | 0`. (bitwise operation).
- Special numbers:
  - `NaN`:
    ```javascript
    var a = 42 / "foo";

    a == NaN; // false
    a === NaN; // false -- comparisons with NaN always return false.

    // we can use isNaN(..), but it has gotchas:
    isNaN(a); // true
    isNaN("foo") // true

    // ES6:
    Number.isNaN(a); // true
    Number.isNan("foo"); // false
    ```
  - `Infinity` and `-Infinity`:
    ```javascript
    1 / 0; // Infinity
    -1 / 0; // -Infinity
    1e1000000; // Infinity
    2 * Number.MAX_VALUE; // Infinity
    ```
  - `-0`: happens as result of some operations, but behaves almost like `0`:
    ```javascript
    -0 === 0; // true
    (-0 === 0) && (1 / -0 === -Infinity); // true -- can be used to test for -0
    ```

#### `void` operator

- "voids" out any value: `void "foo"; // undefined`

#### Value vs. Reference

- Simple scalar primitives (`string`s, `number`s, etc.) are assigned/passed by
  value-copy.
- Compound values (`object`s, etc.) are assigned/passed by reference-copy.
- References are not like references/pointers in other languages. They point to
  the underlying values and not to the variables themselves:
  ```javascript
  var a = [1, 2, 3];
  var b = a;

  b.push(4);
  a; // [1, 2, 3, 4]

  b = [4, 5, 6];
  a; // [1, 2, 3, 4]

  b = a.slice();
  b.push(5);
  a; // [1, 2, 3, 4]
  b; // [1, 2, 3, 4, 5]
  ```
- Be careful with object wrappers:
  ```javascript
  function foo(x) {
    x++;
    console.log(x);
  }

  var a = Number(42);
  foo(a); // 43
  a; // 42
  ```
  Even though we're passing the reference, the underlying value is unboxed from
  the `Number` object in the addition operation.

### Natives

- Object sub-types/built-in objects/natives: `String()`, `Number()`,
  `Boolean()`, `Array()`, `Object()`, `Function()`, `RegExp()`, `Date()`,
  `Error()`, `Symbol()` (added in ES6).
- JavaScript automatically coerces primitives to their corresponding objects
  when accessing properties/methods:
  ```javascript
  var strPrimitive = "I am a string";
  console.log(strPrimitive.length);    // 13
  console.log(strPrimitive.charAt(3)); // "m"
  ```
- The following objects have corresponding primitives: `String`, `Number`,
  `Boolean`.
- `null` and `undefined` have no object wrapper form.
- `Object`, `Function`, `Array` and `RegExp` are object-only, though you should
  prefer their literal form.
- `Date` and `Error` can only be created through their constructed object form.
- You should never wrap primitive values manually.
- There are gotchas, like this one: `!(new Boolean(false)); // false`.
- Unboxing: `.valueOf()`.

- *Never use empty-slot arrays*. They are full ob bugs/inconsistencies
- If you want to create an array with "empty" slots, use this:
  ```javascript
  var a = Array.apply(null, {length: 3});
  ```
  It'll create an array with 3 slots filled with `undefined`, which is more
  reliable then using `Array(3)` or setting the `.length` property to 3.
- You probably should never use `Object(..)`, `Function(..)`, and `RegExp(..)`.
- Except if you want to pass flags to `RegExp(..)`, like this:
  ```javascript
  var namePattern = new RegExp( "\\b(?:" + name + ")+\\b", "ig" );
  ```

### Coercion

- ToString: `a + ""`, `String(..)`, `.toString()` or `JSON.stringify(..)`
- ToNumber: `Number(..)`
- ToBoolean: `Boolean(..)`
- "Falsy" values:
  - `undefined`
  - `null`
  - `false`
  - `+0`, `-0`, and `NaN`
  - `""`
- Due to legacy reasons, all these coerce to `true`:
  ```javascript
  var a = new Boolean(false);
  var b = new Number(0);
  var c = new String("");
  ```

#### Explicit Coercion

- **Strings <--> Numbers**: `String(42)` and `Number("42")`.
  - *Important*: Don't use the `new` keyword, to avoid creating an object.
  - Alternative ways: `(42).toString()` and `+"42"`
  - `parseInt(..)` and `parseFloat(..)` are tolerant to non-numeric characters.
  - Date --> Number: works the same way, the result is the unix timestamp.
    - A explicit approach is better: `.getTime()`. or `Date.now()` (ES5).
- **--> Boolean**: `!!x` or `Boolean(..)` (former is preferred)

#### Implicit Coercion

- **`a + b`**: If either is a string, the result is a string. Otherwise, it is
  always a numeric adition.
- **`a - 0`**: Coerces `a` to a number.
- **`if (..)`, `for (; .. ;)`, `while (..)`, `? :`** * --> Boolean.
- **Note about `||` and `&&`**: they don't return a boolean, but select one of
  the two operands values:
  - `||`: returns the first if it's true, the second otherwise.
  - `&&`: returns the first if it's false, the second otherwise.

#### Loose/Strict Equals

- Use `==` and `!=` when coercion is desired or makes no difference.
- Use `===` and `!==` when type coercion is not desired.
- Implicit coercion also happens when using `>`, `<`, `>=` and `<=`.

- **Abstract Equality rules**:
  - `string`/`number`: string is coerced to `number`.
  - */`boolean`: `boolean` is coerced to `number`.
  - `null`/`undefined`: returns `true`.
  - */`null` or */`undefined`: returns `false`.
  - `object`/non-`object`: non-`object` is to `ToPrimitive(object)`.

- *Never* compare to `== false`.
- *Bad list`:
  - `"" == 0`
  - `"" == []`
  - `0 == []`

### Grammar

- `{a: 42}` isn't an object, but a block with a labeled statement.
- The author discourages using labeled statements.
- ES6 supports object destructuring: `{a, b} = {a: 42, b: "foo"}`.
- "If the JS parser parses a line where a parser error would occur (a missing
  expected `;`), and it can reasonably insert one, it does so."
- Author: "use semicolons wherever you know they are 'required', and limit your
  assumptions about ASI to a minimum."

## Async & Performance

### Async

- JavaScript executes asynchronous code in an "Event Loop".
- Each iteration of the loop is a "tick": user interaction, IO, timers... they
  all enqueue events.
- Only one event is processed at a time.
- Sometimes "processes" should "cooperate" by breaking themselves into smaller
  chunks to allow other "processes" interleaving.
  e.g. Through `setTimeout(.., 0)`.
- JS uses **callbacks** as the "building block" of asynchrony.
  - It is a nonlinear way of thinking.
  - Leads to "trust issues": someone else is continuing your program execution.
- **Promises** (ES6):
  - They resolve those trust issues.
  - `new Promise(fn(resolve, reject))` can be used to create to create Promises
    using `resolve(..)` and `reject(..)` inside `fn`.
  - `Promise.all([..])` can be used to handle multiple Promises together. Other
    variations of Promise patterns exist.
  - `Promise.race([..])` can be used to timeout Promises.
  - `Promise.resolve(..)` wraps values and thenables.
  - `.then(..)` returns a promise: we can also chain Promises.
  - End chains with `.catch(..)` to handle errors.
- **Generators** (ES6):
  - *I haven't read this part yet.*

### Performance

- Ways to increase program performance:
  - Web Workers
  - SIMD
  - asm.js
- Benchmarking:
  - Benchmark.js is good for measuring.
  - You should always consider the context.
  - You should only test non-trivial snippets of code, not tiny optimizations.
  - [jsPerf](https://jsperf.com/) measures snippets on multiple environments.
- Micro-optmizations or engine-specific details shouldn't matter in JS.
- ES6 defines TCO (Tail Call Optimzation).
  - Optimizes the stack when a function is called at the final `return`
    statement.

## ES6 & Beyond

*I haven't read this book yet.*
