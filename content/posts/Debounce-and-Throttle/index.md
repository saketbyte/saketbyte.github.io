---
title: "Debounce and Throttle"
date: 2025-06-20T08:30:38+05:30
draft: false
description: "Understanding Debounce and Throttle function"
tags: ["javascript", "interview", "interesting"]
categories: ["technical"]
---

# Debounce and Throttle Functionality

## Definitions:

Debounce and Throttle are used to control how often a high frequency event triggers the function call, which will lead to calls to the back-end.

-  **Debounce** ensures that the function is called only after a specified delay has passed since the last time it was invoked. This is commonly used in user inputs for search bar, auto-saving a form, form validation once the user has stopped editing a field etc.
-  A debounce function will delay the execution of `func()` until the user stops triggering it for a specified delay in ms. Like we do not want to call search api at each keystroke.

-  **Throttle** ensures that a function is called at most once in every specified time gap regardless of how many times the event is triggered. Useful for infinite scrolling. So we call the required API once immediately, and then wait for t seconds to pass to allow another call to happen again.

Tip - There are comprehensive implementations in lodash library.

---

<!-- <img src="./EventLoopJS.jpg" alt="Event Loop in JS" width="600"/> -->

## Debounce -

As already mentioned `debounce()` wraps the api function call, and everytime the api function has to be called, it goes through the debounce function call, and we check if the timer with some ID, has been active already, we reset it, and wait until it get's over to execute our main api function `func()`. The debounce function will start the timer everytime it is called.

### Overall Logic in Steps:

1. A timeout ID is stored to track scheduled executions.

2. Every time the returned function is called:

   a. It clears the previous scheduled execution.

   b. It schedules a new execution after delay ms.

3. When the delay finishes: `func()` is invoked with the original this context and arguments.

### Code Example

```javascript
// Debounce Function Implementation

// takes in the function on which it should debounce and a default timeout value.
function debounce(func, delay) {
	let timeoutId;

	return function (...args) {
		// 1. Clear any previously scheduled fn call
		clearTimeout(timeoutId);

		// 2. Set a new timer to call fn after delay
		timeoutId = setTimeout(() => {
			func.apply(this, args); // 3. Call fn with correct context and args
		}, delay);
	};
}

// Callback version:

const debounce = (func, delay) => {
	let timeoutId;

	return (...args) => {
		clearTimeout(timeoutId);

		timeoutId = setTimeout(() => {
			func.apply(this, args); // or just: callback(...args);
		}, delay);
	};
};
```

-  We need to bind/apply this to the function to pass correct `this` context and correct event.target information to the api call.
-  Also note that since timer ID is unique in itself but not carrying the information to map it to a specific function call, we are not allowed to share the function with multiple api `func()`s. Each `func()` should have it's own debounce or modify the debounce by adding key to the timer.

### But what is going on under the hood? And why do I need to return a function instead of just calling it in callback of setTimeout?

This is because debounce does not call `func()` immediately. It is different from saying - call this `func()` after delay = 2s, instead it says, take `this` version of my `func()` and after delay = 2s, if the timer is not cleared, execute this. Think about it, your args will be different each time.

### Returned Function - Controller

This function is crucial to the concept of debouncing, because each function call will create a closure which will contain the returned function from debounce having it's own internal state (timeOut ID and relevant arguments) and manages the call based on timer value. Now it is also clear that why do we need to apply reference of this to our returned function so that it takes the correct execution context with it.

Incorrect:

```js
debounce(console.log("Hello"), 300); // this is a blunder in the name of debounce
```

Correct:

```js
const debouncedLog = debounce(() => console.log("Hello"), 300);
debouncedLog();
// Will log after 300ms (if not called again in between)
```

It returns a function with a timer, which is asynchronous task and handled by Web API of `setTimeout()` once that timer is over, it will move the debounced function `func()` to our callback queue/Macrotask queue, which in correct sequence pushes the function to event loop and call stack with required global/local context to execute `func()`.

![Event Loop in JS (Pic from GFG)](EventLoopJS.jpg "Event Loop Diagram from GFG")

(Prerequisite - Event Loop)

Few dots you can connect by seeing the diagram above -

-  Closure is created and the function is returned with it's unique timer ID and args (called Controller function).
-  The function will enter call stack, web API starts to clock the timer (resets if already existed) and execution is popped off until asynch task of timer is finished.
-  Timer is running in the Web API background and is monitored by Event Loop.
-  Timer is over now, so the event loop pushes the task into callback Queue/macrotask queue.
-  If the call stack is empty or at the right turn, this macro task is executed.

Note that if the timer is cancelled by another debounce call, the earlier controller is deleted as the `clearTimeout()` removes the pending timer from Web API environment, and never passed to the callback queue for execution.

---

## Throttle

Throttle is very similar to above Debounce function. Debounce waits before execution while Throttle waits after one execution.

### Overall Logic in steps

1. The first call goes through.

2. The function sets a flag/timer to block further calls.

3. Any calls during the wait period are ignored.

4. After delay ms, the flag resets, and the function can run again

Unlike debounce, throttle does not cancel timers. It remembers the last execution time and checks: "Has enough time passed since last call?"

```js
function throttle(fn, interval = 1000) {
	let lastTime = 0;
	return function (...args) {
		const now = Date.now();
		if (now - lastTime >= interval) {
			lastTime = now;
			fn.apply(this, args);
		}
	};
}

// Callback version

const throttle = (callback, delay = 1000) => {
	let shouldWait = false;

	return (...args) => {
		if (!shouldWait) {
			callback.apply(this, args); // Or just callback(...args)
			shouldWait = true;

			setTimeout(() => {
				shouldWait = false;
			}, delay);
		}
	};
};
```

In above if we make a call to the throttle:

| Time   | Call to `throttledFn()` | Executes? |
| ------ | ----------------------- | --------- |
| 0ms    | Yes (first call)        | Yes       |
| 200ms  | No (still waiting)      | No        |
| 400ms  | No                      | No        |
| 1000ms | Yes (timer expired)     | Yes       |

## Summary

| Feature   | Debounce                         | Throttle                            |
| --------- | -------------------------------- | ----------------------------------- |
| Purpose   | Wait until no calls are made     | Limit execution to every N ms       |
| Use Case  | Input validation, resize         | Scroll, mousemove                   |
| Frequency | Fires **once** after final event | Fires **every interval**            |
| Behavior  | Resets timer on each call        | Ignores calls until interval passes |
