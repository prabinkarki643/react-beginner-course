# React Beginner Course — Detailed Curriculum & Per-Class Checklist

A practical, per-class checklist for the **51-class React Beginner Course**. Each entry maps to exactly one lesson file in `lessons/` and exactly **one 60-minute class**.

**Schedule**: 3 classes per week → ~17 weeks → one semester.

---

## Phase 0 — Setup (1 class, 1 hour)

### Class 1 · Prerequisites & Environment Setup → `01-prerequisites-and-setup.md`
- [ ] What "full setup" means and why we do it on Day 1
- [ ] Install **Node.js LTS** — verify with `node -v` and `npm -v`
- [ ] Install **VS Code** and tour the UI
- [ ] Install **Git** — verify with `git --version`
- [ ] Configure Git: `git config --global user.name` / `user.email`
- [ ] Create accounts: **GitHub**, **Vercel** (via GitHub)
- [ ] Install required VS Code extensions (ES7+ React snippets, Tailwind IntelliSense, Prettier, ESLint, Auto Rename Tag, Path Intellisense, Live Server, GitLens, Material Icon Theme)
- [ ] Open a folder and create your first `.html` file using Live Server
- [ ] Browser DevTools quick tour (Elements, Console)
- **Exercise**: Build a one-page "Hello, world" HTML file and open it with Live Server

---

## Phase 1 — HTML (6 classes, 6 hours)

### Class 2 · HTML Introduction & Document Structure → `02-html-introduction.md`
- [ ] What HTML is (skeleton-of-a-page analogy)
- [ ] The HTML / CSS / JS division of labour
- [ ] `<!DOCTYPE>`, `<html>`, `<head>`, `<body>`
- [ ] Meta tags, `<title>`, character set, viewport
- [ ] Comments and indentation conventions
- **Exercise**: Create a valid HTML5 boilerplate page from memory

### Class 3 · HTML Text, Links, Images, Lists → `03-html-text-links-images.md`
- [ ] Headings `<h1>`–`<h6>` and the one-`<h1>` rule
- [ ] Paragraphs, bold, italic, `<br>`, `<hr>`
- [ ] Links: `<a href>`, `target="_blank"`, anchor links (`#id`)
- [ ] Images: `<img src alt width height>` and the role of `alt`
- [ ] Unordered, ordered and nested lists
- **Exercise**: Build an "About me" page with a photo, links, and a hobbies list

### Class 4 · HTML Tables & Forms — Part 1 → `04-html-tables-and-forms.md`
- [ ] Tables: `<table>`, `<thead>`, `<tbody>`, `<tr>`, `<th>`, `<td>`
- [ ] When to use a table (data) vs not (layout)
- [ ] Intro to forms: `<form>`, `<label>`, `<input>`, `<button>`
- [ ] The `for`/`id` pairing for accessibility
- [ ] Basic input types: `text`, `email`, `password`, `number`
- **Exercise**: Build a "marks table" of 5 subjects and a simple login form

### Class 5 · HTML Forms — Part 2 (Input Types & Attributes) → `05-html-forms-input-types.md`
- [ ] Checkboxes, radios, selects, textareas
- [ ] `placeholder`, `required`, `min`, `max`, `pattern`, `maxlength`
- [ ] `<fieldset>` and `<legend>` for grouped inputs
- [ ] File upload input (preview-only — uploading needs JS/back-end)
- [ ] HTML5 form validation built-ins
- **Exercise**: Build a "student registration" form using every input type covered

### Class 6 · Semantic HTML & Accessibility Basics → `06-html-semantic-and-accessibility.md`
- [ ] Why semantic markup matters (SEO, accessibility, maintenance)
- [ ] `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>`
- [ ] Headings outline & landmarks
- [ ] Alt text, `aria-label`, keyboard navigation basics
- [ ] When `<div>` and `<span>` are still appropriate
- **Exercise**: Refactor the registration form page to use semantic elements

