# Class 13: CSS Grid

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 12 (Flexbox)

## What You Will Learn
- When to reach for Grid versus Flexbox
- `display: grid` plus `grid-template-columns` / `grid-template-rows`
- `gap`, `fr` units, `repeat()`, `minmax()`, `auto-fit` vs `auto-fill`
- Placing items on the grid with line numbers and named areas
- Building a responsive gallery in 10 lines of CSS

CSS Grid is the second of the two big modern layout tools. With Flexbox and Grid in your toolbox, you can build almost any layout cleanly.

---

## 13.1 Flexbox vs Grid — when to use which?

The simplest way to choose:

| Use Flexbox when | Use Grid when |
|------------------|---------------|
| Items flow in **one direction** (row OR column) | You need **rows AND columns** at the same time |
| Items have varying sizes that should adapt | You want **strict alignment** across both axes |
| You are building a navbar, a toolbar, a list of tags | You are building a page layout, a dashboard, a gallery |
| You think "how should these items share this row?" | You think "I want a 3-column grid, with this item spanning two columns" |

They are not rivals — many real layouts use both. A common pattern: Grid lays out the page (header, sidebar, main, footer); Flexbox lays out the contents *inside* each region.

---

## 13.2 Making a grid container

```html
<div class="grid">
  <div class="box">1</div>
  <div class="box">2</div>
  <div class="box">3</div>
  <div class="box">4</div>
  <div class="box">5</div>
  <div class="box">6</div>
</div>
```

```css
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;   /* three equal columns */
  gap: 10px;
}

.box {
  background: cornflowerblue;
  color: white;
  font-size: 24px;
  text-align: center;
  padding: 24px 0;
}
```

Open it. You get a tidy 3-column grid with two rows. The browser figured out the row count for you — six items in three columns = two rows.

That's the magic: with Grid, you declare the **column structure** and the browser arranges the items.

---

## 13.3 `fr` — the fraction unit

`1fr` means "one share of the leftover space". It only exists inside Grid (and Flexbox shorthand). Some examples:

```css
grid-template-columns: 1fr 1fr 1fr;       /* three equal columns */
grid-template-columns: 1fr 2fr;           /* second column is twice as wide */
grid-template-columns: 200px 1fr;         /* fixed sidebar, fluid main column */
grid-template-columns: 200px 1fr 200px;   /* two sidebars and a main column */
```

You can mix `fr`, `px`, `%`, `rem` and `auto` freely. The `fr` units share whatever space is left over after the fixed sizes are taken out. This is **exactly** how design tools and real dashboards lay things out.

### `repeat()` — stop typing `1fr 1fr 1fr 1fr 1fr`

```css
grid-template-columns: repeat(5, 1fr);    /* five equal columns */
grid-template-columns: repeat(4, 100px);  /* four 100px columns */
```

Use `repeat()` any time you have more than two of the same.

---

## 13.4 `grid-template-rows` — when row sizes matter

By default, rows are as tall as their content. You can set explicit row sizes too:

```css
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  grid-template-rows: 100px 200px;
  gap: 10px;
}
```

Now the first row is 100px tall, the second is 200px. Any extra rows (if more items wrap in) take their content height.

---

## 13.5 `gap` — spacing without margins

Just like in Flexbox:

```css
.grid {
  gap: 16px;             /* same for rows and columns */
  gap: 24px 16px;        /* row-gap | column-gap */
  row-gap: 24px;         /* or set them separately */
  column-gap: 16px;
}
```

No more "I'll put margin-right on every item except the last".

---

## 13.6 `minmax()` — flexible columns with limits

`minmax(min, max)` says: this column should be at least `min` wide and at most `max` wide. The single most useful trick for responsive design without media queries.

```css
.grid {
  display: grid;
  grid-template-columns: 1fr minmax(200px, 800px) 1fr;
}
```

That defines a centred middle column that grows up to 800px, with two flexible gutters on either side. Resize the window — the middle stays bounded, the gutters absorb the change.

---

## 13.7 `auto-fit` and `auto-fill` — the responsive magic line

