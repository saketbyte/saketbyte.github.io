---
title: "JavaScript Interview Questions"
date: "2025-06-19T10:00:00+05:30"
draft: false
tags: ["JavaScript", "Interview", "Coding", "Frontend"]
categories: ["JavaScript", "Interview Prep"]
description: "Frequently asked JavaScript interview questions with explanations and code examples."
---

# JavaScript Interview Questions

## 1: Playing with a typical API response and basic functions in JS

```javascript
const users = [
	{ id: 1, name: "name1", premiumUser: true ,"age":"20"},
	{ id: 2, name: "name2", premiumUser: true ,"age":"40"},
	{ id: 3, name: "name3", premiumUser: false,"age":"40" }
	{ id: 4, name: "name4", premiumUser: false,"age":"30" }

];
// Result
// ['name1', 'name2', 'name3']
```

#### Interviewer Perspective

The interview is looking forward to make you comfortable and testing the waters as to what levels can they go to.

### Extracting Names from above object

1. Using a basic for loop

```javascript
const names = [];

for(let i =0;i<user.length;i++)
    names.push(users[i].name);

console.log("Names: "names);
```

2. Using forEach

```javascript
const names = [];
users.forEach((user) => {
	names.push(user.name);
});

console.log(names);
```

3. Using map

```javascript
const names = users.map((user) => user.name);
console.log(names);
```

## Let us check some other interesting functions:

### Filter premium users

```JS
// Filter the names of the users who have availed premium service
const names = users.filter((user) => user.premiumUser).map(user=>user.name);
console.log(user.name);
```

### Sort based on age

Any sorting will take a predicate function which gives a simple answer - what makes this larger/smaller/equal. Based on this boolean value, we can sort anything.

```JS
//  Refer: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort

function compareFn(a, b) {
  if (a is less than b by some ordering criterion) {
    return -1;
  } else if (a is greater than b by the ordering criterion) {
    return 1;
  }
  // a must be equal to b
  return 0;

  // Solution (Remember sort will mutate the original array)
  users.sort((user1, user2) => user1.age < user2.age? -1:1 );
}
```

### Sort based on age, filter only premium users, extract their names.

```JS
// we can chain these operations.
const names = users
            .sort((user1, user2) => user1.age< user2.age? -1:1 )
            .filter((user) => user.premiumUser)
            .map(user=>user.name);

console.log(user.name);
```

How would you sort objects based on some property? Eg - Sort books name with last name of author.

```JS

const books = [
    {name: "book1", author: "author A"},
    {name: "book2", author: "author D"},
    {name: "book3", author: "author C"},
    {name: "book4", author: "author B"},
]

books.sort((book1, book2)=>{
    const lastName1 = book1.author.split(" ")[1];
    const lastName2 = book2.author.split(" ")[1];
    return lastName1<lastName2? -1:1;
})

console.log(books)

// Note that sort will always modify original array on which it is applied on.


```

### Anatomy of Filter:

Taking directly from developer.mozilla.org docs:

The array argument is useful if you want to access another element in the array, especially when you don't have an existing variable that refers to the array.

The following example first uses map() to extract the numerical ID from each name and then uses filter() to select the ones that are greater than its neighbors.

```JS
const names = ["JC63", "Bob132", "Ursula89", "Ben96"];
const greatIDs = names
  .map((name) => parseInt(name.match(/\d+/)[0], 10))
  .filter((id, idx, arr) => {
    // Without the arr argument, there's no way to easily access the
    // intermediate array without saving it to a variable.
    if (idx > 0 && id <= arr[idx - 1]) return false;
    if (idx < arr.length - 1 && id <= arr[idx + 1]) return false;
    return true;
  });
console.log(greatIDs); // [132, 96]

// The array argument is not the array that is being built.
// There is no way to access the array being built from the callback function.

```

### Application of reduce function:

```JS
const arr = [1, 2, 3, 4];
// here acc is like the sum variable in sum+= arr[i]
// Think like - reduce this array, and accumulate the value in acc.
// Pick the current element as curr, and perform some
// operation on curr and acc.
// Starting with an initial val of 0 here.
const total = arr.reduce((acc, curr) => acc + curr, 0);
console.log(total); // 10

```

