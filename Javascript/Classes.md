[[#Explain how class keyword related to function?]]

---

#### Explain how class keyword related to function?
A user class creates a function named User, and stores its methods, getters, setters inside `User.prototype`

![[Pasted image 20240327194725.png]]

```ts
// You need to use the 'this' keyword inside the method, unlike C# which you don't
class Book {
	 constructor(author, title) {  
	   this.author = author;  
	   this.title = title;  
	   this.readCount = 0;  
	 }
 }
 
class User {
	// constructor exists in User.prototype
    constructor(name) {
	    // name exists in User instance 
	    this.name = name; 
	}

	// hello exists in User instance
	hello = () => {};

	// sayHi exists in User.prototype
    sayHi() { // in js classes, methods don't require 'function' keyword
	    alert(this.name); 
	}
}

// Class is the constructor function
alert(typeof User); // function
alert(User === User.prototype.constructor); // true

// The methods are in User.prototype, e.g:
alert(User.prototype.sayHi); // alert(this.name);

// there are exactly two methods in the prototype
// `getOwnPropertyNames` returns all properties of that object, while Object.keys() returns only the enumerable ones
console.log(Object.getOwnPropertyNames(User.prototype)); // constructor, sayHi

console.log(Object.getOwnPropertyNames(user)); // name and hello
```

#### Class with pure functions
```ts
// rewriting class User in pure functions
// note we don't need to create constructor

// 1. Create constructor function
function User(name) {
	// This property belongs to an instance of User
    this.name = name;
    // This method also belongs to an instance of User
	this.goodbye = () => console.log(`goodbye ${this.name}`);
}


// 2. Add the method to prototype
User.prototype.sayHi = function() {
    console.log(this.name);
};
// User.prototype will have sayHi, but it won't have `name`
// User instances will have name and goodbye()

// Usage:
let user = new User("John");
user.sayHi();
```


#### class vs pure functions
There are some differences
1. A function created by 'class' is labelled by a special internal property `FunctionKind:classConstructor`. JS check for this property in several places. Unlike a regular function, it must be called with new or else `TypeError`. For functions, if we call without `new` keyword, there won't be values returned.

2. Class methods are non-enumerable. The enumerable flag is set to false for all methods in the `prototype` property. Meaning for…in over an object won't get us the class methods. Eg. For `User`, `sayHi` won't show up in a for...in loop (`name` and `goodbye` will ) because class methods are non-enumerable.

3. Classes always use strict, all code inside class construct is auto in strict mode.


We can assign class to variables and return class.
```ts
// Assigning class to a variable
let User = class {
    sayHi() {
        alert('Hello');
    }
};
new User().sayHi();


function makeClass(phrase) {
    // returns the class
    return class {
        sayHi() {}
    };
}
let ReturnedClass = makeClass();
let r = new ReturnedClass()
```




#### Class inheritance and prototype

Animal prototype
![[Pasted image 20240327200231.png]]

Rabbit prototype
![[Pasted image 20240327200651.png|600]]

Above diagram is saying `Rabbit.prototype` will point to a prototype, which contains constructor and all class methods and variables. Same with `Animal`.

`Rabbit` instance `__proto__` will also point to the prototype.

`Rabbit.prototype`'s prototype will point to `Animal.prototype` as `Rabbit` extends `Animal`. Meaning `Rabbit` will have access to all methods in `Animal`. The `extends` keyword set `Rabbit.prototype.[[Prototype]]` to `Animal.prototype`.



```js
// Overriding a method - override a base class method with same signature in derived class
stop() {
	// can call base class method
	super.stop();
}
```

#### Why derived class must call super
In JS, distinction between derived constructor and other functions. Derived constructors have a special internal property called `[[ConstructorKind]]:"derived"`.
This affects its behaviour with `new` keyword. When a regular fn executes with `new`, it creates an empty object and assigns it to `this`.

Derived constructors don't do this, expects parent constructor to do this. So parent constructor has to be called, else `this` won't be created. `If derived class has no constructor (when parent has parameter-less constructor), then parent constructor is called.`

There is no difference between no constructor and empty parameter-less constructor.

```ts
class Animal {
    name = 'animal';
    constructor() {
        alert(this.name); // (*)
    }
}
class Rabbit extends Animal {
    name = 'rabbit';
}

new Animal(); // animal
new Rabbit(); // animal
```

#### Static methods and properties

Static properties and methods are stored on the class itself, not in `Class.prototype`.

```ts
class Article {
    constructor(title, date) {
        this.title = title;
        this.date = date;
    }

	// static property
	static publisher = 'Jason';

    static createTodays() {
        // Remember, `this` is Article in static methods!!
        return new this("Today's digest", new Date());
    }
}

let article = Article.createTodays();
console.log(Article); // You will see `publisher` and `createTodays`
console.log( article.title ); // Today's digest

```


#### Prototype vs proto and Object.getPrototypeOf()

To put simply, prototype exists on constructors. `__proto__` exists on instances.

`prototype` is a property of constructor functions and classes.
It's used to define shared methods and properties that instances inherit.
So any instances created with `new className()` will inherit from `className.prototype`.

`__proto__` is a property of instances, that points to the prototype of their constructor.

```js
user = new User();

user.__proto__ === User.prototype; // true
User.prototype.__proto__ = Object.prototype; // true
Object.prototype === null; // Object.prototype is the root prototype


// Prototype exist on functions as well
function Car(model) {
	this.model = model;
}

Car.prototype.constructor; // function Car(model)
car.__proto__ == Car.prototype; // true


// obj don't have prototype, will be undefined
obj.prototype === Object.getPrototypeOf(obj) // false

// think of getPrototypeOf() as __proto__, but readonly
// __proto__ is now deprecated
Object.getPrototypeOf(obj) === MyConstructor.prototype // true
```






#### Access modifiers
By default, all members in TS and JS are public.

JS has no access modifiers, unlike TS.

In C#, by default, classes are internal and class members are private.


#### Extending built-in class

```ts
// Eg. extend Array
class PowerArray extends Array {
    isEmpty() {
        return this.length == 0;
    }
}

arr = new PowerArray(1, 2, 4, 10);
// built-in methods like filter and map return new objects of the inherited type, in this case Power Array
let filtered = arr.filter(item => item < 10);
filtered.isEmpty();
```








