# 9 minute learn: Declaring Variables in JavaScript

This is the second part of independent posts on JavaScript syntax. In the first we investigated the differences between function declarations and expressions. During that posts we touched on a couple of aspects about the JavaScript language that we mearly glanzed over, but we'll look more into in this post; we'll do a deep dive in the differences between different ways of declaring variables.

Where in many other languages there are one way to declare variables, in JavaScript, there are three. They all serve different purposes and there may be different times you want to use them. There's not a simple rule saying use one always. These things are subjective and personal. The best way to choose one is to know what each does and make a educated decition.

In this post we'll look into the differences and we'll be left with the knowledge needed to make the choice right for our particular use case.

## Declaring or Assigning?
Before we start looking into the different keywords for creating variables in JavaScript, we need to get a couple of terms specified. What do we mean when we say declaring a variable, and what do we mean with assigning?

Declaring is allocating a space in the memory. Like clearing out a drawer so we can fit an object in it. Assigning is placing the object in the drawer. We can declare a variable without assigning it:

```js
var foo;
```

In this case we'll have a undefined variable. It's declared, but not assigned. Accessing doesn't make much sense, as it's empty. This is a case of declaring without assigning. Can we do the opposite, assigning without declaring? Yes we can*:

```js
foo = 'Hello, World!';
```

I've marked the previous sentence with an asterix. You can assign a variable without declaring it first, if you're not in strict mode. Strict mode is a particluar mode in JavaScript introduced in 2009, as they couldn't do some changes and still keep backwards compatibility. It turns out, assigning a variable without declaring it first is a bad idea. It's made global and accessible a cross scopes no matter where it's defined. This leads to high probability of collision in naming, and that when you refer to a variable it contains something else than you'd expect. And this is a bug that is difficult to track.

If you're in strict mode, the previous example is invalid syntax. This means, doing

```js
'use strict'; // activate strict mode
foo = 'Hello, World!';
```

would cause the error:

```js
ReferenceError: assignment to undeclared variable foo
```

Which is good. We catch the anti-pattern early and prevent potential bugs.

## Three ways to rule them all
Ignoring not declaring at all, there are three ways to create variables: `var` (the classic), `let` (the re-assignable) and `const` the constant.

Since the beginning of JavaScript, `var` has been the only way to declare variables. Using `var` we can declare re-assignable variables scoped by functions. We'll look into what this means later in the blogpost. We can assign all values to a `var`:

```js
var num = 42;
var str = 'String';
var bool = 1 > 3;
var fn = () => {};
```

The same is true for `let` and `const`. Although it's newer in the JavaScript language (2015 edition of the specification), we can now use them in most browsers.

```js
let num = 42;
let str = 'String';
let bool = 1 > 3;
let fn = () => {};

const num = 42;
const str = 'String';
const bool = 1 > 3;
const fn = () => {};
```

With all alternatives, we can do multiple assignings for each declaration:

```js
var num = 42, str = 'String', bool = 1 > 3;
let num = 42, str = 'String', bool = 1 > 3;
const num = 42, str = 'String', bool = 1 > 3;
```

Why did `let` and `const` get introduced in the language? What need prompted the change? Two things: block scoping and restricted reassigning.

## 360 Function Scope

The main difference between the original `var` and the new `let`/`const` is how they handle scope. Scope is, as we know the collected information we can access in a particular space in our code. When we talk about what scope a variable is in, we mean where we can access it. `var` has something we call function scope: A variable is available in the entire function it's created in or globally if it's created at the top level. Lower scopes inherit the outer scope, so this means that functions created in the scope where we can access `var` variables, can also acces the variable. Let's look at an example to make this a bit more clear:

```js
(function () {
  var num = 42;

  // Can access num.
  console.log(num); //=> 42
}());

console.log(num); //=> ReferenceError: num is not defined
```

And we can nest scope and access the outer ones:

```js
(function () {
  var num = 42;

  // Can access num.
  var myFunction = function () {
    var str = 'String';
    console.log(num);
  };
  myFunction(); //=> 42
  // Can not access str as it's defined in a lower scope
  console.log(str); //=> ReferenceError: str is not defined
}());

// Can not access str as it's defined in a lower scope
console.log(num); //=> ReferenceError: num is not defined
```

Here I use functions to create a new scope. But if you have any experience with other C-based programming languages you know any `{ }` can indicate a scope. But applying that with `var` can be surprising:

```js
for (var i = 0; i < 10; i++) {
  var num = 42 + i;
  // Can access num here:
  console.log(num);
}

// But we can also access num here, outside of the scope:
console.log(num);
```

In fact, you don't even have to visit the code for the variable to be declared:

```js
if (false) {
  var foo = 1;
}
console.log(foo); // undefined
```

As we've seen not getting `ReferenceError` here means that the variable is actually declared. It's not assign (doesn't have value), but it's definitly declared. `foo` isn't limited by the `if`-block, even if it's dead code. There're other rules in play here, but we'll get to that in the next section when we talk about hoisting.

## Conclusion
Following the style of this blog series, I won't tell you what to use. All I can tell you is what is what and how it works. The rest is up to you. You should always adapt your solutions to your problem. This means knowing what solutions there are, and what problem you face. You can't say always use `const` or never use `var`. They serve different purposes. `const` is good when you support newer syntaxes and you want to make sure you newer reassign a variable. It won't help you make immutable structures, but immutable references. `let` and `const` are still hoisted, but they force temporal dead zone and help you debug what is wrong. `let` and `const` let you limit your variables to a `block` and `var` let you think about scopes limited by function bodies.

Even though `let` and `const` is newer in the language specifications, it doesn't mean you have to use them or not use `var`. Use them where it makes sense, and where their features help you express what you want for this particular case.

## Outline
1. Several ways of defining variables
  1. var
  2. let
  3. const
2. Assign vs Declare
2. Multiple assigings. (assign vs declare)


1. Function scope vs Block scope
2. Reassigning (Assign vs immutability)
4. Hoisting and Temporal Dead Zones