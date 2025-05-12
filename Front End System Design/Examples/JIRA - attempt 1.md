
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






