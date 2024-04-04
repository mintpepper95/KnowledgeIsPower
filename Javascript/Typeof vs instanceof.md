```ts
// typeof returns a string indicating the type
let foo = 'foo';
typeof foo; // string

let obj = {}; // If obj is any complex type, we will get obj
typeof obj; // object


foo instanceof String; // false, literal string like above and String() are seperate
foo instanceof Object; // false


// official behavioural error, should be null
typeof null; // object
```

* Use `instanceof` for custom types and complex built-in types (objects), it returns a boolean
* Use `typeof` for simple built-in types, it returns a string indicating the type
