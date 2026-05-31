# Class 9: The Box Model, Colours, Backgrounds & Borders

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 8 (you can write selectors and link a stylesheet)

## What You Will Learn
- Every element on a page is a **box** — content, padding, border, margin
- Why `box-sizing: border-box` is the single most important CSS rule we will set
- Four ways to specify a colour: named, hex, RGB, HSL
- Backgrounds: colour, image, size, position
- Borders, `border-radius`, and `box-shadow`

---

## 9.1 Every element is a box

Open any web page and the browser sees nothing but rectangles stacked next to each other. Even text is wrapped in invisible rectangles. This is the **box model**.

Each box has four layers, working outwards from the centre:

```
┌───────────────────────────────────────┐
│              MARGIN                    │  ← space outside the box
│   ┌──────────────────────────────┐    │
│   │           BORDER              │    │  ← the visible edge
│   │   ┌──────────────────────┐   │    │
│   │   │       PADDING         │   │    │  ← space inside, around the content
│   │   │   ┌──────────────┐   │   │    │
│   │   │   │   CONTENT     │   │   │    │  ← the text or image
│   │   │   └──────────────┘   │   │    │
│   │   └──────────────────────┘   │    │
│   └──────────────────────────────┘    │
└───────────────────────────────────────┘
```

A real-world analogy: think of a photo in a frame on a wall.
- **Content** = the photograph itself
- **Padding** = the cream-coloured mount around the photo, inside the frame
- **Border** = the wooden frame
- **Margin** = the empty wall space between this frame and the next picture

Each layer has its own CSS properties. Let's see them.

---

## 9.2 The four layers in code

```html
<div class="box">Hello box model</div>
```

```css
.box {
  width: 200px;
  height: 100px;

  padding: 20px;                /* inside space */
  border: 4px solid steelblue;  /* the edge */
  margin: 30px;                 /* outside space */

  background-color: lightyellow;
}
```

Open this in the browser, right-click the box and choose **Inspect**, then look at the **Computed** or **Box Model** panel in DevTools. You will see the four coloured layers drawn out — orange for margin, beige for padding, etc.

### Shorthand for `padding` and `margin`

You can give all four sides at once, or specify each side individually.

```css
/* All four sides the same */
padding: 16px;

/* Two values: top/bottom | left/right */
padding: 16px 24px;

/* Three values: top | left/right | bottom */
padding: 10px 24px 20px;

/* Four values: top | right | bottom | left (clockwise from the top) */
padding: 10px 20px 30px 40px;

/* Per-side properties (also valid) */
padding-top: 10px;
padding-right: 20px;
padding-bottom: 30px;
padding-left: 40px;
```

The same rules apply to `margin`.

**Mnemonic for four values**: think of a clock starting at 12 — top, right, bottom, left.

### Margin collapse — a small surprise

If two adjacent vertical margins meet, they do not add up — they **collapse** to the larger of the two. So a paragraph with `margin-bottom: 20px` next to one with `margin-top: 30px` will sit 30px apart, not 50px. This only happens vertically, not horizontally. Just be aware of it.

---

## 9.3 `box-sizing: border-box` — the most important global rule

There is one very confusing default in CSS that we will fix on day one of every project.

By default, `width: 200px` only refers to the **content**. Padding and border are added on top. So a box with `width: 200px`, `padding: 20px` and `border: 4px solid` is actually **248px wide** on screen (200 + 20 + 20 + 4 + 4). This makes laying out grids painful.

The fix is to switch to the `border-box` sizing mode:

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

Now `width: 200px` means **the box is 200px including padding and border**. Content gets whatever space is left over. Far more predictable.

We will put this rule at the top of every stylesheet from now on. It is so common that frameworks like Tailwind apply it by default.

A quick demo:

```html
<div class="box content">content-box</div>
<div class="box border">border-box</div>
```

```css
.box {
  width: 200px;
  padding: 20px;
  border: 4px solid steelblue;
  margin-bottom: 10px;
  background-color: lightyellow;
}

.content {
  box-sizing: content-box;   /* default — total width 248px */
}

.border {
  box-sizing: border-box;    /* total width 200px */
}
```

Open it and measure with DevTools. The two boxes will be visibly different widths.

---

## 9.4 Colours — four ways to specify the same thing

CSS accepts colours in several formats. All of them are valid — pick whichever is clearest for the situation.

### Named colours

There are around 140 named colours. Great for quick demos, useless for brand work because there's no `corBrandBlue`.

```css
color: red;
color: tomato;
color: cornflowerblue;
color: darkslategrey;
```

### Hex codes — `#rrggbb`

