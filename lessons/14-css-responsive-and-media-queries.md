# Class 14: Responsive Design & Media Queries

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Classes 8–13 (selectors, box model, typography, Flexbox, Grid)

## What You Will Learn
- The "mobile-first" mindset — why we design for phones first
- A standard set of breakpoints aligned with Tailwind: 640 / 768 / 1024 / 1280
- `@media (min-width: ...)` vs `(max-width: ...)` — and why we prefer one
- Fluid typography with `clamp()`
- Patterns for hiding and showing content at different breakpoints
- Common responsive pitfalls — and how to avoid them

---

## 14.1 What does "responsive" mean?

A responsive page **adapts its layout** to the screen it's displayed on. The same HTML and CSS look great on a phone, a tablet, a laptop and an ultrawide monitor.

Three tools together make this happen:

1. **Flexible layouts** — using `%`, `fr`, `flex-wrap`, `minmax(...)` so things rearrange naturally (Classes 12–13 cover this).
2. **Media queries** — CSS conditionals that apply rules only at certain screen sizes (this class).
3. **Fluid typography and spacing** — sizes that scale smoothly instead of jumping at breakpoints.

If your layout already uses `repeat(auto-fit, minmax(220px, 1fr))` or `flex-wrap: wrap`, you may need *very few* media queries. That's good — fewer breakpoints means fewer bugs.

---

## 14.2 The viewport meta tag — non-negotiable

Before any CSS will work properly on a phone, you need this line in your HTML `<head>`:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```

Without it, mobile browsers pretend the screen is 980px wide and zoom out to fit, which makes every responsive trick useless. This tag tells the browser: **the viewport is exactly as wide as the device's screen**.

Every HTML5 boilerplate already includes it (we saw it back in Class 2). Just make sure it's there before testing on a phone.

---

## 14.3 Mobile-first thinking

There are two ways to design responsively:

- **Desktop-first**: write CSS for big screens, then add overrides for smaller ones using `max-width`.
- **Mobile-first**: write CSS for small screens, then add overrides for larger ones using `min-width`.

**Mobile-first wins** because:

- Most users browse on phones; we should design for them first.
- Mobile-first CSS is simpler — you start small and **add** complexity, rather than starting complex and stripping it away.
- `min-width` queries stack predictably: each one builds on the last.

The pattern looks like this:

```css
/* Base styles — for phones (no media query) */
.container {
  padding: 16px;
}

/* Tablets and up */
@media (min-width: 768px) {
  .container {
    padding: 24px;
    max-width: 720px;
    margin: 0 auto;
  }
}

/* Desktops and up */
@media (min-width: 1024px) {
  .container {
    max-width: 960px;
  }
}
```

You only override what changes. The base rules carry through unless replaced.

---

## 14.4 Standard breakpoints

There is no official set of breakpoints, but we will use the same ones as Tailwind CSS — because Phase 7 of this course is Tailwind, and the muscle memory transfers.

| Name | `min-width` | Roughly fits |
|------|-------------|--------------|
| `sm` | `640px` | Larger phones, small tablets in portrait |
| `md` | `768px` | Tablets |
| `lg` | `1024px` | Small laptops, tablets in landscape |
| `xl` | `1280px` | Most laptops and desktops |
| `2xl` | `1536px` | Large monitors |

For most projects you only need two or three of these (typically `md` and `lg`). Don't add breakpoints out of habit — add them when something genuinely breaks at a certain width.

### Project-wide rule of thumb

```css
/* 1. Base — mobile */
.element { /* mobile styles */ }

/* 2. Tablet */
@media (min-width: 768px)  { .element { /* tablet overrides */ } }

/* 3. Desktop */
@media (min-width: 1024px) { .element { /* desktop overrides */ } }

/* 4. Wide desktop, only if needed */
@media (min-width: 1280px) { .element { /* xl overrides */ } }
```

---

## 14.5 The `@media` syntax

```css
@media (min-width: 768px) {
  /* CSS rules here apply ONLY when the viewport is at least 768px wide */
  .navbar { display: flex; }
  .card   { padding: 32px; }
}
```

Inside the curly braces, write normal CSS rules. They apply only when the condition is true. There is no limit to how many rules you put inside.

### Combining conditions with `and`

```css
/* Between 768px and 1023px — tablets only */
@media (min-width: 768px) and (max-width: 1023px) {
  .sidebar { display: none; }
}

/* Landscape orientation */
@media (orientation: landscape) {
  .hero { height: 70vh; }
}

