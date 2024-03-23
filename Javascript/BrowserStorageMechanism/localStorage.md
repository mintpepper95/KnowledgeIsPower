Stored data is persistent. Even if user refresh the web page or closes and reopens the browser window or restart our pc, the data will be there.

`localStorage` is an object that can be accessed through browser's `Window` interface.
```ts
localStorage.setItem(key, value);
```
To see what's in `localStorage`, we can use dev tools and go to Application tab to see it.
![[Pasted image 20240323225930.png]]
The only way to delete it using `localStorage.clear()` or `localStorage.removeItem()` or clear browser memory