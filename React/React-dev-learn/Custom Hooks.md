React comes with built in Hooks like `useState`, `useContext` and `useEffect`. 



Custom Hooks: Sharing logic between components

Imagine you want to warn user if their network has accidently gone off while using your app. What do you need? 
You need some state in your component that tracks whether network is online. And an effect that subscribe to global online and offline events, and updates that state.

```ts

import { useState, useEffect } from 'react';

export default function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);

  
  useEffect(() => {
	// inside effect, first we define the handle fns
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    // add event listenere
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

	// cleanup fn, whenever the component gets removed below executes
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

```


What if you want this logic in a different component?
You want to extract the logic. So you have something like below.

In this case, you have some state, that reacts to some windows events via useEffect.

```ts
function StatusBar() {
	// checks if online
	const isOnline = useOnlineStatus();
	
	return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;}




function useOnlineStatus() {
	const [isOnline, setIsOnline] = useState(true);

	useEffect(() => {
		function handleOnline() {
			setIsOnline(true);
		}
		function handleOffline() {
			setIsOnline(false);
		}

		// add event listenere
	    window.addEventListener('online', handleOnline);
	    window.addEventListener('offline', handleOffline);

		// cleanup fn, whenever the component gets removed below executes
	    return () => {
	      window.removeEventListener('online', handleOnline);
	      window.removeEventListener('offline', handleOffline);
	    };
	}, []);
	return isOnline;
}

```


Convention is hook names always start with `use`, component names always start with capital letters.
Hooks may return arbitrary values.

Custom hooks let you share stateful logic, not state itself.
Eg. in the case of `useOnlineStatus` above, each hook has its own state. They are synchronised with external value.

```ts
// This is a stateful form
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('Mary');
  const [lastName, setLastName] = useState('Poppins');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }
  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <label>
        First name:
        <input value={firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name:
        <input value={lastName} onChange={handleLastNameChange} />
      </label>
      <p><b>Good morning, {firstName} {lastName}.</b></p>
    </>
  );
}




// We can extract state logic into a custom hook
import { useFormInput } from './useFormInput.js';

export default function Form() {
  // this gives back handler that updates set, and state
  const firstNameProps = useFormInput('Mary');
  // same here
  const lastNameProps = useFormInput('Poppins');

  return (
    <>
      <label>
        First name:
        <input {...firstNameProps} />
      </label>
      <label>
        Last name:
        <input {...lastNameProps} />
      </label>
      <p><b>Good morning, {firstNameProps.value} {lastNameProps.value}.</b></p>
    </>
  );
}


import { useState } from 'react';

// returns the function handler that updates state and also state itself
export function useFormInput(initialValue) {
  const [value, setValue] = useState(initialValue);

  function handleChange(e) {
    setValue(e.target.value);
  }

  const inputProps = {
    value: value,
    onChange: handleChange
  };

  return inputProps;
}

```

Custom hooks are for sharing state logic. If you want to share the state itself, lift the state up and pass it down via prop drilling.


##### Passing reactive values between Hooks
The code inside custom Hooks will re-run on every re-render of the component.
Custom Hooks needs to be pure.

As custom Hooks re-render together with component, they will receive latest prop and state.

```ts
// Custom hook of useChatRoom, takes in states
export function useChatRoom({ serverUrl, roomId }) {
	useEffect(() => {
		// So if serverUrl or roomId changes, we disconnect existing side effects and reconnects
		const options = {
			serverUrl: serverUrl,
			roomId: roomId
		};

		const connection = createConnection(options);
		connection.connect();
		connect.on('message', msg => showNotification('New msg: ' + msg));
	
		// clean up fn
		return () => connection.disconnect();
	}, [roomId, serverUrl]);
}


// using useChatRoom custom hook
export default function ChatRoom( { roomId }) {
	const [serverUrl, setServerUrl] = useState('https://localhost:1234');

	// It consoles logs, responding to prop and state changes
	useChatRoom({roomId: roomId, serverUrl: serverUrl });


	// Return some jsx based on roomId and server url
	// We can also pass in and update state in our custom hooks via useEffect
	
}


```


A hook that returns delayed value
```ts
function useDelayedValue(value, delay) {
  // value will change, so therefore we want to track the delayed value
  const [delayedValue, setDelayedValue] = useState(value);

  // UseEffect is triggered when value changes. 
  // It now updates the delayed value. Since setTimeout is async, it will still return old value, then after the update ( which triggers a re-render ), it will return new value.
  useEffect(() => {
    setTimeout(() => {
      setDelayedValue(value);
    }, delay);
  }, [value, delay]);


  // Initially, we will return value as delayedValue
  return delayedValue;
}
```

