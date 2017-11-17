# The ways of creating functions in JavaScript
If you've ever seen or written JavaScript, you might have noticed there are two ways of defining functions. Even for people with more experience with the language can find difficult to decipher the differences. In this first part of the blog series where we look a JavaScript syntax, we'll dive deep into the differences between function declarations and function expressions. This post will not cover the differences between different types of functions (arrow functions and normal functions), but just the way we define them.

## Two ways
The two ways we have to define functions in JavaScript is declarations and as expressions:

```js
function myDeclaredFunction () {
  console.log('This is a function declaration');
}
```

and

```js
const myFunctionExpression = function () {
  console.log('This is a function expression');
};
```

We see that one thing that separates them is the variable declaration and the lack of a variable. But we'll dig deeper than this and see what the difference is, both in grammar and usage.

## FunctionDeclarations
One way to define functions in JavaScript is in the top level or in a block, starting with the keyword `function`. This is defined as a part of the [grammar of JavaScript](https://www.ecma-international.org/ecma-262/8.0/#prod-StatementListItem). A function declaration is of type `FunctionDeclaration` (similar to "part of speech" tagging, as noun, verb, etc, in natural language), which is by grammar only allowed to be child of `StatementListItem` which is a part of `StatementList`, which is a child of `Block`, etc, etc. It's all very technical and wordy. But you can think of it like, if the line starts with the keyword `function` inside a block `{}` or at the top of your program, you probably have a function declaration.

In the grammar, it's also said that a `FunctionDeclaration` has to have a name. Or the grammatical term `Identifier`, if you want. The only place where a function declaration can be without identifier, is if it is used in a `export default` setting. In that case you can have a function declaration without an identifier. But everywhere else, your function has to be named.

Following those rules, we can say:

```js
function myDeclaredFunction () { } // This is a function declaration

function myDeclaredFunction () {
  function myDeclaredFunction () { } // This is a function declaration
} // This is a function declaration

export default function () { } // This is a function declaration

function () { } // This is *NOT* a valid function declaration. Nor valid JavaScript

let func = function () { } // This is *NOT* a function declaration
```

Specifying by the grammar may seem very complicated, and it is. But trust me, for the most cases you intuitively see just by glancing over the code what is a declaration. In most cases, just if the line starts with `function` and not wrapped in parenthesis, it is a function declaration.

### Function Expressions
If we know that function declarations are functions defined in `StatementListItem`s starting with the keyword `function`, we know that all other functions are expressions. So functions defined in `let`, `var` and `const` bindings are expressions. Functions defined as arguments to functions are expressions, and functions defined as values on properties in objects are expressions, and in fact, functions defined in the class sugar are also expressions.

We saw that function declarations almost in all cases needs have an identifier (except with default export). Is the same true for expressions? No. Function expressions can be anonymous. More on this in a later section (`Anonymity and Naming of Expressions`). We've already seen an example of a function expression in the previous section, but we can see plenty more examples:

```js

let foo = function () {};
const bar = function name () {};

// also arrow functions are function expression
const baz = () => {};

(function () { }) // This is a function expression, as it is wrapped in parenthesis. Making it an expression.

setTimeout(
  function () { } // This is *NOT* a function declaration
);
// Same as above. Not in StatementListItem starting with keyword function, but expression, as argument to the function setTimeout.

const obj = {
  foo () { } // function expression. Same as:
  bar: function () { } // which is clearer
};

class MyClass {
  foo () { } // Also function expression.
}

// More visible when we think about classes being syntactic sugar for:

function MyClass () { }
Foo.prototype.foo = function foo () { };
// Here we see more clearly that it is in the expression position.
```

Summarized, we can say that in the positions where you can have values like strings and numbers, you have a function expression.

We've seen that grammer specifies if whether we have a declaration or an expression. But does it really matter? Are there any differenes? As it turns out, there are a few. Let's dig further.

## Declarations and Hoisting

The most notable difference, and the difference that can affect you the most, is hoisting. Function declarations are hoisted to the top of the top-level or if you have a function declaration inside a function, to the top of that function. In any case, with function declarations, you can use them above where they are declared.

```js
console.log(add(40, 2)); // Totally legal and valid javascript.

function add(a, b) {
  return a + b;
}
```

This is practially the same as:

```js
function add(a, b) { // "hoisted" to the top in run-time
  return a + b;
}

console.log(add(40, 2));
```

Hoisting is a common thing in JavaScript, and all variable declarations are also hoisted, as we'll see soon. You might ask yourself why anyone would do this. And there's can be several reasons. Most, like often in programming, subjective. There's been a common practise in JavaScript, for a long time, to prioritize code by importance. To the more central the code is, the higher it goes. An effect of that is that the function which describes the module the most is visible as you open a file. This relies heavily on hoisting, as all helper functions would be defined further down in the file. This practise is fading away more and more, but there are still developers preferring this approach.

Hoisting will also occur when doing variable declarations with a function expression. We won't go into all details here, but all variable declarations in JavaScript are hoisted, but the different being, variables aren't assigned. So in contrast to the previous exmple, this wouldn't work:

```js
console.log(add(40, 2)); // Would cause a ReferenceError. add is declared, but is undefined.
// Extra tidbit: The area between the top of the scope and to where the variable is assigned
// is called Temporal Dead Zone.

const add = function add(a, b) {
  return a + b;
};
```

## Anonymity and Naming of Expressions

As we've seen, function declarations can't be anonymous by grammer. But expressions can. And some time that can be convenient where we just use them as one-offs as an argument to functions like `map` or `filter`:

```js
const events = [1, 2, 3, 4].filter(function (i) { // anonymous function expression
  return i % 2 === 0;
});
```

But as with many things, too much is bad. Anonymity can lead to harder debugging, as this causes our stack traces to be obfuscated. Multiple nested anonymous functions can be very hard to navigate through.

In later versions of JavaScript, functions can infer names from variable names:

```js
let myInferredFunction = function () { };
console.log(myInferredFunction.name); // => myInferredFunction

// Also true for arrow functions
let myInferredFunction = () => { };
console.log(myInferredFunction.name); // => myInferredFunction

// Actual function name take precedence
const myInferredFunction = function namedFunction () { };
console.log(myInferredFunction.name); // => namedFunction
```

A couple of things to note in the example. We see functions in JavaScript are what we call first class entities. And this is something we've seen all through the examples with function expressions. We treat functions as values. In JavaScript functions are just objects (but a special kind called callable objects) and as such they have properties. One of these properties is `.name`. Automatically populated with the name of the function. This is used in stack traces in errors, and we can access it directly.

We also see that naming functions as identifier to function expressions, takes precedence to the inferrence. But inferring name is totally valid. How ever, we aren't able to infer the name when passing anonymous functions as arguments. Which means we still have to think about anonymity and debugging. This is most notable in recent trends with using arrow functions everywhere. As arrow functions are always anonymous, unless names can be inferred.

## Recursion and Referring Variable Named Functions

While debugging is helped by inferring function names, there is a difference between inferring and proper naming. It might be more abstract, but it can in some cases create actual bugs, and be very hard to smoke out. It's in regard to recursive functions. Recursive functions can be summerized as functions invoking them self.

Recursion can be a very valuable tool in your programming toolchain. Dividing a problem up to subset and solving one part of it at the time in a potential infinite scale. But inferred names with function expressions can lead to problems: The variables referrning to the functions can be overwritten from outside of the function it self:

```js
let fibonacci = function (num) {
  if (num <= 1) return 1;
  return fibonacci(num - 1) + fibonacci(num - 2);
}
var fibCopy = fibonacci;
fibonacci = function () {
  throw new Error('No, way!');
}
console.log(fibCopy(3));
// => Error: No, way!
```

But if we how ever do named function expressions (or function declarations) this still works as expected:

```js
let fibonacci = function fibonacci (num) {
  if (num <= 1) return 1;
  return fibonacci(num - 1) + fibonacci(num - 2);
}
var fibCopy = fibonacci;
fibonacci = function () {
  throw new Error('No, way!');
}
console.log(fibCopy(3));
// => 3
// (Works as expected)
```

This is as the fibonacci identifier for the function expression in the latter example shadows the variable from outside, and refers to the function, not the variable specified in the outer scope. As such it is never overwritten.

This might seem contrived and like a non-issue. But when using recursion this can actually be a legit issue to come across. And if it happens, it may be very hard to debug.

## Conclusion

We've now seen that there are two ways of assigning functions. We have declarations, which is hoisted with contents, and we have expressions which is not hoisted with contents, but the reference to it may be hoisted if we assign it as a variable. Expressions can be anonymous, they can be named or they can be implicitly named from the variable name. Not naming functions can lead to harder debugging as it will obfuscate stack traces.

We've also seen that using variable names for function expressions is different than using properly named functions. E.g in recursive functions, we can end up referring to the wrong value.

You can use both function declarations and expression, it depends on style and preference. Many like the hoisting ability of function declarations, where you can have more "sentral" functions at the top (visible when you first open the file) and helper functions further down, but other prefer reading sequentially from top to bottom. This is personal and subjective, but now you know the difference and the trade-offs and can be able to make a decision that suits you the best.
