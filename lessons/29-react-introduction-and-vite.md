# Class 29: React Introduction & Vite + TypeScript Setup

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 28 (TypeScript Basics)

## What You Will Learn
- What React is and why it has become the most popular UI library
- The "component-based" way of thinking about a user interface
- Why we use Vite as our build tool and dev server
- How to scaffold a brand-new React + TypeScript project with one command
- A guided tour of every important file in a fresh Vite project
- How to run the dev server and make your first change

---

## 29.1 What is React?

React is a **JavaScript library for building user interfaces**. It was created at Facebook (now Meta) in 2013 and has become the most widely used front-end library in the world.

So far in this course we have built pages with plain HTML, styled them with CSS, and made them interactive with JavaScript and DOM manipulation. That worked beautifully for small pages, but as soon as a page has dozens of interactive parts — buttons, lists, forms, modals, filters — keeping the DOM in sync with our data becomes painful.

**The vanilla problem.** Imagine a shopping basket. When the user adds an item you must: update the basket array, find the basket count element, update its text, find the basket list element, build a new `<li>`, append it, find the total price element, recalculate the total, update its text. Miss one step and the UI lies to the user.

**React's idea.** You describe **what the UI should look like for a given piece of data**, and React works out which bits of the page need to change. You stop touching the DOM by hand — you change the data, React updates the page.

### The LEGO analogy

Think of a React app as something built from **LEGO bricks**. Each brick is small, has a clear purpose, and snaps together with others. In React these bricks are called **components**.

A typical page might be made of:

```
App
├── Navbar
│   ├── Logo
│   └── NavLinks
├── HeroSection
│   ├── Heading
│   └── CallToActionButton
├── ProductGrid
│   ├── ProductCard  (×12)
│   │   ├── ProductImage
│   │   ├── ProductTitle
│   │   └── PriceTag
└── Footer
```

Each component is a self-contained piece. `ProductCard` is reused twelve times with different data. If you improve `PriceTag`, every price on the site improves at once. This is the productivity multiplier React gives you.

---

## 29.2 Why Vite?

To run React in the browser, our `.tsx` files need to be transformed into plain JavaScript. We also want a fast dev server that reloads the page automatically when we save a file. The tool that does all this is called a **build tool**.

**Vite** (pronounced "veet", French for "fast") is the modern build tool we will use throughout this course.

Why Vite over older tools (like Create React App):

| | Vite | Create React App (old) |
|--|------|------------------------|
| Cold start | Under a second | 20–60 seconds |
| Hot reload | Instant | Slow on big apps |
| Built-in TypeScript | Yes | Yes |
| Status | Actively maintained, the modern standard | Deprecated |

Vite uses native browser ES modules during development, which is why it feels so fast.

---

## 29.3 Creating Your First React Project

Open a terminal in the folder where you keep your coursework (for example, `Documents/triton/`). Then run:

```bash
npm create vite@latest my-app -- --template react-ts
```

Let us unpack that command:

- `npm create vite@latest` — ask npm to use the latest Vite project creator
- `my-app` — the folder name for the new project
- `--` — separator between npm's flags and Vite's flags
- `--template react-ts` — use the React + TypeScript template

You should see a prompt confirming the template, then the project will be generated.

Now move into the folder and install dependencies:

```bash
cd my-app
npm install
```

`npm install` downloads every library listed in `package.json` (React, TypeScript, Vite, etc.) into a folder called `node_modules`. This can take a minute the first time.

Finally, start the dev server:

```bash
npm run dev
```

You will see something like:

```
  VITE v5.4.0  ready in 312 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
```

Open `http://localhost:5173` in your browser. You should see the default Vite + React welcome page with a spinning logo and a counter button. **Congratulations — you are running React.**

> **Tip**: Leave the terminal running. Every time you save a file, the browser will reload automatically. To stop the server, press `Ctrl + C` in the terminal.

---

## 29.4 Project Tour — What Every File Does

Open the project folder in VS Code. You will see a structure like this:

```
my-app/
├── node_modules/         ← installed libraries (ignore, never edit)
├── public/               ← static files served as-is (favicon, robots.txt)
│   └── vite.svg
├── src/                  ← YOUR CODE LIVES HERE
│   ├── assets/
│   │   └── react.svg
│   ├── App.tsx           ← the main component
│   ├── App.css           ← styles for App
│   ├── main.tsx          ← the entry point — boots React
│   ├── index.css         ← global styles
│   └── vite-env.d.ts     ← Vite's TypeScript helpers
├── .gitignore
├── index.html            ← the only HTML file in the project
├── package.json          ← project info, scripts, dependencies
├── package-lock.json     ← exact dependency versions (auto-managed)
├── tsconfig.json         ← TypeScript settings
├── tsconfig.node.json    ← TypeScript settings for Vite's config
└── vite.config.ts        ← Vite's settings
```

Let us look at the important ones.

### `index.html`

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite + React + TS</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

This is the **one and only HTML file** the browser ever loads. Notice two important lines:

