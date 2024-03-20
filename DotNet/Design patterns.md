
Singleton - limit creation of a class to only one object. Singleton can extend class and implement interfaces while static can't. We set constructor to private. Lets you access an object anywhere in the program by implementing a static getInstance method.


Factory
Eg. Ideal for generating different instances of subclasses depending on some logic, so we centralised this logic.


Observer
A


Builder
Builder creates an object step by step.
When you have a complex objects with many parameters,  Eg. A house with windows, doors, rooms, garage, pool, garden etc. 

With Builder, you extract object constructor outside its class and move it into a builder object. This builder object has many methods to configure the objects. Eg. specify how many doors, rooms, garage, pool etc.

Can have a director class to help encapsulate building the object, so we pass builder to director and direct will call all the steps sequentially depending what we want to build.

Useful when you want code to create different representations of some products.




Adaptor
Allows incompatible classes to work together, by converting interface of one class to another. So like a translator.



Adaptor class usually extend interface of service ( what we want to call at the end of the day )



Adaptee - Adapter adapts an adaptee







