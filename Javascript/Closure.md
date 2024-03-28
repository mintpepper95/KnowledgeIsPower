

#### What is closure?
It's when a function defined inside another function, can access the variables of the outer function even after the outer function has executed.

[[Let vs const vs var]]

##### Closure difference due to scoping
```ts
function makeArray() {
	const arr = [];
	
	for (var i = 0; i < 5; i++) {
		arr.push(function() {console.log(i)});
	}
	
	return arr;
}
const a = makeArray();
a[0](); // 5, because var is exist inside the fn, not just inside for-loop
a[1](); // 5, because var is exist inside the fn, not just inside for-loop


function makeArray() {
	const arr = [];
	
	for (let i = 0; i < 5; i++) {
		arr.push(function() {console.log(i)});
	}
	
	return arr;
}

const a = makeArray();
// `let` only exists inside the for loop
a[0](); // 0
a[1](); // 1
```
