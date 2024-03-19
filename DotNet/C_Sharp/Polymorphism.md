* Static polymorphism - function overloading ( same name different signatures )
* Dynamic polymorphism - method overriding ( base class can have same name & signature, and overriding can be done using virtual and override )

New vs override
If you create a derived class instance, and specify its type as base class.
Calling the method will still invoke the derived class.

To avoid this, use `new`.

```csharp
public class Interest {
    // "virtual" keyword allows the method to be overridden in derived class using "override"
    public virtual double Bank(double amount, double rate) {
        Console.WriteLine("Base!");
        return amount + (amount * rate);
    }
}

public class ComplexInterest : Interest {
    // "override" overrides a virtual/abstract (must) method, property, indexer, event
    // "new" keyword hides the original method (which doesn't have to be virtual), not encouraged
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


