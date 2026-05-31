# Class 12: Flexbox

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 11 (`display`, `position`)

## What You Will Learn
- Why Flexbox exists and when to reach for it
- The mental model: **container** vs **items**, **main axis** vs **cross axis**
- Container properties: `display: flex`, `flex-direction`, `justify-content`, `align-items`, `gap`, `flex-wrap`
- Item properties: `flex-grow`, `flex-shrink`, `flex-basis`, the `flex` shorthand, `order`, `align-self`
- Practical patterns: a navbar, a centred card, a row of cards that wraps on small screens

This class is *long* on purpose. Flexbox is the single most useful layout tool in CSS — every modern site uses it. Take the time today and save weeks later.

---

## 12.1 Why Flexbox?

Before Flexbox (which arrived in browsers around 2015), arranging boxes side by side meant clever tricks with `float`, `display: inline-block` and manual `margin` calculations. Vertical centring was a meme: "How do I centre a div?" generated thousands of Stack Overflow questions.

Flexbox solves all of that. Give a parent `display: flex` and its direct children become flexible items that align themselves elegantly.

**Use Flexbox when**:
- Items flow in **one direction** — a row OR a column
- You want **alignment** along that one axis (with optional alignment across)
- You want items to **grow or shrink** to fill (or share) the available space

For two-dimensional grids (rows *and* columns at once), CSS Grid is the better tool — that's next class.

---

## 12.2 The mental model — container, items, axes

Flexbox introduces three new vocabulary words. Learn them now and the rest is mostly remembering which property does what.

- **Flex container** — the parent. You make it a container by writing `display: flex`.
- **Flex items** — the direct children of that container. Grandchildren are not affected.
- **Axes** — every flex container has a **main axis** and a **cross axis**, perpendicular to each other.

### Main vs cross axis (this is the key picture)

When `flex-direction: row` (the default), the main axis runs **horizontally** (left to right). The cross axis runs **vertically**.

```
        ───── main axis ─────►
       ┌──────────────────────────────┐
       │                              │
   │   │  ┌────┐ ┌────┐ ┌────┐        │
 cross │  │ 1  │ │ 2  │ │ 3  │        │
 axis  │  └────┘ └────┘ └────┘        │
   ▼   │                              │
       └──────────────────────────────┘
```

When you flip to `flex-direction: column`, the axes swap. Main is now vertical, cross is horizontal.

```
       │ main │
       │ axis │
       │      ▼
       ┌────────────┐
       │   ┌────┐   │
       │   │ 1  │   │
       │   └────┘   │
   ───►│   ┌────┐   │
   cross│  │ 2  │   │
   axis │  └────┘   │
       │   ┌────┐   │
       │   │ 3  │   │
       │   └────┘   │
       └────────────┘
```

`justify-content` always aligns along the **main** axis. `align-items` always aligns along the **cross** axis. Remember that and half the battle is won.

---

## 12.3 Making a flex container

```html
<div class="row">
  <div class="box">1</div>
  <div class="box">2</div>
  <div class="box">3</div>
</div>
```

```css
.row {
  display: flex;
  background: #eef2f7;
  padding: 10px;
}

.box {
  width: 80px;
  height: 80px;
  background: cornflowerblue;
  color: white;
  font-size: 28px;
  text-align: center;
  line-height: 80px;
  margin: 4px;
}
```

Open it. The three boxes line up horizontally. Without `display: flex`, they would stack vertically (because `<div>` is `block` by default). That single line changed everything.

---

## 12.4 `flex-direction` — which way is the main axis?

```css
.row { flex-direction: row; }            /* default — left to right */
.row { flex-direction: row-reverse; }    /* right to left */
.row { flex-direction: column; }         /* top to bottom */
.row { flex-direction: column-reverse; } /* bottom to top */
```

Use `column` for vertically stacked layouts (sidebars, mobile menus, lists where you want extra flex alignment power).

---

## 12.5 `justify-content` — alignment on the **main** axis

This is the most-used flex property. It controls how items are spread out along the main axis.

| Value | What it does |
|-------|--------------|
| `flex-start` | Pack to the start (default) |
| `flex-end` | Pack to the end |
| `center` | Pack to the centre |
| `space-between` | First item at start, last item at end, equal gaps between |
| `space-around` | Equal space *around* each item (half-gap at the ends) |
| `space-evenly` | Equal space *between* each item, including the ends |

```css
.row { display: flex; justify-content: center; }         /* centred */
.row { display: flex; justify-content: space-between; }  /* navbar pattern */
.row { display: flex; justify-content: space-evenly; }   /* even gallery */
```

Try each value on the demo from 12.3. Five seconds per value — see the difference.

---

## 12.6 `align-items` — alignment on the **cross** axis

```css
.row {
  display: flex;
  height: 200px;            /* gives the cross axis some room */
  align-items: center;
}
```

