[[#Async and event loop]]
[[#Thread vs Process]]
[[#What is callback hell?]]


#### Async and event loop
Main thread is the single thread in the browser that runs all JS code.
If main thread is executing then event loop is unable to run code and browser will appear frozen.

Browsers allow us to run certain operations async. Async functions eg. `setTimeout` or `setInterval` are handed off to Web API and once they are finished, the callback fn is put into the callback queue and then pushed onto the call stack.

Async code runs after main thread is clear.
What's the order of below?
```ts
let image;

fetch('coffee.jpg').then((response) => {
  console.log('It worked :)')
  return response.blob();
}).then((myBlob) => {
  let objectURL = URL.createObjectURL(myBlob);
  image = document.createElement('img');
  image.src = objectURL;
  document.body.appendChild(image);
}).catch((error) => {
  console.log('There has been a problem with your fetch operation: ' + error.message);
});

console.log ('All done!');
```
A: All done is run first, then it worked. As fetch is async and runs without blocking. Meaning code continues to execute after it.

#### Thread vs Process
Thread is a unit within a process.
Each thread can only do one task at a time.
![[Pasted image 20240325154558.png|320]]

#### What is callback hell?
Imagine we want to run some callback after a fn has finished.
```ts
function readFileWithCallback(filename, callback) {
	fs.readFile(filename, 'utf8', (err, data) => callback(data));
}
```

##### Callback hell
When we pass a callback to a function, the callback itself may need some other callbacks, which means we also need to define those other callbacks in this callback. And those callbacks may also need other callbacks.

Code becomes confusing. Also not adhere to dependency inversion with SOLID
```ts
// Here we pass callback to each fn
chooseToppings(function(toppings) {
	placeOrder(toppings, function(order) {
		collectOrder(order, function(pizza) {
			eat(pizza);
		})
	})
})


// cleaner way
chooseToppings.then(placeOrder).then(collectOrder).then(eat).catch(failure);
```



#### Promise
Promise constructor takes in an fn called executor. Executor has two parameters, resolve and reject. The keywords `resolve` and `reject` are not keywords, so we can actually call them other names too. They can also be defined functions that expects 0 or 1 argument ( other arguments will be ignored ).

A promise can succeed or fail only once, further resolve/reject will be ignored.
If you add a callback via `then`, `finally`, `catch` to a resolved promise, the callback will be called even if the promise has already settled.

It's preferred to pass new Error object into reject as you preserve the stack trace because those additional details are contained inside the Error object.
```ts
reject(new Error('xxx'));
xxx.catch(err => {
	console.log(err.message);
	console.log(err.stack);
});


reject('xxx');
// err is just a string
xxx.catch(err => {
	console.log(err);
});
```


Promises are eager, when you create a promise, executor is immediately invoked and promise is immediately resolved.
```ts
// promise is created, but resolving happens async
let p = new Promise((resolve, reject) => 
	resolve('hello'))
.then(x => console.log(x));

// first prints hej, then hello
console.log('hej');
```

##### Then
`then` method runs when a promise is settled, it has two arguments, a fn that runs when promise is resolved, and another fn that runs when promise is rejected. `then` always returns a promise.

If any exception is raised inside a promise or a then, it's treated as a rejection.
![[Pasted image 20240325171505.png|500]]


##### Catch
Catch is really just `.then(null, rejctFn)`

If any preceding then or the original promise itself is rejected, control goes to catch.


Quiz
![[Pasted image 20240325171150.png]]
A: Not same. If promise or then throws an error, it's caught by the catch in the first one.
In the second one, if `f1` throws an error, it's not caught by `f2`. `f2` only catches error from `promise`.


Quiz 2, will `catch` trigger?
```ts
new Promise(function(resolve, reject) {
    setTimeout(() => {
        throw new Error('Oops!');
    }, 1000);
}).catch(console.log);
```
No, throwing the error happens async, outside scope of original promise chain, so not caught by `catch`. We need to reject.
```ts
new Promise(function(resolve, reject) {
    setTimeout(() => {
        reject(new Error('Oops!'));
    }, 1000);
}).catch(console.log);

```


##### Rethrowing
If we handle error inside catch, control goes to closest `then`.
```ts
new Promise((resolve, reject) => {
    throw new Error('Oops!');
})
.catch(error => console.log('Error is handled'))
.then(() => console.log('Next succesful handler runs'));
```

If we rethrow inside catch, control goes to next closest error handler. This error handle maybe a `catch` or `then(resolve, reject)` where reject fn is run.
```ts
new Promise((resolve, reject) => {
    throw new Error('Oops!'); })
    .catch(error => throw(error))
    .catch(err => console.log(err + ' I caught it')); // catches the rethrown error
```

##### Finally
Runs some code after a promise is settled (resolved or rejected)
Has no arguments, so we don't know if promise resolved or rejected. Used for cleanup.

Result from the promise or one of its chains is passed through `finally` to the next handler. Only exception is when `finally` handler itself throws an error, then this error is passed instead of previous result. If we try to return anything in finally(), it's ignored.

Here `finally` executes first after promise is rejected. Then `catch` catches error throw.
```ts
new Promise((resolve, reject) => {
  throw new Error("error");
})
  .finally(() => alert("Promise ready")) // triggers first
  .catch(err => alert(err));  // <-- .catch shows the `error`


// Finally itself throws an error, this error is passed to catch
new Promise((resolve, reject) => {
  throw new Error("this error is ignored");
})
	// finally executes first, but throws an error
  .finally(() => { throw new Error("this error is caught"); }  ) // 
  // the error thrown by finally is passed to catch
  .catch(err => console.log(err));  // <-- this error is caught
```




#### Promise API

##### Promise.all
Takes an array of promises as input, also allows non-promise values. These are passed to the output array as they are. Return a new Promise object that will fulfil only if all input promises fulfils.

When a promise in `Promise.all` is rejected, `Promise.all` is immediately cancelled, however it does nothing to cancel other pending promises.
```ts
// syntax
Promise.all([a, b, c]).then(values => {
    // do stuffs
});


// Example
Promise.all([
	new Promise(resolve => setTimeout(() => resolve(1), 3000)),
	new Promise(resolve => setTimeout(() => resolve(2), 2000)),
	new Promise(resolve => setTimeout(() => resolve(3), 1000))
]).then(console.log) // prints [1, 2, 3]


// More complex example
let names = ['jason', 'remy', 'jess'];

let requests = names.map(name => fetch(`https://...${name}`));

Promise.all(requests)
.then(responses => {
	for (let resp of responses) {
		console.log(response.url);
	}
	return responses;
})
.then(responses => responses.map(resp => resp.json()))
.then(users => users.forEach(user => console.log(user)));
```


##### Promise.allSettled
Similar to `Promise.all`, instead of waiting for promises to resolve, it waits for promises to settle (either resolved or rejected).

##### Promise.race
Returns first promise that settles

```ts
Promise.race([
	new Promise(resolve => setTimeout(() => resolve(1), 1000)),
	new Promise((resolve, reject) => setTimeout(() => reject(new Error('whoops')), 2000)),
	new Promise(resolve => setTimeout(() => resolve(3), 3000)),
]).then(console.log) // print 1
```

##### Promise.resolve/reject
Creates immediately resolved or rejected promise.
```ts
// if we pass value, immediately resolves a promise
// note p is still a promise, we call `then` to access its value
let p = Promise.resolve(1);


// if we pass another promise, that promise is returned
let p2 = Promise.resolve(p);
```


#### Promisification
