[[#What are the two things async do?]]
[[#Downside of async/await]]
[[#Chaining async and await]]
[[#await and event loop]]


#### What are the two things async do?
Async keyword does two things.
1. marks function to return a Promise, if a fn returns a value, then it's wrapped inside a Promise
2. Allows us to use `await` inside the fn

```ts
// hello is now a promise
let hello = async () => 'hello';

hello.then(x => console.log(x));
```

Await works only inside async fn, doesn't work in regular functions. 
It pauses execution in the current code until promise resolves.
While `await` may look like it's blocking, JS runtime can execute other tasks while waiting for the promise to settle.

You can use try catch with async/await.

Since async functions return a promise, we can use promise API on them.

##### Downside of async/await
Note async await may block code in function, so it's a good idea to start async tasks concurrently instead of using await (starting them sequentially).
```ts
// timeout for some interval before resolving
function timeout_promise(interval) {
    return new Promise((resolve, reject) => {
        setTimeout(function() {
            resolve('done');
        }, interval);
    });
}


async function time_test() {
	// We start all promises concurrently, fast
	let p = timeout_promise(3000);
	let p2 = timeout_promise(3000);
	let p3 = timeout_promise(3000);
	
	// Start and wait for each promise to settle, slow
	await timeout_promise(3000);
	await timeout_promise(3000);
	await timeout_promise(3000);
}
```

#### Chaining async and await
```ts
await (await myAsyncFunction()).somethingElseAsync()
```


#### await and event loop
When an await is encountered in code ( either in async function or in a module ), awaited expression is executed, with all code that depends on the awaited value pushed onto the microtask queue. Main thread is then freed for next task in event loop. This happens even if awaited value is an already resolved promise or not a promise.

```ts
// Here the async fn is practically sync, as no await statements
async function foo(name) {
  console.log(name, "start");
  console.log(name, "middle");
  console.log(name, "end");
}

foo("First");
foo("Second");
// First start
// First middle
// First end
// Second start
// Second middle
// Second end

// Above equivalent to
function foo(name) {
	return new Promise((resolve, reject) => {
		console.log(name, "start");
		console.log(name, "middle");
		console.log(name, "end");
		resolve();
	})
}
```

Another example.

```ts
async function foo(name) {
  console.log(name, "start");
  // all code that depends on await is pushed to microtask queue
  await console.log(name, "middle");
  console.log(name, "end");
}

foo("First");
foo("Second");
// First start
// First middle, now encounters await
// Second start
// Second middle, now encounters await
// First end
// Second end

// Here is above written in promise
// Presence of then()/await means code will take one extra tick to complete
function foo(name) {
  return new Promise(resolve => {
    console.log(name, "start");
    resolve(console.log(name, "middle"));
  }).then(() => {
    console.log(name, "end");
  });
}
```
