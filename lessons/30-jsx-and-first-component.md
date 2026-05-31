# Class 30: JSX & Your First Component

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 29 (Vite + React + TS project running)

## What You Will Learn
- What JSX (and TSX) really is and why it exists
- How to mix JavaScript values into your markup with `{ }`
- The handful of ways JSX differs from HTML — `className`, `htmlFor`, self-closing tags, camelCase events
- How to write, export and import your very own component
- Fragments (`<>...</>`) for when you do not want an extra wrapper
- The common JSX traps that catch every beginner once

---

## 30.1 What is JSX?

JSX stands for **JavaScript XML**. It lets you write HTML-like markup directly inside your JavaScript (or TypeScript) code. When the file extension is `.tsx`, it is technically called **TSX** — the same thing with TypeScript's type checking on top.

A tiny example:

```tsx
const greeting = <h1>Hello, world!</h1>;
```

That line looks like HTML in the middle of a TS file. It is not. Behind the scenes Vite converts it into a normal function call that returns a description of the element:

```ts
const greeting = React.createElement('h1', null, 'Hello, world!');
```

You will never write `React.createElement` yourself — that is the compiled output. But it is useful to know that **JSX is just sugar for a function call**. This is why JSX behaves like an expression: it has a value, you can assign it to a variable, return it from a function, or put it in an array.

> **Why mix markup into code?** Because the structure of your UI and the data that drives it live together. When you change the data, the markup changes in the same file — no jumping between three different files to update one feature.

---

## 30.2 Embedding Expressions with `{ }`

Anywhere inside JSX, curly braces switch you back into TypeScript. Anything between `{` and `}` is a TS expression, and its value is inserted into the output.

```tsx
function App() {
  const name: string = 'Sita';
  const tasks: number = 3;
  const isOnline: boolean = true;

  return (
    <div>
      <h1>Hello, {name}!</h1>
      <p>You have {tasks} tasks today.</p>
      <p>Tasks doubled: {tasks * 2}</p>
      <p>Status: {isOnline ? 'Online' : 'Offline'}</p>
      <p>Today is {new Date().toLocaleDateString('en-GB')}</p>
    </div>
  );
}

export default App;
```

When you save this and look at the browser you will see something like:

```
Hello, Sita!
You have 3 tasks today.
Tasks doubled: 6
Status: Online
Today is 25/05/2026
```

**Rules for `{ }`:**

- Only **expressions** are allowed — things that have a value (variables, function calls, ternaries, arithmetic).
- **Statements** like `if`, `for`, `let x = ...` are **not** allowed inside `{ }`. (We will learn the React way to do those — `.map()` and ternaries — in Class 32.)
- Booleans, `null` and `undefined` render as nothing. So `{isOnline && <p>Welcome back</p>}` is a clean way to show something conditionally.

---

## 30.3 How JSX Differs From HTML

JSX looks like HTML but there are some small but important differences. They exist because JSX is really JavaScript, and a few HTML attribute names clash with reserved JS keywords.

### 1. `className` instead of `class`

`class` is a reserved word in JavaScript, so JSX uses `className`.

```tsx
// HTML
<div class="card">...</div>

// JSX/TSX
<div className="card">...</div>
```

### 2. `htmlFor` instead of `for`

`for` is also reserved (used by `for` loops), so the `<label>` attribute becomes `htmlFor`.

```tsx
<label htmlFor="email">Email</label>
<input id="email" type="email" />
```

### 3. All tags must be closed

In HTML you can write `<br>`, `<img>`, `<input>` without closing them. In JSX every tag must be closed — either with `</tag>` or as a **self-closing** tag ending in ` />`.

```tsx
<br />
<hr />
<img src="logo.png" alt="Logo" />
<input type="text" />
```

Forget the slash and you will get a compile error.

### 4. Attributes are camelCase, not lowercase

HTML uses lowercase (`onclick`, `tabindex`, `readonly`). JSX uses camelCase (`onClick`, `tabIndex`, `readOnly`).

```tsx
<button onClick={handleClick} tabIndex={0}>Click me</button>
<input type="text" readOnly />
```

### 5. Inline styles take an **object**, not a string

