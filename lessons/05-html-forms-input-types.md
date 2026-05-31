# Class 5: HTML Forms (Part 2) — Input Types & Validation

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 4 — you can build a basic form with labels, text inputs, and a submit button.

## What You Will Learn
- **Choice inputs**: checkboxes, radio buttons
- **Dropdowns** with `<select>` and `<option>`
- **Multi-line text** with `<textarea>`
- **Specialised inputs**: `date`, `file`, `range`
- Attributes that shape user input: `placeholder`, `required`, `min`, `max`, `maxlength`, `pattern`
- **Grouping** inputs with `<fieldset>` and `<legend>`
- HTML5's **built-in validation** — no JavaScript needed

---

## 5.1 Checkboxes — Pick Any Combination

A **checkbox** (`type="checkbox"`) is for *yes/no* choices, or for picking any number of items from a list.

```html
<p>
  <label>
    <input type="checkbox" name="newsletter" /> Sign me up for the newsletter
  </label>
</p>
```

**Things to know:**

- A checkbox without a label is useless — always wrap it (or pair it with a `<label>`).
- The `name` is what identifies the field on submit.
- Add `checked` to make it tick by default: `<input type="checkbox" checked />`.

### A group of checkboxes

When you want users to pick *several* items, use **the same `name`** for each checkbox and give each one a different `value`:

```html
<p>What languages do you know?</p>
<label><input type="checkbox" name="languages" value="english" /> English</label><br />
<label><input type="checkbox" name="languages" value="nepali" /> Nepali</label><br />
<label><input type="checkbox" name="languages" value="hindi" /> Hindi</label><br />
<label><input type="checkbox" name="languages" value="other" /> Other</label>
```

The form data will include one `languages=...` entry per ticked box.

---

## 5.2 Radio Buttons — Pick Exactly One

A **radio button** (`type="radio"`) is for picking *exactly one* item from a group. The trick: every radio in the same group must share the **same `name`**. That is how the browser knows they belong together and only allows one to be selected.

```html
<p>Choose your study mode:</p>
<label><input type="radio" name="mode" value="online" /> Online</label><br />
<label><input type="radio" name="mode" value="in-person" /> In person</label><br />
<label><input type="radio" name="mode" value="hybrid" checked /> Hybrid</label>
```

Click "Online" then "In person" — the previous choice clears automatically because they share `name="mode"`.

| Checkboxes vs Radios | Use when |
|----------------------|----------|
| **Checkbox** | The user can pick zero, one, or many |
| **Radio** | The user must pick exactly one |

> If you have only two opposite options ("Yes/No", "On/Off"), prefer **one checkbox** over two radios. Less clutter.

---

## 5.3 Dropdowns — `<select>` and `<option>`

When the list is long (a country, a year, a city), a dropdown saves space.

```html
<label for="country">Country</label>
<select id="country" name="country">
  <option value="">-- Choose a country --</option>
  <option value="np">Nepal</option>
  <option value="in">India</option>
  <option value="uk">United Kingdom</option>
  <option value="us">United States</option>
</select>
```

**Things to know:**

- The `value` is what gets sent on submit; the text between the tags is what the user sees.
- The first option is often a placeholder with an empty `value` — combine this with `required` (see 5.7) and the form will refuse to submit until the user picks something.
- Add `selected` to pre-select an option: `<option value="np" selected>Nepal</option>`.

### `<optgroup>` — group related options

```html
<select id="country" name="country">
  <optgroup label="South Asia">
    <option value="np">Nepal</option>
    <option value="in">India</option>
    <option value="bd">Bangladesh</option>
  </optgroup>
  <optgroup label="Europe">
    <option value="uk">United Kingdom</option>
    <option value="de">Germany</option>
    <option value="fr">France</option>
  </optgroup>
</select>
```

`<optgroup>` adds a non-selectable heading to part of the list.

---

## 5.4 `<textarea>` — Multi-Line Text

For comments, addresses, biographies — anything that needs more than one line — use `<textarea>`.

```html
<label for="bio">Tell us about yourself</label><br />
<textarea
  id="bio"
  name="bio"
  rows="5"
  cols="40"
  placeholder="A short bio..."
></textarea>
```

| Attribute | What it does |
|-----------|--------------|
| `rows` | Visible height in lines |
| `cols` | Visible width in characters |
| `placeholder` | Grey hint text |
| `maxlength` | Maximum number of characters allowed |

> **Watch out**: unlike `<input>`, `<textarea>` is *not* self-closing. You must write the closing `</textarea>` tag. Any whitespace between the tags becomes the default value — so keep them tightly together if you do not want pre-filled spaces.

---

## 5.5 Specialised Input Types

HTML5 brought several inputs that show special UI on mobile and desktop.

### `type="date"`

```html
<label for="dob">Date of Birth</label>
<input type="date" id="dob" name="dob" />
```

Most browsers show a calendar picker. The value submitted is in `YYYY-MM-DD` format. There are matching `time`, `month`, `week`, and `datetime-local` types if you ever need them.

