JS is an interpretative language that doesn't need compiling. JS is dynamically typed like python. Web browsers have embedded 'engines' that allows javascript to run. The engine parse the js code, compiles the script to machine code for execution.

In-browser js has no access to OS functions. Eg, read/write abitrary files on the hard disk. It's very limited (eg. Drag and drop files into browser).

Different tabs and windows generally do not know about each other and have their own execution environments.  Meaning js from one page may not access the other if the other is from a different site (Same origin policy).

Although js can easily send/get request from server where the current page came from. Its ability to receive data from other sites are crippled for safety concerns (requires explicit agreement from the other side).


[[Let vs const vs var]]
[[Hoisting]]
[[Typeof vs instanceof]]
[[Equality in JS]]
[[Basic data types]]
[[Closure]]
[[for...in vs for...of vs forEach()]]
[['this' keyword and call(), apply(), bind()]]
[[Async and Await]]
[[Classes]]
[[Event loop]]
[[How are methods implemented for primitives]]
[[Objects]]
[[Promise]]
[[setTimeout and setInterval]]



Browser stuff
[[Browser Storage Mechanism]]
[[What happens when you type a URL into the browser]]
[[Webpack notes]]


#### Common JS methods
##### Converting between different primitive types
```ts
// string to number
let s = Number('123'); // 123, if pass other than number then NaN
let s2 = parseInt('123px'); // 12

// number to string
let s3 = 123
s3.toString(); // 123

// We can use the double exclamation marks to convert truthy/falsy values to booleans

// Min and max, if array needs to unpack
Math.min(12,2,3,4, -11);
Math.max(12,2,3,4, -11);

// Infinity
NUMBER.POSITIVE_INFINITY;
NUMBER.NEGATIVE_INFINITY;
```

##### Common array operations
```ts
let s = [1, 2, 3, 4, 5];

// Note JS don't support negative indexing
s[-1]; // Doesn't work!!!


// copy/slice the entire array
let s1 = s.slice();
// slice from and to
let s2 = s.slice(start, end);


// if insert to end
s2.push(6);

// if insert to start
s2.unshift(0);

// pop from end
s2.pop();

// pop from start
s2.shift();

// insert element or delete ith position in array
// note splice works on original arr
arr.splice(start_pos_to_modify, elements_to_delete, new_element_1, new_element_2...);

let arr = [1, 2, 3, 4, 6];

// [1, 2, 3, 4, 5, 6]
arr.splice(4, 0, 5);

// [1, 6]
arr.splice(1, 4);
```


##### Object.keys, Object.values, Object.entries, Object.fromEntries,

Previously we saw methods like map.keys(), map.values(), map.entries() which return iterable. These methods are generic, so if we ever create a data structure of our own, we should implement them too.


Plain objects also support similar methods

Object.keys(obj) - returns an array of keys
Object.values(obj) - return an array of values
Object.entries(obj) - returns an array of `[key, value]` pairs, you can pass it an object or an array, when you pass an array you will get index as key

```ts
let user = {
    name: "John",
    age: 30
};

Object.keys(user) // ["name", "age"]
Object.values(user) // ["John", 30]
Object.entries(user) // [ ["name","John"], ["age",30] ]
// We can use Object.entries to loop through [idx, value] of arr

// loop over values  
for (let value of Object.values(user)) {  
  console.log(value); // John, then 30  
}
// Just like for…in loop, above methods ignore symbolic properties  // (properties that use Symbol(..) as keys)



// loop through index and value of an arr
let arr = [20, 30, 40, 50];
for (const [key, value] of arr.entries()) {
	console.log(key, value);
}
```


###### Transforming objects
Objects lack many methods that exist for arrays, eg. Map, filter and others.
```ts
let prices = {
    banana: 1,
    orange: 2,
    meat: 4
}


let doublePrices = Object.fromEntries(
    // .entries() convert to array first
    // use array methods on that array, eg. map
    // use Object.fromEntries(array) on resulting array to turn it back to object
    // note destructure correct;y with square brackets
    Object.entries(prices).map(([key, value]) => [key, value * 2])
);

```

##### Map

Iteration over Map and Set is always in insertion order, so these collections aren't really unordered, but we can't reorder them or directly get an eleme by its position.

