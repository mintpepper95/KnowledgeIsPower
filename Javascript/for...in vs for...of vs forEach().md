```ts
let arr = ['a', 'b', 'c'];
arr.test = 'bad';

// For...in does NOT ignore non-numeric properties,
// here it prints 'bad' as well
for (let i in arr) {
	console.log(i, arr[i])
}

// For...of ignores non-numeric properties
for (let i of arr) {
	console.log(i);
}

// forEach() accepts a callback fn, also ignores non-numeric properties
arr.forEach(x => console.log(x));
```