/* Hi-resolution displays — for sharper icons */
@media (min-resolution: 2dppx) {
  .logo { background-image: url("logo@2x.png"); }
}
```

For now you'll only really use `min-width` and `max-width`. The rest is useful trivia.

### `min-width` vs `max-width` — why mobile-first prefers `min-width`

Consider styling a navbar:

```css
/* Mobile-first (preferred) */
.navbar { display: block; }
@media (min-width: 768px) {
  .navbar { display: flex; }
}

/* Desktop-first */
.navbar { display: flex; }
@media (max-width: 767px) {
  .navbar { display: block; }
}
```

Both work. Mobile-first reads as: "by default block, on tablet+ switch to flex." Desktop-first reads as: "by default flex, but on small screens go back to block." The first is easier to maintain — the base case is the default, and each breakpoint adds something new.

---

## 14.6 Fluid typography with `clamp()`

Sometimes you want text to grow smoothly instead of jumping between breakpoints. `clamp(min, preferred, max)` is the tool.

```css
h1 {
  font-size: clamp(1.5rem, 4vw + 1rem, 3rem);
}
```

That reads:
- never smaller than `1.5rem`
- ideally `4vw + 1rem` (4% of the viewport width plus 1rem)
- never bigger than `3rem`

On a phone the heading will sit around the minimum. On a wide desktop it will hit the maximum. In between, it scales smoothly.

This often replaces three or four media queries with one line. Here is a useful starter scale:

```css
h1 { font-size: clamp(2rem, 5vw + 0.5rem, 3.5rem); }
h2 { font-size: clamp(1.5rem, 3vw + 0.5rem, 2.5rem); }
h3 { font-size: clamp(1.25rem, 2vw + 0.5rem, 1.75rem); }
```

You can use `clamp()` for spacing too — `padding: clamp(1rem, 5vw, 4rem)` gives generous padding on big screens and tight padding on phones.

---

## 14.7 Hide and show patterns

A common need: show something on desktop, hide it on mobile (or vice versa).

```css
/* Hidden on mobile, shown on tablet and up */
.desktop-only { display: none; }

@media (min-width: 768px) {
  .desktop-only { display: block; }
}

/* Hidden from tablet, shown only on mobile */
@media (min-width: 768px) {
  .mobile-only { display: none; }
}
```

For example, a desktop navbar might show full text links while a mobile screen shows a hamburger button:

```html
<header class="navbar">
  <strong>My Site</strong>
  <button class="mobile-only">☰</button>
  <nav class="desktop-only">
    <a href="#">Home</a>
    <a href="#">About</a>
    <a href="#">Contact</a>
  </nav>
</header>
```

A note about accessibility: `display: none` removes the element from the screen *and* from screen readers. That's usually fine. If you want it visible to screen readers but hidden visually (e.g. a label), use a "visually-hidden" utility:

```css
.sr-only {
  position: absolute;
  width: 1px; height: 1px;
  padding: 0; margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

We will not need that often, but it's good to know it exists.

---

## 14.8 A complete responsive page

A small page with a navbar that becomes a hamburger on mobile, a hero that scales, and a card grid that adapts.

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Responsive demo</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <header class="navbar">
      <strong class="navbar__brand">My Site</strong>
      <button class="navbar__menu mobile-only" aria-label="Menu">☰</button>
      <nav class="navbar__nav desktop-only">
        <a href="#">Home</a>
        <a href="#">Posts</a>
        <a href="#">About</a>
        <a href="#">Contact</a>
      </nav>
    </header>

    <section class="hero">
      <h1 class="hero__title">Build a responsive page</h1>
      <p class="hero__text">
        This page uses one set of styles for phones, then adds a few tweaks
        for tablets and desktops.
      </p>
    </section>

    <section class="cards">
      <article class="card"><h3>Fast</h3><p>Loads quickly on every device.</p></article>
      <article class="card"><h3>Accessible</h3><p>Works for everyone.</p></article>
      <article class="card"><h3>Beautiful</h3><p>Looks great big or small.</p></article>
      <article class="card"><h3>Maintainable</h3><p>Easy to update later.</p></article>
    </section>
  </body>
</html>
```

`style.css`:

```css
*, *::before, *::after { box-sizing: border-box; }

:root {
  --max-width: 1100px;
}

body {
  margin: 0;
  font-family: system-ui, Arial, sans-serif;
  color: #1f2937;
  background: #f9fafb;
  line-height: 1.6;
}

/* ---------- Utility classes ---------- */

.mobile-only { display: block; }
.desktop-only { display: none; }

@media (min-width: 768px) {
  .mobile-only { display: none; }
  .desktop-only { display: block; }
}

/* ---------- Navbar ---------- */

.navbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  background: white;
  border-bottom: 1px solid #e5e7eb;
}

