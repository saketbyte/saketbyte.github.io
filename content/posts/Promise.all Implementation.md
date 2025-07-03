---
title: "Promise Aggregator Functions Implementation"
date: 2025-07-02T20:58:12+05:30
draft: false
description: "This blog talks about implementing Promise api functions from scratch in JS to help understand their working and usage"
tags: ["JavaScript", "Interview", "Coding", "Frontend"]
categories: ["Javascript", "Interview Prep"]
---

## Contents:

1. Promise.all
2. Promise.allSettled
3. Promise.any
4. Promise.race

JavaScript’s Promise.all, Promise.allSettled, Promise.any and Promise.race are essential when managing multiple async operations. But what really happens under the hood? In this post, let us rebuild each from scratch using async/await only and visualize how they interact with the event loop.

First like always a brief refresher of Event loop components.

| Component           | Description                                               |
| ------------------- | --------------------------------------------------------- |
| **Call Stack**      | Runs synchronous code one line at a time                  |
| **Web APIs**        | Handles async operations like `fetch`, `setTimeout`, etc. |
| **Microtask Queue** | Queues `await`/`.then` resolutions                        |
| **Macrotask Queue** | Queues `setTimeout`, DOM events, etc.                     |
| **Event Loop**      | Orchestrates moving tasks from queues into the call stack |

Let us start.

## Promise.all – Fail-Fast-Policy

Promise.all shoots all the promises concurrently and gives us the final result in `max(T_promise_1,T_promise_2.... T_promise_n)` time, but returns immediately as soon as the earliest error/rejection is received.

Understanding the requirements:

**Execution Behaviour** : Resolves only when all promises fulfill. Rejects immediately if any promise fails.

This kind of behaviour is useful when the final outcome has mandatory dependencies on more than one promises. And even if one of the promise fails, we cannot proceed further. Hence for this kind of fail fast and return logic we use promise.all.

**Edge Cases :**

1. The return type should be a promise because user might have to chain it or use await on the response again.

2. If the iterable of promises received is blank then we have to return back blank array.
3. The order of return values stored in returned promise should be the same as that of input respectively.
4. The returned Promise can contain non promise objects.

## Implementation Logic:

Check for the edge case of input being empty and once that is cleared we can dive deeper into the code.

We can iterate on the iterable using forEach or map if we want to create a new array. We also need to check if we have resolved all promises or not, and break _immediately_ if even one of them fails.

**Note that:** forEach is an older functionality which does not utilise asynchronous method so either we wrap it in an asynchronous function or we use an IIFE.
Or we can totally avvoid forEach by using map function.

This function behaves like if and only if everyone has completed the work the entire class will get a games period, even if one fails, the entire class will not get the reward.

### Implementation (with async/await)

```js
// Promise.all() = promiseAll
async function promiseAll(promises) {
	// array to store the result of Promises.
	const results = [];
	// Number of resolved Promises to ensure all are resolved before returning.
	let resolvedCount = 0;

	// As discussed above we need to return a promise object from the function.
	return new Promise((resolve, reject) => {
		if (promises.length === 0) {
			resolve(results);
			return;
		}
		// iterating through each promise index-wise in same order.
		promises.forEach((p, index) => {
			// Used an async IIFE here, but an async func or map can be used too
			// IIFE is called inside forEach separately for each Promise to make it async.
			// forEach(async (p,idx)) won't work it is like a for loop which does not wait for any callback to finish.

			(async () => {
				try {
					const val = await p;
					results[index] = val;
					resolvedCount++;
					if (resolvedCount === promises.length) {
						resolve(results);
					}
				} catch (err) {
					reject(err);
					// Immediately reject on first failure
				}
			})();
		});
	});
}

const output = await promiseAll([Promise.resolve("A"), Promise.resolve("B"), Promise.resolve("C")]);
// ➝ ["A", "B", "C"]

// OR - promiseAll([apiCall1(), apiCall2(), apiCall3()]).then((results) => {});

output.map(() => {
	// do something
});

await promiseAll([
	Promise.resolve("X"),
	Promise.reject("Y")
	// ➝ throws "Y"
]);
```

### What would a map based iteration look like in above case?

```js
// Note the async keyword here allows us to skip asynch IIFE or named function inside it.
 promises.map(async (p, index) => {
      try {
        const val = await p;
        results[index] = val;
        resolvedCount++;
        if (resolvedCount === promises.length) {
          resolve();
        }
      } catch (err) {
        reject(err);
      }
	}
```

## Promise.allSettled – Record everything

`Promise.allSettled()` is a beautiful function which will come to you after all the promises are settled, either rejected or resolved doesn't matter. But they will have arrived in their final state before coming to you.
This is mainly used when the results required are not mandatory, like get all that you can for now, dont care about rejection.

Understanding the requirements:

**Execution Behaviour :** Waits for all promises to finish, regardless of success or failure. Returns an array with status and corresponding value or reason.

