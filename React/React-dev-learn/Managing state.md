In React, we describe UI for different states of the component and then trigger state changes.
So think about the different visual states, determine what triggers them, represent the states with useState and connect event handlers to set the state.

Take example of a form. Here are the various states for a form.
Empty -> Typing -> Submitting -> Error or Success

We might be tempted to come up the following states.
```ts
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);

const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```

However, many of states are redundant.
Eg. 
We don't need `isError`, we already have error.
We don't need both `isEmpty` and `isTyping`.
Also notice `isTyping` and `isSubmitting` can't both be true.

```ts
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing') // typing, submitting, success
```

Now we can connect event handlers to set state.


Eg. A quiz form built in React
When you have a functional component like this, when it re-renders the whole body is re-executed, React then compares the difference between current and previous virtual DOM and only updates part of DOM that's changed.

```ts
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>That's right!</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>City quiz</h2>
      <p>
        In which city is there a billboard that turns air into drinkable water?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button disabled={
          answer.length === 0 ||
          status === 'submitting'
        }>
          Submit
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  // Pretend it's hitting the network.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima'
      if (shouldError) {
        reject(new Error('Good guess but a wrong answer. Try again!'));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```



States shouldn't contain redundant info. Unnecessary states means we might forget to update it and introduce bugs. 

**Note state is persevered between re-renders. And it happens outside the component. When your component re-renders React gives it the current value of states. 

This is also why React doesn't allow you to call hooks conditionally, all executions of the function must have same # of hooks and in same sequence. 


#### Choosing State Structure




1. Group related state. If you want to update two or more state variables at same time, consider merging them into a single state.
2. Avoid contradictions in state, eg. states that disagree with each other, because you would leave room for mistakes. Eg. have two states, isSending and isSent, leaves room for mistakes and makes it hard to understand.
4. Avoid redundant state, if you can calculation some info from props or existing states, don't add new ones. Also don't set prop to as state. When the component re-renders later when prop changes, state won't be updated, as state is only initialised with the prop value in the first render and we don't explicitly update it. Mirroring only makes sense when you want to ignore **all updates** for a specific prop.
5. Avoid dupe state, eg. storing id of an array item instead of the array item itself. If you store the array item itself in the state, later you may update the array, now array item in the state is no longer found in the array item, so out of sync.
6. Avoid deeply nested state, prefer structure state flat. As updating deeply nested states is very verbose.

We can have Set and other data structures in a state.

Eg. A red ball that tracks cursor movement
```ts
const[x, setX] = useState(0);
const[y, setY] = useState(0);

// This is better, if the state variables change together
const [position, setPosition] = useState({x: 0, y: 0});


// Following example
import { useState } from 'react';

export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });


  return (
    <div
      onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  )
}

```



#### Sharing state between components
When you want state of two components to change together, move the state to the closest common parent, and then prop drill. - Lifting state up

Eg. Here only one panel should be active.

```ts
import { useState } from 'react'

export default function Acoordion() {
	// Note initial state, 0 here, is only used on first render
	const [activeIndex, setActiveIndex] = useState(0);

	return (
		<>
		<h2>Almaty, Kazakhstan</h2>
		<Panel title='About' isActive={activeIndex === 0} onShow={()=>setActiveIndex(0)}>
			blah blah blah
		</>

		<Panel title='Etymology' isActive={activeIndex === 1} onShow={()=>setActiveIndex(1)}>
			blah blah blah
		</>	
	)
}

// We can accept an argument ( essentially props ), 
// or destructure it directly like shown here
function Panel = ({
	title,
	children,
	isActive,
	onShow
}) {
	return (
		<section className="panel">
			<h3>{title}</h3>
			{isActive? (<p>{children}</p>) : (<button onClick={onShow}>Show</button>) }
		</section>
	);
}
```


In this case, React is re-rendering Chat when props change. However, `textarea` value comes from `useState`, which is initially empty. Since we didn't update text state via `setText`
, therefore it remains the same.


