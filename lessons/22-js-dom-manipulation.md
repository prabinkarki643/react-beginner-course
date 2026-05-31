# Class 22: DOM Manipulation

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 21 — objects, destructuring, spread

So far, all our JavaScript has printed to the console. Today the real fun begins: we will reach into the HTML page itself and change it from JavaScript. This is what JS was invented for.

## What You Will Learn
- What the DOM is (the page as a tree)
- Selecting elements with `getElementById`, `querySelector` and `querySelectorAll`
- Reading and changing `textContent`, `innerHTML` and attributes
- Creating, appending and removing nodes
- Working with `classList`: `add`, `remove`, `toggle`, `contains`

---

## 22.1 What is the DOM?

When the browser loads an HTML file, it does not keep the file as raw text. It builds a **tree** in memory called the **DOM** — the **Document Object Model**.

Every tag becomes a **node** in the tree. JavaScript can then move around the tree, inspect nodes, change them, add new ones, or remove them.

Given this HTML:

```html
<body>
    <h1>My Page</h1>
    <ul>
        <li>One</li>
        <li>Two</li>
    </ul>
</body>
```

The DOM tree looks like:

```
document
└── html
    └── body
        ├── h1  ("My Page")
        └── ul
            ├── li ("One")
            └── li ("Two")
```

JavaScript talks to this tree through the global `document` object.

> Open any web page, press **F12**, click the **Elements** tab — that is the DOM, live. Click around — those are the nodes JS can reach.

---

## 22.2 The HTML we will work with

Throughout this lesson, create this file once and reuse it. Save it as `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>DOM Practice</title>
    <style>
        body { font-family: sans-serif; padding: 2rem; }
        .highlight { background: yellow; padding: 0.25rem; }
        .done { text-decoration: line-through; opacity: 0.5; }
        li { margin: 0.25rem 0; }
    </style>
    <script src="app.js" defer></script>
</head>
<body>
    <h1 id="title">My To-Do List</h1>
    <p class="subtitle">Things I plan to do today.</p>

    <ul id="todo-list">
        <li class="task">Buy milk</li>
        <li class="task done">Finish CSS homework</li>
        <li class="task">Read one chapter</li>
    </ul>

    <button id="add-btn" data-action="add">Add task</button>
</body>
</html>
```

Then create `app.js` in the same folder. Open `index.html` with Live Server and press F12 to see the console.

---

## 22.3 Selecting elements

### `getElementById`

The oldest selector — returns the element with that `id`, or `null` if it does not exist.

```js
const title = document.getElementById("title");
console.log(title); // <h1 id="title">My To-Do List</h1>
```

### `querySelector`

Takes a **CSS selector** and returns the **first** matching element (or `null`).

```js
const title = document.querySelector("#title");      // by id
const subtitle = document.querySelector(".subtitle"); // by class
const firstTask = document.querySelector("li.task");  // first <li class="task">

console.log(title);
console.log(subtitle);
console.log(firstTask);
```

Because it accepts any CSS selector, `querySelector` is the most flexible — and what we will use most.

### `querySelectorAll`

Returns **all** matching elements as a `NodeList` (array-like).

```js
const tasks = document.querySelectorAll(".task");
console.log(tasks.length); // 3

tasks.forEach(task => {
    console.log(task.textContent);
});
```

**Expected output:**
```
3
Buy milk
Finish CSS homework
Read one chapter
```

> A `NodeList` is array-like — you can use `forEach` and `for…of` on it. To use `.map` or `.filter`, convert it with `Array.from(nodes)` or `[...nodes]`.

### Quick reference

| Method | Returns | Use when |
|---|---|---|
| `getElementById("x")` | one element or `null` | you know the exact id |
| `querySelector("...")` | first match or `null` | any CSS selector, one match |
| `querySelectorAll("...")` | a NodeList of matches | you want every match |

---

## 22.4 Reading and changing content

### `textContent` — plain text only (safe)

```js
const title = document.querySelector("#title");

console.log(title.textContent); // "My To-Do List"

title.textContent = "Today's Plan";
```

After this, the page shows "Today's Plan" as the heading.

### `innerHTML` — text **and** markup (powerful, but careful)

```js
const subtitle = document.querySelector(".subtitle");

subtitle.innerHTML = "Things I <em>plan</em> to do <strong>today</strong>.";
```

The browser parses the string as HTML, so the `<em>` and `<strong>` tags become real elements.

> **Security warning**: never feed user-typed text directly into `innerHTML`. A malicious user could inject `<script>` tags. For plain text, prefer `textContent`. We will come back to this in the React phase — React fixes this by default.

---

## 22.5 Reading and changing attributes

### `getAttribute` and `setAttribute`

```js
const button = document.querySelector("#add-btn");

console.log(button.getAttribute("data-action")); // "add"

button.setAttribute("data-action", "remove");
button.setAttribute("disabled", "true");
```

After this, inspect the button in DevTools — it has the new attributes.

### Direct property access

Many common attributes are also available as plain properties:

```js
const button = document.querySelector("#add-btn");

console.log(button.id);        // "add-btn"
console.log(button.disabled);  // false

button.disabled = true;        // disables it
button.textContent = "Adding…"; // change the label
```

> Note: the HTML attribute is `class`, but the JS property is `className` (because `class` is a reserved word in JS). We rarely need `className` directly though — we have `classList`, see 22.7.

---

## 22.6 Creating, appending and removing nodes

### Creating an element

