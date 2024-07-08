We can request resources that are only needed for specific routes. By combining `Suspense` with libraries like `react-router`, we can dynamically load components based on the current route.

By lazy loading the components per route, we are only requesting the bundle that contains the code necessary for the current route.

```ts
const App = lazy(() => import(/* webpackChunkName: "home" */ "./App"));

const Overview = lazy(() =>
	import(/* webpackChunkName: "overview" */ "./Overview")
);

const Settings = lazy(() =>
	import(/* webpackChunkName: "settings" */ "./Settings")
);



render(
  <Router>
   <Suspense fallback={<div>Loading...</div>}>
     <Switch>
       <Route exact path="/">
          <App />
       </Route>
       <Route path="/overview">
         <Overview />
       </Route>
       <Route path="/settings">
        <Settings />
       </Route>
     </Switch>
   </Suspense>
  </Router>,
document.getElementById("root"))
```