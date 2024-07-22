Class
[[#What are the default access modifiers of class and its members?]]
[[#What is a class destructor? Can it take in parameters and return values?]]
[[#Inheritance]]
[[#What is object initialiser?]]

Partial Class
[[#Why use partial classes?]]
[[#What is a partial method?]]

Static
[[#Explain static class, fields, methods, constructor]]

Polymorphism
[[#What is static polymorphism?]]
[[#What is dynamic polymorphism?]]
[[#Explain Virtual/Override vs New]]


---

### Class
#### What are the default access modifiers of class and its members?
In C#, default access modifier for class itself is internal. Default access modifier for class members are private.

#### What is a class destructor? Can it take in parameters and return values?
A class destructor is a special member function that executes when the class instance goes out of scope, it can not take parameters nor return anything.

```cs
~Line() {
	Console.WriteLIne("Object is being destroyed");
}
```


#### Inheritance
C# does not support multiple class inheritance, only multiple interfaces.
```cs
public class Rectangle {
	double length;
	double width;

	public Rectangle(double length, double width) {
		this.length = length;
		this.width = width;
	}

	public double GetArea() {
		return this.length * this.width;
	}
}


public class TableTop : Rectangle {
	double cost;
	double serialNumber;
	string brand;
	
	public TableTop(double l, double w, double cost) : base(l, w) {
		this.cost = cost;
	}

	public GetCost() {
		return this.cost * this.GetArea();
	}
}
```

#### What is object initialiser?
Object initialisers allow you to set properties/fields after constructor has been called.

A benefit about using object initialiser is atomicity. This ensures object is never partially initialised.

```cs
var tableTop = new TableTop(10, 8, 5) {
	serialNumber = 101;
	brand = 'Hi'
};
```


---

### Partial Class
#### Why use partial classes?
We can split implementation of a class, struct, method, interface in multiple cs files.

It's mainly done for readability.

All partial class definitions must be inside same name and assembly.
They must have same access modifiers.
```cs
// This file contain all members
public partial class Employee {
	public int EmpId { get; set; }
	public string Name { get; set; }
}


// This file contain all methods
public partial class Employee {
	public Employee(int id, string name) {
		this.EmpId = id;
		this.Name = name;
	}
}
```

##### What is a partial method?
Partial class can contain a method split into two cs files.
One file contains the signature.
The other contains the optional implementation.
```cs
public partial class Employee {
	public int EmpId { get; set; }
	public Employee() {
		GenerateId();
	}
}


public partial class Employee {
	public GenerateId() {
		this.EmpId = random();
	}
}
```

---


### Static modifier
#### Explain static class, fields, methods, constructor
In C#, all members inside a static class must be static.

```cs
public static class Calculator {
	private static int result = 0;

	public static int Sum(int x, int y) {
		return x + y;
	}

	public static void Store(int result) {
		Calculator.result = result;
	}
}
```

#### Static fields
Static fields are basically class variables

#### Static methods
Static methods are basically class wide methods.

#### Static constructor
A non-static class can contain a parameter-less static constructor. It's executed only once in a life time, so you don't know when it will be called if we instantiate multiple class instances.


---

### Polymorphism
Can be static or dynamic.

#### What is static polymorphism?
Function overloading + operator overloading. Which method to run is determined at compile time.

##### Function overloading - same name different method signature
```cs
void print(int i) {
	Console.WriteLine($"Printing int: {i}");
}

void print(double d) {
	Console.WriteLine($"Printing int: {d}");
}

void print(string s) {
	Console.WriteLine($"Printing int: {s}");
}
```


#### What is dynamic polymorphism?
Method to be invoked in determined at runtime, the exact method to call is resolved at runtime based on the object's actual type.

##### Method overriding - base and derived classes have same name and signature
Overriding can be done with marking methods `virtual` in base class, and then `override` in derived class. This is what enables runtime polymorphism.

###### What is the difference between virtual + override vs `new`?

```cs
// creating a derived class but casting it to base class
// 
Interest value = new ComplexInterest(10, 5);

// calls derived method, which is 'Complex', because objects created with derived class will call the methods in derived class
// Note this is determined at runtime!
value.Bank();


// use new to avoid above.
```

#### Explain Virtual/Override vs New
```cs
public class Interest {
    // "virtual" keyword allows the method to be overridden in derived class using "override"
   
    public virtual double Bank(double amount, double rate) {
        Console.WriteLine("Base!");
        return amount + (amount * rate);
    }
}

public class ComplexInterest : Interest {
    // "override" overrides a virtual/abstract (must) method, property, indexer, event
    
    // "new" hides the original method (which doesn't have to be virtual), not encouraged
    public override double Bank(double amount, double rate) {
        Console.WriteLine("Complex!");
        return amount + (amount * rate) + 1500;
    }
}

class Program {

    static void Main(string[] args) {

        // Note

        Interest interest = new ComplexInterest();

        // Even if we casted interest to base class, complex Bank() called

        // Complex! instead of Base! printed

        // interest is of type BaseClass, and its value is of type DerivedClass

        // Used so object with values created from DerivedClass uses methods in the DerivedClass

        // using 'new' ensures base calls base method, derive calls derived method

        var value = interest.Bank(1000, 0.1);

    }

}
```
