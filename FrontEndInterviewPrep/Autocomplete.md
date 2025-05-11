Q - Design an autocomplete UI component that allows user to enter a search term into a text box, a list of search results appear in a popup and user can select a result.

Think google search.

### Requirements exploration

First think about requirements.
- The component should be generic enough to be usable by different websites.
- Input field UI and search results UI should be customizable.

Think about - what kind of results should be supported (  just text? images? )
What device will be used? Computers, tablets and phones. Do we need fuzzy search?

---

### Component architecture

![[autocomplete-architecture.png]]
Input - handling user input and pass it to controller
Result - receives result from controller and display it, also handles user selection and informs controller

Cache - store results of previous queries, so controller can check if need to send request to server or get from cache.

Controller - For connecting everything.


#### Controller in React, state management and hooks + context
Note in React, controller concept exist, but in different forms. Like container components for handling logic and fetching data, or hooks to encapsulate data logic, or state management like mobx with action creators. Or services.

```js
// state management as controller
// components dispatch actions, and those state management tools handle the action by invoking the appropriate response, like fetching from server.

// service as controller
// We can also have a service, e.g. UserService to do things like fetch profile and update profile. We can then use this service inside a hook.
```

Given these two are pretty similar, how to choose between hooks and side effects via state management?

Use hooks for component specific concerns, where logic is tied to component itself. Think component lifecycle logic. E.g

While hooks + context can be okay in some circumstances (eg. theming), context will re-render all children of context provider with every update. To combat this, store small chunks of things into context, or use multiple contexts, or use a state management system.

Context should be used for things that we want to render from the context root to downwards, like theming and auth. Think of it as a form of dependency injection.

Use side effects for app wide or shared concerns.
E.g. Need user profile, auth token, settings across components. And when we want a centralised state and effect management pattern.

Like a large amount of application state needed in many places in the app. 
App state update is very frequent.
Logic to update state maybe complex.
Understand how state evolves and visualise the changes.
Managing side effects and persistence etc.


In the case of mobx, we can define the store, and use context to pass it around.

Recall in mobx, action is any functions that updates the state. State changes must happen inside an action, or mobx will throw error.

Observer - components that react to observable changes ( e.g. a store ), we wrap a component with observer HOC.

Observer component will auto re-render if state changes.

Computed values, they are derived from one or more observables and cached efficiently. Mobx only recalculates them if their dependencies change.

Reactions - Reactions are more general than observers, can be used to perform side effects when observables change. It runs a function when observables change.

```js
class TodoStore {
  // this is an observable state
  todos: string[] = [];

  constructor() {
    makeAutoObservable(this); // marks functions as actions automatically
  }

  // we can decorate this action with @action, but it's inferred via makeAutoObservable
  addTodo(todo: string) {
    this.todos.push(todo);
  }

  @computed get completedTodosCount() {
    return this.todos.filter(todo => todo.completed).length;
  }
}
```

---

### Data model and props for the component

Controller
* current search string
* options exposed via component API ( via e.g. props )

Cache
* Initial results
* Cached results


For customisable options, we can have things like below.
* Number of results
* Api url
* Event listeners - 'input', 'focus', 'blur', 'change', 'select' etc
* Customisable rendering options, like passing in an object for font e.g `{ textSize: 12px, textColor: 'red' }`
* Class names - which allow devs to specify their own css class names

There can be some more advanced things like
* minimum query length - without a minimum length, there maybe too many results if query is too short as it's not specific enough. We might only want to trigger search when there's more than 3 characters. Google search doesn't have this btw.
* debounce duration - triggering api on every keystroke can be wasteful, like when user has not finished typing, we could debounce to make sure API doesn't get hit too often. We can debounce and make server not get called until no user input for 300ms.
* timeout duration - how long we wait for a response before determining search has timed out and we display an error.
* Cache related settings - cache duration and etc

Server should provide API that supports query, limit ( max number of items ) and pagination ( page number )

---

### Optimisation and deep dive

#### What happens if user updates query when there's a pending network request?
we will need to make sure not to display results for previous queries to the user. We can't rely on return order of network responses from server as earlier request can be completed later.


