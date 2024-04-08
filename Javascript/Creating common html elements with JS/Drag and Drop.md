```html
    <div id='drag'>
        <div class='corners top-left'></div>
        <div class='corners top-right'></div>
        <div class='corners bottom-left'></div>
        <div class='corners bottom-right'></div>
    </div>
```

We can use html5 drag to implement drag and drop on this div

```css
#drag {
    position: absolute;
    
    top: 50%;
    left: 50%;

    width: 200px;
    height: 200px;

    background-color: Orange;

	/* transition animation */
    transition: 0.4s;
}
```

```ts
let drag_div = document.querySelector('#drag');

// make element draggable
drag_div.setAttribute('draggable', 'true');

// handling drag start
drag_div.addEventListener('dragstart',drag_start,false);

// handling dragging
document.body.addEventListener('dragover',drag_over,false);

// handling drop
document.body.addEventListener('drop',drop,false);



// drag start set event data
function drag_start(event) {
    // window.getComputedStyle'll retrieve the current, actual, properties of an element
    var style = window.getComputedStyle(event.target, null);

    // The DataTransfer object is used to hold the data that is being dragged during a drag and drop operation. It may hold one or more data items, each of one or more data types.
    event.dataTransfer.setData("text/plain",
    (parseInt( style.getPropertyValue("left") ,10) - event.clientX) + ',' + (parseInt(style.getPropertyValue("top"),10) - event.clientY));

}


// drag over
function drag_over(event) {
    // no drop event is triggered if we use a default drag_over event
    event.preventDefault();
    event.stopPropagation();
}

// drop
function drop(event) {
    let drag_div = document.querySelector('#drag');
    var offset = event.dataTransfer.getData("text/plain").split(',');

    // update div position, It would be better if we keep the initial shift of the element relative to the cursor.
    // which is why we store offset
    drag_div.style.left = (event.clientX + parseInt(offset[0],10)) + 'px';
    drag_div.style.top = (event.clientY + parseInt(offset[1],10)) + 'px';

    // we can resize by calculating width and height
    // we prevent default of opening up a link
    event.preventDefault();
    event.stopPropagation();
}
```

