# The ways of creating functions in JavaScript
If you've ever seen or written JavaScript, you might have noticed there are two ways of defining functions. Even people with more experience with the language can find difficult to decipher the differences. In this, first part of a blog series where we look at JavaScript syntax, we'll deep dive into the differences between function declarations and function expressions. This post will not cover the differences between different types of functions (arrow functions, async functions, normal functions, etc), but the way we define them.

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

We see that one thing that separates them is the variable declaration and the lack of an identifier name. But we'll dig deeper than this and see what the difference is, both in grammar and usage.

## FunctionDeclarations
One way to define functions in JavaScript is in the top level or in a block, starting with the keyword `function`. It's defined as a part of the [grammar of JavaScript](https://www.ecma-international.org/ecma-262/8.0/#prod-StatementListItem). Grammar is what defines what words are legal in what position in languages.

A function declaration is of type `FunctionDeclaration` (analogous to "part of speech" tagging, as noun, verb, etc, in natural language), which is by grammar allowed to be child of `StatementListItem` which is a part of `StatementList`, which is a child of `Block`, etc, etc. It's all technical and wordy. But you can think of it like, if the line starts with the keyword `function` inside a block `{}` or at the top of your program, you probably have a function declaration.

In the grammar, it's also said that a `FunctionDeclaration` must have a name. Or the grammatical term `Identifier`, if you want. The exception is when doing `export default` of a function. But everywhere else, you have to name your function declarations.

Following those rules, we can say:

```js
function myDeclaredFunction () { } // This is a function declaration

function myDeclaredFunction () { // block
  function myDeclaredFunction () { } // This is a function declaration
} // This is a function declaration

export default function () { } // This is a function declaration (without identifier)

function () { } // This is *NOT* a valid function declaration. Nor valid JavaScript

let func = function () { } // This is *NOT* a function declaration
```

Specifying by the grammar may seem complicated, and it's. But trust me, for the most cases you intuitively see by glancing over the code what is a declaration. In most cases, if the line starts with `function` and not wrapped in parenthesis, it's a function declaration.

### Function Expressions
If we know that function declarations are functions defined in `StatementListItem`s starting with the keyword `function`, we know that all other functions are expressions. Functions defined in `let`, `var` and `const` bindings are expressions. Functions defined as arguments to functions are expressions. Functions defined as values on properties in objects are expressions, and in fact, functions defined in the class syntactic sugar are also expressions.

We saw that function declarations almost in all cases needs have an identifier (except with default export). Is the same true for expressions? No. Function expressions _can_ be anonymous (see section `Anonymity and Naming of Expressions`). We've already seen an example of a function expression in the previous section, but we can list plenty more examples:

```js
let foo = function () {};
const bar = function name () {}; // Same as above, but named

// also arrow functions are function expression
const baz = () => {};

(function () { }); // This is a function expression, as it's wrapped in parenthesis. Making it an expression.

setTimeout(
  function () { } // This is a function expression
);
// Same as above. Not in StatementListItem starting with keyword function,
// but expression, as argument to the function setTimeout.

const obj = {
  foo () { } // function expression. Same as:
  bar: function () { } // which might be clearer
};

class MyClass {
  foo () { } // Also function expression.
}

// Classes may be surprising, but it's easier to see if we desugar it:

function MyClass () { }
// Here we see more explicitly that it's in the expression position.
Foo.prototype.foo = function foo () { };
```

Summarized, we can say that in the positions where you can have values like strings and numbers, you have a function expression.

We've seen that grammar specifies if whether we have a declaration or an expression. But are there any differences? And does it actually matter? As it turns out, in some cases it does matter. Let's dig further.

## Declarations and Hoisting
The most notable difference, and the difference that can affect you the most, is hoisting. Function declarations are hoisted to the top of the top-level or if you have a function declaration inside a function, to the top of that function. In any case, with function declarations, you can use them above where they are declared.

```js
console.log(add(40, 2)); // Totally legal and valid javascript.

function add(a, b) {
  return a + b;
}
```

This is practically the same as:

```js
function add(a, b) { // "hoisted" to the top in run-time
  return a + b;
}

console.log(add(40, 2));
```

