[[#What is a custom CSS property and why should we use it?]]
[[#How to create a custom property in css? What's its scope? And how do we use it in other places in the css?]]
[[#How to access custom properties in JS?]]
[[#How to set values of custom properties in js?]]

---
#### What is a custom CSS property and why should we use it?
Properties that are prefixed with `--`, e.g. `--name` are custom properties which can be used in other declarations with `var()`.

Style sheets can have CSS with repeated values. `--main-text-color` is easier to understand than `#00ff00`.


#### How to create a custom property in css? What's its scope? And how do we use it in other places in the css?
```css
/* the selector given is the scope ( so selector element or child of selector ) where custom property can be used. So a common practice is to define custom property at the root so it can be used globally */

section {
	--main-bg-color: brown;
}

:root {
	--main-bg-color: brown;
}


/* to use it */
.details {
	background-color: var(--main-bg-color);
}
```

Note custom properties are case sensitive, so `--my-color` != `--my-Color`.

#### How to access custom properties in JS?
```ts
// retrieve css property as string
window.getComputedStyle(element).getPropertyValue('--some_var');
```


#### How to import styles into a stylesheet from other stylesheets?
```css
/* browser will start another stylesheet request after loading the initial css, means they are downloaded sequentially */
@import 'my-own-styles.css';
```

#### How to set values of custom properties in js?
```ts
setInterval(() => {
    let date = new Date();
    let seconds_ratio = date.getSeconds() / 60;
    let minutes_ratio = (date.getMinutes() + seconds_ratio) / 60;
    let hours_ratio = (date.getHours() + minutes_ratio) / 12;


    let seconds_hand = document.querySelector('.seconds-hand');
    let minutes_hand = document.querySelector('.minutes-hand');
    let hours_hand = document.querySelector('.hours-hand');


    seconds_hand.style.setProperty('--rotate', seconds_ratio * 360);
    minutes_hand.style.setProperty('--rotate', minutes_ratio * 360);
    hours_hand.style.setProperty('--rotate', hours_ratio * 360);
}, 1000)
```