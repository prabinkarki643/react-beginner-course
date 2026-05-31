# Class 25: Async — Promises, async/await, Fetch

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 24 — ES6+ features and modules

This is the final JavaScript class — and one of the most exciting. Until now, our programs have been **self-contained**: every value lives inside the same file. Today, we open the door to the wider internet. We will ask a real public server for data, wait for it to arrive, and render it on the page. This is exactly how React apps fetch data from APIs.

## What You Will Learn
- Synchronous vs asynchronous code (why we need promises at all)
- Promises and the `.then()` / `.catch()` / `.finally()` chain
- `async` / `await` — the modern, readable form
- The `fetch` API and how to read JSON responses
- Checking `response.ok` for HTTP errors
- A complete page that fetches and renders 10 posts from JSONPlaceholder

---

## 25.1 Synchronous vs asynchronous

JavaScript reads your code top-to-bottom. Most code is **synchronous** — each line finishes before the next begins.

```js
console.log("1");
console.log("2");
console.log("3");
```

**Output:**
```
1
2
3
```

But some things take time: requesting a file over the network, reading a database, waiting for a timer. We cannot freeze the whole program for them. Those operations are **asynchronous** — they start now, finish later, and call us back when they are done.

A simple async demo using `setTimeout`:

```js
console.log("Before");

setTimeout(() => {
    console.log("Inside timeout (1 second later)");
}, 1000);

console.log("After");
```

**Output:**
```
Before
After
Inside timeout (1 second later)
```

The `setTimeout` was scheduled, but JS did not wait — it carried on, printed "After", and only later did the callback fire. This is **asynchronous** behaviour.

---

## 25.2 What is a Promise?

A **Promise** is a placeholder for a value that does not exist yet.

Think of it like ordering food at a restaurant. The waiter brings a **receipt** immediately — that is the promise. The actual food (the result) arrives later. The promise can end in three states:

| State | Meaning |
|---|---|
| `pending` | Cooking — not done yet. |
| `fulfilled` (resolved) | The food arrived. Here is your result. |
| `rejected` | Something went wrong. Here is the error. |

A simple promise we create ourselves:

```js
const food = new Promise((resolve, reject) => {
    setTimeout(() => {
        const lucky = Math.random() > 0.2; // 80% success
        if (lucky) {
            resolve("Veg momo!");
        } else {
            reject(new Error("Out of stock"));
        }
    }, 1000);
});

food
    .then(value => console.log("Got:", value))
    .catch(err => console.log("Failed:", err.message))
    .finally(() => console.log("Done."));
```

**Likely output (success case):**
```
Got: Veg momo!
Done.
```

**Or (failure case):**
```
Failed: Out of stock
Done.
```

- `.then(fn)` runs `fn` when the promise resolves.
- `.catch(fn)` runs `fn` if it rejects.
- `.finally(fn)` runs `fn` either way (clean-up).

You will rarely write `new Promise` yourself — most APIs already return promises. But understanding the model helps.

### Chaining `.then`

Each `.then` returns a new promise, so they chain:

```js
fetchUser()
    .then(user => fetchPosts(user.id))
    .then(posts => console.log(posts))
    .catch(err => console.error(err));
```

Chains were the standard for years. Modern code prefers a much nicer style: `async`/`await`.

---

## 25.3 `async` and `await`

`async`/`await` is **syntax sugar** over promises. Same machinery, much easier to read.

Rules:
1. Mark a function as `async` to use `await` inside it.
2. `await` pauses the function until the promise resolves, then gives you the value.
3. Errors are caught with normal `try`/`catch`.

The promise chain rewritten with `async`/`await`:

```js
async function loadFeed() {
    try {
        const user = await fetchUser();
        const posts = await fetchPosts(user.id);
        console.log(posts);
    } catch (err) {
        console.error(err);
    }
}

loadFeed();
```

It reads **like synchronous code**. That is the whole point.

> A common confusion: `await` only pauses **the function it is in** — the rest of the program keeps running. JavaScript is still single-threaded.

