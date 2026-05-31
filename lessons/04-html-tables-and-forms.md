# Class 4: HTML Tables & Forms (Part 1)

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 3 — you can write headings, paragraphs, links, images, and lists.

## What You Will Learn
- How to build a proper **HTML table** with `<table>`, `<thead>`, `<tbody>`, `<tr>`, `<th>`, `<td>`
- When a table is the right choice — and when it definitely is not
- The anatomy of a **form**: `<form>`, `<label>`, `<input>`, `<button>`
- Why the `for`/`id` pairing on labels matters
- The basic input types: `text`, `email`, `password`, `number`

---

## 4.1 Tables — What They Are For

A **table** displays **two-dimensional data**: rows and columns that genuinely belong together. Examples:

- A list of students with their roll numbers and marks.
- A train timetable.
- A price comparison.

A table is **not** for visual layout. Twenty years ago developers abused tables to position page content. We do not do that any more — we use CSS Flexbox and Grid (Classes 12 and 13). If the rows and columns are not real data, do not use a table.

Quick test: ask yourself, *"Could I put a heading row at the top and have it make sense?"* If yes, it is a table. If no, it is not.

---

## 4.2 Building a Table

The minimum required tags are `<table>`, `<tr>` (table row), `<th>` (table heading cell), and `<td>` (table data cell). Modern tables also use `<thead>` and `<tbody>` to separate the heading row from the body.

```html
<table>
  <thead>
    <tr>
      <th>Roll No.</th>
      <th>Name</th>
      <th>Marks</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>01</td>
      <td>Sita Sharma</td>
      <td>87</td>
    </tr>
    <tr>
      <td>02</td>
      <td>Ram Karki</td>
      <td>92</td>
    </tr>
    <tr>
      <td>03</td>
      <td>Hari Thapa</td>
      <td>76</td>
    </tr>
  </tbody>
</table>
```

**Mental model**: a table is a list of rows (`<tr>`); each row contains a list of cells. Heading cells (`<th>`) are bold and centred by default; data cells (`<td>`) are plain. The browser handles all alignment for you.

### `<thead>` and `<tbody>` — why bother?

- `<thead>` marks the heading row. Screen readers announce the heading as a label for each cell ("Marks, 87").
- `<tbody>` marks the data rows.
- They make styling cleaner later (CSS can target heading rows differently).

They are optional but professional code always includes them.

### Adding a caption

Use `<caption>` to give the table a short title — placed directly inside `<table>` before `<thead>`:

```html
<table>
  <caption>Term 1 results — BCA Semester 2</caption>
  <thead>
    <tr>
      <th>Roll No.</th>
      <th>Name</th>
      <th>Marks</th>
    </tr>
  </thead>
  <!-- ... -->
</table>
```

Captions help screen readers announce what the table is about before reading any data.

---

## 4.3 A Complete Table Page

Save the following as `marks.html` and open with Live Server:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Marks Table</title>
  </head>
  <body>
    <h1>BCA Semester 2 — Subject Marks</h1>

    <table>
      <caption>Term 1 results for Sita Sharma</caption>
      <thead>
        <tr>
          <th>Subject</th>
          <th>Marks Obtained</th>
          <th>Full Marks</th>
          <th>Percentage</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>HTML &amp; CSS</td>
          <td>87</td>
          <td>100</td>
          <td>87%</td>
        </tr>
        <tr>
          <td>JavaScript</td>
          <td>78</td>
          <td>100</td>
          <td>78%</td>
        </tr>
        <tr>
          <td>React</td>
          <td>92</td>
          <td>100</td>
          <td>92%</td>
        </tr>
        <tr>
          <td>Mathematics</td>
          <td>65</td>
          <td>100</td>
          <td>65%</td>
        </tr>
        <tr>
          <td>English</td>
          <td>74</td>
          <td>100</td>
          <td>74%</td>
        </tr>
      </tbody>
    </table>
  </body>
</html>
```

**What to expect**: a plain unstyled table with bold heading cells and rows of data underneath. The `&amp;` is the HTML entity for `&` — we will meet a few of these over the course.

> A note on the unstyled look: HTML tables are deliberately ugly until CSS styles them. They have no borders or padding by default. In Class 9 we will style this exact table.

---

## 4.4 Forms — What They Are For

A **form** is how a user **sends data** to a website. Every time you log in, sign up, search Google, or post a comment, you are submitting a form.

Forms are the most interactive part of HTML. They wrap inputs that collect data and a button that submits it.

```html
<form>
  <!-- inputs go here -->
  <button type="submit">Submit</button>
</form>
```

For the next two classes we focus on the **markup** — what each input looks like. The *handling* of the data (what happens when you click Submit) comes later in JavaScript and React.

---

## 4.5 The Building Blocks

Every form is made of four things:

| Element | Role |
|---------|------|
| `<form>` | The container that groups all the inputs |
| `<label>` | The text that describes an input |
| `<input>` | The interactive control (text box, checkbox, etc.) |
| `<button>` | The control the user clicks to submit |

### Labels and inputs

A `<label>` describes an `<input>`. They are linked by matching the label's `for` attribute to the input's `id`:

```html
<label for="username">Username</label>
<input type="text" id="username" name="username" />
```

Why bother linking them?

- Clicking the **label** focuses the input — bigger click target, friendlier on mobile.
- **Screen readers** announce the label when the user reaches the input.
- It is the law in many countries for public-sector sites (accessibility requirements).

> **Rule**: every input gets a label, every label is linked with `for`/`id`.

You can also nest the input inside the label, which removes the need for `for`/`id`:

```html
<label>
  Username
  <input type="text" name="username" />
