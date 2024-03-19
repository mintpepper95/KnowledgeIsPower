C# has two types, value types and reference types.
When working with these types, sometimes you need to convert value types to reference types or vice versa.

Reference types are stored on the heap.
Value types are actual values stored on the stack.
#### What is boxing?

CLR boxes the value type, by creating a new System.Object on the heap and wraps the value of `i` in it, and then assigns its address to o, on the stack.

![[Pasted image 20240319190222.png]]
Here object o is an address and not value itself.
It's created on the stack.
```csharp
// Boxing
// in C#, if you take a struct ( which typically lives on the stack )
// and assign it to an object
// The box is an object, inside the box is a value type
object o = 42;
object o2 = 42;

// Not same object, we are comparing the boxes (references) not the values, doesn't print
if (o == o2) {
    Console.WriteLine("same");
}

// To check the inside content and not the box itself, use object.Equals, prints
if (o.Equals(42)) {
    Console.WriteLine("Same!");
}

// In C# 8, 'is' works with boxes
// o is an Object containing 10, prints
if (o is 10)
    Console.WriteLine("Yes");
```


#### What is unboxing?
Extracting value type from an object and converting it to a value type.


![[Pasted image 20240319190513.png]]
```csharp
// Unboxing
object o = 10;
int j = (int)o;
```