.navbar__brand { font-size: 18px; }

.navbar__menu {
  font-size: 24px;
  background: none;
  border: none;
  cursor: pointer;
}

.navbar__nav { display: flex; gap: 20px; }
.navbar__nav a { text-decoration: none; color: #374151; }
.navbar__nav a:hover { color: hsl(210, 80%, 45%); }

/* ---------- Hero ---------- */

.hero {
  max-width: var(--max-width);
  margin: 0 auto;
  padding: clamp(2rem, 6vw, 5rem) 16px;
  text-align: center;
}

.hero__title {
  font-size: clamp(1.75rem, 5vw + 0.5rem, 3.5rem);
  line-height: 1.2;
  margin: 0 0 12px;
}

.hero__text {
  max-width: 600px;
  margin: 0 auto;
  color: #4b5563;
  font-size: 1.125rem;
}

/* ---------- Card grid ---------- */

.cards {
  max-width: var(--max-width);
  margin: 0 auto;
  padding: 0 16px 48px;
  display: grid;
  gap: 16px;
  grid-template-columns: 1fr;             /* mobile: 1 column */
}

@media (min-width: 640px) {
  .cards { grid-template-columns: 1fr 1fr; }     /* sm: 2 columns */
}

@media (min-width: 1024px) {
  .cards { grid-template-columns: repeat(4, 1fr); }  /* lg: 4 columns */
}

.card {
  background: white;
  padding: 24px;
  border-radius: 12px;
  border: 1px solid #e5e7eb;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);
}

.card h3 { margin: 0 0 8px; }
.card p  { margin: 0; color: #4b5563; }
```

Open it and resize the browser window slowly from 320px wide to 1600px wide. Notice:

- The hamburger appears under 768px; the link list appears above.
- The heading grows smoothly thanks to `clamp()`.
- The cards go from 1 column → 2 columns → 4 columns at the breakpoints.

You now have all the moving parts you need to build a properly responsive site.

---

## 14.9 Common pitfalls

- **Forgot the viewport meta tag.** Everything looks zoomed out on a phone. Always check this first.
- **Designed at 1440px and didn't test smaller.** Resize the browser as you write CSS — make small windows your default, not the exception.
- **Too many breakpoints.** If a layout only needs two breakpoints, only use two. Three is fine. Five is usually a sign something is wrong.
- **Mixing `min-width` and `max-width`** in the same file. Pick one (mobile-first means `min-width`) and stick with it.
- **Testing only in Chrome DevTools.** It's a great first check but does not catch everything. Use a real phone before claiming a page is done.
- **Hiding important content on mobile.** If a feature matters on desktop, find a way to surface it on mobile — don't just `display: none` it.

---

## Practice Exercises

1. **Mobile-first refactor** — Take a small page you wrote earlier with one media query using `max-width`. Rewrite it mobile-first using `min-width`. Compare the diff.

2. **Three-up cards** — Build a card grid that shows 1 column under 640px, 2 columns from 640px–1023px, and 3 columns from 1024px upwards. Use Grid (not Flexbox) so the cards line up evenly.

3. **Clamp the heading** — Write a hero heading that ranges from `1.5rem` on a phone to `4rem` on a wide desktop, using a single `clamp()` line. No media queries allowed.

4. **Show/hide swap** — Add a "Sign in" button that appears as a full text button on desktop and as a small icon on mobile. Use `.mobile-only` and `.desktop-only` utilities.

5. **Stress test** — Take the demo from section 14.8 and open it on the smallest and largest screens you have. Note any spot where text overflows, cards look squashed, or padding feels wrong. Fix at least one.

---

## Key Takeaways
- Always include the **viewport meta tag** in your HTML `<head>`
- Design **mobile-first** — base styles for phones, `min-width` queries to add complexity for larger screens
- Use the breakpoints **640 / 768 / 1024 / 1280** so your mental model matches Tailwind later
- `clamp(min, preferred, max)` makes text and spacing scale smoothly without breakpoints
- Flexible layouts (Flexbox wrap, Grid `minmax`) cut down on the number of media queries you need
- Next class: pull everything together — style the portfolio page from Class 7 into a polished, responsive site.