Hoisting is a common thing in JavaScript, and all variable declarations are also hoisted, as we'll see soon. You might ask yourself why anyone would do this. And there can be multiple reasons. Most, like often in programming, subjective. There's been a common practice in JavaScript, for a long time, to prioritize code by importance. To the more central the code is, the higher it goes. An effect of that is that the function which describes the module the most is visible as you open a file. This relies heavily on hoisting, as all helper functions would be defined further down in the file. This practice is fading away more and more, but there are still developers preferring this approach.

Hoisting will also occur when doing variable declarations with a function expression. We won't go into all details here, but all variable declarations in JavaScript are hoisted, but the different being, variables aren't assigned. So in contrast to the previous example, this wouldn't work:

```js
console.log(add(40, 2)); // Would cause a ReferenceError. We declare `add`, but it's undefined.
// Extra tidbit: The area between the top of the scope and to where
// we assign the variable is called Temporal Dead Zone.

const add = function add(a, b) {
  return a + b;
};
```

## Anonymity and Naming of Expressions

As we've seen, function declarations can't be anonymous by grammar. But expressions can. And some time that can be convenient where we  use them as one-offs as an argument to functions like `map` or `filter`:

```js
const events = [1, 2, 3, 4].filter(function (i) { // anonymous function expression
  return i % 2 === 0;
});

// or as some prefer it, using ArrowFunctions
const events = [1, 2, 3, 4].filter(i => i % 2 === 0);
```

But as with other conveniences, there can be a thing as too much. Anonymity can lead to harder debugging, as it obfuscates our stack traces. Nested anonymous functions can be hard to navigate through.

We can see the output of a stack trace with no names by throwing an error from within an anonymous function:

```js
try {
  (function() {
    throw new Error('Error');
  })();
} catch (e) {
  console.log(e.stack);
}
```

Will lead to something like:

```
@file:///test.js:3:11
@file:///test.js:2:4
test.js:6:3
```

As you see. It still has line and column info, but we know with source-maps and transpiling, source location isn't always trustworthy. With named functions:

```js
try {
  (function myFunction () {
    throw new Error('Error');
  })();
} catch (e) {
  console.log(e.stack);
}
```

...we get:

```
myFunction@file:///test.js:3:11
@file:///test.js:2:13
```

Now, in later versions of JavaScript, functions can infer names from variable names:

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

A couple of things to note in the example. We see functions in JavaScript are what we call first class entities. And this is something we've seen all through the examples with function expressions. We treat functions as values. In JavaScript functions are objects (but a special kind called callable objects) and as such they have properties. One of these properties is `.name`. A getter for reflecting the name of the function. Engines use `.name` in stack traces of errors, but we can access it directly.

We also see that naming functions as identifier to function expressions, takes precedence to the inference. But inferring name is totally valid. How ever, we aren't able to infer the name when passing anonymous functions as arguments. Which means we still have to think about anonymity and debugging. This is most notable in recent trends with using arrow functions everywhere. As arrow functions are always anonymous, unless names are inferred.

## Recursion and Referring Inferred Functions

While debugging is helped by inferring function names, there is a difference between inferring and proper naming. It might be more abstract, but it can in some cases create actual bugs, and be hard to smoke out. It's when using recursive functions. We can summarise recursive functions as functions invoking them self.

Recursion can be a valuable tool in your programming toolchain. Dividing a problem up to subsets and solving one part at the time in a potentially infinite scale. But inferred names with function expressions can lead to problems: The variables can be overwritten from outside of the function it self:

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

This is as the fibonacci identifier for the function expression in the latter example shadows the variable from outside, and refers to the function, not the variable specified in the outer scope. As such it's never overwritten.

This might seem contrived. But when using recursion this can actually be a legit bug. And if it happens, it may be hard to investigate.

## Conclusion

We've now seen that there are two ways of assigning functions. We have declarations, which is hoisted with contents, and we have expressions which is not hoisted with contents, but the reference to it may be hoisted if we assign it as a variable. Expressions can be anonymous, they can be named or they can be implicitly named from the variable name. Not naming functions can lead to harder debugging as it will obfuscate stack traces.

We've also seen that using variable names for function expressions is different than using properly named functions. E.g in recursive functions, we can end up referring to the wrong value.

You can use both function declarations and expression, it depends on style and preference. Many like the hoisting ability of function declarations, where you can have more "relevant" functions at the top (visible when you first open the file) and helper functions further down, but other prefer reading sequentially from top to bottom. This is personal and subjective, but now you know the difference and the trade-offs and can be able to make a decision that suits you the best.
