[[#var]]
[[#let]]
[[#const]]

What's the difference between the 3 declarations?

#### var
`var` is global when declared outside function, and it's function scoped when declared inside the function.

`var` declarations can be hoisted to the top, see [[Hoisting]].

```js
// We might accidently re-define the variable
var greeter = 'hi';
if (cond === true) {
	var greeter = 'bye';
}
```


#### let
Block scoped. It ends at next `}`. `let` can be updated, but not re-declared.
While `var` can be re-declared.

It's perfectly safe to return an object declared with `let`. JS garbage collection will only ever clean up something that's not in use.

Like `var`, `let` declarations can also be hoisted to the top.


#### const
Block scoped like `let`. Can not be updated or redeclared.
Note although `const` object can't be updated, its properties can.
Must be initialised during declaration.
Can't be re-assigned.