### `type="file"`

```html
<label for="cv">Upload your CV (PDF)</label>
<input type="file" id="cv" name="cv" accept=".pdf,application/pdf" />
```

The `accept` attribute hints which file types to show. To allow multiple files, add `multiple`:

```html
<input type="file" id="photos" name="photos" accept="image/*" multiple />
```

> A pure HTML page cannot actually *upload* a file anywhere — that needs a back-end server. We will only use this input for previews until then.

### `type="range"`

A slider for picking a number between a minimum and maximum.

```html
<label for="volume">Volume: <input type="range" id="volume" name="volume" min="0" max="100" step="5" /></label>
```

- `min` / `max` — the range of values
- `step` — how much the slider jumps each notch
- The default value is the middle of the range unless you set `value="…"`

### `type="color"`, `type="tel"`, `type="url"`, `type="search"`

Useful for specific data types. They behave like `text` on desktop but trigger context-appropriate keyboards on mobile.

```html
<label for="phone">Phone</label>
<input type="tel" id="phone" name="phone" />

<label for="website">Website</label>
<input type="url" id="website" name="website" placeholder="https://..." />
```

---

## 5.6 Shaping Inputs with Attributes

These attributes work on most input types — learn them once, use them everywhere.

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `placeholder` | Grey hint text inside the field | `placeholder="you@example.com"` |
| `required` | The form will not submit if this is empty | `required` |
| `min` / `max` | Allowed range for numbers, dates, ranges | `min="0" max="100"` |
| `maxlength` | Max number of characters in a text field | `maxlength="20"` |
| `minlength` | Minimum number of characters | `minlength="8"` |
| `pattern` | A regular expression the value must match | `pattern="[0-9]{10}"` |
| `readonly` | User can see but not change the value | `readonly` |
| `disabled` | Greyed out; value is not submitted | `disabled` |
| `autocomplete` | Helps browsers remember and fill values | `autocomplete="email"` |
| `autofocus` | Cursor jumps here when the page loads | `autofocus` |

### Examples

```html
<!-- 10-digit phone number, required -->
<input type="tel" name="phone" required pattern="[0-9]{10}" placeholder="9800000000" />

<!-- Username: 3-20 letters or digits -->
<input type="text" name="username" required minlength="3" maxlength="20" pattern="[a-zA-Z0-9]+" />

<!-- Read-only field showing a generated ID -->
<input type="text" name="userId" value="USR-00123" readonly />
```

> **Tip**: a `pattern` is a tiny regular expression. We will not study regex in this course, but `[0-9]{10}` simply means "exactly ten digits, each one a number from 0 to 9".

---

## 5.7 HTML5 Built-In Validation

When you add `required`, `pattern`, `min`, `max`, `minlength`, `maxlength`, `type="email"`, or `type="url"`, the browser **checks the value automatically** the moment the user submits the form. If something is wrong:

- The form does **not** submit.
- The first invalid field is **highlighted** and **focused**.
- A short **tooltip** explains what is wrong.

You get all of this for free — no JavaScript needed.

```html
<form>
  <p>
    <label for="email">Email</label><br />
    <input type="email" id="email" name="email" required />
  </p>
  <p>
    <label for="pwd">Password (min 8 chars)</label><br />
    <input type="password" id="pwd" name="password" required minlength="8" />
  </p>
  <p>
    <button type="submit">Sign Up</button>
  </p>
</form>
```

Try submitting it empty, or with a password of "abc" — the browser blocks you and explains why. This is a powerful first line of defence, but it is **not** a security feature: users can disable it in DevTools. For real applications we still need to validate on the server (or in this front-end-only course, in our JavaScript and Zod schemas later on).

---

## 5.8 Grouping Inputs — `<fieldset>` and `<legend>`

When a form has several distinct sections, wrap each one in a `<fieldset>` with a `<legend>` describing it. The browser adds a thin border and shows the legend at the top — clear visual grouping at zero cost.

```html
<form>
  <fieldset>
    <legend>Personal Details</legend>
    <p><label for="name">Full Name</label><br />
       <input type="text" id="name" name="name" required /></p>
    <p><label for="dob">Date of Birth</label><br />
       <input type="date" id="dob" name="dob" /></p>
  </fieldset>

  <fieldset>
    <legend>Account</legend>
    <p><label for="email">Email</label><br />
       <input type="email" id="email" name="email" required /></p>
    <p><label for="pwd">Password</label><br />
       <input type="password" id="pwd" name="password" required minlength="8" /></p>
  </fieldset>

  <button type="submit">Create Account</button>
</form>
```

`<fieldset>` is also semantically important: screen readers announce the legend as the *purpose* of the group of inputs inside.

---

## 5.9 Putting It All Together — Student Registration Form

