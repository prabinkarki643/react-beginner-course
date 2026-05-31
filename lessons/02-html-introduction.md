# Class 2: HTML Introduction & Document Structure

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 1 — Node, VS Code, the Live Server extension, and a project folder you can save files into.

## What You Will Learn
- What HTML is and how the browser actually uses it
- The three-way split between HTML, CSS, and JavaScript
- Every part of the **HTML5 boilerplate**: `<!DOCTYPE>`, `<html lang>`, `<head>`, `<meta>`, `<title>`, `<body>`
- How to write **comments** and use sensible **indentation**
- How to type a boilerplate from memory using VS Code's built-in `!` shortcut

---

## 2.1 What is HTML?

**HTML** stands for **HyperText Markup Language**. It is not a programming language — it is a *markup* language, which means it describes the **structure** of a document using tags.

If a web page were a person:

| Layer | What it does | Analogy |
|-------|--------------|---------|
| **HTML** | Describes structure: headings, paragraphs, images, lists | The **skeleton** — bones and joints |
| **CSS** | Describes appearance: colours, fonts, layout | The **clothes** — what the person wears |
| **JavaScript** | Describes behaviour: clicks, animations, data | The **brain & muscles** — how the person reacts |

A page can exist with HTML only — it will be plain but it will work. A page cannot exist with only CSS or only JavaScript. **HTML is the foundation everything else sits on.**

When you visit `https://example.com`, your browser downloads an HTML file, reads it top to bottom, and uses the tags to build the picture you see on screen.

---

## 2.2 How HTML Tags Work

A tag is a label wrapped in angle brackets. Most tags come in pairs — an **opening tag** and a **closing tag** — with content in between:

```html
<p>This is a paragraph.</p>
```

- `<p>` is the **opening tag**.
- `This is a paragraph.` is the **content**.
- `</p>` is the **closing tag** (note the forward slash).

Together this is called an **element**.

A few tags are **self-closing** — they have no content and no closing tag:

```html
<br>
<hr>
<img src="cat.jpg" alt="A cat">
```

Tags can also carry **attributes** — extra information written `key="value"`:

```html
<a href="https://triton.edu.np">Visit Triton</a>
```

Here `href` is the attribute name and `"https://triton.edu.np"` is its value.

> **UK spelling note**: HTML attribute names are part of the language itself, so they are *always* American English (`color`, `center`, etc.). Your *prose* and your own variable names should still be UK English. This is the one tiny exception you must accept.

---

## 2.3 The HTML5 Boilerplate

Every modern HTML page begins with the same starter template, called the **boilerplate**. Type this out — do not copy-paste yet, because typing it builds muscle memory.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My First Real Page</title>
  </head>
  <body>
    <h1>Welcome</h1>
    <p>This is the body of my page.</p>
  </body>
</html>
```

Save this as `index.html`, open it with Live Server, and you will see "Welcome" and "This is the body of my page." in the browser.

Now let us break down every line.

### `<!DOCTYPE html>`

Always the very first line. It tells the browser, *"Treat this file as modern HTML5."* Without it, browsers may fall back into a 1990s-compatibility mode called "quirks mode" and your page will look slightly broken in subtle ways. Never omit it.

### `<html lang="en">`

The **root element** — every other tag lives inside this one. The `lang="en"` attribute tells the browser the page is in English. Screen readers use this to pick the correct pronunciation, and search engines use it to decide which audience to show the page to. Use `"ne"` for Nepali, `"hi"` for Hindi, etc.

### `<head>`

The `<head>` is **invisible** — nothing inside it appears on the page. It contains metadata: information *about* the page, not the page's content. Browsers, search engines and social networks read it.

### `<meta charset="UTF-8" />`

Tells the browser to use **UTF-8** encoding — the universal character set that supports English, Nepali (नेपाली), emojis (🎉) and every other writing system. Always include this; without it, accented characters can appear as nonsense like `Ã©`.

### `<meta name="viewport" content="width=device-width, initial-scale=1.0" />`

Tells mobile browsers, *"Display this page at its real width — do not pretend the phone is a tiny desktop."* Without it, your pages will look zoomed-out on phones. Every modern page includes this line.

### `<title>My First Real Page</title>`

The text shown in the browser tab and in search-engine results. Keep it short, descriptive, and unique per page.

### `<body>`

Everything **visible** to the user goes here — headings, paragraphs, images, buttons, forms. This is where you will do 99% of your work for the next six classes.

---

## 2.4 Visualising the Structure

The boilerplate is a **tree** — tags nested inside other tags. Here is a simplified view:

```
html
├── head
│   ├── meta (charset)
│   ├── meta (viewport)
│   └── title
└── body
    ├── h1
    └── p
