[[#Explain briefly difference between flexbox and grid and how to layout a basic flexbox]]
[[#Explain main axis, cross axis and their relations with justify-content and align-items properties]]
[[#Explain flex direction and flex wrap]]
[[#Flex flow shortcut]]
[[#What is flex-basis]]
[[#What is flex-shrink]]
[[#What is flex-grow? What two things happen when you define flex-grow on a flex-item?]]





---
#### Explain briefly difference between flexbox and grid and how to layout a basic flexbox
![[Flexbox-vs-CSS-Grid.webp|550]]


To layout a flexbox, we set display to `flex` on a parent container. Now all of its direct children will become flex items. 

By default flex container is block level element. We can make a flex container inline by setting display to `flex-inline` instead of `flex`. 

![[Pasted image 20240427191859.png|500]]


### Flex Model
##### Explain main axis, cross axis and their relations with justify-content and align-items properties

![[Pasted image 20240427192406.png|500]]

##### Main Axis
Axis running in direction of the flex items being laid out.
So if flex items laid out horizontally, that's row direction, main axis is therefore the horizontal axis. Conversely if flex items are laid out vertically, that's column direction, so main axis is the vertical axis.

`justify-content` controls alignment of items along main axis.

##### Cross Axis
Runs perpendicular to the main axis. 

`align-items` controls alignment of items along cross axis.

##### Possible values of justify-content
Greyish color is the empty space.
![[Pasted image 20240427200504.png|500]]
##### Possible values of align-items
![[Pasted image 20240427200724.png|500]]



#### Img
A Flex Item is default stretched by **align-items: stretch**. See above.
We need to set `align-items` to `center` on the flex container to properly display the img.




#### Explain flex direction and flex wrap
##### Flex direction
`flex-direction` specifies which direction main axis runs, default is row, can be set to column.
##### Flex wrap
When you have a fixed width or height, eventually your flexbox children will overflow outside their container, which will display the scrollbar and breaking the layout.
![[Pasted image 20240427193119.png|500]]

We can correct above by setting `flex-wrap` to `wrap`

#### Flex flow shortcut
`flex-flow` is a shorthand for `flex-direction` and `flex-wrap`
```css
flex-direction: column;
flex-wrap: wrap;

/* equivalent to above */
flex-flow: row wrap;
```


#### Explain flex-grow, flex-shrink and flex-basis and the shortcut to these

`flex-basis`, `flex-grow`, `flex-shrink` are related to responsive design and how much flex items occupy when we adjust browser window.

##### What is flex-basis
Initial width(row direction)/height (column direction) of the flex item.
It will override width/height properties if `flex-basis` is set to other than `auto`

Has no effect on absolutely positioned flex items ( since they are outside of normal doc flow)

##### What is flex-grow? What two things happen when you define flex-grow on a flex-item?

How fast the flex item expands as the available space of a flex container grows (when space is available, eg. when we enlarge and stretch browser window)

Flex-grow property is used on flex items. 
Default value is 1. 

Two things happen when you define `flex-grow` on a flex item.
* It makes flex item grow all the way to the edge of container if there are space available
* It specifies how much a particular flex item grows relative to other flex items in the container.

Make sure you add it to all flex items if you want to use it. By setting different `flex-grow` values on different flex items, as we adjust browser window (flex-container available space), we will see items growing at different rate. Higher value `flex-grow` items will grow faster.


##### What is flex-shrink
How fast the flex item shrinks as we the available space of a flex container shrinks (when space is limited, eg. when we make browser window smaller).

Similar to `flex-grow`, defines how much flex item shrinks relative to other flex items. If you don't want some item to shrink then set it to 0.


##### Flex property and what is flex: 1?

`flex: 1` sets `flex-grow` and `flex-shrink` to 1 and `flex-basis` to 0 (flex-basis 0 means no initial starting value, flex items will take available space equally). If you want flex items to take up all free space along main axis.

Note the space it takes up is after things like margin and padding.

Basically says the element can grow as much as it can, it can shrink as much as it needs to, and it starts at a size of 0.

You should do this instead of `width: 100%` for each flex item.

![[Pasted image 20240427195412.png|500]]



If we set flex: 3 for the last flex item, total unit will be 1 + 1 + 1 + 3 = 6. This means last element will take up half of the space. The remaining space is divided between the remaining 3 flex items.
![[Pasted image 20240427195803.png|500]]


#### An example of a nested flexbox

![[Pasted image 20240427195901.png|550]]

DOM tree
![[Pasted image 20240427195914.png]]

CSS rule for this layout
```css
/* first set flex container */
section {
    display: flex;
}

/* set flex values on articles */
article {
    flex: 1 200px;
}

/* set the 3rd article's children to lay out like flex items */
article:nth-of-type(3) {
    flex: 3 200px;
    
    display: flex;
    flex-flow: column;
}


/* style the first div of 3rd article */
article:nth-of-type(3) div:first-child {
    flex:1 100px;
    display: flex;
    flex-flow: row wrap;
    align-items: center;
    justify-content: space-around;
}

/* finally set some styling on buttons */
button {
    /*
buttons will take up much space
as they can and sit on same line,
but if they can't fit on same line, they drop down
*/
    flex: 1 auto;
    margin: 5px;
    font-size: 18px;
    line-height: 1.5;
}
```





