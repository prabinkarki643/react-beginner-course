# Class 8: CSS Introduction, Syntax & Selectors

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Classes 2–7 (you can write a valid HTML page)

## What You Will Learn
- What CSS is and why it exists as a separate language
- Three ways to add CSS to a page, and which one we should prefer
- The basic CSS rule: `selector { property: value; }`
- The core selectors: element, class, ID, group, descendant, child, attribute
- Pseudo-classes: `:hover`, `:focus`, `:nth-child`
- A first taste of **specificity** — who wins when two rules clash

---

## 8.1 What is CSS and why do we need it?

You have spent six classes writing HTML. Open any of those pages in a browser and you will see plain black Times-New-Roman text on a white page. It works, but it looks like a Word document from 1998. The page has no colour, no layout, no personality.

**CSS (Cascading Style Sheets)** is the language we use to *style* HTML. If HTML is the skeleton of a page, CSS is the clothes, the hair, the make-up. HTML decides *what* is on the page; CSS decides *how it looks*.

A useful split to remember:

| Language | Job | Real-world analogy |
|----------|-----|--------------------|
| HTML | Structure (what is there) | The skeleton of a body |
| CSS  | Appearance (how it looks) | The skin, hair, clothes |
| JavaScript | Behaviour (what it does) | The muscles and brain |

CSS is called **Cascading** because many rules can apply to the same element, and the browser has a clear order of priority for deciding which one wins. We will see this in section 8.7.

---

## 8.2 Three ways to add CSS to a page

There are three places you can write CSS. Two of them are convenient for a five-second test; one is what real projects use.

### Way 1 — Inline (on the element itself)

```html
<h1 style="color: blue;">Hello</h1>
```

This works, but the styling is glued to that single element. If you have 50 headings, you would have to repeat the rule 50 times. Avoid it.

### Way 2 — Internal (a `<style>` block in the `<head>`)

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Internal CSS</title>
    <style>
      h1 {
        color: blue;
      }
    </style>
  </head>
  <body>
    <h1>Hello</h1>
  </body>
</html>
```

Better — the rule lives in one place. Useful for a tiny demo. The downside: every page in your site has to repeat the whole `<style>` block.

### Way 3 — External stylesheet (a separate `.css` file) — preferred

Create two files in the same folder.

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>External CSS</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <h1>Hello</h1>
    <p>This paragraph is styled by an external file.</p>
  </body>
</html>
```

`style.css`:

```css
h1 {
  color: blue;
}

p {
  color: dimgrey;
  font-size: 18px;
}
```

Open `index.html` with Live Server. You should see a blue heading and grey body text.

**Why external wins**:
- One file styles every page in the site
- Browsers cache the file, so pages load faster
- Designers and developers can work on `.css` and `.html` separately
- It is what every real project does — that is what we are here to learn

From Class 9 onwards we will always use an external `style.css`.

---

## 8.3 Anatomy of a CSS rule

A CSS rule has four parts. Learn this vocabulary now — we will use it every class.

```css
h1 {
  color: navy;
  font-size: 24px;
}
```

- **Selector** — `h1` — tells CSS *which* elements to target
- **Declaration block** — everything between `{` and `}`
- **Property** — `color`, `font-size` — the thing you want to change
- **Value** — `navy`, `24px` — what you want to change it to

Every declaration ends with a semicolon `;`. Technically the last one is optional, but always include it — it saves you from a hard-to-find bug later when you add a new line.

### Comments

Use `/* ... */` for comments. Comments can span one line or many lines.

```css
/* This is a one-line comment */

/*
  This is a longer comment.
  Use it to explain WHY something is styled this way,
  not WHAT the rule does (the code already shows that).
*/

h1 {
  color: navy; /* brand colour from the style guide */
}
```

---

## 8.4 The basic selectors

A selector says *which* elements to style. Here are the five you will use 95% of the time.

### Element selector

Targets every element of that tag name.

```html
<h1>About me</h1>
<p>I am learning CSS.</p>
<p>It is going well.</p>
```

```css
h1 {
  color: darkblue;
}

p {
  color: #333333;
  line-height: 1.6;
}
```

Every `<h1>` becomes dark blue, every `<p>` becomes dark grey.

### Class selector — the workhorse

Targets every element that has a particular `class` attribute. In CSS we write a `.` before the name.

```html
<p class="lead">This is the introduction paragraph.</p>
<p>This is a normal paragraph.</p>
<p class="lead">Another introduction.</p>
```

```css
.lead {
  font-size: 20px;
  font-weight: bold;
  color: #111111;
}
```

