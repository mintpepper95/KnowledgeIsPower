`sessionStorage` is similar to `localStorage`, where the data is only accessible through the client. However, `sessionStorage` data is auto deleted when user closes the browser tab/window where the data is stored.

However if we just refresh the web page, data won't be deleted.

`sessionStorage.setItem('greeting', 'hello')

Like `localStorage`, we can also find `sessionStorage` data under Application tab in dev tools.

```ts
sessionStorage.removeItem('greeting');
sessionStorage.clear();
```
Data can also be deleted by calling above methods or clearing out browser's memory.