### A small full example

```js
const wait = (ms) => new Promise(resolve => setTimeout(resolve, ms));

async function run() {
    console.log("Step 1");
    await wait(500);
    console.log("Step 2 (half a second later)");
    await wait(500);
    console.log("Step 3 (one second after step 1)");
}

run();
console.log("This line runs before steps 2 and 3");
```

**Output:**
```
Step 1
This line runs before steps 2 and 3
Step 2 (half a second later)
Step 3 (one second after step 1)
```

---

## 25.4 The `fetch` API

`fetch` is the built-in browser function for making HTTP requests. It returns a promise that resolves to a `Response` object.

```js
fetch("https://jsonplaceholder.typicode.com/posts/1")
    .then(response => response.json())
    .then(data => console.log(data));
```

Or, with `async`/`await`:

```js
const response = await fetch("https://jsonplaceholder.typicode.com/posts/1");
const data = await response.json();
console.log(data);
```

**Expected output:**
```
{
  userId: 1,
  id: 1,
  title: "sunt aut facere repellat provident occaecati...",
  body: "quia et suscipit\nsuscipit recusandae..."
}
```

> The URL we are hitting — `jsonplaceholder.typicode.com` — is a free fake-data API. It pretends to be a blog with 100 posts and 10 users. Perfect for learning. We will use it for the rest of the course.

### Why two awaits?

- The first `await fetch(...)` waits for the **server to respond**.
- The second `await response.json()` waits for the **body to be parsed** as JSON.

`response.json()` itself returns a promise, so it needs awaiting too.

---

## 25.5 Handling HTTP errors

This is the part beginners always miss. `fetch` only rejects (throws) on **network failure** — if your wifi is off, for example. If the server returns a 404 or 500, `fetch` **still resolves**. You have to check `response.ok` (or `response.status`) yourself.

```js
async function loadPost(id) {
    try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${id}`);

        if (!response.ok) {
            throw new Error(`HTTP ${response.status} — ${response.statusText}`);
        }

        const data = await response.json();
        console.log(data);
    } catch (err) {
        console.error("Failed to load:", err.message);
    }
}

loadPost(1);    // works
loadPost(9999); // 404 — caught by our throw
```

`response.ok` is `true` for status codes 200–299. Any other status (404, 500, …) means we should treat it as an error.

---

## 25.6 The HTML for our demo

We are about to build a small page that fetches 10 posts and renders them.

**index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Posts</title>
    <style>
        body { font-family: sans-serif; padding: 2rem; max-width: 720px; }
        article { border: 1px solid #ddd; border-radius: 8px; padding: 1rem; margin-bottom: 1rem; }
        article h2 { margin: 0 0 0.5rem; font-size: 1.1rem; text-transform: capitalize; }
        article p { margin: 0; color: #444; }
        .status { color: #666; }
        .error { color: #b00020; }
    </style>
    <script src="app.js" defer></script>
</head>
<body>
    <h1>Latest posts</h1>
    <p id="status" class="status">Loading...</p>
    <main id="posts"></main>
</body>
</html>
```

---

## 25.7 The complete fetching app

`app.js`:

```js
const statusEl = document.querySelector("#status");
const postsEl = document.querySelector("#posts");

const renderPosts = (posts) => {
    postsEl.innerHTML = "";
    for (const post of posts) {
        const article = document.createElement("article");
        article.innerHTML = `
            <h2>${post.title}</h2>
            <p>${post.body}</p>
        `;
        postsEl.appendChild(article);
    }
};

const showError = (message) => {
    statusEl.textContent = message;
    statusEl.classList.add("error");
};

async function loadPosts() {
    try {
        const response = await fetch(
            "https://jsonplaceholder.typicode.com/posts?_limit=10"
        );

        if (!response.ok) {
            throw new Error(`Server replied with ${response.status}`);
        }

        const posts = await response.json();
        statusEl.textContent = `Loaded ${posts.length} posts.`;
        renderPosts(posts);
    } catch (err) {
        showError("Sorry — could not load posts. " + err.message);
    }
}

loadPosts();
```

