# Class 27: Git and GitHub Basics

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 26 (you can open a terminal and run commands inside a project folder)

## What You Will Learn
- Why version control matters — the "save game" analogy
- Initialising a repository and recording snapshots with `git add` and `git commit`
- Writing commit messages developers will actually want to read
- The role of `.gitignore`
- Working on a feature branch and merging it back
- Connecting a local project to GitHub and pushing it
- Cloning, pull requests, and the basics of collaboration
- Recovering from the three most common Git mistakes

---

## 27.1 Why version control?

Imagine you are playing a long video game. You spend three hours building a base, then a dragon shows up and destroys it. Without saves, you have to start again from scratch. With saves, you reload and try a different tactic.

**Git is the save-game system for your code.** Every time you "save" (we call it a **commit**), Git takes a snapshot of every file in your project. You can later:

- Look at any old snapshot.
- Compare two versions to see exactly what changed.
- Revert to a previous state if something goes wrong.
- Try a risky experiment on a separate "save slot" (a **branch**) without touching the main one.

**GitHub** is a website that hosts your Git repositories online so you can back them up, share them, and collaborate with other developers. **Git is the tool, GitHub is the cloud service.** You can use Git without GitHub, but in real projects you almost always use both together.

You should already have Git installed from Class 1. Confirm it:

```bash
git --version
```

If you see a version like `git version 2.42.0`, you are ready.

---

## 27.2 First-time setup (once per machine)

Git needs to know who you are so it can label every commit with your name and email. Set this once per machine:

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# On modern Git, default the main branch name:
git config --global init.defaultBranch main
```

> Use the **same email you used on GitHub** so your commits link up to your profile.

---

## 27.3 Initialising a repository

Let's turn an ordinary folder into a Git project. Reuse the `greeter-app` from Class 26 (or make a new folder).

```bash
cd greeter-app
git init
```

You will see:

```
Initialized empty Git repository in /Users/you/greeter-app/.git/
```

A hidden `.git/` folder has just been created. **That folder is your repository** — it stores every commit, every branch, every change. Delete it and the project becomes an ordinary folder again.

Check the current state:

```bash
git status
```

You will see all your files listed as **Untracked**. They exist in the folder, but Git is not yet watching them.

---

## 27.4 The three-stage workflow

Git has three "areas" your files move between:

```
Working directory   →   Staging area   →   Repository
   (your edits)         (git add)          (git commit)
```

- **Working directory**: the files on disk you are editing right now.
- **Staging area**: a holding bay for changes you want to include in the next commit.
- **Repository**: the permanent history of saved snapshots.

> **Analogy:** writing a shopping list. You **add** items you want this week to the list (`git add`), then you go shopping and **commit** to buying them (`git commit`). Items you decide against never leave the working directory.

---

## 27.5 Your first commit

Inside `greeter-app`:

```bash
# 1. See what Git can see
git status

# 2. Stage all changes (the . means "everything in this folder")
git add .

# 3. Stage just one file (more precise)
git add index.js

# 4. Save the snapshot with a message
git commit -m "Initial commit: greeter app skeleton"
```

**Expected output (commit):**

```
[main (root-commit) e7a1b2c] Initial commit: greeter app skeleton
 4 files changed, 27 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 index.js
 create mode 100644 package.json
 create mode 100644 package-lock.json
```

Now `git status` will say `nothing to commit, working tree clean`. You have your first save.

To see history:

```bash
git log --oneline
```

You should see one line: your commit's short ID and message.

---

## 27.6 Writing good commit messages

A commit message is a tiny note to your future self (and your team) explaining **why** this change happened. Future-you will thank present-you for taking five seconds longer.

**Use the imperative mood**, as if completing the sentence *"If applied, this commit will…"*:

| Good | Bad |
|------|-----|
| `Add login form` | `Added login form` |
| `Fix typo in README` | `fixed some stuff` |
| `Remove unused chalk import` | `update` |
| `Refactor greet function for readability` | `final final v2 FIXED` |

A simple rule: **first line 50 characters or fewer, capitalised, no full stop.** If you need more detail, leave a blank line and write a longer paragraph beneath.

```bash
git commit -m "Add coloured greeting to CLI" -m "Uses chalk to make the greeting easier to read at a glance."
```

---

## 27.7 `.gitignore` essentials

We saw `.gitignore` briefly in Class 26. Now we know why it matters: anything listed in `.gitignore` is **invisible to Git** — it is never tracked, never staged, never committed.

A starter `.gitignore` for a Node/React project:

```
# Dependencies
node_modules

