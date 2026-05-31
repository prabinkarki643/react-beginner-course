# Class 26: Node.js, npm and Tooling

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Classes 16–25 (JavaScript fundamentals, modules, fetch)

## What You Will Learn
- What Node.js is and how it differs from the browser
- Running JavaScript files with `node` and using the REPL
- Creating a project with `npm init -y` and reading `package.json`
- The difference between `dependencies` and `devDependencies`
- Semantic versioning, `^` and `~`
- Installing, running and one-off-executing packages with `npm install`, `npm run` and `npx`
- Why we never commit `node_modules` to Git

---

## 26.1 What is Node.js?

Up to now, every line of JavaScript you have written has lived inside a web page and been run by your browser. That works for the front end, but JavaScript is just a language — there is no rule saying it has to stay in the browser.

**Node.js is a programme that runs JavaScript on your computer, outside the browser.** It uses the same engine that powers Google Chrome (called V8), but wraps it so that JavaScript can read files, talk to the network, run scripts, and start servers.

A simple way to picture it:

| Where JavaScript runs | What it can do |
|-----------------------|----------------|
| **Browser** (Chrome, Firefox, etc.) | Change the page, listen to clicks, call APIs over `fetch`. Cannot read files on your computer. |
| **Node.js** (your terminal) | Read and write files, run command-line tools, start servers. Has **no `window`, no `document`, no DOM**. |

You are not going to write servers in this course — that is the back-end course's job. We need Node.js for one reason only: **every modern front-end tool runs on it.** Vite, TypeScript, ESLint, Tailwind, shadcn — all of them are command-line programmes written in JavaScript that need Node.js to run.

> **Analogy:** the browser is a kitchen for cooking web pages. Node.js is the workshop in the back where you sharpen the knives, build the trolleys and prepare the ingredients. You need both, but they do very different jobs.

---

## 26.2 Browser vs Node: quick comparison

```js
// Works in the browser, breaks in Node:
document.querySelector("h1"); // ReferenceError: document is not defined

// Works in Node, breaks in the browser:
const fs = require("fs");
fs.readFileSync("notes.txt", "utf-8");
```

Both files are "just JavaScript". The difference is **what global objects and APIs are available**. Knowing where your code is going to run is half the battle.

You should already have Node.js installed from Class 1. Check it quickly:

```bash
node -v
npm -v
```

