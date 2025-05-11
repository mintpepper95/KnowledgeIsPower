Q - Design an autocomplete UI component that allows user to enter a search term into a text box, a list of search results appear in a popup and user can select a result.

Think google search.

### Requirements exploration

First think about requirements.
- The component should be generic enough to be usable by different websites.
- Input field and search results UI should be customizable.

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
Note in React, controller concept exist, in different forms. Like Container components for handling logic and fetching data, or hooks to encapsulate data logic, or state management like mobx with action creators. Or services.

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
* minimum query length - without a minimum length, there maybe too many results if query is too short as it's not specific enough. We might only want to trigger search when there's more than 3 characters
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

Cache remembers the responses of each query, so probably better.
Note not advisable to 




---




