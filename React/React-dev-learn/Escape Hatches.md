
#### Referencing values with Refs

The difference between ref and state here is that, setting state re-renders a component, changing ref does not!

```ts
const ref = useRef(0);

// useRef returns an object, eg. { current: 0} here, it's mutable
import { useRef } from 'react';

export default function Counter() {
	let ref = useRef(0);

	function handleClick() {
		ref.current = ref.current + 1;
		alert('You clicked ', ref.current, ' times!');
	}

	return (
		<button onClick={handleClick}>
			Click me!
		</button>
	);
}

```

Use a ref when you want to store timeout Ids, manipulating DOM elements, storing some value which does not impact rendering logic.

Storing timeout ids - declaring regular variables like `let timeoutID`, it doesn't survive between re-renders because every render runs the component and initialise its variables again. Here's an example of stopwatch

```ts
import { useState, useRef } from 'react';

export default function Stopwatch() {
  // stores date which stopwatch starts
  const [startTime, setStartTime] = useState(null);

  // stores current time
  const [now, setNow] = useState(null);


  // stores timerID
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

	// clear previous timerId if any
    clearInterval(intervalRef.current);

	// Start an interval to keep update now
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}

```

Recall state works like a snapshot, so you can't read latest state from an async operation like setTimeout, it will get the old state despite you may have updated the state earlier. You can read the latest input text in a ref. A ref is mutable, so you can always read `current` property at anytime.



#### Manipulating DOM with Refs

Eg. when you want to focus a node, measure its size/position etc.

Focusing a text input, note ref attribute does not exist on functional components, only on built in ones like div/input etc. 
```ts
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }


  // tells React to put <input /> DOM node into inputRef.current
  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}

```


##### Access another component's DOM nodes with forwardRef
You can put ref on built-in component like `input`, but doesn't work if you put ref on your own component. This is because by default React does not let a component access DOM nodes of other components. This is by intention as manipulating another component's DOM nodes makes your code fragile.

To have components expose their DOM nodes, we specify using forward ref.

```ts
import { forwardRef, useRef } from 'react';

// forwardRef allow custom component DOM nodes to be exposed
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});



export default function Form() {
  const inputRef = useRef(null);


  // We can now manipulate child component from parent
  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}




// Another example
import { forwardRef } from 'react';

export default forwardRef(function SearchInput(props, ref) {
    return (
      <input ref={ref}
        placeholder="Looking for something?"
      />
    );
  }
);

```



This allows `MyInput` to receive ref and the ref is passed to `input`.


##### When React attaches the refs
In React every update is split into two phases,
1. During render, React calls your components to figure out what should be on screen
2. During commit, React applies changes to DOM.

In general, you don't want to access refs during rendering. During first render, DOM nodes have not been created, so `ref.current` will be `null`. And during rendering of updates, DOM nodes haven't been updated yet. So too early to read them.

React sets `ref.current` during commit. Before updating the DOM, React sets affected `ref.current` values to `null`. After updating the DOM, React sets them to the corresponding DOM nodes.

Usually you will access refs from event handlers. If you want to do something with ref, but there's no particular event to do it, you might need an effect.


##### Best practices for DOM manipulation with refs

Refs are an escape hatch. Only use them when you have to step outside React, eg. managing focus, scroll position, measure size, calling browser APIs React don't expose.

Sticking to non-destructive actions like measuring sizes and focusing should be okay. If you try and modify the DOM manually, you risk conflicting with changes React is making.

So avoid changing DOM nodes managed by React. Modifying, adding children or removing children from elements managed to React can lead to crashes. You should only use ref to  modify parts of DOM React does not update.

You can pass ref conditionally
```ts
<li ref={index === i ? selectedRef : null}>
```







