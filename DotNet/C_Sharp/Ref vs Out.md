`Ref` must be initialised before passing into a method, not `out`.
`out` requires assigning a value before returning back from caller to callee, even if it's already `initialised` when calling the method.



To pass by reference, you can either use `out` or `ref` keyword.

The obvious use is for updating things like string (although it's reference type), bool, int.
Normally when you update the value inside the function it doesn't actually update the value of the caller variable.

However they can be used on reference types too, see below.
```csharp
// An example
void change_name(Book b1) {
	// This works
	b1.Name = "xxx";

	// This doesn't update b1, unless we set Book as ref
	b1 = new Book("Figma");
}
```


* We can use `out` if multiple values needs to be returned from a function.

 * In method overloading (same method name different signature), methods can't be overloaded if one method takes an argument as ref, and another takes an argument as out. It's only possible when one method takes input with ref/out and the other takes input without ref/out.

