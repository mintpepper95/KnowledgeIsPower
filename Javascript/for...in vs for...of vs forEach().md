```ts
let arr = ['a', 'b', 'c'];
arr.test = 'bad';

// For...in does NOT ignore non-numeric properties, loops through both direct and inherited properties along the prototype chain
// here it prints 'bad' as well
for (let i in arr) {
	console.log(i, arr[i])
}

// For...of ignores non-numeric properties
for (let i of arr) {
	console.log(i);
}

// forEach() accepts a callback fn, also ignores non-numeric properties like for...in
arr.forEach(x => console.log(x));
```