( Here we are proposing two solutions and talking trade-offs )
We can perhaps attach a timestamp to request to determine which response is relevant.
Or save the results in object/map, keyed by search query and only present results corresponding to input value of search.

Note in React, we have useEffect clean-up function which returns every time at the start of every Effect and when component is unmounted. So if a new Effect is going to run (decided by its deps array), React will run the clean-up function beforehand, which sets an `ignore` variable to true which indicate request tied to this render is outdated, and don't set data from the outdated request's response.

Cache remembers the responses of each query, so probably better.

Some times requests can fail, the component can auto retry firing the query.

If we detect device is offline, then not a lot we can do since our component calls to the server. We should indicate there's no network connection in the component.


#### Cache
E.g. using a hashmap to store query and its result.

The downside of storing directly from query to result is that there could be a lot of similar results for similar queries, eg. `fac` and `face` etc.

Alternatively we can assign a unique id to each search result. Eg. we search for some query and get back 10 results. They would have id from 1 to 10.

We can now have another dictionary which maps search query to a list of ids where id is the result id. Processing is negligible if there are only a few items.

##### Which approach to use will depend on where this component is used.
If the component is used on a short-lived page. Like Google search, then option 1 would be best. Even if there's dupe data, user is unlikely to use search so often that memory becomes an issue, as they will navigate away from the page.

If the component is used on page which is long-lived. Like facebook or Instagram, then option 3 might be more viable. Do note we might want expiration as cache may be out of date.


##### Initial results

We might also want to show some initial results, like what's trending. It could be an option on the component and added to the cache where query string is empty.

##### Caching strategy
When to evict cache so we keep a good balance between performance and timeliness of the cached data.

Eg. Google search cache might be rarely updated, they can live for hours. But for stock/currency/pricing we might want the cache to be short-lived or not cache at all, because we care about the accuracy given they change rapidly.

We can add cache options to our component, such as time-to-live for retaining the cache.

### Performance
Performance relates to client side performance as server-side is out of scope.

#### Loading speed
We can show results for previous queries near instantly due to cache.

#### Debounce/throttling
Limit the number of network requests to reduce server load and also processing.

#### Memory usage
Long-lived pages might have auto-complete components which accumulate too many results in the cache, hogging memory. Purging the cache is essential in these cases. Can be done when browser is idle or number of cache entries exceed a threshold.

#### Virtualised lists
If result contains many items ( e.g hundreds and thousands ). rendering that many DOM nodes in the browser would have an impact on performance and cost memory.

We can use virtualisation (windowing) to help us scale. Virtualisation is a performance optimisation technique to render only DOM nodes currently visible in the viewport and a small buffer. As user scrolls, Dom nodes are rendered and unmounted dynamically.

### User experience

#### Handling different states
E.g. loading, error, no network

#### Handling long strings
Long text should be handled appropriately, usually via truncating.

#### Mobile friendliness
Search component should be easy to use on mobile view.

#### Keyboard interaction
E.g. Shortcut-key to let user to focus on the input. Like `/` which focuses on search bar on youtube.

#### Typos in search
It's easy to make typos. A fuzzy search allows us to match results that matches the query closely instead of exact matches. We can have fuzzy logic on the server side.

#### Query results positioning
The list of autocomplete suggestions typically appear below the input. However, if input is at bottom of the window, there would be an insufficient space to fully display the results. In this case, we might want to render above the input.

### Accessibility

#### Screen reader
Use semantic HTML or aria attributes ( for when semantic HTML is not enough ).

`aria-label` for input. This provides an accessible name for the input element. Screen reader will read it to describe purpose of the input.

`role='combobox'` for input. A text input with a list of possible options.

`aria-haspopup` to indicate element can trigger a popup. 

`aria-expanded` to indicate whether popup element is currently opened or closed.

#### Keyboard interaction
Hit `enter` to perform a search, can get this behaviour by wrapping `input` in a `form`.

Up and down arrows to navigate options, wrapping around when end of list is reached.

Escape key to dismiss popup result.










---




