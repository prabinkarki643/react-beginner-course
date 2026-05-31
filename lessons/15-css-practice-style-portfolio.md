# Class 15: CSS Practice — Style the Portfolio Page

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Classes 7–14 (the portfolio HTML, plus everything in the CSS phase)

## What You Will Learn
- How to plan a small design system **before** typing CSS — palette, scale, spacing
- How to combine **Flexbox** (navbar, hero), **Grid** (projects), and **media queries** (responsiveness) in one project
- How to add **polish**: hover transitions, shadows, focus states
- A complete, paste-ready `style.css` you can drop next to your portfolio HTML

This is your first end-to-end "build a real thing" class. Everything you learned in Classes 8–14 lands in one file.

---

## 15.1 The plan — design before you type

Most beginner CSS becomes a mess because the styling is improvised. We will avoid that by deciding three things first.

### 1. Colour palette

Pick a small, intentional set — no more than five colours plus shades of grey.

| Role | Variable | Value | Why |
|------|----------|-------|-----|
| Brand primary | `--colour-primary` | `hsl(210, 80%, 50%)` | Steel blue — friendly, professional |
| Brand primary (dark) | `--colour-primary-dark` | `hsl(210, 80%, 40%)` | Hover state |
| Accent | `--colour-accent` | `hsl(28, 90%, 55%)` | Warm orange for highlights |
| Text | `--colour-text` | `#1f2937` | Dark slate, easier than pure black |
| Text muted | `--colour-text-muted` | `#6b7280` | For meta info, captions |
| Background | `--colour-bg` | `#f9fafb` | Page background |
| Surface | `--colour-surface` | `#ffffff` | Cards, the navbar |
| Border | `--colour-border` | `#e5e7eb` | Subtle dividers |

### 2. Typographic scale

Six sizes is plenty:

| Token | Value | Use |
|-------|-------|-----|
| `--text-sm` | `0.875rem` | Captions, meta |
| `--text-base` | `1rem` | Body |
| `--text-lg` | `1.25rem` | Lead paragraph |
| `--text-xl` | `1.5rem` | Subheadings |
| `--text-2xl` | `2rem` | Section headings |
| `--text-3xl` | `clamp(2rem, 5vw + 0.5rem, 3.5rem)` | Hero title (fluid) |

### 3. Spacing scale

Stick to multiples of `4px` (or `0.25rem`). Tailwind, Material and every modern design system does this.

| Token | Value | Approx use |
|-------|-------|-----------|
| `--space-1` | `0.25rem` (4px) | Tight gaps |
| `--space-2` | `0.5rem` (8px) | Small gaps |
| `--space-4` | `1rem` (16px) | Standard padding |
| `--space-6` | `1.5rem` (24px) | Card padding |
| `--space-8` | `2rem` (32px) | Section gaps |
| `--space-16` | `4rem` (64px) | Big vertical rhythm |

Five minutes spent picking these tokens will save you fifty minutes of "does this look right?" later.

---

## 15.2 The HTML we are styling