**Edge Cases and logic:**

1. Must wait for all promises to settle ie iether resolved(fulfilled) or rejected.
2. Empty input must return back empty output.
3. The result is an array of JS objects which contain status and value for sucess and status and reason for failure.

Promise.allSettled is like a teacher asking has everyone settled or not, it does not matter if the homework of students is finished or not. Just settle down!

### Implementation

```js
async function promiseAllSettled(promises) {
	const results = new Array(promises.length);

	// creating a new promise
	await new Promise((resolve) => {
		let settledCount = 0;
		// looping over the promises iterable
		promises.map(async (p, index) => {
			try {
				const val = await p;
				// recording the results in same order
				results[index] = { status: "fulfilled", value: val };
			} catch (err) {
				results[index] = { status: "rejected", reason: err };
			} finally {
				settledCount++;
				// sanity check for count of promises
				if (settledCount === promises.length) {
					resolve();
				}
			}
		});
	});

	return results;
}

const result = await promiseAllSettled([Promise.resolve("OK"), Promise.reject("Failed")]);

console.log(result);

// [
// { status: "fulfilled", value: "OK" },
// { status: "rejected", reason: "Failed" }
// ]
```

## Promise.any – Any earliest fulfillment

**Execution Behaviour:**
This function Resolves with the first fulfilled promise and will Reject only if all Promises fail. Basically checking the OR of settled states of all promises where fulfilled is T and rejected is F.

So if we have p1 and p2 (assume both resolve), in sequence given to promise.any, it will immediately return p1 as resolved.

**Edge cases and logic:**

1. Empty input should have an empty array in returned promise.
2. Return at the first success.
3. Reject if all rejected.

Think of it like, if a teacher has forgotten her red pen today, and she asks if any student has a red pen. If none of the students answer, the Promise fails, if even one has it, the job is done.

```js
async function promiseAny(promises) {
	const errors = new Array(promises.length);

	await new Promise((resolve, reject) => {
		let rejectedCount = 0;

		promises.map(async (p, index) => {
			try {
				const val = await p;
				resolve(val); // resolves first success
			} catch (err) {
				errors[index] = err;
				rejectedCount++;
				if (rejectedCount === promises.length) {
					reject(new AggregateError(errors, "All promises rejected"));
				}
			}
		});
	});
}

const result = await promiseAny([Promise.reject("A"), Promise.reject("B"), Promise.resolve("C")]);
// ➝ "C"
//If all fail:

await promiseAny([Promise.reject("X"), Promise.reject("Y")]);
// throws AggregateError: All promises rejected
```

## Promise.race – Any earliest settlement

**Execution Behaviour :** Resolves or rejects as soon as the first promise settles does not matter fulfilled or rejected. Gives the output from the fastest settlment.

**Edge case and logic :**

Do not confuse this with Promise.all(). In case of failure of a promise, it will return immediately just like Promise.all(), but in case of fulfillment of a promise, Promise.all() will look for other promises' result while Promise.race will return with that also.

This helps us to pick the fastest results of all.

```js
async function promiseRace(promises) {
	return new Promise((resolve, reject) => {
		// using for of to account for non promise objects.
		for (const p of promises) {
			(async () => {
				try {
					const val = await Promise.resolve(p);
					resolve(val);
				} catch (err) {
					reject(err);
				}
			})();
		}
	});
}

const result = await promiseRace([new Promise((res) => setTimeout(() => res("Fast"), 100)), new Promise((res) => setTimeout(() => res("Slow"), 500))]);
//  "Fast"
```

## Summary of all 4 functions so far:

| Function               | **_fulfills_**               | **_rejects_**               | **Settled on**           | **Output**                           |
| ---------------------- | ---------------------------- | --------------------------- | ------------------------ | ------------------------------------ |
| **Promise.all**        | **all fulfill**              | **any one rejects**         | Last to fulfill (if all) | Array of values in order             |
| **Promise.allSettled** | Never                        | Never                       | After all settle         | Array of `{status, value/reason}`    |
| **Promise.any**        | **first fulfills**           | **all reject**              | First to fulfill or all  | Value of first fulfilled, else error |
| **Promise.race**       | **first to settle fulfills** | **first to settle rejects** | First to settle          | Value or reason of first to settle   |

## Fun analogy to remember the behaviours:

| Function               | Teacher’s Promise Rule                                                                        |
| ---------------------- | --------------------------------------------------------------------------------------------- |
| **Promise.all**        | _“If even one student has not completed the homework, no reward for entire class, rejected.”_ |
| **Promise.allSettled** | _“Every student's record is tracked. No conclusion of failed or settled.”_                    |
| **Promise.any**        | _“I just need one student to give me a red pen?”_                                             |
| **Promise.race**       | _“Whichever student tries to share their thought process/answer first.”_                      |
