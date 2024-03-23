JS is an interpretative language that doesn't need compiling. JS is dynamically typed like python. Web browsers have embedded 'engines' that allows javascript to run. The engine parse the js code, compiles the script to machine code for execution.

In-browser js has no access to OS functions. Eg, read/write abitrary files on the hard disk. It's very limited (eg. Drag and drop files into browser).

Different tabs and windows generally do not know about each other and have their own execution environments.  Meaning js from one page may not access the other if the other is from a different site (Same origin policy).

Although js can easily send/get request from server where the current page came from. Its ability to receive data from other sites are crippled for safety concerns (requires explicit agreement from the other side).


[[Let vs const vs var]]
[[Hoisting]]

