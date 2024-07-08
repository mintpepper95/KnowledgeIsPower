[[#What is static import and how does it affect bundle size?]]
[[#What is dynamic import and how does it affect bundle size, and how does that improve FCP and TTI?]]


---

#### What is static import and how does it affect bundle size?

For statically imported components, bundlers like Webpack would bundle the modules into one bundle. A larger bundle size can affect loading time of our app depending on user device and network connection. before app is able to render its contents, it first has to load and parse all modules.

We don't always need all modules at once - some components may render upon user interaction or page scroll, we can dynamically import these components.



#### What is dynamic import and how does it affect bundle size, and how does that improve FCP and TTI?
Eg. You have a chat app, that has multiple components: `UserInfo`, `ChatList`, `ChatInput`, `EmojiPicker`. 

`EmojiPicker` is not directly used on initial page load, it won't render unless user clicks on the emoji button. This means it's unnecessary to add `EmojiPicker` module to our initial bundle. which potentially increase the load time.

We can dynamically import the `EmojiPicker` component. Instead of statically importing it, we import when we want to show it.

In React, you can dynamically import components with `Suspense.`
See here for more about `Suspense` and `React.lazy`.




##### Splitting the bundle
Instead of adding `EmojiPicker` to the initial bundle, we split it into its own bundle, reducing the size of the initial bundle.

A smaller initial size means faster initial load, user doesn't have to stare at a blank screen for so long before able to interact with our app. We can display a fall-back component while we import our component lazily.
![[dp_1.webm]]


##### What load sequence looks like after bundle splitting

We load the initial bundle, parse it, compile it and execute it.
When we click on the emoji picker we load the emoji picker bundle.

![[dp1.webm]]



##### Dynamic import to improve FCP and TTI
We reduced the initial bundle size! Although user may still have to wait a while when loading emoji picker, we have improved ux by making sure app is rendered and interactive while user waits for component to load.

In this picture, `GET /` is the initial request to load the page.
`GET /bundle.js` is the main JS bundle that's loaded. Once it's loaded, app has enough content to paint onto screen.

And after main app is rendered, page is considered interactive.

`GET /picker.js` is loaded when it's needed.

![[dp_2.webm]]


 TTI ( time to interactive ) - Page displays useful content, event handlers are registered for most visible page elements, and page responds to user interactions within 50 ms.