### Class 7 · HTML Practice — Build a Portfolio Page → `07-html-practice-portfolio-page.md`
- [ ] Plan a 4-section single-page portfolio (Hero, About, Projects, Contact)
- [ ] Build the structure using only HTML — no styling yet
- [ ] Add a contact form
- [ ] Validate with the W3C HTML validator
- **Exercise**: Submit the completed `portfolio.html` (we style it in Class 15)

---

## Phase 2 — CSS (8 classes, 8 hours)

### Class 8 · CSS Introduction, Syntax & Selectors → `08-css-introduction-and-selectors.md`
- [ ] What CSS is (the "clothes" on the HTML skeleton)
- [ ] Three ways to add CSS (inline, internal, **external** — preferred)
- [ ] Selectors: element, class, ID, group, descendant, child, attribute
- [ ] Pseudo-classes: `:hover`, `:focus`, `:nth-child`
- [ ] Specificity — who wins when rules clash
- **Exercise**: Apply at least 5 selector types to the portfolio page

### Class 9 · Box Model, Colours, Backgrounds, Borders → `09-css-box-model-colours-backgrounds.md`
- [ ] Every element is a box: `content` → `padding` → `border` → `margin`
- [ ] `box-sizing: border-box` (and why we always set it)
- [ ] Colour systems: named, hex, RGB, HSL
- [ ] `background-color`, `background-image`, `background-size`
- [ ] Borders & border-radius
- **Exercise**: Style a card component with padding, border, shadow and rounded corners

### Class 10 · Typography & Units → `10-css-typography-and-units.md`
- [ ] `font-family`, web-safe fonts, Google Fonts
- [ ] `font-size`, `font-weight`, `line-height`, `letter-spacing`
- [ ] Units: `px`, `rem`, `em`, `%`, `vh`, `vw` — when to use which
- [ ] Text utilities: `text-align`, `text-transform`, `text-decoration`
- **Exercise**: Apply a Google Font, set a typographic scale (rem-based) to the portfolio

### Class 11 · Layout — Display, Position, Overflow, z-index → `11-css-layout-display-position.md`
- [ ] `display`: `block`, `inline`, `inline-block`, `none`
- [ ] `position`: `static`, `relative`, `absolute`, `fixed`, `sticky`
- [ ] `overflow`, `z-index`, stacking context
- [ ] Centring techniques (before Flexbox)
- **Exercise**: Build a fixed navbar and a back-to-top button using `position`

### Class 12 · Flexbox → `12-css-flexbox.md`
- [ ] When Flexbox is the right tool
- [ ] Container properties: `display: flex`, `flex-direction`, `justify-content`, `align-items`, `gap`, `flex-wrap`
- [ ] Item properties: `flex-grow`, `flex-shrink`, `flex-basis`, `order`, `align-self`
- [ ] Practical patterns: nav bar, card row, centring anything
- **Exercise**: Build a responsive navbar and a 3-card row that wraps on small screens

### Class 13 · CSS Grid → `13-css-grid.md`
- [ ] Grid vs Flexbox — when to pick which
- [ ] `display: grid`, `grid-template-columns`, `grid-template-rows`, `gap`
- [ ] `fr` units, `repeat()`, `minmax()`, `auto-fit`/`auto-fill`
- [ ] Grid areas and named templates
- **Exercise**: Build a responsive 3-column gallery with Grid

### Class 14 · Responsive Design & Media Queries → `14-css-responsive-and-media-queries.md`
- [ ] Mobile-first thinking
- [ ] Breakpoints: 640px, 768px, 1024px, 1280px (matches Tailwind defaults)
- [ ] `@media (min-width: ...)` and `@media (max-width: ...)`
- [ ] Fluid typography with `clamp()`
- [ ] Hide/show on different breakpoints
- **Exercise**: Make the portfolio page fully responsive across phone / tablet / desktop

### Class 15 · CSS Practice — Style the Portfolio → `15-css-practice-style-portfolio.md`
- [ ] Apply everything from Classes 8–14 to the portfolio page from Class 7
- [ ] Add a colour palette and typographic scale
- [ ] Use Flexbox for the nav, Grid for projects, media queries for responsiveness
- [ ] Polish: hover effects, transitions, shadows
- **Exercise**: Deliver a finished, responsive portfolio page

