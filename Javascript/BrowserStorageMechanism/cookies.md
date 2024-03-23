Like `localStorage`, cookies are persistent, meaning it will remain in the browser even if user refreshes the web page or closes/reopens the browser window.

Unlike `localStorage` or `sessionStorage`, cookies are sent to the server with every request.

Cookies can be deleted when we explicit delete it or clear browser memory or when cookies reach their expiration date.

Cookies are better for storing sensitive data than `localStorage` or `sessionStorage`.
Cookies can be marked as `HttpOnly`, meaning they can only be accessed by the server and not by JS running in the browser.