Both paragraphs with `class="lead"` get the style. The normal one is untouched. You can re-use the same class on as many elements as you like — that is the point.

An element can also carry **multiple classes** separated by spaces:

```html
<button class="btn btn-primary">Save</button>
```

`btn` could style every button the same way (padding, border radius), and `btn-primary` could add a brand colour. This stack-up pattern is how almost every CSS framework (including Tailwind) works.

### ID selector

Targets exactly *one* unique element. In HTML each `id` value must appear only once per page. In CSS we write `#` before the name.

```html
<header id="top">
  <h1>Welcome</h1>
</header>
```

```css
#top {
  background-color: lightyellow;
  padding: 20px;
}
```

**Rule of thumb**: prefer classes for styling. Reserve IDs for things that are genuinely unique on the page (e.g. anchor targets like `id="contact"`) or for JavaScript hooks.

### Group selector — share styles across many selectors

If several selectors share the same styles, list them with commas.

```css
h1, h2, h3 {
  font-family: Georgia, serif;
  color: #222222;
}
```

This is exactly the same as writing three separate rules — just shorter.

### Universal selector

`*` matches *every* element. Use sparingly — in modern CSS you will mostly see it for the global reset we cover in Class 9.

```css
* {
  margin: 0;
  padding: 0;
}
```

---

## 8.5 Combinator selectors — selectors based on relationships

The selectors above target elements directly. Combinators target elements based on where they sit relative to other elements.

### Descendant selector (a space)

`A B` means *any `B` anywhere inside an `A`*, no matter how deeply nested.

```html
<article>
  <p>Inside an article.</p>
  <div>
    <p>Also inside an article (deeper).</p>
  </div>
</article>
<p>Outside the article.</p>
```

```css
article p {
  color: darkgreen;
}
```

Both paragraphs *inside* the `<article>` become dark green. The last one (outside) is unaffected.

### Child selector (`>`)

`A > B` means *only direct children*. Skips grandchildren.

```css
article > p {
  color: crimson;
}
```

With the same HTML above, only the first paragraph (a direct child of `<article>`) becomes crimson. The one nested inside `<div>` is not a direct child and is ignored.

### Attribute selector (`[attr]`)

Match elements based on their attributes. Very handy for forms.

```html
<input type="text" placeholder="Name" />
<input type="email" placeholder="Email" />
<input type="password" placeholder="Password" />
```

```css
/* every input with type="email" */
input[type="email"] {
  border: 2px solid steelblue;
}

/* any element that has a `disabled` attribute, regardless of value */
[disabled] {
  opacity: 0.5;
  cursor: not-allowed;
}
```

---

## 8.6 Pseudo-classes — styling element states

A **pseudo-class** styles an element *only when it is in a certain state*. We write it with a colon `:`.

The three you will use immediately:

### `:hover` — mouse is over the element

```html
<a href="#">Hover over me</a>
```

```css
a {
  color: steelblue;
  text-decoration: none;
}

a:hover {
  color: tomato;
  text-decoration: underline;
}
```

Move your mouse over the link — the colour changes. Move it away — it changes back.

### `:focus` — the element has keyboard focus

This fires when a user **Tab**s onto an input or clicks into it. Critical for accessibility — never remove the focus outline without replacing it with something visible.

```css
input:focus {
  outline: none;
  border: 2px solid steelblue;
  background-color: #f5faff;
}
```

### `:nth-child(n)` — pick by position

This targets an element based on its position among its siblings.

```html
<ul>
  <li>Apple</li>
  <li>Banana</li>
  <li>Cherry</li>
  <li>Date</li>
  <li>Elderberry</li>
</ul>
```

```css
li:nth-child(1) {
  color: red;          /* first item only */
}

li:nth-child(odd) {
  background-color: #f5f5f5;  /* zebra striping */
}

li:nth-child(even) {
  background-color: #ffffff;
}
```

You can also use formulas: `:nth-child(2n)` (every second), `:nth-child(3n+1)` (every third starting at the first). Stick to `odd` and `even` for now — they cover most real situations.

A few other useful pseudo-classes you will meet later:

| Pseudo-class | What it matches |
|--------------|-----------------|
| `:first-child` | The first child of its parent |
| `:last-child` | The last child of its parent |
| `:checked` | A checked checkbox or radio |
| `:disabled` | A disabled form control |
| `:not(.x)` | Anything that is *not* `.x` |

---

## 8.7 Specificity — who wins when two rules clash?

Imagine you write these rules:

