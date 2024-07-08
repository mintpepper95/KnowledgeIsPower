[[#Difference between the two]]

---
#### Difference between the two
`React.lazy` is a function used to dynamically import a component in React, breaking it into a separate chunk. `React.lazy` works only with default export as of now.

`Suspense` is a component used to handle loading state of components imported with `React.lazy`.

When you dynamically load a component with `React.lazy` and `Suspense`, it's loaded only once the first time it's needed. After that it's cached and reused.






