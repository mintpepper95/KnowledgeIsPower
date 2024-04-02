[[#What is a generator function and a generator object?]]
[[#Generators are iterables]]
[[#Yield*]]
[[#Passing value into next()]]
#### What is a generator function and a generator object?
A generator fn returns a Generator. A Generator is anything that has `next()` , which returns an object with two properties, `value` and `done`.

We use generators for lazy evaluation.

```ts
function* generateSequence() {
	yield 1;
	yield 2;
	return 3;
}

// Note code execution has not started yet
let g = generateSequence();

// When .next() is called, it runs code until next yield statement, the fn then pauses, and yielded value is returned
// one is an object, one = { value: 1, done: false }
let one = g.next();

// if we call again, returns next yield
let two = g.next();

// Code execution reaches return statement, so finished
let three = g.next(); // three = { value: 3, done: true }
```

#### Generators are iterables

```ts
function* generator() {
	yield 1;
	yield 2;
	yield 3;
}


let gen = generator();
// We can use for...of loop
for (let value of gen) {
	console.log(value);// 1, 2, 3
}
```

#### Yield*
We use `yield* some_generator()` to delegate to another generator, it means we yield each element from `some_generator` until it exhausts, then resume producing its values.

```ts
function * anotherGenerator(i) {
  yield i + 1;
  yield i + 2;
  yield i + 3;
}

function * generator(i) {
  yield* anotherGenerator(i);

  yield 5;
  yield 6;
  yield 7;
}

var gen = generator(1);
for (let i = 0; i < 7; i++) {
	// 2, 3, 4, 5, 6, 7
	console.log(gen.next());
}
```


#### Passing value into next()
We can also pass values into generators, the value we pass in becomes the result of yield.
```ts
function* gen() {
    let ask1 = yield "2+7= ?";
    alert(ask1);
    let ask2 = yield "3*3= ?";
    alert(ask2);
}

let generator = gen();
alert(generator.next().value); // print 2+2= ?
alert(generator.next(4).value); // inside alert 4, then print 3*3= ?
alert(generator.next(9).done); // inside alert 9,then print true
```
![[Pasted image 20240402121925.png|470]]

