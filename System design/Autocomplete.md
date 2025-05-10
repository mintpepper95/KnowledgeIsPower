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

### Data model



