[[#useMemo(fn, deps)]]
[[#useCallback]]
[[#Difference between useMemo and useCallback?]]
[[#When do React components re-renders and what is React.memo? How does React.memo know when props has changed? When to use React.memo? Does it affect state?]]


For performance optimisations.

#### useMemo(fn, deps)
Allows us to remember computed value between renders.

React looks at the deps, if any of deps have changed, React will re-calculate the fn.
Other wise will just return the cached value.


```ts
// a memoized fn
const allPrimes = React.useMemo(() => {
  const result = [];
  for (let counter = 2; counter < selectedNum; counter++) {
    if (isPrime(counter)) {
      result.push(counter);
    }
  }
  return result;
// selectedNum is the component state, essentially the input for this fn
}, [selectedNum]);
```



#### useCallback
```ts
// Recall fn are compared by refs
const functionOne = function() {
  return 5;
};
const functionTwo = function() {
  return 5;
};
console.log(functionOne === functionTwo); // false
```

So when we define a function within our components, it'll be re-generated on every single re-render, producing identical function with a unique reference.

```ts
const handleBoost = React.useCallback(() => {
  setCount(currentValue => currentValue + 1234);
}, []);



// Another example
function Parent({...}) {
	const [a, setA] = useState(0);

	return <Pure onChange={() => doSomething(a)} />
}


// Applying useCallback
function Parent({...}) {
	const [a, setA] = useState(0);

	const onPureChange = React.useCallback(() => doSomething(a), [a]);
	
	return <Pure onChange={onPureChange} />
}
// Note we need to add `a` to deps. If not when state a changes, React will return a fn with old value of a due to closure. 

```



#### Difference between useMemo and useCallback?

`useCallback` serves the same purpose as `useMemo`, but specifically for functions.
```ts
// remember some fn so don't have to re-generate
React.useCallback(function helloWorld(){}, []);

// remember some calculations so don't have to re-compute
React.useMemo(() => function helloWorld(){}, []);

// Example
const fn = () => 42 // assuming expensive calculation here
const memoFn = useCallback(fn, [dep]) // return memoized fn
const memoFnReturn = useMemo(fn, [dep]) // return memoized value, 42
```



#### When do React components re-renders and what is React.memo? How does React.memo know when props has changed? When to use React.memo? Does it affect state?

Recall react components re-renders when its state or prop changes. But it will also re-render when it's parent re-renders, in this last case where it re-renders due to parent, we can optimise performance with `React.memo`.

We can memo components. When a component is wrapped in `React.memo`, React will render the component and memo the rendered output. Before next render, if new props are the same, React will will reuse the memoized output and skip the next rendering.

By default `React.memo` does shallow comparison of props and objects of props.

 * You can use `React.memo` when your component is a pure component (functional component and renders same output given same props)
 * Your component re-renders often and is provided with the same props (a common case is forced re-render due to parent re-render). If component re-renders often but with different props, then no point of using `React.memo.`
 * Note `React.memo` only affects props, memoized components are always re-rendered when state changes.
```ts
// Transform our PrimeCalculator into a pure component:
const PurePrimeCalculator = React.memo(PrimeCalculator);


// Another example
const Movie = ({ title, releaseDate }) => {
	return (
		<div>
			<div>Movie Title: {title}</div>
			<div>Release Date: {releaseDate}</div>
		</div>
	)
}

const memoMovie = React.memo(Movie);

// initial render, memoMovie fn is invoked
<memoMovie title = 'Heat' releaseDate ='1995' />

// Second render, memoMovie fn is not invoked due to same parameter
<memoMovie title = 'Heat' releaseDate ='1995' />

```
