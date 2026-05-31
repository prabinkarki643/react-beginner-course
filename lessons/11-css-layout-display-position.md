# Class 11: Layout — `display`, `position`, `overflow`, `z-index`

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Classes 8–10 (box model, colours, typography)

## What You Will Learn
- `display`: `block`, `inline`, `inline-block`, `none` — and why headings sit on their own line while links don't
- `position`: `static`, `relative`, `absolute`, `fixed`, `sticky` — pinning, floating and nudging elements
- `overflow` — what happens when content is too big for its box
- `z-index` and the stacking context — controlling which element sits on top
- Centring techniques *before* Flexbox arrives (next class)

---

## 11.1 Why this class matters

So far our boxes have just stacked on top of each other in HTML order. That's fine for a single article, but real pages have navbars pinned to the top, sidebars, floating buttons, and overlays. To move beyond a one-column page we need `display` and `position`.

Flexbox and Grid (the next two classes) handle most modern layouts, but `display` and `position` are still the foundation. Every CSS layout property builds on them.

---

## 11.2 `display` — block, inline, and friends

Every HTML element has a default `display` value. Some flow top-to-bottom; others flow side-by-side. Knowing why is the first step to controlling layout.

### `block` — takes the full width, sits on its own line

`<div>`, `<p>`, `<h1>`–`<h6>`, `<section>`, `<article>`, `<header>`, `<footer>`, `<ul>`, `<li>` are all `block` by default.

- Stretches to fill the parent's width
- Always starts on a new line
- Respects `width` and `height`

```html
<p>First paragraph.</p>
<p>Second paragraph.</p>
```

The two `<p>` elements stack vertically — that's `display: block` at work.

### `inline` — flows like text, only as wide as needed

`<span>`, `<a>`, `<strong>`, `<em>`, `<img>` (sort of) are `inline` by default.

- Sits on the same line as siblings
- Width and height are *ignored*
- Only horizontal padding/margin behaves predictably

```html
<p>
  Read more on the <a href="#">documentation page</a> or in the
  <a href="#">tutorial</a>.
</p>
```

The two links sit inline with the surrounding text. Try giving them `width: 200px` — nothing happens. That is inline behaviour.

### `inline-block` — best of both

Sits on the same line as siblings (like inline) *and* respects width/height (like block). Useful for buttons or tags in a row.

```html
<a class="tag" href="#">CSS</a>
<a class="tag" href="#">HTML</a>
<a class="tag" href="#">JavaScript</a>
```

```css
.tag {
  display: inline-block;
  padding: 6px 12px;
  margin-right: 8px;
  background-color: #e5e7eb;
  border-radius: 999px;
  text-decoration: none;
  color: #1f2937;
  font-size: 14px;
}
```

The three tags sit on one line, each with its own padding and rounded shape.

### `none` — remove the element from the layout entirely

```css
.hidden { display: none; }
```

The element disappears as if it were never in the HTML. It does not take up space.

> **Compare with `visibility: hidden`** — that one hides the element but still reserves its space. And `opacity: 0` keeps it fully there but invisible (and still clickable). Three different "hide" options for three different needs.

### Summary table

| Value | New line? | Width / height respected? | Padding & margin? |
|-------|-----------|---------------------------|-------------------|
| `block` | Yes | Yes | Yes (all sides) |
| `inline` | No | No | Horizontal only |
| `inline-block` | No | Yes | Yes (all sides) |
| `none` | Removed from layout | — | — |

We will meet `display: flex` next class and `display: grid` the class after that.

---

## 11.3 `position` — getting elements out of the normal flow

The `position` property controls how an element is placed and whether it participates in the normal top-to-bottom flow.

### `position: static` (the default)

Normal flow. The properties `top`, `right`, `bottom`, `left` have no effect. Every element is `static` until you change it.

### `position: relative` — nudged from its normal spot

Still sits in the normal flow, but you can shift it with `top` / `right` / `bottom` / `left`. The space where it *would* have been is preserved.

```html
<p>Normal paragraph.</p>
<p class="nudged">I am nudged 20px down and 30px right.</p>
<p>Another normal paragraph.</p>
```

```css
.nudged {
  position: relative;
  top: 20px;
  left: 30px;
  background-color: lightyellow;
}
```

The third paragraph does *not* slide up to fill the gap — `relative` only displaces visually.

`relative` is also a **positioning anchor** for child elements. We will use this immediately for `absolute`.

### `position: absolute` — pinned to the nearest positioned ancestor

Removed from the normal flow entirely. Other elements behave as if it were not there. Coordinates (`top`, `left`, etc.) are measured from the closest ancestor that has `position` set to anything other than `static` — usually `relative`.

The classic pattern: **a "badge" pinned to the top-right of a card**.

```html
<article class="card">
  <span class="card__badge">NEW</span>
  <h3>Product title</h3>
  <p>Some description text here.</p>
</article>
```

```css
.card {
  position: relative;            /* anchor for the badge */
  padding: 20px;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  width: 300px;
}

.card__badge {
  position: absolute;
  top: 12px;
  right: 12px;
  background-color: tomato;
  color: white;
  font-size: 12px;
  padding: 4px 8px;
  border-radius: 999px;
}
```

