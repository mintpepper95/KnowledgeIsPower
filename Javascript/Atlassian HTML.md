Creating a basic set of radios, they must have same name if you want to define a radio group, where you can only select one radio from a group.

The id of radio is used by label to associate labels with radio buttons.

If we omit value for radios, then when submitting form data, the value `on` is assigned.

Note we have a default checked state, to select a radio button by default.

When you add event listener with `change` event to radios belong to same names, only the radio that is selected will invoke it's listening method.

So if you want to update other radio buttons you have to query them and update them individually inside the listening code.

```html
     <div>
      <input type="radio" name='shoes' id ='adidass' value='adidas' checked />
      <label for="adidass">Adidas shoes</label>

      <input type="radio" name='shoes' id='nike' value='nike'>
      <label for="nike">Nike shoes</label>

      <input type="radio" name='shoes' id="puma" value="puma">
      <label for="puma">Puma shoes</label>
     </div>
```



```
/*

We have a form where users submit their personal information. Recently we got complaints that it doesn't work as expected.

  

We identified the following issues that need fixing:

  

- All invalid inputs get highlighted when clicking on the Submit button. This is correct, but the disabled inputs should not get highlighted: Email, "Phone Number" and "Company Name". Lastly, if the "Is Employed" checkbox is checked, "Company Name" should be enabled and highlighted if it's empty upon submission.

  
  
  

- When the "Is Employed" checkbox is checked, the "Company Name" input gets enabled, and the user can type a value there. When we uncheck the "Is Employed" checkbox, the "Company Name" input should reset its value and be empty. Currently, it does not.

  
  
  

- When the "Is Employed" checkbox is checked, the "My Income Sources" radio buttons should change to the "I Have Passive Income" checkbox. It should be checked if the "I Have Passive Income" radio button was selected. When the "Is Employed" checkbox is unchecked, the radio buttons should be shown again, and the appropriate radio button should be selected based on the checkbox state.

*/


use hidden

  

const isEmployed = document.querySelector("input[type=checkbox]");

  
  

const radio = document.querySelector('input[type="radio"][name="passiveIncome"]');

console.log(radio);

radio.addEventListener('change', (ev) => {

  if (ev.target.checked === true) {

    isEmployed.checked = true;

  }

})

  
  

isEmployed.addEventListener('change', function() {

  const companyName = document.getElementById("companyName");

  companyName.disabled = !companyName.disabled;

  if (companyName.disabled === true) {

    companyName.value = '';

  }

  if (isEmployed.checked === true ) {

      const radio = document.querySelector('input[type="radio"][name="passiveIncome"]');

      radio.checked = true;

  }

});

  

function isValid(value) {

  if (!(/^[a-z0-9]+$/i).test(value)) {

    return false;
  }

  if (value.length > 10 || value.length < 3) {

    return false;

  }

  return true;

}

  

const inputs = document.querySelectorAll("input:not([type=checkbox])");

  

inputs.forEach(input => {

  input.addEventListener('change', function() {

    if (isValid(input.value)) {

      input.parentElement.classList.remove("invalid");

    } else {

      input.parentElement.classList.add("invalid");

    }

  });

});

  

const submitBtn = document.getElementById("submitBtn");

  

submitBtn.onclick = () => {

    const inputs = document.querySelectorAll("input:not([type=checkbox])");

  

    inputs.forEach(input => {

        // highlight company name if empty upon submission

        if (input.id === 'companyName' && input.textContent == '') {

          input.focus();

        }

        if (isValid(input.value) || input.disabled === true ) {

          input.parentElement.classList.remove("invalid");

        } else {

          input.parentElement.classList.add("invalid");

        }

    });

};

  

/*

  

let total_todos = 0;

let skip = 0

  

let fetchTodos = () => {

  fetch(`https://dummyjson.com/todos?limit=20&skip=${skip}`).then(items => items.json()).then(items => {

    let container = document.querySelector('.container');

    items = items.todos;

  

      for (let item of items) {

        let id = item['id'];

        let todo = item['todo'];

        let elem = document.createElement('div');

        elem.innerText = `${id}: ${todo}`;

        container.appendChild(elem);

      }

      skip += 20;

      // update count

      let count = document.querySelector('.total');

      count.innerText = `${skip}`;

      // disable button if skip == 100

      if (skip === 100) {

        let button = document.querySelector('button');

        button.style.visibility = 'hidden';        

      }

  }

  )

}

  
  
  

// add click handler for load more

fetchTodos();

let button = document.querySelector('button');

button.addEventListener('click', fetchTodos);

  