```js
const li = document.createElement("li");
li.textContent = "Go for a walk";
li.classList.add("task");
```

At this point `li` exists in memory but is **not yet on the page**.

### Appending to a parent

```js
const list = document.querySelector("#todo-list");
list.appendChild(li);
```

Now the new `<li>` appears at the end of the list.

### A complete add-a-task block

```js
const list = document.querySelector("#todo-list");

const li = document.createElement("li");
li.textContent = "Go for a walk";
li.classList.add("task");
list.appendChild(li);

console.log("List now has " + list.children.length + " items.");
// "List now has 4 items."
```

### `append` (modern, more flexible)

`append` lets you add multiple things at once and accepts strings as well as nodes:

```js
list.append(li, "Some text directly", document.createElement("hr"));
```

### Removing a node

The easiest way is `.remove()` — the node tells itself to leave the tree.

```js
const firstTask = document.querySelector("li.task");
firstTask.remove();
```

That's it — the first `<li>` is gone from the page.

### Inserting at a specific position

```js
const newFirst = document.createElement("li");
newFirst.textContent = "Drink water";
newFirst.classList.add("task");

const list = document.querySelector("#todo-list");
list.prepend(newFirst); // adds at the top
```

There is also `insertBefore(newNode, referenceNode)`, but `prepend`/`append` cover most cases.

---

## 22.7 `classList` — managing CSS classes

`classList` is the modern, easy way to deal with classes.

```js
const firstTask = document.querySelector("li.task");

// Add a class
firstTask.classList.add("highlight");

// Remove a class
firstTask.classList.remove("highlight");

// Toggle a class (adds if missing, removes if present)
firstTask.classList.toggle("done");
firstTask.classList.toggle("done"); // back to original

// Check whether a class is present
console.log(firstTask.classList.contains("task")); // true
console.log(firstTask.classList.contains("done")); // false
```

You can pass multiple classes:

```js
firstTask.classList.add("highlight", "important");
firstTask.classList.remove("highlight", "important");
```

### Why `classList` instead of `className`

You could write `el.className = "task done"`, but that wipes any existing classes. `classList` lets you add or remove one class at a time without affecting the others. Use `classList` 99% of the time.

---

## 22.8 A complete worked example

Drop this into `app.js` against the HTML from 22.2:

```js
// 1. Update the heading
const title = document.getElementById("title");
title.textContent = "Today's Plan";

// 2. Highlight the subtitle
const subtitle = document.querySelector(".subtitle");
subtitle.classList.add("highlight");

// 3. Add two new tasks programmatically
const list = document.querySelector("#todo-list");
const newTasks = ["Drink water", "Stretch for 5 minutes"];

for (const text of newTasks) {
    const li = document.createElement("li");
    li.textContent = text;
    li.classList.add("task");
    list.appendChild(li);
}

// 4. Mark every "Buy milk" task as done
const all = document.querySelectorAll(".task");
all.forEach(task => {
    if (task.textContent === "Buy milk") {
        task.classList.add("done");
    }
});

// 5. Disable the button while we "save"
const button = document.querySelector("#add-btn");
button.disabled = true;
button.textContent = "Saving…";

// 6. Log a summary
console.log("Total tasks on screen: " + list.children.length);
```

**What you should see on the page:**
- The heading reads "Today's Plan"
- The subtitle has a yellow highlight
- The list has two new items
- "Buy milk" has strikethrough styling
- The "Add task" button is disabled and says "Saving…"

---

## 22.9 Looking ahead

Everything we did today happened **once**, when the page loaded. That is because there are no user actions yet — no clicks, no typing.

In Class 23, we add events. Click a task → it gets crossed off. Type in a box → a new task appears. Click a delete button → it disappears. That is when the page truly comes to life.

For today, we are also not handling events — so our exercises will be "set up the UI in JS and inspect it in DevTools".

---

## Practice Exercises

Use the `index.html` from 22.2 for all of these.

1. **Rename and recolour**: In `app.js`:
   - Change the `<h1>` text to your own name.
   - Add a class `"highlight"` to the subtitle.
   - Add a class `"done"` to the last task.

2. **Build a list from data**: Replace the existing tasks with a list generated from this array. Loop through it, create one `<li class="task">` per item, mark `done` ones with the `done` class, and append all of them to `#todo-list`. Clear out the old children first (`list.innerHTML = ""`).

   ```js
   const tasks = [
       { text: "Buy milk", done: false },
       { text: "Finish CSS homework", done: true },
       { text: "Read one chapter", done: false },
       { text: "Go for a walk", done: false }
   ];
   ```

3. **Count tasks**: Add a `<p id="count"></p>` somewhere in the HTML, and in JS set its `textContent` to read like `"4 tasks, 1 done"` based on the array above.

4. **Toggle every other task**: Using `querySelectorAll` and a loop, give every **odd-indexed** task a `highlight` class (so the 2nd and 4th are highlighted).

---

## Key Takeaways
- The DOM is a tree of nodes built from your HTML. JS reaches it via `document`.
- `querySelector` (one) and `querySelectorAll` (all) take any CSS selector — your main tools.
- Use `textContent` for plain text (safe) and `innerHTML` only for trusted HTML strings.
- Build nodes with `createElement`, attach them with `append`/`prepend`/`appendChild`, remove them with `.remove()`.
- Use `classList.add` / `remove` / `toggle` / `contains` to manage CSS classes — never overwrite `className`.

Next class: events. We listen for clicks, keypresses and form submissions, and the page finally responds to the user.