1. `<div id="root"></div>` — an empty container. React will fill this with the entire app.
2. `<script type="module" src="/src/main.tsx"></script>` — loads our entry point.

### `src/main.tsx`

```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.tsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
)
```

This file does three things:

1. Finds the `<div id="root">` element from `index.html`.
2. Creates a "React root" attached to it.
3. Renders our `<App />` component inside it.

The `!` after `getElementById('root')` is a TypeScript hint that says "trust me, this element definitely exists". Without it, TypeScript would warn that the value could be `null`.

`<React.StrictMode>` is a development-only wrapper that turns on extra warnings and helps you catch common mistakes early. Leave it on.

### `src/App.tsx`

This is the component you will edit most. We will look at it in the next section.

### `vite.config.ts`

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
})
```

This tells Vite to use the React plugin so it understands `.tsx` files. You rarely edit this in beginner projects.

### `tsconfig.json`

The settings for the TypeScript compiler — which rules are strict, which files to include, where output goes. Vite reads this so it knows how to compile your TS code.

### `package.json`

```json
{
  "name": "my-app",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "preview": "vite preview"
  },
  "dependencies": { "react": "...", "react-dom": "..." },
  "devDependencies": { "vite": "...", "typescript": "...", ... }
}
```

The scripts are how you run common tasks:

- `npm run dev` — start the dev server (what we did above)
- `npm run build` — produce an optimised production build in `dist/`
- `npm run preview` — preview the production build locally

---

## 29.5 Editing `App.tsx` — Your First Change

Open `src/App.tsx`. It looks roughly like this (Vite's default):

```tsx
import { useState } from 'react'
import reactLogo from './assets/react.svg'
import viteLogo from '/vite.svg'
import './App.css'

function App() {
  const [count, setCount] = useState(0)

  return (
    <>
      <div>
        <a href="https://vite.dev" target="_blank">
          <img src={viteLogo} className="logo" alt="Vite logo" />
        </a>
      </div>
      <h1>Vite + React</h1>
      <div className="card">
        <button onClick={() => setCount((c) => c + 1)}>
          count is {count}
        </button>
      </div>
    </>
  )
}

export default App
```

Do not worry about every detail yet — we will cover JSX, components, props, and state in the next several classes. For now, just **replace the whole file** with:

```tsx
function App() {
  const studentName: string = 'Aarav';
  const today: string = new Date().toLocaleDateString('en-GB');

  return (
    <div style={{ fontFamily: 'system-ui, sans-serif', padding: '2rem' }}>
      <h1>Hello, React!</h1>
      <p>My name is {studentName}.</p>
      <p>Today is {today}.</p>
      <p>This page was built with Vite + React + TypeScript.</p>
    </div>
  );
}

export default App;
```

Save the file. The browser will reload by itself. You should see your three lines on screen — no `Ctrl + R` required.

**Try changing the value of `studentName` to your own name** and watch the page update the instant you save. This automatic refresh is called **Hot Module Replacement (HMR)** and it is one of the things that makes Vite feel so quick.

---

## 29.6 A Mental Model for Where Things Run

Here is the journey from `npm run dev` to a page on screen:

1. Vite starts a local server at `http://localhost:5173`.
2. Your browser asks for `index.html`.
3. The HTML asks for `/src/main.tsx`.
4. Vite compiles `main.tsx` on the fly (TS → JS, JSX → function calls) and sends it to the browser.
5. `main.tsx` finds `<div id="root">` and renders `<App />` inside it.
6. The `App` function runs and returns some JSX, which React turns into real DOM nodes.
7. The page appears.

Every time you save a file, Vite recompiles the changed module and tells the browser to swap it in — no full reload needed.

---

## Practice Exercises

1. **Scaffold and run.** From scratch, create a project called `class-29-demo` using the Vite command. Install dependencies. Start the dev server. Confirm you can open it in your browser.
2. **Personalise the page.** In `App.tsx`, replace the contents so the page shows your name, your course (BCA), and the name of your college. Use the `style` prop to give the page some padding and a sensible font.
3. **Break things on purpose.** Inside `App.tsx`, delete the closing `</div>` tag. Save the file. Read the error message in the terminal and the browser overlay. Put it back and confirm the error disappears. Knowing what errors look like is half the battle.
4. **Look around `package.json`.** Open `package.json` and identify: (a) the React version, (b) the Vite version, (c) the three scripts available. Write the answers in a comment at the top of `App.tsx`.

---

## Key Takeaways
- React is a library for building UIs from **components** — small, reusable pieces.
- **Vite** is a fast, modern build tool that compiles our `.tsx` files and serves them with hot reload.
- `npm create vite@latest my-app -- --template react-ts` scaffolds a complete React + TypeScript project in one command.
- `index.html` is the single HTML file. `src/main.tsx` is the entry point. `src/App.tsx` is the main component you edit.
- `npm run dev` starts the development server at `http://localhost:5173`.
- Next class: we will learn **JSX/TSX** properly and write our very first custom component.
