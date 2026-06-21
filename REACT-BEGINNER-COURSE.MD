# React Beginner Course — HTML, CSS, JavaScript & React

## Course Overview

This **51-class beginner course** takes BCA Semester 2 students from zero web-development knowledge to building and deploying a modern, production-style React application. Students start with the fundamentals — HTML, CSS, JavaScript — then move into modern tooling (Node.js, npm, Git, TypeScript) before learning React 18 with Vite, Tailwind CSS, shadcn/ui, React Hook Form, Zod, Axios, React Query, React Router, and TanStack Table. The course ends with a capstone **Users & Posts Dashboard**, deployed live to **Firebase Hosting**.

The course is **frontend-only**. Wherever data fetching is taught, students use public placeholder APIs (**JSONPlaceholder** and **DummyJSON**), so 100% of the focus stays on the React side. Students who want the backend (Express, MongoDB, authentication, payments) can continue with the MERN course afterwards.

**Level**: Absolute beginner — no prior coding experience required.
**Project**: A **Users & Posts Dashboard** with mock authentication, full CRUD, an admin-style data table, dark-mode theming and **Firebase Hosting** deployment.
**Schedule**: 3 classes per week, 1 hour per class (~17 weeks total — one semester).

---

## Instructor

| | |
|---|---|
| **Name** | Er. Prabin Karki |
| **Education** | Bachelor in Computer Science and Engineering |
| **Email** | prabinkarki643@gmail.com |
| **Portfolio** | [prabin-karki.com.np](https://prabin-karki.com.np) |

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Markup** | HTML5 | Page structure and semantics |
| **Styling** | CSS3 | Layout, typography, responsive design |
| **Styling (React phase)** | Tailwind CSS | Utility-first CSS framework |
| **Language (foundation)** | JavaScript (ES6+) | Logic, DOM, async |
| **Language (React phase)** | TypeScript | Type safety on top of JavaScript |
| **Framework** | React 18+ (via Vite) | Component-based UI development |
| **Build Tool** | Vite | Fast dev server and modern bundler |
| **UI Components** | shadcn/ui | Copy-and-own accessible component collection |
| **Form Handling** | React Hook Form + Zod | Performant forms with schema validation |
| **Data Fetching** | TanStack Query (React Query) | Server-state management and caching |
| **Routing** | React Router v6 | Client-side navigation |
| **HTTP Client** | Axios | API communication |
| **Tables** | TanStack Table (`@tanstack/react-table`) | Admin-style data tables with pagination, search, filters |
| **Toasts** | Sonner (via shadcn) | Success and error notifications |
| **Version Control** | Git + GitHub | Source control and code hosting |
| **Deployment** | Firebase Hosting (Google) | Production hosting (frontend) with free `.web.app` URL |
| **Placeholder APIs** | JSONPlaceholder, DummyJSON | Realistic API data for learning |

---

## Setup Requirements

Before Class 1, students must install the following software and create the required accounts. Class 1 verifies the setup together — but installing in advance saves time.

### Required Software

| Software | Version | Windows | Mac |
|----------|---------|---------|-----|
| **Node.js** (LTS) | 20+ | Download from [nodejs.org](https://nodejs.org) and run the installer | `brew install node` or download from [nodejs.org](https://nodejs.org) |
| **npm** | 10+ | Included with Node.js | Included with Node.js |
| **Git** | 2.40+ | Download from [git-scm.com](https://git-scm.com) — the installer also gives you **Git Bash**, the terminal Windows students use for the course | `brew install git` or Xcode Command Line Tools (`xcode-select --install`) |
| **GitHub CLI (`gh`)** | 2.40+ | `winget install GitHub.cli` (or `.msi` from [cli.github.com](https://cli.github.com)) | `brew install gh` |
| **VS Code** | Latest | Download from [code.visualstudio.com](https://code.visualstudio.com) | Download from [code.visualstudio.com](https://code.visualstudio.com) |
| **Modern browser** | Latest | Chrome, Edge, or Firefox | Chrome, Edge, Safari, or Firefox |

### Recommended VS Code Extensions

| Extension | Purpose |
|-----------|---------|
| ES7+ React/Redux/React-Native snippets | Quick React component snippets (`rafce`, `useState`, etc.) |
| Tailwind CSS IntelliSense | Autocomplete and previews for Tailwind classes |
| Prettier — Code formatter | Automatic code formatting on save |
| ESLint | Catches common JavaScript and React mistakes |
| Auto Rename Tag | Renames matching JSX/HTML tags automatically |
| Path Intellisense | Autocompletes file paths in imports |
| Live Server | One-click preview of HTML/CSS files (used in Phases 1–2) |
| GitLens | See who changed what, when, and why |
| Material Icon Theme | Friendlier file-tree icons (optional) |

### Accounts Needed

| Service | Purpose | URL |
|---------|---------|-----|
| **GitHub** | Code hosting and version control. After signing up, run `gh auth login` once in the terminal to authenticate the CLI. | [github.com](https://github.com) |
| **Firebase (Google)** | Free hosting for the capstone (`.web.app` URL with HTTPS) — sign in with your Google account. | [console.firebase.google.com](https://console.firebase.google.com) |

### Verify Installation

After installing, open a terminal (**Git Bash** on Windows, **Terminal** on macOS) and run these commands:

```bash
node --version      # Should show v20.x.x or higher
npm --version       # Should show 10.x.x or higher
git --version       # Should show 2.40.x or higher
gh --version        # Should show 2.x.x or higher
gh auth status      # Should say "Logged in as <your-username>"
code --version      # Should show a VS Code version
```

---

## Phase Breakdown

The 51 classes are grouped into **12 phases**. Every lesson file is sized for exactly one 60-minute class.

---

### Phase 0 — Setup (Class 1)

**Goal**: Get every student's machine working and verified before any coding starts.

| Class | Topic | Key Concepts |
|-------|-------|-------------|
| 01 | Prerequisites & Environment Setup | Node.js LTS, VS Code + extensions, Git first-time config (+ Git Bash on Windows), GitHub CLI install + `gh auth login`, GitHub & Firebase accounts, VS Code tour, browser DevTools, Live Server |

**Outcome**: A correctly set-up development environment for every student.

---

### Phase 1 — HTML Fundamentals (Classes 2–7)

**Goal**: Be able to write well-structured, semantic HTML5 pages and forms by hand.

| Class | Topic | Key Concepts |
|-------|-------|-------------|
| 02 | HTML Introduction & Document Structure | HTML5 boilerplate, `<!DOCTYPE>`, `<head>` vs `<body>`, meta tags, comments |
| 03 | HTML Text, Links, Images & Lists | Headings, paragraphs, bold/italic, anchor tags, images with `alt`, ordered/unordered lists |
| 04 | HTML Tables & Forms — Part 1 | Tables (data, not layout), `<form>`, `<label>` + `<input>`, basic input types |
| 05 | HTML Forms — Part 2 (Input Types & Validation) | Checkboxes, radios, selects, textareas, dates, files, `required`, `pattern`, `<fieldset>` |
| 06 | Semantic HTML & Accessibility Basics | `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>`, alt text, ARIA basics, keyboard navigation |
| 07 | HTML Practice — Build a Portfolio Page | Plan and build a 4-section portfolio (Hero, About, Projects, Contact) using everything from Classes 2–6 |

**Outcome**: A complete, valid, hand-written portfolio HTML page that will be styled in Phase 2.

---

### Phase 2 — CSS Fundamentals (Classes 8–15)

**Goal**: Be able to style any HTML page using modern CSS, Flexbox, Grid, and responsive design.

| Class | Topic | Key Concepts |
|-------|-------|-------------|
| 08 | CSS Introduction, Syntax & Selectors | External stylesheets, element / class / ID selectors, pseudo-classes, specificity |
| 09 | Box Model, Colours, Backgrounds, Borders | `box-sizing: border-box`, colour systems (hex, RGB, HSL), backgrounds, borders, shadows |
| 10 | Typography & CSS Units | `font-family`, Google Fonts, `px` vs `rem` vs `em` vs `%` vs `vh`/`vw`, typographic scale |
| 11 | Layout — Display, Position, Overflow, z-index | `display` values, `position` (relative/absolute/fixed/sticky), stacking context, centring techniques |
| 12 | Flexbox | Main vs cross axis, `justify-content`, `align-items`, `flex-wrap`, navbar and card-row patterns |
| 13 | CSS Grid | Grid vs Flexbox, `grid-template-columns`, `fr`, `repeat()`, `minmax()`, `auto-fit`, grid areas |
| 14 | Responsive Design & Media Queries | Mobile-first thinking, Tailwind-aligned breakpoints, `@media`, fluid typography with `clamp()` |
| 15 | CSS Practice — Style the Portfolio | Apply a colour palette, typographic scale, Flexbox nav, Grid project gallery, full responsive behaviour |

**Outcome**: A polished, fully responsive portfolio site built from scratch with vanilla HTML and CSS.

---

### Phase 3 — JavaScript Fundamentals (Classes 16–25)

**Goal**: Be able to write modern JavaScript, manipulate the DOM, handle events, and fetch data from real APIs.

| Class | Topic | Key Concepts |
|-------|-------|-------------|
| 16 | JS Introduction, Variables & Data Types | `<script defer>`, `console.log`, `let`/`const`, the 7 primitive types, `typeof` |
| 17 | Operators & Conditionals | Arithmetic / comparison / logical operators, `===` vs `==`, `if`/`else if`/`else`, `switch`, truthy and falsy |
| 18 | Loops | `for`, `while`, `do…while`, `for…of`, `for…in`, `break`, `continue` |
| 19 | Functions | Declarations vs expressions, arrow functions, default & rest parameters, scope, closures |
| 20 | Arrays & Array Methods | Indexing, mutating vs non-mutating methods, `map`, `filter`, `find`, `reduce`, chaining |
| 21 | Objects, Destructuring, Spread/Rest | Object literals, destructuring (with rename and defaults), spread/rest, references vs values |
| 22 | DOM Manipulation | The DOM tree, `querySelector`, `textContent`, attributes, creating and removing nodes, `classList` |
| 23 | Events & Event Handling | `addEventListener`, the event object, `preventDefault`, event delegation, building an interactive Todo list |
| 24 | ES6+ Features & Modules | Template literals, optional chaining, nullish coalescing, `import`/`export`, `<script type="module">` |
| 25 | Async — Promises, async/await, Fetch | Synchronous vs async, Promises, `try/catch`, the Fetch API against **JSONPlaceholder** |

**Outcome**: A modular, multi-file JavaScript Todo app that can fetch data from a public API.

---

### Phase 4 — Tooling (Classes 26–27)

**Goal**: Be comfortable in the terminal, with npm, and with Git.

| Class | Topic | Key Concepts |
|-------|-------|-------------|
| 26 | Node.js, npm & package.json | Node.js vs browser, `npm init`, `dependencies` vs `devDependencies`, semver (`^`, `~`), `npx`, `node_modules`, `.gitignore` |
| 27 | Git & GitHub Essentials | `init`, `status`, `add`, `commit`, branching (`switch`/`merge`), pushing to GitHub, pull requests (concept), recovering from mistakes |

**Outcome**: The portfolio page from Phase 1 is now hosted publicly on the student's own GitHub.

---

### Phase 5 — TypeScript (Class 28)

**Goal**: Understand TypeScript enough to use it confidently in the React phase.

| Class | Topic | Key Concepts |
|-------|-------|-------------|
| 28 | TypeScript Basics | Why TS, compile-time vs runtime errors, primitive annotations, arrays and tuples, `interface` vs `type`, union & intersection types, optional properties, narrowing, generics intro, utility types (`Partial`, `Pick`, `Omit`, `Record`) |

**Outcome**: A TypeScript mental model ready for React.

---

### Phase 6 — React Fundamentals (Classes 29–38)

**Goal**: Be able to build a non-trivial React 18 + TypeScript application using hooks and context.

| Class | Topic | Key Concepts |
|-------|-------|-------------|
| 29 | React Introduction & Vite + TypeScript Setup | What React is, why Vite, `npm create vite@latest`, project tour (`main.tsx`, `App.tsx`), `npm run dev` |
| 30 | JSX & Your First Component | JSX is HTML-in-TS, `{ expressions }`, `className`, function components, export/import, fragments |
| 31 | Components, Props & Composition | Typed props with `interface`, default values, the `children` prop, the container pattern |
| 32 | Lists, Keys & Conditional Rendering | `.map()` to render arrays, the `key` prop, `&&` and ternary patterns, empty-state UI |
| 33 | useState & Event Handling | `useState<T>`, immutable updates, functional updates, `onClick`/`onChange`/`onSubmit`, typing events |
| 34 | Controlled Forms in React | Controlled inputs, single source of truth, multi-field forms with one state object, inline validation |
| 35 | useEffect & Side Effects | The mental model, dependency arrays, cleanup, fetching with `useEffect` against JSONPlaceholder |
| 36 | Lifting State Up | Sharing state between siblings, passing handlers down, parent–child composition |
| 37 | Custom Hooks | The `use*` convention, building `useToggle`, `useLocalStorage<T>`, `useDebounce<T>` |
| 38 | Context API for Global State | `createContext`, `<Provider>`, `useContext`, a typed `ThemeContext` consumed across the tree |

**Outcome**: A small React + TypeScript app that fetches data, manages state with hooks and context, and is split into reusable components.

---

### Phase 7 — Modern UI: Tailwind + shadcn/ui (Classes 39–42)

**Goal**: Style React apps with Tailwind and assemble interfaces with shadcn/ui components.

| Class | Topic | Key Concepts |
|-------|-------|-------------|
| 39 | Tailwind CSS in a React Project | Installing Tailwind (v4) in Vite, core utility classes, responsive (`sm:`, `md:`, `lg:`) and state (`hover:`, `focus:`) variants, the `cn()` helper |
| 40 | shadcn/ui — Setup & Core Components | `npx shadcn@latest init`, adding components (Button, Input, Card, Badge, Separator, Label), the `@/components/ui/` convention |
| 41 | shadcn — Form, Dialog, Sonner Toasts | The `<Form>` primitive (preview), `<Dialog>` modal flow, `<AlertDialog>` destructive confirm, mounting `<Toaster />` and firing `toast.success/error` |
| 42 | Theming, Dark Mode & Customisation | CSS variables, the `dark` class strategy, building a `<ThemeProvider>` and `<ThemeToggle>`, customising the palette |

**Outcome**: Beautiful, accessible, themeable interfaces built quickly with shadcn primitives.

---

### Phase 8 — Forms with React Hook Form + Zod (Classes 43–44)

**Goal**: Build production-grade forms with the same patterns used in real React codebases.

| Class | Topic | Key Concepts |
|-------|-------|-------------|
| 43 | React Hook Form Basics | `useForm`, `register`, `handleSubmit`, `formState.errors`, `defaultValues`, `reset`, `isSubmitting`, wiring to shadcn inputs |
| 44 | Zod Validation + shadcn `<Form>` Integration | Defining Zod schemas, `z.infer<typeof schema>`, `zodResolver`, full `<Form>`/`<FormField>`/`<FormMessage>` integration |

**Outcome**: Typed, validated, accessible forms with inline error UI.

---

### Phase 9 — Data with Axios + React Query (Classes 45–47)

**Goal**: Fetch, cache, and mutate server data using the patterns used by real production teams.

| Class | Topic | Key Concepts |
|-------|-------|-------------|
| 45 | Axios + React Query — `useQuery` | Axios instance, why React Query, `<QueryClientProvider>`, ReactQueryDevtools, `useQuery({ queryKey, queryFn })`, `staleTime`/`gcTime` |
| 46 | `useMutation`, Query Keys & Cache Invalidation | `useMutation({ mutationFn, onSuccess, onError })`, query keys factory, hierarchical invalidation, one hook per action, optimistic update intro |
| 47 | Loading, Error, Empty States | The five UI states, shadcn `<Skeleton>`, error UI with retry, empty CTAs, Sonner toasts from mutations |

**Outcome**: A small CRUD app against JSONPlaceholder with proper loading, error, empty, and success states.

---

### Phase 10 — Routing & Tables (Classes 48–49)

**Goal**: Multi-page apps and admin-style data tables.

| Class | Topic | Key Concepts |
|-------|-------|-------------|
| 48 | React Router v6 | `<BrowserRouter>`, `<Routes>`, `<Route>`, `<Link>`/`<NavLink>`, URL parameters, `useNavigate`, nested routes with `<Outlet />`, 404 catch-all |
| 49 | TanStack Table — Pagination, Search, Filters | Generic `<DataTable<TData, TValue>>`, `ColumnDef<T>[]`, server-style pagination with `manualPagination`, URL-synced search via `useSearchParams`, debounced search (300 ms), row actions |

**Outcome**: A multi-page app with a real admin-style data table whose state survives a page refresh.

---

### Phase 11 — Capstone: Users & Posts Dashboard (Classes 50–51)

**Goal**: Build and deploy a small but production-style React application that uses everything from the course.

| Class | Topic | Key Concepts |
|-------|-------|-------------|
| 50 | Capstone Part 1 — Dashboard Build | Project brief, scaffold (Vite + React + TS + Tailwind + shadcn), Axios + React Query, routes wired, mock login (RHF + Zod), Posts module (list, detail, create), Sonner toasts |
| 51 | Capstone Part 2 — Polish & Firebase Deployment | `<UsersTable>` (TanStack Table), skeletons, error retry, empty states, form polish, `npm run build`, push to GitHub via `gh repo create`, `firebase login`, `firebase init hosting` (SPA), `firebase deploy --only hosting`, custom domain |

**Outcome**: A live application on a public Firebase Hosting URL (`.web.app`), in the student's own GitHub repository — ready for a CV or portfolio.

---

## Quick Reference

| Class | Key Concepts | Tools & Libraries |
|-------|-------------|-------------------|
| 01 | Environment setup, accounts, DevTools | Node.js, VS Code, Git (+ Git Bash), GitHub CLI (`gh`), GitHub, Firebase |
| 02–07 | HTML structure, semantic markup, forms, accessibility | HTML5, Live Server |
| 08–15 | Selectors, box model, Flexbox, Grid, responsive design | CSS3 |
| 16–25 | ES6+ syntax, DOM, events, modules, async, Fetch | JavaScript, JSONPlaceholder |
| 26 | Package management, scripts, `node_modules` | Node.js, npm |
| 27 | Version control, branching, publishing | Git, GitHub |
| 28 | Type annotations, interfaces, generics, utility types | TypeScript |
| 29–30 | Component model, JSX, Vite TS project setup | React, Vite, TypeScript |
| 31–32 | Typed props, lists, keys, conditional rendering | React |
| 33–34 | Local state, controlled forms, typed events | React (useState) |
| 35 | Side effects, `useEffect`, dependency arrays, cleanup | React (useEffect) |
| 36 | Lifting state up, parent–child composition | React |
| 37 | Reusable hooks, generics in hooks | React (custom hooks) |
| 38 | Context, providers, typed `useContext` | React (Context API) |
| 39 | Utility-first styling, responsive variants | Tailwind CSS |
| 40–41 | Component library, dialogs, toasts | shadcn/ui, Sonner |
| 42 | Theming, dark mode, CSS variables | Tailwind, shadcn |
| 43–44 | Form handling, schema validation, typed forms | React Hook Form, Zod, shadcn `<Form>` |
| 45 | HTTP client, server-state caching | Axios, TanStack Query |
| 46 | Mutations, cache invalidation, query keys factories | TanStack Query |
| 47 | Loading/error/empty UX, toasts, retries | shadcn `<Skeleton>`, Sonner |
| 48 | Client-side routing, nested layouts | React Router v6 |
| 49 | Headless table, URL state, debounced search, pagination | TanStack Table, React Router |
| 50 | App architecture, feature folders, mock auth | Everything above |
| 51 | Production build, GitHub push, Firebase Hosting deployment | Firebase CLI |

---

## What Students Will Build

### Vanilla Web Projects (Phases 1–3)

- A **complete, valid, semantic HTML page** (Phase 1, Class 7)
- A **fully responsive styled portfolio** with Flexbox, Grid, and media queries (Phase 2, Class 15)
- A **vanilla JavaScript Todo app** with DOM manipulation, events, and ES6 modules (Phase 3, Classes 22–24)
- A **JS page that fetches data** from JSONPlaceholder and renders it (Phase 3, Class 25)

### React Mini-Projects (Phases 6–10)

- A **typed counter** and **theme toggle** (Classes 33, 38)
- A **multi-field controlled sign-up form** with inline validation (Class 34)
- A **data-fetching page** with loading and error states (Class 35)
- A small **CRUD app** for posts using JSONPlaceholder (Classes 45–47)
- A **multi-page application** with nested routes and a 404 page (Class 48)
- An **admin-style users table** with URL-synced search and pagination (Class 49)

### Capstone — Users & Posts Dashboard (Phases 11)

A small but production-style admin dashboard featuring:

- **Mock authentication** with a login form (React Hook Form + Zod) and a fake JWT stored in `localStorage`
- **Protected routes** (must be logged in to see `/posts` and `/users`)
- **Posts module**: list, detail, and create-post views with React Hook Form + Zod and shadcn `<Form>`
- **Users module**: an admin-style `<DataTable>` (TanStack Table) with URL-synced search and pagination
- **Sonner toast notifications** on every successful or failed mutation
- **Dark mode toggle** that remembers the user's choice across sessions
- **Loading skeletons**, **error UI with retry**, and **empty states** everywhere
- **Production build** with `npm run build` and a local preview
- **Live deployment** on Firebase Hosting (a free `.web.app` URL), in the student's own GitHub repository

---

## Estimated Timeline

| Phase | Classes | Hours | Weeks |
|-------|---------|-------|-------|
| Phase 0 — Setup | 01 | 1 | Day 1 |
| Phase 1 — HTML | 02–07 | 6 | ~2 |
| Phase 2 — CSS | 08–15 | 8 | ~3 |
| Phase 3 — JavaScript | 16–25 | 10 | ~3.5 |
| Phase 4 — Tooling (Node, Git) | 26–27 | 2 | <1 |
| Phase 5 — TypeScript | 28 | 1 | <1 |
| Phase 6 — React Fundamentals | 29–38 | 10 | ~3.5 |
| Phase 7 — Tailwind + shadcn/ui | 39–42 | 4 | ~1.5 |
| Phase 8 — Forms (RHF + Zod) | 43–44 | 2 | <1 |
| Phase 9 — Data (Axios + React Query) | 45–47 | 3 | 1 |
| Phase 10 — Routing & Tables | 48–49 | 2 | <1 |
| Phase 11 — Capstone + Deployment | 50–51 | 2 | <1 |
| **Total** | **51 classes** | **51 hours** | **~17 weeks (one semester)** |

*Schedule: 3 classes per week, 1 hour per class. Each lesson file is sized for exactly **one** 60-minute class — there are no two-class lessons.*

---

## What This Course Does NOT Cover

To keep the scope focused and beginner-friendly, this course deliberately omits:

- **Back-end frameworks** (Express, NestJS, etc.) — out of scope
- **Databases** (MongoDB, PostgreSQL, etc.) — out of scope
- **Real authentication** against a server — only mocked patterns
- **Mobile development** (React Native)
- **Advanced testing** (Vitest, Playwright) — briefly mentioned only

Students who want a full-stack continuation should look at the **MERN course** — it picks up from a similar React starting point and adds Express, MongoDB, JWT auth, file uploads, and payment integration.

---

## Final Outcome

By the end of the 51 classes, every student will:

1. Be comfortable writing semantic HTML5 and modern, responsive CSS3.
2. Be able to write modern JavaScript and TypeScript and reason about asynchronous code.
3. Be able to build a React 18 + TypeScript application using hooks and context.
4. Use Tailwind + shadcn/ui to build accessible, themeable interfaces quickly.
5. Build production-grade forms with React Hook Form + Zod.
6. Fetch, cache, and mutate data with Axios + React Query.
7. Build admin-style data tables with TanStack Table.
8. Navigate multi-page apps with React Router v6.
9. Deploy a real, working application to Firebase Hosting from a GitHub repository.
10. Have a **public, live URL** they can put on their CV or portfolio.

> "If you can finish this course, you can apply for a junior front-end React role." — Er. Prabin Karki
