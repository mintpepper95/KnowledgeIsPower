

dynamic class??

focus fetching data first, then map to a list of nodes. And set this node data via useState. When state is defined, render those nodes.

a bit confused with destructuring syntax, note we need it in the parameter

try using export default

writing html semantic, don't div


use onMouseOver and onMouseLeave cause they bubble which is what we want
put className highlight and onMouseOver and onMouseLeave all on the same place, which is outer parent





Think how to expand, where to put useMouseEnter/Leave, and semantic html

```html
// if we want to use nav
<nav class="crumbs">
  <ol>
    <li class="crumb"><a href="#">Bikes</a></li>
    <li class="crumb"><a href="#">BMX</a></li>
    <li class="crumb">Jump Bike 3000</li>
  </ol>
</nav>
```

Ask whether we should use nav + ul + a href

Placeholder link
`<a href="#">Future Link</a>`


a tags can't be a descendant of a tag. Use like a span/div instead.