This is the kind of form you might fill in for college admissions. Save it as `register.html` and try it in your browser. Try to submit it half-empty and watch the validation kick in.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Student Registration</title>
  </head>
  <body>
    <h1>Student Registration</h1>

    <form>
      <fieldset>
        <legend>Personal Details</legend>

        <p>
          <label for="full-name">Full Name</label><br />
          <input
            type="text"
            id="full-name"
            name="fullName"
            required
            minlength="2"
            maxlength="60"
            placeholder="e.g. Sita Sharma"
            autofocus
          />
        </p>

        <p>
          <label for="dob">Date of Birth</label><br />
          <input type="date" id="dob" name="dob" required />
        </p>

        <p>Gender:</p>
        <label><input type="radio" name="gender" value="female" required /> Female</label><br />
        <label><input type="radio" name="gender" value="male" /> Male</label><br />
        <label><input type="radio" name="gender" value="other" /> Prefer not to say</label>
      </fieldset>

      <fieldset>
        <legend>Contact</legend>

        <p>
          <label for="email">Email</label><br />
          <input type="email" id="email" name="email" required placeholder="you@example.com" />
        </p>

        <p>
          <label for="phone">Phone (10 digits)</label><br />
          <input
            type="tel"
            id="phone"
            name="phone"
            required
            pattern="[0-9]{10}"
            placeholder="9800000000"
          />
        </p>

        <p>
          <label for="country">Country</label><br />
          <select id="country" name="country" required>
            <option value="">-- Choose a country --</option>
            <option value="np">Nepal</option>
            <option value="in">India</option>
            <option value="uk">United Kingdom</option>
            <option value="us">United States</option>
          </select>
        </p>
      </fieldset>

      <fieldset>
        <legend>Course Preferences</legend>

        <p>Which courses are you interested in? (Pick any)</p>
        <label><input type="checkbox" name="courses" value="html-css" /> HTML &amp; CSS</label><br />
        <label><input type="checkbox" name="courses" value="javascript" /> JavaScript</label><br />
        <label><input type="checkbox" name="courses" value="react" /> React</label><br />
        <label><input type="checkbox" name="courses" value="node" /> Node.js</label>

        <p>
          <label for="hours">Hours of study per week: <span id="hours-value">10</span></label><br />
          <input type="range" id="hours" name="hours" min="1" max="40" step="1" value="10" />
        </p>
      </fieldset>

      <fieldset>
        <legend>About You</legend>

        <p>
          <label for="bio">Short bio (max 200 chars)</label><br />
          <textarea
            id="bio"
            name="bio"
            rows="4"
            cols="50"
            maxlength="200"
            placeholder="Tell us a little about yourself..."
          ></textarea>
        </p>

        <p>
          <label for="cv">Upload your CV (PDF)</label><br />
          <input type="file" id="cv" name="cv" accept=".pdf,application/pdf" />
        </p>
      </fieldset>

      <fieldset>
        <legend>Account</legend>

        <p>
          <label for="pwd">Choose a password (min 8 chars)</label><br />
          <input type="password" id="pwd" name="password" required minlength="8" />
        </p>

        <p>
          <label>
            <input type="checkbox" name="terms" required />
            I agree to the terms and conditions
          </label>
        </p>
      </fieldset>

      <p>
        <button type="submit">Register</button>
        <button type="reset">Clear Form</button>
      </p>
    </form>
  </body>
</html>
```

**What to expect**: a long but completely functional form, neatly grouped into five fieldsets. Try clicking **Register** with empty fields — the browser stops you and highlights the first problem. Try typing letters in the phone field — the pattern check fails on submit.

> The number next to "Hours of study per week" is hard-coded to "10" for now. We will make it update live when we hit JavaScript events in Class 23.

---

## Practice Exercises

### Exercise 1 — Build a feedback form (15 minutes)
Create `feedback.html` with the following fields, all properly labelled and validated:

- **Name** (required, 2–50 chars)
- **Email** (required)
- **Rating** (radio buttons: 1–5 stars, required)
- **Categories** (checkboxes: Speed, Accuracy, Friendliness, Value — pick any)
- **Comments** (`<textarea>`, max 500 chars)
- A **Submit** and a **Reset** button

Wrap each section in a `<fieldset>` with a `<legend>`.

### Exercise 2 — Test the validation
Try to submit the form leaving each required field empty in turn. Confirm the browser blocks the submission and tells you why.

### Exercise 3 — Make the file input accept images only
Add a "Profile photo" file input that only accepts JPEG and PNG files (`accept="image/jpeg,image/png"`).

---

## Key Takeaways
- **Checkboxes** = zero or many; **radios** = exactly one (same `name` across the group).
- `<select>` saves space when the list is long; use `<optgroup>` to group options.
- `<textarea>` needs a closing tag and accepts `rows`, `cols`, and `maxlength`.
- HTML5 ships with **free validation**: `required`, `min`/`max`, `minlength`/`maxlength`, `pattern`, plus typed inputs like `email` and `url`.
- Group sections with `<fieldset>` + `<legend>` — better visuals and better accessibility.
- Built-in validation is **not** security — it is a first line of defence only.

Next class we will look at **semantic HTML** — using the right tag for the right job — and learn the basics of **accessibility** so our pages work for everyone.
