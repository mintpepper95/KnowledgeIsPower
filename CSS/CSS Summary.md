[[#Imagining a basic html, with script and stylesheet]]
[[#What does cascading mean in the context of CSS?]]
[[#What is specificity and explain order of specificity from most to least?]]



#### Imagining a basic html, with script and stylesheet
```html
<!DOCTYPE html>
<html lang='en'>
<head>
	<meta charset='utf-8'>
    <script src="./index.js" defer></script>
    <link rel="stylesheet" href="./index.css">
</head>
<body>
</body>
```


#### What does cascading mean in the context of CSS?
It means that given rules with same specificity, the last one applies.

Also note some CSS properties by default inherit the values from parent.
E.g. `color` and `font-family` will pass down to children.

Things like width, margins, padding and borders don't inherit.
```html
<p>We can change the color by targeting the element with a selector, such as this <span>span</span>.</p>
```
``` css
/* span is also red in this case **/
p {
    color: red;
}



/* looks for p that is direct descendent of div, 
  not nested, not space does not affect selection */
div>span
div > span


/* Selects all span inside div, including nested ones */
div span


/* an edge case, imagine a span nested inside 10 divs, 
  that would be 11 elements, but still lower priority than a class assigned to this span, because the numbers only matter in that position /*

.span-class {}

div.... div > span {}



/* !important overrides everything */
p {
	color: green !important; // this would override even inline styles
}
```

![[Pasted image 20250325005803.png]]

![[Pasted image 20250325011235.png]]

Combining selectors
![[Pasted image 20250325011439.png]]


![[Pasted image 20250325011602.png]]


Universal selector, selects every element inside something

Targets all elements inside plates
![[Pasted image 20250325011835.png]]

Adjacent sibling

A + B, selects all B elements that directly follows after A, that's on the same level.

General sibling

A ~ B

Here, select all chopstick elements that are after food, note chopstick has to be on the same level as food, and can't be nested inside food.
![[Pasted image 20250325012144.png]]



Pesudo classes use `:` while pseudo elements use `::`


First-child pseudo selector
p:first-child, selects any p elements that are first child of another element.


// select orange inside p elements that are first-child of some elements.
plate:first-child orange {
  color: orange;
}

// Select orange that are first child, and they have to be nested inside plate
plate orange:first-child


Nth-child pseudo selector
**div p:nth-child(2)** selects the second **p** in every **div**



Last-child pseudo selector
bento:nth-last-child(3)


First of type 
span:first-of-type, selects the first span in all elements, included nested. As long as a span element is inside some elements and is the first child, then it's selected.


Nth of type
For example, select all the even order elements

plate:nth-of-type(even)

![[Pasted image 20250325015408.png]]


plate:nth-of-type(2n+3)

Selects every second element, starting from the 3rd element. So 3rd, 5th, 7th... element.s



p span:only-of-type

Selects span elements inside p, given that span is the only span item inside p


span:last-of-type
Selects span elements, given they are the last element inside some other elments.

![[Pasted image 20250325020007.png]]


apple:not(.small, .medium)
Selects all apples elements if those apples not of class small or medium


Attribute selector
[type] 
Selects all elements that have the a `type='anything` attribute

Eg. [for] selects the following elements
```
<apple for='Jason' />
<pear  for='Tim' />
<orange for='Oliver' />
```

a[href]
Selects all `<a href='anything' />` elements


Attribute Starts with selector
[attribute^='value']

.toy[category^='Swim']
Selects all toy class elements that have category attribute which starts with 'Swim', eg. category could be `Swimwear` or `Swim Loop`


Attribute Ends with selector
img[src$='.jpg']

Selects all img elements, with src attribute ending with `.jpg`


Attribute Wild card selector
img[src*='/thumbnails/']
Selects all img elements, with `/thumbnails/` containing inside their src attributes




![[Pasted image 20250325021527.png]]








#### What is specificity and explain order of specificity from most to least?
It's how css decides which rule is applied when we have multiple rules targeting the same element.

![[cssspecificity-calc-1_kqzhog.webp]]

![[Pasted image 20250325004718.png]]

##### Some more cascading rule examples

First two rules competing over bg color.

```css
First two rules competing over bg color.

/* specificity: 0101 */
#outer a {
	background-color: red;
}

/* specificity: 0201 */
#outer #inner a {
	background-color: blue;
}



Next two rules competing over color.
/* specificity: 0104 */
#outer div ul li a {
	color: yellow;
}

/* specificity: 0113 */
#outer div ul .nav a {
	color: white;
}



Last 3 rules competing over link hover style.
/* specificity: 0024 */
div div li:nth-child(2) a:hover {
	border: 10px solid black;
}

/* specificity: 0023 */
div li:nth-child(2) a:hover {
	border: 10px dashed black;
}

/* specificity: 0033 */
div div .nav:nth-child(2) a:hover {
	border: 10px double black;
}

```











