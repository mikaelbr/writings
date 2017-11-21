# Function Parameters in Detail

This is the third part of independent blog posts on JavaScript syntax in detail. This time we'll look at function arguments. You might think that this is a small topic, but there's surprisingly lots to say about it. In this post we'll cover everything from function overloading to default arguments. Everything you need to know about function arguments and then some.

## Argument or Parameter?

First let's get some terms out of the way. What is actually the difference between arguments and parameters? Often these terms are used interchangably. Really there's no need to destinguish between them, but we'll cover it here for completness. As with all terms it may help you relay information in a more consise manner, given that the person you are talking to also have the same understanding of the terms.

An argument is something you pass into a function, while parameters is something a function has. When invoking a function you give it values as arguments, but when you access the values again they're parameters.

```js
// Defining function (parameters)
function add (myParameter1, myParamter2) {
  return myParameter1 + myParameter2;
}

// Invoking function (arguments)
add(argument1, argument2);
```

Not a huge difference, but it might be helpful to know if you ever want to express something like "how about renaming the parameters" in a code review.

## Natural Born Dynamic Arguments

Function overloading is the act of defining multiple functions with the same name but different parameter lists and the language chosing what function to use based on arguments when invoking. JavaScript is a dynamic language. As such it doesn't have function overloading. This is as function overloading would require a more stricter type system for it to resolve what functions to use (there are types in JavaScript, but not a strict language). But JavaScript plays on the strength of being dynamic and allows you, for good and for bad, to take in what ever arguments you want. Arguments in JavaScript is by nature dynamic where you can pass in what ever amount of arguments you want:

```js
function fn (arg1, arg2, arg3) {
  console.log(arg1, arg2, arg3);
}

fn(); //=> undefined undefined undefined
fn('a'); //=> a undefined undefined
fn('a', 'b'); //=> a b undefined
fn('a', 'b', 'c'); //=> a b c
fn('a', 'b', 'c', 'd'); //=> a b c
```

This means we could implement some kind of overloading if we wanted to:

```js
function fn (arg1, arg2, arg3) {
  if (typeof arg3 === 'undefined') {
    // Do something
  } else {
    // Do something else
  }
}
```

We could make this into a pattern if we wanted and call it something fancy, but essentially we have a overloaded function, as we do different actions based on different function signature.

## Magic Argument XXL

In JavaScript there are a couple of globally defined helper variables. For instance in browser environment we have `window` and `document`. There's also a variable that is automatically and contextually created: `arguments`. If you refer to the variable `arguments` inside a normal function (non-arrow function, see below) it's populated with data passed to the function:

```js
function logAll (/* no explicit parameters */) {
  for(let i of arguments) {
    console.log(i);
  }
}
logAll(1, 2, 3, 4);
//=> 1
//=> 2
//=> 3
//=> 4
```

And by doing this we see what the `arguments` variable is all about. It's a list of all **arguments**. Not parameters. But actual values passed to the function. You see there are no parameters to the function `logAll`, but there are arguments.

This object is an Array like object similar to say NodeList in the browser. It's not quite an array, but behaves in many ways like an array. So we could iterate over the values as seen above, but there are some limitations. For instance, it doesn't inherit the Array prototype. This means we can't do things like:

```js
function logEven (/* no explicit parameters */) {
  const even = arguments.filter(i => i % 2 === 0);
  for(let i of even) {
    console.log(i);
  }
}
logEven(1, 2, 3, 4);
// TypeError: arguments.filter is not a function
```

We have to convert `arguments` into an array. We can do this in one easy step, by using `Array.from`:

```js
function logEven (/* no explicit parameters */) {
  const even = Array.from(arguments).filter(i => i % 2 === 0);
  for(let i of even) {
    console.log(i);
  }
}
logEven(1, 2, 3, 4);
//=> 2
//=> 4
```

We use `Array.from` to convert the array like object to an actual array. Arrays in JavaScript are actually a form of iterable objects, and you could implement your own type, as with NodeList and `arguments`. The `arguments` variable also has a property, `callee`, that refers to the executing function.

### Anonymous Arguments

It's important to remember that with Arrow Functions, `arguments` isn't set. Arrow Functions have transitive scopes in a way, where they refer to the outer context. Meaning it doesn't have it's own `arguments`. In other words: this won't work:

```js
const logEven = (/* no explicit parameters */) => {
  const even = Array.from(arguments).filter(i => i % 2 === 0);
  for(let i of even) {
    console.log(i);
  }
};
logEven(1, 2, 3, 4);
// ReferenceError: arguments is not defined
```

This isn't much of a problem in practise though, for instead of the implicit automatically populated `arguments`, you can use rest values.

## All About the Rest

Instead of using the somewhat mystic `arguments` variable you can use a more explicit alternative. One that is a pattern in other places in the JavaScript langauge also: rest. This is a syntactic extension for collecting all tailing values of something. For instance, it's used in [destructuring lists](https://gist.github.com/mikaelbr/9900818). For functions it can look like this:

```js
function logAll (...args) {
  for(let i of args) {
    console.log(i);
  }
}
logAll(1, 2, 3, 4);
//=> 1
//=> 2
//=> 3
//=> 4
```

Here we say collect all arguments to this array `args`. And this is an actual array. Not an array-like object, but an actual array. We can iterate it, and it has `filter` as it's attached to the Array prototype. In the example above we do the entire argument list, but we can cherry pick parameters from the start of the signature:

```js
function logAll (prefix, ...args) {
  for(let i of args) {
    console.log(`${prefix}: ${i}`);
  }
}
logAll('My Prefix', 1, 2, 3, 4);
```

Here we say, store the first argument as parameter `prefix` and collect the rest of the arguments in the list `args`. We could name `args` what ever we want. Like `tail` or `thisIsATotallyIrretatinglyLongNameForAllOtherArgumentsExceptTheFirstOne`.

You can't do it the other way around. You can't collect all except the last value. That is illigal JavaScript syntax and would cause a syntax error.

```js
function logAll (...args, suffix) { }
// SyntaxError: parameter after rest parameter
```

## Default Arguments

## Function Invoking Default Arguments

## Conclusion

The dynamic nature of JavaScript makes arguments and parameter for an interesting case. You can pass in what ever values you want and it's up to the function to handle what it. You could either fail horribly or handle it in a different way. It's up to you and what is best for the problem you're solving. In some cases this dynamic nature is an advantage of flexibility. In other cases it can be a footgun where it causes errors in your code. The important part is knowing how it works so you can choose what approach is best and be able to understand how to handle the code.

Explicit is almost always better than implicit. If you can explicitly define your parameters and with an explicit API you should do that. This means not relying on `arguments` variable, but using rest, and instead of overloading functions through arguments you should have separate functions with separate names being more explicit with the use case. It is how ever, cases where things like checking argument count and argument types makes sense. When making functions, in any language, thin about the consumer of the function. Think about the invokation site. What makes more sense for using it. Optimizing for terseness and overly dehydrated, DRY, functions will make your code harder to understand.
