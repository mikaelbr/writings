# 9 minute learn: Declaring Variables in JavaScript

This is the second part of independent posts on JavaScript syntax. In the first, we investigated the differences between function declarations and expressions. During that posts, we touched on a couple of aspects about the JavaScript language that we merely glazed over, but we'll look more into in this post; we'll do a deep dive in the differences between different ways of declaring variables.

Where in many other languages there is one way to declare variables, in JavaScript, there are three. They all serve different purposes and there may be different times you want to use them. There's not a simple rule saying use one always. These things are subjective and personal. The best way to choose one is to know what each does and make an educated decision.

In this post, we'll look into the differences and we'll be left with the knowledge needed to make the choice right for our particular use case.

## Declaring or Assigning?
Before we start looking into the different keywords for creating variables in JavaScript, we need to get a couple of terms specified. What do we mean when we say declaring a variable, and what do we mean by assigning?

Declaring is allocating a space in the memory. Like clearing out a drawer so we can fit an object in it. Assigning is placing the object in the drawer. We can declare a variable without assigning it:

```js
var foo;
```

In this case, we'll have an undefined variable. It's declared, but not assigned. Accessing doesn't make much sense, as it's empty. This is a case of declaring without assigning. Can we do the opposite, assigning without declaring? Yes, we can*:

```js
foo = 'Hello, World!';
```

I've marked the previous sentence with an asterisk. You can assign a variable without declaring it first if you're not in strict mode. Strict mode is a particular mode in JavaScript introduced in 2009, as they couldn't do some changes and still keep backward compatibility. It turns out, assigning a variable without declaring it first is a bad idea. It's made global and accessible across scopes no matter where it's defined. This leads to the high probability of collision in naming, and that when you refer to a variable it contains something else than you'd expect. And this is a bug that is difficult to track.

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

With `var`s we can do multiple declarations of the same variable, but not with `let`/`const`:

```js
var foo = 42;
var foo = 'String';
console.log(foo); //=> 'String'


// But doesn't work:
let foo = 42;
let foo = 'String';
// SyntaxError: Identifier 'foo' has already been declared
```

Why did `let` and `const` get introduced in the language? What need prompted the change? Two things: block scoping and restricted reassigning.

## 360 Function Scope

The main difference between the original `var` and the new `let`/`const` is how they handle scope. The scope is, as we know the collected information we can access in a particular space in our code. When we talk about what scope a variable is in, we mean where we can access it. `var` has something we call function scope: A variable is available in the entire function it's created in or globally if it's created at the top level. Lower scopes inherit the outer scope, so this means that functions created in the scope where we can access `var` variables, can also access the variable. Let's look at an example to make this a bit more clear:

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

As we've seen not getting `ReferenceError` here means that the variable is actually declared. It does not assign (doesn't have value), but it's definitely declared. `foo` isn't limited by the `if`-block, even if it's dead code. There're other rules in play here, but we'll get to that in the next section when we talk about hoisting.

This concept of functional scopes instead of block scopes can be alienating if you come to JavaScript from another language, expecting it to have block scope. Well, now it does. With `let` and `const`, you can constrict scopes by blocks in JavaScript:

```js
for (var i = 0; i < 10; i++) {
  let num = 42 + i;
  // Can access num here:
  console.log(num);
}

// Cannot access num here
console.log(num);
// ReferenceError: num is not defined
```

The same goes for `const`. Both `let` and `const` have block scopes, while `var` still has function scope. This is important for JavaScript. The ECMAScript specification (the standard which JavaScript is an implementation of), works pretty hard on being backward compatible. `let` was introduced as an alternative to `var` but with block scope. They couldn't change the semantics of `var` as that would have crashed *a lot* of websites out there. You could have introduced a new tag such as `"use strict"` but that would further fragment code and it's easier to introduce a new keyword as a syntax extension.

If `let` is `var` with block scope, what is `const`? `const` is like `let` but without ability of reassigning:

```js
let num = 42;
num = 52; // Totally valid

// Not as valid:

const str = 'String';
str = 'Another string';
// TypeError: invalid assignment to const `str'
```

This also means you can't declare a `const` variable without assigning it:

```js
const str;
// SyntaxError: missing = in const declaration
```

I use the word reassign explicitly here. You might have heard it spoken of as a constant, but that really makes the wrong associations. `const` doesn't guarantee your values to remain the same (to be unchangeable or immutable). The declaration only set the semantics of the variable, not the value the variable refers to. When you have primitive data types like numbers and strings, using `const` practically means that you always have a reference to those values, as they cannot be changed in JavaScript. But that's not true for, say, Objects:

```js

