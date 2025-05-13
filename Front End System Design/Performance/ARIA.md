Set of attributes you can add to html elements to to make web content and web app more accessible to people with disabilities.

It supplements HTML so interactions and widgets commonly used in app can be passed to assistive tech. So only assist tech users will notice the differences.

Note simply adding ARIA to elements do not make them more accessible, proper implementation is key.

Sometimes we should not be using ARIA, and instead should use proper semantic HTML.
```html
// Bad
<a role="button">Submit</a>

// Good
<button>Submit</button>
```

Also don't need to add unnecessary ARIA to html

```html
// Bad
<h2 role="tab">Heading tab</h2>

// Good
<div role="tab"><h2>Heading tab</h2></div>


// Another bad
<ul role="list">...</ul>

// Good
<ul>...</ul>
```

Always support keyboard navigation
All non-disabled interactive ARIA controls must be keyboard accessible.



`role` attribute describes the role of an element for assist programs, like screen readers or magnifiers.

```html
// Screen reader will read this as a button than a link
<a href="#" role="button">Button Link</a>
```

`properties` describe characteristics or relationships to an object.

```html
// 'aria-describedby' is used to reference another element that contains descriptive info about current element
<div aria-describedby="more-info">Self-destruct</div>

<div id="more-info">This page will self-destruct in 10 seconds.</div>
```

`states/values` define current conditions or data associated with the element.
```html
<div role="button" aria-pressed="false"> Self-destruct</div>
```


---

## 1. **ARIA Roles**

Roles define **what an element is**.

| Role              | Purpose                                                           |
| ----------------- | ----------------------------------------------------------------- |
| `button`          | Marks an element as a button (even if it's a `<div>` or `<span>`) |
| `dialog`          | Identifies a modal or popup dialog                                |
| `navigation`      | Indicates a navigation landmark                                   |
| `tab`, `tabpanel` | Used for custom tab interfaces                                    |
| `alert`           | Marks a message that should be announced immediately              |
| `progressbar`     | Indicates a progress indicator                                    |

## 2. **ARIA Attributes**

Attributes give **extra information** about roles or elements. They usually start with `aria-`.

| Attribute                                       | Description                                                                   |
| ----------------------------------------------- | ----------------------------------------------------------------------------- |
| `aria-label`                                    | Custom label for an element                                                   |
| `aria-hidden`                                   | Hides element from screen readers                                             |
| `aria-expanded`                                 | Indicates if an element is expanded (e.g. dropdown)                           |
| `aria-controls`                                 | References the element controlled by another (like a tab controlling a panel) |
| `aria-checked`, `aria-selected`, `aria-pressed` | Reflect interactive states                                                    |
| `aria-describedby`                              | Points to element that describes this one                                     |
| `aria-live`                                     | Defines how updates should be announced (e.g. for alerts)                     |

Whenever possible, **use native HTML elements** like `<button>`, `<input>`, `<nav>`, etc. because they:

- Already have built-in accessibility
    
- Don’t require ARIA
    

> Use ARIA **only when you build custom components** (e.g. custom dropdown, tab interface, modals, etc.)
