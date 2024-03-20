
We can use `const` on value types and strings. Can't be used with reference types.
```csharp
const int x = 10;
const string msg = "Hello";
```

We use `readonly` on reference types. However, we can still modify object properties.
```csharp
readonly int[] numbers = {1, 2, 3};

// We can update 
numbers[0] = 10;
```


