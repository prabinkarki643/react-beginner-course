# Class 35: useEffect & Side Effects

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 34 (controlled forms)

## What You Will Learn
- What a "side effect" is and why React separates it from rendering
- The `useEffect(fn, deps)` hook and its mental model
- The three shapes of the dependency array: `[]`, `[a, b]` and **omitted**
- Cleanup functions for timers, intervals and subscriptions
- Fetching data with `useEffect + fetch` against JSONPlaceholder
- Loading and error UI patterns
- Why React Query (coming in Class 45) is usually a better choice for data fetching

---

## 35.1 What is a Side Effect?

So far, every component we have written has been **pure** — given the same props and state, it returns the same JSX. Pure rendering is great because it is predictable and easy to test.

But real apps need to talk to the outside world: fetch data from an API, save to `localStorage`, set a timer, subscribe to a WebSocket, change the page title, log analytics. These are **side effects** — anything the component does beyond returning JSX.

React deliberately keeps rendering pure. Side effects go into a special hook: **`useEffect`**. React runs your effect *after* the render is on screen, which is a safe time to talk to the outside world.

> **Analogy**: rendering is "drawing the picture". Effects are "everything else you do once the picture is up" — turning on the lights, playing the music, ringing the doorbell.

---

## 35.2 The `useEffect` Hook

```tsx
import { useEffect } from 'react';

useEffect(() => {
  // do the side effect here
}, [dependencies]);
```

Two arguments:

1. A **function** that performs the side effect.
2. A **dependency array** that tells React when to re-run the function.

A tiny example — change the page title whenever a counter changes:

```tsx
import { useState, useEffect } from 'react';

function PageTitleDemo() {
  const [count, setCount] = useState<number>(0);

  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  return (
    <div style={{ padding: '1rem', fontFamily: 'system-ui' }}>
      <p>Look at the browser tab title.</p>
      <button onClick={() => setCount((c) => c + 1)}>
        Increment ({count})
      </button>
    </div>
  );
}

export default PageTitleDemo;
```

Open this in the browser and watch the tab title change every time you click. The effect runs whenever `count` changes.

---

## 35.3 The Dependency Array — Three Shapes

The dependency array is the single most important thing to get right. There are three patterns, each with very different behaviour.

### 1. `[]` — run **once**, on mount

```tsx
useEffect(() => {
  console.log('Component mounted');
}, []);
```

Empty array → the effect runs once, after the first render. It does not run again. Use this for "do this when the component first appears" — initial data fetches, setting up a subscription, etc.

> In React 18 with `<StrictMode>` (which Vite enables by default), effects run **twice** in development. This is intentional — it helps you spot effects that are missing cleanups. In production they run once.

### 2. `[a, b, c]` — run **on changes**

```tsx
useEffect(() => {
  console.log('userId changed to', userId);
}, [userId]);
```

The effect runs after the first render, and again every time any value in the array changes (compared with `===`). Use this when the effect depends on some piece of state or prop.

### 3. Omitted — runs after **every** render

```tsx
useEffect(() => {
  console.log('Render finished');
}); // no dependency array
```

The effect runs after every single render. This is almost always **wrong** and a sign of a bug. If you find yourself reaching for this, stop and reconsider — you probably want `[]` or `[someValue]`.

### Cheat sheet

| Array | Runs |
|-------|------|
| `[]` | Once, after the first render |
| `[a, b]` | After first render, then when `a` or `b` change |
| (omitted) | After every render — usually wrong |

---

## 35.4 The Cleanup Function

Some effects need to be **undone** before the next run, or when the component is removed from the screen. The cleanup function does that — return it from the effect.

```tsx
useEffect(() => {
  // set up
  return () => {
    // clean up
  };
}, [deps]);
```

### Example — an interval timer

Without cleanup, this is a memory leak:

```tsx
useEffect(() => {
  setInterval(() => console.log('tick'), 1000);
}, []);
```

Every time the component re-mounts, a new interval is added, but the old one keeps running. After a few mounts the console fills with ticks.

The right version:

```tsx
import { useEffect, useState } from 'react';

function Clock() {
  const [now, setNow] = useState<Date>(new Date());

  useEffect(() => {
    const id = setInterval(() => setNow(new Date()), 1000);
    return () => clearInterval(id);    // cleanup
  }, []);

  return <p>The time is {now.toLocaleTimeString('en-GB')}</p>;
}

export default Clock;
```

