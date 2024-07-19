C# defaults to pass by value, even for a reference object.
In this case, it means the reference itself is copied.
So because the copied reference still points to the same object, hence we can modify it.
However if we change the reference to point to a different object, original is not affected outside the function.

```cs
public Book {
	public string name;

	// FAILS
	public static void ChangeName(Book b1, string new_name) {
		// below will update successfully
		b1.name = new_name; 

		// this won't update b1 in the caller
		b1 = new Book { name = "figma" };
	}

	// WORKS
	public static void ChangeNameRef(ref Book b1, string new_name) {
		// below will update b1 in the caller
		b1 = new Book { name = "figma" };
	}
}

```