Two hex digits each for red, green, blue. Each pair runs `00`–`ff` (0–255). This is what most designers will hand you.

```css
color: #ff0000;   /* pure red */
color: #00ff00;   /* pure green */
color: #0000ff;   /* pure blue */
color: #333333;   /* dark grey */
color: #ffffff;   /* white */
color: #000000;   /* black */
```

If both digits in a pair are the same, you can use the three-digit shorthand:

```css
color: #333;      /* same as #333333 */
color: #fff;      /* same as #ffffff */
color: #f00;      /* same as #ff0000 */
```

### RGB and RGBA

Same idea, but in decimal (0–255). RGBA adds an **alpha** channel (0 = transparent, 1 = opaque).

```css
color: rgb(255, 0, 0);            /* red */
color: rgb(51, 51, 51);           /* dark grey */
background-color: rgba(0, 0, 0, 0.5);  /* black at 50% opacity */
```

Use RGBA whenever you want a see-through colour — e.g. an overlay on an image.

### HSL and HSLA — the designer-friendly format

HSL stands for **Hue, Saturation, Lightness**. Once you understand it, building a palette becomes much easier.

- **Hue**: 0–360, the position on a colour wheel (0 = red, 120 = green, 240 = blue)
- **Saturation**: 0% (grey) to 100% (fully vivid)
- **Lightness**: 0% (black) to 100% (white); 50% is the pure colour

```css
color: hsl(0, 100%, 50%);     /* red */
color: hsl(210, 60%, 50%);    /* steel blue */
color: hsl(210, 60%, 80%);    /* the same blue, but pale */
color: hsl(210, 60%, 20%);    /* the same blue, but dark */
```

Want a darker version of your brand colour? Reduce the lightness. Want a muted version? Reduce the saturation. Try doing that with hex codes — you cannot.

### Which to use?

For this course, mostly **hex** because that's what designers send. Reach for **HSL** when you are building a palette and **RGBA** when you need transparency.

---

## 9.5 Backgrounds

A box can have a background colour, a background image, or both.

### `background-color`

```css
.card {
  background-color: #f5f7fa;
}
```

### `background-image` with `background-size` and `background-position`

```css
.hero {
  width: 100%;
  height: 400px;

  background-image: url("hero.jpg");
  background-size: cover;        /* fills the box, may crop */
  background-position: center;   /* centre the image */
  background-repeat: no-repeat;  /* do not tile */
}
```

`background-size` values you will use most:

| Value | What it does |
|-------|--------------|
| `cover` | Fills the box completely; some image may be cropped |
| `contain` | Fits the whole image inside the box; may leave gaps |
| `100% 100%` | Stretches the image to the exact box size (can distort) |
| `200px 100px` | Sets a fixed width and height for the image |

### Shorthand

You can combine the lot into one `background` declaration:

```css
.hero {
  background: #1a1a2e url("hero.jpg") center / cover no-repeat;
}
```

That reads: **dark background colour, image centred, sized to cover, no repeat**.

### Gradients — backgrounds without images

Gradients are images you generate from CSS. No file needed.

```css
.banner {
  height: 200px;
  background: linear-gradient(to right, #4a90d9, #845ec2);
}

.button {
  background: linear-gradient(180deg, #6db4ff 0%, #2c79d3 100%);
}
```

`linear-gradient(direction, from-colour, to-colour)`. Direction can be a keyword (`to right`, `to bottom right`) or an angle (`45deg`, `180deg`).

There is also `radial-gradient(...)` for circular gradients — same idea, circular spread instead of linear.

---

## 9.6 Borders

A border is the line around the box. It has three properties: width, style, colour.

```css
.box {
  border-width: 2px;
  border-style: solid;
  border-color: steelblue;
}
```

Or, much more commonly, the shorthand:

```css
.box {
  border: 2px solid steelblue;
}
```

### Border styles

| Style | Looks like |
|-------|-----------|
| `solid` | A solid line (almost always what you want) |
| `dashed` | Dashes |
| `dotted` | Dots |
| `double` | Two parallel lines |
| `none` | No border |

### Per-side borders

```css
.card {
  border: 1px solid #ddd;             /* all four sides */
  border-bottom: 3px solid steelblue; /* override the bottom only */
}
```

A common pattern: a faint border all round, with a bold accent border on the left.

```css
.alert {
  background-color: #fff8e1;
  padding: 12px 16px;
  border: 1px solid #ffe082;
  border-left: 6px solid #fbc02d;  /* the accent stripe */
}
```

### `border-radius` — rounded corners

