![[Pasted image 20240723022806.png]]

```cs
try {
    // parse string to int
    var number = int.parse(Console.ReadLine());
    
    // string interpolation
    Console.WriteLine($"Number is {num}");
} catch (Exception ex) {
	Console.WriteLine("Error");
} catch (InvalidCastException inex) {
	// throw some exception
	throw new Exception("Error occured");
}
```