[[#What is a pseudo class element? What is a pseudo element?]]
[[#How to count specificity of a css rule?]]

---

#### What is a pseudo class element? What is a pseudo element?
A pseudo class element selects an element under a specific state. Single colon.
```css
a:hover
a:visited

/* selects all li elements that are second within its parent */
li:nth-child(2)

/* find any article that is a first child (direct or descendent) */
article:first-child {
    font-weight: bold
}
```

A pseudo element selects a specific part of an element. Double colon.
```css
p::first-line
```


#### How to count specificity of a css rule?

Order of decreasing specificity
* Inline style
* number of id selectors
* number of classes and pseudo-classes(elements in specific states, `a:hover`)
* number of tags and pseudo-elements (styles a specific part of an element, `p::first-line`)


##### Some examples
```css
/* 024 */
div div li:nth-child(2) a:hover {
        border: 10px dashed black;
}
```
![[Pasted image 20240430222614.png|200]]


```css
/*  033 */
div div .nav:nth-child(2) a:hover {
	border: 10px double black;
}
```
![[Pasted image 20240430222834.png|300]]








