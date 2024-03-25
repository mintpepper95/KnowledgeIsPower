[[#What is the event loop? What's the order of execution inside it?]]
[[#What happens in the event loop when you call setTimeout?]]
[[#Microtask queue]]
[[#Macrotask queue (callback queue)]]

While JS can only do one thing at a time. You can still do things concurrently in the browser through APIs. Web APIs allow us to do AJAX, manipulate, setTimeout, accessing local storage etc.


#### What is the event loop? What's the order of execution inside it?
Event loop is a process that allows js to process code async.
MacroTasks queue is also known as callback queue.

##### Order of event loop
![[Pasted image 20240325182651.png|400]]
Call stack -> MicroTask > Render > Macrotask ( one macrotask )

So call stack gets cleared, then microtask queue gets cleared, then page rendering, and finally we process 1 macrotask from the callback queue

![[Pasted image 20240325181058.png|600]]

#### What happens in the event loop when you call setTimeout?
When a setTimeout is processed in the stack, it's sent to the corresponding API built in to the browser which waits til the specified time to send this operation back to macrotask queue for processing. DOM events and fetch responses are also queued there before your code runs them.


#### Microtask queue
Mainly for promises, e.g.. an execution of .then/catch/finally and promise handler becomes a microtask. Microtask are used under the cover of await as well.

Microtask queue has a higher priority than the macrotask queue. This means promise handling has priority over async operations.

#### Macrotask queue (callback queue)
setTimeout, setInterval, I/O, big tasks

Immediately after every macrotask ( async operation ), JS engine executes all tasks from microtask queue, prior to running any other macrotasks or rendering or anything else. So no UI or network event between microtasks.

If we want to execute a function async, before the changes are rendered or new events handled, we can schedule it with queueMicrotask.

A microtask queue is a way to execute result of an async function asap, rather being put at the end of the call stack. Promises that resolve before current function ends will be executed right after current function .



Quiz
```ts
console.log(1);
// The first line executes immediately, it outputs `1`.
// Macrotask and microtask queues are empty, as of now.

setTimeout(() => console.log(2));
// `setTimeout` appends the callback to the macrotask queue.
// - macrotask queue content:
//   `console.log(2)`

Promise.resolve().then(() => console.log(3));
// The callback is appended to the microtask queue.
// - microtask queue content:
//   `console.log(3)`

Promise.resolve().then(() => setTimeout(() => console.log(4)));
// The callback with `setTimeout(...4)` is appended to microtasks
// - microtask queue content:
//   `console.log(3); setTimeout(...4)`

Promise.resolve().then(() => console.log(5));
// The callback is appended to the microtask queue
// - microtask queue content:
//   `console.log(3); setTimeout(...4); console.log(5)`

setTimeout(() => console.log(6));
// `setTimeout` appends the callback to macrotasks
// - macrotask queue content:
//   `console.log(2); console.log(6)`

console.log(7);
// Outputs 7 immediately.
```