If your `portfolio.html` from Class 7 looks slightly different — adapt the class names. The structure below is what we will assume. Save this as `portfolio.html` if you don't have one yet.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Aarav Sharma — Portfolio</title>

    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link
      href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap"
      rel="stylesheet"
    />

    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <header class="site-header">
      <a class="site-header__brand" href="#top">Aarav Sharma</a>
      <nav class="site-header__nav">
        <a href="#about">About</a>
        <a href="#projects">Projects</a>
        <a href="#contact">Contact</a>
      </nav>
    </header>

    <main>
      <section id="top" class="hero">
        <p class="hero__eyebrow">Frontend developer</p>
        <h1 class="hero__title">Hi, I'm Aarav. I build clean, friendly websites.</h1>
        <p class="hero__text">
          Currently a BCA student at Triton College, learning React and
          modern web development.
        </p>
        <a href="#contact" class="button">Get in touch</a>
      </section>

      <section id="about" class="section">
        <h2 class="section__title">About me</h2>
        <p>
          I started learning HTML and CSS a few months ago, and I'm now
          comfortable building small responsive sites. I enjoy clear
          design, well-structured code, and a good cup of milk tea.
        </p>
        <ul class="skills">
          <li>HTML</li>
          <li>CSS</li>
          <li>JavaScript</li>
          <li>Git</li>
        </ul>
      </section>

      <section id="projects" class="section">
        <h2 class="section__title">Projects</h2>
        <div class="projects">
          <article class="project">
            <div class="project__thumb"></div>
            <h3 class="project__title">Marks calculator</h3>
            <p class="project__text">A small HTML form that computes percentage and grade.</p>
            <a href="#" class="project__link">View →</a>
          </article>

          <article class="project">
            <div class="project__thumb"></div>
            <h3 class="project__title">Recipe page</h3>
            <p class="project__text">A semantic recipe page with images and a printable layout.</p>
            <a href="#" class="project__link">View →</a>
          </article>

          <article class="project">
            <div class="project__thumb"></div>
            <h3 class="project__title">To-do list</h3>
            <p class="project__text">A static to-do list — JavaScript coming in Phase 3.</p>
            <a href="#" class="project__link">View →</a>
          </article>

          <article class="project">
            <div class="project__thumb"></div>
            <h3 class="project__title">Photo gallery</h3>
            <p class="project__text">A responsive grid gallery built with CSS Grid.</p>
            <a href="#" class="project__link">View →</a>
          </article>
        </div>
      </section>

      <section id="contact" class="section">
        <h2 class="section__title">Contact</h2>
        <form class="contact-form">
          <label>
            Name
            <input type="text" name="name" required />
          </label>
          <label>
            Email
            <input type="email" name="email" required />
          </label>
          <label>
            Message
            <textarea name="message" rows="4" required></textarea>
          </label>
          <button type="submit" class="button">Send message</button>
        </form>
      </section>
    </main>

    <footer class="site-footer">
      <p>© 2026 Aarav Sharma. Built with HTML and CSS.</p>
    </footer>
  </body>
</html>
```

That's the skeleton. Now to dress it.

---

## 15.3 Step 1 — reset and tokens

Start the stylesheet with the global reset and the design tokens (variables) we agreed on.

```css
/* ============================================================
   1. Reset & tokens
   ============================================================ */

*, *::before, *::after {
  box-sizing: border-box;
}

:root {
  /* Colour */
  --colour-primary: hsl(210, 80%, 50%);
  --colour-primary-dark: hsl(210, 80%, 40%);
  --colour-accent: hsl(28, 90%, 55%);

  --colour-text: #1f2937;
  --colour-text-muted: #6b7280;

  --colour-bg: #f9fafb;
  --colour-surface: #ffffff;
  --colour-border: #e5e7eb;

  /* Typography */
  --font-body: 'Inter', system-ui, Arial, sans-serif;

  --text-sm:  0.875rem;
  --text-base: 1rem;
  --text-lg:  1.25rem;
  --text-xl:  1.5rem;
  --text-2xl: 2rem;
  --text-3xl: clamp(2rem, 5vw + 0.5rem, 3.5rem);

  --leading-tight: 1.2;
  --leading-normal: 1.6;

  /* Spacing */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-4: 1rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --space-12: 3rem;
  --space-16: 4rem;

  /* Other */
  --radius: 12px;
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.08);

  --max-width: 1100px;
}
```

Already, before adding any visible styles, every future rule will use these tokens. Want to change the primary colour later? One line.

---

## 15.4 Step 2 — base typography and layout

```css
/* ============================================================
   2. Base
   ============================================================ */

body {
  margin: 0;
  font-family: var(--font-body);
  font-size: var(--text-base);
  line-height: var(--leading-normal);
  color: var(--colour-text);
  background-color: var(--colour-bg);
}

img, video { display: block; max-width: 100%; }

a {
  color: var(--colour-primary);
  text-decoration: none;
}

