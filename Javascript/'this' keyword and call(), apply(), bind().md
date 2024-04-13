[[#What are the difference between the 3 methods?]]



## What are the difference between the 3 methods?

They all allow us to set `this` to functions.
### Call()
Replaces 'this' inside the fn with whatever you assign it.
![[Pasted image 20240324202405.png]]

```ts
const greet = function () {
	console.log(this); // By default, 'this' will be the global object
}

// Args are the args you pass into the func.
// Now 'this' inside greet will refer to this name object
greet.call({name: 'Jason'});


// However if we don't pass `this` to call(), then `this` will still be global object
var data ='Wisen';
function display() {
	console.log(this.data);
}

// recall if 'this' not passed,
// it's bound to the global object
display.call(); // Wisen
```

###### Another example of call
```ts
function Product(name, price) {
    this.name = name;
    this.price = price;
}

function Food(name, price) {
    // calls Product to assign name and price
    Product.call(this, name, price);
    this.category = 'food';
}

let food = new Food('cheese',5);
// food has properties name and price
console.log(food.name);
```
###### And Another
```ts
function say(phrase) {
    console.log(this.name +': ' + phrase);
}
let user = { name: 'Jason' };
say.call(user, 'Hello'); // Jason: Hello
```


### Apply()
Similar to `Call()`, excepts arguments are passed as an array.
![[Pasted image 20240324202650.png]]

```ts
let numbers = [1, 3, 5, 7];
// second arg to apply() must be an array
let max = Math.max.apply(null, numbers); // 7
```
### Bind()
Creates and returns another function which you can execute later, the returned function will have new context for `this`
![[Pasted image 20240324202653.png]]

```ts
let binded_greet = greet.bind({name: 'Jason'});

binded_greet();
```






