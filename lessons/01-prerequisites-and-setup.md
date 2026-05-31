# Class 1: Prerequisites & Environment Setup

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: None — this is the first class. You only need a laptop (macOS or Windows) with administrator rights and a working internet connection.

## What You Will Learn
- Why we spend the very first class on tools (and not on code)
- How to install **Node.js**, **VS Code**, and **Git** on macOS and Windows
- How to configure Git so your work is credited to you
- How to create the **GitHub** and **Vercel** accounts we will use all semester
- Which **VS Code extensions** to install for HTML, CSS, JS, and React
- A quick tour of **VS Code**, **browser DevTools**, and the **Live Server** extension

---

## 1.1 Why a "Setup Day" Matters

Imagine starting a cooking class without an oven, knife, or hob. You can read every recipe in the world and still cook nothing. Programming is the same — the *kitchen* must be ready before we cook anything.

A developer's kitchen has four core appliances:

| Tool | What it does | Real-world analogy |
|------|--------------|--------------------|
| **Node.js** | Runs JavaScript outside the browser; powers our build tools | The cooker — gives energy to everything |
| **VS Code** | Where we write code | The kitchen worktop — clean, well-lit, organised |
| **Git** | Tracks every change you make to your code | The recipe book that remembers every edit |
| **GitHub + Vercel** | Stores your code online and puts your website on the internet | The shop window that displays your work |

Once these are in place, every future class can focus on the *cooking* — writing actual code — instead of fighting with setup.

---

## 1.2 Check What You Already Have

Open a terminal:

- **macOS**: press `Cmd` + `Space`, type **Terminal**, press `Enter`.
- **Windows**: press the **Windows** key, type **PowerShell**, press `Enter`.

Type each of these and press `Enter`:

```bash
node -v
npm -v
git --version
code -v
```

If a command returns a version number (for example `v20.11.1`), it is already installed. If you see "command not found" or "is not recognised", we will install it in the next steps. Do not worry — this is normal on a fresh machine.

---

## 1.3 Install Node.js (LTS Version)

**Node.js** lets us run JavaScript on our own machine and powers the build tools we will use later (Vite, npm, and so on). We always pick the **LTS** ("Long-Term Support") version — it is the stable one businesses use.

### Step 1 — Download

Go to **https://nodejs.org** and click the green **LTS** button. The site usually detects your operating system automatically.

### Step 2 — Install

- **macOS**: open the downloaded `.pkg` file and click **Continue** → **Continue** → **Agree** → **Install**. Enter your Mac password when asked.
- **Windows**: open the downloaded `.msi` file and click **Next** → **I accept** → **Next** → **Next** → **Install**. Allow it to make changes when Windows asks.

### Step 3 — Verify

**Close and reopen your terminal**, then run:

```bash
node -v
npm -v
```

You should see something like:

```
v20.11.1
10.2.4
```

If both commands print a version, Node is installed correctly. `npm` (Node Package Manager) is bundled with Node — you get two tools for one install.

> **Tip**: Your exact version numbers will differ. As long as Node is **v18 or higher**, you are fine for this course.

---

## 1.4 Install VS Code

**Visual Studio Code** (VS Code) is a free, lightweight code editor made by Microsoft. It is the most popular editor among professional web developers.

### Step 1 — Download

Go to **https://code.visualstudio.com** and click the big **Download** button. The site will pick the right installer for your OS.

### Step 2 — Install

- **macOS**: open the downloaded `.zip`, drag the **Visual Studio Code** icon into your **Applications** folder, then open it from there.
- **Windows**: run the `.exe` installer. On the *Select Additional Tasks* screen, tick **all** the boxes — especially **"Add to PATH"** and **"Open with Code"**.

### Step 3 — Verify

Open VS Code, then in your terminal type:

```bash
code -v
```

If you see a version number, you can launch VS Code straight from the terminal — handy later when we say "open this folder in VS Code".