| Value | What it does |
|-------|--------------|
| `stretch` | Items fill the cross axis (default) |
| `flex-start` | Pack to the top (or left in column mode) |
| `flex-end` | Pack to the bottom |
| `center` | Centre on the cross axis |
| `baseline` | Align by text baseline (useful for typography) |

The famous **centre anything** snippet:

```css
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;          /* one full screen tall */
}
```

A box inside that parent is centred both horizontally and vertically. No tricks, no `transform`, no margins. Compare with the absolute centring from Class 11 and notice how much simpler this is.

---

## 12.7 `gap` — spacing between items

```css
.row {
  display: flex;
  gap: 16px;          /* 16px between every pair of items */
}
```

Before `gap`, we used `margin-right` on each item and removed it from the last one — fiddly. `gap` makes that history.

You can also separate row and column gaps in wrapping or grid layouts:

```css
.row { gap: 24px 16px; }   /* 24px row gap, 16px column gap */
```

---

## 12.8 `flex-wrap` — multi-row layouts

By default, flex items stay on **one line** and shrink to fit. With `flex-wrap`, they jump to a new line when there is no room.

```css
.row {
  display: flex;
  flex-wrap: wrap;     /* allow wrapping */
  gap: 16px;
}
```

The combination of `flex-wrap: wrap` and well-chosen item widths is the easiest way to make a card row that becomes a card grid on small screens. We will use it in section 12.13.

---

## 12.9 Item properties — `flex-grow`, `flex-shrink`, `flex-basis`

So far we have set everything on the container. Now let's look at properties you set on each **item**. These control how each item shares the leftover space.

### `flex-basis` — the starting size

`flex-basis` is the size of the item **before** any growing or shrinking. It's like `width` (or `height` in column mode) but only used inside a flex container.

```css
.item { flex-basis: 200px; }
```

### `flex-grow` — share of the *extra* space

A number — usually 0 or 1. If the row has spare space, items with `flex-grow > 0` grow to fill it. If two items both have `flex-grow: 1`, they share equally. If one has `flex-grow: 2`, it takes twice as much as the others.

```html
<div class="row">
  <div class="item">A</div>
  <div class="item grow">B</div>
  <div class="item">C</div>
</div>
```

```css
.row { display: flex; gap: 8px; }
.item { padding: 16px; background: #e5e7eb; }
.grow { flex-grow: 1; }
```

`B` stretches to fill all the leftover space. `A` and `C` stay at their content width. This is the **navbar trick** — give the middle "spacer" `flex-grow: 1` to push the right-hand items to the edge.

### `flex-shrink` — how willing to shrink

Default `1` — items shrink when there is not enough room. Set to `0` to prevent shrinking.

```css
.icon {
  flex-shrink: 0;     /* keep this 32px icon at exactly 32px */
  width: 32px;
}
```

Stops icons from being squashed when the parent is narrow. A small but useful trick.

### The `flex` shorthand — what you will actually write

In real code you rarely write the three properties separately. The shorthand combines them:

```css
.item { flex: 1; }            /* same as flex: 1 1 0  →  grow, shrink, basis 0 */
.item { flex: 0 0 200px; }    /* don't grow, don't shrink, exactly 200px */
.item { flex: 1 1 25%; }      /* grow & shrink, starting at 25% */
```

| Shorthand | Long form | When |
|-----------|-----------|------|
| `flex: 1` | grow 1, shrink 1, basis 0 | "share the row equally" |
| `flex: 0 0 auto` | fixed at its content size | "do not stretch this item" |
| `flex: 0 0 200px` | fixed at 200px | navbar logo, sidebar |

If you only remember one: `flex: 1` on items to make them share the row evenly.

---

## 12.10 `order` and `align-self`

### `order`

Rearrange items visually without changing the HTML. Lower numbers come first; default is `0`.

```css
.first  { order: -1; }    /* pushed to the start */
.last   { order: 99; }    /* pushed to the end */
```

Handy for mobile-vs-desktop reordering (e.g. show the image first on mobile, second on desktop). Use sparingly — the HTML source order should match the meaning of the page for screen readers.

### `align-self` — override the parent's `align-items` for one item

```css
.row { display: flex; align-items: center; }
.special { align-self: flex-end; }   /* this one sticks to the bottom */
```

---

## 12.11 Property reference — container

| Property | Values | Purpose |
|----------|--------|---------|
| `display` | `flex`, `inline-flex` | Become a flex container |
| `flex-direction` | `row`, `row-reverse`, `column`, `column-reverse` | Direction of the main axis |
| `flex-wrap` | `nowrap`, `wrap`, `wrap-reverse` | Whether items can wrap to a new line |
| `justify-content` | `flex-start`, `flex-end`, `center`, `space-between`, `space-around`, `space-evenly` | Alignment along the main axis |
| `align-items` | `stretch`, `flex-start`, `flex-end`, `center`, `baseline` | Alignment along the cross axis |
| `align-content` | same as above | Alignment of *multiple lines* when wrapped |
| `gap` | length | Spacing between items |

