[[#What are custom hooks used for? Describe an example]]
[[#Explain custom hooks let you share stateful logic, not state itself]]


#### What are custom hooks used for? Describe an example

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
    // add event listeners
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
We extract the logic.

In this case, you have some state, that reacts to some windows events via useEffect.

```ts
// Our hook encapsulate some states and logics for updating the state
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

// Use case
function StatusBar() {
	// checks if online
	const isOnline = useOnlineStatus();
	
	return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;}
}


```


Convention is React hook names always start with `use`, React component names always start with capital letters.
Hooks may return arbitrary values.




#### Explain custom hooks let you share stateful logic, not state itself
Eg. in the case of `useOnlineStatus` above, each `useOnlineStatus` hook has its own state. They are synchronised with external value in that hook via `addEventListener`. But they are different states.

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
  // this gives back value and onChange
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


#### When does Custom Hooks re-render and do they receive the latest props and states from component?
The code inside custom Hooks will re-run on every re-render of the component.
Custom Hooks needs to be pure.

As custom Hooks re-render together with component, they will receive latest prop and state.

In the code you often see side effects like `fetch` being called inside `useEffect`, that's because this will make sure it only execute when component mounts. And not cause redundant network requests when component re-renders.

`useEffect` are escape hatches. They should be reduced to minimum. When we do have to use them, we can wrap them inside custom hooks. Since implementation is sealed inside the hook, we can later refactor the code of the custom hook to get ride of `useEffect` while not affecting any consumers of the custom hook.







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