The badge sits 12px from the top-right corner of the card. If you remove `position: relative` from `.card`, the badge will jump to the top-right of the *page* (the viewport) — because the nearest positioned ancestor falls back to the `<body>`. **Always set `position: relative` on the parent before using `position: absolute` on the child.**

### `position: fixed` — pinned to the viewport

Like `absolute`, but measured from the viewport (browser window). Stays in place when you scroll. Use for navbars, floating action buttons, "back to top" buttons.

```html
<button class="back-to-top">↑</button>
```

```css
.back-to-top {
  position: fixed;
  bottom: 24px;
  right: 24px;
  width: 48px;
  height: 48px;
  border-radius: 50%;
  border: none;
  background-color: hsl(210, 80%, 50%);
  color: white;
  font-size: 18px;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}
```

Add enough content to make the page scroll. The button stays glued to the bottom-right corner the whole time.

### `position: sticky` — hybrid of relative and fixed

Behaves like `relative` until you scroll past it, at which point it sticks to a position. Perfect for "sticky section headers" and sometimes navbars.

```html
<header class="site-header">My site</header>
<main>
  <h2 class="sticky-heading">Section A</h2>
  <p>... long content ...</p>
  <h2 class="sticky-heading">Section B</h2>
  <p>... long content ...</p>
</main>
```

```css
.sticky-heading {
  position: sticky;
  top: 0;                   /* sticks 0px from the top of the viewport */
  background: white;
  padding: 8px 0;
  border-bottom: 1px solid #e5e7eb;
}
```

Scroll and watch each heading stick to the top of the viewport until the next one pushes it out.

> **Gotcha**: `position: sticky` only works if the parent container is tall enough to scroll *and* you specify a `top` (or `bottom`) value. Forgetting either is the most common reason it "doesn't work".

### Summary table

| `position` | Removed from flow? | Anchored to | Scrolls with page? |
|------------|--------------------|-------------|--------------------|
| `static` | No | — | Yes |
| `relative` | No (just nudged) | Original spot | Yes |
| `absolute` | Yes | Nearest positioned ancestor | Yes (with that ancestor) |
| `fixed` | Yes | Viewport | No |
| `sticky` | No | Original spot, until threshold | Yes, then sticks |

---

## 11.4 `overflow` — what happens when content overflows

When the content inside a box is taller (or wider) than the box, you control the result with `overflow`.

```html
<div class="box">
  Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do
  eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim
  ad minim veniam, quis nostrud exercitation ullamco laboris nisi.
</div>
```

```css
.box {
  width: 200px;
  height: 80px;
  border: 1px solid steelblue;
  padding: 8px;
}

/* Try each of these in turn */
.box { overflow: visible; }   /* default — content spills outside (ugly) */
.box { overflow: hidden; }    /* anything outside the box is clipped */
.box { overflow: auto; }      /* scrollbars appear only when needed */
.box { overflow: scroll; }    /* scrollbars always present */
```

You can also use `overflow-x` and `overflow-y` independently — useful for horizontal-scroll image galleries:

```css
.gallery {
  overflow-x: auto;
  overflow-y: hidden;
  white-space: nowrap;        /* keep children on one line */
}
```

`overflow: hidden` is also used as a hack to **trim text** combined with three properties:

```css
.title {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;    /* ... at the end */
  max-width: 200px;
}
```

That truncates long titles with a tidy `...`.

---

## 11.5 `z-index` and the stacking context

When two elements overlap, which one appears on top?

By default, elements later in the HTML are drawn on top of earlier ones. Once you start using `position` (anything other than `static`), you can override that with `z-index` — higher numbers come forward.

```html
<div class="layer red"></div>
<div class="layer blue"></div>
```

```css
.layer {
  width: 200px;
  height: 200px;
  position: absolute;
  top: 50px;
}

.red {
  background: tomato;
  left: 50px;
  z-index: 1;
}

.blue {
  background: cornflowerblue;
  left: 150px;
  z-index: 2;    /* sits on top of the red one */
}
```

A sensible scale for a project:

| Layer | `z-index` |
|-------|-----------|
| Default | `auto` (0) |
| Sticky headers | `10` |
| Navbar dropdowns | `100` |
| Modal overlay | `1000` |
| Toast notifications | `2000` |

**Important rule**: `z-index` only works on positioned elements (`relative`, `absolute`, `fixed`, `sticky`). It is silently ignored on `static` elements.

### Stacking context — the trap

A "stacking context" is like a private z-index scope. Once a parent creates one (by having a non-static position *and* a `z-index`, or certain other properties like `opacity` less than 1), its children stack only against each other — never against elements outside.

In practice this catches every CSS beginner. If your modal with `z-index: 9999` is still hidden behind a navbar with `z-index: 10`, the culprit is almost always a parent that started its own stacking context. The fix is usually to render the modal at the top level of the `<body>`, not inside a styled card.

We will not dive deeper today — just remember the symptom.

---

## 11.6 Centring things before Flexbox

Flexbox makes centring trivial. Before that arrives, here are the techniques that have been used for years.

