# Class 3: HTML Text, Links, Images & Lists

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 2 — you can write the HTML5 boilerplate and open a page with Live Server.

## What You Will Learn
- The six heading levels and the **one-`<h1>`-per-page** rule
- Paragraphs and inline emphasis: `<strong>`, `<em>`, `<br>`, `<hr>`
- **Links** with `<a>`, including new-tab links and **anchor links** to sections on the same page
- **Images** with `<img>` and why the `alt` attribute is non-negotiable
- **Lists**: unordered, ordered, and nested

---

## 3.1 Headings

HTML provides six heading levels, from `<h1>` (largest, most important) to `<h6>` (smallest, least important).

```html
<h1>Main Page Title</h1>
<h2>Section Title</h2>
<h3>Sub-section Title</h3>
<h4>A Smaller Heading</h4>
<h5>Even Smaller</h5>
<h6>The Smallest Heading</h6>
```

**The one-`<h1>` rule**: a page should have **exactly one `<h1>`** — the main subject of the page. Everything else uses `<h2>` for sections, `<h3>` for sub-sections, and so on. Think of it like a book:

| Heading | Used for | Book analogy |
|---------|----------|---------------|
| `<h1>` | The page's one main topic | Book title |
| `<h2>` | Major sections | Chapter titles |
| `<h3>` | Topics within a section | Sub-chapter headings |
| `<h4>`–`<h6>` | Rarely needed | Foot-notes and asides |

Why does this matter?

- **Search engines** read your heading hierarchy to understand your page.
- **Screen readers** let blind users jump from heading to heading.
- **Maintenance**: a clear outline is easier to redesign later.

> Headings are *not* about size. If you want big text, use CSS. Pick the heading level based on *meaning*, not appearance.

---

## 3.2 Paragraphs & Inline Text

The `<p>` tag wraps a paragraph. The browser automatically adds vertical spacing above and below.

```html
<p>HTML stands for HyperText Markup Language.</p>
<p>It is the foundation of every web page on the internet.</p>
```

Within a paragraph, you often want to emphasise certain words. Use these inline tags:

| Tag | Meaning | Visual default |
|-----|---------|----------------|
| `<strong>` | This text is **important** | **Bold** |
| `<em>` | This text has **emphasis** | *Italic* |
| `<br>` | Line break — start a new line without ending the paragraph | (No content) |
| `<hr>` | Thematic break — separates two topics | A horizontal line |

Example:

```html
<p>
  Welcome to <strong>Triton College</strong>. We are <em>excited</em>
  to start this React course with you.
</p>

<hr />

<p>
  Class meets three times a week.<br />
  Each class is exactly one hour.
</p>
```

> **`<strong>` vs `<b>` and `<em>` vs `<i>`**: visually they look the same, but `<strong>` and `<em>` carry *meaning* (assistive tech announces them differently). `<b>` and `<i>` are purely visual. Always prefer the meaningful versions.

---

## 3.3 Links — `<a>`

Links are what makes the *web* a web. They are created with the `<a>` (anchor) tag and the `href` attribute (short for "hypertext reference").

### A basic link

```html
<a href="https://triton.edu.np">Visit Triton College</a>
```

This renders as a clickable, underlined "Visit Triton College".

### Link to another local page

```html
<a href="about.html">About Me</a>
```

The browser loads `about.html` from the same folder as the current page.

### Open in a new tab

Add `target="_blank"` so the link opens in a new tab and the user does not lose your page:

```html
<a href="https://triton.edu.np" target="_blank" rel="noopener noreferrer">
  Visit Triton (new tab)
</a>
```

The `rel="noopener noreferrer"` pair is a small security and privacy precaution — always include it with `target="_blank"`.

### Anchor links (jump to a section)

You can link to any element on the *same page* by giving that element an `id` and using `#id` as the `href`:

```html
<!-- The link at the top -->
<a href="#contact">Jump to contact</a>

<!-- ... lots of content ... -->

<!-- The target further down the page -->
<h2 id="contact">Contact</h2>
<p>Email: hello@example.com</p>
```

