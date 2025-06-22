---
title: "Hoisting and Scopes"
date: 2025-06-22T13:07:39+05:30
draft: false
description: "Clearing the confusion around Hoisting, scopes, var let const etc."
tags: ["JavaScript", "Interview", "Coding", "Frontend"]
categories: ["JavaScript", "Interview Prep"]
---

## Table of Contents

1. [Hoisting](#hoisting)
2. [Hoisting Behavior](#hoisting-behavior-of-variables)
3. [Scope Related Behaviour of Variables](#scope-related-behavior-of-variables)
4. [Temporal Dead Zone](#temporal-dead-zone)
5. [Fun Question](#fun-question)
6. [Summary](#summary)

---

# Hoisting

(Bubble up and fill in the fragrance of undefined! XD)

In JavaScript, **hoisting** refers to the behavior where variable declarations are moved to the top of their containing scope—either function or block—during the compilation phase. However, how this hoisting occurs differs significantly between `var`, `let`, and `const`.

---

## Hoisting Behavior of variables

### `var` Variables

Variables declared using `var` are **hoisted and initialized with `undefined`**. This means that even if you try to access the variable before its declaration, the code won't throw an error; instead, you'll get `undefined`.

```javascript
console.log(x); // undefined
var x = 10;
```

### `let` and `const` Variables

Unlike `var`, variables declared with `let` and `const` are also hoisted, **but they are not initialized during hoisting**. Attempting to access them before their actual declaration will result in a `ReferenceError`.

```javascript
console.log(x); // ReferenceError: Cannot access 'x' before initialization
let x = 10;
```

This behavior is due to the **Temporal Dead Zone (TDZ)**, a concept introduced in ES6 to make JavaScript behavior more predictable and reduce the likelihood of bugs.

---

## Scope Related Behavior of variables

Scope is not just about accessibility of a variable, but we also need to check lifetime, environment and bindings of the given variable in JS.

<!-- | Feature | Description | Example |
|-----------|----------------------------|------------------|
| **Bold** | Highlights important text | **Important** |
| _Italic_ | Emphasizes words/phrases | _Emphasized_ |
| `Code` | Displays inline code | `print("Hello")` |
| [Link](#) | Adds clickable hyperlinks | [Click me](#) | -->

| Type      | Narrowest Scope it can exhibit | Meaning                                                                             | Reassignment                      |
| --------- | ------------------------------ | ----------------------------------------------------------------------------------- | --------------------------------- |
| **var**   | function                       | available through out the function even if declared within a block ({ }) inside it. | Yes                               |
| **let**   | block                          | Not available beyond the block ({ }) in which it is defined.                        | Yes                               |
| **const** | block                          | Not available beyond the block ({ }) in which it is defined.                        | Mutable but cannot be reassigned. |

If we declare them in the global scope, let and const (and class) will also create global declarations that is why I have mentioned narrowest scope it can exhibit. Where this gets confusing is, unlike var and function, when let, const, and class are used in the global scope, they do not create global properties, whereas var and function do.

```JS
var x // global
{
  var y // also global as it can not exhibit the narrow scope of blocks, only of functions.
}
```

Remember above and the hoisting concept for the fun question in the end.

## Temporal Dead Zone

The **Temporal Dead Zone** is a period between the start of a block scope and the point where the variable is declared and initialized. During this period, any reference to the variable will throw a `ReferenceError`.

### Key Characteristics of TDZ:

1. **Declaration Phase**: When `let` or `const` is encountered, the variable is hoisted but left uninitialized.
2. **Initialization Phase**: The variable gets initialized only when execution reaches the line of declaration.
3. **Accessing During TDZ**: Accessing the variable before initialization results in a `ReferenceError`.

#### Example:

```javascript
console.log(x); // ReferenceError
let x = 10;
```

In this example, `x` is hoisted, but not initialized. Trying to log it before the `let` declaration triggers a `ReferenceError` because it’s still in the TDZ.

---

### Coding standards

To avoid running into errors caused by the Temporal Dead Zone:

-  Always declare your variables at the **top of their scope** before using them.
-  Prefer using `let` and `const` over `var` for clearer and more predictable code.

#### Improved Example:

```javascript
let x; // Declare x at the beginning of the scope
console.log(x); // undefined (no error)
x = 10; // Initialize x
```

In this case, although `x` is not yet assigned a value at the time of logging, it’s declared properly and no error occurs because it’s not being accessed before the declaration line.

---

## Fun Question:

Guess the outputs of the following:

**Code A**

```javascript
for (var i = 0; i < 5; i++) {
	setTimeout(() => console.log(i), 0);
}
```

**Code B**

```javascript
for (let i = 0; i < 5; i++) {
	setTimeout(() => console.log(i), 0);
}
```

### Answer

Remember that var is function scoped, so the same i is shared across all iterations. By the time setTimeout runs, the loop has completed, and i is 3. While let is block scoped, so a new i is created per iteration. Each closure in setTimeout captures the correct value of i at that time.

Wait let me provide some more background information for this:

### Closure

A closure is a function that “remembers” the variables from where it was defined, even after that outer scope has finished execution. It is like remembering the taste of chocolate even if it has finished. XD
Remember that each function will have an enviornment record in the call stack.

Take a look at this code:

```JS
function chocoPie() {
    let whiteChocolate = 1;

    function melted(){
        const taste = ++whiteChocolate;
        return taste;
    }

    return melted;
}

const afterTaste = chocoPie();
// we are storing a reference to the melted function, even when the outer function has completed it's execution.

```

Above, the execution environment of chocoPie (outer function) is retained as a reference to the melted(inner) function even though the call of chocoPie is done. So we have access to the value of whiteChocolate even after the outer function has completed it's execution. This is called Closure, a combination of function objects which retains the environment in function call stack. Happens in nested functions always.

Now here, the difference between var and let is not that clear, because whiteChocolate is function scoped for both. Both var and let will pass through this funnel of scope.

### Back to fun question:

**Note:** In code A and code B examples, `let` is in it's narrowest scope ie `block`, while `var` behaves in it's `function` scope. Hence the difference arises.

-  **Execution Context :** Every time a function runs, a new execution context is created and pushed to the call stack. The execution context includes a Lexical Environment that holds variable bindings.

-  **Lexical Environment :** A Lexical Environment is a structure that holds identifiers (variables/functions) and references to outer environments (called the outer environment reference). Think of it like a scope + closure reference.

-  **Behind the Scences :** The for loop code runs first — no setTimeout fires yet. After 0ms, setTimeout callbacks are put into the task queue. When the call stack is empty, each callback is pulled from the queue and executed in its own execution context.

#### var

_Now var i is hoisted and declared in the global execution context (or function context if inside a function). There is only one i shared by all iterations. Each time setTimeout() is called, a callback function is created, but it closes over the same environment, i.e., the same i. So when the callbacks run, the loop has already completed, and i is now 5. All callbacks log 5, because they all reference the same i from the shared environment._

#### let

_Unlike var, let is block-scoped. The JavaScript engine treats this for loop as creating a new block-scoped environment for each iteration. For every iteration, a new Lexical Environment is created with a fresh i, unique to that loop iteration. The arrow function passed to setTimeout() closes over that unique environment. So when the callbacks run: Each closure has its own i, so they log 0, 1, 2, 3, 4._

---

## Homework question:

```JS
const counter = () => {
    let count = 0;

    return{

        increase: (step = 1) => { count+=step;},
        getValue: () => { return count;};
    };
};

// Think about how closures are working in above.
```

---

Solution:

```js
// Test like this:

const counter = () => {
	let count = 0;

	return {
		increase: (step = 1) => {
			count += step;
		},
		getValue: () => {
			return count;
		}
	};
};

const f = counter();
f.increase();
console.log(f.getValue());
f.increase();
console.log(f.getValue());
f.increase();
console.log(f.getValue());

// output- 1 2 3
```

---

## Summary

-  `var` is hoisted and initialized to `undefined`. So you can refer to a `var` variable and it will output undefined.
-  `let` and `const` are hoisted but not initialized, leading to a TDZ. So you can refer but it will throw ReferenceError.
-  YES, `var, let` and `const` all are hoisted but var is not moved to Temporal Dead Zone.
-  Accessing `let` or `const` variables before their declaration throws a `ReferenceError`.
-  TDZ helps enforce cleaner coding by encouraging early declarations and preventing undefined access.
