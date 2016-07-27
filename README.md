# Introduction to ES6

## Objectives

1. Describe the major new features in ES6
2. Explain how `let` and `const` differ from `var`
3. Describe how to use arrow functions
4. Explain the value of the spread operator

## Introduction

Up to now, you've been working in the browser and on the server with the version of JavaScript known as ECMAScript 5. ECMAScript is an evolving standard, the complete history of which we don't have time to cover here.

But you've probably also heard talk about this thing called ECMAScript 6, ES6, or ES2015 (they kinda missed the deadline on that one). [ES6](http://es6-features.org/) is the next specification for JavaScript, and it's finally started to appear in a major way in browsers and on servers thanks to Node.js 5.

You can get the complete rundown of ES6 features in Node.js [here](https://nodejs.org/en/docs/es6/); but we'll also run through the features that you might see in the upcoming labs here. For our purposes, anything that can be enabled with a simple `"strict mode"` declaration is fair game — but we won't teach you about the stuff behind the `--es_staging` or `--harmony` flag just yet.

## Aside: Strict Mode

You might not have encountered it yet (at least knowingly), but ES5 has a way to opt in to a special version of JavaScript called [_strict mode_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode).

You can read about the ins and outs of strict mode at the link above — generally, it turns silent failures into thrown errors, and it helps prevent variables sneaking into the global scope.

That is, in standard mode, the following code would run — it just wouldn't do anything:

```javascript
delete Object.prototype;
```

But in strict mode, this would throw a `TypeError`:

```javascript
"use strict";

delete Object.prototype;
```

Strict mode can be invoked by writing

```javascript
'use strict';
```

at the top of the current script — strict mode then applies to the entire script. Alternatively, you can apply strict mode to individual functions:

```javascript
function strictFunction() {
  "use strict"

  // this will throw an error in strict mode
  delete Object.prototype
}

function standardFunction() {
  // this will silently fail in standard mode
  delete Object.prototype
}
```

Strict mode does just as its name implies: it enforces _stricter_ rules on the execution of your code. Note that some transpilers (like [babel](http://babeljs.io/)) can set strict mode for you.

Strict mode also enables some newer features that ES6 developers wanted to make sure users explicitly opted in to.

## Node.js ES6 Features

Remember, read the [docs](https://nodejs.org/en/docs/es6/) for the complete low-down on ES6 in Node.js. We'll cover a subset of the new features below.

* [`const` and `let`](#block-scoping)
* [Classes](#classes-use-strict)
* [Arrow Functions](#arrow-functions)
* [Promises](#promises)
* [Object Literal Extensions](#object-literal-extensions)
* [Spread Operator](#spread-operator)
* [Template Strings](#template-strings)
* [Destructuring](#destructuring)

### Block Scoping

#### `let` ("use strict")

The keyword [`let`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let) is a new way of declaring local variables. How does it differ from good ol' `var`? Variables declared with `let` have block-level scope:

```javascript
// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Scoping_rules
function letTest() {
  let x = 31;
  if (true) {
    let x = 71;  // different variable
    console.log(x);  // 71
  }
  console.log(x);  // 31
}
```

Notice how `x` declared outside of the `if` block differs from the `x` declared inside the block. Block-level scope means that the variable is available only in the block (`if`, `for`, `while`, etc.) in which it is defined; it differs from JavaScript's normal function-level scope, which restricts the variable to the function in which it is defined (or `global`/`window` if it's a global variable).

#### `const`

The keyword [`const`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const) does not require strict mode. Like `let`, it declares a variable with block-level scope; additionally, it prevents its variable identifier from being reassigned.

That means that the following will throw an error:

```javascript
const myVariable = 1;

myVariable = 2; // syntax error
```

However, this does not mean that the variable's value is immutable — the value can still change.

```javascript
const myObject = {};

// this works
myObject.myProperty = 1;

// 1
console.log(myObject.myProperty)
```

### Classes ("use strict")

"Wait," you say. "JavaScript has prototypal, not class-based, inheritance."

You're still right. But [classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) in JavaScript are awesome, and you'll be seeing them increasingly as ES6 adoption increases.

Consider the following simple example, based largely on the examples from MDN. We want to create a `Polygon` prototype and inherit from it. We'll start with ES5:

```javascript
function Polygon(height, width) {
  this.height = height;
  this.width = width;
}
```

Cool, so we got to the ES6 class `constructor`, which works just like a constructor in ES5. But now how do we implement the `area()` getter? Well, not very nicely — let's back up and rewrite what we just wrote:

```javascript
function Polygon(height, width) {
  this.height = height;
  this.width = width;

  // :(
  this.area = this.calcArea();
}

Polygon.prototype.calcArea = function() {
  return this.height * this.width;
};

const rectangle = new Polygon(10, 5);

console.log(rectangle.area);
```

Okay, so that worked, but look at how difficult it is to reason about. We have to plan in advance for the properties that we want to set, and `area` is not calculated dynamically — it's set when the `Polygon` is instantiated and then forgotten about, so if somehow a `Polygon`'s `height` and `width` changed, its `area` would need to be updated separately. Gross.

Moreover, extending this ES5 `Polygon` is a bit onerous:

```javascript
function Square(sideLength) {
  Polygon.call(this, sideLength, sideLength);
}

Square.prototype = new Polygon();

const square = new Square(5);

// Polygon { height: 5, width: 5, area: 25 }
console.log(square);
```

Well, that seems like it just about works. But what if we check the variable `square`'s constructor?

```javascript
// [Function: Polygon]
square.constuctor;
```

That's not right. It should be `[Function: Square]`. We can fix it, though:

```javascript
Square.prototype.constructor = Square;

const square2 = new Square(6);

// { [Function: Square] constructor: [Circular] }
square2.constructor
```

Eh, close enough?

(Note: The point here isn't to land on a perfect approach to object inheritance in JavaScript, it's to show that such a goal isn't feasible and won't be achieved in a nice way.)

Now let's try with ES6:

```javascript
class Polygon {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }

  // whaaaaat -- getters!
  get area() {
    return this.calcArea();
  }

  calcArea() {
    return this.height * this.width;
  }
}

const rectangle = new Polygon(10, 5);

console.log(rectangle.area);
```

Let's extend it:

```javascript
class Square extends Polygon {
  constructor(sideLength) {
    super(sideLength, sideLength)
  }
}

const mySquare = new Square(5);

// Square { height: 5, width: 5 }
mySquare;

// [Function: Square]
mySquare.constructor;

// 25
mySquare.area;
```

Whoa. That was easy.

![that was easy](http://i.giphy.com/zcCGBRQshGdt6.gif)

### Arrow functions

[Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) provide not only a terser way to define a function but also _lexically bind the current `this` value_. This ain't your grandpa's JS.

```javascript
const greet = (greeting, person) => {
  return greeting + ', ' + person + '!';
};

// 'Hello, Marv'
greet('Hello', 'Marv');

var a = [
  'Hydrogen',
  'Helium',
  'Lithium',
  'Beryl­lium'
];

// compare this implementation...
var a2 = a.map(function(s){ return s.length });

// ... to this implementation with the fat arrow
var a3 = a.map(s => s.length);
```

Fat arrows also have implicit returns — the following are equivalent:

```javascript
var a3 = a.map(s => s.length);
var a4 = a.map(s => {
  return s.length;
});
```

If the function only accepts one argument, parentheses are optional:

```javascript
// this...
var a3 = a.map(s => s.length);

// ... is the same as this
var a3 = a.map((s) => s.length);
```

If there are zero or two or more arguments, though, you must use parens:

```javascript
var evens = [1, 2, 3, 4].reduce((memo, i) => {
  if (i % 2 === 0) {
    memo.push(i)
  }

  return memo;
}, []);
```

### Promises

[Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) offer a new way of handling asynchronicity.

We'll cover them in greater detail in a later lesson, but for now know that `Promise` is available globally in Node.js.

Also know that it's awesome.

```javascript
const promise = new Promise((resolve, reject) => {
  return someIntenseTask().then(result => {
    if (result.success) {
      return resolve(result)
    }

    return reject(result.error)
  })
})

promise.then(result => {
  return doSomething(result);
}).catch(error => handleError(error))
```

### Object literal extensions

ES6 gives us a number of handy [new ways to deal with objects](https://github.com/lukehoban/es6features#enhanced-object-literals). They're features that you either wish JavaScript had, or ones you didn't know you needed.

```javascript
const prop = function() {
  return "I'm a prop!";
}

const myObj = {
  // computed (dynamic) property names
  ['foo' + 'bar']: 'something',

  // methods
  shout() {
    return 'AH!'
  },

  // short for `prop: prop`
  prop
}

// 'something'
myObj.foobar

// "I'm a prop!"
myObj.prop()

// 'AH!'
myObj.shout()
```

### Spread operator

The [spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator) — `...` — is unassuming but incredibly powerful.

We can use it for arrays:

```javascript
const a = [1, 2, 3]
const b = [0, ...a, 4, 5]

// [0, 1, 2, 3, 4, 5]
b
```

functions:

```javascript
function printArgs() {
  // recall that every function gets an `arguments`
  // object
  console.log(arguments);
}

// using `a` from above
// { '0': 1, '1': 2, '2': 3 }
printArgs(...a);
```


### Template Strings

[Template strings](https://nodejs.org/en/docs/es6/) in ES6 are most commonly used for string interpolation. Instead of writing:

```javascript
var foo = 'bar';
var sentence = 'I went to the ' + foo + ' after working in ES5 for too long.';
```

we can now write:

```javascript
var foo = 'bar';
var sentence = `I went to the ${foo} after working in ES5 for too long.`;
```

and we'll get the same result.

You can also use _tagged template literals_ to perform more advanced manipulation:

A _tag_ is simply a function whose first argument is an array of strings and whose subsequent arguments are the values of the substitution expressions (the things in `${}`).

Here's the example from [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#Tagged_template_literals):

```javascript
var a = 5;
var b = 10;

function tag(strings, ...values) {
  console.log(strings[0]); // "Hello "
  console.log(strings[1]); // " world "
  console.log(values[0]);  // 15
  console.log(values[1]);  // 50

  return "Bazinga!";
}

tag`Hello ${ a + b } world ${ a * b }`;
// "Bazinga!"
```

### Destructuring

Destructuring makes it easier than ever to pull values out of objects and arrays and store them in variables. We destructure an array by putting our new variable names at the corresponding index and an object by giving our variable the same name as the key we are interested in.

```js
const [a, b] = [1, 2];
// a === 1 && b === 2

const { a, b } = { a: 1, b: 2 }
// a === 1 && b === 2
```

To see what other amazing things we can to with destructuring, check out the [docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment).

## Conclusion

There are _tons_ of new features in ES6, and we don't have time to cover them all here. Check out [the docs](https://nodejs.org/en/docs/es6/), play around in console, and have fun!

## Resources

- [let](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let
- [const](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const
- [classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes
- [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
- [Enhanced object literals](https://github.com/lukehoban/es6features#enhanced-object-literals): https://github.com/lukehoban/es6features#enhanced-object-literals
- [Template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals
- [Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions
