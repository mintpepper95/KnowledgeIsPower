
* Function overloading
* Optionals 
* Tuples
* Generics
* Generics with keyof
* Typescript Utility types: Partial, Required, Pick, Omit
* Readonly and Immutability
* Enums and literal types
* Classes and member visibility
* Generics in classes
* Mapped Types
* Readonly and static in classes
* Abstract classes
* Mixins
* Conditional Types
* Utility Types Part 2



#### Function overloading
Types DO NOT exist at run time.

1. Define overload signatures
2. Define implementation signatures
```ts

interface Coordinate {
	x: number;
	y: number;
}

// 1. Define overload signtures
function parseCoordinate(obj: Coordinate): Coordinate;
function parseCoordinate(x: number, y: number): Coordinate;
function parseCoordinate(str: string): Coordinate;


// 2. Define implementation signature which is the actual implementation, it must be generic enough to account for all overload signatures. They aren't visible.

// unknown is any, but you have to cast it before using it
// implementation needs to account both definitions, so we make arg2 optional
function parseCoordinate(arg1: unknown, arg2?: unknown): Coordinate {
	let coord: Coordinate = {
		x: 0,
		y: 0
	}
	// we are doing typechecking at runtime here, typeof will return object
	// types DO NOT exist in runtime context
	if (typeof arg1 === 'object') {
		coord = {
			// We tell ts arg1 is Coordinate
			...(arg1 as Coordinate) 
		}
	} else if (typeof arg1 === 'string') {
		(arg1 as string).split(',').forEach(str => {
			const [key, value] = str.split(':');
			// TS doesn't know what key is, so we cast it
			coord[key as 'x' | 'y'] = parseInt(value, 10);
		})
	} else {
		x: arg1 as number,
		y: arg2 as number,
	}
	return coord;
}

// When you call this method, VSCode will tell you there's two signatures to choose from
```



Optionals
Note optional fields has to go after required
```ts
// Optional parameters
function printIngredient(quantity: string, ingredient: string, extra?: string) {
	console.log(`${quantity} ${ingredient} ${extra ?  `${extra}` : ''}`);
}

// Optional fields
interface User {
	id: string,
	info?: {
		email?: string;
	}
}
function getEmail(user: User) {
	return user?.info?.email ?? '';
}


// Optional callbacks
function addWithCallback(x: number, y: number, callback?: () => void ) {
	// only invokes if exist
	return callback?.();
}
```

Tuples
```ts
type ThreeDCoordinate = [x: number, y: number, z: number];

function add3DCoordinate(c1: ThreeDCoordinate, c2: ThreeDCoordinate) {
	return [
		c1[0] + c2[0],
		c1[1] + c2[1], 
		c1[2] + c2[2], 
	];
}


function simpleStringState(initial: string): [() => string, (v: string) => void] {
	let str: string = initial;
	// This is a closure, this str is captured
	return [
		() => str,
		(v: string) => {
			str = v;
		}
	];
}


cont [str1getter, str1setter] = simpleStringState('hello');
str1getter(); // gets str
str1setter('goodbye');
```


Generics
```ts
// Generics
function simpleState<T>(initial: T): [() => T, (v: T) => void] {
	let val: T = initial;
	// This is a closure, this str is captured
	return [
		() => val,
		(v: T) => {
			val = v;
		}
	];
}

// VSCode will hint you, depending on the input you type into the fn
const [str1getter, str1setter] = simpleState(10);

// I want null and string, to achieve it, specify the type like below
const [str2getter, str2setter] = simpleState<string | null>(null);



// ranker
interface Rank<T> {
	item: T
	rank: number
}


function ranker<RankItem>(items: RankItem[], rank: (v: RankItem) => number): RankItem[] {

	// generate a list of item with score
	const ranks: Rank<RankItem>[] = items.map(item => ({item, rank: rank(item)}));

	// sort by score
	ranks.sort((a, b => a.rank - b.rank));

	// remove the score
	return ranks.map(rank => rank.item);
}
```


Generics with keyof

More strict typing on input when invoke the function, basically prevents you from messing up

