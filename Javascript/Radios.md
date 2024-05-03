```html
<div class="input-container">
        <label>I Have Passive Income</label>
        <input type="radio" name="passiveIncome">
</div>

<div class="input-container">
        <label>I Don't Have Passive Income</label>
        <input type="radio" name="passiveIncome">
</div>


// update radio button
      

```


```
/*

We have a form where users submit their personal information. Recently we got complaints that it doesn't work as expected.

  

We identified the following issues that need fixing:

  

- All invalid inputs get highlighted when clicking on the Submit button. This is correct, but the disabled inputs should not get highlighted: Email, "Phone Number" and "Company Name". Lastly, if the "Is Employed" checkbox is checked, "Company Name" should be enabled and highlighted if it's empty upon submission.

  
  
  

- When the "Is Employed" checkbox is checked, the "Company Name" input gets enabled, and the user can type a value there. When we uncheck the "Is Employed" checkbox, the "Company Name" input should reset its value and be empty. Currently, it does not.

  
  
  

- When the "Is Employed" checkbox is checked, the "My Income Sources" radio buttons should change to the "I Have Passive Income" checkbox. It should be checked if the "I Have Passive Income" radio button was selected. When the "Is Employed" checkbox is unchecked, the radio buttons should be shown again, and the appropriate radio button should be selected based on the checkbox state.

*/

  

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