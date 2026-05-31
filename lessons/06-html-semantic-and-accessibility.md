# Class 6: Semantic HTML & Accessibility Basics

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Classes 2–5 — you can build complete pages with text, images, lists, tables, and forms.

## What You Will Learn
- What **semantic HTML** means and why it matters
- The page-structure elements: `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>`
- The **heading outline** and how landmarks help screen readers
- Accessibility essentials: meaningful `alt` text, `aria-label`, keyboard navigation
- When `<div>` and `<span>` are still the right choice

---

## 6.1 What "Semantic" Means

A **semantic** HTML element is one whose tag *describes the meaning* of the content inside it. Examples:

- `<nav>` says "this is navigation".
- `<article>` says "this is a self-contained article".
- `<header>` says "this is the top section".

Compare with a **non-semantic** element like `<div>`. A `<div>` is just "a box" — it tells you nothing about what is inside.

You could build every page using only `<div>`s — it would *look* the same on screen. But three audiences would suffer:

1. **Search engines** — Google reads your tags to understand your page. `<article>` tells it "here is a piece of content worth indexing". A `<div>` tells it nothing.
2. **Screen readers** — Blind users navigate by jumping between **landmarks** (header, nav, main, footer). A page of `<div>`s has no landmarks, so they have to read everything top to bottom.
3. **Future developers** (including future you) — A page of named sections is far easier to maintain than a sea of `<div>`s.

**Bottom line**: pick the tag that *describes* your content. Use `<div>` only when no semantic tag fits.

---

## 6.2 The Page-Structure Elements

These tags model the regions of a typical web page:

| Element | What it represents | Typical contents |
|---------|---------------------|------------------|
| `<header>` | The top of a page or section | Logo, site title, top navigation |
| `<nav>` | A block of navigation links | Main menu, sidebar links, footer link list |
| `<main>` | The page's main, unique content | Everything specific to this page |
| `<section>` | A thematic grouping with a heading | "About", "Services", "Contact" |
| `<article>` | A piece of self-contained content | A blog post, a news item, a comment |
| `<aside>` | Sidebar content, tangentially related | Author bio, related links, ads |
| `<footer>` | The bottom of a page or section | Copyright, secondary links, social icons |

Here is how they fit together for a typical homepage:

```
┌─────────────────────────────────────────────┐
│  <header>                                   │
│   ┌──────────────────────────────────────┐  │
│   │ Logo                  <nav>          │  │
│   │                       Home About …   │  │
│   └──────────────────────────────────────┘  │
├─────────────────────────────────────────────┤
│  <main>                                     │
│   <section> Hero / Intro </section>         │
│   <section> Features </section>             │
│   <section> Testimonials </section>         │
│   ┌──── <aside> ────┐                       │
│   │ Related links   │                       │
│   └─────────────────┘                       │
├─────────────────────────────────────────────┤
│  <footer>                                   │
│   © 2026 Triton College                     │
└─────────────────────────────────────────────┘
```

A page should normally have **one** `<header>`, **one** `<main>`, and **one** `<footer>` at the top level. `<section>`, `<article>`, `<nav>`, and `<aside>` can appear as many times as you need.

> A small page header *inside* an article (e.g. with the article title) can also be a `<header>` — these tags describe *relative* sections, not just the outer ones.

---

## 6.3 A Semantic Page Skeleton

Here is the same content you could have written entirely with `<div>`s, expressed in semantic tags. Save as `semantic-demo.html` and open with Live Server.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Triton College Blog</title>
  </head>
  <body>
    <header>
      <h1>Triton College Blog</h1>
      <nav aria-label="Main">
        <ul>
          <li><a href="#latest">Latest</a></li>
          <li><a href="#about">About</a></li>
          <li><a href="#contact">Contact</a></li>
        </ul>
      </nav>
    </header>

    <main>
      <section id="latest">
        <h2>Latest Posts</h2>

        <article>
          <header>
            <h3>Starting a React Course at Triton</h3>
            <p>Posted on <time datetime="2026-05-30">30 May 2026</time></p>
          </header>
          <p>
            This week we kick off our first React course for BCA Semester 2 students.
            Here is what to expect over the next seventeen weeks…
          </p>
          <a href="#">Read more</a>
        </article>

        <article>
          <header>
            <h3>Why HTML Still Matters in 2026</h3>
            <p>Posted on <time datetime="2026-05-23">23 May 2026</time></p>
          </header>
          <p>
            React feels like magic, but underneath every component is plain HTML.
            Learn the foundations and the framework gets much easier.
          </p>
          <a href="#">Read more</a>
        </article>
      </section>

      <aside aria-label="Related links">
        <h2>You might also like</h2>
        <ul>
          <li><a href="#">Course schedule</a></li>
          <li><a href="#">Meet the instructor</a></li>
          <li><a href="#">Student gallery</a></li>
        </ul>
      </aside>
    </main>

    <footer>
      <p>&copy; 2026 Triton College. All rights reserved.</p>
      <nav aria-label="Footer">
        <ul>
          <li><a href="#">Privacy</a></li>
          <li><a href="#">Terms</a></li>
        </ul>
      </nav>
    </footer>
  </body>
