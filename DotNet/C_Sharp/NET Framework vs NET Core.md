* .NET Framework (windows only, older and closed source)
* .NET Core (cross-platforms, newer, MS made everything free and open source, stripped down version of .NET Framework)
* .NET standard is a minimum set of requirements every platform needs to fulfil, defines a common set of APIs that can be used by .NET apps.




#### JIT compilation
JIT is part of CLR (common language runtime) for executing .NET apps. 

When you compile code in C#, it's converted into an intermediate language (IL) by the compiler.

IL is converted into machine code (0&1s) by JIT compiler.








- CLR ( common language runtime)

Running application (including memory, threading and garbage collection, type safety , exception hadnling etc)

CLR uses JIT compiler to translate  CIL (Common intermediate language, compiled from source code) to machine code.

- Class library

Provides types for strings, numbers, dates etc. Includes apis for reading/writing to files, connecting to databases etc. These are codes already written by other developers.