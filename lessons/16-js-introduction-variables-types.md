# Class 16: JavaScript Introduction, Variables & Data Types

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 15 — you can build a styled HTML/CSS page

Welcome to JavaScript! For the last fifteen classes you have been building **what the page looks like**. From today, you will start telling the page **how to behave**. This is the moment our static portfolio starts coming alive.

## What You Will Learn
- What JavaScript is and where it runs (browser and Node.js)
- How to attach JS to an HTML page using `<script defer>`
- How to use `console.log` and the browser DevTools console
- The difference between `let`, `const` (and why we avoid `var`)
- The seven primitive data types and how to inspect them with `typeof`

---

## 16.1 What is JavaScript?

Think of a web page as a person.

- **HTML** is the **skeleton** — the bones and structure.
- **CSS** is the **clothes and make-up** — how the person looks.
- **JavaScript** is the **brain and muscles** — how the person thinks and moves.

When you click a button and something happens, when a form checks your email before sending it, when a number on a page updates without reloading — that is JavaScript at work.

JavaScript was created in 1995 in just ten days by Brendan Eich at Netscape. Despite its rushed birth, it is now the most widely used programming language in the world.

### Where does JavaScript run?

JavaScript was originally built to run **inside the browser** (Chrome, Firefox, Safari, Edge). Every modern browser ships with a built-in JavaScript engine (Chrome uses V8, Firefox uses SpiderMonkey).

In 2009, a developer named Ryan Dahl took the V8 engine out of Chrome and turned it into a standalone program called **Node.js**. This meant JavaScript could now also run **on the server**, on your laptop, on a Raspberry Pi — anywhere you can install Node.

For this course:

- **Now (Classes 16–25)**: JavaScript in the browser.
- **Later (Class 26 onwards)**: JavaScript in Node, so we can build React projects.

---

## 16.2 Your first JavaScript: linking JS to HTML

JavaScript on its own does nothing — it needs to be loaded by a page. Create a new folder, then create two files side by side.

**index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>JS Day One</title>
    <script src="app.js" defer></script>
</head>
<body>
    <h1>Open your DevTools console</h1>
    <p>Press F12 (or Cmd + Option + I on Mac).</p>
</body>
</html>
```

**app.js**
```js
console.log("Hello from JavaScript!");
```

Open `index.html` with Live Server, press **F12**, click the **Console** tab. You should see:

```
Hello from JavaScript!
```

### Why `defer`?

By default, when the browser hits a `<script>` tag it **stops everything**, downloads the JS, runs it, then carries on. If your JS tries to touch elements that have not been built yet, it will fail.

`defer` tells the browser: "download this script in the background while you keep parsing HTML, then run it after the page is ready." This is the modern, recommended way to load scripts.

> Rule of thumb: **always** add `defer` to your `<script>` tags.

---

## 16.3 `console.log` and the DevTools console

`console.log` is your best friend. It prints values to the console so you can see what your code is doing.

```js
console.log("A plain string");
console.log(42);
console.log(true);
console.log("Two values:", 5, 10);
```

**Expected output:**
```
A plain string
42
true
Two values: 5 10
```

Other handy console methods:

```js
console.warn("This is a warning");   // yellow background
console.error("This is an error");   // red background
console.table([{ id: 1 }, { id: 2 }]); // nice table view
```

You can also type code directly into the console — try typing `1 + 1` and pressing **Enter**.

---

## 16.4 Variables: `let` and `const`

A **variable** is a labelled box that holds a value. You put something in, and you can take it out later by name.

### `const` — the locked box

`const` means "constant". Once you put a value in, you cannot replace it.

```js
const name = "Prabin";
const college = "Triton";
console.log(name, college);
```

**Output:**
```
Prabin Triton
```

If you try to reassign it, JS will throw an error:

```js
const age = 20;
age = 21; // TypeError: Assignment to constant variable.
```

### `let` — the rewritable box

`let` lets you change the value later.

```js
let score = 0;
console.log(score); // 0

score = 10;
console.log(score); // 10

score = score + 5;
console.log(score); // 15
```

### The rule we will follow all course long

> **Use `const` by default. Only switch to `let` when you genuinely need to reassign.**

This sounds strict, but it stops a lot of bugs before they happen. If a value is never meant to change, locking it down makes that obvious.

### Why we avoid `var`

`var` is the original (1995) way of declaring variables. It has two annoying behaviours:

1. It ignores **block scope** (we will meet scope properly in Class 19).
2. It gets **hoisted** to the top of its function, leading to surprising bugs.

Modern JS (ES6, 2015) gave us `let` and `const` precisely to fix this. **You will never need `var` in this course.** If you see it in older code, just mentally replace it with `let`.

### Naming rules

```js
const userName = "ok";      // camelCase — preferred
const user_name = "ok";     // snake_case — works but not idiomatic
const UserName = "ok";      // PascalCase — we reserve this for classes/components
const _private = "ok";      // leading underscore — sometimes used for "internal"