---

## Phase 3 — JavaScript (10 classes, 10 hours)

### Class 16 · JS Introduction, Variables & Data Types → `16-js-introduction-variables-types.md`
- [ ] What JavaScript is and where it runs (browser / Node)
- [ ] Linking JS to HTML with `<script>`, `defer`
- [ ] `console.log()` and the browser console
- [ ] `let`, `const`, **avoid** `var`
- [ ] Primitive types: `string`, `number`, `boolean`, `null`, `undefined`, `bigint`, `symbol`
- **Exercise**: Print 5 different types to the console and inspect with `typeof`

### Class 17 · Operators & Conditionals → `17-js-operators-and-conditionals.md`
- [ ] Arithmetic, assignment, comparison, logical, ternary operators
- [ ] `==` vs `===` (and why `===` always wins)
- [ ] `if` / `else if` / `else`
- [ ] `switch` statements
- [ ] Truthy & falsy values
- **Exercise**: Write a "grade calculator" that converts a percentage to a letter grade

### Class 18 · Loops → `18-js-loops.md`
- [ ] `for`, `while`, `do…while`
- [ ] `for…of` (arrays) and `for…in` (object keys)
- [ ] `break` and `continue`
- [ ] Common pitfalls (infinite loops, off-by-one)
- **Exercise**: Print the first 100 primes; build a multiplication table generator

### Class 19 · Functions → `19-js-functions.md`
- [ ] Function declarations vs expressions
- [ ] Arrow functions (and when to prefer them)
- [ ] Parameters, default values, rest parameters
- [ ] `return` and the difference between expression and statement
- [ ] Scope and closures (gentle intro)
- **Exercise**: Build a reusable `formatCurrency(amount, currency)` helper

### Class 20 · Arrays & Array Methods → `20-js-arrays-and-methods.md`
- [ ] Creating and indexing arrays
- [ ] Mutating methods: `push`, `pop`, `shift`, `unshift`, `splice`
- [ ] Non-mutating methods: `map`, `filter`, `find`, `some`, `every`, `reduce`, `includes`, `slice`
- [ ] Chaining methods
- **Exercise**: Given a list of students, compute totals, averages and a top-3 ranking

### Class 21 · Objects, Destructuring, Spread/Rest → `21-js-objects-and-destructuring.md`
- [ ] Object literals, dot vs bracket access
- [ ] Object methods, shorthand syntax
- [ ] Destructuring objects and arrays
- [ ] Spread/rest with arrays and objects
- [ ] Shallow copies and references
- **Exercise**: Convert a list of users into a normalised "by id" lookup object

### Class 22 · DOM Manipulation → `22-js-dom-manipulation.md`
- [ ] The DOM tree concept
- [ ] `document.querySelector`, `querySelectorAll`, `getElementById`
- [ ] Reading and changing `textContent`, `innerHTML`, attributes
- [ ] Creating, appending, removing nodes
- [ ] `classList`: `add`, `remove`, `toggle`
- **Exercise**: Build a "to-do list" UI that updates the DOM (in-memory only, no events yet)

### Class 23 · Events & Event Handling → `23-js-events.md`
- [ ] `addEventListener` vs inline handlers
- [ ] Event object, `preventDefault()`, `stopPropagation()`
- [ ] Click, input, change, submit, keydown
- [ ] Event delegation pattern
- **Exercise**: Make the to-do list interactive (add, toggle, delete)

### Class 24 · ES6+ Features & Modules → `24-js-es6-modules-and-features.md`
- [ ] Template literals
- [ ] Optional chaining `?.` and nullish coalescing `??`
- [ ] Short-circuit evaluation (`&&`, `||`)
- [ ] `import` / `export` (named & default)
- [ ] `type="module"` on `<script>`
- **Exercise**: Refactor the to-do app into 3 modules (`storage.js`, `dom.js`, `app.js`)

