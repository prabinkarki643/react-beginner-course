# Class 7: HTML Practice — Build a Portfolio Page

> **Time budget**: 1 hour (≈ 15 min plan + 35 min build + 10 min review)
> **Prerequisites**: Classes 2–6 — the entire HTML toolkit.

## What You Will Learn
- How to **plan** a multi-section page before you write any HTML
- How to combine **semantic landmarks**, **lists**, **images**, **tables**, and **forms** into one cohesive page
- How to validate your HTML using the **W3C Validator**
- The full, finished `portfolio.html` — you will keep this file and **style it in Class 15**

This is a **workshop class**. There is little new theory. Most of the time is spent typing, saving, and refreshing.

---

## 7.1 The Brief

Build a single HTML file called `portfolio.html` with **four sections**:

1. **Hero** — your name, a short tagline, and a photo or placeholder.
2. **About** — a few paragraphs about you, your background, and the technologies you are learning.
3. **Projects** — a list of three projects you have made (or plan to make). Each one has a title, a one-line description, and a link.
4. **Contact** — a working contact form.

Plus an overall **header** at the top with a navigation menu and a **footer** at the bottom.

**Constraints:**

- HTML only — **no CSS, no JavaScript**.
- Use **semantic landmarks** (`<header>`, `<nav>`, `<main>`, `<section>`, `<footer>`) everywhere appropriate.
- Every `<img>` must have a meaningful `alt`.
- Every form input must have a linked `<label>`.
- The page must pass the W3C HTML Validator with **zero errors**.

You will keep this file. In Class 15 we will style it. In Class 27 we will publish it to GitHub.

---

## 7.2 Plan Before You Type (≈ 5 minutes)

Even tiny pages benefit from a quick plan. Open a notes file or a paper page and sketch the heading outline. For this portfolio it should look like:

```
h1  Your name
  h2  About
  h2  Projects
    h3  Project 1 name
    h3  Project 2 name
    h3  Project 3 name
  h2  Contact
```

One `<h1>` (your name), three `<h2>`s (one per section), and `<h3>`s for the individual projects. Skipping levels is forbidden, as discussed in Class 6.

Now sketch the landmarks:

```
header
  h1 + nav (links to #about, #projects, #contact)
main
  section#hero      (with the h1 and tagline)
  section#about
  section#projects
  section#contact
footer
  copyright
```

Notice the *anchor IDs* (`#about`, `#projects`, `#contact`) — clicking the nav links will jump to those sections. We met anchor links in Class 3.

---

## 7.3 Build It in Five Slices

We will build the page in five small pieces, refreshing in Live Server after each one. Going slice by slice keeps mistakes obvious and easy to fix.

### Slice 1 — Boilerplate + header + nav

Open VS Code, create `portfolio.html`, type `!` and `Tab` for the boilerplate, then add a header with a nav:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Sita Sharma — Portfolio</title>
  </head>
  <body>
    <header>
      <h1>Sita Sharma</h1>
      <nav aria-label="Main">
        <ul>
          <li><a href="#about">About</a></li>
          <li><a href="#projects">Projects</a></li>
          <li><a href="#contact">Contact</a></li>
        </ul>
      </nav>
    </header>
  </body>
</html>
```

Save and refresh — you should see a large name and three nav links.

> The `<h1>` is here in the header because the page is *about you*. The hero section will not have its own `<h1>` — we already used our one.

### Slice 2 — `<main>` and the About section

Inside `<body>`, add a `<main>` element with the About section:

```html
<main>
  <section id="about">
    <h2>About Me</h2>
    <img
      src="https://placehold.co/200x200"
      alt="A placeholder for my profile photo"
      width="200"
      height="200"
    />
    <p>
      Hello! I am <strong>Sita Sharma</strong>, a BCA Semester 2 student at
      <em>Triton College</em>. I am currently studying modern front-end
      development with HTML, CSS, JavaScript, and React.
    </p>
    <p>
      In my spare time I enjoy reading novels, playing the guitar, and
      experimenting with small web projects.
    </p>

    <h3>Technologies I am learning</h3>
    <ul>
      <li>HTML &amp; CSS</li>
      <li>JavaScript (ES6+)</li>
      <li>TypeScript</li>
      <li>React with Vite</li>
      <li>Tailwind CSS and shadcn/ui</li>
    </ul>
  </section>
</main>
```

Save, refresh. You should see your About section appear underneath the header. Click the "About" link in the nav — the page jumps to this section.

### Slice 3 — The Projects section

Add this *inside* `<main>`, after the About section:

```html
<section id="projects">
  <h2>Projects</h2>

  <article>
    <h3>Personal Portfolio</h3>
    <p>The page you are reading right now — built with pure HTML.</p>
    <p>
      <a href="#" target="_blank" rel="noopener noreferrer">View project</a>
    </p>
  </article>

  <article>
    <h3>To-Do List App</h3>
    <p>A simple in-browser to-do list. Planned with JavaScript in Class 22.</p>
    <p>
      <a href="#" target="_blank" rel="noopener noreferrer">View project</a>
    </p>
  </article>

  <article>
    <h3>Users &amp; Posts Dashboard</h3>
    <p>
      The capstone project for this course — a React app showing users and
      posts from the JSONPlaceholder API.
    </p>
    <p>
      <a href="#" target="_blank" rel="noopener noreferrer">View project</a>
    </p>
  </article>
