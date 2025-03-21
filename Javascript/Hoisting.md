[[#What is hoisting? And what is the difference between between `var` hoisting and `let`, `const` hoisting?]]

[[#Are functions hoisted? What about arrow functions?]]

---
### What is hoisting? And what is the difference between  between `var` hoisting and `let`, `const` hoisting?

`var`, `let`, `const` and function declarations are moved to the top of their scope before code execution.

We can access `var` variables before initialisation, because they are auto assigned `undefined`.
However, trying to access `let` and `const` will result in ref error.

```ts
console.log(counter); // undefined
var counter = 1;


console.log(let_counter); // Ref error, can not access let_counter before initialisation
let counter = 1;
```

---
### Are functions hoisted? What about arrow functions?
JS also hoists function declarations ( moving them to the top of the script ).

```ts
// An example of function hoisting
let x = 10;
let y = 20;
add(x, y); // We called 'add' before its declaration

function add(x, y) {
	return x + y;
}


// not hoisted, value is undefined, 
// so if we run this we will get typeError
sayHello();

var sayHello() = function() { console.log('hello'); };
```

Anonymous functions are not hoisted.