### Class 25 · Async — Promises, async/await, Fetch → `25-js-async-promises-fetch.md`
- [ ] Synchronous vs asynchronous code
- [ ] Promises: `then`/`catch`/`finally`
- [ ] `async` / `await` and `try/catch`
- [ ] The Fetch API against **JSONPlaceholder**
- [ ] Parsing JSON, handling HTTP errors
- **Exercise**: Fetch posts from `https://jsonplaceholder.typicode.com/posts` and render them

---

## Phase 4 — Tooling (2 classes, 2 hours)

### Class 26 · Node.js, npm & package.json → `26-nodejs-npm-and-tooling.md`
- [ ] What Node.js is and the difference between Node and the browser
- [ ] Running JS files with `node file.js`
- [ ] `npm init -y`, the `package.json` anatomy
- [ ] `dependencies` vs `devDependencies`
- [ ] Semantic versioning (`^`, `~`)
- [ ] `npm install`, `npm run`, `npx`
- [ ] What `node_modules` and `.gitignore` are for
- **Exercise**: Create a tiny Node project that prints a greeting via a script in `package.json`

### Class 27 · Git & GitHub Essentials → `27-git-and-github-basics.md`
- [ ] Why version control matters (the "save game" analogy)
- [ ] `git init`, `git status`, `git add`, `git commit`
- [ ] Writing good commit messages
- [ ] Branching & merging basics (`git switch -c`, `git merge`)
- [ ] Push to GitHub (`git remote add origin`, `git push -u`)
- [ ] Pull requests, `README.md`, `.gitignore`
- [ ] Recovering from common mistakes (`git restore`, `git reset --soft HEAD~1`)
- **Exercise**: Publish the portfolio page to a public GitHub repository

---

## Phase 5 — TypeScript (1 class, 1 hour)

### Class 28 · TypeScript Basics → `28-typescript-basics.md`
- [ ] What TypeScript is and why it exists
- [ ] Compile-time vs runtime errors
- [ ] Primitive type annotations: `string`, `number`, `boolean`
- [ ] Arrays and tuples
- [ ] Object types and `interface` vs `type`
- [ ] Union and intersection types
- [ ] Optional properties (`?`), readonly, narrowing with `typeof`
- [ ] Function parameter and return types
- [ ] Generics (gentle intro with `Array<T>` and a simple generic function)
- [ ] Utility types you will actually use: `Partial`, `Pick`, `Omit`, `Record`
- **Exercise**: Convert a JS function and an object to TS; explore errors deliberately

---

## Phase 6 — React Fundamentals (10 classes, 10 hours)

### Class 29 · React Introduction & Vite + TypeScript Setup → `29-react-introduction-and-vite.md`
- [ ] What React is (component-based UI)
- [ ] Why Vite (fast dev server, ESM, built-in TS)
- [ ] Create a project: `npm create vite@latest my-app -- --template react-ts`
- [ ] Project tour: `index.html`, `main.tsx`, `App.tsx`, `vite.config.ts`
- [ ] `npm install`, `npm run dev`
- **Exercise**: Scaffold a Vite + React + TS app and edit the default page

### Class 30 · JSX & Your First Component → `30-jsx-and-first-component.md`
- [ ] JSX/TSX = HTML inside TS
- [ ] Embedding expressions with `{ }`
- [ ] `className` (not `class`), self-closing tags
- [ ] Creating a function component, exporting and importing it
- [ ] Fragment `<>...</>`
- **Exercise**: Build `Header`, `Footer` and a `<HelloCard name="..." />` component

### Class 31 · Components, Props & Composition → `31-react-components-and-props.md`
- [ ] Props as inputs to components
- [ ] Typing props with `interface` / `type`
- [ ] Default values for props
- [ ] Children prop and composition (the "container" pattern)
- [ ] Splitting a UI into components
- **Exercise**: Build a typed `<Card title description footer>` with a `children` slot

### Class 32 · Lists, Keys & Conditional Rendering → `32-react-lists-keys-conditional.md`
- [ ] Rendering arrays with `.map()`
- [ ] The `key` prop — why, what value to use
- [ ] Conditional rendering: `&&`, ternary, early return
- [ ] Empty-state UI
- **Exercise**: Build a static `<UserList>` rendering 5 users with badges for "admin"