### Horizontal centring of a block element — `margin: 0 auto`

The classic. Requires a fixed (or max-) width.

```css
.container {
  max-width: 720px;
  margin: 0 auto;        /* 0 top/bottom, auto left/right */
  padding: 0 16px;
}
```

The browser splits the leftover horizontal space equally on both sides — centring the element.

### Centring text — `text-align: center`

Works on inline content inside a block.

```css
.hero {
  text-align: center;
}
```

### Absolute centring — the position trick

Centre any element (vertically *and* horizontally) within a positioned parent:

```html
<div class="parent">
  <div class="child">Centred</div>
</div>
```

```css
.parent {
  position: relative;
  width: 400px;
  height: 200px;
  background: #f3f4f6;
}

.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);   /* shift back by half its own size */
  background: white;
  padding: 16px;
  border: 1px solid #e5e7eb;
}
```

The `transform: translate(-50%, -50%)` pulls the element back by 50% of its own width/height so the centre — not the top-left corner — lands at the centre of the parent. This trick predates Flexbox and is still very useful for overlays and modals.

Once Flexbox arrives next class, you will reach for `display: flex; justify-content: center; align-items: center;` instead, and the world will feel a lot easier.

---

## 11.7 Putting it together — a fixed navbar and a back-to-top button

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Layout demo</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <header class="navbar">
      <strong class="navbar__brand">My Site</strong>
      <a href="#" class="navbar__link">Home</a>
      <a href="#" class="navbar__link">About</a>
      <a href="#" class="navbar__link">Contact</a>
    </header>

    <main class="container">
      <h1>Welcome</h1>
      <p>Scroll the page. Notice the navbar stays at the top and a button appears at the bottom-right.</p>

      <p>Filler paragraph. </p>
      <p>Filler paragraph. </p>
      <p>Filler paragraph. </p>
      <p>Filler paragraph. </p>
      <p>Filler paragraph. </p>
      <p>Filler paragraph. </p>
      <p>Filler paragraph. </p>
      <p>Filler paragraph. </p>
      <p>Filler paragraph. </p>
      <p>Filler paragraph. </p>
      <p>Filler paragraph. </p>
      <p>Filler paragraph. </p>
      <p>End of page.</p>
    </main>

    <a href="#" class="back-to-top">↑</a>
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

body {
  margin: 0;
  font-family: system-ui, Arial, sans-serif;
  color: #1f2937;
  background: #f9fafb;
  padding-top: 64px;             /* leaves room for the fixed navbar */
}

/* Fixed navbar */
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 56px;
  padding: 0 24px;
  background: white;
  border-bottom: 1px solid #e5e7eb;
  z-index: 10;
}

.navbar__brand {
  display: inline-block;
  line-height: 56px;             /* vertical centring trick for inline content */
  margin-right: 24px;
}

.navbar__link {
  display: inline-block;
  line-height: 56px;
  margin-right: 16px;
  text-decoration: none;
  color: #374151;
}

.navbar__link:hover {
  color: hsl(210, 80%, 45%);
}

/* Centred page container */
.container {
  max-width: 720px;
  margin: 0 auto;
  padding: 24px 16px;
}

/* Floating back-to-top button */
.back-to-top {
  position: fixed;
  bottom: 24px;
  right: 24px;
  width: 48px;
  height: 48px;
  text-align: center;
  line-height: 48px;
  border-radius: 50%;
  background: hsl(210, 80%, 50%);
  color: white;
  text-decoration: none;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  z-index: 100;
}

.back-to-top:hover {
  background: hsl(210, 80%, 45%);
}
```

Two layout patterns you will reuse forever: a fixed navbar at the top, and a floating button at the bottom-right.

---

## Practice Exercises

1. **Display detective** — Take a page with five `<p>` elements. Set the middle one to `display: inline-block`, the next to `display: inline`, and the last to `display: none`. Predict what you will see, then check.

2. **Badge on a card** — Build a product card. Place a small red badge labelled "SALE" pinned to the top-right corner. Make sure the badge stays in the right place when the card is moved around the page.

3. **Sticky section headers** — Build a page with three sections, each with its own `<h2>`. Make every `<h2>` sticky to the top of the viewport when its section is in view.

4. **Centring lab** — Centre a 200×200 box inside a 600×400 container three different ways:
   - With `margin: 0 auto` (only horizontal centring)
   - With absolute positioning + transform (both axes)
   - Write down which felt easier and why.

5. **Stacking puzzle** — Place three overlapping divs with z-indexes of 1, 2 and 3. Then add `position: relative` to a parent of the middle one. Notice what happens to the stacking order.

---

## Key Takeaways
- `display` decides whether elements stack (`block`) or flow inline (`inline`); `inline-block` gives you both behaviours
- `position: relative` lets you nudge an element *and* anchor children with `absolute`
- `position: fixed` pins to the viewport; `position: sticky` is the modern hybrid
- `overflow` controls what happens to content too big for its box — `hidden`, `auto` and `scroll` are the three you use
- `z-index` only works on positioned elements; mind the stacking context
- These techniques work, but they are clunky. Next class we replace most of them with **Flexbox**.