### Understanding Splice function:

```JS
// Splice is used to modify the original array - insert, delete opertations.
// Syntax:
// array.splice(starthere, deleteCount, item1, item2, ..., itemN)
// startHere - starts to modify consecutively from this element
// delete these many from startHere and insert the remaining after it
// Remember it need not match delete count with N items,
// the x number of elements are deleted and N-x are inserted.

const items = ['a', 'b', 'c', 'd'];
items.splice(1, 2, 'x', 'y', 'z'); // delete 2 and insert remaining
console.log(items); // ['a', 'x', 'y', 'z','d']
```

### Finding all users which satisfy a condition:

We can use the some method in above requirement. The some() method is an iterative method. It calls a provided callbackFn function once for each element in an array, until the callbackFn returns a truthy value. If such an element is found, some() immediately returns true and stops iterating through the array.

```JS
const isValExists = (val, users) => users.some((user) => user.name === val)
// Or see the below example from Mozilla docs where we are trying to check
// if the array is strictly increasing or not.
// filter takes only positive elements some uses num as current element,
// idx as index, and arr to access the array returned
// by filter in the intermediate step.

const numbers = [3, -1, 1, 4, 1, 5];
const isIncreasing = !numbers
  .filter((num) => num > 0)
  .some((num, idx, arr) => {
    // Without the arr argument, there's no way to easily access the
    // intermediate array without saving it to a variable.
    if (idx === 0) return false;
    return num <= arr[idx - 1];
  });
console.log(isIncreasing); // false

```

### Implement a range function which takes start, end and generates an array with those values

```Javascript

// Method one: Using for loop

const genRange = (start, end) =>{
    const result = [];
    for(let i = start, i <=end; i++){
        result.push(i);
    }
    return result;
}

// Method two: Using array.keys method

const genRange = (start,end) => {
    // spreading an array of size e-s+1 keys will give us the value easily,
    // but we need to map it from 0 to n-1 to a to b hence we use map.
    return [...Array(end-start+1).keys()].map((item) => item+start);
}
// end - start - number of elements,

```

## Summary

| Function Name   | Basic Syntax                                            | One-Line Use Case                          | Mutates Original Array | Returns a Value                       |
| --------------- | ------------------------------------------------------- | ------------------------------------------ | ---------------------- | ------------------------------------- |
| `forEach`       | `arr.forEach((item, idx, arr) => { ... })`              | `arr.forEach(item => console.log(item))`   | No                     | No                                    |
| `map`           | `arr.map((item, idx, arr) => { ... })`                  | `arr.map(item => item.name)`               | No                     | Yes (new array)                       |
| `filter`        | `arr.filter((item, idx, arr) => { ... })`               | `arr.filter(item => item.age > 30)`        | No                     | Yes (new array)                       |
| `reduce`        | `arr.reduce((acc, curr, idx, arr) => { ... }, initVal)` | `arr.reduce((acc, curr) => acc + curr, 0)` | No                     | Yes (single value)                    |
| `sort`          | `arr.sort((a, b) => { ... })`                           | `arr.sort((a, b) => a.age - b.age)`        | Yes                    | Yes (same array)                      |
| `splice`        | `arr.splice(start, deleteCount, ...items)`              | `arr.splice(1, 2, 'x', 'y')`               | Yes                    | Yes (array of removed items)          |
| `slice`         | `arr.slice(start, end)`                                 | `arr.slice(1, 3)`                          | No                     | Yes (new array)                       |
| `some`          | `arr.some((item, idx, arr) => { ... })`                 | `arr.some(item => item.active)`            | No                     | Yes (boolean)                         |
| `every`         | `arr.every((item, idx, arr) => { ... })`                | `arr.every(item => item.active)`           | No                     | Yes (boolean)                         |
| `find`          | `arr.find((item, idx, arr) => { ... })`                 | `arr.find(item => item.id === 2)`          | No                     | Yes (first matched item or undefined) |
| `findIndex`     | `arr.findIndex((item, idx, arr) => { ... })`            | `arr.findIndex(item => item.id === 2)`     | No                     | Yes (index or -1)                     |
| `flat`          | `arr.flat(depth)`                                       | `[1, [2, [3]]].flat(2)`                    | No                     | Yes (new flattened array)             |
| `concat`        | `arr1.concat(arr2)`                                     | `arr1.concat(arr2)`                        | No                     | Yes (new array)                       |
| `includes`      | `arr.includes(value)`                                   | `arr.includes(5)`                          | No                     | Yes (boolean)                         |
| `indexOf`       | `arr.indexOf(value)`                                    | `arr.indexOf('a')`                         | No                     | Yes (index or -1)                     |
| `join`          | `arr.join(separator)`                                   | `arr.join(', ')`                           | No                     | Yes (string)                          |
| `reverse`       | `arr.reverse()`                                         | `arr.reverse()`                            | Yes                    | Yes (same array)                      |
| `push`          | `arr.push(item)`                                        | `arr.push('new')`                          | Yes                    | Yes (new length)                      |
| `fill`          | `arr.fill(value, start, end)`                           | `arr.fill(0, 1, 3)`                        | Yes                    | Yes (same array)                      |
| `Array.from`    | `Array.from(arrayLike, mapFn?)`                         | `Array.from('123', Number)`                | No                     | Yes (new array)                       |
| `Array.isArray` | `Array.isArray(value)`                                  | `Array.isArray([1, 2])`                    | No                     | Yes (boolean)                         |