### Class 33 · useState & Event Handling → `33-react-usestate-and-events.md`
- [ ] What "state" means in React
- [ ] `useState<T>` with a TS type argument
- [ ] Updating state correctly (immutability, functional updates)
- [ ] Event handlers: `onClick`, `onChange`, `onSubmit`
- [ ] Typing event objects (`React.ChangeEvent`, `React.FormEvent`)
- **Exercise**: Build a counter and a colour-mode toggle

### Class 34 · Controlled Forms in React → `34-react-controlled-forms.md`
- [ ] Controlled vs uncontrolled inputs
- [ ] Single source of truth: `value` + `onChange`
- [ ] Building a multi-field form with one state object
- [ ] Simple inline validation
- **Exercise**: Build a sign-up form (name, email, password, confirm) with inline errors

### Class 35 · useEffect & Side Effects → `35-react-useeffect.md`
- [ ] What "side effect" means
- [ ] `useEffect(fn, deps)` — the mental model
- [ ] Dependency arrays: `[]`, `[a, b]`, omitted
- [ ] Cleanup functions
- [ ] Fetching data with `useEffect` + Fetch (JSONPlaceholder)
- [ ] Why we will later prefer React Query
- **Exercise**: Fetch and render 10 posts from JSONPlaceholder with loading/error states

### Class 36 · Lifting State Up → `36-react-lifting-state.md`
- [ ] The "shared state" problem
- [ ] Moving state to the closest common ancestor
- [ ] Passing handlers down as props
- [ ] When NOT to lift (Context is coming)
- **Exercise**: Build a "filtered list" — parent owns the filter, child owns the items

### Class 37 · Custom Hooks → `37-react-custom-hooks.md`
- [ ] Why extract reusable logic
- [ ] The `use*` naming convention
- [ ] Building `useToggle`, `useLocalStorage`, `useDebounce`
- [ ] Typing custom hooks
- **Exercise**: Build `useFetch<T>(url)` and use it in two different components

### Class 38 · Context API for Global State → `38-react-context-api.md`
- [ ] The prop-drilling problem
- [ ] `createContext`, `<Provider>`, `useContext`
- [ ] A typed `ThemeContext` (light / dark)
- [ ] Why Context is not always the answer (re-render concerns)
- **Exercise**: Build a `ThemeProvider` and a `<ThemeToggle>` button consumed anywhere in the tree

---

## Phase 7 — Tailwind + shadcn/ui (4 classes, 4 hours)

### Class 39 · Tailwind CSS in a React Project → `39-tailwind-css-in-react.md`
- [ ] What utility-first CSS is
- [ ] Adding Tailwind to a Vite + React app
- [ ] Core utilities: spacing, sizing, colours, typography, layout
- [ ] Responsive prefixes (`sm:`, `md:`, `lg:`)
- [ ] State variants (`hover:`, `focus:`, `disabled:`)
- [ ] Composing readable Tailwind class strings
- **Exercise**: Rebuild the "card row" from CSS Class 12 using only Tailwind