*/
```



### Implicit coercion
In JS, when you attempt to compare object with a primitive, JS will try to convert the object to a primitive value.

If an object is used in numerical operation, JS will convert object to a primitive and then to a number if possible.

For unary operators, eg. +value or -value, always coerced to a number.

For subtraction, eg. `true -  '2'`, it will coerce value to a number

For `null` and `undefined`, when converted to string they remain as `"null"` and `"undefined"`.

null gets coerced to 0 when to number.
undefined gets coerced to NaN when to number.


`[]` coerced into empty string ""
`{}` coerced into `[object Object]`

```js
// any number plus a string is a string
let m = 123;
m + 1; // "1231"
1 + m; // "1123"
// for minus symbol, it'll be coerced into a number



'5' > 3 // true, '5' coerced into 5
true > 0 // true, true coerced to 1
5 == [5] // true, because [5].toString() gives "5"
"5,12" == [5,12] // true, because [5,12] converts to primitive, which is stirng, "5,12"
null > 0 // false, null is coerced to 0 in numerical context


"5,12" - true // non-number string - 0 will be NaN

undefined > 0 // undefined gets converted NaN, what is less than any numbers
undefined + 12 // NaN

[1] > true // false, array [1] gets converted to a primitive string '1', but since '>' requires numbers, it gets converted to a numeric 1, 1 > true, true is also 1, so 1 > 1 means false


new String('str') == 'str'; // true, object is converted to a primitive value 'str'

new Number('6') == 6; // true
new Number('6') == '6'; // true

[5] == [5] // false, not same instance
{ a: 'a' } == { a: 'a' } // also false, same as above


// also fine to have things like
true ++ 12 // Uncaught syntax error
true + +12 // valid, 13
12 - +true // valid, 11 as true gets coerced to 1

// [] is empty array, which is an object,it will be coerced into empty string "", string + number is string
[] + 12 // "12"
[] - 12 // since it's subtraction, [] coerced into empty string, emp string coerces into 0, 0 - 12 = -12
[] == "0" // false, [] is coerced to "", which isn't "0" 
[] == 0   // true,  [] is coerced to "", then coerced to "0" since comparing to a number

"12" + 12 // "1212"
"number" + 15 + 3 // "number153"

true - '2' // -1, true coerced to 1, 1 - '2' is -1

v = {} // "[object Object]"
let vs = {a: 1} // "[object Object]"
vs + 123 // "[object Object]123"

[1, {}, 3].toString() // '1,[object Object],3'


// if at start, empty, if after, treated as string '[object Object]'
{} + 'hello' // NaN, since {} is at start, it's seen as an empty code block, which is nothing, so essentially + 'hello', which is NaN


+ 'hello' // NaN
+'hello' // NaN
++'hello' // NaN



// boolean, 0 coerce into false
'0' == false;
'' == false;

```



### What is CORS
Ensure that the API you are fetching data from allows requests from your domain. Otherwise, the browser may block the request due to CORS restrictions.

E.g I'm on page `https://youtube.com`, and I'm making a fetch request to `https://meta.com`, a different origin.

For a cross origin request, browser will check if `meta.com` allows requests from `youtube.com` . It does this by adding `Origin` header in its request ( browser does this automatically ) . The meta server checks `Origin` header and see if request is allowed., if server agrees to accept, it adds a special header `Access-Control-Allow-Origin` to the response. Browser checks this header, if missing or header mismatch the origin, it blocks access to the response, and we see CORS error.

![[Pasted image 20250320004226.png]]

An example of a permissive server response
```http
200 OK
Content-Type:text/html; charset=UTF-8
Access-Control-Allow-Origin: https://javascript.info
```

##### Safe requests and unsafe requests and preflight request
There are two types of cross origin requests, safe and unsafe ones.

Safe
* GET, POST, HEAD

Unsafe
* PATCH, PUT, DELETE

For unsafe requests, browser does not make such requests right away, it sends a `preflight` request to ask for permission. If server agrees to serve the requests, it respond with an empty body and status 200 and headers. Then browser automatically makes the actual request if preflight is successful.

Note server will still add `Access-Control-Allow-Origin` header to main response. Successful preflight does not relieve it.


### Prototype chain

