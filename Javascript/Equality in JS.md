Always use strictly equality `===` in JS.

Loose equality performs type coercion before comparison.

```ts
// Regular comparison
let a = '0';
let b = 0;

// Double equality performs type coercion before comparison
// Here 'a' is converted to number before comparison.
a == b; // true
```

With strict equality, it first checks whether the type differ. If differ return false. If the type match, then it checks if the values are the same or not.

Any falsy values such `0`, `undefined`, `null`, `NaN`  becomes false when casted to bool.
`NaN` is a number type value, to indicate not a number



1. If one operand is a string and the other is a number, JavaScript will typically attempt to coerce the string into a number before making the comparison.
    
2. If one operand is an object and the other is a primitive (like a number or string), JavaScript will attempt to call the `valueOf()` or `toString()` methods of the object to convert it to a primitive value.
    
3. If both operands are objects (or arrays), JavaScript will compare their references, not their contents.