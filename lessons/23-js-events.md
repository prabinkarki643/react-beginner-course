# Class 23: Events & Event Handling

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 22 — DOM manipulation

Last class, we changed the page **once**. Today the page starts **listening**. When the user clicks, types, submits or hovers, our JavaScript runs in response. This is the heart of interactive web pages.

## What You Will Learn
- `addEventListener` and why it beats inline handlers
- The event object, `preventDefault()` and `stopPropagation()`
- Common events: `click`, `input`, `change`, `submit`, `keydown`
- The event delegation pattern (one listener handling many elements)

---

## 23.1 The HTML we will work with

Create these files and run with Live Server.

**index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Events Practice</title>
    <style>
        body { font-family: sans-serif; padding: 2rem; max-width: 480px; }
        button { padding: 0.4rem 0.8rem; cursor: pointer; }
        input { padding: 0.4rem; width: 100%; box-sizing: border-box; }
        .done { text-decoration: line-through; opacity: 0.5; }
        li { display: flex; gap: 0.5rem; align-items: center; margin: 0.25rem 0; }
        li button { margin-left: auto; }
    </style>
    <script src="app.js" defer></script>
</head>
<body>
    <h1 id="title">Events Demo</h1>

    <button id="say-hi">Say hi</button>
    <p id="hi-output"></p>

    <hr>

    <label for="name-input">Type your name:</label>
    <input id="name-input" type="text" placeholder="Aarya">
    <p id="name-output"></p>

    <hr>

    <form id="task-form">
        <input id="task-input" type="text" placeholder="New task..." required>
        <button type="submit">Add</button>
    </form>
    <ul id="task-list"></ul>
</body>
</html>
```

**app.js** — start empty; we will add code section by section.

---

## 23.2 `addEventListener` — the proper way

To make an element respond to a user action, we **register a listener**: a function that runs every time that event fires.

```js
const button = document.querySelector("#say-hi");

button.addEventListener("click", () => {
    console.log("Button was clicked!");
});
```

`addEventListener` takes two things:
1. The **event name** as a string (`"click"`, `"submit"`, `"keydown"`…).
2. The **handler function** to run.

Click the button — you should see `Button was clicked!` in the console each time.

### Why not inline handlers?

You may have seen the old style:

```html
<button onclick="alert('Hi')">Old way</button>
```

This works, but:
- It mixes JS into your HTML.
- Only one handler is allowed per event.
- It is harder to remove or change later.

`addEventListener` solves all three. We always use it.

### Removing a listener

If you ever need to detach a handler, save it in a variable first:

```js
const handleClick = () => console.log("clicked");
button.addEventListener("click", handleClick);
// later...
button.removeEventListener("click", handleClick);
```

Anonymous arrow functions cannot be removed — there is no name to reference. So if you might need to remove it, name it.

---

## 23.3 The event object

The handler function actually receives an argument — the **event object**. By convention, we call it `event` or `e`.

```js
const button = document.querySelector("#say-hi");

button.addEventListener("click", (event) => {
    console.log(event);
    console.log("Clicked at:", event.clientX, event.clientY);
    console.log("Target element:", event.target);
});
```

Click the button — the console shows a big `PointerEvent` object with lots of useful information: where the click happened, which element was clicked, which mouse button, and more.

Most useful properties:

| Property | What it is |
|---|---|
| `event.target` | the element the event actually happened on |
| `event.currentTarget` | the element the listener is attached to (often the same) |
| `event.type` | the event name (e.g. `"click"`) |
| `event.key` | for keyboard events, the key pressed (e.g. `"Enter"`) |
| `event.clientX` / `event.clientY` | mouse position |

---

## 23.4 `click` — the most common event

Let's update the hi output element:

```js
const button = document.querySelector("#say-hi");
const output = document.querySelector("#hi-output");

let clicks = 0;

button.addEventListener("click", () => {
    clicks = clicks + 1;
    output.textContent = "Clicked " + clicks + " time" + (clicks === 1 ? "" : "s");
});
```

Click the button repeatedly — the paragraph updates each time. Notice we are using a closure: the handler "remembers" the `clicks` variable across calls.

---

## 23.5 `input` — fires on every keystroke

`input` fires on each character typed into a text input. Useful for live previews and search-as-you-type.

```js
const nameInput = document.querySelector("#name-input");
const nameOutput = document.querySelector("#name-output");