```

We call this the **DOM tree** (Document Object Model). When we get to JavaScript in Class 22, we will manipulate this exact tree from code. For now, just notice the parent-child relationships.

---

## 2.5 Comments

A **comment** is a note for humans — the browser ignores it completely. Use comments to explain *why* something is there, not *what* it is (the code already shows what).

```html
<!-- Top of the page: branding and navigation will go here later -->
<header>
  <h1>My Blog</h1>
</header>

<!-- Main content -->
<main>
  <p>Welcome to my blog.</p>
</main>
```

Anything between `<!--` and `-->` is invisible to visitors but visible to anyone who views your source code. Never put passwords or secrets in comments — anyone can read them.

You can quickly comment or uncomment a line in VS Code with `Cmd/Ctrl + /`.

---

## 2.6 Indentation Conventions

HTML still works if you write everything on one line, but it becomes painful to read. The convention is:

- **Indent each child** by **2 spaces** (some teams use 4 — pick one and stay consistent).
- **Closing tags line up** with their opening tags.
- **One tag per line** for blocks; small inline tags can stay on the same line.

Compare these two snippets. Both render identically, but only one is readable:

**Painful:**
```html
<div><h2>Title</h2><p>Paragraph 1.</p><p>Paragraph 2.</p></div>
```

**Readable:**
```html
<div>
  <h2>Title</h2>
  <p>Paragraph 1.</p>
  <p>Paragraph 2.</p>
</div>
```

Prettier (which you installed in Class 1) will do this for you when you save. But you must understand *why* we indent — so you can read other people's code, even when it has not been formatted.

---

## 2.7 The Magic `!` Shortcut in VS Code

Now that you understand the boilerplate, let us learn the shortcut professional developers actually use.

1. Create a new file called `quick.html` (the `.html` extension matters).
2. With the file open, type a single exclamation mark: `!`
3. Press `Tab`.

VS Code instantly generates a complete HTML5 boilerplate for you. This is called an **Emmet abbreviation** and it is built into VS Code.

You now know what each generated line means — so you are not blindly copying, you are saving keystrokes.

---

## 2.8 Putting It All Together

Here is a complete, commented file showing everything from this class. Save it as `class-02.html`, open it with Live Server, and you should see a basic page with a title, a subtitle, and a short message.

```html
<!DOCTYPE html>
<!-- The lang attribute helps screen readers pronounce text correctly -->
<html lang="en">
  <head>
    <!-- UTF-8 supports every language and emoji -->
    <meta charset="UTF-8" />

    <!-- Tells phones to render at the device's real width -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

    <!-- The tab title -->
    <title>About Me — Sita Sharma</title>
  </head>

  <body>
    <!-- Main heading: one h1 per page -->
    <h1>About Me</h1>

    <!-- A short introduction -->
    <p>Hello! I am Sita, a BCA second-semester student at Triton College.</p>
    <p>I am learning to build modern web applications with React.</p>
  </body>
</html>
```

**What to expect when you open it**: a plain white page with a large "About Me" heading and two paragraphs underneath. No colours, no images — that is correct. We add those in later classes.

---

## Practice Exercises

### Exercise 1 — Boilerplate from memory (15 minutes)
Open a new file called `memory.html`. **Without looking at any notes**, type the full HTML5 boilerplate from memory, including:

- `<!DOCTYPE html>`
- `<html lang="en">`
- A `<head>` with the charset meta, the viewport meta, and a `<title>` of your choice.
- A `<body>` with one `<h1>` saying "My First Boilerplate" and one `<p>` describing what you learned today.

Open it with Live Server to confirm it works.

### Exercise 2 — Comment the file
Add a comment above the `<head>` saying "Metadata: not visible to users" and a comment above the `<body>` saying "Everything visible goes here". Save and confirm the comments do not appear in the browser.

### Exercise 3 — Change the language
Change `lang="en"` to `lang="ne"` (Nepali) and add a `<p>` with Nepali text (or your mother tongue). Reload and confirm the characters display correctly thanks to UTF-8.

---

## Key Takeaways
- **HTML = structure**; CSS = appearance; JavaScript = behaviour.
- Every page starts with the **HTML5 boilerplate** — `<!DOCTYPE html>` is non-negotiable.
- The `<head>` is invisible metadata; the `<body>` is everything visible.
- The viewport meta tag makes pages render correctly on phones.
- Comments (`<!-- … -->`) are for humans; the browser ignores them.
- Indent two spaces per level; Prettier does it for you on save.

Next class we will fill the `<body>` with real content — headings, paragraphs, links, images, and lists.