</section>
```

Each project is its own `<article>` because it is *self-contained* (you could lift it out of the page and it would still make sense — that is the test for `<article>`).

The "View project" links currently go to `#` (the top of the page) — replace them with real URLs once those projects exist.

### Slice 4 — The Contact section (with a form)

Still inside `<main>`, after the Projects section:

```html
<section id="contact">
  <h2>Contact</h2>
  <p>Got a question or a project idea? Drop me a message.</p>

  <form>
    <p>
      <label for="contact-name">Your Name</label><br />
      <input
        type="text"
        id="contact-name"
        name="name"
        required
        minlength="2"
        maxlength="60"
        placeholder="e.g. Ram Karki"
      />
    </p>

    <p>
      <label for="contact-email">Your Email</label><br />
      <input
        type="email"
        id="contact-email"
        name="email"
        required
        placeholder="you@example.com"
      />
    </p>

    <p>
      <label for="contact-subject">Subject</label><br />
      <select id="contact-subject" name="subject" required>
        <option value="">-- Choose a topic --</option>
        <option value="job">A job opportunity</option>
        <option value="collab">A collaboration idea</option>
        <option value="question">A question about my work</option>
        <option value="other">Something else</option>
      </select>
    </p>

    <p>
      <label for="contact-message">Message</label><br />
      <textarea
        id="contact-message"
        name="message"
        rows="5"
        cols="50"
        required
        maxlength="500"
        placeholder="Tell me a little about it..."
      ></textarea>
    </p>

    <p>
      <label>
        <input type="checkbox" name="copy" />
        Send a copy of this message to my own inbox
      </label>
    </p>

    <p>
      <button type="submit">Send Message</button>
      <button type="reset">Clear</button>
    </p>
  </form>
</section>
```

Save, refresh, and try the form. Try clicking "Send Message" with empty fields — the browser's built-in validation stops you. The form will not actually send anywhere yet (we have no JavaScript), but the markup is ready.

### Slice 5 — The Footer

Outside `<main>`, just before `</body>`:

```html
<footer>
  <p>&copy; 2026 Sita Sharma. Built with pure HTML.</p>
  <nav aria-label="Footer">
    <ul>
      <li><a href="#about">About</a></li>
      <li><a href="#projects">Projects</a></li>
      <li><a href="#contact">Contact</a></li>
    </ul>
  </nav>
</footer>
```

The footer repeats the section links — common pattern on long pages.

---

## 7.4 Validating Your HTML

Now that the page is complete, let us check that the markup is technically correct.

1. Go to **https://validator.w3.org/#validate_by_input**.
2. Copy the entire contents of your `portfolio.html`.
3. Paste it into the **"Validate by Direct Input"** box.
4. Click **Check**.

A clean page returns: *"Document checking completed. No errors or warnings to show."*

If you see errors, read them carefully — the validator points to the exact line. The most common beginner mistakes are:

- An unclosed tag (e.g. `<p>` with no `</p>`).
- An attribute without quotes (`<input type=text>`).
- A label `for` pointing to an `id` that does not exist.

Fix them, re-validate, repeat.

> Validating is not optional. Browsers are very forgiving and will often *try* to render broken HTML — but the result is unpredictable. Always validate before you call a page "done".

---

## 7.5 The Complete Final HTML

