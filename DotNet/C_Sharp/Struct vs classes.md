Struct are NOT immutable by default.
However since it's a value type, if you pass a struct to a method and update it inside the method, it won't change the actual value unless we use `ref` keyword.

#### Struct
Value type, so faster than a class object. 

- Struct can't include a parameterless constructor/destructor
- Struct can implement interfaces just like class
- Struct cannot inherit
- Struct members can't be abstract, sealed, virtual or protected

A struct CAN contain properties, auto-properties, methods just like classes

```csharp
struct Coordinate {
	public int x;
	public int y;
	
	public Coordinate(int x, int y) {
			this.x = x;
			this.y = y;
	}

	public void SetOrigin() {
		this.x = 0;
		this.y = 0;
	}

	public static Coordinate GerOrigin() {
		return new Coordinate();
	}
}

var point = new Coordinate(10, 20);
```