```ts
// This is saying, we are forcing KeyType to be a key that exist in data type
// KeyType is a custom type we defined that cover the keys in T
function pluck<T, KeyType extends keyof T>(items: T[], key: KeyType): T[KeyType][] {
	return items.map(item => item[key]);
}


const dogs = [
	{ name: 'Mimi', age: 12},
	{ name: 'LG', age: 13},
]

// VSCode knows it's either name or age
console.log(pluck(dogs, 'age'));
console.log(pluck(dogs, 'name'));





interface BaseEvent {
	time: number;
	user: string;
}

interface EventMap {
	// addToCart has time, user and quantity
	addToCart: BaseEvent & { quantity: number; productID: string; },
	checkout: BaseEvent
}

// name is addTocart/check/, data is EventMap[Name]
function sendEvent<Name extends keyof EventMap>(name: Name, data: EventMap[Name]): void {

	console.log([name, data])
}

// VSCode will hint us to pick addToCart or checkout,
// if we pick addToCart, it will then prompt all the properties due to how we defined data above
sendEvent('addToCart', { productId: 'foo', user: 'baz', quantity: 1, time: 10});
sendEvent('cehckout', { user: 'bak', time: 20});

```


Utility Types

Partial - takes a type, and make everything optional

Required - takes a type, makes everything required

Pick -  takes only the specified properties

Omit - omits the specified properties

```ts

interface MyUser {
	name: string;
	id: string;
	email?: string;
}

const merge = (user: MyUser, overrides: Partial<MyUser>): MyUser => {
	// overrides updates user properties
	return {
		...user,
		...overrides
	}
}

// Pick, we can use this in fn definition to restrict the methods we can call
// Here we specify we can only access email, which is optional, and name, which is required
type JustEmailAndName = Pick<MyUser, 'email' | 'name'>;

// The omit will make sure id is omitted from user
// We have to actually omit it ourself btw
// MyUser['id'] will be the type of id property, in this case it's string
const mapById = (users: MyUser[]): Record<MyUser['id'], Omit<MyUser, 'id'>> => {
	// a is initially an empty object,
	// for each user v in users, we add a new property where user[id] is the key, and the user object is the value. The spread operator is to create a new object that includes this new property.
	return users.reduce((a, v) => {
		// we pull out id and other from v
		const { id, ...other } = v;
		return {
			...a,
			[id]: other,
		}
	}, {});
}

```


Readonly and Immutability
```ts
interface Cat {
	// we add readonly to any fields we don't wanna change
	name: string;
	breed: string;
}

// or we can make whole Cat readonly
function makecat(name: string, breed: string): Readonly<Cat> {
	return {
		name,
		breed
	}
}

// now we can't change cat
const usl = makeCat('Usul', 'Tabby');

// for const, we can still update array itself, so we use readonly
const reallyConst = [1, 2, 3] as const; // We can't update values now
```


Enum and literal types
```ts
enum LoadingState {
	beforeLoad = 'beforeLoad',
	loading = 'loading',
	loaded = 'loaded',
}


const isLoading(state: LoadingState) => state === LoadingState.loading;



// Literal types, eg. we want to constraint dice
// here dice accepts only 1/2/3
function rollDice(dice: 1 | 2 | 3): number {
let pip = 0;
	for (let i = 0; i < dice; i++) {
		// because random() gives [0, 1)
		pip += Math.floor(Math.random() * 6) + 1	
	}
	return pip;
}
```


Classes and member visibility
```ts
interface Database {
	get(id: string): string;
	set(id: string, value: string): void;
}


interface Persistable {
	saveToString(): string;
	restoreFromString(storedState: string): void;
}

// implements the interface
class InMemoryDatabase implements Database {
	// so can't be modified outside class
	protected db: Record<string, string> = {};

	get(id: string): string {
		return this.db[id];
	}

	set(id: string, value: string): void {
		this.db[id] = value;
	}
}


// Persistable db, it gets everything InMemoryDatabase
class PersistentMemoryDB extends InMemoryDatabase implements Persistable {
	saveToString(): string {
		// protected makes db visible here
		return JSON.stringify(this.db);
	}

	restoreFromString(storedState: string): void {
		this.db = JSON.parse(storedState);
	}
}


// test
const myDB = new InMemoryDatabase();
myDB.set('foo', 'bar');


const myDB2 = new PersistentMemoryDB();
myDB2.restoreFromString(saved);

```