nameInput.addEventListener("input", (event) => {
    const value = event.target.value;
    if (value === "") {
        nameOutput.textContent = "";
    } else {
        nameOutput.textContent = "Hello, " + value + "!";
    }
});
```

Type your name — the greeting updates live.

Notice `event.target.value` — that is how we read what the user has typed.

### `change` — fires only when the user "commits"

`change` is similar to `input`, but on text inputs it only fires when the input **loses focus** (you click elsewhere or press Tab). On checkboxes and `<select>` it fires whenever the value changes.

| Event | When it fires |
|---|---|
| `input` | every keystroke |
| `change` | when you leave the field (text) or change the choice (checkbox/select) |

For most form-style behaviour, prefer `input`.

---

## 23.6 `submit` and `preventDefault()`

Forms have a special default behaviour: when submitted, the browser **reloads the page**, sending the data to the URL in the `action` attribute. For a single-page app, we almost always want to stop that.

```js
const form = document.querySelector("#task-form");
const input = document.querySelector("#task-input");

form.addEventListener("submit", (event) => {
    event.preventDefault();         // 🛑 stop the page reloading
    console.log("Form submitted:", input.value);
});
```

Type something and press Enter (or click "Add"). The console logs the value, and **the page does not reload**.

Without `preventDefault()`, the URL would change to `?...` and the page would reload — losing all state. `preventDefault()` is essential for forms.

`preventDefault()` also works on:
- `<a>` clicks (stops navigation)
- Right-click menus (`contextmenu`)
- Drag-and-drop, etc.

---

## 23.7 A working mini to-do list

Now let's wire up the form to actually add tasks. Each task will have a "delete" button.

```js
const form = document.querySelector("#task-form");
const input = document.querySelector("#task-input");
const list = document.querySelector("#task-list");

form.addEventListener("submit", (event) => {
    event.preventDefault();

    const text = input.value.trim();
    if (text === "") return; // ignore empty submissions

    // Build the new <li>
    const li = document.createElement("li");
    li.textContent = text;

    // A delete button on the right
    const deleteBtn = document.createElement("button");
    deleteBtn.textContent = "×";
    deleteBtn.type = "button";
    deleteBtn.addEventListener("click", () => {
        li.remove();
    });

    li.appendChild(deleteBtn);
    list.appendChild(li);

    // Clear the input for the next task
    input.value = "";
    input.focus();
});
```

Try it out — type tasks, press Enter, and click × to remove them.

---

## 23.8 `keydown` — keyboard events

Keyboard events fire on the focused element. The most common is `keydown`.

```js
const input = document.querySelector("#name-input");

input.addEventListener("keydown", (event) => {
    console.log("Key:", event.key);

    if (event.key === "Enter") {
        console.log("Enter pressed!");
    }
    if (event.key === "Escape") {
        input.value = "";
    }
});
```

Useful keys: `"Enter"`, `"Escape"`, `"ArrowUp"`, `"ArrowDown"`, `"Tab"`, `"Backspace"`, plus the actual character keys like `"a"`, `"A"`.

You can also listen on `document` to catch keys anywhere on the page (useful for shortcuts like "press / to focus search").

---

## 23.9 Event bubbling and `stopPropagation()`

When you click on the delete `×` inside a `<li>`, that click also "bubbles up" — first to the `<li>`, then to the `<ul>`, then to the `<body>`, and so on. Every parent gets a chance to handle it.

```js
list.addEventListener("click", () => console.log("list clicked"));

