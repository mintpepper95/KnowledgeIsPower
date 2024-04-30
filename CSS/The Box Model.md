[[#Explain the difference between block and inline element space they occupy, width/height, vertical and horizontal margins,]]
[[#What is the box model?]]
[[#What is border-box? How is it different compared to normal box model?]]
[[#Margin collapse (vertical margins only)]]
[[#What is overflow?]]
[[#Values and units]]

---

#### Explain the difference between block and inline element: space they occupy, width/height, vertical and horizontal margins, 

![[Pasted image 20240430224819.png]]
Div is block. a and span are inline.


#### What is the box model?
![[Pasted image 20240430224916.png|350]]


#### What is border-box? How is it different compared to normal box model?

You can turn on border-box at the root or under a specific scope
```css
:root {
	box-sizing: border-box;
}


/* Border-box for specific scope */
.box {
    box-sizing: border-box;
}
```


Border Box
Includes padding, but not margin
![[Pasted image 20240430225020.png|350]]

Normal Box
Padding and margin are both excluded
![[Pasted image 20240430225043.png|350]]


#### Margin collapse (vertical margins only)
If you have two positive value margins touching each other, the larger one becomes the margin.

```css
.one {
    margin-bottom: 50px;
}

.two {
    /* as long as < 50px, margin will be 50px, else will be whatever specified here */
    margin-top: 49px;
}


<div class="container">
    <p class="one">I am paragraph one.</p>
    <p class="two">I am paragraph two.</p>
</div>
```


#### What is overflow?
When content of child leaks outside the parent.
![[Pasted image 20240430225950.png|300]]

overflow:hidden
![[Pasted image 20240430230011.png]]

overflow:scroll
![[Pasted image 20240430230039.png|300]]

#### Values and units
Absolute values - px, pc and etc

Relative values - em, vw, vh and etc

##### What is em?
Em is relative to the font-size of the element, if element has no font-size, they inherit font-size from ancestor. If no font-size defined anywhere, uses browser default font size.
```css
.example {
	font-size: 20px;
	
	/* radius is 10px */
	border-radius: .5em;
	
	/* radius is 40px */
	padding: 2em;
}
```

##### What is pc(percentage)?
Percentages are always relative to some other value.
If you set element font-size as a percentage, it'll be proportional to parent font-size.
If you set element width as a percentage, it'll be percentage to parent width.

1.5em is same as 150%
