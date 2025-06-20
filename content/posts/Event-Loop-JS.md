---
title: "Event Loop JS"
date: 2025-06-20T08:52:17+05:30
draft: true
description: "A short summary of the post"
tags: ["example"]
categories: ["blog"]
---

The event loop is an important concept in JavaScript that enables asynchronous programming by handling tasks efficiently. Since JavaScript is single-threaded, it uses the event loop to manage the execution of multiple tasks without blocking the main thread.

```js
console.log("Start");

setTimeout(() => {
	console.log("setTimeout Callback");
}, 0);

Promise.resolve().then(() => {
	console.log("Promise Resolved");
});

console.log("End");
```
