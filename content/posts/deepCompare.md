---
title: "Compare Deeply Nested JS objects Interview Question"
date: 2025-06-26T14:12:22+05:30
draft: false
description: "A short summary of the post"
tags: ["JavaScript", "Interview", "Coding", "Frontend"]
categories: ["Javascript", "Interview Prep"]
---

# Compare Deeply Nested Objects in JS

This is a frequently asked question in interviews for Frontend or in general roles where you are expected to work in Javascript.

I will go through each and every step which is required for a beginner to understand all the concepts required to tackle this or any deviation of the question. I was asked the question in my first interview of job switch, and I messed it up by figuring out all the logic during interview, never having seen this earlier. I do not want that to happen to me or my readers again. 😇

## Table of Contents:

1. [ Objects in JS](#objects-in-js)
2. [Why this is even needed?](#why-do-we-need-such-a-function)
3. [Nesting is not the only challenge.](#nesting-is-not-the-only-challenge)
4. [Recursive and Iterative approach.](#recursive-approach-for-deep-compare)
5. [Slight deviation - track the changes.](#slight-deviation-question)

## Objects in JS

Objects store a collection of data. It is most often the format of JSON responses and requests in api calls that is why it is so important to know how to play with these. It will have some key-value pairs, methods as properties, passed as reference(keep this in mind), can show properties of inheritance based on the proptotype.

## Why do we need such a function?

In JavaScript, objects are a reference type. Two distinct objects are never equal, even if they have the same properties. Only comparing the same object reference with itself yields true. Hence even if 2 objects are storing exactly same data, if their memory reference is not same,the comparison will yield false.

```js
// source: developer.mozilla.org

// Two variables, two distinct objects with the same properties
const fruit = { name: "apple" };
const anotherFruit = { name: "apple" };

fruit == anotherFruit; // return false
fruit === anotherFruit; // return false

// Two variables, a single object
const fruit = { name: "apple" };
const anotherFruit = fruit; // Assign fruit object reference to anotherFruit

// Here fruit and anotherFruit are pointing to same object
fruit == anotherFruit; // return true
fruit === anotherFruit; // return true

fruit.name = "grape";
console.log(anotherFruit); // { name: "grape" }; not { name: "apple" }
```

## Nesting is not the only challenge

Now that it is clear there is a need to have a deepCompare function to check the equality of objects, we need to understand our requirements better. Deeply nested objects can always have a different kind of structure, nesting and data types, so we must have a robust function which does not have nested structure as it's dependency, otherwise it will be hard-coded and fail every now and then. The object can have immense nesting, based on the api call or any other requirement, so we need to make sure we have optimised our approach as well.

1. WE cannot just compare with ===.
2. It should be independent of the structure of object nesting.
3. It should be independent of the data types being used either primitive or user defined. (This requires many checks to be added in the iterative version.)
4. Return type is boolean, and takes in 2 objects.
5. Use early return logic so that we do not compare rest of the part as soon as difference is pointed out or equality is guaranteed.

## Recursive Approach for deep compare:

### Early return logic: Think of it like base cases

What could be the factors which can lead us to certainty of our result:

1. If the reference address of both objects are same - TRUE as it means it is pointing to the same memory. `a===b` ? **handle reference equality**

2. Check if either of the objects are null, if so return if a===b again. **handle null**

3. Check if either of the object is not of same type or not. **handle typer mismatch**

4. Now once we are sure that both the arguments have Object as it's type, we can firstly extract the keys from both, and check if the keys are same in number or not.

5. When we are sure that both args are objects and have same number of keys, we can pull up the socks and get started with deeper comparison of objects.

6. Now that we have our early return/ base cases ready, we can check for rest of the cases which might be present. What checks should be there? As we know there are so many configurations an object can have, it is best to ask the interviewer if they can provide a sample JSON or object structure that needs to be compared.

Example - All primitive Types, Set, Date, Map, Symbols, Arrays, Objects, Functions etc.

7. **Let us go through these checks one by one:**

There could be other kind of data types configuring our object which I have not included here, but the point is, refer the documentation for correct way of comparison of these objects as some have key-value pairs, some methods return non-iterable lists which will break the code.
Instead we need to define custom comparison checks for each case.

a. Date -

```js
if (a instanceof Date && b instanceof Date) return a.getTime() === b.getTime();
```

b. RegExp -

```js
if (a instanceof RegExp && b instanceof RegExp) return a.source === b.source && a.flags === b.flags;
```

c. function reference based comparison.

```js
if (typeof a === "function" || typeof b === "function") return a === b;
```

Note that we can also compare using Stringify, but that would be a text based comparison, if one of the functions is arrow function but gives same output or something, it will return false. Not a good practise and tricky to compare functions in objects.

d. Set

```js
if (a instanceof Set && b instanceof Set) {
	if (a.size !== b.size) return false;
	for (const val of a) if (!b.has(val)) return false;
	return true;
}
```

e. Map

```js
if (a instanceof Map && b instanceof Map) {
	if (a.size !== b.size) return false;
	for (const [key, val] of a) {
		// if the key is not present in b or if it is a nested map value (Note there is a recursive call explained later)
		if (!b.has(key) || !deepCompare(val, b.get(key))) return false;
	}
	return true;
}
```

f. Arrays

```js
if (Array.isArray(a) && Array.isArray(b)) {
	if (a.length !== b.length) return false;
	for (let i = 0; i < a.length; i++) {
		// compare here
	}
	return true;
}
```

g. Generic object (Will have to recur due to possibility of nesting):

```js
// Generic Object comparison (plain objects, class instances)
if (typeof a === "object" && typeof b === "object") {
	if (a.constructor !== b.constructor) return false;

	const keysA = Reflect.ownKeys(a);
	const keysB = Reflect.ownKeys(b);

	if (keysA.length !== keysB.length) return false;

	for (const key of keysA) {
		if (!keysB.includes(key)) return false;
		if (!deepCompare(a[key], b[key])) return false;
	}

	return true;
}

// Fallback for unmatched types or values
return false;
```

8. In above cases we have our transition cases/other checks written correctly, we can think about how do we traverse in an object which is present inside another object, this is like recursive structure. And we can call the function on it's branches and return from there.
   As soon as we expect branching of the next nested object start, we make a recursive call at that point.

### Putting this all together:

```js
function deepCompare(a: any, b: any): boolean {
	// Deep Equality comparison by reference
	if (a === b) return true;

	// Handle NaN value
	if (typeof a === "number" && typeof b === "number" && isNaN(a) && isNaN(b)) return true;

	// Handle null value
	if (a === null || b === null) return a === b;

	// Type mismatch in objects
	if (typeof a !== typeof b) return false;

	// Logical cases based on data types:
	// Handle Date
	if (a instanceof Date && b instanceof Date) return a.getTime() === b.getTime();

	// Handle RegExp
	if (a instanceof RegExp && b instanceof RegExp) return a.source === b.source && a.flags === b.flags;

	// Handle Functions: compare by reference
	if (typeof a === "function" || typeof b === "function") return a === b;

	// Handle Set
	if (a instanceof Set && b instanceof Set) {
		if (a.size !== b.size) return false;
		for (const val of a) if (!b.has(val)) return false;
		return true;
	}

	// Handle Map - recursive call as nesting can occur
	if (a instanceof Map && b instanceof Map) {
		if (a.size !== b.size) return false;
		for (const [key, val] of a) {
			if (!b.has(key) || !deepCompare(val, b.get(key))) return false;
		}
		return true;
	}

	// Generic Object comparison (plain objects, class instances)
	if (typeof a === "object" && typeof b === "object") {
		if (a.constructor !== b.constructor) return false;

		const keysA = Reflect.ownKeys(a);
		const keysB = Reflect.ownKeys(b);

		if (keysA.length !== keysB.length) return false;

		for (const key of keysA) {
			if (!keysB.includes(key)) return false;
			if (!deepCompare(a[key], b[key])) return false;
		}

		return true;
	}

	// Fallback for unmatched types or values
	return false;

	// Arrays -  recursive call as nesting can occur
	if (Array.isArray(a) && Array.isArray(b)) {
		if (a.length !== b.length) return false;
		for (let i = 0; i < a.length; i++) {
			if (!deepCompare(a[i], b[i])) return false;
		}
		return true;
	}

	return false;
}
```

## Iterative Approach for deep compare:

An iterative deep comparison avoids recursion by using an explicit stack (or queue) to simulate the call stack. Instead of calling deepCompare recursively on each nested object or array, we maintain a stack of comparison tasks — each representing a pair of values (a, b) that need to be compared.

Since recursive stack is not maintained, we have to mimic the functionality by using a stack.

1. To mimic first function call, initialize a stack with the root pair.

2. While the stack is not empty:

a. Pop a pair from the stack.

b. Apply all the definitive checks on a and b like - reference comparison, type comparison, object comparison, number of keys comparison etc, if they mismatch, return false.

c. Otherwise, push all corresponding nested pairs into the stack for comparison in the next iteration mimic further function calls in the order of which they popped from stack.

```js
function deepCompareIterative(a, b) {
	const stack = [[a, b]];

	while (stack.length > 0) {
		const [val1, val2] = stack.pop();

		// Fast path for identical primitives or references
		if (val1 === val2) continue;

		// NaN equality
		if (typeof val1 === "number" && typeof val2 === "number" && isNaN(val1) && isNaN(val2)) continue;

		// Type mismatch or nulls
		if (typeof val1 !== typeof val2 || val1 === null || val2 === null) {
			return false;
		}

		// Dates
		if (val1 instanceof Date && val2 instanceof Date) {
			if (val1.getTime() !== val2.getTime()) return false;
			continue;
		}

		// RegExp
		if (val1 instanceof RegExp && val2 instanceof RegExp) {
			if (val1.source !== val2.source || val1.flags !== val2.flags) return false;
			continue;
		}

		// Functions (by reference only)
		if (typeof val1 === "function" || typeof val2 === "function") {
			if (val1 !== val2) return false;
			continue;
		}

		// Sets
		if (val1 instanceof Set && val2 instanceof Set) {
			if (val1.size !== val2.size) return false;
			for (let item of val1) {
				if (![...val2].some((el) => deepCompareIterative(el, item))) return false;
			}
			continue;
		}

		// Maps
		if (val1 instanceof Map && val2 instanceof Map) {
			if (val1.size !== val2.size) return false;
			for (let [key, value] of val1) {
				if (!val2.has(key)) return false;
				stack.push([value, val2.get(key)]);
			}
			continue;
		}

		// Arrays
		if (Array.isArray(val1) && Array.isArray(val2)) {
			if (val1.length !== val2.length) return false;
			for (let i = 0; i < val1.length; i++) {
				stack.push([val1[i], val2[i]]);
			}
			continue;
		}

		// Generic Objects (plain objects or class instances)
		if (typeof val1 === "object" && typeof val2 === "object") {
			if (val1.constructor !== val2.constructor) return false;

			const keys1 = Reflect.ownKeys(val1);
			const keys2 = Reflect.ownKeys(val2);
			if (keys1.length !== keys2.length) return false;

			for (const key of keys1) {
				if (!keys2.includes(key)) return false;
				stack.push([val1[key], val2[key]]);
			}
			continue;
		}

		// Primitive inequality fallback
		return false;
	}

	return true;
}
```

I hope the concept is clear now. Try writing down the code by hand on a piece of paper. I always do that to make sure I am able to recall the core conepts and some of the syntax to bring fluidity in my code writing.

## Slight Deviation Question

How would you return the actual changes at the same level of nesting instead of mere true and false?

(Hint: Memoisation method in Recursive approach!)

Thank you for reading!