## 12.12 Property reference — items

| Property | Values | Purpose |
|----------|--------|---------|
| `flex-grow` | number | How greedy when sharing extra space |
| `flex-shrink` | number | How willing to shrink |
| `flex-basis` | length / `auto` | Starting size |
| `flex` | shorthand of the above three | What you actually write |
| `order` | integer | Visual order (default `0`) |
| `align-self` | same values as `align-items` | Override for one item |

---

## 12.13 Three practical patterns

### Pattern 1 — The classic navbar

`index.html`:

```html
<header class="navbar">
  <strong class="navbar__brand">My Site</strong>
  <nav class="navbar__nav">
    <a href="#">Home</a>
    <a href="#">Posts</a>
    <a href="#">About</a>
  </nav>
  <button class="navbar__cta">Sign in</button>
</header>
```

`style.css`:

```css
*, *::before, *::after { box-sizing: border-box; }
body { margin: 0; font-family: system-ui, Arial, sans-serif; }

.navbar {
  display: flex;
  align-items: center;        /* vertically centre everything */
  gap: 24px;
  padding: 12px 24px;
  background: white;
  border-bottom: 1px solid #e5e7eb;
}

.navbar__brand {
  font-size: 18px;
}

.navbar__nav {
  display: flex;
  gap: 16px;
  flex-grow: 1;               /* take the spare room — pushes the button right */
}

.navbar__nav a {
  text-decoration: none;
  color: #374151;
}

.navbar__cta {
  padding: 8px 16px;
  background: hsl(210, 80%, 50%);
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
```

The brand, the links, and the sign-in button all sit on one row, vertically centred, with the button pushed to the right by `flex-grow: 1` on the nav.

### Pattern 2 — Centring anything

```html
<main class="screen">
  <div class="card">
    <h2>Welcome</h2>
    <p>This box is perfectly centred — both axes — using three lines of CSS.</p>
  </div>
</main>
```

```css
.screen {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: #f3f4f6;
}

.card {
  width: 320px;
  padding: 24px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  text-align: center;
}
```

Three CSS properties solve a problem that gave us nightmares for fifteen years. This is why Flexbox is loved.

### Pattern 3 — A row of cards that wraps on small screens

```html
<section class="cards">
  <article class="card">Card 1</article>
  <article class="card">Card 2</article>
  <article class="card">Card 3</article>
</section>
```

```css
.cards {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  padding: 16px;
}

.card {
  flex: 1 1 250px;            /* grow, shrink, starting basis 250px */
  background: white;
  padding: 24px;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);
}
```

What that `flex: 1 1 250px` does:
- Each card wants to be at least **250px** wide
- Each card can **grow** to fill the row
- Each card can **shrink** if the row is narrow
- When there is not enough room for three across, they **wrap** to a new line

Resize the browser window. On a wide screen all three cards sit on one row. As you drag narrower, they snap into two rows of two-plus-one, then finally a single column. Zero media queries. This pattern alone covers maybe 60% of real layout problems.

---

## 12.14 Common pitfalls

- **Vertical alignment not working?** The container needs height. `align-items: center` aligns items within whatever vertical room the container has. If the container is only as tall as the items, there is nothing to centre.
- **Items overflowing instead of wrapping?** You forgot `flex-wrap: wrap`. The default is `nowrap`.
- **Margins not pushing things apart correctly?** Use `gap` instead. Modern, clean, no edge cases.
- **`flex: 1` on a button is making it too wide?** That's exactly what `flex: 1` does. Switch to `flex: 0 0 auto`.
- **A child of a flex item is not aligned the way you want?** Flexbox only affects *direct* children. Make the flex item itself a flex container.

---

## Practice Exercises

1. **Centring lab (redux)** — Recreate the absolute-centring exercise from Class 11 using Flexbox. Note how much less code you need.

2. **Navbar** — Build a responsive navbar with: a logo on the left, three links in the middle, a "Sign up" button on the right. Use `gap` for spacing and `flex-grow` to push the button to the right.

3. **Card row** — Build three identical cards in a row, each with a title and a description. Each card should be at least 280px wide. Make them wrap to a new line on small screens without a single media query.

4. **Vertical list** — Build a vertical list of three items where each item has an icon on the left, a title in the middle that fills the row, and a "More" button on the right. (Hint: nested flex containers.)

5. **Order shuffle** — Make three flex items. Without changing the HTML, use `order` to display them in reverse. Then make the second one appear first.

---

## Key Takeaways
- `display: flex` makes the **parent** a flex container; its direct children become flex items
- Main axis = direction items flow; cross axis = perpendicular
- `justify-content` aligns on the main axis, `align-items` on the cross axis
- `flex: 1` on items makes them share the row; `flex: 0 0 auto` keeps them at their natural size
- `flex-wrap: wrap` plus a flex-basis is the secret to responsive layouts without media queries
- Next class: **CSS Grid** — when you need rows *and* columns at once.
