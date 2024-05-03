
This is what the Holy Grail looks like.

![[Pasted image 20240503162713.png|550]]

Below are the HTML and CSS.

```html
<!DOCTYPE html>
<head>
    <script defer src="./index.js"></script>

    <link rel="stylesheet" href="./index.css">
</head>

<body>
    <div class="main-container">
        <header class="header">
            <div>HEADER</div>
        </header>

        <div class="middle">
            <nav class="middle-left">
                <div>MENU</div>
                <ul class="middle-left-items">
                    <li>ITEM 1</li>
                    <li>ITEM 2</li>
                    <li>ITEM 3</li>
                    <li>ITEM 4</li>
                    <li>ITEM 5</li>
                </ul>
            </nav>

            <main class="middle-center">
                <h6>CONTENT</h6>
                <p>lorasuhjakdk....</p>
            </main>

            <aside class="middle-right">
                <div>AD</div>
                <div>AD</div>
            </aside>
        </div>

        <!-- To have white gaps,
            margins must be set on the element with background color,
            in this case would be the footer element and not div
        -->
        <footer class="footer">
            <div>FOOTER</div>
        </footer>
        
    </div>
</body>
```

Styling
```css
:root {
    /* make our element appear in center, aligned */
    display: flex;
    justify-content: center;
    align-items: center;
}

  
/* this is container for everything */
.main-container {
    display: flex;
    flex-direction: column;

    border: 5px black solid;
    font-size: 90px;
}

  

/* this is header section */
.header {
    /* We can both horzintal and vertical align text inside header,
    by wrapping text in a div,
    and then making it a flex item under header*/
    display: flex;
    justify-content: center;
    align-items: center;

    background-color: aqua;
    border-bottom: 5px black solid;
}

  

/* this is middle section */
.middle {
    /* its flex items will be laid out in a row */
    display: flex;
    border-bottom: 5px black solid;
}


/* below are the 3 flex items inside middle section */
.middle-left {
    margin-bottom: 80px;
    font-size: 29px;

    /* aligns menu text horizontally */
    text-align: center;
}


/* setting bg color for all the items */
.middle-left-items {
    background-color:palevioletred;
}
  
.middle-center {
    border: 5px black solid;
    border-top: 0;
    border-bottom: 0;
    background-color: bisque;
}


.middle-right {
    background-color:palevioletred;
    margin-bottom: 80px;
    text-align: center;
}

  

/* setting bg color for 'menu' div */
nav div {
    background-color:palevioletred;
}


ul {
    /* so no space between ol/ul container and list items */
    margin: 0;

    /* hides the bullet point */
    list-style-type: none;

    /* use padding for creating empty spaces */
    padding-left: 0px;
    padding-right: 20px;
}

  

main h6 {
    font-size:60px;
    text-align: center;

    /* there's margin (empty space) around heading, we remove the margin
     so content below heading element can appear right underneath it
    */
    margin: 0;
}


main p {
    font-size:16px;
}

  
.footer {
    display: flex;
    justify-content:center;
    align-items:center;

    background-color: aqua;
    margin-bottom: 50px;
}
```