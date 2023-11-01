# Concurrency Limit Runner

Run async tasks in JavaScript, limiting concurrency.

## Usage

In this example, we'll fetch URLs, no more than three at a time.

```js
import {runTasks} from 'concurrency-limit-runner';

const urls = [ /* ... */ ];
const tasks = [];
for (const url of urls) {
    tasks.push(async () => await fetch(url));
}

for await (const response of runTasks(3, tasks.values())) {
    console.log(response.status);
}
```

Here's another example that just sleeps, instead of fetching URLs.

```js
import {runTasks} from 'concurrency-limit-runner';

function sleep(ms) { return new Promise(r => setTimeout(r, ms)); }

const tasks = [];
for (let i = 0; i < 20; i++) {
    tasks.push(async () => {
        console.log(`start ${i}`);
        await sleep(Math.random() * 1000);
        console.log(`end ${i}`);
        return i;
    });
}

for await (const value of runTasks(3, tasks.values())) {
    console.log(`output ${value}`);
}
```

## Async Iterators and Generators (`for await`)

This library is built around [async generator functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for-await...of#iterating_over_async_generators), which many JS developers may not be familiar with.

A [generator function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) (aka `function*` function) returns a "generator," an iterator of results. Generator functions are allowed to use the `yield` keyword where you might have normally used a `return` keyword. The first time a caller calls `next()` on the generator (or uses a `for...of` loop), the `function*` function runs until it `yield`s a value; that becomes the `next()` value of the iterator. But the subsequent time `next()` is called, the generator function resumes from the `yield` statement, right where it left off, even if it's in the middle of a loop. (You can also [`yield*`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield*), to yield all of the results of another generator function.)

An "async generator function" (`async function*`) is a generator function that returns an "async iterator," which is an iterator of promises. You can call `for await...of` on an async iterator. Async generator functions can use the `await` keyword, as you might do in any `async function`.

In the example, we call `runTasks()` with an array of task functions; we call [`.values()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/values) on the array to convert the array into an iterator.

`runTasks()` is an async generator function, so we can call it with a `for await...of` loop. Each time the loop runs, we'll process the result of the latest completed task.