a:hover {
  color: var(--colour-primary-dark);
  text-decoration: underline;
}
```

---

## 15.5 Step 3 — the navbar (Flexbox)

A simple two-block layout — brand on the left, links pushed to the right.

```css
/* ============================================================
   3. Site header
   ============================================================ */

.site-header {
  position: sticky;
  top: 0;
  z-index: 10;

  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-4);

  padding: var(--space-4) var(--space-6);
  background: var(--colour-surface);
  border-bottom: 1px solid var(--colour-border);
}

.site-header__brand {
  font-weight: 700;
  font-size: var(--text-lg);
  color: var(--colour-text);
}

.site-header__nav {
  display: flex;
  gap: var(--space-6);
}

.site-header__nav a {
  color: var(--colour-text);
  font-weight: 500;
}

.site-header__nav a:hover {
  color: var(--colour-primary);
  text-decoration: none;
}
```

The navbar **sticks** to the top as you scroll. `justify-content: space-between` pushes the nav to the far right without needing a spacer.

---

## 15.6 Step 4 — the hero

A wide, centred hero with a fluid title and a primary button.

```css
/* ============================================================
   4. Hero
   ============================================================ */

.hero {
  max-width: var(--max-width);
  margin: 0 auto;
  padding: var(--space-16) var(--space-6);
  text-align: center;
}

.hero__eyebrow {
  margin: 0 0 var(--space-2);
  font-size: var(--text-sm);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--colour-primary);
}

.hero__title {
  margin: 0 0 var(--space-4);
  font-size: var(--text-3xl);
  line-height: var(--leading-tight);
  letter-spacing: -0.5px;
}

.hero__text {
  max-width: 640px;
  margin: 0 auto var(--space-8);
  font-size: var(--text-lg);
  color: var(--colour-text-muted);
}
```

### The button — a reusable component

```css
.button {
  display: inline-block;
  padding: var(--space-4) var(--space-8);

  background: var(--colour-primary);
  color: #ffffff;
  font-weight: 600;
  border-radius: 999px;     /* pill shape */
  border: none;
  cursor: pointer;

  text-decoration: none;
  box-shadow: 0 4px 12px rgba(74, 144, 217, 0.35);
  transition: background-color 0.2s ease, transform 0.1s ease;
}

.button:hover {
  background: var(--colour-primary-dark);
  text-decoration: none;
  color: #ffffff;
}

.button:active {
  transform: translateY(1px);
}

.button:focus-visible {
  outline: 3px solid var(--colour-accent);
  outline-offset: 2px;
}
```

`transition` makes the colour change feel smooth. `:focus-visible` shows a focus ring **only for keyboard users** — great for accessibility without annoying mouse users with rings on every click.

---

## 15.7 Step 5 — sections and the about list

```css
/* ============================================================
   5. Sections
   ============================================================ */

.section {
  max-width: var(--max-width);
  margin: 0 auto;
  padding: var(--space-8) var(--space-6);
}

.section__title {
  margin: 0 0 var(--space-6);
  font-size: var(--text-2xl);
  line-height: var(--leading-tight);
}

.skills {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-2);
  list-style: none;
  padding: 0;
  margin: var(--space-4) 0 0;
}

.skills li {
  padding: var(--space-1) var(--space-4);
  background: var(--colour-surface);
  border: 1px solid var(--colour-border);
  border-radius: 999px;
  font-size: var(--text-sm);
  color: var(--colour-text);
}
```

The skills become a row of soft pill tags — Flexbox with `gap`, no manual margins required.

---

## 15.8 Step 6 — the projects grid (CSS Grid)

The signature responsive-without-media-queries pattern from Class 13.

```css
/* ============================================================
   6. Projects
   ============================================================ */

.projects {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: var(--space-6);
}