```To force component to reset its state, we can pass it a different key. Like 
```html
<Chat key={email} />
```
Now switching between recipients will reset Chat component. This is because a different key tells React this is a different chat component, and thus it's re-created from scratch rather than re-rendered. And when it's re-created from scratch, useState will take the initial value of empty so text is cleared.


Uncontrolled component - A component where the parent component can't influence it.
Controlled component - A component where parent component can influence child via props.
When writing a component, consider what info should be controlled by parent ( via props ) and what info should be uncontrolled ( via state ). 

##### A single source of truth for each state
For each unique piece of state, you will have to choose the component that owns it. This is known as single source of truth. It means that for each piece of state, there is a specific component that holds that info. So we lift up the shared state to the common shared parent, and pass it down.

#### Preserving and resetting state

React preserves a component state for as long as it's being rendered at its position in the ui tree.

When you give a component state, that state does not live inside the component, it's held inside React ( which explains why state persist across renders ). React associates each piece of state with the correct component by where that component sits in the render tree.

If we remove the component via updating some state that don't render it due to conditional rendering, then its state disappears. When react removes a component, it destroys its state.


In this case, updating `isFancy` does NOT reset Counter state because Counter sits at the same position. You always have `Counter` as the first child of the div. Because it's the same component at the same position, from React's view, it's the same counter. You can even return 2 jsx based on some conditions. As long as their Counter sits at the same position, the state is retained. React doesn't know where you place conditions, all it sees is the tree you return.


```ts
export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} /> 
      ) : (
        <Counter isFancy={false} /> 
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}
```


**Also when you render a different component in the same position, it resets state of its entire subtree.**

![[Screenshot 2024-03-11 at 23-15-31 Preserving and Resetting State – React.png|650]]


So if you want to preserve state between re-renders, the structure of the tree needs to match up from one render to another. If structure is different, state gets destroyed because React destroys state when it removes a component from the tree.


Never nest component function definitions!
* Note you should not define a component a inside another component b. When you do this, it means a different component a is created for every render of b. React will see that you are rendering a different component in the same position, and reset all the states below. *



##### Resetting state at same position

By default React preserves state of a component while it stays at the same position. If we don't want that, we can use 'key' prop. Which resets the state in the component.

Specifying a key tells React to use that key as part of the position, instead of order within the parent. Note keys aren't globally unique, they only specify position within the parent.

```ts
import { useState } from 'react';

const players = ['Jason', 'Jone', 'George', 'Tim'];

export default function Scoreboard() { 
  const [playerIdx, setPlayerIdx] = useState(0);

  // Without key prop, Counter will have the same score state ( since it's defined within Counter ) when the prop 'person' changes
  return (
    <div>
      <Counter key={playerIdx} person={players[playerIdx]} />
      <button onClick={() => {
        setPlayerIdx((playerIdx + 1) % players.length);
      }}>
        Next player!
      </button>
    </div>
  );
}
```


Another example
Typing a message and switching recipient does not reset the input. 

This is because chat maintains its own state of text, and that state is persisted across renders even though the prop `contact` changes. So adding a key prop to where Chat is returned within the parent will solve this issue.

```ts
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'Chat to ' + contact.name}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>Send to {contact.email}</button>
    </section>
  );
}
```



If you want to reverse the order of two components and keeping their states.
You could have if else branch for rendering, while the order will be different ( so state will be loss ), adding a key prop will fix this and persist the state.

```ts
if (reverse) {
    return (
      <>
        <Field key='lastName' label="Last name" /> 
        <Field key='firstName' label="First name" />
        {checkbox}
      </>
    );
  } else {
    return (
      <>
        <Field key='firstName' label="First name" /> 
        <Field key='lastName' label="Last name" />
        {checkbox}
      </>
    );    
  }
