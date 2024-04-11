There are 8 basic data types in javascript.

Almost everything in JS is an object ( contrast with everything is object in python ).

Only null, undefined, strings, numbers, boolean and symbols are not objects (these are primitive values/types). Anything other than a primitive value is an object.

#### BigInt
Not equivalent to number.
#### String
Note strings are immutable.
##### String template
```ts
`xxxx ${variable}`

// indexOf Looks for a substring, optional 2nd arg is start pos to search,
// returns the start index or -1 if not found
str.indexOf('widget', 2);

// includes, returns a bool
str.includes('hello world');

// slicing
str.slice(start, [,end]);

// negative slicing is possible
let str = 'stringify';
str.slice(-4, -1); // 'gif';

```

#### Number
```ts
// Round down, -1.1 becomes -2
Math.floor();

// Round up
Math.ceil();

// Round to nearest
Math.round(3.5); //4


let num = 12.345;
// round to nearest decimal ad returns result as string
num.toFixed(1) // '12.3'


// inf values
let inf = Infinity;
let minus_inf = -Infinity;


// reading string to number
// parseInt and parseFloat read until can't read anymore
parseInt('100px'); // 100
parseInt('a23'); // NaN
parseFloat('13.3.4.4'); // 13.3


// Random
// betweem [0, 1)
Math.random();
Math.max(1,2,3);
Math.min(1,2,3);
```

#### Boolean

#### Null
Null is a special value that does not belong to any other types, it's used to indicate the lack of value.

#### Undefined
Indicates a variable has not yet been initialised, or lack of return in a function.

#### Object
All other types are 'primitives', as their values can only contain a single thing. Objects can store more complex data.

#### Symbol
Only intended use is to avoid name clashes between properties.

Symbols are always different.

```ts
// Symbol represent a unique identifier
// Symbol takes in a 'descriptor', aka symbol name
let id1 = Symbol('id');

let id2 = Symbol('id');

// false, even though they have same description
id1 == id2; 




// Symbols dont auto convert to string
console.log(id1);

// alert doesn't work
alert(id1);

// alert now works too
alert(id1.toString());
```

###### Hidden properties
```ts
// imagine this is some 3rd party code
// we shouldn't add fields to it, as unsafe
let user_jason = {
	name: "jason"
};

// Imagine if we added a property id, 3rd party code may already has it
// Therefore gets overwritten!
// But symbols can't be accessed accidently due to being unique,
// 3rd party code prob won't even see it, so it's ok
// only we can access
let sid = Symbol("id");
user_jason[sid] = 'Our id value';



// Symbols in object literal
let id = Symbol('id');
let user = {
	name: 'jason',
	[id]: 123,
}
```

###### Global symbols
If for some reasons we want same name symbol to be the same entity.
```ts
let id = Symbol.for('id');

// this reads the id symbol again
let id_again = Symbol.for('id');

console.log(id === id_again); // true
```


#### Destructuring

##### Array destructuring

```ts
let arr = ['Jason', 'Xu'];

// Array destructuring
// assigning variables to arr elements
let [firstName, lastName] = arr;

let user = [];
[user.name, user.surname] = arr;


// default values
// name is Jason, surname is No
let [name = 'Guest', surname = 'No'] = ['Jason'];



// Unwanted elems can be thrown away via extra comma
let [firstName,, title]= ['Jason', 'skiped', 'Xu', 'also_skipped'];

// We can use array destructuring on any obj with iterable
let [a, b, c] = 'abc';
let [one, two, three] = new Set([1, 2, 3]);


// swap variable hack
[a, b] = [b, a]



// rest '...' operator
// You can use any name instead of 'rest' in place, juse use triple dots
// Can also use it in function parameter
let [name1, name2, ...rest] = ['Jason', 'Xu', 'rest1', 'rest2', 'rest3'];

rest.length // 3, ['rest1', 'rest2', 'rest3']
```


##### Object destructuring
We can use it to extra only what we need.
```ts
// Basic syntax
let { var1, var2 } = { var1: …, var2: … }


// Eg
let options = {
    title: 'Menu',
    width: 100,
    height: 200
}

// Note order does not matter, just name has to be same
// We can also have default value for any of them
let { height, width = 100, title } = options;


// if we want variable to have different name than the property
// { sourceProperty: targetVariable}
// w & v are the variables
let {width: w, height: h, title} = options;



// Rest '...'
let { prop: varName = default_value, ...rest } = object;
```

Nested destructuring
```ts
let options = {
    size: {
        width: 100,
        height: 200
    },
    items: ['Cake', 'Donut'],
    extra: true
};

// nested destructuring
let {
    // size from the object, assign var width and height
    size: {
        width,
        height
    },
    // items from the object, assign item1 and item2
    items: [item1, item2],
    title = 'Menu' // not present in object

}
// title => Menu
// width => 100
// height => 200
// item1 => Cake
// item2 => Donut
```

##### Parameter destructuring
Used when you are passing in parameters that are related to each other, pass them as object and destructure them in function signature

Can also use it to rename parameters.
```ts
function({ title, width: width_var_name, height: height_var_name, items }) {
	// do stuff
	console.log(width_var_name);
	console.log(height_var_name);
}

```


#### JSON
`JSON.stringify` converts obj to JSON string.
`JSON.parse` converts JSON string back to obj