This is the trick everyone shares when they first discover Grid. Combine `repeat()`, `auto-fit` (or `auto-fill`) and `minmax()`:

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 16px;
}
```

Read it as: **fit as many columns as you can; each column must be at least 220px wide; let the rest share equally**.

Drop in any number of cards. On a wide desktop they form a 5- or 6-column grid. On a phone they become a single column. No media queries. No JavaScript. One line.

### `auto-fit` vs `auto-fill`

The difference matters only when there are not enough items to fill the row.

- `auto-fit` — empty tracks **collapse**, leaving fewer wider columns
- `auto-fill` — empty tracks are **preserved**, leaving blank spaces

```css
.gallery-fit  { grid-template-columns: repeat(auto-fit,  minmax(220px, 1fr)); }
.gallery-fill { grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); }
```

Most of the time you want `auto-fit` — items spread out to fill the row. Use `auto-fill` only if you specifically want consistent column widths even when half empty.

---

## 13.8 Placing items — line numbers and `span`

Grid lines are numbered starting at 1. For a 3-column grid you have lines 1, 2, 3, 4.

```
   1     2     3     4
   │     │     │     │
   ├─────┼─────┼─────┤
   │ A   │ B   │ C   │
   ├─────┼─────┼─────┤
   │ D   │ E   │ F   │
```

Tell an item which lines to start and end at:

```css
.feature {
  grid-column-start: 1;
  grid-column-end: 3;       /* spans columns 1 and 2 (lines 1 → 3) */
}
```

Or, much shorter:

```css
.feature {
  grid-column: 1 / 3;        /* shorthand: start / end */
  grid-row: 1 / 3;           /* rows 1 and 2 */
}
```

Or using `span`:

```css
.feature {
  grid-column: span 2;       /* take two columns from wherever I land */
}
```

A common dashboard layout — first card spans two columns to feel important:

```html
<section class="dashboard">
  <div class="card big">Featured</div>
  <div class="card">Stat A</div>
  <div class="card">Stat B</div>
  <div class="card">Stat C</div>
  <div class="card">Stat D</div>
</section>
```

```css
.dashboard {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}

.card {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 24px;
}

.big {
  grid-column: span 2;       /* takes the first two columns */
  background: hsl(210, 80%, 95%);
}
```

---

## 13.9 Named areas — `grid-template-areas`

For larger page layouts, naming areas reads like ASCII art. You assign each region a name, then "draw" the layout in the parent.

```html
<div class="page">
  <header class="page__header">Header</header>
  <nav    class="page__nav">Nav</nav>
  <main   class="page__main">Main</main>
  <aside  class="page__aside">Aside</aside>
  <footer class="page__footer">Footer</footer>
</div>
```

```css
.page {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  gap: 12px;
  min-height: 100vh;

  grid-template-areas:
    "header header header"
    "nav    main   aside"
    "footer footer footer";
}