```ts
// Map([iterable]), receives array of iterable whose elements are size 2
let map = new Map([[1, 2], [3, 4]]);

// Basic map example
let count = new Map();

// Note set returns the map itself, so we can chain set()
count.set('key', 'value');
count.set('key', 'value').set('key2', 'value2').set('key3', 'value3');


// Map[key] isn't the right way to use a Map
// Because even though it works, it is treating map as a plain JS object, // so it implies all corresponding limitations (eg. only string/symbol keys)
count.get('key'); // retrieve value, if not found returns undefined

count.has('key'); // check if key exists, returns boolean

count.delete('key'); // returns true if deleted else false

// Map is an array of iterable of size 2, we can loop it
for (let [key, value] of count) {
	console.log(`key is ${key}, value is ${value}`);
}

// You can also use foreach on map, note value first key second
// value first and key second for arr.ForEach() as well
count.forEach((value, key) => console.log(key, value));


// We can't use Object as a key for an Object, only primitive values, fine for Map.
let john = { name: 'john' };
let ben = { name: 'ben' };
let obj = {};

obj[ben] = 123;
obj[john] = 234; // ben object also gets replaced, because key is string due to coercion when we set the keys, "[object Object]"
```



##### Set
```ts
let set = new Set(['oranges', 'apples', 'pear'])


for (let val of set) alert(val);



// common methods
set.add(item);
set.has(item); // returns boolean
set.delete(item); // removes the element
set.clear(); // empties set


// note value argument appears twice in forEach, that's for compatibility with Map
set.forEach((value, valueAgain, set) => {
    alert(value);
});

set.keys() // returns an iterable
set.values() // same as set.keys(), for compatibility with Map
set.entries() // returns entries [value, value], for compatibility with Map

```

##### Radios

Creating a basic set of radios, they must have same name if you want to define a radio group, where you can only select one radio from a group.

The id of radio is used by label to associate labels with radio buttons.

If we omit value for radios, then when submitting form data, the value `on` is assigned.

Note we have a default checked state, to select a radio button by default.

```html
     <div>
      <input type="radio" name='shoes' id ='adidass' value='adidas' checked />
      <label for="adidass">Adidas shoes</label>

      <input type="radio" name='shoes' id='nike' value='nike'>
      <label for="nike">Nike shoes</label>

      <input type="radio" name='shoes' id="puma" value="puma">
      <label for="puma">Puma shoes</label>
     </div>
```

Another example with checkbox
```js
// note label.for and input.id is same, linked, clicking on label will toggle checkbox
<label for='bool'>Please accept this</label>
<input id='bool' type='checkbox'>



let checkbox = document.querySelector('input[type="checkbox"][id="bool"]');

// add event listener
checkbox.addEventListener("change", (ev) => {
  console.log(ev.target.checked);
});
```
###### Select radio using query selector

For a radio group, if we want to add same event listeners to all of them. We can achieve this with `querySelectorAll` then `forEach`

To manually reset radio group, we can loop through all the radios, and set `checked` to false. Or just reset entire form if it's in a form.

```js
// getting radio group
let radios = document.querySelectorAll('input[type="radio"][name="shoes"]);

radios.forEach(radio => {
  radio.addEventListener("change", (ev) => {
	  // with change, ev.target.checked will always be true since 'change' only fires when new selection is made
    if (ev.target.value === "nike") {
      console.log("nike is selected");
    } else {
      console.log("nike is de-selected");
    }
  });
});


// query selector using 'not'
// grabs all inputs, ":not()" excludes checkbox type inputs
document.querySelectorAll('input:not([type="checkbox"])');


// css query, grav all number class elments during hover, exclude ones with 'active' class
.number:hover:not(.active) {
  background-color: cornflowerblue;
}
```


##### classList
Allows you to add and remove classes on elements

`classList` is more more flexibility than `className`, `className` will treat the class as an string whereas `classList` allows you to modify individual classes.
```js
// Recall .class to select based on class, #id to select based on id
const div = document.querySelector('div');

// add a class
div.classList.add('highlight');

// remove a class
div.classList.remove('hidden');

// toggle a calss
div.classList.toggle('dark-mode');



// Input stuff
const sampleInput = document.querySelector('#ln');

// disabling
sampleInput.disabled = true;

// focusing
sampleInput.focus = true;

// accessing parent element, null if node has no parent
sampleInput.parentElement.classList.add('invalid');

// accessing style
sampleInput.parentElement.style.color = 'red';

// visiblity
sampleInput.style.visibility = 'hidden';   

// updating text
// in most cases, use textContent as it's faster, innerText retrieves only visible text, textContent retrieves all texts, whether text is rendered or not (text maybe not rendered due to display: hidden)

// use textContent when you are trying to alter text, use innerText for grabbing text
sampleInput.parentElement.innerText = 'updated';


// hide and show elements
sampleInput.style.display = "block";

sampleInput.style.display = "none";


```



