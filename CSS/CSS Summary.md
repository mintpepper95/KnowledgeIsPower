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
```


#### What is specificity and explain order of specificity from most to least?
It's how css decides which rule is applied when we have multiple rules targeting the same element.

![[cssspecificity-calc-1_kqzhog.webp]]

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











