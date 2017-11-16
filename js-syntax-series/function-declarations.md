# The ways of creating functions in JavaScript

If you've ever seen or written JavaScript, you might have noticed there are two ways of defining functions. Even for people with more experience with the language can find difficult to decipher the differences. In this first part of the blog series where we look a JavaScript syntax, we'll dive deep into the differences between function declarations and function expressions. This post will not cover the differences between different types of functions (arrow functions and normal functions), but just the way we define them.

## Two ways

The two ways we have to define functions in JavaScript is declarations and as expressions:

```
function myDeclaredFunction () {
  console.log('This is a function declaration');
}
```

and

```
const myFunctionExpression = function () {
  console.log('This is a function expression');
};
```

We see that one thing that separates them is the variable declaration and the lack of a variable. But we'll dig deeper than this and see what the difference is, both in grammar and usage.

## FunctionDeclarations

One way to define functions in JavaScript is in the top level or in a block, starting with the keyword `function`. This is defined as a part of the [grammar of JavaScript](https://www.ecma-international.org/ecma-262/8.0/#prod-StatementListItem). A function declaration is of type `FunctionDeclaration` (similar to "part of speech" tagging, as noun, verb, etc, in natural language), which is by grammar only allowed to be child of `StatementListItem` which is a part of `StatementList`, which is a child of `Block`, etc, etc. It's all very technical and wordy. But you can think of it like, if the line starts with the keyword `function` inside a block `{}` or at the top of your program, you probably have a function declaration.

In the grammar, it's also said that a `FunctionDeclaration` has to have a name. Or the grammatical term `Identifier`, if you want. The only place where a function declaration can be without identifier, is if it is used in a `export default` setting. In that case you can have a function declaration without an identifier. But everywhere else, your function has to be named.

Following those rules, we can say:

```
function myDeclaredFunction () { } // This is a function declaration

function myDeclaredFunction () {
  function myDeclaredFunction () { } // This is a function declaration
} // This is a function declaration

export default function () { } // This is a function declaration

function () { } // This is *NOT* a function declaration

let func = function () { } // This is *NOT* a function declaration
```

Specifying by the grammar may seem very complicated, and it is. But trust me, for the most cases you intuitively see just by glancing over the code what is a declaration. In most cases, just if the line starts with `function` and not wrapped in parenthesis, it is a function declaration.

### Function Expressions

If we know that function declarations are functions defined in `StatementListItem`s starting with the keyword `function`, we know that all other functions are expressions. So functions defined in `let`, `var` and `const` bindings are expressions. Functions defined as arguments to functions are expressions, and functions defined as values on properties in objects are expressions, and in fact, functions defined in the class sugar are also expressions.

We saw that function declarations almost in all cases needs have an identifier. Is the same true for expressions? No. With function expressions we can allow them to be anonymous. More on this in a later section (`Anonymity and Naming of Expressions`)

## Declarations and Hoisting

## Anonymity and Naming of Expressions

## Referring Variable Named Functions

## Conclusion

We've now seen that there are two ways of assigning functions. We have declarations, which is hoisted with contents, and we have expressions which is not hoisted with contents, but the reference to it may be hoisted if we assign it as a variable. Expressions can be anonymous, they can be named or they can be implicitly named from the variable name. Not naming functions can lead to harder debugging as it will obfuscate stack traces.

We've also seen that using variable names for function expressions is different than using properly named functions. E.g in recursive functions, we can end up referring to the wrong value.

You can use both function declarations and expression, it depends on style and preference. Many like the hoisting ability of function declarations, where you can have more "sentral" functions at the top (visible when you first open the file) and helper functions further down, but other prefer reading sequentially from top to bottom. This is personal and subjective, but now you know the difference and the trade-offs and can be able to make a decision that suits you the best.
