Adding logic to getter or setter later won't break the code.
Allows data binding to work properly.

Changing from a field to a property is a breaking change which will require any assembly which requires your assembly to be recompiled. By making it an auto property avoids this issue.

One important design principle is never expose field as public ,but rather always access everything via properties. This is because you can never tell when a field is accessed or set.

Remember this is about collaboration coding and encapsulation, if you are the only coder, then these things don't matter.