You should see two version numbers (anything from `v18` or above is fine for this course). If you get an error, jump back to Class 1 and install the LTS version from [nodejs.org](https://nodejs.org).

---

## 26.3 Running a `.js` file with Node

Create a folder somewhere sensible (e.g. `Desktop/node-playground`) and open it in VS Code. Make a file called `hello.js`:

```js
// hello.js
const name = "Triton";
console.log(`Hello from Node, ${name}!`);

const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((n) => n * 2);
console.log("Doubled:", doubled);
```

Open the VS Code terminal (`View → Terminal` or `` Ctrl+` ``) and run:

```bash
node hello.js
```

**Expected output:**

```
Hello from Node, Triton!
Doubled: [ 2, 4, 6, 8, 10 ]
```

Notice there is no HTML, no browser, no `<script>` tag. Node has loaded your file straight into its JavaScript engine.

> **macOS vs Windows:** the `node` command is identical on both. Only file paths differ — on Windows the terminal might show `C:\Users\you\Desktop\...` instead of `/Users/you/Desktop/...`.

---

## 26.4 The Node REPL

REPL stands for **R**ead–**E**val–**P**rint–**L**oop. It is an interactive shell for typing one line of JavaScript at a time. Run:

```bash
node
```

You will see a `>` prompt. Try a few things:

```
> 2 + 2
4
> const greeting = "Hi"
undefined
> greeting.toUpperCase()
'HI'
> [1, 2, 3].reduce((a, b) => a + b, 0)
6
> .exit
```

Press `Ctrl+C` twice (or type `.exit`) to leave. The REPL is brilliant for testing tiny snippets without having to create a file.

---

## 26.5 What is npm?

**npm** stands for **N**ode **P**ackage **M**anager. It comes bundled with Node.js and does two jobs:

1. **Downloads packages** — chunks of reusable code other developers have published.
2. **Manages your project** — keeps track of which packages your project depends on so anyone can rebuild it.

A **package** is just a folder of JavaScript code published on the npm registry ([npmjs.com](https://www.npmjs.com/)). Some you will meet in this course:

| Package | What it does |
|---------|--------------|
| `react` | The React library itself |
| `vite` | The build tool that runs our dev server |
| `typescript` | The TypeScript compiler |
| `tailwindcss` | The utility-first CSS framework |
| `zod` | Data validation |

---

## 26.6 Creating a project with `npm init -y`

Every npm project starts with a `package.json` file. Let's make one:

```bash
mkdir greeter-app
cd greeter-app
npm init -y
```

The `-y` flag means "say yes to all the defaults" so you do not get asked ten questions. Open the folder in VS Code and look at the generated `package.json`:

```json
{
  "name": "greeter-app",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

> **Think of `package.json` as a recipe card.** It lists the project's name, version, the ingredients (packages) it needs, and the commands (scripts) you can run to cook it.

---

## 26.7 Anatomy of `package.json`

Let's go through the fields you will actually care about.

| Field | What it does |
|-------|--------------|
| `name` | The project's name (lowercase, no spaces). |
| `version` | The project's version, in the form `MAJOR.MINOR.PATCH` (e.g. `1.4.2`). |
| `scripts` | Custom commands you can run with `npm run`. |
| `dependencies` | Packages your **finished app needs to run** (e.g. React). |
| `devDependencies` | Packages **only needed during development** (e.g. TypeScript, Vite, Prettier). |
| `main` | The file that loads first when someone imports your package. You can ignore this for apps. |

### dependencies vs devDependencies

- A user running your finished React app needs **React** itself to be in the bundle → that goes in `dependencies`.
- A user does **not** need the TypeScript compiler at runtime — the code is already compiled to JavaScript by the time it reaches them → that goes in `devDependencies`.

The split keeps production builds smaller and clearer.

---

## 26.8 Semantic versioning, `^` and `~`

Open any real `package.json` and you will see versions written like this:

```json
"dependencies": {
  "react": "^18.3.1",
  "axios": "~1.7.2"
}
```

Versions follow **semantic versioning** (semver) with three numbers: `MAJOR.MINOR.PATCH`.

| Part | Bumped when… |
|------|--------------|
| **MAJOR** (`18.x.x`) | Breaking changes — old code might stop working. |
| **MINOR** (`x.3.x`) | New features added, but old code still works. |
| **PATCH** (`x.x.1`) | Bug fixes only, no new features. |

The little symbol in front tells npm how flexible it is allowed to be when updating:

| Symbol | Meaning | Example range |
|--------|---------|---------------|
| `^18.3.1` | Allow any **minor or patch** update, no major changes. | `>=18.3.1 <19.0.0` |
| `~1.7.2` | Allow only **patch** updates. | `>=1.7.2 <1.8.0` |
| `18.3.1` (no symbol) | Exact version, never change. | only `18.3.1` |

`^` is the default that `npm install` adds, and it is the right choice 95% of the time.

---

## 26.9 Installing packages

Let's actually install something. Inside `greeter-app`:

```bash
# A production dependency — code your app needs to run
npm install chalk

# A dev-only dependency — only needed while developing
npm install --save-dev prettier
```

Shorthand forms (used everywhere in the wild):

```bash
npm i chalk            # same as npm install chalk
npm i -D prettier      # same as npm install --save-dev prettier
```

Three things have now happened:

1. A `node_modules/` folder has appeared — full of downloaded code.
2. A `package-lock.json` file has appeared — a precise lockfile recording **exact** versions used.
3. Your `package.json` now has:

```json
"dependencies": {
  "chalk": "^5.3.0"
},
"devDependencies": {
  "prettier": "^3.3.3"
}
```

> **`package.json` says "any 5.x of chalk is fine". `package-lock.json` says "but in this project, right now, we are using exactly 5.3.0".** Always commit both files to Git.

---

## 26.10 npm scripts and `npm run`

The `scripts` block in `package.json` lets you give friendly names to commands. Edit your `package.json`:

```json
{
  "name": "greeter-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "format": "prettier --write ."
  },
  "dependencies": {
    "chalk": "^5.3.0"
  },
  "devDependencies": {
    "prettier": "^3.3.3"
  }
}
```

Create `index.js`:

```js
// index.js
import chalk from "chalk";

const name = process.argv[2] ?? "world";
console.log(chalk.green(`Hello, ${name}!`));
console.log(chalk.yellow("You ran this from an npm script."));
```

Because `chalk` v5 is an ES module, add this one line to `package.json` so Node treats your file as a module:

```json
"type": "module",
```

Now run:

```bash
npm start
npm start -- Prabin
npm run format
```

**Expected output (first command):**

```
Hello, world!
You ran this from an npm script.
```

A few rules of thumb:

- `npm start` and `npm test` are special — you can drop the `run`.
- For every other script, you must say `npm run <script-name>`.
- Anything after `--` is passed to the underlying command.

---

## 26.11 `npx` — run a package without installing it

Sometimes you want to use a tool **once** without adding it to your project. That is what `npx` is for. It downloads the package to a cache, runs it, and forgets it.

```bash
# Create a new React + Vite project (we will use this in Class 29)
npx create-vite@latest my-app

# Initialise shadcn/ui in an existing project
npx shadcn@latest init
```

You will see `npx` constantly when bootstrapping new projects. The rule is simple:

- **One-off tool?** Use `npx`.
- **Tool used every day by this project?** `npm install --save-dev` it.

---

## 26.12 `node_modules` and `.gitignore`

After installing a few packages, your `node_modules/` folder may already hold **thousands of files**. It can easily grow to hundreds of megabytes.

**Golden rules:**

1. **Never** edit anything inside `node_modules/` — your changes will be wiped the next time someone runs `npm install`.
2. **Never** commit `node_modules/` to Git. It is huge, machine-specific, and entirely re-creatable.
3. Anyone who clones your project can rebuild it with a single command:

```bash
npm install
```

To keep Git well behaved, create a file called `.gitignore` in the project root:

```
node_modules
.env
.DS_Store
dist
```

We will look at Git properly in Class 27, but get into the habit now: **every Node project gets a `.gitignore` with `node_modules` in it from day one.**

---

## Practice Exercises

### Exercise 1: Build a tiny greeter project
1. Create a folder `morning-greeter` and run `npm init -y` in it.
2. Add `"type": "module"` to `package.json`.
3. Install `chalk` as a normal dependency.
4. Create `index.js` that prints a coloured "Good morning, <name>!" using `chalk` and the first command-line argument.
5. Add an npm script called `greet` that runs your file.
6. Run it: `npm run greet -- Prabin`.

### Exercise 2: Read a real `package.json`
1. Visit `https://github.com/vitejs/vite/blob/main/package.json` in your browser.
2. Find **three** things in `devDependencies` you do not recognise. Google each one and write a one-sentence description in a `notes.md` file.
3. Identify which fields use `^` and which use exact versions. Why might library maintainers be stricter than app developers?

### Exercise 3: Break it on purpose
1. Inside your `morning-greeter` project, delete the `node_modules/` folder.
2. Run `node index.js`. What error do you see?
3. Run `npm install` and try again. What just happened, and why does it make `node_modules` so easy to throw away?

---

## Key Takeaways
- **Node.js** runs JavaScript outside the browser; there is no DOM, but there is access to files and the network.
- **`npm init -y`** scaffolds a `package.json`, the recipe card of every JavaScript project.
- **`dependencies`** ship with your app; **`devDependencies`** are only needed while you build it.
- **Semver** (`^`, `~`) controls how flexible npm is when updating packages.
- **`npm run <script>`** executes a named command from `package.json`; **`npx`** runs a package once without installing it.
- **`node_modules/`** is large, machine-specific and disposable — always `.gitignore` it.
- Next class we tame the chaos: Git and GitHub, the time machine that protects every project you ever write.
