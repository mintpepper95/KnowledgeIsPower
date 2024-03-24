Recall in js primitives are just values.
So how can we access properties if they are just values?
```
let str = 'Hello';
str.toUpperCase();
```
1. str is a primitive. In the moment of accessing its property, a special object is created that knows the value of str, and has useful methods like toUpperCase().
2. Method runs and returns a new string.
3. The special object is destroyed, leaving str alone.

What happens in the following?
```ts
let str = 'hello';
str.test = 5;
console.log(str.test);
```
A: When you attempt to assign a property (`test`) to a primitive string using dot notation or bracket notation, JS temporarily converts the primitive string into a String object, adds the property to that object, and then discards the object. This means that the property `test` is added to a temporary String object, not to the original primitive string `str`.