When the `Clock` component is removed, React calls the cleanup function, which clears the interval. No leak.

### When to use cleanup

- **Timers**: `setInterval`, `setTimeout` → call `clearInterval` / `clearTimeout`.
- **Event listeners**: `window.addEventListener` → call `window.removeEventListener`.
- **Subscriptions**: WebSocket, EventSource, browser APIs → close / unsubscribe.
- **Fetch with `AbortController`**: cancel the in-flight request if the component unmounts (covered next).

---

## 35.5 Fetching Data with `useEffect`

The classic use of `useEffect` is to fetch data when a component mounts. We will fetch a list of posts from **JSONPlaceholder**, a free fake API at `https://jsonplaceholder.typicode.com`.

### Step 1 — type the response

```tsx
// src/types/post.ts
export interface Post {
  userId: number;
  id: number;
  title: string;
  body: string;
}
```

### Step 2 — the component

```tsx
// src/PostList.tsx
import { useEffect, useState } from 'react';
import type { Post } from './types/post';

function PostList() {
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function load() {
      try {
        setLoading(true);
        setError(null);
        const res = await fetch(
          'https://jsonplaceholder.typicode.com/posts?_limit=10',
          { signal: controller.signal }
        );
        if (!res.ok) {
          throw new Error(`HTTP ${res.status}`);
        }
        const data: Post[] = await res.json();
        setPosts(data);
      } catch (err) {
        if ((err as Error).name === 'AbortError') return;   // ignore aborts
        setError((err as Error).message);
      } finally {
        setLoading(false);
      }
    }

    load();

    return () => controller.abort();    // cleanup: cancel the request
  }, []);

  if (loading) return <p>Loading posts…</p>;
  if (error)   return <p style={{ color: 'crimson' }}>Error: {error}</p>;
  if (posts.length === 0) return <p>No posts found.</p>;

  return (
    <ul style={{ listStyle: 'none', padding: 0, maxWidth: 600 }}>
      {posts.map((post) => (
        <li
          key={post.id}
          style={{
            padding: '0.75rem 1rem',
            border: '1px solid #ddd',
            borderRadius: '8px',
            marginBottom: '0.5rem',
          }}
        >
          <h3 style={{ marginTop: 0 }}>{post.title}</h3>
          <p style={{ margin: 0 }}>{post.body}</p>
        </li>
      ))}
    </ul>
  );
}

export default PostList;
```

Use it in `App.tsx`:

```tsx
import PostList from './PostList';

function App() {
  return (
    <div style={{ padding: '2rem', fontFamily: 'system-ui' }}>
      <h1>Latest posts</h1>
      <PostList />
    </div>
  );
}

export default App;
```

Reload the page. You should see "Loading posts…" briefly, then a list of ten posts from JSONPlaceholder.

### What that code does, step by step

1. We declare three pieces of state: `posts`, `loading`, `error`.
2. In the effect (runs once after mount), we create an `AbortController` so we can cancel the fetch if the component unmounts.
3. The async helper `load()` does the fetching. We turn loading on, fetch the URL, check for HTTP errors, parse JSON, set the posts, turn loading off.
4. If the fetch was aborted, the `AbortError` is silently ignored — the component is gone, no point updating state.
5. The cleanup function calls `controller.abort()`. If the component unmounts before the fetch finishes, the request is cancelled.
6. The render uses three early returns for the loading / error / empty states, then renders the list.

This is the classic "fetch on mount" pattern. **Memorise it.**

---

## 35.6 Fetching When a Prop Changes

If the URL depends on a prop (for example, a post id), put that value in the dependency array.

```tsx
// src/PostDetail.tsx
import { useEffect, useState } from 'react';
import type { Post } from './types/post';

interface PostDetailProps {
  postId: number;
}

function PostDetail({ postId }: PostDetailProps) {
  const [post, setPost] = useState<Post | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function load() {
      try {
        setLoading(true);
        setError(null);
        setPost(null);
        const res = await fetch(
          `https://jsonplaceholder.typicode.com/posts/${postId}`,
          { signal: controller.signal }
        );
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const data: Post = await res.json();
        setPost(data);
      } catch (err) {
        if ((err as Error).name === 'AbortError') return;
        setError((err as Error).message);
      } finally {
        setLoading(false);
      }
    }

    load();
    return () => controller.abort();
  }, [postId]);    // re-fetch whenever postId changes

  if (loading) return <p>Loading post #{postId}…</p>;
  if (error)   return <p style={{ color: 'crimson' }}>Error: {error}</p>;
  if (!post)   return null;

  return (
    <article style={{ maxWidth: 600, padding: '1rem' }}>
      <h2>{post.title}</h2>
      <p>{post.body}</p>
    </article>
  );
}