```css
p {
  color: black;
}

.note {
  color: blue;
}

#first {
  color: red;
}
```

```html
<p>Plain paragraph.</p>
<p class="note">Note paragraph.</p>
<p class="note" id="first">Very special paragraph.</p>
```

What colour is each paragraph?

1. Plain — **black** (only the element selector matches)
2. Note — **blue** (class beats element)
3. Very special — **red** (ID beats class beats element)

CSS uses a points system called **specificity**. A simple way to think about it:

| Selector type | Specificity score |
|---------------|------------------|
| Element (`p`, `div`) | 1 |
| Class (`.note`), attribute, pseudo-class | 10 |
| ID (`#first`) | 100 |
| Inline `style="..."` attribute | 1000 |

The highest score wins. If two rules tie, the one **written later** in the stylesheet wins.

**Practical advice for now**:

- Use classes for almost everything — they keep specificity flat and easy to reason about
- Avoid IDs for styling — they win too easily and create bugs
- Avoid `!important` (we will mention it later) — it is the nuclear option

Specificity feels mysterious for a few weeks. Then it clicks. Just remember: **classes beat element selectors; IDs beat classes**.

---

## 8.8 A complete example — putting it all together

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Selectors demo</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <header id="top">
      <h1>My CSS Selector Lab</h1>
      <nav>
        <a href="#">Home</a>
        <a href="#">About</a>
        <a href="#">Contact</a>
      </nav>
    </header>

    <main>
      <p class="lead">Welcome to a small demo page.</p>
      <p>This is a normal paragraph.</p>

      <ul>
        <li>Apple</li>
        <li>Banana</li>
        <li>Cherry</li>
        <li>Date</li>
      </ul>

      <form>
        <input type="text" placeholder="Your name" />
        <input type="email" placeholder="Your email" />
        <button>Send</button>
      </form>
    </main>
  </body>
</html>
```

`style.css`:

```css
/* Element selectors — site-wide defaults */
body {
  font-family: Arial, sans-serif;
  color: #222222;
  line-height: 1.6;
}

h1, h2 {
  color: navy;
}

/* ID selector — the unique header bar */
#top {
  background-color: #eef3fa;
  padding: 16px;
}

/* Class selector */
.lead {
  font-size: 20px;
  font-weight: bold;
}

/* Descendant selector */
nav a {
  color: steelblue;
  margin-right: 12px;
  text-decoration: none;
}

/* Pseudo-class */
nav a:hover {
  color: tomato;
  text-decoration: underline;
}

/* Attribute selector */
input[type="email"] {
  border: 2px solid steelblue;
}

input:focus {
  outline: none;
  border-color: tomato;
  background-color: #fff7f5;
}

/* :nth-child for zebra-striped list */
li:nth-child(odd) {
  background-color: #f5f5f5;
}
```

Save both files, refresh the browser, hover the nav links, tab through the form. Every concept in this class is in those 40 lines.

---

## Practice Exercises

1. **Selector hunt** — Create an HTML page with a heading, three paragraphs (one with `class="highlight"`, one with `id="intro"`), and a list of five items. Write CSS that:
   - Makes all paragraphs use Verdana
   - Gives the highlighted paragraph a yellow background
   - Makes the introduction paragraph dark red and bold
   - Zebra-stripes the list items with `:nth-child`

2. **Hover playground** — Make three styled buttons. Use `:hover` to change each button's background colour and `:focus` to add a coloured ring. Try keyboard-tabbing between them and make sure focus is clearly visible.

3. **Specificity puzzle** — Without running the code, predict the colour of each paragraph below, then check by running it.
   ```html
   <p>One</p>
   <p class="x">Two</p>
   <p class="x" id="y">Three</p>
   ```
   ```css
   p { color: black; }
   .x { color: blue; }
   #y { color: red; }
   ```

4. **Refactor** — Take the portfolio HTML from Class 7. Link a `style.css`, then style at least one selector of each type covered today (element, class, ID, descendant, attribute, pseudo-class).

---

## Key Takeaways
- CSS styles HTML — always prefer an **external stylesheet** linked with `<link rel="stylesheet">`
- Every rule is `selector { property: value; }` and lines end with `;`
- Use **classes** for almost everything; reserve **IDs** for genuinely unique elements
- **Pseudo-classes** (`:hover`, `:focus`, `:nth-child`) style elements in particular states
- When two rules clash, **specificity** decides: ID > class > element; if equal, the later rule wins
- Next class: every element on the page is a box — let's learn the box model, colours and backgrounds.