.page__header { grid-area: header; background: #1f2937; color: white; padding: 12px; }
.page__nav    { grid-area: nav;    background: #f3f4f6; padding: 12px; }
.page__main   { grid-area: main;   background: white;   padding: 12px; }
.page__aside  { grid-area: aside;  background: #f3f4f6; padding: 12px; }
.page__footer { grid-area: footer; background: #1f2937; color: white; padding: 12px; text-align: center; }
```

Read the `grid-template-areas` block top to bottom — that **is** the page. Repeating a name across cells (`header header header`) makes that area span them. Use a `.` (dot) for an empty cell.

This is the cleanest way to build a classic "holy grail" layout (header / nav / main / aside / footer) without thinking about row and column numbers.

---

## 13.10 A responsive gallery — the whole thing

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Gallery</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <h1 class="page-title">Photo gallery</h1>

    <section class="gallery">
      <figure class="card"><div class="thumb"></div><figcaption>Sunrise</figcaption></figure>
      <figure class="card"><div class="thumb"></div><figcaption>Mountain</figcaption></figure>
      <figure class="card"><div class="thumb"></div><figcaption>River</figcaption></figure>
      <figure class="card"><div class="thumb"></div><figcaption>Forest</figcaption></figure>
      <figure class="card"><div class="thumb"></div><figcaption>City</figcaption></figure>
      <figure class="card"><div class="thumb"></div><figcaption>Desert</figcaption></figure>
      <figure class="card"><div class="thumb"></div><figcaption>Ocean</figcaption></figure>
      <figure class="card"><div class="thumb"></div><figcaption>Lake</figcaption></figure>
    </section>
  </body>
</html>
```

`style.css`:

```css
*, *::before, *::after { box-sizing: border-box; }
body { margin: 0; font-family: system-ui, Arial, sans-serif; background: #f9fafb; color: #1f2937; }

.page-title {
  max-width: 1100px;
  margin: 32px auto 16px;
  padding: 0 16px;
}

.gallery {
  max-width: 1100px;
  margin: 0 auto;
  padding: 16px;

  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 20px;
}

.card {
  margin: 0;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);
}

.thumb {
  height: 160px;
  background: linear-gradient(135deg, hsl(210, 80%, 60%), hsl(280, 60%, 60%));
}

figcaption {
  padding: 12px 16px;
  font-weight: 600;
}
```

That single `grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));` does all the responsive work. Drag the browser narrower — the gallery becomes 3 columns, then 2, then 1. No media queries needed for the layout.

---

## 13.11 Property reference

### Container

| Property | Purpose |
|----------|---------|
| `display: grid` | Become a grid container |
| `grid-template-columns` | Define column tracks |
| `grid-template-rows` | Define row tracks |
| `gap`, `row-gap`, `column-gap` | Space between tracks |
| `grid-template-areas` | Name the cells for placement |
| `justify-items` | Horizontal alignment within each cell |
| `align-items` | Vertical alignment within each cell |
| `justify-content` | Distribute the whole grid horizontally (when there is leftover room) |
| `align-content` | Same vertically |

### Items

| Property | Purpose |
|----------|---------|
| `grid-column` | Shorthand for start / end column lines (`1 / 3` or `span 2`) |
| `grid-row` | Same for rows |
| `grid-area` | Place by area name or `row-start / col-start / row-end / col-end` |
| `justify-self` | Override horizontal alignment for this item |
| `align-self` | Override vertical alignment for this item |

---

## 13.12 Common pitfalls

- **Track sizes wrong?** Remember: `fr` is leftover space, not total space. `1fr 2fr` is "one share" and "two shares" of whatever is left after fixed columns.
- **Items overflow the cell?** Add `min-width: 0` to the item. Grid items have an implicit minimum of `auto` (their content), which can blow past the column width.
- **`grid-template-areas` not working?** Each row must have the same number of cells; names with spaces in them don't work; all names must be defined as `grid-area` on the children.
- **Mixing it up with Flexbox?** That's fine and normal. Use Grid for the big-picture page layout, Flexbox for the alignment *inside* each grid cell.

---

## Practice Exercises

1. **Three columns** — Build a 3-column grid with `repeat(3, 1fr)` and a 16px gap. Drop six div boxes into it. Confirm you get two rows.

2. **Sidebar layout** — Build a layout with a fixed 240px sidebar on the left and a flexible main area on the right (`grid-template-columns: 240px 1fr`). Add a 16px gap. Try resizing — only the main area should change width.

3. **Responsive gallery** — Recreate the gallery in section 13.10 with your own colour and content. Confirm by resizing that the number of columns adapts.

4. **Holy grail layout** — Use `grid-template-areas` to build the classic header / nav / main / aside / footer layout. The header and footer should each span all three columns. Make the layout collapse to a single column on mobile by overriding the areas in a media query (preview of Class 14).

5. **Feature card** — In a 3-column grid of six cards, make the first card span two columns and two rows so it stands out as the "featured" item.

---

## Key Takeaways
- Grid is for **two-dimensional** layouts — rows and columns together
- `grid-template-columns: 1fr 1fr 1fr` (or `repeat(3, 1fr)`) is your starting point
- `repeat(auto-fit, minmax(220px, 1fr))` is the **responsive one-liner** you will reach for again and again
- Use `grid-column: span N` or `grid-area` to place items deliberately
- `grid-template-areas` reads like ASCII art and is great for whole-page layouts
- Next class: media queries — making the design adapt to phones, tablets and desktops on purpose.