export default PostDetail;
```

Use it like:

```tsx
import { useState } from 'react';
import PostDetail from './PostDetail';

function App() {
  const [id, setId] = useState<number>(1);

  return (
    <div style={{ padding: '2rem', fontFamily: 'system-ui' }}>
      <button onClick={() => setId((n) => Math.max(1, n - 1))}>Previous</button>
      <button onClick={() => setId((n) => n + 1)} style={{ marginLeft: '0.5rem' }}>
        Next
      </button>
      <PostDetail postId={id} />
    </div>
  );
}

export default App;
```

Click Next/Previous. The component re-fetches every time `id` changes — because `id` is in the dependency array.

---

## 35.7 Common `useEffect` Mistakes

### Forgetting the dependency array

```tsx
useEffect(() => {
  fetch('/api/data').then(...);
}); // no array → fires every render → infinite loop if you setState inside
```

Always provide an array. Even `[]` is fine if the effect truly has no dependencies.

### Lying about dependencies

If you use a value inside the effect, it must be in the array — otherwise the effect uses a stale value.

```tsx
useEffect(() => {
  console.log(userId);     // uses userId
}, []);                    // but it is not in the array — bug
```

### Putting an object or array literal in deps

```tsx
useEffect(() => { ... }, [{ a: 1 }]);  // brand-new object every render → fires every render
```

Put primitives (strings, numbers, booleans) in the array, or memoise the object (a topic for later).

### Doing async directly in `useEffect`

```tsx
useEffect(async () => { ... }, []); // WRONG — effects must return undefined or a cleanup
```

Define an inner `async` function and call it, as we did above.

---

## 35.8 Why We Will Move to React Query Later

The `useEffect + fetch` pattern works, but it has rough edges:

- You write the same `loading / error / data` boilerplate in every component.
- Two components asking for the same data fetch it twice.
- You have to write your own caching, retry, refetch-on-window-focus and so on.

In **Class 45** we introduce **TanStack Query (React Query)**, a small library that handles all of this for you:

```tsx
const { data, isLoading, error } = useQuery({
  queryKey: ['posts'],
  queryFn: () => fetch('https://jsonplaceholder.typicode.com/posts').then((r) => r.json()),
});
```

Same idea, a fraction of the code, with caching and deduplication built in. We are doing it by hand now so that when you meet React Query, you appreciate the problems it solves.

---

## Practice Exercises

1. **Live clock.** Build a `<DigitalClock />` that displays the current time updated every second. Use a `setInterval` inside `useEffect` and return a cleanup function. Format with `toLocaleTimeString('en-GB')`.
2. **Document title from state.** Build a `<Counter />` whose document title changes to "Clicked X times". The effect should depend on the count.
3. **Users list from JSONPlaceholder.** Fetch `https://jsonplaceholder.typicode.com/users` once on mount. Show a "Loading…" message, then a list of names and emails. Handle the error case (try changing the URL to a wrong one to trigger the error UI).
4. **Switchable detail view.** Use the `<PostDetail postId={id} />` pattern from section 35.6. Add a `<input type="number">` so the user can type a post id between 1 and 100. The detail view should re-fetch whenever the id changes. Show a loading message while fetching.

---

## Key Takeaways
- A **side effect** is anything a component does beyond returning JSX — fetching, timers, subscriptions, DOM writes.
- `useEffect(fn, deps)` runs `fn` after the render. Its **dependency array** controls when it re-runs.
- `[]` = run once. `[a, b]` = run when these change. Omitted = every render (usually wrong).
- Return a **cleanup function** for timers, intervals, listeners and abortable fetches.
- The canonical fetch pattern: state for `data / loading / error`, an `AbortController` for cleanup, three early returns in the render.
- React Query (Class 45) will replace most hand-rolled `useEffect + fetch` code with something shorter and smarter.
- Next class: when two siblings need the same data — **lifting state up**.
