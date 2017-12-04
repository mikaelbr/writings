# Function Parameters in Detail

This is the third part of stand alone blog series on JavaScript syntax in
detail. This time we'll look at function arguments and parameters. You might
think that this is a small topic, but there's suprisingly much to say about it.
In this post we'll cover everything from function overloading to default
arguments and parameter destructuring. Everything you need to know about
function arguments and then some.

See the previous blogpost in the "in Detail" series:
[Variable Declarations in Detail](https://medium.com/@mikaelbrevik/variable-declarations-in-detail-29407b4e4802)

## Argument or Parameter?

First let's get some terms out of the way. What is the difference between
arguments and parameters? We often use these terms interchangably. There's no
need to destinguish between them, but we'll cover it here for completness. As
with all terms it may help us relay information precisely, given that the person
we're talking to also have the same understanding of the terms.

An argument is something we pass into a function, while parameters is something
a function has. When invoking a function we give it values as arguments, but
when we access the values again they're parameters.

```js
// Defining function (parameters)
function add(myParameter1, myParamter2) {
  return myParameter1 + myParameter2;
}

// Invoking function (arguments)
add(argument1, argument2);
```

Not a huge difference, but it might be helpful to know if we ever want to
express something like "how about renaming the parameters" in a code review.

## Natural Born Dynamic Arguments

Function overloading is the act of defining multiple functions/methods with the
same name but different parameter lists, and making the language chose what
function to use based on arguments when invoking. JavaScript is a dynamic
language. As such, it doesn't have function overloading in the traditional
static programming language way you might be used to. This is as function
overloading would require a more stricter type system for it to resolve what
functions to use (there are types in JavaScript, but not a strict language).
JavaScript plays on the strength of being dynamic and allows us, for good and
for bad, take in what ever arguments we want. Arguments in JavaScript are by
nature dynamic, and we can pass in what ever amount of arguments we want:

```js
function fn(arg1, arg2, arg3) {
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
function fn(arg1, arg2, arg3) {
  if (typeof arg3 === 'undefined') {
    // Do something
  } else {
    // Do something else
  }
}
```

We could make this into a pattern if we wanted and call it something fancy, but
essentially we have a overloaded function, as we do different actions based on
different function signatures.

## Magic Argument XXL

In JavaScript there are a couple of globally defined helper variables. For
instance in browser environment we have `window` and `document` or in Node.js we
have `global` or `process`. There's also a variable that is automatically and
contextually created: `arguments`. If we refer to the variable `arguments`
inside a normal function (non-arrow function, see below) it's populated with
data passed to the function:

```js
function logAll(/* no explicit parameters */) {
  for (let i of arguments) {
    console.log(i);
  }
}
logAll(1, 2, 3, 4);
//=> 1
//=> 2
//=> 3
//=> 4
```

And by doing this we see what the `arguments` variable is all about. It's a list
of **all arguments**. Not parameters. But actual values passed to the function.
We see there are no parameters to the function `logAll`, but there are four
arguments.

This magically created `arguments` object is an Array-like object, similar to
say `NodeList` in the browser. It's not quite an array, but behaves in many ways
like an array. We could iterate over the values as seen above, but there are
some limitations. For instance, it doesn't inherit the Array prototype. This
means we can't do things like:

```js
function logEven(/* no explicit parameters */) {
  const even = arguments.filter(i => i % 2 === 0);
  for (let i of even) {
    console.log(i);
  }
}
logEven(1, 2, 3, 4);
// TypeError: arguments.filter is not a function
```

We have to convert `arguments` into an array by using a function defined on the
`Array` type object, `Array.from`:

```js
function logEven(/* no explicit parameters */) {
  const even = Array.from(arguments).filter(i => i % 2 === 0);
  for (let i of even) {
    console.log(i);
  }
}
logEven(1, 2, 3, 4);
//=> 2
//=> 4
```

We use `Array.from` to convert the `Array`-like objects to an actual arrays.
Arrays in JavaScript are a form of iterable objects, and we could implement our
own type, as with `NodeList` and `arguments`. The `arguments` variable also has
a property, `callee`, that refers to the executing function.

It also seems mutating the magic `arguments` value, causes parameter variables
to change. Avoid this, as it would be very surprising in most cases.

```js
function log(a) {
  arguments[0] = 'Bye';
  console.log(a);
}
log('Hello'); //=> 'Bye'
```

### Anonymous Arguments

It's important to remember that with Arrow Functions, `arguments` isn't set.
Arrow Functions have transitive scopes in a way, where they refer to the outer
context. Meaning it doesn't have it's own `arguments`. In other words: this
won't work:

```js
const logEven = (/* no explicit parameters */) => {
  const even = Array.from(arguments).filter(i => i % 2 === 0);
  for (let i of even) {
    console.log(i);
  }
};
logEven(1, 2, 3, 4);
// ReferenceError: arguments is not defined
```

This isn't much of a problem in practise though, for instead of the implicit
automatically populated `arguments`, we can use rest values.

## All About the Rest

Instead of using the somewhat mystic `arguments` variable we can use a more
explicit alternative. One that is a pattern in other places in the JavaScript
langauge also: `rest`. This is a syntactic extension for collecting all tailing
values of _something_. For instance, it's used in
[destructuring lists](https://gist.github.com/mikaelbr/9900818). For functions
it can look like:

```js
function logAll(...args) {
  for (let i of args) {
    console.log(i);
  }
}
logAll(1, 2, 3, 4);
//=> 1
//=> 2
//=> 3
//=> 4
```

Here we say collect all arguments to this array `args`. And this is an actual
`Array`. Not an `Array`-like object, but an actual `Array`. We can iterate it,
and it has `filter` as it's attached to the Array prototype. In the example
above we do the entire argument list, but we can cherry pick parameters from the
start of the signature:

```js
function logAll(prefix, ...args) {
  for (let i of args) {
    console.log(`${prefix}: ${i}`);
  }
}
logAll('My Prefix', 1, 2, 3, 4);
```

Here we say, store the first argument as parameter `prefix` and collect the rest
of the arguments in the list `args`. We could name `args` what ever we want.
Like `tail` or
`thisIsATotallyIrretatinglyLongNameForAllOtherArgumentsExceptTheFirstOne`.

We can't do it the other way around. We can't collect all except the last value.
That is illigal JavaScript syntax and would cause a syntax error.

```js
function logAll (...args, suffix) { }
// SyntaxError: parameter after rest parameter
```

While in the parameter list we do rest, spread is used in argument position.
With the same syntax extension, we can take an `Array`-like structure and apply
them to a function as if they where indevidual arguments.

```js
function logAll(a, b, c, d) {
  console.log(a);
  console.log(b);
  console.log(c);
  console.log(d);
}

const myArray = [1, 2, 3, 4];
logAll(...myArray);
//=> 1
//=> 2
//=> 3
//=> 4
```

This will make all items in `myArray` arguments to `logAll`, binding `a` to `1`,
`b` to `2`, etc.

In parameter position it's called rest, in argument position, spread.

## Default Parameters

Traditionally, in JavaScripit we would do a similar effect to the overloading
from the first chapter to do default parameters. Checking if an parameter's
`undefined`, and act accordingly.

```js
function foo(a, callback) {
  if (typeof callback === 'undefined') {
    callback = function() {};
  }

  // do something with a and safely invoke callback
}
foo(42);
```

When the parameter list grows and we have several optional parameters, this
becomes tedious and hard to read and maintain. We should probably not have too
many parameters, but that's a different discussion. We can use default
parameters to simplify this code:

```js
function foo(a, callback = function() {}) {
  // do something with a and safely invoke callback
}
foo(42);
```

This works on parameter level, not arguments. So the `arguments` variable from
before will not be populated with the default values.

We can have several default parameters, in any order. But defaults get set only
when the argument is `undefined`. Not `null` or other falsy values, but strictly
`undefined`.

```js
function foo(a, callback = function() {}, b) {
  console.log(callback);
}
foo(42, null); //=> null
foo(42, false); //=> false
foo(42); //=> function
```

Default parameters doesn't have to be created inline. We can refer to variables
defined in a outer scopea nd use all types of expressions. Like constants.

```js
// Some configuration
const DEFAULT_TIMEOUT = 42;

function delay(fn, time = DEFAULT_TIMEOUT) {
  // Implementation
}
```

Arguments are executed from left to right and _evaluated at call time_. Firstly,
this means parameters can refer to other parameters to the left:

```js
function foo(a, b = a.foo) {
  console.log(b);
}
foo({ foo: 42 }); //=> 42
```

In this case `a` will shadow any other defined variables defined in an outer
scope. We could also refer to the function itself in cases we wanted to do
recursion with overridable passed in function.

But this also means we can do inline function calls:

```js
const create42 = () => 42;
function foo(a = create42()) {
  console.log(a);
}
foo(); //=> 42
```

This isn't the same as invoking it outside the parameter list and refering to
the value, but the function `create42` is only ever invoked if an argument isn't
passed to the function. This is where the call time evaluation plays a part. We
can prove this by doing logging a comment inside `create42`:

```js
function create42() {
  console.log('invoked');
  return 42;
}
function foo(a = create42()) {}
foo(); // Logs 'invoked'
foo(50); // Logs nothing
```

Now this can be powerful as we can create objects every time we invoke a
function, avoiding shared state between the functions, but it can also mean
potential performance hits. Also, we can't declare variables or functions inside
the function body with the same name as the function called in the parameter
list. This would cause a `ReferenceError`.

But it allows us to do fun things, as creating required parameters:

```js
function required(a, fn) {
  throw new TypeError(`Argument ${a} in ${fn.name} is required`);
}
function foo(a = required('a', foo)) {}
foo(); // TypeError
foo(42); // No Error
```

## Destructuring Paramters

Variables in parameters can be treated as any other values. This means we can
destructure the information passed as arguments. When destructuring we can do:

```js
function log(obj) {
  const { foo } = obj;
  console.log(foo);
}
log({ foo: 42 });
```

Where we unpack `foo` from the object passed in as argument and used as
parameter inside the `log` function body. The `obj` parameter also refers to the
same value, so we can move the destructuring to be inlined in the parameter
list.

```js
function log({ foo }) {
  console.log(foo);
}
log({ foo: 42 });
```

This means in many ways we can have named arguments. And named arguments can in
many cases be preferred over regular parameter lists as they are easier to
change and refactor. The downside is to pack/unpack objects everytime we invoke
a function.

Read more about
[destructuring and all posibilities in this gist](https://gist.github.com/mikaelbr/9900818).

## Conclusion

The dynamic nature of JavaScript makes arguments and parameter for an
interesting case. We can pass in what ever values we want and it's up to the
function to handle what it. We could either fail horribly or handle it in a
different way. It's up to us to deside what is best for the problem we're
solving. In some cases this dynamic nature is an advantage of flexibility. In
other cases it can be a footgun where it causes errors in our code. The
important part is knowing how it works so we can choose what approach is best.

Explicit is almost always better than implicit. If we can explicitly define our
parameters and with an explicit API we should do that. This means not relying on
`arguments` variable, but using rest, and instead of overloading functions
through arguments we should have separate functions with separate names being
more explicit with the use case. It's how ever, cases where things like checking
argument count and argument types makes sense. When making functions, in any
language, thin about the consumer of the function. Think about the invokation
site. What makes more sense for using it. Optimizing for terseness and overly
dehydrated, DRY, functions will make our code harder to understand.