.project {
  background: var(--colour-surface);
  border: 1px solid var(--colour-border);
  border-radius: var(--radius);
  overflow: hidden;
  box-shadow: var(--shadow-sm);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.project:hover {
  transform: translateY(-4px);
  box-shadow: var(--shadow-md);
}

.project__thumb {
  height: 140px;
  background: linear-gradient(135deg, var(--colour-primary), var(--colour-accent));
}

.project__title {
  margin: var(--space-4) var(--space-4) var(--space-2);
  font-size: var(--text-lg);
}

.project__text {
  margin: 0 var(--space-4) var(--space-4);
  color: var(--colour-text-muted);
}

.project__link {
  display: inline-block;
  margin: 0 var(--space-4) var(--space-4);
  font-weight: 600;
}
```

On hover, the card lifts and its shadow deepens — a small touch that makes the whole page feel alive.

---

## 15.9 Step 7 — the contact form

A vertical form using Flexbox for the labels.

```css
/* ============================================================
   7. Contact form
   ============================================================ */

.contact-form {
  display: flex;
  flex-direction: column;
  gap: var(--space-4);
  max-width: 560px;
}

.contact-form label {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  font-weight: 600;
}

.contact-form input,
.contact-form textarea {
  padding: var(--space-4);
  border: 1px solid var(--colour-border);
  border-radius: 8px;
  font: inherit;
  background: var(--colour-surface);
  transition: border-color 0.15s ease, box-shadow 0.15s ease;
}

.contact-form input:focus,
.contact-form textarea:focus {
  outline: none;
  border-color: var(--colour-primary);
  box-shadow: 0 0 0 3px rgba(74, 144, 217, 0.2);
}

.contact-form .button {
  align-self: flex-start;     /* don't stretch full width */
}
```

The focus state is the accessibility win — a clear blue ring instead of the default browser outline.

---

## 15.10 Step 8 — the footer and media queries

```css
/* ============================================================
   8. Footer
   ============================================================ */

.site-footer {
  border-top: 1px solid var(--colour-border);
  padding: var(--space-8) var(--space-6);
  text-align: center;
  color: var(--colour-text-muted);
  font-size: var(--text-sm);
  background: var(--colour-surface);
}

/* ============================================================
   9. Responsive tweaks
   ============================================================ */

/* Hide the nav links on the smallest screens — we'll keep just the brand */
@media (max-width: 480px) {
  .site-header__nav { display: none; }
}

/* More breathing room on tablets and up */
@media (min-width: 768px) {
  .hero    { padding: var(--space-16) var(--space-8); }
  .section { padding: var(--space-12) var(--space-8); }
}

/* Big screens — give the page wider gutters */
@media (min-width: 1280px) {
  .site-header { padding-left: var(--space-12); padding-right: var(--space-12); }
}
```

Because so much of the layout is built with Grid `auto-fit` and Flexbox `wrap`, we needed very few media queries. That's the goal of mobile-first design.

---

## 15.11 The complete `style.css` — paste this in

Here is everything from sections 15.3–15.10 combined into one file. Save it as `style.css` next to your `portfolio.html`.

```css
/* ============================================================
   Portfolio — style.css
   Built for Class 15 of the React Beginner Course
   ============================================================ */

/* 1. Reset & tokens
   ---------------------------------------------------------- */
*, *::before, *::after {
  box-sizing: border-box;
}

:root {
  --colour-primary: hsl(210, 80%, 50%);
  --colour-primary-dark: hsl(210, 80%, 40%);
  --colour-accent: hsl(28, 90%, 55%);

  --colour-text: #1f2937;
  --colour-text-muted: #6b7280;

  --colour-bg: #f9fafb;
  --colour-surface: #ffffff;
  --colour-border: #e5e7eb;

  --font-body: 'Inter', system-ui, Arial, sans-serif;

  --text-sm:  0.875rem;
  --text-base: 1rem;
  --text-lg:  1.25rem;
  --text-xl:  1.5rem;
  --text-2xl: 2rem;
  --text-3xl: clamp(2rem, 5vw + 0.5rem, 3.5rem);

  --leading-tight: 1.2;
  --leading-normal: 1.6;

  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-4: 1rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --space-12: 3rem;
  --space-16: 4rem;

  --radius: 12px;
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.08);

  --max-width: 1100px;
}

/* 2. Base
   ---------------------------------------------------------- */