Here is the full file you should now have, ready to save and keep. Replace the example name, photo path, and project descriptions with your own.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Sita Sharma — Portfolio</title>
  </head>
  <body>
    <header>
      <h1>Sita Sharma</h1>
      <p>Aspiring front-end developer · BCA Semester 2 · Triton College</p>

      <nav aria-label="Main">
        <ul>
          <li><a href="#about">About</a></li>
          <li><a href="#projects">Projects</a></li>
          <li><a href="#contact">Contact</a></li>
        </ul>
      </nav>
    </header>

    <main>
      <!-- ABOUT -->
      <section id="about">
        <h2>About Me</h2>
        <img
          src="https://placehold.co/200x200"
          alt="A placeholder for my profile photo"
          width="200"
          height="200"
        />
        <p>
          Hello! I am <strong>Sita Sharma</strong>, a BCA Semester 2 student
          at <em>Triton College</em>. I am currently studying modern
          front-end development with HTML, CSS, JavaScript, and React.
        </p>
        <p>
          In my spare time I enjoy reading novels, playing the guitar, and
          experimenting with small web projects.
        </p>

        <h3>Technologies I am learning</h3>
        <ul>
          <li>HTML &amp; CSS</li>
          <li>JavaScript (ES6+)</li>
          <li>TypeScript</li>
          <li>React with Vite</li>
          <li>Tailwind CSS and shadcn/ui</li>
        </ul>

        <h3>Recent Coursework</h3>
        <table>
          <caption>Marks in the current semester</caption>
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
      </section>

      <!-- PROJECTS -->
      <section id="projects">
        <h2>Projects</h2>

        <article>
          <h3>Personal Portfolio</h3>
          <p>The page you are reading right now — built with pure HTML.</p>
          <p>
            <a href="#" target="_blank" rel="noopener noreferrer">
              View project
            </a>
          </p>
        </article>

        <article>
          <h3>To-Do List App</h3>
          <p>
            A simple in-browser to-do list. Planned with JavaScript in
            Class 22.
          </p>
          <p>
            <a href="#" target="_blank" rel="noopener noreferrer">
              View project
            </a>
          </p>
        </article>

        <article>
          <h3>Users &amp; Posts Dashboard</h3>
          <p>
            The capstone project for this course — a React app showing
            users and posts from the JSONPlaceholder API.
          </p>
          <p>
            <a href="#" target="_blank" rel="noopener noreferrer">
              View project
            </a>
          </p>
        </article>
      </section>

      <!-- CONTACT -->
      <section id="contact">
        <h2>Contact</h2>
        <p>Got a question or a project idea? Drop me a message.</p>

        <form>
          <p>
            <label for="contact-name">Your Name</label><br />
            <input
              type="text"
              id="contact-name"
              name="name"
              required
              minlength="2"
              maxlength="60"
              placeholder="e.g. Ram Karki"
            />
          </p>

          <p>
            <label for="contact-email">Your Email</label><br />
            <input
              type="email"
              id="contact-email"
              name="email"
              required
              placeholder="you@example.com"
            />
          </p>

          <p>
            <label for="contact-subject">Subject</label><br />
            <select id="contact-subject" name="subject" required>
              <option value="">-- Choose a topic --</option>
              <option value="job">A job opportunity</option>
              <option value="collab">A collaboration idea</option>
              <option value="question">A question about my work</option>
              <option value="other">Something else</option>
            </select>
          </p>

          <p>
            <label for="contact-message">Message</label><br />
            <textarea
              id="contact-message"
              name="message"
              rows="5"
              cols="50"
              required
              maxlength="500"
              placeholder="Tell me a little about it..."
            ></textarea>
          </p>

          <p>
            <label>
              <input type="checkbox" name="copy" />
              Send a copy of this message to my own inbox
            </label>
          </p>

          <p>
            <button type="submit">Send Message</button>
            <button type="reset">Clear</button>
          </p>
        </form>
      </section>
    </main>

    <footer>
      <p>&copy; 2026 Sita Sharma. Built with pure HTML.</p>
      <nav aria-label="Footer">
        <ul>
          <li><a href="#about">About</a></li>
          <li><a href="#projects">Projects</a></li>
          <li><a href="#contact">Contact</a></li>
        </ul>
      </nav>
    </footer>
  </body>
</html>
```

Open this in Live Server. It will look very plain — no colours, no fancy layout. That is intentional. In Class 15 you will style this exact file into a polished, responsive portfolio. **Do not delete it.**

---

## 7.6 A Quick Self-Check

Before you call the exercise finished, tick all of these:

- [ ] The page has exactly **one `<h1>`**.
- [ ] Heading levels go in order — no skipping (`h1` → `h2` → `h3`).
- [ ] Every section uses a **semantic landmark** where appropriate.
- [ ] The `<nav>` links use `#id` and jump to the right sections.
- [ ] Every `<img>` has meaningful `alt` text.
- [ ] Every form input has a linked `<label>`.
- [ ] Required fields use `required`.
- [ ] The page passes the **W3C Validator** with zero errors.

---

## Practice Exercises

### Exercise 1 — Personalise the file (10 minutes)
Replace the placeholder name, photo URL, About paragraphs, and the three projects with your own real details. Keep the structure.

### Exercise 2 — Add a "Skills" subsection
Inside the About section, add a new `<h3>Skills</h3>` followed by a `<ul>` with at least five skills (HTML, CSS, JS, Git, English, problem-solving — anything truthful).

### Exercise 3 — Validate and screenshot
Run your final file through the W3C Validator. If it passes, take a screenshot of the green "No errors" message. If not, fix the listed errors and try again. Save the file — we will need it in Class 15.

---

## Key Takeaways
- **Plan first**: heading outline → landmarks → content. Five minutes of planning saves twenty minutes of refactoring.
- Build in **small slices** and refresh after each one. Mistakes are easy to find that way.
- Real-world portfolio pages combine **everything**: text, images, lists, tables, forms, semantic landmarks.
- The **W3C Validator** is your sanity check — pass it before you call any HTML "done".
- Pure HTML pages look plain. **That is correct.** Styling is CSS's job, starting Class 8.

Next class we will start **CSS** — the layer that turns this skeleton into something people actually enjoy looking at.