</label>
```

Both styles are valid. The separate version is more flexible for styling, so we will use it most often.

---

## 4.6 The `name` Attribute

When a form is submitted, the browser sends `name=value` pairs to the server. The **`name`** attribute is what identifies each field in the data.

```html
<input type="text" id="username" name="username" />
<input type="email" id="email" name="email" />
```

If the user types "sita" and "sita@example.com", the data sent looks like:

```
username=sita&email=sita@example.com
```

Rules of thumb:

- `id` is for **linking the label** (and CSS/JavaScript later).
- `name` is for **the submitted data**.
- They are often the same word, but they do not have to be.

---

## 4.7 Basic Input Types

The `type` attribute on `<input>` decides what kind of control you see. Class 5 covers the rest — for today, learn the four most common ones.

### `type="text"` — single-line text

```html
<label for="full-name">Full Name</label>
<input type="text" id="full-name" name="fullName" placeholder="e.g. Sita Sharma" />
```

The `placeholder` is grey hint text that disappears as soon as the user starts typing.

### `type="email"` — email address

```html
<label for="email">Email</label>
<input type="email" id="email" name="email" placeholder="you@example.com" />
```

Identical to `text` in appearance, but the browser checks that the value contains an `@` and a domain when the form is submitted. On phones, it also opens the email-friendly keyboard with the `@` key visible.

### `type="password"` — hidden text

```html
<label for="password">Password</label>
<input type="password" id="password" name="password" />
```

Characters appear as dots so a shoulder-surfer cannot read the password.

### `type="number"` — numeric input

```html
<label for="age">Age</label>
<input type="number" id="age" name="age" min="0" max="120" />
```

Shows up/down spinner arrows in most browsers. The `min` and `max` attributes set the allowed range.

---

## 4.8 The Submit Button

A form needs a way to submit. Use a `<button>` with `type="submit"`:

```html
<button type="submit">Log In</button>
```

`<button>` can also be `type="button"` (does nothing automatically — for JavaScript later) or `type="reset"` (clears the form). For now, always use `type="submit"` inside a form.

> **Best practice**: always set the `type` explicitly. A `<button>` without a type defaults to `submit`, which is often *not* what you want once you add JavaScript.

---

## 4.9 A Complete Login Form

Save as `login.html` and open with Live Server:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Log In</title>
  </head>
  <body>
    <h1>Log In</h1>

    <form>
      <p>
        <label for="email">Email</label><br />
        <input type="email" id="email" name="email" placeholder="you@example.com" />
      </p>

      <p>
        <label for="password">Password</label><br />
        <input type="password" id="password" name="password" />
      </p>

      <p>
        <button type="submit">Log In</button>
      </p>
    </form>
  </body>
</html>
```

**What to expect**: a plain heading, two labelled text fields, and a "Log In" button. Try clicking the labels — the focus jumps to the matching input.

> We have wrapped each label/input pair in a `<p>` purely for spacing. Real-world forms use CSS for layout, but this keeps things tidy for now.

---

## 4.10 Combining Tables and Forms

Sometimes forms appear next to data tables. Save this complete page as `class-04.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Class 4 — Tables and Forms</title>
  </head>
  <body>
    <h1>BCA Semester 2 — Marks Portal</h1>

    <h2>Current Marks</h2>
    <table>
      <caption>Marks as of today</caption>
      <thead>
        <tr>
          <th>Subject</th>
          <th>Marks</th>
        </tr>
      </thead>
      <tbody>
        <tr><td>HTML &amp; CSS</td><td>87</td></tr>
        <tr><td>JavaScript</td><td>78</td></tr>
        <tr><td>React</td><td>92</td></tr>
      </tbody>
    </table>

    <h2>Log in to Update</h2>
    <form>
      <p>
        <label for="user-id">Student ID</label><br />
        <input type="number" id="user-id" name="studentId" min="1" />
      </p>
      <p>
        <label for="email">Email</label><br />
        <input type="email" id="email" name="email" placeholder="you@triton.edu.np" />
      </p>
      <p>
        <label for="pwd">Password</label><br />
        <input type="password" id="pwd" name="password" />
      </p>
      <p>
        <button type="submit">Log In</button>
      </p>
    </form>
  </body>
</html>
```

This page has a small data table at the top and a labelled login form below — both unstyled but both fully accessible.

---

## Practice Exercises

### Exercise 1 — Class timetable (15 minutes)
Build a table for your weekly class timetable. Columns: **Day**, **Subject**, **Time**, **Room**. Include a `<caption>`, a `<thead>`, and at least five rows in the `<tbody>`.

### Exercise 2 — Sign-up form
Create `signup.html` with a form containing:

- **Full Name** — `type="text"`
- **Email** — `type="email"`
- **Password** — `type="password"`
- **Age** — `type="number"` with `min="13"` and `max="120"`
- A **Submit** button labelled "Create Account"

Every input must have a properly linked `<label>` and a sensible `name`.

### Exercise 3 — Click the label
Click each label in your sign-up form. Confirm the corresponding input gains focus. If a label does not work, check that its `for` value exactly matches the input's `id`.

---

## Key Takeaways
- Use **tables for two-dimensional data**, never for page layout.
- Wrap heading cells in `<thead>` and data rows in `<tbody>` — and always add a `<caption>`.
- A form is a `<form>` containing `<label>` / `<input>` pairs and a `<button>`.
- Link every `<label>` to its `<input>` via `for`/`id`.
- `id` is for labels; `name` is for the submitted data.
- The four core input types: `text`, `email`, `password`, `number`.

Next class we will cover every other input type — checkboxes, radios, dropdowns, date pickers, file uploads — plus HTML5's built-in validation rules.