```js
// prototype chain

// when you call this fn with 'new' keyword, JS does the following
// 1. creates an empty object
// 2. set 'this' to point to it
// 3. assign 'make' to 'this', the object
// 4. returns the new object
function Vehicle(make) {
	this.make = make;
}

// 'this' will refer to the object that calls this method
// When we add method to a prototype, all Vehicle instances will inherit it.
// What happens? A vehicle instance tries to find 'describe()' but doesn't find it. JS follows the prototype chain and finds it on vehicle.prototype.
Vehicle.prototype.describe = function() {
	return `This is a vehicle made by ${this.make}`;
}

function Car(make, model) {
	// calls Vehicle fn, with this sets to the current object, 
	// and assigns 'make' to 'this'
	// then assigns 'model' to 'this'
	Vehicle.call(this, make);
	this.model = model;
}

// creates a new object that inherits Vehicle.prototype,
// meaning this object inherits all methods from Vehicle.prototype
// Car.prototype = Vehicle.prototype; // ❌ They become same object!
// Meaning any changes to Car.prototype will affect Vehicle.prototype, eg. when we add a new method to Car.prototype
// However this removes original Car.prototype, which includes the constructor, meaning Car.prototype.constructor will point to function Vehicle() and not function Car()
Car.prototype = Object.create(Vehicle.prototype);

// Add back the correct constructor, as Vehicle.prototype does not have Car constructor
Car.prototype.constructor = Car;

Car.prototype.describe = function() {
	// Calls Vehicle's describe with current 'this'
	return `${Vehicle.prototype.describe.call(this)}`
}

const myCar = new Car('Toyota', 'Corolla');
console.log(myCar.describe()); // This is a vehicle made by Toyota
```


### Test html
```html



<form id='my-form'>
  <input type='checkbox' id='is-employed'>
  <label id='is-employed-label'>Is Employed</label>
  
  <br>
  <br>
  <div>
    <label for='email'>Email</label>
    <input type='text' id='email'>
  </div>
 
  <br>
  <div>
    <label for='phone-number'>Phone Number</label>
    <input type='text' id='phone-number'>
  </div>

  <br>
  <div>
    <label for='name'>Company Name</label>
    <input type='text' id='name'>
  </div>
  
  
  <div style='display: none'>
  <input type='radio' id='income-source' />
  <label>My income source</label>
  </div>
  
  <div style='display: none'>
  <input type='checkbox' id='income-source-alt' />
  <label>I have passive income</label>
  </div>

  
  <br>
  <button type='submit'>
   Submit
  </button>
  
  
</form>


```

js
```js
let throttle = false;
let email = document.querySelector('#email');
// throttle below
email.addEventListener('input', ev => {
	if (!throttle) {
  	throttle = true;
    setTimeout(() => {
  	console.log(ev.target.value)
  	throttle = false;
  }, 1000);
  }
});

let checkbox = document.querySelector('#is-employed');
checkbox.addEventListener('change', () => {
  let company_name_input = document.querySelector('#name');
	if (!checkbox.checked) {
    company_name_input.value = '';
  	company_name_input.disabled = true;
    
     // hide my income sources
    let income_source = document.querySelector('#income-source').parentElement;
    income_source.style.display = 'none';
    // show alt
    let income_source_alt = document.querySelector('#income-source-alt').parentElement;
    income_source_alt.style.display = 'block';
  
  } else {
  	company_name_input.disabled = false;
    
    // show income sources
    let income_source = document.querySelector('#income-source').parentElement;
    income_source.style.display = 'block';
    // show alt
    let income_source_alt = document.querySelector('#income-source-alt').parentElement;
    income_source_alt.style.display = 'none';
  }
});





let form = document.querySelector('form');
form.addEventListener('submit', (e) => {
  let inputs = document.querySelectorAll('input:not([type="checkbox"])');
  inputs.forEach(input => {
  	// check if empty and not disabled, then highlight
    if (input.value === '' && input.disabled == false) {
    	// add highlight to parent
      input.parentElement.classList.add('highlight');
    } else {
    	input.parentElement.classList.remove('highlight');
    }
  })
  
 // checks input empty, if empty highlight the label
  e.preventDefault();
})

```




### css

specificity

box-width: margin/padding/border/width

div>span

p>span

media query - screen orientation

```css
/* if exceed 600px, and orientation is landscape */
@media (min-width: 600px) and (orientation: landscape) {
  body {
    flex-direction: row;
  }
}

@media (orientation: portrait) {
  body {
    flex-direction: column;
  }
}
```

### Open link in new tab or window