# Build output
dist
build

# Environment variables (never commit secrets!)
.env
.env.local

# Editor and OS files
.DS_Store
.vscode/*
!.vscode/settings.json

# Logs
*.log
```

> **Rule of thumb:** if a file is **generated**, **personal to one machine**, or **secret**, it belongs in `.gitignore`.

Commit your `.gitignore` *before* installing any packages, ideally as part of your first commit. If you have already committed `node_modules` by mistake, see § 27.13.

---

## 27.8 Branching basics

A **branch** is an independent line of work. You start a branch when you want to try something new without disturbing your stable code.

```bash
# Create a branch and switch to it in one go
git switch -c feature/coloured-output

# See all branches
git branch
```

The `*` next to a branch shows which one you are on.

Now make a change — for example, update the greeting in `index.js`. Stage and commit it on the branch:

```bash
git add index.js
git commit -m "Add yellow tagline to greeting"
```

The `main` branch is untouched. You have created a parallel save file.

### Merging the branch back

When you are happy with the feature, return to `main` and **merge** the work in:

```bash
git switch main
git merge feature/coloured-output
```

You will see one of two outcomes:

| Outcome | What it means |
|---------|---------------|
| **Fast-forward** | `main` had no new commits since the branch split, so Git simply moves the `main` pointer to the tip of your feature branch. Clean and linear. |
| **Merge commit** | `main` had its own new commits in the meantime, so Git creates a new commit that joins the two histories together. |

Delete the branch once it is merged:

```bash
git branch -d feature/coloured-output
```

> **Why bother with branches in a solo project?** Because real teams work in branches, and forming the habit now will save your bacon when you start collaborating.

---

## 27.9 Pushing to GitHub

Now we take this local repository and push it to the cloud. You should already have a GitHub account from Class 1.

### Step 1 — create an empty repo on GitHub

1. Go to [github.com](https://github.com), click **New repository**.
2. Name it `greeter-app`.
3. **Do not** tick "Add a README" or any other initialiser — we already have a local project.
4. Click **Create repository**.

### Step 2 — connect your local repo to GitHub

GitHub will show two URLs (HTTPS and SSH). Copy the **HTTPS** one — it looks like `https://github.com/yourname/greeter-app.git`.

```bash
# Tell Git where "origin" lives
git remote add origin https://github.com/yourname/greeter-app.git

# Rename your branch to main if it is still called master
git branch -M main

# Push and remember the link (-u sets the upstream)
git push -u origin main
```

The `-u` part means "from now on, `git push` and `git pull` know which remote branch to use without me typing it." You will only need it on the first push.

Refresh the GitHub page and your files will appear. Welcome to the cloud.

---

## 27.10 Cloning an existing project

Going the other way — bringing a GitHub project down onto your machine — is one command:

```bash
git clone https://github.com/some-user/some-project.git
cd some-project
npm install
```

That is how you start work on every project at a new job, every shadcn template, every open-source library.

---

## 27.11 Pull requests (the concept)

When you work with others (or with your future self), you do not usually push straight to `main`. Instead:

1. **Branch** off `main`.
2. Make and push your commits on the branch.
3. Open a **Pull Request (PR)** on GitHub — a polite request that says *"please review my branch and merge it into main."*
4. Teammates review the diff, leave comments, you push fixes.
5. Once approved, the PR is **merged** on GitHub and the branch can be deleted.

Pull requests are how every serious team ships code. You will use them in the capstone project.

---

## 27.12 The daily commands you will actually use

```bash
git status                    # What has changed?
git add <file>                # Stage one file
git add .                     # Stage everything
git commit -m "Message"       # Save a snapshot
git log --oneline             # See history
git switch -c new-branch      # Create + switch to a branch
git switch main               # Switch to an existing branch
git merge other-branch        # Merge another branch into the current one
git push                      # Send commits to GitHub
git pull                      # Get the latest commits from GitHub
git clone <url>               # Copy a remote repo locally
```

Bookmark this block. 90% of your Git life is right here.

---

## 27.13 Recovering from common mistakes

Git looks scary because so many people have made horror stories out of these three situations. Here is the calm version.

### Mistake 1 — "I changed a file but I want to throw the changes away"

```bash
# Unstage AND undo a file
git restore index.js

# Undo every uncommitted change in the project (use with care)
git restore .
```

Your last commit is untouched. Only the unsaved edits disappear.

### Mistake 2 — "I just committed but I want to undo the commit (keep the changes)"

You committed too early, or used a bad message, or staged the wrong thing.

```bash
git reset --soft HEAD~1
```

This rewinds the last commit but leaves all your changes **staged**, ready to recommit. You can edit files, restage, and commit again with a better message.

> If you want to undo the commit **and** unstage everything (but keep the file changes on disk), use `git reset HEAD~1` (without `--soft`).
>
> Never use `git reset --hard` unless you really mean "throw the work away forever".

### Mistake 3 — "I committed straight to `main` by accident; that should have been a branch"

This happens to everyone at some point. The fix:

```bash
# 1. Create a branch at the current commit (saves your work)
git switch -c feature/my-work

# 2. Switch back to main
git switch main

# 3. Rewind main by one commit
git reset --hard HEAD~1

# 4. Carry on working on your feature branch
git switch feature/my-work
```

Your accidental commit is now safely on `feature/my-work`, and `main` is back to where it should have been. **Only do this if you have not pushed yet.**

### Bonus — "I accidentally committed `node_modules`"

```bash
# Add node_modules to .gitignore first, then:
git rm -r --cached node_modules
git commit -m "Remove node_modules from tracking"
git push
```

---

## Practice Exercises

### Exercise 1: Take your portfolio public
1. Open the portfolio page you built in Class 7 / 15.
2. `cd` into it and run `git init`.
3. Create a `.gitignore` with `node_modules`, `.DS_Store` and `.env`.
4. Make your first commit: `"Initial commit: portfolio page"`.
5. Create a new GitHub repo called `my-portfolio`.
6. Connect and push following § 27.9. Confirm it appears at `github.com/yourname/my-portfolio`.

### Exercise 2: Practise a branch flow
1. In the portfolio repo, create a branch: `git switch -c feature/new-section`.
2. Add a "Skills" section to your HTML, commit on the branch.
3. Switch back to `main`, merge the branch, delete it.
4. Push to GitHub and check the commit appears in the history tab.

### Exercise 3: Survive a mistake
1. Make a change to any file and run `git commit -am "oops bad message"`.
2. Use `git reset --soft HEAD~1` to undo just the commit (keep the changes staged).
3. Recommit with a sensible message in imperative mood.
4. Run `git log --oneline` and confirm history looks clean.

---

## Key Takeaways
- **Git** records snapshots of your project; **GitHub** stores them in the cloud.
- The flow is **edit → `git add` → `git commit` → `git push`**.
- Write **imperative-mood**, sub-50-character commit messages.
- **`.gitignore`** keeps generated, secret and personal files out of the repo.
- **Branches** let you experiment safely; **merges** bring the work back into `main`.
- **`git restore`** undoes file edits; **`git reset --soft HEAD~1`** rewinds a single commit while keeping your work.
- Next class we introduce TypeScript — the language we will use for every line of React from Class 29 onwards.