body {
  margin: 0;
  font-family: var(--font-body);
  font-size: var(--text-base);
  line-height: var(--leading-normal);
  color: var(--colour-text);
  background-color: var(--colour-bg);
}

img, video { display: block; max-width: 100%; }

a {
  color: var(--colour-primary);
  text-decoration: none;
}
a:hover {
  color: var(--colour-primary-dark);
  text-decoration: underline;
}

/* 3. Site header (navbar)
   ---------------------------------------------------------- */
.site-header {
  position: sticky;
  top: 0;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-4);
  padding: var(--space-4) var(--space-6);
  background: var(--colour-surface);
  border-bottom: 1px solid var(--colour-border);
}

.site-header__brand {
  font-weight: 700;
  font-size: var(--text-lg);
  color: var(--colour-text);
}

.site-header__nav {
  display: flex;
  gap: var(--space-6);
}

.site-header__nav a {
  color: var(--colour-text);
  font-weight: 500;
}

.site-header__nav a:hover {
  color: var(--colour-primary);
  text-decoration: none;
}

/* 4. Hero
   ---------------------------------------------------------- */
.hero {
  max-width: var(--max-width);
  margin: 0 auto;
  padding: var(--space-16) var(--space-6);
  text-align: center;
}

.hero__eyebrow {
  margin: 0 0 var(--space-2);
  font-size: var(--text-sm);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--colour-primary);
}

.hero__title {
  margin: 0 0 var(--space-4);
  font-size: var(--text-3xl);
  line-height: var(--leading-tight);
  letter-spacing: -0.5px;
}

.hero__text {
  max-width: 640px;
  margin: 0 auto var(--space-8);
  font-size: var(--text-lg);
  color: var(--colour-text-muted);
}

/* Buttons (reusable) */
.button {
  display: inline-block;
  padding: var(--space-4) var(--space-8);
  background: var(--colour-primary);
  color: #ffffff;
  font-weight: 600;
  border-radius: 999px;
  border: none;
  cursor: pointer;
  text-decoration: none;
  box-shadow: 0 4px 12px rgba(74, 144, 217, 0.35);
  transition: background-color 0.2s ease, transform 0.1s ease;
}
.button:hover {
  background: var(--colour-primary-dark);
  color: #ffffff;
  text-decoration: none;
}
.button:active { transform: translateY(1px); }
.button:focus-visible {
  outline: 3px solid var(--colour-accent);
  outline-offset: 2px;
}

/* 5. Sections
   ---------------------------------------------------------- */
.section {
  max-width: var(--max-width);
  margin: 0 auto;
  padding: var(--space-8) var(--space-6);
}

.section__title {
  margin: 0 0 var(--space-6);
  font-size: var(--text-2xl);
  line-height: var(--leading-tight);
}

.skills {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-2);
  list-style: none;
  padding: 0;
  margin: var(--space-4) 0 0;
}
.skills li {
  padding: var(--space-1) var(--space-4);
  background: var(--colour-surface);
  border: 1px solid var(--colour-border);
  border-radius: 999px;
  font-size: var(--text-sm);
  color: var(--colour-text);
}

/* 6. Projects (Grid)
   ---------------------------------------------------------- */
.projects {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: var(--space-6);
}