---

## 2: null vs undefined

### Interviewer Perspective

How well do you understand type system and usage clarity while handling API responses or default values while error handling.

### Answer

#### Undefined

This is a primitive value, with type as undefined. The variable is declared but not assigned yet. It is set by JS engine automatically.

#### null

This is a null value but weirdly it has a type of Object in JS. It is explicitly set to variables manually. Also a falsy value by nature.

Please note (lose equality) `JS console.log(null == undefined)` will return true as both are false values. In essence undefined is a parallel to notion of missing, while null is to empty.
In the temporal dead zone, the value of any variable is undefined. ie, declared but not assigned any value but tried to log - it will show undefined.

## 3: Write a function multiply(a)(b) which returns product of a and b

### Interviewer Perspective

The interviewer wishes to check your grasp on concept of currying and it's BTS.

<!-- Add insight into what the interviewer is trying to assess -->

### Correct Answer

Currying is a technique in functional programming where a function with multiple arguments is transformed into a sequence of nested functions, each taking a single argument at a time.

Let me write the code:

```JS

const multiply = (a) =>{
    return (b) =>{ return a*b};
}

// OR

const multiply = a => b => a*b;

```

But what is going on here?
We return functions as the result of a function, and to which we pass further parameter values. This can be expanded recursively as well.

```JS

function currySum(expectedArgsCount) {
  const args = [];

  function curried(...newArgs) {
    args.push(...newArgs);

    // If we have enough arguments, compute the sum
    if (args.length >= expectedArgsCount) {
      return args.slice(0, expectedArgsCount).reduce((a, b) => a + b, 0);
    }

    // Otherwise, return the curried function again
    return curried;
  }

  return curried;
}

const sum = currySum(3);

console.log(sum(1)(2)(3));      // 6
console.log(sum(1, 2)(3));      // 6
console.log(sum(1)(2, 3));      // 6
console.log(sum(1, 2, 3));      // 6


```

### Need?

It can be used in setInputValue or to apply a function on another function, basically Higher Order functions.

```JS
const hasRole = role => user => user.roles.includes(role);

const users = [
  { name: 'Alice', roles: ['admin', 'user'] },
  { name: 'Bob', roles: ['user'] }
];

const admins = users.filter(hasRole('admin'));
// Alice

// Example 2
const getThisPropertyObject = curry((prop, obj) => obj[prop]);
const getAge = get("age");
const customFilter = curry((fn,val) => val.map(fn));

customFilter(getAge, [{age:2}])
// 2
```

---

## 4: Shuffle an Array of Objects

### Interviewer Perspective

They probably want to check language and slight algorithmic profficiency.

### Correct Answer

The approach I am using here is to attach one random value to the object in an array, and then sort based on this random value.

### Code Example

