Flexible means re-usable.

Imagine a basic component. We can re-use it.
Now suddenly we need a tooltip on hover for some instances of this component.
And we need a visual dot in other instances of this component.

![[Pasted image 20250513123514.png]]

Now this simple `BlockComponent` has so many props.

![[Pasted image 20250513123619.png]]

Re-usable doesn't mean flexible. Flexible is about the ability to understand and augment.

The problem with `BlockComponent` is that in the steps of making it re-usable, we added a lot of business logics into the component. Which makes it hard to be used with a different set of rules. So you would need to understand all the logics and then add more props.

Focus on what it should look like. Don't focus on how to render the different examples into one component, ideally should just be one prop. `<BlockComponent type={} />`

We can then have a separate function for logic.

So now we have `BlockComponent` which handles some common UI, but also specific components for different types of `BlockComponent`.

Note we accept children because it's more flexible as we can pass in whatever we want, without changing this component itself.
![[Pasted image 20250513124856.png]]

We can also specify behaviours.
In the case of `DisabledCta`, note `onClick` handler will never be called, even if passed in by accident.
![[Pasted image 20250513125225.png]]

Now we need to figure out how to pass a type to `BaseCta`, and have it return the specific `Cta` according to that type.

We could use a switch statement. Or we can use an object like a dictionary.

```ts
components = {
	reserve: ReserveCta,
	join: JoinCta,
	cancael: CancelCta,
	disable: DisableCta,
	reactivate: ReactivateCta
}

// now we can grab the correct component to render.
// React allows you to dynamically render a component, note it has to be capitalised
const Component = components[type];
```

See below

![[Pasted image 20250513125826.png]]

Final refactored component

![[Pasted image 20250513125941.png]]

Now we can use individual Ctas alone, we can also add or remove Ctas with ease.