Generics in Typescript
```ts
// both key and value are generics
interface Database<T, K> {
	get(id: K): T;
	set(id: K, value: T): void;
}


interface Persistable {
	saveToString(): string;
	restoreFromString(storedState: string): void;
}

type DBKeyType = string | number | symbol;

// implements the interface
class InMemoryDatabase<T, K extends DBKeyType> implements Database<T> {
	// so can't be modified outside class
	// cause empty obj can be anything, so we have to cast it
	protected db: Record<K, T> = {} as Record<K, T>;

	get(id: K): T {
		return this.db[id];
	}

	set(id: K, value: T): void {
		this.db[id] = value;
	}
}

// we limit key type
class PersistentMemoryDB<T, K extends DBKeyType> extends InMemoryDatabase<T, K> implements Persistable {
	saveToString(): string {
		// protected makes db visible here
		return JSON.stringify(this.db);
	}

	restoreFromString(storedState: string): void {
		this.db = JSON.parse(storedState);
	}
}


// test, notice me specify the type
const myDB2 = new PersistentMemoryDB<number, string>();
myDB2.set('fool', 12);
```


Mapped Types
```ts

type MyFlexibleDogInfo = {
	name: string;

	// same as below
	// for any key of type string, we want string | number value
	[key: string]: string | number;

// addind this record allows us to specify more properties of key & vlaue both being string
} // & Record<string, string>;

const dog: MyFlexibleDogInfo = {
	name: 'LG',
	age: '12'
}




interface DogInfo {
	name: string;
	age: number;
}

type OptionsFlags<Type> = {
	// for each property of the Type, remap it as null
	[Property in keyof Type]: null;
};

// takes original DogInfo, and makes both name and age boolean
type DogInfoOptions = OptionsFlag<DogInfo>;


type Listeners<Type> = {
	// for each property, we specify newValue as type of that property, can be optional
	// I added double templates symbols intentionally to fix incorrect code highlighting in obsidian
	[Property in keyof Type as `on${Capitalize<string & Property>}Change``]?: (newValue: Type[Property]) => void;
} & {
	[Property in keyof Type as `on${Capitalize<string & Property>}Delete``]?: (newValue: Type[Property]) => void;
}


// DogInfoListeners type will have all the optional 'onXXX' properties
type DogInfoListeners = Listeners<DogInfo>;



function listenToObject<T>(obj: T, listners: Listeners<T>): void {
	throw 'needs implement'
}

const lg: DogInfo = {
	name: 'LG',
	age: 13
}


listenToObject(lg, {
	onNameChange: (v: string) => {},
	onAgeChange: (v: number) => {},
	onNameDelete: () => {},
	onAgeDelete: () => {},
} )
```


Readonly and static in classes
```ts

class Doogy {
	// member constructor
	constructor(public readonly name: string, public readonly age: number) {

	}
}

class DogList {
	private doggies: Doggy[] = [];

	// hide ctor, so we can't create more
	private constructor() {};

	static addDog(dog: Doggy) {
		DogList.instance.doggies.push(dog);
	}

	static instance: DogList = new DogList();

}
```


Abstract classes
```ts

abstract class StreetFighter {
	constructor() {}
	
	move() {}

	fight() {
		console.log(`${this.name} attacks with ${this.getSpecialAttack()}`);
	}

	abstract getSpecialAttack(): string {
	}

	abstract get name(): string {}
}



class Ryu extends StreetFighter {
	getSpecialAttack(): string {
		return 'Hadoken';
	}

	get name(): string {
		return 'Ryu';
	}
}


class ChunLi extends StreetFighter {
	getSpecialAttack(): string {
		return 'Lightning Kick';
	}

	get name(): string {
		return 'Chun-Li';
	}
}

```


Mixins
```ts

function myLogFunction() {
	return (str: string) => console.log(str)'
}

const logger = myLogFunction();
logger('logged');


// You can return a class
function createLoggerClass() {
	return class MyLoggerClass {
		private completeLog: string = '';

		log(str: string) {
			console.log(str);
			this.completeLog += str + '\n';
		}
		
		dumpLog() {
			return this.completeLog;
		}
	}
}

const MyLogger = createLoggerClass();
const logger = new MyLogger();
logger.log('foo');
logger.dumpLog();




function CreateSimpleMemoryDataBase<T>() {
	return class SimpleMemoryDatabse {
		private db: Record<string, T> = {};

		set(id: string, value: T) {
			this.db[id] = value;
		}

		get(id: string, value: T) {
			return this.db[id];
		}

		getObject(): object {
			return this.db;
		}
	}
}


const StringDatabase = CreateSimpleMemoryDataBase<string>();
const sbd1 = new StringDatabase();
sbd1.set('a', 'hello');

// Constructor<T> is any ctor function that takes in any number of arguments for any type, and returns a value of any type, it's very generic.
type Constructor<T> = new (...args: any[]) => any;

// Dumpable takes in a type T that is a ctor fn (Base) and has getObject method
function Dumpable<T extends Constructor<{getObject(): object}>>(Base: T) {
	return class Dumpable extends Base {
		dump() {
			console.log(this.getObject());
		}
	}
}


const DumpableStringDatabase = Dumpable(StringDatabase);
const sbd2 = new DumpableStringDatabase();
sbd2.set('yo ', 'hello');
sbd2.dump();

```


Conditional Types
```ts
// you have to install it, can do with yarn add node-fetch
// node-fetch has no type definitions, yarn add @types/node-fetch, so we get hinting
import fetch from 'node-fetch';

interface PokemonResults {
	 count: number;
	 next?: string;
	 previous?: string;
	 results: {
		 name: string,
		 url: string
	 }[];
}


// conditional type, type is different depending on whether T is assignable/compatiable to undefined
type FetchPokemonResult<T> = T extends undefined? Promise<PokemonResults[]> : void;

// A generic type T that can be either undefined or a function that takes in PokemonResults and returns void, T extends something meaning narrowing down T to a certain type, that T must be assignable to the specific type
function fetchPokemon<T extends undefined | ((data: PokemonResults) => void)>(url: string, cb?: T) {

	if (cb) {
		fetch(url).then(resp => resp.json()).then(cb);
		return undefined as FetchPokemonResult<T>;
	} else {
		fetch(url).then(resp => resp.json()) as FetchPokemonResult<T>;
	}
}

fetchPokemon('...url', data => {
	data.results.forEach(pokemon => console.log(pokemon.name));
})


(async function() {
	// using generic syntax for casting to PokemonResult
	const data = <PokemonResults>(await fetchPokemon('...url'));
	data.results.forEach(pokemon => console.log(pokemon.name));

})();

```


Utility Types Part 2

`Parameters<Type>` - allows you to access the parameter types dynamically
```ts
type AddFunction = (x: number, y: number) => number;  

type AddFunctionParams = Parameters<AddFunction>;  // AddFunctionParams is inferred as [number, number]`

// Here we are extracting parameter type `T` fomr function type `<T>(arg: T) => T`
type T2 = Parameters<<T>(arg: T) => T>;

// type T2 = [arg: unknown]

```

`ReturnType<Type>` - similar to `Parameters<Type>`, allows you to get return type
```
// T0 = string
type T0 = Return< () => string >;


// T2 = unknown
type T2 = ReturnType< <T>() => T >;


// T3 = number[]
type T3 = ReturnType< <T extends U, U extends number[] >() => T >;

```


`ConstructorParameters` - Construct a tuple  from types of constructor fn
```ts
class C {
	constructor(a: number, b: string) {}
}

// T3 = [a: number, b: string]
type T3 = ConstructorParameters<typeof C>;

```


```ts
// Parameter type
type Name = {
	first: string;
	last: string;
}

// It will give you back something that has Name, as well as a fullname property
// For combining multiples types into one
fnction addFullName(name: Name): Name & { fullName: string; } {
	return {
		...name,
		fullName: `${name.first} ${name.last}`
	}
}

// T will match any functions
function permuteRows<T extends (...args:any[]) => any>(iteratorFunction: T, data: Parameters<T>[0][]): ReturnType<T>[] {
	return data.map(iteratorFunc);
}

// using it
permuteRows(addFullName, [{ first: 'jack', last: 'herr' }]);



class PersonWithFullName {
	constructor(public name: Name) {}

	get fullName() {
		return `${this.name.first} ${this.name.last}`;
	}
}

// T will match any constructors, 
function createObjects<T extends new (...args:any[]) => any>(ObjectType: T, data: ConstructorParameters<T>[0][]): InstanceType<T> {
	return data.map(item => new ObjectType(item));
}

// using it
createObject(PersonWithFullName, [{ first: 'jason', last: 'xu' }]);












```