```tsx
// HTML
<div style="color: red; padding: 8px;">...</div>

// JSX/TSX
<div style={{ color: 'red', padding: '8px' }}>...</div>
```

Two pairs of braces: the outer `{ }` switches to TS, the inner `{ }` is the actual JS object. Property names are camelCase (`backgroundColor`, not `background-color`).

### 6. A component must return ONE root element

```tsx
// WRONG — two top-level elements
function App() {
  return (
    <h1>Title</h1>
    <p>Paragraph</p>
  );
}

// RIGHT — wrap them
function App() {
  return (
    <div>
      <h1>Title</h1>
      <p>Paragraph</p>
    </div>
  );
}
```

If you do not want an extra `<div>` in your final HTML, use a **Fragment** (covered in 30.6).

### Quick reference table

| HTML | JSX/TSX |
|------|---------|
| `class="x"` | `className="x"` |
| `for="id"` | `htmlFor="id"` |
| `onclick="..."` | `onClick={handler}` |
| `onchange="..."` | `onChange={handler}` |
| `<br>` | `<br />` |
| `<img src="...">` | `<img src="..." />` |
| `style="color: red"` | `style={{ color: 'red' }}` |
| `tabindex` | `tabIndex` |
| `readonly` | `readOnly` |

---

## 30.4 Writing Your First Component

A **component** is a TypeScript function that returns JSX. That is it.

There are three rules every component must follow:

1. The name must start with a **capital letter** — `Header`, not `header`. React uses the capital letter to tell the difference between your component and a normal HTML tag.
2. It must return JSX (or `null`).
3. It can only return **one root element** (use a Fragment if you need to).

### Example: a `Greeting` component

Create a new file `src/Greeting.tsx`:

```tsx
function Greeting() {
  return (
    <div style={{ padding: '1rem', border: '1px solid #ccc', borderRadius: '8px' }}>
      <h2>Welcome to React!</h2>
      <p>This greeting comes from its own component.</p>
    </div>
  );
}

export default Greeting;
```

The line `export default Greeting;` makes the component available to other files.

### Using it in `App.tsx`

Now open `src/App.tsx` and import the new component:

```tsx
import Greeting from './Greeting';

function App() {
  return (
    <div style={{ padding: '2rem', fontFamily: 'system-ui, sans-serif' }}>
      <h1>My First Component-Based Page</h1>
      <Greeting />
      <Greeting />
      <Greeting />
    </div>
  );
}

export default App;
```

Save and refresh — you will see the same greeting card three times. That is the power of components: write once, use everywhere.

> **Notice the self-closing tag**: `<Greeting />`. Components are used exactly like HTML tags, with the same closing rule.

### Default export vs named export

```tsx
// Default export — one per file, imported without braces
export default Greeting;
// import Greeting from './Greeting'

// Named export — many per file, imported with braces
export function Greeting() { ... }
// import { Greeting } from './Greeting'
```

In this course we will mostly use **default exports** for components (one component per file). We use **named exports** for utility files like `types/user.ts` where you might export several interfaces.

---

## 30.5 Splitting `App.tsx` Into Components

Let us refactor a simple page into three components.

**Before — everything in `App.tsx`:**

```tsx
function App() {
  return (
    <div style={{ fontFamily: 'system-ui', padding: '1rem' }}>
      <header style={{ borderBottom: '1px solid #eee', padding: '1rem 0' }}>
        <h1>My Blog</h1>
      </header>
      <main>
        <p>Welcome to my blog.</p>
      </main>
      <footer style={{ borderTop: '1px solid #eee', padding: '1rem 0', marginTop: '2rem' }}>
        <p>© 2026 Prabin</p>
      </footer>
    </div>
  );
}

export default App;
```

**After — three small components:**

```tsx
// src/Header.tsx
function Header() {
  return (
    <header style={{ borderBottom: '1px solid #eee', padding: '1rem 0' }}>
      <h1>My Blog</h1>
    </header>
  );
}

export default Header;
```

```tsx
// src/Footer.tsx
function Footer() {
  return (
    <footer style={{ borderTop: '1px solid #eee', padding: '1rem 0', marginTop: '2rem' }}>
      <p>© 2026 Prabin</p>
    </footer>
  );
}

export default Footer;
```

