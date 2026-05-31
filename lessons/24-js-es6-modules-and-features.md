# Class 24: ES6+ Features & Modules

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 23 — events

Modern JavaScript (everything from ES6 / 2015 onwards) added a handful of small, beautiful features that change how code reads. Today we cover the ones you will see on every page of React code: template literals, optional chaining, nullish coalescing, short-circuit evaluation, and the module system that lets us split code across files.

## What You Will Learn
- Template literals with expressions and multi-line strings
- Optional chaining `?.`
- Nullish coalescing `??` and how it differs from `||`
- Short-circuit evaluation (`&&` and `||` returning values)
- `import` / `export` (named and default) — and `<script type="module">`

---

## 24.1 Template literals

You already met string concatenation with `+`. Template literals use backticks (`` ` ``) and the `${ }` syntax to **embed values** directly inside the string.

### Single-line

```js
const name = "Aarya";
const marks = 78;

// Old way
const message1 = "Hi " + name + ", you scored " + marks + "/100.";

// Template literal
const message2 = `Hi ${name}, you scored ${marks}/100.`;

console.log(message2); // "Hi Aarya, you scored 78/100."
```

So much easier to read — and that's before we get to the rest.

### Any expression is allowed inside `${ }`

It is not just variables. Any JavaScript expression works.

```js
const a = 5, b = 3;

console.log(`Sum: ${a + b}`);                        // "Sum: 8"
console.log(`Bigger: ${a > b ? "a" : "b"}`);         // "Bigger: a"
console.log(`Today: ${new Date().toDateString()}`);  // "Today: Sun May 31 2026"
```

### Multi-line strings (no escapes needed)

```js
const html = `
    <article class="post">
        <h2>${name}'s notes</h2>
        <p>Marks: ${marks}/100</p>
    </article>
`;

console.log(html);
```

Real line breaks become real line breaks. No more `"line one\n" + "line two\n"`.

### Recap

Template literals replace string concatenation **almost completely**. From today onwards, prefer them.

---

## 24.2 Optional chaining `?.`

Reading a nested property on something that might be `null` or `undefined` used to crash:

```js
const user = { name: "Aarya" }; // no address property

console.log(user.address.city); // TypeError: Cannot read properties of undefined
```

The old fix was a guard chain:

```js
const city = user && user.address && user.address.city;
console.log(city); // undefined
```

Optional chaining `?.` does this for you. If anything to its left is `null` or `undefined`, the whole expression short-circuits to `undefined` instead of crashing.

```js
const user = { name: "Aarya" };

console.log(user?.address?.city); // undefined  (no error!)
console.log(user?.name);          // "Aarya"
```

### Optional chaining on methods and arrays

```js
const obj = {};

// Call a method only if it exists
obj.greet?.();           // undefined  (no error)

const list = null;
console.log(list?.[0]);  // undefined  (no error)
```

Use `?.` whenever a value **might** be missing — typically when it comes from an API, props, or user input.

---

## 24.3 Nullish coalescing `??`

`??` returns the **right-hand value** if the left-hand value is `null` or `undefined` — otherwise it returns the left.

```js
const a = null;
const b = a ?? "default";
console.log(b); // "default"

const c = "Aarya";
const d = c ?? "default";
console.log(d); // "Aarya"
```

### `??` vs `||` — the subtle but important difference

`||` returns the right-hand side if the left is **falsy** (which includes `0`, `""`, `false`).
`??` only kicks in for `null` or `undefined`.

```js
const score = 0;

console.log(score || 10); // 10   ← whoops! 0 is falsy, so the fallback wins
console.log(score ?? 10); // 0    ← what we usually want
```

This is huge for things like prices, counts, and "user typed 0" cases.

### Combining with `?.`

A classic pattern:

```js
const user = { settings: null };

const theme = user?.settings?.theme ?? "light";
console.log(theme); // "light"
```

"Read `theme` if it exists at all the levels; otherwise default to `'light'`." This one line replaces about eight lines of older code.

---

## 24.4 Short-circuit evaluation

`&&` and `||` are not strictly true/false — they return one of their **operands**, and they stop as soon as the answer is known.

### `&&` returns the first falsy thing (or the last value)

```js
console.log(true && "hi");   // "hi"
console.log("a" && "b");     // "b"
console.log(0 && "b");       // 0   (0 is falsy, stop early)
```

Why care? Conditional execution:

```js
const isAdmin = true;
isAdmin && console.log("Welcome, admin");
// runs the console.log only if isAdmin is truthy
```

This pattern shows up in React all the time:

```jsx
{isLoggedIn && <Dashboard />}
```

### `||` returns the first truthy thing (or the last value)

```js
console.log(false || "fallback");   // "fallback"
console.log("" || 0 || "first truthy"); // "first truthy"
console.log("a" || "b");            // "a"
```

Old-school "default value":

```js
const username = userInput || "guest";
```

(Use `??` instead if `0` or `""` are valid values.)

---

## 24.5 Splitting code across files: ES modules

Up to now, every project has had one `app.js`. Real projects split code into many files, each doing one thing. JavaScript's official answer is **ES modules**: every file gets to choose what it shares with others.

### `<script type="module">`

Module files must be loaded with the `type="module"` attribute:

```html
<script type="module" src="app.js"></script>
```

Modules are automatically deferred and have stricter scope (no global leaks). They also must be served over `http://` — **opening the file with double-click will not work**. Use Live Server.

### Project structure

We will split a tiny calculator into three files:

```
project/
├── index.html
├── math.js
├── format.js
└── app.js
```

### Named exports

`math.js`:

```js
// Each export is a named thing
export const PI = 3.14159;

export function add(a, b) {
    return a + b;
}

export function multiply(a, b) {
    return a * b;
}
```

`format.js`:

```js
export const formatGBP = amount => `£${amount.toFixed(2)}`;
```

### Default exports

A file can have **one** default export. Use this when a file's main purpose is one thing.

`format.js` (alternative — default + named):

```js
const formatGBP = amount => `£${amount.toFixed(2)}`;
export default formatGBP;

// you can still have named exports alongside
export const formatNPR = amount => `Rs ${amount.toFixed(2)}`;
```

### Importing

`app.js`:

```js
// Named imports — names and braces must match the export
import { add, multiply, PI } from "./math.js";

// Default import — choose any name, no braces
import formatGBP from "./format.js";

// Mixing default and named
import formatGBP, { formatNPR } from "./format.js";

// Rename on the way in
import { add as plus } from "./math.js";

console.log(add(2, 3));         // 5
console.log(multiply(4, 5));    // 20
console.log(PI);                // 3.14159
console.log(formatGBP(19.5));   // "£19.50"
console.log(formatNPR(199));    // "Rs 199.00"
console.log(plus(1, 1));        // 2
```

### `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Modules demo</title>
    <script type="module" src="app.js"></script>
</head>
<body>
    <h1>Open the console</h1>
</body>
</html>
```

Open this with Live Server (not double-click!), check the console, you should see the values logged.

### Named vs default — when to use which?

| Use case | Style |
|---|---|
| Single, headline thing in the file | **default** export |
| A bunch of related helpers | **named** exports |
| Both? | one default + several named (common) |

In the React world you will see both patterns. Components are usually default-exported; hooks and utilities are usually named-exported.

---

## 24.6 The module graph (a mental model)

When `app.js` imports from `math.js`, the browser:
1. Loads `app.js`.
2. Sees it needs `./math.js`, downloads that too.
3. Runs `math.js` (so its `export`s exist).
4. Then runs `app.js` (now its `import`s resolve).

This forms a **graph** — your app is a tree of files importing from each other. The browser does all the wiring for you.

Key rules:
- Each module is **run once**, regardless of how many files import it.
- A module's top-level code is **its initialisation** — keep it side-effect free where possible.
- Circular imports (A imports B, B imports A) work in many cases but can cause subtle bugs — avoid them.

---

## 24.7 Putting it together: refactoring our to-do app into modules

Recall the to-do app from Class 23 — one big `app.js`. We can now split it like this:

`storage.js`
```js
let tasks = [];

export function getTasks() {
    return tasks;
}

export function addTask(text) {
    const id = Date.now();
    tasks = [...tasks, { id, text, done: false }];
    return id;
}

export function toggleTask(id) {
    tasks = tasks.map(t =>
        t.id === id ? { ...t, done: !t.done } : t
    );
}

export function removeTask(id) {
    tasks = tasks.filter(t => t.id !== id);
}
```

`dom.js`
```js
export function renderList(listEl, tasks) {
    listEl.innerHTML = "";
    for (const t of tasks) {
        const li = document.createElement("li");
        li.dataset.id = t.id;
        li.className = t.done ? "done" : "";
        li.innerHTML = `
            <span>${t.text}</span>
            <button data-action="toggle">✓</button>
            <button data-action="delete">×</button>
        `;
        listEl.appendChild(li);
    }
}
```

`app.js`
```js
import { getTasks, addTask, toggleTask, removeTask } from "./storage.js";
import { renderList } from "./dom.js";

const form = document.querySelector("#task-form");
const input = document.querySelector("#task-input");
const list = document.querySelector("#task-list");

const refresh = () => renderList(list, getTasks());

form.addEventListener("submit", (e) => {
    e.preventDefault();
    const text = input.value.trim();
    if (!text) return;
    addTask(text);
    input.value = "";
    refresh();
});

list.addEventListener("click", (e) => {
    const button = e.target;
    if (button.tagName !== "BUTTON") return;
    const id = Number(button.closest("li").dataset.id);
    if (button.dataset.action === "toggle") toggleTask(id);
    if (button.dataset.action === "delete") removeTask(id);
    refresh();
});

refresh();
```

Three files, each with a clear job:
- `storage.js` — the data and how to change it.
- `dom.js` — how to draw the data on screen.
- `app.js` — wires events to actions.

This separation is the same idea React uses, just enforced more strongly.

---

## Practice Exercises

1. **Template literal makeover**: Take any code from earlier classes that used string concatenation (the grade calculator, the to-do list) and rewrite the strings as template literals.

2. **Safe access**: Given this object, write expressions that read `user.address.city` safely (no crash) and fall back to `"unknown"` when missing.
   ```js
   const user = { name: "Aarya" }; // no address
   ```
   Then change `user` so it has `address: { city: "Pokhara" }` and rerun. Same code, both work.

3. **`||` vs `??` quiz**: For each line, predict the result then check.
   ```js
   const a = 0   || "fallback";
   const b = 0   ?? "fallback";
   const c = ""  || "fallback";
   const d = ""  ?? "fallback";
   const e = null|| "fallback";
   const f = null?? "fallback";
   ```

4. **Refactor into modules**: Take any single-file project from earlier classes and split it into at least two modules: one for data/utility functions, one for the page wiring. Load with `<script type="module">` and verify it works in Live Server.

---

## Key Takeaways
- Template literals (`` `Hi ${name}` ``) replace concatenation almost entirely.
- `?.` lets you safely access deep properties without crashing.
- `??` defaults only on `null`/`undefined` — unlike `||`, it does not fall through for `0` or `""`.
- `&&` and `||` actually return values, enabling concise patterns like `isAdmin && doThing()`.
- ES modules use `import` / `export`, served with `<script type="module">`. Default vs named exports are both common.

Next class: async JavaScript — promises, `async`/`await`, and fetching real data from the internet.
