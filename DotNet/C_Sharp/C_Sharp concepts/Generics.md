[[#What does a Generic class with Generic field look like?]]
[[#Generic constraints]]




---

#### What does a Generic class with Generic field look like?
Note T in Generics is not a reserved keyword. T or any name like Tkey, Tsession… will be the type parameter.

```cs
class DataStore<T> {
    public T[] data = new T[10];

    // Generic method with a generic arg
    public void Update(int position, T value) {
        if (position >= 0 && position < data.Length) {
            this.data[position] =  value;
        }
    }

	// Generic method with a generic return type
    public T GetData(int position) {
        if (position >= 0 && position < data.Length) {
            return this.data[position];
        } else {
            // return a null T
            return default(T);
        }
    }
}

// instantiate generic class
DataStore<string> store = new DataStore<string>();
store.Data = "Hello Seattle";
```

---

#### How does Generic improves performance by avoiding boxing and unboxing?

Recall boxing is converting a value type to a ref type.
```cs
int number = 42;
object boxedNumber = number; // Boxing
```

Unboxing is vice versa - converting ref type to value type
```cs
int num = (int) boxedNumber; // Unboxing
```

Boxing a value means we have to allocate some space on the heap (for the object) and then copy the value into that object.

A generic list like `List<T>` lives on the heap. Its elements are also directly stored on the heap in the array, instead of being boxed and having the array containing references to the boxes.

`List<int>`
`[1, 2, 3]`

`ArrayList`, non-generic, stores references to objects
`[*a, *b, *c ]`

The price of boxing is therefore about allocating new storage on the heap and copying the int to the new storage, and having `ArrayList` containing references to those storage places.

---

#### Generic constraints
It provides additional requirements to the type. 

Here it means `T` can only be a reference type
`DataStore<T> where T: class`

Some other ones


`DataStore<T> where T: new()`
Must be a ref type with a public parameter less constructor

`DataStore<T> where T: SomeBaseClass`
Must be a derived from `SomeBaseClass`

