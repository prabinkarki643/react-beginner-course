# React Beginner Course — HTML, CSS, JavaScript & React

A complete **51-class** beginner course taking BCA second-semester students from zero web development knowledge to building and deploying a modern React application.

## Instructor

**Er. Prabin Karki**
Bachelor in Computer Science and Engineering
[prabin-karki.com.np](https://prabin-karki.com.np) · prabinkarki643@gmail.com

## Who This Course Is For

- **BCA Semester 2 students** (or equivalent level)
- Complete beginners to web development
- Students who can use a computer, browse the web, and install software, but have **never written code before**

## What You Will Learn

By the end of this course, every student will be able to:

1. Write semantic HTML5 and modern, responsive CSS3
2. Write JavaScript including ES6+ features, the DOM, events, and async code
3. Use Git and GitHub for version control
4. Build a modern React application with **Vite + TypeScript**
5. Style it beautifully with **Tailwind CSS** and **shadcn/ui**
6. Build complex forms with **React Hook Form + Zod**
7. Fetch and mutate data with **Axios + React Query (TanStack Query)**
8. Display data in admin-style tables with **TanStack Table**
9. Navigate multi-page apps with **React Router v6**
10. Deploy the finished app live on **Vercel**

## The Capstone Project

In the final two classes, students build and deploy a **Users & Posts Dashboard** — a small but production-style app that uses every tool taught in the course. It pulls real data from the **JSONPlaceholder** and **DummyJSON** public APIs and is deployed live to Vercel.

## Course Structure

| Phase | Classes | Topic | Weeks |
|-------|---------|-------|-------|
| 0 — Setup | 1 | Prerequisites & environment | Day 1 |
| 1 — HTML | 2–7 | HTML fundamentals + portfolio page | ~2 |
| 2 — CSS | 8–15 | CSS fundamentals + Flexbox + Grid + responsive | ~3 |
| 3 — JavaScript | 16–25 | JS fundamentals + DOM + async | ~3 |
| 4 — Tooling | 26–27 | Node.js, npm, Git, GitHub | ~1 |
| 5 — TypeScript | 28 | TypeScript basics for React | <1 |
| 6 — React | 29–38 | React fundamentals (Vite, hooks, context) | ~3 |
| 7 — UI | 39–42 | Tailwind + shadcn/ui + theming | ~1.5 |
| 8 — Forms | 43–44 | React Hook Form + Zod | <1 |
| 9 — Data | 45–47 | Axios + React Query | ~1 |
| 10 — Routing & Tables | 48–49 | React Router + TanStack Table | <1 |
| 11 — Capstone | 50–51 | Users & Posts dashboard + Vercel deploy | <1 |
| **Total** | **51 classes** | | **~17 weeks** |

**Schedule**: 3 classes per week, 60 minutes per class. Each lesson file is sized for exactly **one** class.

## Tech Stack

| Layer | Technologies |
|-------|--------------|
| **Markup** | HTML5 |
| **Styling** | CSS3, Tailwind CSS |
| **Language** | JavaScript (ES6+) → TypeScript |
| **Build Tool** | Vite |
| **Framework** | React 18+ |
| **UI** | shadcn/ui |
| **Forms** | React Hook Form + Zod |
| **Data** | Axios + TanStack Query |
| **Tables** | TanStack Table |
| **Routing** | React Router v6 |
| **Toasts** | Sonner (via shadcn) |
| **Version Control** | Git + GitHub |
| **Deployment** | Vercel |
| **APIs (teaching)** | JSONPlaceholder, DummyJSON |

## Prerequisites — Before Class 1

Every student must arrive at Class 1 with:

### Software installed
- [ ] **Node.js LTS** (latest LTS) — [nodejs.org](https://nodejs.org)
- [ ] **VS Code** — [code.visualstudio.com](https://code.visualstudio.com)
- [ ] **Git** — [git-scm.com](https://git-scm.com)
- [ ] A modern browser — Chrome, Edge, or Firefox

### Accounts created
- [ ] **GitHub** — [github.com](https://github.com)
- [ ] **Vercel** — [vercel.com](https://vercel.com) (sign in with GitHub)

### VS Code extensions
- [ ] **ES7+ React/Redux/React-Native snippets** (`dsznajder.es7-react-js-snippets`)
- [ ] **Tailwind CSS IntelliSense** (`bradlc.vscode-tailwindcss`)
- [ ] **Prettier — Code formatter** (`esbenp.prettier-vscode`)
- [ ] **ESLint** (`dbaeumer.vscode-eslint`)
- [ ] **Auto Rename Tag** (`formulahendry.auto-rename-tag`)
- [ ] **Path Intellisense** (`christian-kohler.path-intellisense`)
- [ ] **Live Server** (`ritwickdey.LiveServer`) — for HTML/CSS classes
- [ ] **GitLens** (`eamodio.gitlens`)
- [ ] **Material Icon Theme** (`PKief.material-icon-theme`) — optional but recommended

Full step-by-step setup instructions are in [Class 1](lessons/01-prerequisites-and-setup.md).

## Repository Structure

```
react-beginner-course/
├── README.md                  # This file — public-facing overview
├── SUMMARY.md                 # Detailed curriculum + per-class checklists
├── CLAUDE.md                  # Project instructions for Claude Code
├── .gitignore
├── assets/                    # Images and diagrams
└── lessons/                   # 50 class files (Markdown)
    ├── 01-prerequisites-and-setup.md
    ├── 02-html-introduction.md
    ├── ... (47 more) ...
    └── 50-capstone-part-2-and-deployment.md
```

## How To Use This Course

### Students
1. Complete the **Prerequisites** above (install all software, create accounts).
2. Read [Class 1](lessons/01-prerequisites-and-setup.md) on Day 1 to verify your setup.
3. Read one lesson per class. Type out every code example yourself — do not copy-paste.
4. Complete the **Practice Exercises** at the end of each class.
5. Ask questions early and often. There is no such thing as a silly question.

### Instructors
1. Read [SUMMARY.md](SUMMARY.md) for the full curriculum, checklists and timing.
2. Read [CLAUDE.md](CLAUDE.md) for the project's tone, style and rules.
3. Each lesson file in `lessons/` is the **single source of truth** for that class — teach from it directly.
4. Use the **Practice Exercises** at the end of each lesson as in-class or homework tasks.

## Related Courses

- **Full-Stack / MERN continuation**: `../react-node-course/` — for students who want to learn the backend (Express + MongoDB + Mongoose), JWT authentication, file uploads, and payment integration after completing this course.

## Licence

Course content © Er. Prabin Karki. Free to use for educational purposes with attribution.
