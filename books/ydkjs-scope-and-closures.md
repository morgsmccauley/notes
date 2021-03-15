# Scope & Closures

## What is Scope?
Scope is the set of rules that determines where and how a variable (identifier) can be looked-up. This look-up may be for the purposes of assigning to the variable, which is an LHS (left-hand-side) reference, or it may be for the purposes of retrieving its value, which is an RHS (right-hand-side) reference. The ability to store values and pull values out of variables is what gives a program *state*.

These LHS and RHS references refer to sides of the **assignment operator**. LHS references result from assignments operations. This can occur with either the `=` operator or by passing arguments (assigning) to functions.

LHS and RHS references are resolved by looking at the current scope, and if not found, moving to the next. Scope look-up always starts at the innermost scope being executed at the time, and works its way outward/upward until the first match or global scope is reached.

Unfulfilled RHS references result in  `ReferenceError`s being thrown. Unfulfilled LHS references result in an automatic, implicitly-created global of that name (if not in "Strict Mode"), or a  `ReferenceError`  (if in "Strict Mode").

## Lexical Scope
There are two predominant models for how scope works. The first and most common is **Lexical Scope**. The second, which is still used by some languages is called **Dynamic Scope**.

Lexical scope means that scope is defined by author-time decisions of where functions are declared. The lexing phase of compilation is essentially able to know where and how all identifiers are declared, and thus predict how they will be looked-up during execution.

Scope look-ups stop at the first match. The same identifier name can be specified at multiple layers of nested scope. This is called "shadowing" - the inner identifier "shawdows" the out identifier. Global variables are also automatically properties of the global object (`window` in browsers, etc.) and thus can be accessed regardless of being shadowed.

The `eval(..)` function in JavaScript takes a string as an argument, and treats the contents of the string as if it had actually been authored code at that point in the program.  `with` is a short-hand syntax for making multiple property references against an object without repeating the object reference itself each time.

The JavaScript *Engine* has a number of performance optimisations that are performed during the compilation phase. Some of these related to statically analyzing code as it lexes. Both `eval(..)` and `with` have the ability to "cheat" lexical scope. These optimisations are pointless when dynamically created scope come in to play and are therefore not performed. **cheating lexical scope leads to poorer performance.**

## Function Versus Block Scope
Anonymous function expressions are quick and easy to type, and many libraries and tools tend to encourage this idiomatic style of code. However, they have several draw-backs to consider:

1.  Anonymous functions have no useful name to display in stack traces, which can make debugging more difficult.

2.  Without a name, if the function needs to refer to itself, for recursion, etc., the  **deprecated**  `arguments.callee`reference is unfortunately required. Another example of needing to self-reference is when an event handler function wants to unbind itself after it fires.

3.  Anonymous functions omit a name that is often helpful in providing more readable/understandable code. A descriptive name helps self-document the code in question.

Functions are the most common unit of scope in JavaScript but are not the only form. Block-scope refers to the idea that variables and functions can belong to an arbitrary block (generally, any  `{ .. }`  pair) of code, rather than only to the enclosing function.

Other than functions. Starting with ES3, the  `try/catch`  structure has block-scope in the  `catch`  clause.

In ES6, the  `let`  (and also `const`) keyword (a cousin to the  `var`  keyword) is introduced to allow declarations of variables in any arbitrary block of code.  `if (..) { let a = 2; }`  will declare a variable  `a`  that essentially hijacks the scope of the  `if`'s  `{ .. }`block and attaches itself there. When using `var`, the declaration will not be contained in the `if () { ... }` block and will leak to the surrounding scope. Both `let` and `const` will not be hoisted.

Block scope should not be seen as an outright replacement of `var` function scope. Use both where appropriate.

Immediately invoked function expressions (IIFE) can be achieved by wrapping a function in a `( )` pair, making it an expression, then executing that function by adding another `()` on the end, i.e. `(function foo(){ .. })()`. IIFE's "hide" the variables and functions (including the IIFE identifier `foo`) inside a scope, not allowing it to leak in to global scope. The benefit of this being avoiding collision between two different identifiers, which can be common if an IIFE were not used across multiple libraries.

## Hoisting
The JavaScript  _Engine_  first compiles code before it executes, and in so doing, it splits up statements like  `var a = 2;`  into two separate steps:

1.  First,  `var a`  to declare it in that  _Scope_. This is performed at the beginning, before code execution.

2.  Later,  `a = 2`  to look up the variable (LHS reference) and assign to it if found.

What this leads to is that all declarations in a scope, regardless of where they appear, are processed  _first_  before the code itself is executed. You can **visualise** this as declarations (variables and functions) being "moved" to the top of their respective scopes, which we call "hoisting". Declarations themselves are hoisted, but assignments, even function expression assignments, are  _not_  hoisted. Functions are hoisted first, and then variables.

While multiple/duplicate `var` declarations are effectively ignored, subsequent function declarations will override previous ones.

## Scope Closure
Closures happen as a result of writing code that relies on lexical scope. **Closure is when a function can remember and access its lexical scope even when the function is invoked outside its lexical scope.**

```javascript
function foo() {
var a = 2;

function bar() {
console.log( a );
}

return bar;
}

var baz = foo();

baz(); // 2
```
The function `bar()` has lexical scope access to the inner scope of `foo()`. A reference of `bar()` is then passed to `baz()`. When `baz()` is executed, `bar()` is therefore executed, but outside of its declared lexical scope. `bar()` still has lexical scope closure over the inner scope of `foo()`, which keeps that scope alive for `bar()` (and `baz()`) to reference at any later time. **`bar()`  still has a reference to its lexical scope, this reference is called closure.**

```javascript
function CoolModule() {
var something = "cool";
var another = [1, 2, 3];

function doSomething() {
console.log( something );
}

function doAnother() {
console.log( another.join( " ! " ) );
}

return {
doSomething: doSomething,
doAnother: doAnother
};
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```
The above is a pattern in JavaScript we call _module_. The most common way of implementing the module pattern is often called "Revealing Module", which is the variation we present here.

Modules require two key characteristics:
1. An outer wrapping function being invoked, to create the enclosing scope - `CoolModule()`
2. The return value of the wrapping function must include a reference to at least one inner function that then has closure over the private inner scope of the wrapper - `return { ... }`

The object we return has references on it to our inner functions (and therefore their lexical scope through closure), but _not_ to our inner data variables. We keep those hidden and private. It's appropriate to think of this object return value as essentially a **public API for our module**.

ES6 adds first-class syntax support for the concept of modules. When loaded via the module system, ES6 treats a file as a separate module. Each module can both `import` other modules or specific API members, as well `export` their own public API members. The contents inside the _module file_ are treated as if enclosed in a scope closure, just like with the function-closure modules seen earlier.

The contents inside the _module file_ are treated as if enclosed in a scope closure, just like with the function-closure modules seen earlier.