const obj = { num: 42 };

obj = { num: 52 };
// TypeError: invalid assignment to const `obj'

// But I can do this:
obj.num = 52;
```

In the example above we see how `const` prevents me from reassigning the variable. But it doesn't change the semantics of what it refers to. JavaScript Objects are changeable by default and using `const` doesn't change that. But it does prevent you from accidentally reassigning.

## To Hoist or Dead Zone?

Is really the block vs function scope the only difference? No. There is one more theoretical difference. And I say theoretically here, as I'd argue in practice, it doesn't really matter. Here I'm talking about something in JavaScript called hoisting. Hoisting is a concept that might be foreign, but it has been with JavaScript from the beginning. It takes all declared variables (or function declarations) and moves them to happen at the top of the scope in runtime. For variables, this happens only for the declarations (see section on declarations vs assignments), not assignment. We don't move the values to the top of the scope.

```js
(function () {
  console.log(foo); //=> undefined
  var foo = 'Hello, World';
}());
```

will essentially be the same as

```js
(function () {
  var foo;
  console.log(foo); //=> undefined
  foo = 'Hello, World';
}());
```

Implicitly. When running the code. This is why it's long been practiced in JavaScript, to explicitly move all declarations to the top of your scope (like the latter example). This way you won't be surprised when you can refer to a variable before it's defined. With `let` and `const` this behaves a bit differently, however. Trying the same in a block:

```js
// Empty block statement (causing a scope for let)
{
  console.log(foo);
  // ReferenceError: foo is not defined
  let foo = 'Hello, World!';
}
```

Here we see we don't get undefined, but a ReferenceError. But if we dig further, I think `let`s and `const`s are technically still hoisted to the top of the scope:

```js
let foo = 'Bye, World!';
{
  console.log(foo);
  // ReferenceError: foo is not defined
  let foo = 'Hello, World!';
}
```

This example you might expect to work, but it actually produces the same output as the previous one. It seems the declaration is still kind of hoisted to the top of the scope, but you are in a temporal state where you aren't allowed to refer to the variable without causing a `ReferenceError`. This temporary state is referred to as the `Temporal Dead Zone` (TDZ): The area between the top of the scope and where the value is assigned.

There are a couple of cases where using `var` and `let`/`const` may differ due to the TDZ. Consider this case:

```js
function logList(l) {
  for (var l of l) console.log(l);
}

logList([1, 2, 3]);
// Outputs:
// => 1
// => 2
// => 3
```

It's not good naming, but it works. The `l` from the `for-of` loop shadows the parameter list. The same when using `let` or `const` is a different story:

```js
function logList(l) {
  for (var l of l) console.log(l);
}

logList([1, 2, 3]);
// ReferenceError: l is not defined
```

As I can see, there are a couple of rules in play here. `var`s can be declared multiple times, and the TDZ. Take the first example. It's essentially translated to something similar to

```js
function logList(l) {
  var l;
  l = l[0];
  console.log(l);
  l = l[1];
  console.log(l);
  l = l[2];
  console.log(l);
}
```

Which doesn't cause any problems as we can do multiple declarations without changing the contents of a variable when using `var`:

```js
function log (foo) {
  var foo;
  var foo;
  var foo;
  console.log(foo);
}
log('Hello');
//=> 'Hello'

// Same goes on top level

var bar = 'Hello';
var bar;
var bar;
console.log(bar);
//=> 'Hello'
```

The declaration is hoisted to the top anyways, so doesn't matter. With `let`, it's translated to roughly the same:

```js
function logList(l) {
  let l = l[0];
  console.log(l);
  l = l[1];
  console.log(l);
  l = l[2];
  console.log(l);
}
```

And now we know referring to a variable between the declaration and the top of the scope causes a `ReferenceError` so the initial `l[0]` doesn't work.

## Conclusion
Following the style of this blog series, I won't tell you what to use. All I can tell you is what is what and how it works. The rest is up to you. You should always adapt your solutions to your problem. This means knowing what solutions there are, and what problem you face. You can't say always use `const` or never use `var`. They serve different purposes. `const` is good when you support newer syntaxes and you want to make sure you newer reassign a variable. It won't help you make immutable structures, but immutable references. `let` and `const` are still hoisted, but they force temporal dead zone and help you debug what is wrong. `let` and `const` let you limit your variables to a `block` and `var` let you think about scopes limited by function bodies.

Even though `let` and `const` is newer in the language specifications, it doesn't mean you have to use them all the time and not use `var`. Use them where it makes sense, and where their features help you express what you want for a given case.