Some properties are skipped
1. Function properties (methods)
2. Symbols
3. Properties that have values equal to `undefined`
4
```ts
let student = {
    name: 'John',
    age: 30,
    isAdmin: false,
    wife: null
};
let json = JSON.stringify(student);
alert( typeof jason); // string
/* 
{
"name": "John",
"age": 30,
"isAdmin": false,
"wife": null
}

JSON supports the following data types
Objects {..}
Array [...]
Primitives like strings, numbers, bool, null
*/




// Some properties are skipped by JSON.stringify()
let user = {
    sayHi() { alert("Hello" }, // ignored
    [Symbol('id')]: 123, //ignored
    something: undefined //ignored
};

// also nested objects are supported and converted automatically
let meetup = {
    title: 'Conference',
    room: {
        number: 23,
        participants: ['john', 'ann']
    }
};

JSON.stringify(meetup);
/* The whole structure is stringified:
{
	"title":"Conference",
	"room":{"number":23,"participants":["john","ann"]},
}
*/
```

Note if circular reference `JSON.stringify` will fail.
```ts
let room = {
    number: 23
};

let meetup = {
    title: "Conference",
    participants: ['john', 'ann']
};

// meet up now reference room
meetup.place = room;

// room now reference meetup
room.occupiedBy = meetup;

// circular reference, JSON stringify will fail
```


JSON.parse - to decode JSON string
```ts
let numbers = "[0, 1, 2, 3]";

// numbers will be an Array [0, 1, 2, 3]
numbers = JSON.parse(numbers);

// for nested objects
let userData = '{ "name": "John", "age": 35, "isAdmin": false }';
let user = JSON.parse(userData);

// property name and string value must contain double quotes
//  must contain double quotes

// only bare values, so no 'new' is allowed
```



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



// multi-dimensional array
let m = [[1, 2, 3]];


// Splice - delete continguous elements from given idx, modifies arr
arr.splice(start, [deleteCount], [insert1], [insert2]...);

let arr = ['i', 'study', 'js', 'right', 'now'];
// removes first 3 elements
arr.splice(0, 3, 'let\'s', 'dance');

// neg index also works
let arr = [1, 2, 5];
// 0 count so deletes nothing
// at idx -1, push 3 and 4
arr.splice(-1, 0, 3, 4);

// slice, returns sliced elements as new arr
arr.slice([start], [end]);



// indexOf, includes
// They work similar to String,


// find and findIndex
// They return the first item that satisfies the condition, else undefined
// find returns the actual element while findIndex returns the idx
arr.find(item => item.id == 1);

// Use find when you need to find the actual element, use includes if you only want to know whether the element exists and don't care about its value


// filter
// returns a new array that satisfies the filtering condition, so it returns an array
// find returns the first element




// sorts in-place
arr.sort();

// can also pass custom compare
function compare(a, b) {
	if (a > b) return 1;
	else if (a == b) return 0;
	else return -1;
}
arr.sort(compare);

// or shortcut way
arr.sort((a, b) => a - b);


// split() splits str into arr
// join() turns arr back to str
let arr = ['A', 'B', 'C'];
let str = arr.join(';'); // A;B;C



// Array.isArray()
Typeof {} // returns object
Typeof []  // returns object

Array.isArray( {} ) // false
Array.isArray( [] ) // true

```
Arrays methods are: 
shift/pop (pop from first & last)
unshift/push (append to start & end)

Note [[for...in vs for...of vs forEach()]], for...in loops all properties.




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
Inner exclamation checks element == false, outer exclamation flips the result
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
const sayHello = (function() {
	let message = 'Hello';

	function sayHello() {
		console.log('message');
	}

	return sayHello;
}) () // Immediately invoking means sayHello fn is returned to and assigned to sayHello variable.

```


##### First class functions
Means fn can be treated like objects, we can assign them, push them to arrays, pass them as parameters to other functions, and return fn from fn.


##### Higher order functions
Takes functions as arguments, or returns functions. E.g. map, filter, reduce, forEach.

```ts
// using reduce to sum up arr
let arr = [1, 2, 3, 4, 5];

// accumulator is initially 0
arr.reduce((accumulator, current) => accumulator + current ), 0)

// without initial value, the first element becomes the initial value


// forEach modifies arr in-place
// forEach modifies the existing array, map returns a new arr
arr.forEach((value, key) => arr[key] = value * 2)
```

#### Pass by value
JS always pass by value. It means JS copies the value of the passing variables into arguments for the function.

Hence any changes you make inside the function does not affect variables outside.

When you pass objects to function, you are passing the reference of that object, not the actual object. JS copies the reference to the argument. Now both the variable and argument are referencing the same object.

Therefore changes you make to the object inside the function will be reflected outside.

##### Garbage collections
JS uses reachability - if values are no longer accessible (no incoming references), then they are garbage collected.


![[Pasted image 20240324231825.png|180]]
![[Pasted image 20240324231831.png|240]]

In case of multiple references to an object, they must all be removed for the object to be considered unreachable.

![[Pasted image 20240324231948.png|400]]

![[Pasted image 20240324232012.png|400]]
![[Pasted image 20240324232025.png|400]]

###### Unreachable island
Setting family to null makes everything become unreachable.
In this case objects have incoming references, but they are no longer accessible so garbage collected.

![[Pasted image 20240324232113.png|400]]
