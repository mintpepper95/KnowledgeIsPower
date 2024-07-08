For dealing with a large list of items.

Rendering only items on the screen in a dynamic list instead of the entire list to improve rendering performance.

We can fetch more items as user scrolls ( pagination ), and unload previous rows and replacing them with new ones.

#### How it works
Initially list virtualisation only renders the visible items that fit within the container's in the DOM.

As user scrolls through the list, a scroll event is triggered. Virtualisation library listens to this event and calculates which items should be visible based on scroll position.

As you scroll, the virtualisation library dynamically renders new visible items while removing off screen ones.


#### React-window
```ts
import React from "react";  
import { FixedSizeList } from "react-window";  

// Takes two argument, 
// first is an iterable object, meaning array will have 100000 elements that are undefined
// second is a map function that's called on every element of array being created
const data = Array.from({ length: 100000 }, (_, index) => `Item ${index}`);  

// This fn is a ui component that renders each piece of data
const renderRow = ({ index, style }) => (  
  <div style={{ ...style, display: "flex", alignItems: "center", borderBottom: "1px solid lightgrey" }}>  
    <span>{data[index]}</span>  
  </div>  
);  

// Rendering only visible items on screen
// FixedSizeList implicitly passes props to children
const VirtualizedListExample = () => (
	// outer div
  <div style={{ height: "400px", width: "300px", border: "1px solid lightgrey" }}> 
	// Dynamic list
    <FixedSizeList  
      height={400}  
      width={300}  
      itemCount={data.length}  
      itemSize={40} // Height of each item (row)  
    >  
      {renderRow}  
    </FixedSizeList>  
  </div>  
);  
  
export default VirtualizedListExample;
```


#### Benefits and limitations
Improved performance - by rendering only visible items, virtualisation reduces initial load time and minimize DOM manipulation (virtualised lists will re-use existing DOM nodes for new visible items) required during scrolling.

Reduced memory usage - virtualisation keeps only a limited number of items on the DOM, which reduces memory usage.

##### Limitation
Extra complexity
Breaks ctrl + f find on browsers since most items will be in a virtual state, and can't be found on the DOM, so search doesn't work.



