# React Beginner Course — Claude Project Instructions

## Project Identity
This is a **teaching/educational project** designed to take complete beginners (BCA, second semester) from zero to building a modern React application. The instructor is **Er. Prabin Karki**, a senior full-stack developer teaching at Triton College (Nepal) and a partner college.

This course is **front-end focused only** — there is no backend module. Wherever HTTP/API integration is taught, students use public placeholder APIs such as **JSONPlaceholder** (`https://jsonplaceholder.typicode.com`) and **DummyJSON** (`https://dummyjson.com`) so the focus stays on the React side.

## Project Purpose
Teach BCA second-semester students to build modern, production-style React applications, progressing from HTML/CSS/JS fundamentals to a complete React app using Vite, TypeScript, Tailwind CSS, shadcn/ui, React Hook Form, Zod, React Query, React Router, and TanStack Table. Students finish the course by building and deploying a **Users & Posts Dashboard** capstone project.

## Tech Stack
- **Markup**: HTML5
- **Styling**: CSS3 → Tailwind CSS
- **Language (foundation)**: JavaScript (ES6+)
- **Language (React phase)**: TypeScript
- **Build Tool**: Vite
- **Framework**: React 18+
- **UI Components**: shadcn/ui (installed via the `npx shadcn@latest init --template vite` flow)
- **Forms**: React Hook Form
- **Validation**: Zod
- **HTTP Client**: Axios
- **Server State**: TanStack Query (React Query)
- **Tables**: TanStack Table (`@tanstack/react-table`)
- **Routing**: React Router v6
- **Toasts**: Sonner (via shadcn)
- **Package Manager**: npm
- **Version Control**: Git + GitHub
- **Deployment**: Firebase Hosting
- **Placeholder APIs**: JSONPlaceholder, DummyJSON

## Project Structure
```
react-beginner-course/
├── CLAUDE.md              # This file — project instructions
├── README.md              # Public-facing course overview
├── SUMMARY.md             # Detailed curriculum & per-class checklist
├── .gitignore
├── assets/                # Course images, diagrams, screenshots
└── lessons/               # 51 individual class files
    ├── 01-prerequisites-and-setup.md
    ├── 02-html-introduction.md
    ├── 03-html-text-links-images.md
    ├── 04-html-tables-and-forms.md
    ├── 05-html-forms-input-types.md
    ├── 06-html-semantic-and-accessibility.md
    ├── 07-html-practice-portfolio-page.md
    ├── 08-css-introduction-and-selectors.md
    ├── 09-css-box-model-colours-backgrounds.md
    ├── 10-css-typography-and-units.md
    ├── 11-css-layout-display-position.md
    ├── 12-css-flexbox.md
    ├── 13-css-grid.md
    ├── 14-css-responsive-and-media-queries.md
    ├── 15-css-practice-style-portfolio.md
    ├── 16-js-introduction-variables-types.md
    ├── 17-js-operators-and-conditionals.md
    ├── 18-js-loops.md
    ├── 19-js-functions.md
    ├── 20-js-arrays-and-methods.md
    ├── 21-js-objects-and-destructuring.md
    ├── 22-js-dom-manipulation.md
    ├── 23-js-events.md
    ├── 24-js-es6-modules-and-features.md
    ├── 25-js-async-promises-fetch.md
    ├── 26-nodejs-npm-and-tooling.md
    ├── 27-git-and-github-basics.md
    ├── 28-typescript-basics.md
    ├── 29-react-introduction-and-vite.md
    ├── 30-jsx-and-first-component.md
    ├── 31-react-components-and-props.md
    ├── 32-react-lists-keys-conditional.md
    ├── 33-react-usestate-and-events.md
    ├── 34-react-controlled-forms.md
    ├── 35-react-useeffect.md
    ├── 36-react-lifting-state.md
    ├── 37-react-custom-hooks.md
    ├── 38-react-context-api.md
    ├── 39-tailwind-css-in-react.md
    ├── 40-shadcn-ui-setup-and-basics.md
    ├── 41-shadcn-forms-dialogs-toasts.md
    ├── 42-theming-dark-mode-customisation.md
    ├── 43-react-hook-form-basics.md
    ├── 44-zod-validation-with-rhf.md
    ├── 45-axios-and-react-query-intro.md
    ├── 46-mutations-and-cache-invalidation.md
    ├── 47-loading-error-empty-states.md
    ├── 48-react-router-v6.md
    ├── 49-tanstack-table-pagination-search-filters.md
    ├── 50-capstone-users-posts-dashboard-part-1.md
    └── 51-capstone-part-2-and-deployment.md
```

## Schedule
- **Total classes**: 51
- **Frequency**: 3 classes per week
- **Total duration**: ~17 weeks (one semester)
- **Class length**: 60 minutes (each lesson file is sized for exactly one class)

## Teaching Guidelines
- **Audience**: BCA second-semester students. Assume basic computer literacy but **no prior coding experience**.
- **Language**: UK English. Clear, simple sentences. Avoid jargon unless explained immediately with an analogy.
- **Approach**: Theory first (with analogies) → hands-on code example → step-by-step walkthrough → practice exercises.
- **Pace**: Each class builds on the previous — never skip fundamentals. Re-establish prerequisites where helpful.
- **Code examples**: Every concept must include a complete, runnable code snippet. Show expected output where possible.
- **Exercises**: Each class ends with **Practice Exercises** that a student can do in 15–30 minutes.

## Rules for Content Creation
1. Keep explanations simple and beginner-friendly — students are seeing this for the first time.
2. Use real-world analogies (HTML = skeleton, CSS = clothes, JS = behaviour; React component = LEGO brick, etc.).
3. Every code block must be complete and runnable as-is.
4. Include expected output or behaviour for each example.
5. Each class should fit in 30–60 minutes of teaching.
6. Begin every class with **What You Will Learn** (3–6 bullets).
7. End every class with **Practice Exercises** (2–4 tasks) and a **Key Takeaways** section.
8. Use **UK English** spelling (colour, organise, behaviour, optimise, customise, etc.).
9. Make **no assumptions** about prior knowledge — explain everything when it first appears.
10. Build incrementally — each class adds to previous knowledge.
11. Prefer TypeScript for all React lessons. Use `.tsx` (not `.jsx`).
12. Use the **shadcn preset** flow for React projects: `npx shadcn@latest init --template vite` (or the documented current flow).
13. For API examples, always use **JSONPlaceholder** or **DummyJSON** — never invent fake URLs.
14. No backend code. If a topic naturally requires a server (auth, file uploads, etc.), explain it conceptually and use the placeholder APIs instead.

## Voice and Style
- Friendly, encouraging, never condescending.
- Short paragraphs. Bullet points where helpful.
- Tables for comparison or quick reference.
- Code first, words second when a concept is best shown.
- Always invite curiosity ("Try changing this and see what happens.").

## What This Course Does NOT Cover
- Backend frameworks (Express, Nest, etc.) — out of scope.
- Databases (MongoDB, PostgreSQL, etc.) — out of scope.
- Authentication implementation against a real server — only mocked patterns.
- Mobile development (React Native).
- Advanced testing (Vitest, Playwright) — briefly mentioned only.

Students who want a full-stack continuation should be pointed to the **MERN course** at `/triton-college/react-node-course/`.

## Reference Course
A closely related MERN course exists at `../react-node-course/` and may be consulted for tone, depth, and code style. This new course is **beginner-only and frontend-only**, so do not copy backend material from there.
