
My first attempt

![[jira-attempt-1.excalidraw]]


Feedback from chatGPT

#### Instead of placing state updating logic and calling backend ( imported from service classes) in the reducers, use custom hooks.

Now, the reducer will only be responsible for updating state.

Introduce hooks, they grab dispatch methods from stores. And create functions that invokes dispatches (from `useReducer`) from reducer and call service classes to send http requests, eg. functions to create ticket, update ticket etc. And finally return those methods.

In the component, we extract the handlers from the hooks.
In the event handler, we call the hook.

This helps to keep the reducer pure, and better extension, e.g. we can use hooks inside hooks and encourages separation of concerns. And if you ever want things like load spinner, error boundaries and etc. Can't have them inside reducer. 

Reducer - state update

```ts
// Action has a type and payload
{
  type: 'ADD_TICKET',
  columnId: '1',           // The ID of the column to add the ticket to
  ticket: {                // The new ticket object
    id: 123,
    name: 'New Ticket',
    status: 'To Do',
  }
}

export const boardReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_TICKET':
      return {
        // do stuff...
	}

	default:
		return state;
}
```

Hook - orchestrate action using reducer and side effects using service
```ts
const useBoard = () => {
  // pass in reducer and initial state
  const [state, dispatch] = useReducer(boardReducer, initialState);

  const addTicket = async (columnId, newTicket) => {
    // Update state optimistically
    dispatch({ type: 'ADD_TICKET', columnId, ticket: newTicket });

    // Send the ticket to the server
    try {
      await addTicketToServer(newTicket);
      // Optionally, you can update state after success if you want to set something like an id.
    } catch (error) {
      // Rollback the ticket addition if the API call fails
      console.error('Ticket add failed, rolling back.');
    }
  };

  return {
    state,
    addTicket,
  };
```
Service - raw api calls

```ts
// some fetch methods that calls the back end
```
Components - UI + hooks
```ts
// Inside the component

// use hook
const { state, addTicket } = useBoard();

// This is a component event handler
 const handleAddTicket = (columnId) => {
    const newTicket = {
      id: Date.now(),
      name: newTicketName,
      status: 'To Do',
    };
    
    addTicket(columnId, newTicket);
    
    setNewTicketName(''); // Reset the input after adding
  };
```


##### React-query
We can replace functionality of useReducer + custom hook to do state update and side effect with react-query.

```ts
// useAddTicket.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';

async function postTicket(ticketData) {
  const res = await fetch('/api/tickets', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(ticketData),
  });
  if (!res.ok) throw new Error('Failed to add ticket');
  return res.json();
}

export function useAddTicket() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: postTicket,
    onSuccess: (newTicket) => {
      // Update ticket list cache (optional)
      queryClient.setQueryData(['tickets'], (old = []) => [...old, newTicket]);
    },
  });
}

// using it inside the component
const { mutate: addTicket, isLoading, error } = useAddTicket();

const handleAdd = () => {
  addTicket({ title: 'New Ticket', status: 'To Do' });
};
```


A component calls `useQuery`, a mutation happens, then react query updates the internal cache, and notifies all components that some data has changed. Those components will auto re-render with the new data.

Hence we don't actually need context here, because these are all server states. We would only need context/states for states not from server.

React query vs context
React query has automatic caching, refetching, can easily manual rollback etc.
React query owns the data.
Handles mutation with optimsitic updates.

Imagine adding a new ticket,
The new task **appears instantly in the UI**, even before the server replies.  
If it fails, it disappears.  
If it succeeds, the server sends the “real” ticket (with proper ID), and cache syncs.



Overall structure would look like following

![[Pasted image 20250513083156.png]]


#### Conflict resolution
Not just naively last operation win. Imagine two users both updating a ticket. One changes title, the other changes score. We want both.

So instead of just tracking state, track operation. So we can apply transformations to integration changes from different users.

#### Debounce update
We could perhaps debounce updates. And group multiple updates into a single api call. 

#### React.memo
Might be worth using React.memo to memoise components to skip rendering, especially if components re-render due to parent rendering.


#### Discuss the advantages and disadvantages of having one store vs separate stores for columns and tickets

Single store pros:
* Simplified state management
* Don't need to worry about updating multiple stores

Single store cons:
* Potentially more renders
* Less modular, you need to handle logics to different entities in the same context


Multi store pros:
* Separation of concerns - easy to maintain
* Scalability - Allows you to fine tune performance when app grows with many stores
* Fewer component re-renders, unless

Multi store cons:
* More complex state management
* Need to worry about updating multiple stores


#### Deferring load via import on interaction
We defer loading of actual share dialog until share button is pressed. Improving both FCP and TTI.

![[Pasted image 20250514010304.png]]


### Inline CSS

Inline critical CSS for FCP into your html instead of importing it from a separate file. This makes sure style is applied immediately, without waiting for external files to be fetched, parsed and applied. This speeds up FCP. You also reduce the number of HTTP requests. 


### Above the fold Images and Below the fold images

ATF Images are ones visible to user on page load.  All ATF images should be sized ( with an explicitly defined height and width ) so browser knows how much space to reserve when loading them. Else might cause layout shift, which will frustrate users. 
BTF Images are ones not immediately visible to user on page load.  User needs to interactive/scroll to see them. Therefore can be lazy loaded. 




Modern bundlers like webpack and vite perform **tree shaking** only on ESM code. Because ESM have static import export statements.

While commonjs uses dynamic `require`, so bundler can't reliably know which exports are used at build time








JavaScript is the second biggest [contributor to page size](https://almanac.httparchive.org/en/2020/page-weight#fig-2) and the second most [requested web resource](https://almanac.httparchive.org/en/2020/page-weight#fig-4) on the internet after images.

- **Gzip** and **Brotli** are the most common ways to compress JavaScript and are widely supported by modern browsers.
- **Brotli** offers a **better compression ratio** at similar compression levels.

To reduce payload sizes, you can minify JavaScript before compression. [Minification](https://web.dev/reduce-network-payloads-using-text-compression/#minification) complements compression by removing whitespace and any unnecessary code to create a smaller but perfectly valid code file.


Note modern web bundlers auto minify and do tree shaking.



**SEO considerations:** Most web crawlers can interpret server rendered websites in a straight-forward manner. Things get slightly complicated in the case of client-side rendering as large payloads and a waterfall of network requests (e.g for API responses) may result in meaningful content not being rendered fast enough for a crawler to index it. Crawlers may understand JavaScript but there are limitations. As such, some workarounds are required to make a client-rendered website SEO friendly.
