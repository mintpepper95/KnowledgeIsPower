[[#Creating an object, add and remove properties]]
[[#Can we use reserved key words like for, let, return for Object keys and variables?]]
[[#Accessing objects with square brackets and dot]]
[[#hasOwnProperty vs In]]
[[#Object.assign vs Object.create]]
[[#Copying nested objects]]
[[#'this' keyword]]
[[#Optional chaining]]

---

#### Creating an object, add and remove properties
Object can be created with constructor syntax or object literal syntax.

```ts
let user = new Object();
let user2 =  {
	name: 'john',
	age: 30,
	'hello seatlle': 104,
};

// We can also add/remove property
user.isAdmin = true;
delete user.age;

// Properties aren't ordered
// Keys can only be strings and numbers
```


---

#### Can we use reserved key words like for, let, return for Object keys and variables?
Objects can use reserved words like `for`, `let`, `return` as keys, but we can't use reserved words for variables. 


---

#### Accessing objects with square brackets and dot
Square brackets allows us to use string, result of expressions or variables as key.
Dot has to be the property name, else value will be undefined. 
So dot for static keys, square brackets for dynamic keys.

---
#### hasOwnProperty vs In
```ts
// If key is found somewhere in the prototype chain
'a' in object;

// If you just want to test for properties directly present in the object and not inherited properties along the object's prototype chain
obj.hasOwnProperty('key');


// Example
function TestObj(){
    this.name = 'Dragon';
}
TestObj.prototype.gender = 'male';

let o = new TestObj();
o.hasOwnProperty('name'); // true
'name' in o; // true

o.hasOwnProperty('gender'); // false
'gender' in o; // true
```


---
#### Object.assign vs Object.create
For merging properties to target
```ts
let user = { name: 'Jason' };
let p1 = { canView: true };
let p2 = { canEdit: true };

// Copies the properties to user
// If same name property exist in user, it's overwritten
Object.assign(user, p1, p2);

// Now user has properties 'name', 'canView' and 'canEdit'

// If properties are references to other objects instead of primitives, we need to deep copy. Otherwise changes to original property will affect the property in the copied object.
let user = {
	size: {
		height: 182,
		weight: 65,
	}
}

// copies user to an empty object
let clone = Object.assign({}, user);

// This will update the property in clone as well, since the copied property is a reference
user.size.weight++;



// Object.assign vs Object.create
// Object.assign(target, source, sourc2...) is for copying all enumerable own properties from source to targert
// Object.create() creates a new object that inherits from the prototype object

let person = {
	name: 'Jason'
}

let new_person = Object.create(person);

new_person.__proto__ === person;
// name property does NOT exist in new_person, it exist in person, and JS needs to follow the prototype chain to find it

```

---
#### Object.entries
```ts
// returns a list of key value pairs
Object.entries(user);
```

---
#### Copying nested objects
Both `Object.assign` and  `{...obj}` can only copy non-nested objects.
We can use `JSON.stringify()` to copy nested objects without functions.

---
#### 'this' keyword 
In js, `this` is evaluated at run time, it's value is the object before the dot.
Unlike in other languages, where `this` always ref the object where method is defined.
Arrow fn have no `this`, `this` inside arrow fn will be taken from outer scope.


```ts
function makeUser() {
    return {
        name: 'John',
        ref: this
    };
}
// When you call makeUser, `this` will be the global object
// So user.ref will be the global object
let user = makeUser();
// Since global object has no name property, prints undefined
console.log(user.ref.name);



// Now works properly
function makeUser() {
	return {
		name: 'John',
		// now `this` will equate to the object that's calling ref
		ref() {
			return this;
		}
	}
}

let user = makeUser();
user.ref().name; // John
```

---
#### Optional chaining
In `C#` optional chaining returns null.
In js, optional chaining returns undefined.

```ts
// Call a fn that may not exist, if not exist returns undefined
user.sayHi?.();

// Call a property that may not exist, if not exist returns undefined
user?.[key];
```