.project {
  background: var(--colour-surface);
  border: 1px solid var(--colour-border);
  border-radius: var(--radius);
  overflow: hidden;
  box-shadow: var(--shadow-sm);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.project:hover {
  transform: translateY(-4px);
  box-shadow: var(--shadow-md);
}

.project__thumb {
  height: 140px;
  background: linear-gradient(135deg, var(--colour-primary), var(--colour-accent));
}

.project__title { margin: var(--space-4) var(--space-4) var(--space-2); font-size: var(--text-lg); }
.project__text  { margin: 0 var(--space-4) var(--space-4); color: var(--colour-text-muted); }
.project__link  { display: inline-block; margin: 0 var(--space-4) var(--space-4); font-weight: 600; }

/* 7. Contact form
   ---------------------------------------------------------- */
.contact-form {
  display: flex;
  flex-direction: column;
  gap: var(--space-4);
  max-width: 560px;
}

.contact-form label {
  display: flex;
  flex-direction: column;
  gap: var(--space-2);
  font-weight: 600;
}

.contact-form input,
.contact-form textarea {
  padding: var(--space-4);
  border: 1px solid var(--colour-border);
  border-radius: 8px;
  font: inherit;
  background: var(--colour-surface);
  transition: border-color 0.15s ease, box-shadow 0.15s ease;
}

.contact-form input:focus,
.contact-form textarea:focus {
  outline: none;
  border-color: var(--colour-primary);
  box-shadow: 0 0 0 3px rgba(74, 144, 217, 0.2);
}

.contact-form .button { align-self: flex-start; }

/* 8. Footer
   ---------------------------------------------------------- */
.site-footer {
  border-top: 1px solid var(--colour-border);
  padding: var(--space-8) var(--space-6);
  text-align: center;
  color: var(--colour-text-muted);
  font-size: var(--text-sm);
  background: var(--colour-surface);
}

/* 9. Responsive tweaks
   ---------------------------------------------------------- */
@media (max-width: 480px) {
  .site-header__nav { display: none; }
}

@media (min-width: 768px) {
  .hero    { padding: var(--space-16) var(--space-8); }
  .section { padding: var(--space-12) var(--space-8); }
}

@media (min-width: 1280px) {
  .site-header { padding-left: var(--space-12); padding-right: var(--space-12); }
}
```

Save the two files, open `portfolio.html` with Live Server, and you have a polished, responsive portfolio site built entirely with what we've learned in the last 8 classes.

---

## 15.12 Polish checklist — go through these before "done"

- [ ] **Hover states** on every interactive thing (links, buttons, project cards) — already in.
- [ ] **Focus rings** on buttons and form fields — already in via `:focus-visible`.
- [ ] **Real images** in the project thumbnails instead of the gradient — replace `.project__thumb` with `<img>` if you have screenshots.
- [ ] **Page title** in `<title>` matches the hero title.
- [ ] **`alt` text** on every image — important if you add real screenshots.
- [ ] **Test on mobile** — open Chrome DevTools and toggle the device toolbar (the phone icon, top-left of the panel). Try a few device presets.
- [ ] **Test keyboard navigation** — Tab through the page. Every link, button and input must show a visible focus state.
- [ ] **Lighthouse audit** — DevTools → Lighthouse → "Analyse page load". Aim for 90+ on Accessibility and Best Practices.

---

## Practice Exercises

1. **Make it yours** — Change the colour tokens to match your favourite colour palette. Try a green accent or a darker purple primary. See how much of the design changes from editing four lines.

2. **Add a section** — Add a "Hobbies" or "Education" section between About and Projects. Use the existing `.section` and `.section__title` classes — you should not need new CSS.

3. **Dark mode preview** — Add a `body.dark` rule that swaps the colour tokens for a dark theme. Try it:
   ```css
   body.dark {
     --colour-bg: #0f172a;
     --colour-surface: #1e293b;
     --colour-text: #f1f5f9;
     --colour-text-muted: #94a3b8;
     --colour-border: #334155;
   }
   ```
   Add `class="dark"` to the `<body>` to test. You will appreciate why we used variables.

4. **Stress test** — Open the page at 320px wide (smallest iPhone size). Anything overflow? Any text too small? Fix one issue you find.

5. **Hand-in** — Once you're happy, push your `portfolio.html` and `style.css` to GitHub (we cover Git in Class 27 — for now, ZIP them up). This is your first deployable piece of work.

---

## Key Takeaways
- A short design system — **colour tokens, type scale, spacing scale** — saves you from improvising and creates consistency for free
- Use **CSS variables** so changes can be made in one place
- **Flexbox** for one-dimensional bits (navbar, skill pills, form), **Grid** for the project gallery, **media queries** for the few places those don't reach
- **Hover transitions** and **focus rings** are tiny touches that make a portfolio feel professional
- You have just built your first complete, responsive site — CSS phase complete. Next class we move to JavaScript: making things *do* something.
