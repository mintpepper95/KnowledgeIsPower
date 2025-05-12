[[#Explain the different ways to load resources. Eager, lazy, prefetch, preload]]
[[#What are preload and prefetch?]]
[[#When to use import on interaction?]]
[[#What is a facade (fuh-sard)?]]
[[#Authentication]]
[[#How to import on interaction in vanilla JS and react?]]
[[#What is hydration? Why is only needed for SSR and not CSR?]]


---

#### Explain the different ways to load resources. Eager, lazy, prefetch, preload

Lazy-load non-critical resources, load them when user interacts with UI that requires them.

Your page may contain code/data for a component or resource that isn't immediately necessary. Eg. Many parts of UI are hidden unless user click or scrolls.

This can apply to first party code you write, and also to third party widgets.

Loading these resources eagerly (right away) can block the main thread if they are costly, delaying how soon user can interact with more critical parts of a page even if the page looks ready. Clicks and taps often don't work because event listeners and click handlers have not yet been attached to ui elements.


We can load them at better moments, like
* When user clicks to interact with component for the first time
* Scrolls component into view
* Deferring load of the component til browser is idle

The different ways to load resources, are
* Eager - load right away, normal way of loading
* Lazy ( router-based ) - loads when user navigates to a route/component
* Lazy ( on interaction ) - loads when user clicks UI
* Lazy ( in viewport ) - loads when users scrolls component into view
* Prefetch - Load prior to needed, but after critical resources are loaded
* Preload - eagerly, with a greater level of urgency


Eg. Google Doc lazily loads share feature when you click on the share button.

---
#### What are preload and prefetch?

##### Preload
Loading resources needed for current page ASAP.
Typically for resources needed immediately, eg. stylesheets, scripts, fonts, images etc.

Higher priority.

##### Prefetch
Loading resources needed for future navigations.
Tells browser to fetch resources likely to be used on subsequent pages or interactions, loading them in the background with a lower priority.

Improves performance of future navigations by preloading those resources in the background.

---

#### When to use import on interaction?
Import on interaction for first party code should ONLY be done if you can't prefetch resources prior to interaction. 

This pattern is more relevant for third party code, where you want to defer it to a later time if non-critical. 

Eg. You might be using `react-scroll` library to implement a feature that scrolls back to the top of a page. Rather than eagerly loading this dep, we can load it on interaction with the button.

---

#### What is a facade (fuh-sard)?
One option for implementing load-on-interaction is using a facade, a simple preview/placeholder for a more costly component.

Basically a fake component ( can just be css and html ), when clicked loads the actual bundles for the real component.

When the user clicks on the facade, only then code for the resource is loaded, if they don't need it then we don't load the resources giving users a better performance and ux.

Think youtube thumbnail.

Synchronously loading third party scripts block the browser parser and can delay hydration (SSR).

---

#### What is hydration? Why is only needed for SSR and not CSR?
Hydration is process of attaching JS to static HTML elements that have been generated on the server, to make the pages interactive. In server side rendering, fully rendered HTML and JS are generated on the server and sent to the client. This HTML is static, hydration is necessary to attach event listeners to the page and make it interactive. Once HTML and JS bundles have been downloaded by the browser, JS takes over and attaches event listeners to the HTML.

Hydration is not needed for client side rendering. Server sends a very basic HTML page with placeholder and links to JS scripts. Entire JS bundle is loaded and executed on the client, which constructs the DOM elements, rendering the UI. So content is generated entirely on the client side. No initial content means no hydration is needed.

---

#### Authentication
Apps may need to support auth with a service. These can occasionally be large with heavy JS execution costs, so we might not want to eagerly load them if user isn't going to login. We can instead dynamically import them upon user clicking on the login button. This keeps the main thread more free during initial load.

Similarly with Chat widgets, by using a facade. We can reply it with a fake replica with HTML and CSS. Only when the widget button is clicked we load the actual chat window. 

---
#### How to import on interaction in vanilla JS and react?

In vanilla JS, we can use dynamic imports to lazy load modules. 

```
// Eg, we scroll back to top when user clicks on the button which triggers the below handler

handleScrollToTop() {
    import('react-scroll').then(scroll => {
      scroll.animateScroll.scrollToTop({
      })
    })
}
```

##### React

Code splitting - Instead of sending large JS to user as soon as when first page of your app is loaded, split your bundle into multiple pieces and only sends what's necessary at the beginning.

In React we can achieve this with `React.lazy`. It provides a way to separate components in an app into separate chunks of JS. We can take care of loading it with suspense.

For example
```ts
import React, { lazy, Suspense } from 'react';
import MessageList from './MessageList';
import MessageInput from './MessageInput';

// make it a separate chunk
const EmojiPicker = lazy(
  () => import('./EmojiPicker')
);

const Channel = () => {
  ...
  return (
    <div>
      <MessageList />
      <MessageInput />
      {emojiPickerOpen && (
        <Suspense fallback={<div>Loading...</div>}>
          <EmojiPicker />
        </Suspense>
      )}
    </div>
  );
};
```

---

#### Import on interaction for 1st party code as part of progressive loading

Imagine you have a hotel booking app, if everything is loaded upfront, this will leave user waiting a long time.

![[Pasted image 20240606104911.png|600]]

SSR can be an improvement, since the pages are delivered sooner, but it won't be interactive until client side completes hydration, so there maybe this weird state where the page looks ready, but not interactive.


**The idea is to download minimal code initially so page is visually complete quickly.
As user starts interacting with the page, we use those interactions to determine what other code to load.**


![[Pasted image 20240606110929.png|580]]

#### Trade offs of shifting costly work to user interactions
##### What happens if it takes a long time to load a script after user clicks?

Small granular chunks minimise chance of user going to wait long for code and data.

We can also prefetch the resources after critical content on the page is done loading.

##### Lack of functionality before user interaction?
Instead of facades, maybe lazy load the components when they scroll into view rather than deferring load until interaction.