</html>
```

**What to expect**: a plain, unstyled but clearly *structured* page — a header at the top, two blog post cards in the middle, a "You might also like" list off to one side, and a footer. The browser does not visually distinguish them yet (that is CSS's job), but a screen reader now hears:

- "Banner, heading: Triton College Blog"
- "Navigation, Main"
- "Main"
- "Article, Starting a React Course at Triton…"

Each major section is announced by its **role** — a huge win for users with a screen reader.

---

## 6.4 The Heading Outline

Every page should form a logical **heading tree**:

```
<h1>      ← The page topic (only one)
  <h2>    ← A section
    <h3>  ← A sub-section
    <h3>  ← Another sub-section
  <h2>    ← Another section
```

**Two non-negotiable rules:**

1. **One `<h1>` per page.**
2. **Do not skip levels.** Go `h1` → `h2` → `h3`, not `h1` → `h3`.

You pick the level based on *meaning*, not size. If you want a smaller `<h2>`, change the CSS — never demote it to `<h3>` just to look smaller.

Many browsers and tools (including the WAVE accessibility extension) can show you the outline. A good outline = a good page.

---

## 6.5 What Are Landmarks?

When a screen-reader user lands on your page, they typically press a key (often `D`) to jump from landmark to landmark instead of reading the entire page. The page-structure elements above are *landmarks*:

| Element | Landmark role |
|---------|---------------|
| `<header>` (top level) | `banner` |
| `<nav>` | `navigation` |
| `<main>` | `main` |
| `<aside>` | `complementary` |
| `<footer>` (top level) | `contentinfo` |
| `<section>` with a heading | `region` |
| `<form>` | `form` |

You get all of this **for free** simply by using the right tag. That is the whole point of semantic HTML.

When you have *several* of the same landmark (e.g. a main `<nav>` and a footer `<nav>`), label them so they can be told apart:

```html
<nav aria-label="Main">
  <!-- main menu -->
</nav>

<nav aria-label="Footer">
  <!-- footer links -->
</nav>
```

`aria-label` is a short text label read by screen readers — it does not show on screen.

---

## 6.6 Accessibility Essentials

**Accessibility** (often shortened to **a11y** — "a", then 11 letters, then "y") means making your site usable by *everyone*, including people who:

- Cannot see the screen and use a screen reader.
- Cannot use a mouse and navigate by keyboard.
- Have low vision and need larger text or higher contrast.
- Have motor difficulties and need bigger click targets.

You do not need to be an expert in any of these. Following a few simple habits makes you accessible by default.

### Habit 1 — Meaningful `alt` text

We covered this in Class 3. Recap: every `<img>` needs `alt`. Describe the image's *purpose in context*, not literally what is shown.

```html
<!-- Bad -->
<img src="award.png" alt="award.png" />

<!-- Good -->
<img src="award.png" alt="Triton College Best Lecturer award 2024" />
```

### Habit 2 — Label every input

Every form input needs a `<label>` (Classes 4 and 5). For *icon-only* buttons (a gear icon, a magnifying glass), add an `aria-label`:

```html
<button aria-label="Search">
  <!-- magnifying glass icon -->
</button>

<button aria-label="Open settings">
  <!-- gear icon -->