### Class 40 · shadcn/ui — Setup & Core Components → `40-shadcn-ui-setup-and-basics.md`
- [ ] What shadcn/ui is (and isn't — it's not a npm package, it's components you own)
- [ ] Initialise: `npx shadcn@latest init` (Vite + React + TS)
- [ ] Adding components: `npx shadcn@latest add button input card badge separator`
- [ ] Building a simple "profile card" with shadcn primitives
- **Exercise**: Replace the hand-built card with shadcn `<Card>` + `<Button>` + `<Badge>`

### Class 41 · shadcn — Form, Dialog, Sonner Toasts → `41-shadcn-forms-dialogs-toasts.md`
- [ ] Add: `npx shadcn@latest add form dialog alert-dialog sonner label`
- [ ] The shadcn `<Form>` primitive (preview — full RHF integration in Class 44)
- [ ] `<Dialog>` for modal flows; `<AlertDialog>` for destructive confirm
- [ ] Sonner toasts (`<Toaster />` + `toast.success(...)`)
- **Exercise**: Build a "delete user" flow with confirm dialog + success toast

### Class 42 · Theming, Dark Mode & Customisation → `42-theming-dark-mode-customisation.md`
- [ ] CSS variables in shadcn (`--background`, `--foreground`, etc.)
- [ ] Adding a dark-mode toggle with the `dark` class strategy
- [ ] Customising the colour palette
- [ ] Tailwind config + `tailwind.config.ts` walkthrough
- **Exercise**: Build a dark/light theme toggle that persists via `useLocalStorage`

---

## Phase 8 — Forms (2 classes, 2 hours)

### Class 43 · React Hook Form Basics → `43-react-hook-form-basics.md`
- [ ] Why use a forms library (performance, less code, ergonomics)
- [ ] `useForm`, `register`, `handleSubmit`, `formState.errors`
- [ ] `defaultValues`, `reset`
- [ ] Submitting and disabling while submitting
- [ ] Wiring an RHF form to shadcn `<Input>` and `<Button>` (without `<Form>` for now)
- **Exercise**: Build a "contact us" form with RHF (no validation lib yet)

### Class 44 · Zod Validation + Integration with shadcn Form → `44-zod-validation-with-rhf.md`
- [ ] Why Zod (single source of truth for schema + types)
- [ ] Defining a schema: `z.string().min(...).email()`, `z.number()`, refinements
- [ ] `z.infer<typeof schema>` to derive TS types
- [ ] Connecting Zod with RHF using `zodResolver`
- [ ] Using shadcn `<Form>`, `<FormField>`, `<FormLabel>`, `<FormControl>`, `<FormMessage>`
- **Exercise**: Build a "create post" form (title, body, tags) with Zod + shadcn `<Form>`

---

## Phase 9 — Data (3 classes, 3 hours)

### Class 45 · Axios + React Query — `useQuery` → `45-axios-and-react-query-intro.md`
- [ ] Why Axios over Fetch (interceptors, base URL, typed responses)
- [ ] Configuring an Axios instance with `baseURL` (JSONPlaceholder)
- [ ] What React Query (TanStack Query) is — server state vs client state
- [ ] `QueryClient`, `QueryClientProvider`, ReactQueryDevtools
- [ ] `useQuery({ queryKey, queryFn })` to fetch a list and a detail
- [ ] `staleTime`, `gcTime` — sensible defaults
- **Exercise**: Build a `/posts` list page and a `/posts/:id` detail page (no router yet — single-page swap is fine)

### Class 46 · `useMutation`, Query Keys & Cache Invalidation → `46-mutations-and-cache-invalidation.md`
- [ ] `useMutation({ mutationFn, onSuccess, onError })`
- [ ] Optimistic UI updates (intro)
- [ ] Query key factories (`postsKeys.all`, `postsKeys.detail(id)`)
- [ ] `queryClient.invalidateQueries({ queryKey })`
- [ ] One hook per action: `usePosts`, `useCreatePost`, `useUpdatePost`, `useDeletePost`
- **Exercise**: Build full CRUD for posts against JSONPlaceholder (the writes are mocked server-side, that's fine)

### Class 47 · Loading, Error, Empty States → `47-loading-error-empty-states.md`
- [ ] The four UI states: idle / loading / error / empty / data
- [ ] Skeleton components with shadcn `<Skeleton>`
- [ ] Error UI with a "Retry" button
- [ ] Empty-state UI with a clear call-to-action
- [ ] Sonner toasts from mutation success/error
- [ ] Retries, refetch on focus
- **Exercise**: Polish the CRUD page from Class 46 with skeletons, error retry and toasts

---

## Phase 10 — Routing & Tables (2 classes, 2 hours)

### Class 48 · React Router v6 → `48-react-router-v6.md`
- [ ] Why client-side routing (no full page reloads)
- [ ] `BrowserRouter`, `Routes`, `Route`
- [ ] `Link`, `NavLink`, `useNavigate`
- [ ] URL parameters (`:id`) with `useParams`
- [ ] Nested routes with `<Outlet />`
- [ ] A `<Layout>` pattern with a navbar
- [ ] 404 catch-all
- **Exercise**: Add routes `/`, `/posts`, `/posts/:id`, `/users`, `*` to the data project

### Class 49 · TanStack Table — Pagination, Search, Filters → `49-tanstack-table-pagination-search-filters.md`
- [ ] When a real `<DataTable>` matters
- [ ] `@tanstack/react-table` setup
- [ ] Defining `ColumnDef`s (text, badge, date, row actions)
- [ ] Client-side pagination, sort
- [ ] Server-style pagination with `manualPagination` and `pageCount`
- [ ] URL-synced search and filter (`useSearchParams`)
- [ ] Debounced search input
- [ ] Row actions: edit dialog, delete with confirm
- **Exercise**: Replace the card list of users with an admin-style `<UsersTable>`

---

## Phase 11 — Capstone (2 classes, 2 hours)

### Class 50 · Capstone Part 1 — Users & Posts Dashboard → `50-capstone-users-posts-dashboard-part-1.md`
- [ ] Project brief: a small dashboard with **Users** and **Posts** modules using JSONPlaceholder
- [ ] Routes: `/`, `/login` (mock), `/users`, `/users/:id`, `/posts`, `/posts/:id`, `/posts/new`
- [ ] App shell: shadcn navbar + sidebar + theme toggle
- [ ] Users module: list, detail, CRUD (mocked)
- [ ] Posts module: list, detail, "Create post" form (RHF + Zod)
- [ ] Wire `useQuery` / `useMutation`, query keys factory, Sonner toasts
- **Exercise**: Reach a runnable state by end of class — anything unfinished becomes homework

### Class 51 · Capstone Part 2 — Polish & Vercel Deploy → `51-capstone-part-2-and-deployment.md`
- [ ] Add a `<UsersTable>` view using TanStack Table (URL-synced pagination + search)
- [ ] Loading skeletons, error retry buttons, empty states
- [ ] Form polish (disabled-while-submitting, reset on success)
- [ ] `npm run build` and a local production preview
- [ ] Push project to GitHub
- [ ] Connect the repo to Vercel and deploy
- [ ] Environment variables in Vercel (intro only — none required for JSONPlaceholder)
- [ ] What's next: real back-end, auth, tests, animations
- **Exercise**: Submit the live Vercel URL and the GitHub repo

---

## Quick Reference — Tech Stack Per Class

| Class | Key technology |
|------:|----------------|
| 1 | Node.js, VS Code, Git, GitHub, Vercel accounts |
| 2–7 | HTML5 |
| 8–15 | CSS3, Flexbox, Grid, media queries |
| 16–25 | JavaScript ES6+, DOM, Fetch |
| 26 | Node.js + npm |
| 27 | Git + GitHub |
| 28 | TypeScript |
| 29–38 | React 18 + Vite + TS |
| 39 | Tailwind CSS |
| 40–42 | shadcn/ui |
| 43–44 | React Hook Form + Zod |
| 45–47 | Axios + TanStack Query + Sonner + JSONPlaceholder |
| 48 | React Router v6 |
| 49 | TanStack Table |
| 50–51 | Everything combined + Vercel deployment |

---

## Estimated Timeline (3 classes / week)

| Phase | Classes | Weeks |
|-------|---------|-------|
| 0 — Setup | 1 | < 1 |
| 1 — HTML | 6 | 2 |
| 2 — CSS | 8 | ~3 |
| 3 — JavaScript | 10 | ~3.5 |
| 4 — Tooling | 2 | < 1 |
| 5 — TypeScript | 1 | < 1 |
| 6 — React Fundamentals | 10 | ~3.5 |
| 7 — UI | 4 | ~1.5 |
| 8 — Forms | 2 | < 1 |
| 9 — Data | 3 | 1 |
| 10 — Routing & Tables | 2 | < 1 |
| 11 — Capstone | 2 | < 1 |
| **Total** | **51 classes** | **~17 weeks (one semester)** |
