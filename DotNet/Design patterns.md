
#### Singleton - limit creation of a class to only one object. Singleton can extend class and implement interfaces while static can't. We set constructor to private. Lets you access an object anywhere in the program by implementing a static `getInstance()` method ( so like if not initialised, initialised and return it, else just return it ).


#### Adaptor
Allows incompatible classes to work together, by converting interface of one class to another. So like a translator.

```csharp
interface IModernDatabase {
	void Insertdata(string data);
}

interface ILegacyDatabase {
	void Insert(string data);
}


class LegacyAdapter : IModernDatabase  
{  
    private readonly ILegacyDatabase _legacyDatabase;  
  
    public LegacyAdapter(ILegacyDatabase legacyDatabase)  
    {  
        _legacyDatabase = legacyDatabase;  
    }  
  
    public void InsertData(string data)  
    {  
        _legacyDatabase.Insert(data);  
    }  
}


// Usage - adpating legacy db to modern db
ILegacyDatabase legacyDatabase = new LegacyDatabase();  
IModernDatabase modernDatabase = new LegacyAdapter(legacyDatabase);  
  
modernDatabase.InsertData("Hello world!");

```
lgacy db and modern db have different methods to insert to library. Legacy db is using an outdated interface. We want to integrate legacy db with modern db that has a different interface.

Adaptor inherits interface of modern db, takes in an instance of legacy database, and implements the new api by calling the outdated api underneath.



#### Factory
Eg. Ideal for generating different instances of subclasses depending on some logic, so we centralised this logic.


#### Template method pattern
Abstract class has a template method, which defines the skeleton of an algorithm.
Inside this method, some calls are abstract or virtural.

We can then implement this abstract classes with concrete classes, where the concrete class override the virtual methods and implements the abstract method.



#### Observer
Allows reacting to events happening in other objects without coupling to their classes.

For example, you have a subject class which has methods for attaching, detaching, notifying observers.

You can create a observer list property with a `List<xxxObserver>` inside the class, call attach() to attach observers to the list. And then when something happens, call notify with loops through this observer list and notify each observer.



Visitor





#### Builder
Builder creates an object step by step.
When you have a complex objects with many parameters,  Eg. A house with windows, doors, rooms, garage, pool, garden etc. 

With Builder, you extract object constructor outside its class and move it into a builder object. This builder object has many methods to configure the objects. Eg. specify how many doors, rooms, garage, pool etc.

Can have a director class to help encapsulate building the object, so we pass builder to director and direct will call all the steps sequentially depending what we want to build.

Useful when you want code to create different representations of some products.