</button>
```

### Habit 3 — Keyboard navigation just works

The good news: links (`<a>`), buttons (`<button>`), and form inputs are **all keyboard-accessible by default**. Users can press `Tab` to move between them and `Enter` or `Space` to activate them.

The bad news: if you make a fake button out of a `<div>`, it does **not** work with the keyboard. Always use the right tag:

| If you want… | Use |
|--------------|-----|
| A clickable thing that goes somewhere | `<a href="…">` |
| A clickable thing that triggers an action | `<button>` |

Never `<div onclick="…">`. Keyboard users cannot reach it.

### Habit 4 — Sensible link text

Avoid "click here" or "read more" on their own. Screen reader users often pull up a list of *just the link text* on a page — "click here, click here, click here" tells them nothing.

```html
<!-- Bad -->
<p>For the schedule, <a href="schedule.pdf">click here</a>.</p>

<!-- Good -->
<p>Download the <a href="schedule.pdf">course schedule (PDF)</a>.</p>
```

### Habit 5 — Colour is not the only signal

We are not on CSS yet, but remember for later: a coloured error message must also have *text* saying "Error". A red box alone is invisible to colour-blind users.

---

## 6.7 When `<div>` and `<span>` Are Still Right

Semantic tags do not cover *every* situation. Sometimes you need a plain box just to attach styling or scripting to. That is what `<div>` and `<span>` are for.

| Element | Type | Use for |
|---------|------|---------|
| `<div>` | Block-level (full width, stacks vertically) | A generic container with no special meaning |
| `<span>` | Inline (flows within text) | A run of inline content (a styled word, for example) |

Examples:

```html
<!-- A wrapper used purely for CSS grid/flex layout -->
<div class="card-grid">
  <article class="card">...</article>
  <article class="card">...</article>
  <article class="card">...</article>
</div>

<!-- Highlighting one word inside a paragraph -->
<p>
  The price is
  <span class="price">£49.99</span>
  including VAT.
</p>
```

The rule: **reach for a semantic tag first**. If none describes your content, use `<div>` or `<span>` without guilt.

---

## 6.8 Refactoring a `<div>` Page into Semantic HTML

Here is a tiny page written with only `<div>`s — accurate but uninformative:

```html
<div>
  <div>
    <div>My Blog</div>
    <div>
      <a href="#">Home</a>
      <a href="#">Posts</a>
    </div>
  </div>
  <div>
    <div>Latest Post</div>
    <div>This week we started a React course…</div>
  </div>
  <div>© 2026</div>
</div>
```

And here is the same page rewritten with the right tags:

```html
<header>
  <h1>My Blog</h1>
  <nav aria-label="Main">
    <ul>
      <li><a href="#">Home</a></li>
      <li><a href="#">Posts</a></li>
    </ul>
  </nav>
</header>

<main>
  <article>
    <h2>Latest Post</h2>
    <p>This week we started a React course…</p>
  </article>
</main>

<footer>
  <p>&copy; 2026</p>
</footer>
```

Both render almost identically. The second version is leagues ahead for search engines, screen readers, and the next developer to touch the code.

---

## Practice Exercises

### Exercise 1 — Refactor the registration form (15 minutes)
Take your `register.html` from Class 5 and wrap it in semantic landmarks:

- A `<header>` containing the page `<h1>`.
- A `<main>` containing the form.
- A `<footer>` containing a short copyright line.
- Add a `<nav aria-label="Main">` inside the header with two dummy links ("Home", "Help").

### Exercise 2 — A two-article blog page
Create `blog.html` with:

- A `<header>` with a site title and a navigation list.
- A `<main>` containing **two** `<article>` elements, each with its own `<header>` (containing the article title and date), a paragraph, and a "Read more" link.
- An `<aside>` with a short "About the author" paragraph.
- A `<footer>` with a copyright line.

### Exercise 3 — Find one fake button
Search any web page you visit today for a clickable element that is *not* a `<a>` or a `<button>`. Try to use it with the keyboard only (`Tab` + `Enter`). Note what happens. (This is a "noticing" exercise — no code.)

---

## Key Takeaways
- A **semantic tag** describes its content's meaning; a `<div>` does not.
- Landmarks (`<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`) give screen-reader users a "table of contents" of your page for free.
- Keep **one `<h1>`** per page and never skip heading levels.
- Use `<a>` for navigation, `<button>` for actions — never `<div onclick>`.
- Always write meaningful `alt` text and label icon-only buttons with `aria-label`.
- `<div>` and `<span>` are still fine when no semantic tag fits.

Next class is a **hands-on workshop**: we will combine everything from Classes 2–6 to build a complete, unstyled portfolio page that we will later style in Class 15.
