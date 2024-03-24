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