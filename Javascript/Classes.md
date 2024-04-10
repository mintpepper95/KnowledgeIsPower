[[#Explain how class keyword related to function?]]


#### Explain how class keyword related to function?
A user class creates a function named User, and stores its methods, getters, setters inside `User.prototype`

![[Pasted image 20240327194725.png]]

```ts
// You need to use the 'this' keyword inside the method, unlike C# which you don't
class Book {  
	 author;  
	 title;  
	 readCount;
	 
	 constructor(author, title) {  
	   this.author = author;  
	   this.title = title;  
	   this.readCount = 0;  
	 }
 }
 

class User {
    constructor(name) { this.name = name; }
    sayHi() { alert(this.name); }
}

// Class is a function
alert(typeof User); // function

// Or more precisely, the constructor method
alert(User === User.prototype.constructor); // true

// The methods are in User.prototype, e.g:
alert(User.prototype.sayHi); // alert(this.name);

// there are exactly two methods in the prototype
alert(Object.getOwnPropertyNames(User.prototype)); // constructor, sayHi
```

However, class is more than syntactic sugar.
```ts
// rewriting class User in pure functions

// 1. Create constructor function
function User(name) {
    this.name = name;
}

// a function prototype has "constructor" property by default,
// so we don't need to create it
// 2. Add the method to prototype
User.prototype.sayHi = function() {
    alert(this.name);
};

// Usage:
let user = new User("John");
user.sayHi();
```
There are some differences
1. A function created by 'class' is labelled by a special internal property `FunctionKind:classConstructor`. JS check for this property in several places. Unlike a regular function, it must be called with new.

3. Class methods are non-enumerable. The enumerable flag is set to false for all methods in the 'prototype'. Meaning for…in over an object won't get us the class methods.

5. Classes always use strict, all code inside class construct is auto in strict mode.


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
![[Pasted image 20240327200231.png]]
We can see that class fields are set on individual objects, not in the prototype`
```ts
class User {
    name = "John";
}

let user = new User();
alert(user.name); // John
alert(User.prototype.name); // undefined
```


![[Pasted image 20240327200651.png|600]]
A derived class such as Rabbit will have access to all methods in Animal. The `extends` keyword set `Rabbit.prototype.[[Prototype]]` to `Animal.prototype`.

Overriding a method - override a base class method with same signature in derived class

```ts
stop() {
	// can call base class method
	super.stop();
}
```

#### Why derived class must call super
In JS, distinction between derived constructor and other functions. Derived constructors have a special internal property called `[[ConstructorKind]]:"derived"`.
This affects its behaviour with `new` keyword. When a regular fn executes with `new`, it creates an empty object and assigns it to `this`.

Derived constructors don't do this, expects parent constructor to do this. So parent constructor has to be called, else `this` won't be created. If derived class has no constructor (can happen when parent has parameter less constructor), then parent constructor is called.

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
alert( article.title ); // Today's digest
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








