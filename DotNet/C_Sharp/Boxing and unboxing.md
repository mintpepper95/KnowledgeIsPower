[[#Where are value and reference types stored?]]
[[#What is boxing?]]
[[#What is unboxing?]]
[[#Performance]]
[[#Pointers to ref types are stored on the stack]]
[[#Value types declared within a reference type are stored on the heap]]
[[#Generics avoid boxing/unboxing]]



C# has two types, value types and reference types.
When working with these types, sometimes you need to convert value types to reference types or vice versa.

#### Where are value and reference types stored?
Reference types are stored on the heap.
Value types are actual values stored on the stack.
#### What is boxing?

CLR boxes the value type, by creating a new System.Object on the heap and wraps the value of `i` in it, and then assigns its address to o, on the stack.

![[Pasted image 20240319190222.png]]
Here object o is an address and not value itself.
It's created on the stack.
```csharp
// Boxing
// assigning int to an object
// The box is an object, inside the box is a value type
// Note boxing is an implicit conversion, we don't need to cast explicitly
int i = 123;

object o = i;
```

![[boxing-operation-i-o-variables.gif]]


#### What is unboxing?
Extracting value type from an object and converting it to a value type.

![[unboxing-conversion-operation.gif]]

```csharp
// Unboxing, explicit conversion is required
// The item being unboxed must be a ref to an object that was created by boxing
object o = 10;
int j = (int)o;
```


### Performance
Both boxing and unboxing are expensive.
For Boxing, since we want to convert value type to a ref type.
- Allocating memory on the heap.
- Copying the value from the stack ( where value type lives ) to the newly allocated heap memory.

For Unboxing,
- Type checking at runtime to ensure the correct type is being unboxed.
- Copying the value from the heap back to the stack.

`ArrayList` only holds objects, so use `List<int>`, `List<T>` class is specific for a value type if `T` is a value.  `List` is created on the heap, and integers are stored directly on the heap within `List` contiguous block of memory as raw values.

![[1690826560012.png]]


#### Pointers to ref types are stored on the stack
```csharp
Building house = new Building();
```
For variables of reference types, the pointer that points to the actual object is stored on the stack. While the actual building object is stored on the heap.


#### Value types declared within a reference type are stored on the heap
Also when we assign int values to a List, those int values are stored on the heap as part of List's internal array storage. Therefore no boxing/unboxing, since int values are on the heap (on the heap in the array)


#### Generics avoid boxing/unboxing

With Generics you avoid doing boxing/unboxing since you are dealing with the parameter type `T`, which is a natural parameter that the compiler will replace it with the concrete type at compilation time without doing the boxing operation at runtime.