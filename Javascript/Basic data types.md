There are 8 basic data types in javascript.

Almost everything in JS is an object ( contrast with everything is object in python ).

Only null, undefined, strings, numbers, boolean and symbols are not objects (these are primitive values/types). Anything other than a primitive value is an object.

#### BigInt
Not equivalent to number.
#### String
##### String template
```ts
`xxxx ${variable}`
```

#### Number

#### Boolean

#### Null
Null is a special value that does not belong to any other types, it's used to indicate the lack of value.

#### Undefined
Indicates a variable has not yet been initialised, or lack of return in a function.

#### Object
All other types are 'primitives', as their values can only contain a single thing. Objects can store more complex data.

#### Symbol
New in ES6



#### Misc.

##### Array
```ts
// ways of creating array
let a = new Array();
a[0] = 'dog';
a[1] = 'cat';

let b = ['dog', 'cat'];

// Note array length is not necessarily equal to the number of items
a[100] = 'foxes';
a.length; // 101

// Undefined if you query non-existing index
a[101]; // undefined
```

##### Chained assignments
```ts
// chained assignments are evaluated right to left
let a, b, c;
a, b, c, d = 4;
```

##### Comma
```ts
// allows use to evaluate severa expressions, however only last result is returned, here a is 7
let a = (1 + 2, 3 + 4);
```


##### Double negation
Used to convert a non-boolean into a boolean
```ts
// first negation will return false, second negation will return the inverse, which is true
console.log(!!'asdadsa');
```


##### Null coalescing operator
Returns the first defined value.
```ts
// can be chained
string ans = Ans1 ?? Ans2 ?? Ans3;
```

##### Looping
`break` breaks out of current loop.
`continue` skips to the next iteration.

##### Switch
Switch equality check is always strict


##### Function expressions
When functions are created and assigned to variables directly.
```ts
let f1 = function() {
	console.log('hi');
}
```

Functions expressions are created only when execution reaches it and usable from then.
Function declarations can be called earlier than it's defined due to hoisting [[Hoisting]]

##### Immediately Invoked Function Expression
Can be used to create closure

```ts
// Note message not accesible outside the fn
const sayHello = ( function() {
	let message = 'Hello';

	function sayHello() {
		console.log('message');
	}

	return sayHello;
}) () // Immediately invoking means sayHello fn is returned to and assigned to sayHello variable.

```


##### First class functions
Means fn can be treated like variables, we can assign them, push them to arrays, pass them as parameters to other functions, and return fn from fn.


##### Higher order functions
Takes functions as arguments, or returns functions. E.g. map, filter, reduce, forEach.

```ts
// using reduce to sum up arr
let arr = [1, 2, 3, 4, 5];

// accumulator is initially 0
arr.reduce((accumulator, current) = > accumulator + current ), 0)



// forEach modifies arr in-place
arr.forEach((value, key) => arr[key] = value * 2)
```

#### Pass by value
JS always pass by value. It means JS copies the value of the passing variables into arguments for the function.

Hence any changes you make inside the function does not affect variables outside.

When you pass objects to function, you are passing the reference of that object, not the actual object. JS copies the reference to the argument. Now both the variable and argument are referencing the same object.

Therefore changes you make to the object inside the function will be reflected outside.