```python
"""

We are building a word processor and we would like to implement a "reflow" functionality that also applies full justification to the text.

  

Given an array containing lines of text and a new maximum width, re-flow the text to fit the new width. Each line should have the exact specified width. If any line is too short, insert '-' (as stand-ins for spaces) between words as equally as possible until it fits.

  

Note: we are using '-' instead of spaces between words to make testing and visual verification of the results easier.

  
  
  
  

# lines = [ "The day began as still as the night abruptly lighted with brilliant flame" ]

  
  

# can treat as one single line

  

reflowAndJustify(lines, 25) "reflow lines and justify to length 25" =>

  

        [ "The-day-began-as-still-as"

          "the-----night----abruptly"

          "lighted---with--brilliant"

          "flame" ]

  

reflowAndJustify(lines, 26) "reflow lines and justify to length 26" =>

  

        [ "The--day-began-as-still-as",

          "the-night-abruptly-lighted",

          "with----brilliant----flame" ]

  

reflowAndJustify(lines, 40) "reflow lines and justify to length 40" =>

  

        [ "The--day--began--as--still--as-the-night",

          "abruptly--lighted--with--brilliant-flame" ]

  

reflowAndJustify(lines, 14) "reflow lines and justify to length 14" =>

  

        ['The--day-began',

         'as---still--as',

         'the------night',

         'abruptly',

         'lighted---with',

         'brilliant',

         'flame']

  

reflowAndJustify(lines, 15) "reflow lines and justify to length 15" =>

  

        ['The--day--began',

         'as-still-as-the',

         'night--abruptly',

         'lighted----with',

         'brilliant-flame']

  

lines2 = [ "a b", "c d" ]         

  

reflowAndJustify(lines2, 20) "reflow lines2 and justify to length 20" =>

  

        ['a------b-----c-----d']

        [a b, c d]

        The--day--began--as-still

  

reflowAndJustify(lines2, 4) "reflow lines2 and justify to length 4" =>

  

        ['a--b',

         'c--d']

  

reflowAndJustify(lines2, 2) "reflow lines2 and justify to length 2" =>

  

        ['a',

         'b',

         'c',

         'd']

  

All Test Cases:

                 lines, reflow width

reflowAndJustify(lines, 24)

reflowAndJustify(lines, 25)

reflowAndJustify(lines, 26)

reflowAndJustify(lines, 40)

reflowAndJustify(lines, 14)

reflowAndJustify(lines, 15)

reflowAndJustify(lines2, 20)

reflowAndJustify(lines2, 4)

reflowAndJustify(lines2, 2)

  

n = number of words OR total characters

"""

lines = ["The day began as still as the","night abruptly lighted with","brilliant flame"]

lines2 = ["a b","c d"]

  
  
  
  
  
  
  
  

# [The--day--began--as--still]  == new

# [as--the--night--abruptly]

  
  
  

# lines = [ "The day began as still as the",

#           "night abruptly lighted with",

#           "brilliant flame" ]

  

# reflowAndJustify(lines, 24) "reflow lines and justify to length 24" =>

  

#         [ "The--day--began-as-still",  

#           "as--the--night--abruptly",

#           "lighted--with--brilliant",

#           "flame" ] // <--- a single word on a line is not padded with spaces

  
  
  

'''

words: "The,day,began,as,still,as,the"   began

number:24

current_count: 7

current_words: [the, day, ] 

  

new_word_length:

  
  

'''

def reflowAndJustify(lines, number):

    current_words = []

    current_count = 0

    output = []

    for line in lines:

        # append word into current_words   

        words = line.split(' ')

        for word in words:

            if current_count < number:

                # I can add word into current_words

  

                new_word_length = current_count + len(word)

                if current_count != 0:

                    new_word_length += 1  #'The-day-began-'

                # we can add

                if new_word_length < number:

                    current_words.append(word)

                    current_count = new_word_length

                else:

                    # condition for single word

                    if len(current_words) == 1:

                        output.append(current_words[0])

                        # clear output

                        current_count = 0

                        current_words = []

                        continue

                    # not enough to fit a new word

                    idx = 0

                    while current_count <= number:

                        w = current_words[idx]   #[the-, day-]

                        w += '-'

                        current_count += 1

                        idx = (idx + 1) % len(current_words)

                    # current_count == number

                    output_line = '-'.join(current_words)

                    output.append(output_line)

                    # reset

                    current_count = 0

                    current_words = []

    return output     

  
  

print(reflowAndJustify(lines, 24))

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

# n is total number of words

  

def wrapLines(words, max_count):

    current_line = ''

    ans = []

    for idx in range(0, len(words)):

        # add

        if current_line == '':

            current_line += words[idx]

            continue

        new_length = 1 + len(words[idx]) + len(current_line)

        if new_length <= max_count:

            # put into current line

            current_line += ('-' + words[idx])

        else:

            # put current_line into ans

            ans.append(current_line)

            # reset

            current_line = ''

            current_line += words[idx]

    # if current_line != ''

    if current_line != '':

        ans.append(current_line)

    return ans
```
  