```css
.button {
  border-radius: 6px;       /* soft rounded corners */
}

.tag {
  border-radius: 999px;     /* pill shape */
}

.avatar {
  width: 80px;
  height: 80px;
  border-radius: 50%;       /* perfect circle (when width === height) */
}
```

You can also round each corner individually:

```css
border-top-left-radius: 12px;
border-top-right-radius: 12px;
border-bottom-left-radius: 0;
border-bottom-right-radius: 0;
```

---

## 9.7 `box-shadow` — depth without images

Box shadows give depth and lift to your UI. The syntax looks scary but boils down to **offsetX offsetY blur spread colour**.

```css
.card {
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
```

That reads:
- `0` — no horizontal offset
- `2px` — shadow falls 2px down
- `4px` — blur radius (softness)
- `rgba(0, 0, 0, 0.1)` — black at 10% opacity

A subtle shadow like this is the workhorse of modern UI design — every card on every web page uses some variant of it.

A few common presets to copy:

```css
/* Subtle — for cards on white pages */
.shadow-sm { box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05); }

/* Medium — for raised cards */
.shadow-md { box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); }

/* Large — for floating menus and modals */
.shadow-lg { box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15); }
```

You can also use shadows *inside* the box with the `inset` keyword:

```css
.input:focus {
  box-shadow: inset 0 0 0 2px steelblue;
}
```

---

## 9.8 Putting it together — a profile card

Let's build a single styled card using everything from this class.

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Profile card</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <article class="profile-card">
      <div class="profile-card__avatar"></div>
      <h2 class="profile-card__name">Aarav Sharma</h2>
      <p class="profile-card__role">Frontend Developer</p>
      <p class="profile-card__bio">
        Loves React, clean CSS, and a good cup of milk tea.
      </p>
      <button class="profile-card__button">Follow</button>
    </article>
  </body>
</html>
```

`style.css`:

```css
/* The single most important rule */
*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Arial, sans-serif;
  background-color: #f0f3f7;
  padding: 40px;
}

.profile-card {
  width: 280px;
  margin: 0 auto;            /* centres horizontally */
  padding: 24px;

  background-color: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);

  text-align: center;
}

.profile-card__avatar {
  width: 96px;
  height: 96px;
  margin: 0 auto 16px;
  border-radius: 50%;
  background: linear-gradient(135deg, #6db4ff, #845ec2);
}

.profile-card__name {
  margin: 0 0 4px;
  font-size: 20px;
  color: #1f2937;
}

.profile-card__role {
  margin: 0 0 12px;
  color: #6b7280;
  font-size: 14px;
}

.profile-card__bio {
  margin: 0 0 20px;
  color: #374151;
  line-height: 1.5;
}

.profile-card__button {
  padding: 10px 24px;
  background-color: hsl(210, 80%, 50%);
  color: #ffffff;
  border: none;
  border-radius: 999px;       /* pill */
  font-size: 14px;
  cursor: pointer;
  box-shadow: 0 2px 6px rgba(74, 144, 217, 0.4);
}

.profile-card__button:hover {
  background-color: hsl(210, 80%, 45%);   /* slightly darker */
}
```

That single card uses **every** concept from this class: the global reset, the box model, three colour formats, a gradient background, rounded corners, a soft shadow, and a hover state.

---

## Practice Exercises

1. **Box model sketch** — Make a div with `width: 200px`, `padding: 20px`, `border: 5px solid red`, `margin: 30px`. Use DevTools to confirm the total space it occupies. Then add `box-sizing: border-box` and observe the change.

2. **Colour palette** — Pick a base hue and produce five shades of it using HSL, by varying only the lightness. For example: `hsl(160, 60%, 95%)`, `hsl(160, 60%, 80%)`, `hsl(160, 60%, 60%)`, `hsl(160, 60%, 40%)`, `hsl(160, 60%, 20%)`. Apply each to a row of divs and look at the result.

3. **Card challenge** — Build a "blog post card" with:
   - A coloured top stripe (`border-top: 4px solid`)
   - A title and short description
   - A subtle shadow that grows slightly on `:hover`
   - Rounded corners
   - A white background on a pale grey page

4. **Shadow study** — Copy the three preset shadows from section 9.7 onto three identical boxes side by side. Notice how subtle differences in blur and offset change the feeling from "flat" to "floating".

---

## Key Takeaways
- Every element is a **box** with content, padding, border and margin
- Always set `box-sizing: border-box` globally — your sanity depends on it
- Pick colours with **hex** when matching a design, **HSL** when building a palette, **RGBA** for transparency
- `background` and `border` shorthands are concise and readable once you know the order
- `box-shadow` adds depth — subtle shadows make a flat page feel modern
- Next class: typography — the fonts, sizes and units that make text readable and beautiful.
