


#### Select
![[Pasted image 20240408220733.png]]

```html
<label for="cars">Choose a car:</label>
 <!-- select takes a list of options -->
  <select name="cars" id="cars">
    <option value="volvo">Volvo</option>
    <option value="saab">Saab</option>
    <option value="opel">Opel</option>
    <option value="audi">Audi</option>
  </select>
```


#### Links
A html link
```html
<a href="https://www.google.com/">Visit Google</a>
```



#### Label and Input
This is how we create labels and inputs in html.
Input can have optional type attribute, such as `radio`, `checkbox`, `password`.

```html
  <!--for attribute specifies the name of input label is bound to -->
  <label for="cheese">Do you like cheese?</label>
  <input name="cheese" id="cheese" />

```

#### Adding/changing/removing html attributes via setAttribute and removeAttribute

We can use `setAttribute` to add/change attribute of HTML elements.
```ts
let label = document.createElement('label');
label.textContent = 'Some sample label';

let field = document.createElement('input');
field.setAttribute('value', some_value);

// set field as readonly
field.setAttribute('readonly', true);
field.removeAttribute('readonly');


// creating a line break element and appending it to a form element
let br = document.createElement('br');
form.append(br);
```

#### Event listener
```ts
// when user types something
field.addEventListerner('input', event => {});

// when user press a key
document.addEventListerner('keydown', keypress);
function keypress(event) {
	// LEFT = 37, UP = 38, RIGHT = 39, DOWN = 40
	switch(event.keyCode) {}
}



// input events are fired when input changes

// change events are fired when input change + lose focus
// OR select change option
```



#### Querying elements
```ts
// select via id
document.querySelector('#info');

// select via class
document.querySelector('.info');
```


#### Updating element style with js
```ts

function rotateHand(target, ratio) {
	// target is a dom element, in this case we are setting a custom property
    target.style.setProperty('--rotation', ratio * 360);
}
```


#### Dealing with time
```ts
let date = new Date();

// plus days
let endDate = new Date(Date.now() + days*24*60*60*1000);

// or
let endDate2 = new Date(d.getTime() + 1*24*60*60*1000);


// finding hour, min, second from a date object
let hours = date.getHours();
let minutes = date.getMinutes();
let seconds = date.getSeconds();





// find time in between two date objects
let remain_time_in_milliseconds = d2 - d1;
let remain_time_in_seconds = remain_time_in_milliseconds / 1000;

let minutes = Math.floor(remain_time_in_seconds / 60);
let seconds = Math.floor(remain_time_in_seconds % 60);





```