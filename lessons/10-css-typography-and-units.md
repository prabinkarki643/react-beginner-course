# Class 10: Typography & CSS Units

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Classes 8–9 (selectors and the box model)

## What You Will Learn
- Choosing fonts: web-safe stacks vs custom Google Fonts
- Sizing text: `font-size`, `font-weight`, `line-height`, `letter-spacing`
- CSS units demystified: `px`, `rem`, `em`, `%`, `vh`, `vw` — and when to use each
- Aligning and decorating text: `text-align`, `text-transform`, `text-decoration`
- Designing a typographic scale that feels intentional, not random

---

## 10.1 Why typography matters

Around 95% of the web is text. Get the typography right and the page feels professional before you have added a single colour. Get it wrong and even the prettiest layout will look amateurish.

Good typography means three things:
1. **Readable** — the right size, line height and contrast for the reader's eyes
2. **Hierarchical** — headings clearly stand out from body text
3. **Consistent** — the same scale and weights repeated through the page

CSS gives us a handful of properties to control all of this.

---

## 10.2 `font-family` — choosing typefaces

The `font-family` property accepts a *list* of fonts. The browser tries the first; if it is not available, it tries the second, and so on. The last entry should always be a generic family (`serif`, `sans-serif`, `monospace`) so the browser has something to fall back to.

```css
body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
```

Wrap any font name that contains a space in quotes: `'Segoe UI'`, `'Times New Roman'`.

### Web-safe fonts — installed on most systems

These are safe bets because they ship with operating systems.

| Stack | Use |
|-------|-----|
| `Arial, Helvetica, sans-serif` | Generic sans-serif |
| `Georgia, 'Times New Roman', serif` | Classic serif |
| `'Courier New', monospace` | Code |
| `'Segoe UI', Tahoma, sans-serif` | Modern UI sans-serif |

### The "system font" stack

The fastest possible font load — use whatever native UI font the operating system already has.

```css
body {
  font-family:
    -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
    'Helvetica Neue', Arial, sans-serif;
}
```

The page picks up San Francisco on macOS, Segoe UI on Windows, Roboto on Android. No download. Snappy.

---

## 10.3 Custom fonts with Google Fonts

System fonts are fast, but most designs use a custom typeface. Google Fonts is the easiest way to add one.

### Step 1 — pick a font on [fonts.google.com](https://fonts.google.com)

Search for something like **Inter**, **Poppins** or **Roboto**. Click the styles you want (e.g. **400** for regular, **600** for semi-bold, **700** for bold). Google gives you a link tag.

### Step 2 — add the link to your HTML `<head>`

```html
<head>
  <meta charset="UTF-8" />
  <title>Typography demo</title>

  <!-- Preconnect lines speed up the font request -->
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link
    rel="preconnect"
    href="https://fonts.gstatic.com"
    crossorigin
  />
  <link
    href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap"
    rel="stylesheet"
  />

  <link rel="stylesheet" href="style.css" />
</head>
```

### Step 3 — use it in CSS

```css
body {
  font-family: 'Inter', system-ui, Arial, sans-serif;
}
```

That last `system-ui, Arial, sans-serif` is a fallback in case the Google request fails. Always include one.

`display=swap` in the URL tells the browser: *show fallback text immediately, then swap to the custom font when it arrives*. Without it, the page can stay blank for a second.

---

## 10.4 Sizing text: `font-size`, `font-weight`, `line-height`

### `font-size`

The most fundamental property. Browsers default to 16px on the body.

```css
body { font-size: 16px; }
h1   { font-size: 32px; }
small { font-size: 12px; }
```

Until section 10.5 we are using `px`. Soon you will see why `rem` is better for most cases.

### `font-weight`

How thick the letters are. Common values:

| Value | Meaning |
|-------|---------|
| `100` / `200` | Thin |
| `300` | Light |
| `400` / `normal` | Regular (default) |
| `500` | Medium |
| `600` / `700` / `bold` | Semibold / Bold |
| `800` / `900` | Extra bold |

```css
.title    { font-weight: 700; }
.subtitle { font-weight: 500; }
.body     { font-weight: 400; }
```

Important: a weight only works if the font file ships with that weight. If you only imported `400` and `700` from Google Fonts, asking for `500` will silently fall back.

### `line-height` — vertical breathing room

The space between lines of text. Use **unitless** numbers — they scale with the font size.

```css
body { line-height: 1.6; }   /* 1.6 × the font size */
h1   { line-height: 1.2; }   /* tighter for big headings */
```

| Use | Suggested `line-height` |
|-----|------------------------|
| Body paragraphs | `1.5` to `1.7` |
| Large headings | `1.1` to `1.3` |
| Short labels (single line) | `1` |

A page with `line-height: 1.6` on the body feels instantly more readable than one stuck at the default.

### `letter-spacing` — fine-tuning