Open `index.html` with Live Server. You should see:
1. "Loading..." for a fraction of a second.
2. "Loaded 10 posts." followed by ten styled cards.

If you turn off your wifi and reload, you should see the friendly error message instead of a crash. That is what `try`/`catch` buys us.

> Tip: `?_limit=10` is a JSONPlaceholder feature — it returns only the first 10. JSONPlaceholder also supports `/posts/1` for one post, `/posts?userId=1` for one user's posts, and so on.

---

## 25.8 Loading state — a better user experience

In a real app, while data is loading, we want to show a spinner or skeleton. While data is missing, we want a friendly empty state. The pattern:

```js
async function loadPosts() {
    // 1. Loading
    statusEl.textContent = "Loading...";
    statusEl.classList.remove("error");
    postsEl.innerHTML = "";

    try {
        const response = await fetch("https://jsonplaceholder.typicode.com/posts?_limit=10");
        if (!response.ok) throw new Error(`HTTP ${response.status}`);

        const posts = await response.json();

        // 2. Empty state
        if (posts.length === 0) {
            statusEl.textContent = "No posts to show.";
            return;
        }

        // 3. Success
        statusEl.textContent = `Loaded ${posts.length} posts.`;
        renderPosts(posts);
    } catch (err) {
        // 4. Error
        statusEl.textContent = "Could not load posts. " + err.message;
        statusEl.classList.add("error");
    }
}
```

These four states — **loading / empty / error / success** — are the same four states React Query will give us "for free" later in the course. The mental model is set today.

---

## 25.9 Sending data — POST, PUT, DELETE

Although we focus on GET (reading) today, `fetch` can also send data. JSONPlaceholder accepts fake writes — they look real but nothing is saved.

```js
async function createPost() {
    const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
            title: "My first post",
            body: "Hello, world",
            userId: 1
        })
    });

    const created = await response.json();
    console.log(created); // { id: 101, ... }
}

createPost();
```

We will use this style in Class 46 with Axios + React Query. For now, just know it exists.

---

## 25.10 Where this leads

Everything you wrote today maps almost one-to-one onto the React phase:

| Today | Later in React |
|---|---|
| `loadPosts()` runs on page load | `useEffect(() => { loadPosts() }, [])` |
| `statusEl.textContent = "Loading..."` | `if (isLoading) return <Spinner />` |
| `posts.length === 0` empty state | "Empty state" UI in Class 47 |
| `try / catch` error | React Query's `error` state |
| Manual `fetch` + state | `useQuery` (Class 45) hides all of it |

You have just hand-built what React Query will eventually give you in three lines. That is exactly the right way round.

---

## Practice Exercises

1. **Fetch one user**: Use JSONPlaceholder's `/users/1` endpoint to fetch a single user. Log their `name`, `email` and `address.city`. Use optional chaining (`?.`) for safety.

2. **Render users**: Build a page that shows a list of all 10 users from `/users`. Each card should show name, email, company name. Handle loading and error states.

3. **Posts by user**: Use `/posts?userId=1` to fetch only one user's posts. Render them in a list. Then add a button that, when clicked, fetches `?userId=2` instead.

4. **Error simulation**: Change your URL to an obviously bad one (`/postzzz`) and confirm your error message shows. Now turn your wifi off and reload — confirm the same message shows. This is why we wrap fetch calls in `try`/`catch`.

---

## Key Takeaways
- Async code lets your program do other things while waiting for slow operations.
- A **Promise** is a placeholder for a future value: `pending`, `fulfilled`, or `rejected`.
- `async`/`await` is the modern, readable way to use promises. Wrap awaits in `try`/`catch`.
- `fetch(url)` returns a response; **check `response.ok`** because `fetch` does not throw on 404/500.
- The loading / empty / error / success pattern you built today is the foundation of every React data screen.

That's the end of Phase 3 — you now know enough JavaScript to start writing React. In Class 26 we set up Node and npm so we can install React and run a build tool.
