![[Pasted image 20240324202059.png]]

Web APIs can do stuffs concurrently unlike js.

Async functions like setTimeout gets sent to the Web APIs in Browser.

Once timer up, we put setTimeout's function to callback queue.

Once stack is empty, it takes thing in Callback queue and move them to the stack and the stack which executes them.