```tsx
// src/App.tsx
import Header from './Header';
import Footer from './Footer';

function App() {
  return (
    <div style={{ fontFamily: 'system-ui', padding: '1rem' }}>
      <Header />
      <main>
        <p>Welcome to my blog.</p>
      </main>
      <Footer />
    </div>
  );
}

export default App;
```

The page looks identical — but now `Header` and `Footer` can be reused on any future page, and `App.tsx` is far easier to read.

---

## 30.6 Fragments — When You Do Not Want a Wrapper

Sometimes you want to return multiple elements without adding an extra `<div>` to the DOM. React gives you a **Fragment** for this — it groups elements together but produces no HTML of its own.

```tsx
function NameAndAge() {
  return (
    <>
      <p>Name: Prabin</p>
      <p>Age: 30</p>
    </>
  );
}
```

`<> ... </>` is the short Fragment syntax. The longer form `<React.Fragment> ... </React.Fragment>` does the same job but is only needed when you want to pass a `key` prop to it (we will see this in Class 32).

**Use a Fragment when:**
- You are returning a list of sibling elements that do not belong inside any particular tag.
- Wrapping in a `<div>` would break your layout (for example, inside a flex or grid container).

---

## 30.7 Common JSX Gotchas

These are the four traps every beginner falls into once. Learn them now and save yourself an hour later.

### 1. Forgetting the single root element

```tsx
// WRONG — adjacent elements
return (
  <h1>Hello</h1>
  <p>World</p>
);
```

Fix: wrap in a `<div>` or a Fragment.

### 2. Lower-case component names

```tsx
// WRONG — React treats this as an unknown HTML tag
function greeting() { return <p>Hi</p>; }
<greeting />

// RIGHT
function Greeting() { return <p>Hi</p>; }
<Greeting />
```

### 3. Missing the curly braces

```tsx
// WRONG — renders the literal text "name"
<h1>Hello, name!</h1>

// RIGHT — renders the value of the variable
<h1>Hello, {name}!</h1>
```

### 4. Using `class` instead of `className`

```tsx
// WRONG — TypeScript warning, attribute ignored
<div class="card">...</div>

// RIGHT
<div className="card">...</div>
```

### 5. Returning the wrong thing

A component must return JSX or `null`. Returning a plain object is the most common accidental mistake:

```tsx
// WRONG — returns an object
function App() {
  return { hello: 'world' };
}

// RIGHT
function App() {
  return <p>hello world</p>;
}
```

---

## Practice Exercises

1. **Personal card.** Create a new component `ProfileCard` in `src/ProfileCard.tsx` that displays your name, your favourite programming language and your email — each inside its own `<p>`. Style the card with `style` (border, padding, max width). Use it three times inside `App.tsx`.
2. **Expression playground.** Inside `App.tsx`, declare four variables: `name: string`, `age: number`, `isStudent: boolean`, and `subjects: string[]`. Render each one in JSX using `{ }`. Show `subjects.length` separately, and use a ternary to render "Adult" or "Minor" based on `age`.
3. **Fix the JSX.** Copy the broken snippet below into `App.tsx`, then fix every error you can find. (There are at least five.)
   ```tsx
   function header() {
     return (
       <h1 class="title">My Page</h1>
       <img src="logo.png">
       <p style="color: red">Hello, name!</p>
     );
   }
   ```
4. **Refactor into components.** Take any of the HTML pages you built in Classes 7 or 15 and rebuild the structure as a React page made of at least four components: `Navbar`, `Hero`, `MainContent`, and `Footer`. Use inline `style` for now — proper styling comes later.

---

## Key Takeaways
- JSX/TSX is HTML-like markup that compiles down to JavaScript function calls.
- Use `{ }` to embed any TS expression. Statements are not allowed inside the braces.
- Important differences from HTML: `className`, `htmlFor`, self-closing tags, camelCase events, `style={{ }}` as an object.
- A component is a capital-letter-named function that returns JSX.
- One file = one component (default export). Import it where you need it.
- A component must return a single root element — use a Fragment `<>...</>` when you do not want an extra wrapper.
- Next class: components become much more useful when they accept **props** — inputs from the outside.
