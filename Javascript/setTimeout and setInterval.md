[[#How do setTimeout and setInterval work in regards to event loop?]]
[[#setTimeout]]
[[#setTimeout is not immediate]]
[[#How do recursively setTimeout and setInterval differ?]]
[[#Passing class methods into `setTimeout`]]


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



#### Passing class methods into `setTimeout`
```ts
class Button {
    constructor(value) {
        this.value = value;

		// You would have to bind 'this' here, or use arrow syntax
		// this.click = this.click.bind(this);
    }
    // We lose 'this' in functions like setTimeout
    click() {
        alert(this.value);
    }
}


let button = new Button("hello");
// Here setTimeout() causes JS to use global scope. Meaning `this` inside click() will no longer refer to the class instance. Use arrow function instead
setTimeout(button.click, 1000); // undefined


// Fixed with arrow fn
class Button {
    constructor(value) {
        this.value = value;
    }

    // Arrow function, this will always ref the instance
    // So value will always be correct
    click = () => {
        alert(this.value);
    }
}
```
