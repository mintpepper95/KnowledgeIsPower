[[#How do setTimeout and setInterval work in regards to event loop?]]
[[#setTimeout]]
[[#setTimeout is not immediate]]
[[#How do recursively setTimeout and setInterval differ?]]



#### How do setTimeout and setInterval work in regards to event loop?
[[Promise#Async and event loop]]

#### setTimeout

```ts
// basic syntax
setTimeout(fn, delay = 1000, arg1, arg2...)


// clearTimeout
let my_greeting = setTimeout(name => alert(`hello ${name}`), 1000);

// Canceling a timeout before timer elapses
clearTimeout(my_greeting)


// Nested setTimeout
// We can call setTimout recursively
let i = 0;

setTimeout(function run() {
	console.log(i);
	i += 1;
	setTimeout(run, 100);
}, 100);


// We can also do it with setInterval
setInterval(function run() {
	console.log(i);
	i += 1;
}, 100);

```

Note the delay here is the min time it takes to execute, callback can't run until main thread is clear (stack is empty).


#### setTimeout is not immediate
setTime and setInterval only runs after main thread.
Here the event loop will only execute the callback function of `setTimeout` after the current synchronous code execution is complete.
```ts
// This will run after for-loop!
setTimeout(() => {
	console.log('World');
}, 0);

// This will run first!
for (let i = 0; i < 1000; i++) {
	console.log('hi');
}
```


#### How do recursively setTimeout and setInterval differ?

With recursive setTimeout, the delay starts to countdown after current code has finished. So it excludes time taken to run the current code.
![[Pasted image 20240325154039.png]]


`setInerval` includes time to execute the current code. 
![[Pasted image 20240325154025.png]]


