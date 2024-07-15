Confusion is caused by high cognitive load. Since we spend far more time reading than writing, we should avoid excessive cognitive load into our code.

There are two types of cognitive load
* Intrinsic - caused by the inherent difficulty of a task. It can't be reduced.
* Extraneous - caused by the way info is presented, factors not directly relevant to the task, can be reduced, this is what we will focus on.

### Some examples
#### Complex conditionals
```ts
// BAD, confusing
if (val > constant
	&& (cond1 || cond2)
	&& (cond4 && !cond5)
) {
	// do stuff
}

// GOOD, introduce variables with meaningful names
isValid = val > someConstant;
isAllowed = cond1 || cond2;
isSecured = cond4 && !cond5;

if isValid && isAllowed && IsSecured {
	// do stuff
}
```

#### Nested Ifs

Use guard clauses
```ts
// BAD
if Valid {
	if isSecure {
		// do stuff
	}
}

// GOOD
if !isValid || !isSecure;
	return

// do stuff
```


#### Inheritance nightmare

```ts
AdminController extends UserController extends GuestController extends BaseController
// wtf

```

#### Too many small methods, classes or modules
We want to maintain a balance between SOLID and having too many methods/classes

#### Feature rich languages
Reduce complexity by limiting number of choices.
So we don't have to understand why it's done this way than some other way.

#### Don't abuse DRY
A little copying is fine. Don't want to create tight coupling between unrelated components.