Adds or removes horizontal space between letters. Use sparingly.

```css
h1            { letter-spacing: -0.5px; }   /* big headings often tighten */
.uppercase    { letter-spacing: 0.08em; }   /* uppercase looks better spaced out */
```

---

## 10.5 CSS units — `px`, `rem`, `em`, `%`, `vh`, `vw`

This single section will save you weeks of confusion. Each unit has a specific job.

### `px` — pixels (absolute)

A `px` is always the same physical size on screen. Predictable, but it ignores the user's browser settings — including the option to make text bigger for accessibility.

```css
.box { width: 200px; padding: 16px; }
```

**Use for**: borders, small fixed offsets, anything you want to stay one exact size regardless of font scaling.

### `rem` — root em (relative to the page's base font size)

`1rem` = the `font-size` of the `<html>` element. By default browsers set that to 16px, so `1rem = 16px`. The magic: if the user (or you) changes the root font size, *everything in rem scales with it*.

```css
html { font-size: 16px; }   /* default — 1rem = 16px */

h1   { font-size: 2rem; }     /* 32px */
h2   { font-size: 1.5rem; }   /* 24px */
body { font-size: 1rem; }     /* 16px */
small { font-size: 0.875rem; } /* 14px */
```

**Use for**: nearly every `font-size`, and for paddings/margins/widths you want to scale with the user's settings. This is the unit you will use most.

A common pro trick — set `html { font-size: 62.5%; }`. That makes `1rem = 10px`, so the maths becomes trivial: `1.6rem = 16px`, `2.4rem = 24px`. Tailwind and most design systems prefer the default 16px base, so we will stick with that.

### `em` — relative to *this element's* font size

Tricky cousin of `rem`. `1em` equals the `font-size` of the element you are styling, *not* the root.

```css
.button {
  font-size: 16px;
  padding: 0.5em 1em;   /* 8px 16px */
}

.button.large {
  font-size: 20px;       /* now padding becomes 10px 20px automatically */
}
```

That auto-scaling can be very handy on components like buttons or pills. The risk: `em` *compounds* when nested (a `1.2em` inside a `1.2em` becomes 1.44 × the parent), which can produce surprises. **Beginner rule of thumb**: prefer `rem` unless you specifically want this auto-scaling behaviour.

### `%` — percent of the parent

```css
.image { width: 100%; }     /* fill the parent container */
.column { width: 50%; }     /* half the parent's width */
```

**Use for**: widths in fluid layouts; almost never for font sizes.

### `vh` and `vw` — viewport height and width

`1vh` = 1% of the **viewport** (visible browser area) height. `1vw` = 1% of its width.

```css
.hero {
  height: 100vh;   /* exactly one screen tall */
}

.modal {
  width: 80vw;     /* 80% of the viewport's width */
  max-width: 600px;
}
```

**Use for**: hero sections, full-screen overlays. Watch out on mobile where browser UI eats some viewport height.

### Quick decision table

| Property | Best unit | Reason |
|----------|-----------|--------|
| `font-size` | `rem` | Scales with user preference |
| Padding / margin / gap | `rem` | Stays consistent with type scale |
| Border width | `px` | Should not scale |
| Container width | `%` or `max-width` in `rem` | Flexible |
| Hero section height | `vh` | Tied to screen |
| Component-relative spacing (e.g. button padding) | `em` | Scales with its own text |

---

## 10.6 Text alignment, transform, and decoration

### `text-align`

```css
.title    { text-align: center; }
.right    { text-align: right; }
.justify  { text-align: justify; }   /* both edges aligned */
```

`left` is the default in left-to-right languages.

### `text-transform` — change case without changing the HTML

```css
.tag    { text-transform: uppercase; }
.name   { text-transform: capitalize; }   /* First Letter Of Every Word */
.shout  { text-transform: lowercase; }
```

`text-transform` only affects what is *displayed*; the underlying HTML stays the same. Screen readers read the original.

### `text-decoration`

```css
a            { text-decoration: none; }           /* remove the underline */
.strike      { text-decoration: line-through; }   /* old price */
.underline   { text-decoration: underline 2px dotted tomato; }
```

You can layer in the colour, style and thickness in one shorthand: `text-decoration: underline 2px dotted tomato;`.

---

## 10.7 Designing a typographic scale

A **typographic scale** is a small, fixed set of sizes you use throughout the site. Picking from a scale (rather than typing random numbers) makes everything feel cohesive.

A common simple scale based on `rem`:

| Use | Size |
|-----|------|
| `h1` | `2.5rem` (40px) |
| `h2` | `2rem` (32px) |
| `h3` | `1.5rem` (24px) |
| `h4` | `1.25rem` (20px) |
| Body | `1rem` (16px) |
| Small / muted | `0.875rem` (14px) |
| Caption | `0.75rem` (12px) |

Use **CSS custom properties** (variables) to define them once and re-use everywhere:

```css
:root {
  --font-body: 'Inter', system-ui, Arial, sans-serif;

  --text-xs:  0.75rem;
  --text-sm:  0.875rem;
  --text-base: 1rem;
  --text-lg:  1.25rem;
  --text-xl:  1.5rem;
  --text-2xl: 2rem;
  --text-3xl: 2.5rem;

  --leading-tight: 1.2;
  --leading-normal: 1.6;
}

body {
  font-family: var(--font-body);
  font-size: var(--text-base);
  line-height: var(--leading-normal);
}

h1 { font-size: var(--text-3xl); line-height: var(--leading-tight); }
h2 { font-size: var(--text-2xl); line-height: var(--leading-tight); }
h3 { font-size: var(--text-xl);  line-height: var(--leading-tight); }
.muted { color: #6b7280; font-size: var(--text-sm); }
```

Change the variable in one place, and the whole site updates. This is the same pattern Tailwind, Material UI and every modern design system use.

---

## 10.8 Putting it together — a typography demo page

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Typography demo</title>

    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link
      href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap"
      rel="stylesheet"
    />

    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <article class="post">
      <p class="post__tag">Tutorial</p>
      <h1 class="post__title">Learning CSS typography in one hour</h1>
      <p class="post__meta">By Prabin Karki · 5 min read</p>

      <p class="post__lead">
        Typography is what makes a page feel professional. Three properties
        will get you most of the way there.
      </p>

      <p>
        The default browser font-size is 16 pixels. Everything in this
        demo is measured in <code>rem</code>, so resizing the root font
        scales the whole page at once.
      </p>

      <h2>Why we use rem</h2>
      <p>
        It respects the user's browser settings. A reader who increases
        their default font size for accessibility sees the whole page
        grow accordingly.
      </p>

      <p class="post__muted">Last updated 2 February 2026.</p>
    </article>
  </body>
</html>
```

`style.css`:

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}

:root {
  --font-body: 'Inter', system-ui, Arial, sans-serif;

  --text-sm:  0.875rem;
  --text-base: 1rem;
  --text-lg:  1.25rem;
  --text-xl:  1.5rem;
  --text-2xl: 2rem;
  --text-3xl: 2.5rem;
}

body {
  margin: 0;
  font-family: var(--font-body);
  font-size: var(--text-base);
  line-height: 1.65;
  color: #1f2937;
  background-color: #f9fafb;
}

.post {
  max-width: 640px;
  margin: 40px auto;
  padding: 32px;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.post__tag {
  margin: 0 0 8px;
  font-size: var(--text-sm);
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: hsl(210, 80%, 45%);
}

.post__title {
  margin: 0 0 8px;
  font-size: var(--text-3xl);
  line-height: 1.2;
  letter-spacing: -0.5px;
}

.post__meta {
  margin: 0 0 24px;
  font-size: var(--text-sm);
  color: #6b7280;
}

.post__lead {
  font-size: var(--text-lg);
  color: #111827;
}

.post__muted {
  margin-top: 32px;
  font-size: var(--text-sm);
  color: #6b7280;
}

code {
  font-family: 'Courier New', monospace;
  background: #f3f4f6;
  padding: 2px 6px;
  border-radius: 4px;
  font-size: 0.95em;
}
```

Open it. Then try changing one line: `:root { font-size: 18px; }`. Notice how every size in the page grows proportionally. That is the power of `rem`.

---

## Practice Exercises

1. **Pick a font** — Visit fonts.google.com, choose any sans-serif you like, and load it into a small HTML page. Apply it to the body. Make a heading 2rem and a paragraph 1rem with a line-height of 1.6.

2. **Build a scale** — Define CSS custom properties for `--text-xs` through `--text-3xl` (see section 10.7). Make six paragraphs, each using a different size. Adjust the visual rhythm until it feels right.

3. **Unit comparison** — Make four boxes, each 200 wide, but use a different unit for each: `200px`, `12.5rem`, `50%`, `30vw`. Resize the browser and observe which boxes stay still and which grow or shrink.

4. **Article page** — Recreate the demo from section 10.8 from scratch, with your own content (a short article about a hobby). Use:
   - A Google Font of your choice
   - A scale of at least four sizes via CSS variables
   - Uppercase tag with letter-spacing
   - `text-transform: capitalize` on author names
   - A muted secondary colour

---

## Key Takeaways
- `font-family` accepts a fallback list — always end with a generic family like `sans-serif`
- Use `rem` for almost every size — it scales with the user's browser settings
- `px` for borders, `vh`/`vw` for hero sections, `%` for fluid widths
- A **typographic scale** defined once with CSS variables keeps everything consistent
- `line-height: 1.5`–`1.7` is the sweet spot for readable body text
- Next class: how to actually place these boxes on the page — `display`, `position`, `overflow` and `z-index`.
