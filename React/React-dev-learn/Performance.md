
### Code splitting in React

Without bundle splitting, your whole app is bundled into one alrge file. Browser has to download/parse everything before app loads.

With bundle splitting, only essential code is loaded upfront. Other parts can load as needed (lazy loading), reducing initial load time.

When you lazy load a component, that part is split out into a separate file (chunk) during build time (webpack, vite etc).

This chunk is NOT included in the initial HTML/JS bundle sent to the browser.

Browser only downloads this chunk when lazy-loaded component is rendered for the first time. And after it's parsed and executed, React renders the lazy component.

```js
// Example of React.lazy()
import React, { Suspense } from 'react';

// React.lazy() is used to dynamically import the module, meaning MyComponent won't be included in the main bundle, it will be split into its own chunk and only loaded when needed.
const LazyComponent = React.lazy(() => import('./MyComponent'));

// Suspense wraps the lazy component to display the fallback UI
function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <LazyComponent />
    </Suspense>
  );
}

```