Clicking the link scrolls the page to the heading with `id="contact"`. This is how every "Back to top" or table-of-contents link works.

### Complete link example

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Links Demo</title>
  </head>
  <body>
    <h1>Links Demo</h1>

    <p>
      Go to the
      <a href="#footer">footer</a>, or visit
      <a href="https://developer.mozilla.org" target="_blank" rel="noopener noreferrer">MDN</a>
      to learn more.
    </p>

    <p>... imagine lots of content here ...</p>
    <p>... and even more content ...</p>

    <h2 id="footer">Footer</h2>
    <p>You jumped to the footer!</p>
  </body>
</html>
```

Open this with Live Server and click "footer" — the page scrolls smoothly.

---

## 3.4 Images — `<img>`

Images are added with the self-closing `<img>` tag.

```html
<img src="profile.jpg" alt="Sita Sharma smiling at the camera" width="200" height="200" />
```

| Attribute | What it does |
|-----------|--------------|
| `src` | The path to the image — can be a local file (`profile.jpg`) or a URL (`https://...`) |
| `alt` | A short description for screen readers and for when the image fails to load |
| `width` | Optional. Display width in pixels. |
| `height` | Optional. Display height in pixels. |

### The `alt` attribute is mandatory

Always write `alt`. There are two reasons:

1. **Accessibility** — a blind user's screen reader reads the `alt` text aloud. No `alt` means they hear "image, image, image" and learn nothing.
2. **Reliability** — if the image fails to download, browsers show the `alt` text instead. Users still understand the page.

**Good `alt` text** describes the image's *purpose* in context:

| Image | Good `alt` | Bad `alt` |
|-------|-----------|-----------|
| Photo of a smiling cat | `alt="A ginger cat sitting on a windowsill"` | `alt="image1"` |
| Company logo in the header | `alt="Triton College"` | `alt="logo.png"` |
| Purely decorative line drawing | `alt=""` (empty — tells screen readers to skip) | (omitted) |

> Yes, `alt=""` (empty) is correct for *purely decorative* images. Omitting `alt` entirely is *wrong*.

### Specifying width and height

Always set `width` and `height`. This prevents the page from "jumping" when the image loads — the browser reserves the right amount of space in advance.

```html
<img src="cat.jpg" alt="A ginger cat on a windowsill" width="400" height="300" />
```

You can also resize with CSS later — but setting the attributes here gives the browser an *aspect ratio* to plan with.

### Where the image file lives

For a local image, the `src` is **relative to the HTML file**.

```
my-project/
├── index.html
└── images/
    └── profile.jpg
```

In `index.html`, the path is:

```html
<img src="images/profile.jpg" alt="My profile photo" width="200" height="200" />
```

For a quick test you can also use a free placeholder service:

```html
<img src="https://placehold.co/400x300" alt="A placeholder image" width="400" height="300" />
```

---

## 3.5 Lists

Lists are everywhere on the web — menus, ingredients, steps, search results. HTML offers two kinds: unordered and ordered.

### Unordered lists — `<ul>`

For items where **order does not matter** (a list of hobbies, ingredients, features).

```html
<h2>My Hobbies</h2>
<ul>
  <li>Reading</li>
  <li>Football</li>
  <li>Learning React</li>
</ul>
```

Each `<li>` is a "list item". The browser renders bullet points by default.

### Ordered lists — `<ol>`

For items where **order matters** (steps in a recipe, top-10 list, instructions).

```html
<h2>How to Make Tea</h2>
<ol>
  <li>Boil water.</li>
  <li>Add a tea bag to a cup.</li>
  <li>Pour the boiling water over the tea bag.</li>
  <li>Wait three minutes, then remove the bag.</li>
  <li>Add milk and sugar to taste.</li>
</ol>
```

The browser renders numbers automatically — you do not type "1." yourself.

### Nested lists

A list item can contain another list, allowing sub-points:

```html
<h2>Course Topics</h2>
<ul>
  <li>
    HTML
    <ul>
      <li>Structure</li>
      <li>Forms</li>
      <li>Semantic tags</li>
    </ul>
  </li>
  <li>
    CSS
    <ul>
      <li>Box model</li>
      <li>Flexbox</li>
      <li>Grid</li>
    </ul>
  </li>
  <li>JavaScript</li>
  <li>React</li>
</ul>
```

The indentation in your code helps you see the structure. Browsers indent the nested list visually too.

---

## 3.6 Putting It All Together — An "About Me" Page

Here is a complete page using every concept from this class. Save as `about-me.html`, open with Live Server, and tweak the text to be about *you*.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>About Me — Sita Sharma</title>
  </head>
  <body>
    <h1>About Me</h1>

    <!-- A small navigation list of anchor links -->
    <p><strong>Jump to:</strong></p>
    <ul>
      <li><a href="#intro">Introduction</a></li>
      <li><a href="#hobbies">Hobbies</a></li>
      <li><a href="#links">Useful Links</a></li>
    </ul>

    <hr />

    <h2 id="intro">Introduction</h2>
    <img
      src="https://placehold.co/200x200"
      alt="A placeholder for my profile photo"
      width="200"
      height="200"
    />
    <p>
      Hello! I am <strong>Sita Sharma</strong>, a BCA second-semester student
      at <em>Triton College</em>. I am learning to build modern web
      applications with React.
    </p>

    <h2 id="hobbies">Hobbies</h2>
    <ul>
      <li>Reading novels</li>
      <li>Playing the guitar</li>
      <li>
        Coding
        <ul>
          <li>Web development</li>
          <li>Mobile apps (one day)</li>
        </ul>
      </li>
    </ul>

    <h2 id="links">Useful Links</h2>
    <ol>
      <li>
        <a href="https://developer.mozilla.org" target="_blank" rel="noopener noreferrer">
          MDN Web Docs
        </a>
        — the best HTML/CSS/JS reference.
      </li>
      <li>
        <a href="https://github.com" target="_blank" rel="noopener noreferrer">GitHub</a>
        — where my code lives.
      </li>
    </ol>
  </body>
</html>
```

**What to expect**: a plain page with a heading, a small jump-menu, a placeholder photo, a paragraph, a nested hobbies list, and a numbered list of links. Clicking the jump-menu items scrolls to the right heading.

---

## Practice Exercises

### Exercise 1 — Your real About Me page (15 minutes)
Replace the placeholder content in the example above with your own information. Use a real photo if you have one — otherwise keep the placeholder URL. Confirm:

- There is exactly **one `<h1>`**.
- Every `<img>` has a meaningful `alt`.
- External links open in a new tab with `rel="noopener noreferrer"`.

### Exercise 2 — A recipe page
Create `recipe.html` with:

- An `<h1>` with the recipe name.
- A `<p>` describing the dish briefly.
- An `<h2>` "Ingredients" followed by a `<ul>`.
- An `<h2>` "Steps" followed by an `<ol>`.
- An anchor link at the top called "Jump to steps" that scrolls to the steps section.

### Exercise 3 — Nested list challenge
Build a `<ul>` describing the structure of this course, with three top-level items (HTML, CSS, JavaScript), each containing a sub-`<ul>` of two or three topics.

---

## Key Takeaways
- Use **one `<h1>`** per page; pick heading levels by *meaning*, not size.
- `<strong>` and `<em>` carry meaning; prefer them over `<b>` and `<i>`.
- Links open in a new tab with `target="_blank"` plus `rel="noopener noreferrer"`.
- **Anchor links** (`href="#id"`) jump to elements with matching `id` attributes.
- Every `<img>` needs an `alt`; set `width` and `height` to stop page jumps.
- `<ul>` for unordered, `<ol>` for ordered, nested freely as needed.

Next class we will look at **tables** for displaying data and start building our first **forms**.
