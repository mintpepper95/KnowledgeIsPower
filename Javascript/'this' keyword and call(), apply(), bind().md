[[#What are the difference between the 3 methods?]]





## What are the difference between the 3 methods?

We can set `this` to functions.

```ts
const greet = function () {
	console.log(this); // By deafult, 'this' will be the global object
}
```

### Call()
Replaces 'this' inside the fn with whatever you assign it.
![[Pasted image 20240324202405.png]]

```ts
// Args are the args you pass into the func.
// Now 'this' inside greet will refer to this name object
greet.call({name: 'Jason'})
```

### Apply()
Similar to `Call()`, excepts arguments are passed as an array.
![[Pasted image 20240324202650.png]]

### Bind()
Creates and returns another function which you can execute later, the returned function will have new context for `this`
![[Pasted image 20240324202653.png]]

```ts
let binded_greet = greet.bind({name: 'Jason'});

binded_greet();
```