> **macOS extra step**: In VS Code, press `Cmd` + `Shift` + `P`, type **"Shell Command: Install 'code' command in PATH"**, and press `Enter`. This enables the `code` command in your terminal.

---

## 1.5 A Quick Tour of VS Code

Open VS Code and look at the five main areas:

| Area | Where it is | What it does |
|------|-------------|--------------|
| **Activity Bar** | Far left, vertical | Switches between Explorer, Search, Source Control, Extensions, and Debug |
| **Side Bar** | Next to the Activity Bar | Shows files in your project (when in Explorer mode) |
| **Editor** | Centre, large area | Where you actually write code |
| **Terminal panel** | Bottom (open with `` Ctrl + ` ``) | A terminal built into the editor — very convenient |
| **Status Bar** | Bottom strip | Shows the file type, line number, Git branch, and errors |

A few keyboard shortcuts you will use every single day:

| Action | macOS | Windows |
|--------|-------|---------|
| Open Command Palette | `Cmd + Shift + P` | `Ctrl + Shift + P` |
| Save file | `Cmd + S` | `Ctrl + S` |
| Open terminal | `` Ctrl + ` `` | `` Ctrl + ` `` |
| Toggle side bar | `Cmd + B` | `Ctrl + B` |
| Format document | `Shift + Option + F` | `Shift + Alt + F` |

Try them now — practice makes them muscle memory.

---

## 1.6 Install the VS Code Extensions We Will Use

Extensions add helpful features to VS Code. To install one, click the **Extensions** icon on the Activity Bar (the four squares), type the name in the search box, and click **Install**.

Install **all** of these now — we will use them in later classes:

| Extension | Why we need it |
|-----------|----------------|
| **ES7+ React/Redux/React-Native snippets** | Shortcut keystrokes that generate React boilerplate (later in the course) |
| **Tailwind CSS IntelliSense** | Autocomplete for Tailwind class names |
| **Prettier — Code formatter** | Automatically tidies your code on save |
| **ESLint** | Highlights JavaScript/TypeScript mistakes as you type |
| **Auto Rename Tag** | When you rename `<div>` to `<section>`, the closing tag updates automatically |
| **Path Intellisense** | Autocompletes file paths inside `import` and `href` statements |
| **Live Server** | Opens your HTML file in the browser and reloads it whenever you save |
| **GitLens** | Shows who changed each line and when — invaluable in team work |
| **Material Icon Theme** | Pretty icons next to files so you can spot file types at a glance |

### Make Prettier format on save

Open the Command Palette (`Cmd/Ctrl + Shift + P`), type **"Preferences: Open Settings (UI)"**, then search:

1. **Default Formatter** → choose **Prettier — Code formatter**.
2. **Format On Save** → tick the box.

Now every time you press save, Prettier tidies your code for you.

---

## 1.7 Install Git

**Git** is the world standard for version control. Think of it as a "save game" feature for your code — you can rewind to any point in your project's history.

### Step 1 — Install

- **macOS**: in the terminal, type:
  ```bash
  git --version
  ```
  If Git is missing, macOS will pop up a dialog offering to install the **Command Line Developer Tools**. Click **Install** and wait a few minutes.
- **Windows**: go to **https://git-scm.com/download/win**, download the installer, and run it. Accept all the default options — they are fine.

### Step 2 — Verify

```bash
git --version
```

You should see something like `git version 2.43.0`.

### Step 3 — Configure your identity

Git wants to know who you are so every saved change is signed with your name. Run these two commands, replacing the placeholders with your own details:

```bash
git config --global user.name "Your Full Name"
git config --global user.email "you@example.com"
```

Use the same email you will use for your GitHub account in the next step — it will save trouble later.

Check it worked:

```bash
git config --global user.name
git config --global user.email
```

You should see exactly what you just typed.

> **Tip**: Set the default branch name to `main` (the modern convention):
> ```bash
> git config --global init.defaultBranch main
> ```

---

## 1.8 Create a GitHub Account

**GitHub** is the website where developers store and share Git projects. Think of it as **Google Drive for code**.

1. Go to **https://github.com** and click **Sign up**.
2. Use the **same email** you gave to Git in the previous step.
3. Choose a sensible username — many employers will look at your GitHub profile, so keep it professional (`prabin-karki` is better than `xX_dragonslayer_Xx`).
4. Complete the email verification.

You do not need a paid plan. The free tier is more than enough for this entire course.

---

## 1.9 Create a Vercel Account

**Vercel** is a hosting service that puts your website on the internet for free. We will deploy our final React project there in Class 51.

1. Go to **https://vercel.com** and click **Sign up**.
2. Click **Continue with GitHub** — this links the two accounts so deployments happen automatically later.
3. Authorise Vercel when GitHub asks.

That is all — we will not deploy anything today, but the account is now ready when we need it.

---

## 1.10 Your First HTML File with Live Server

Time to actually use what we installed.

### Step 1 — Create a project folder

Pick a sensible place on your computer (for example `Documents/triton-react`) and create a folder called `class-01`.

### Step 2 — Open it in VS Code

In the terminal:

```bash
cd Documents/triton-react/class-01
code .
```

The dot means "the current folder". VS Code opens with this folder as the workspace.

### Step 3 — Create `index.html`

In the Explorer side bar, click the **New File** icon and name it `index.html`. Paste this in:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My First Page</title>
  </head>
  <body>
    <h1>Hello, world!</h1>
    <p>Today I set up my development environment.</p>
  </body>
</html>
```

Save the file (`Cmd/Ctrl + S`).

### Step 4 — Open with Live Server

Right-click anywhere inside the file and choose **"Open with Live Server"**.

Your default browser opens at an address such as `http://127.0.0.1:5500/index.html` and shows your page. Now change "Hello, world!" to "Hello, Triton College!" and save — the page reloads automatically. This live reload will save you hours over the semester.

---

## 1.11 Browser DevTools — A 5-Minute Tour

Every modern browser ships with **DevTools** — a built-in inspector that lets you look inside any web page.

Open them on your "Hello, world" page:

- **macOS**: `Cmd + Option + I`
- **Windows**: `F12` or `Ctrl + Shift + I`

You will see several tabs. For now, only two matter:

| Tab | What it shows |
|-----|---------------|
| **Elements** | The live HTML structure of the page. You can edit it on the fly to experiment. |
| **Console** | Where JavaScript messages, errors and warnings appear. We will live here from Class 16 onwards. |

Try this:

1. Click the **Elements** tab.
2. Find the `<h1>` line.
3. Double-click the text and change it to **"Hello, DevTools!"** — the page updates instantly.

These changes are temporary (refresh the page and they disappear), but they are perfect for quick experiments.

---

## Practice Exercises

### Exercise 1 — Verify your install (10 minutes)
Run all four commands and write the version numbers in a notes file:

```bash
node -v
npm -v
git --version
code -v
```

### Exercise 2 — Your "Hello, world" page (10 minutes)
Create a folder called `hello-world`, add an `index.html` file with the boilerplate from Section 1.10, and open it with Live Server. Edit the heading to greet you by name, then save and watch the page reload.

### Exercise 3 — DevTools experiment (5 minutes)
Open DevTools on your page, change the heading text from the Elements tab, then refresh the page. Confirm that the change disappears — DevTools edits are *temporary*.

---

## Key Takeaways
- A fully set-up environment is the foundation for the rest of the course — never skip it.
- **Node.js LTS**, **VS Code**, **Git**, **GitHub**, and **Vercel** are the five core pieces.
- VS Code extensions and Prettier-on-save make daily work much easier.
- **Live Server** auto-reloads your page in the browser whenever you save — your new best friend.
- **DevTools** lets you peek inside any web page and experiment safely.

Next class we will start writing real HTML and learn the structure of every web page on the internet.