```javascript
const shuffler = (input) => {
    return input
            .map(item) => ({randomVal:Math.random(), inputVal:item})
            .sort(item1,item2)=> (item1.randomVal-item2.randomVal)
            .map((i) => i.inputVal);

// map and attach randomVal property
// sort based on the random value distribution
// map and remove that random val again.
}
```

---

## 5: Behavior of `this` keyword

There can be 3 places where `this` is used.

1. Inside a function
2. Inside an Object
3. Inside a Class
4. Inside a nested function in a class

### 1. Inside a function

When we use this inside a Standalone function, in strict mode this would be undefined but in non-strict mode this will refer to the global/window object.

```javascript
function useOfThis() {
	console.log(this);
}

useOfThis();
// window
```

### 2. Inside an Object

Let us define an object with some properties and a function which uses `this` to refer to one of these properties.
In this case, the `this` keyword will always refer to the object which calls it. Here chocolateCake.

```javascript
const chocolateCake = {
	ingredients: ["semolina", "baking powder", "cocoa", "powdered sugar", "chocolate syrup", "dry fruits"],
	steps: "Bake something like you would bake a cake. :)",
	cook() {
		// do something
		console.log("Cake is baked using", this.ingredients);
	}
};

chocolateCake.cook();
// output - Cake is baked using ["semolina", "baking powder" .....]
```

**NOT SO SIMPLE**

```javascript
const cookLater = chocolateCake.cook;
cookLater();
// this is global or undefined in this case. :)
```

### 3. Inside a Class Method:

This is simple, the `this` keyword will associate itself with the closest object with which it is called.

```js
class Cake {
	constructor(flavor) {
		this.flavor = flavor;
	}

	bake() {
		console.log("Oven started... and the smell of", this.flavor, "cake is coming to you!");
	}
}
const ananas = new Cake("ananas");
ananas.bake();
```

---

### 4. A function nested inside a Class Method (Tricky one)

In case we have a function nested inside a class function, and we try to call the class function with help of an object, the `this` keyword will go crazy. It starts to behave like a normal function and so it outputs undefined in strict mode or global object in non-strict mode.
Bear with me I will explain the fix as well.

**Reason:** First let us understand why? The reason `this` inside a nested function doesn’t refer to the class instance is because `this` is dynamically bound based on how a function is called, not where it is written unless it’s an arrow function which borrows the scope of it's just outer lexical scope for `this`.

```js
class Cake {
	constructor() {
		this.cakeSize = 1;
	}

	bake() {
		function makeVegan() {
			console.log("The vegan cake size is ", this.cakeSize);
		}
		makeVegan();
	}
}

const veganCake = new Cake();
veganCake.bake();

// Output - The vegan cake size is undefined
```

### How to solve?

There are three possible methods to resolve it: a. using arrow function or b. defining this\_ as `this` just outside the `function` c. use .bind()

```js
//a. Using arrow function
bake() {
  const makeVegan = () => {
			console.log("The vegan cake size is ", this.cakeSize);
  };
  makeVegan();
}

//b. Using this_

bake() {
    const this_ = this
    function makeVegan() {
			console.log("The vegan cake size is ", this.cakeSize);
		}
		makeVegan();
}

//c. Using .bind()

bake() {
  const makeVegan = () => {
			console.log("The vegan cake size is ", this.cakeSize);
  };
  makeVegan.bind(this)();
}
```

### Let us recap:

| Context                      | `this` refers to                           |
| ---------------------------- | ------------------------------------------ |
| Function                     | `window` or `global` / undefined if strict |
| Object method                | The object itself                          |
| Class method                 | The class instance                         |
| Function inside class method | Global or `undefined` (NOT class instance) |
| Arrow function               | **Lexically inherited** from parent scope  |

---

<!--
## 6: <!-- Replace with your question -->

<!-- ### Interviewer Perspective -->

<!-- Add insight into what the interviewer is trying to assess -->

<!-- ### Correct Answer -->

<!-- Write the detailed explanation of the answer -->

<!-- ### Code Example -->

<!-- ### One-liner Summary -->

<!-- Write a short, crisp answer here for revision -->

<!-- ### Supporting Image -->

<!-- ![Alt text describing the image](path/to/image.png) -->
