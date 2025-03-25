Always use strictly equality `===` in JS.

Loose equality performs type coercion before comparison.

```ts
// Regular comparison
let a = '0';
let b = 0;

// Double equality performs type coercion before comparison
// Here 'a' is converted to number before comparison.
a == b; // true


let a = 'ABC';
let b = 1;
let c = '1'

a > b; // false, since one of them is a number, js will attempt to convert other into a number, which makes 'a' NaN since invalid number, which is smaller than everything, so false

a > c // true, since both string, compared char by char, in Unicode numbers are before letters, so 'A' > '1' 


let d = { name: 123 };
d > 1; // false, we convert object to primitive first, in this case, '[object Object]', which is a string. We will then try to convert it to a number, which becomes NaN, NaN is smaller than 1

// Note in Unicode, '[' is smaller than 'A' and thus all letters, but it's greater than numbers
d > "123"; // true, as we first convert d to the string '[object Object]'

// If the result of `valueOf()` is not a primitive (like a string or number), it will fall back to the `toString()` method.


// A few words about null
// ">"  performs type coercion which coerces null into a number
null >= 0; // true

// No type coercion here
null == 0; // false

null == null && null == undefined; // true

// false, '0', [], "" all coerces to 0 when coercing into number



"6xas" < Infinity
// "6xas" converts to NaN, any comparison with Nan is always false, except below

NaN !== NaN; // true

```

With strict equality, it first checks whether the type differ. If differ return false. If the type match, then it checks if the values are the same or not.

Any falsy values such `0`, `undefined`, `null`, `NaN`  becomes false when casted to bool.
`NaN` is a number type value, to indicate not a number, it's also smaller than all other numbers


1. If one operand is a string and the other is a number, JavaScript will typically attempt to coerce the string into a number before making the comparison.
    
2. If one operand is an object and the other is a primitive (like a number or string), JavaScript will attempt to call the `valueOf()` or `toString()` methods of the object to convert it to a primitive value.
    
3. If both operands are objects (or arrays), JavaScript will compare their references, not their contents.