// const 1user = "no";      // SyntaxError — cannot start with a digit
// const user-name = "no";  // SyntaxError — no hyphens
// const class = "no";      // SyntaxError — reserved word
```

---

## 16.5 The seven primitive types

A **primitive** is the simplest kind of value JavaScript knows about. There are seven of them.

### 1. `string` — text

```js
const single = 'single quotes';
const double = "double quotes";
const backtick = `backticks (template literal)`;
console.log(single, double, backtick);
```

We will explore the backtick version (template literals) properly in Class 24.

### 2. `number` — integers and decimals

JavaScript has only **one** number type — there is no separate "integer" vs "float".

```js
const age = 21;
const price = 9.99;
const negative = -5;
const tiny = 0.0001;
console.log(age, price, negative, tiny);
```

There are also two special number values:

```js
console.log(1 / 0);          // Infinity
console.log("hello" * 2);    // NaN  (Not a Number)
```

### 3. `boolean` — `true` or `false`

```js
const isStudent = true;
const isTired = false;
console.log(isStudent, isTired);
```

### 4. `null` — "intentionally nothing"

You, the programmer, are saying "this box is empty on purpose."

```js
const selectedUser = null;
console.log(selectedUser); // null
```

### 5. `undefined` — "not set yet"

JavaScript itself says "you never put anything in this box."

```js
let result;
console.log(result); // undefined
```

> A useful mental model: **`null` is the programmer saying "empty". `undefined` is JavaScript saying "nothing yet".**

### 6. `bigint` — for huge whole numbers

Regular numbers in JS start losing precision around 2^53. For larger integers, use `bigint`:

```js
const huge = 9007199254740993n; // note the trailing 'n'
console.log(huge);
```

You will rarely need this, but it exists.

### 7. `symbol` — unique identifiers

A `symbol` is a guaranteed-unique value, mostly used by library authors:

```js
const id = Symbol("id");
console.log(id);
```

You will almost never write `Symbol` directly as a beginner — but you should know the word.

---

## 16.6 `typeof` — asking what type something is

The `typeof` operator returns the type of a value as a string.

```js
console.log(typeof "hello");      // "string"
console.log(typeof 42);           // "number"
console.log(typeof true);         // "boolean"
console.log(typeof undefined);    // "undefined"
console.log(typeof 10n);          // "bigint"
console.log(typeof Symbol("x"));  // "symbol"

// One famous quirk:
console.log(typeof null);         // "object"  (a 50-year-old bug we have to live with)
```

The `typeof null` returning `"object"` is a historical mistake from 1995 that can never be fixed without breaking the entire web. Just remember: **`null` is a primitive, even though `typeof` lies about it.**

---

## 16.7 Putting it all together

**app.js**
```js
const studentName = "Aarya";
let marks = 78;
const isPassing = marks >= 40;

console.log("Name:", studentName, "-", typeof studentName);
console.log("Marks:", marks, "-", typeof marks);
console.log("Passing?", isPassing, "-", typeof isPassing);

marks = marks + 5; // bonus marks
console.log("Updated marks:", marks);
```

**Expected output:**
```
Name: Aarya - string
Marks: 78 - number
Passing? true - boolean
Updated marks: 83
```

Try changing `marks` to a string like `"seventy-eight"` and see what `isPassing` becomes!

---

## Practice Exercises

1. **Five types**: Declare five `const` variables, one of each of these types: `string`, `number`, `boolean`, `null`, `undefined`. Log each one along with its `typeof` to the console.

2. **Profile card**: Create variables for `firstName`, `lastName`, `age`, `isStudent`. Print a single line that reads:
   ```
   Aarya Sharma, age 20, student: true
   ```
   (For now, use string concatenation with `+`. We will learn a nicer way in Class 24.)

3. **`const` vs `let`**: Try to write a small program that breaks on purpose by reassigning a `const`. Read the error message in DevTools. What line does it point to?

4. **`null` vs `undefined`**: Create two variables, one set to `null`, one declared but never assigned. Log both, and log their `typeof`. Explain in a comment which one you (the programmer) made empty and which one JavaScript left empty.

---

## Key Takeaways
- JavaScript runs in the browser today, and (from Class 26) on Node.js too.
- Always link scripts with `<script src="..." defer></script>`.
- Use `const` by default, `let` when you must reassign, and never `var`.
- The seven primitives are `string`, `number`, `boolean`, `null`, `undefined`, `bigint`, `symbol`.
- `typeof` tells you the type of a value — except for the `null` quirk.

Next class: we make decisions in our code with operators, conditionals and the truthy/falsy table.
