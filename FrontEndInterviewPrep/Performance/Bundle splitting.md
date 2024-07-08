When building a modern web app, bundlers like `Webpack` takes up app source code and bundles it into one or more bundles. When a user visits a website, the bundle is requested and loaded in order to display data to the user screen.

The bigger the bundle, the longer it takes before engine reaches the line which first rendering call is made. Until that time, user has to stare at a blank screen.

Below shows the loading of a large bundle.
Note the delay between FCP and TTI could be of several reasons
* browser still needs to execute more JS for interactivity
* browser may still needs to load/process additional resources
* JS can be render-blocking, when they are executing
![[bundle_1.webm]]


---

We want to display data to user as quickly as possible, a larger bundle leads to an increased amount of loading time, processing time and execution time.

Instead of requesting one giant bundle that contains everything, we can split the bundle into smaller bundles.

![[Pasted image 20240606112409.png]]

We can reduce the time it takes to load, process and execute a bundle. This helps speeding up to FCP ( first contentful paint - the time from when user first navigated to the page to when any part of the page content is rendered on screen )

TTI - time it takes before all contents painted to the screen and has been interactive

---

A bigger bundle doesn't necessarily mean a longer execution time, it could happen that we load a ton of code we'd never use. Maybe some parts of the bundle only get executed on certain user interactions.

But the browser engine has to load, parse and compile code that's not used on the initial render before user is able to see anything on screen. So fetching a larger bundle than necessary can hurt performance of your app.



#### Code splitting by itself can still improve performance
With code splitting, even if chunks are loaded synchronously they can still help optimise load time.

Smaller chunks allow browsers to load resources in parallel, instead of waiting for a single large bundle to download, browser can fetch smaller bundles concurrently, potentially reducing overall load time.

Caching - smaller chunks are more likely to be cached by the browser. Therefore subsequent visits to the app may require fewer resources.

Reduced parse time - smaller bundles can reduce the time it takes browser to parse and compile JS code.

With that being said ,full benefits of code splitting are realised when loading is deferred until needed.

