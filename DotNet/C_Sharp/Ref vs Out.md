`Ref` must be initialised before passing into a method, not `out`.
`out` requires assigning a value before returning back from caller to callee, even if it's already `initialised` when calling the method.


A data is a value type if it holds the actual data within its memory.

When you pass a value type from one method to another, system creates a separate copy hence value types aren't modified.

Unlike value types, reference type contains the reference. Eg. String, Array, Class, Delegate.

When you pass a reference type from one method to another, it can be changed.

Note C# defaults to pass by value, meaning that even for a reference object, if you change the data inside the object, it will be visible to the caller. But if you change the passed argument to refer to a different object, then it will not be visible to the caller.





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


```cs
exmaple usage of out
static void readPerson(out string name, out int age) {
	name = "Jackson";
	age = 10;
}

// usage, here it reads the name and age and assign it to the passed in variables
string initialName;
int initialAge;
readPerson(out initialName, out initialAge);


// same as above even if we assign the variabnles initially
string initialName = "Tom";
int initialAge = 11;
readPerson(out initialName, out initialAge);

```

