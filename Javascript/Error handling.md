[[#Does try catch work with async code?]]



```ts
try {

} catch (ex) {

}
```


#### Does try catch work with async code?
No, try catch only works with sync code. By the time the callback fn of an async code is executed, the engine has already left try catch. To catch exception inside a scheduled fn like `setTimeout` or `setInterval`, try catch must be in the callback fn.

```ts
setTimeout(() => {
	try {
		// do something
	} catch (err) {
		// handle it
	}
}, 1000);
```

#### Throwing new error and Rethrowing
The rethrown error will be caught by outer level try catch
```ts
// Throwing new err
if (xxx) {
	throw new Error('throwing a new err');
}

// Rethrowing
setTimeout(() => {
	try {
		// do something
	} catch (err) {
		throw err;
	}
}, 1000);
```