// Now add a listener on the items inside
list.querySelectorAll("li").forEach(li => {
    li.addEventListener("click", () => console.log("li clicked"));
});
```

If you click on an `<li>`, you will see:
```
li clicked
list clicked
```

Sometimes you want to stop the bubble:

```js
deleteBtn.addEventListener("click", (event) => {
    event.stopPropagation(); // do not trigger the <li> click handler
    li.remove();
});
```

Use `stopPropagation()` only when you really need it — most of the time, bubbling is helpful.

> Quick distinction: **`preventDefault()`** stops the browser's default behaviour (form submit, link navigation). **`stopPropagation()`** stops the event from bubbling up to parents. Two different things.

---

## 23.10 Event delegation — the clever pattern

Bubbling is annoying when it gets in the way, but it is brilliant when you exploit it.

**The problem**: imagine 100 tasks. If we attach a click listener to every delete button, that is 100 listeners. Adding a new task means adding another listener. It works, but it does not scale.

**The trick**: attach **one** listener to the parent `<ul>`. When a click bubbles up, check what was actually clicked and react accordingly.

```js
const list = document.querySelector("#task-list");

list.addEventListener("click", (event) => {
    // event.target is the actual element that was clicked
    if (event.target.tagName === "BUTTON") {
        const li = event.target.closest("li");
        li.remove();
    }
});
```

Now we have **one** listener that handles every delete button — including buttons added in the future. This is **event delegation**, and it is the standard way to handle "lists of similar items".

### Using `data-*` attributes for more options

Often, items have multiple actions (delete, toggle done, edit). Use `data-action` attributes so the parent listener can branch:

```html
<li data-id="1">
    Buy milk
    <button data-action="toggle">✓</button>
    <button data-action="delete">×</button>
</li>
```

```js
list.addEventListener("click", (event) => {
    const button = event.target;
    if (button.tagName !== "BUTTON") return;

    const li = button.closest("li");
    const action = button.dataset.action; // reads data-action

    if (action === "toggle") {
        li.classList.toggle("done");
    } else if (action === "delete") {
        li.remove();
    }
});
```

One listener, two actions, scales to thousands of items. This is the pattern React-flavoured frameworks use under the hood.

---

## 23.11 A complete worked example

Here is the full app.js for an interactive to-do list using submit + delegation.

```js
const form = document.querySelector("#task-form");
const input = document.querySelector("#task-input");
const list = document.querySelector("#task-list");

// Add a task
form.addEventListener("submit", (event) => {
    event.preventDefault();

    const text = input.value.trim();
    if (text === "") return;

    const li = document.createElement("li");
    li.innerHTML = `
        <span class="text">${text}</span>
        <button type="button" data-action="toggle">✓</button>
        <button type="button" data-action="delete">×</button>
    `;
    list.appendChild(li);

    input.value = "";
    input.focus();
});

// One listener for all task actions
list.addEventListener("click", (event) => {
    const button = event.target;
    if (button.tagName !== "BUTTON") return;

    const li = button.closest("li");
    const action = button.dataset.action;

    if (action === "toggle") {
        li.classList.toggle("done");
    } else if (action === "delete") {
        li.remove();
    }
});
```

Add a few tasks, toggle them, delete them. You now have a real, working interactive page.

> Caveat about `innerHTML` and user input: in 23.11 we are injecting `text` directly into HTML. That is okay for our own typing, but is unsafe for **untrusted** input. The right fix is `textContent` on a span. We will revisit this when we move to React — React escapes content for us automatically.

---

## Practice Exercises

Use the `index.html` from 23.1.

1. **Click counter**: Make the "Say hi" button update its own label so it reads `"Clicked X times"` directly on the button, with no extra paragraph.

2. **Live character count**: Below the name input, show a live character count: `"7 characters"`. Update it on every `input` event. When the value is empty, show nothing.

3. **Interactive to-do**: Complete the to-do app from 23.11. Then add:
   - A counter that shows `"3 tasks, 1 done"`, updated on every add/toggle/delete.
   - Pressing Escape inside the input clears it.

4. **Delegation only**: Refactor the delete button in 23.7 (the version with a per-button listener) to use event delegation instead — one listener on the `<ul>`. Compare the two approaches in your own words.

---

## Key Takeaways
- Use `addEventListener` — never inline `onclick=""`.
- The handler receives an **event object** with `target`, `key`, coordinates and more.
- `event.preventDefault()` stops browser defaults (form reload, link navigation).
- `event.stopPropagation()` stops the event bubbling up to parents.
- **Event delegation** — one listener on the parent, branching on `event.target` — is the way to handle lists efficiently.

Next class: ES6+ syntax (template literals, optional chaining, modules) — the modern JavaScript that makes the React code you are about to meet readable.