```



Eg. Imagine we have some `<Contact />` components as list items. We can change the order of the list such that . If we want to maintain the state within each Contact, we can do something like key={contact.id} .

```ts
import { useState } from 'react';
import Contact from './Contact.js';

export default function ContactList() {
  const [reverse, setReverse] = useState(false);

  const displayedContacts = [...contacts];
  if (reverse) {
    displayedContacts.reverse();
  }


  // Note here, li key={contact.id} means we persist the component state based on the contact.id. Without key prop, the state is maintained but it would be incorrect when we reverse the list.
  return (
    <>
      <label>
        <input
          type="checkbox"
          value={reverse}
          onChange={e => {
            setReverse(e.target.checked)
          }}
        />{' '}
        Show in reverse order
      </label>
      <ul>
        {displayedContacts.map(contact =>
          <li key={contact.id}>
            <Contact contact={contact} />
          </li>
        )}
      </ul>
    </>
  );
}

const contacts = [
  { id: 0, name: 'Alice', email: 'alice@mail.com' },
  { id: 1, name: 'Bob', email: 'bob@mail.com' },
  { id: 2, name: 'Taylor', email: 'taylor@mail.com' }
];

```




#### Extracting state logic into a reducer

##### useReducer
Components with many state updates across many event handlers can get overwhelming. We can consolidate all state update logic outside your component in a single function called reducer.

Basically, useReducer takes in a reducer fn, and a list of initial tasks.
Event handlers will all dispatch an action with some properties.
In the reducer, we switch on action.type, and then update the state.


So what's changed now is that we simply dispatch actions inside the component for updating. While all the updating logic is moved outside the component.


```ts
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

// fn and initial state
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false }
];

export default function TaskApp() { 
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}
```


We can lift state up by moving state in child component to the parent. And then if we want the child component to trigger update of state, we could pass down setState from parent to child as part of props and trigger it inside the child.

##### Comparing useState and useReducer.
You write less code upfront with `useState`. With `useReducer`, you have to write both a reducer function and dispatch actions.

With debugging, it's easier to debug with `useReducer`, as you can just add a console.log to your reducer to see action and what updated.

Reducer is also easier to test, and it's decoupled from the component, meaning you can test it in isolation.



#### Passing data via Context

Props drilling can be painful, not suitable for things like theming. because you would have to pass it to every component.

1. Create a context.
2. Use that context from component that needs the data.
3. Provide that context from component that specifies the data.



1. Create context.
```ts
// Creating a context
import { createContext } from 'react';


// Here it's default value is 1, you can pass anything, even object
export const LevelContext = createContext(1);
```

2. Use the context.
```ts
// useContext is a hook. Tells React that Heading wants to read LevelContext.
export default function Heading({ children }) {
	const level = useContext(LevelContext);
	// ...
}
```

3. Provide the context.
```ts
// Tells React, if any component inside <Section> asks for LevelContext, give them this level. It will use nearest LevelContext.Provider in the UI tree.
import { LevelContext } from './LevelContext.js'

export default function Section({ level, childrne }) {
	render() {
		<section className='section'>
		<LevelContext.Provider value={level + 1}>
			{children}
		</LevelContext.Provider>
		</section>
	}

}
```


Result with useContext

```ts
// App.js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```ts
// Section
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

// Nested Section will read outer level section to get its level
export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}


// Heading
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

// Note Heading component under each Section will read the level context
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading must be inside a Section!');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

```ts
// LevelContext
import { createContext } from 'react';

export const LevelContext = createContext(0);
```




Before you use context.
1. Try props drilling. It's more explicit.
2. Extract components and pass JSX as children. Eg. Maybe you want to pass data props like `posts` to visual components that don't use them directly. 
Eg. 
`<Layout posts={posts}`

Instead make Layout take children as a prop. This reduces number of layers between the component specifying the data and the one that needs it.
`<Layout> <Post posts={posts} /> <Layout />`
