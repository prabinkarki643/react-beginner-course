# Class 45: Axios and React Query — Fetching Data the Modern Way

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 44 (Zod + RHF), Class 25 (Promises and Fetch), Class 35 (`useEffect`)

## What You Will Learn
- Why Axios is preferred over the built-in Fetch API for real applications
- Creating a configured Axios instance with a `baseURL`
- What React Query (TanStack Query) actually is — server state vs client state
- Setting up `QueryClient`, `QueryClientProvider` and the React Query DevTools
- Using `useQuery({ queryKey, queryFn })` to read data
- Sensible defaults for `staleTime` and `gcTime`
- Building reusable `usePosts()` and `usePost(id)` hooks

---

## 45.1 The Problem We Want to Solve

In Class 35 you fetched posts inside `useEffect` and managed `loading` / `error` / `data` manually. That works, but every fetch needs ~15 lines of repetitive code. And you have no caching, no automatic refetching, no way to share the same data between components without lifting state or using Context.

In this class we replace that pattern with two industry-standard tools:

- **Axios** — a small HTTP-client library that fixes the rough edges of Fetch.
- **React Query** (officially **TanStack Query**) — a library that manages all server-state concerns for you: loading, errors, caching, refetching, and synchronisation.

> Analogy: doing manual fetches is like cooking every meal from scratch including grinding the flour. Axios is the pre-milled flour. React Query is the kitchen appliance that handles timing, temperature and serving automatically.

---

## 45.2 Why Axios Over Fetch?

Fetch is built into every browser, which is great. But for application code it has three issues that you end up working around constantly:

| Concern | Fetch | Axios |
|---------|-------|-------|
| JSON parsing | `await response.json()` every time | Automatic |
| HTTP errors (4xx / 5xx) | Does **not** throw — you must check `response.ok` yourself | Throws automatically |
| Base URL | Must repeat the full URL on every call | Configure once on the instance |
| Headers | Repeat on every request | Configure once on the instance |
| Request timeout | Build it yourself with `AbortController` | One config option |
| TypeScript responses | Cast manually | Generic types: `api.get<Post[]>(...)` |

In short: Axios lets you write less code, and the code you do write is more honest about errors.

---

## 45.3 Installing Axios and React Query

```bash
npm install axios @tanstack/react-query @tanstack/react-query-devtools
```

- `axios` — the HTTP client.
- `@tanstack/react-query` — the server-state manager.
- `@tanstack/react-query-devtools` — a development panel that visualises all your queries (extremely useful for learning and debugging).

---

## 45.4 Creating an Axios Instance

Rather than `import axios from "axios"` every time, we create **one** configured instance and import it across the app. This is the single best habit you can form when working with HTTP.

```ts
// src/lib/api.ts
import axios from "axios";

const api = axios.create({
  baseURL: "https://jsonplaceholder.typicode.com",
  headers: {
    "Content-Type": "application/json",
  },
  timeout: 10_000, // 10 seconds
});

export default api;
```

What this gives you:

- `baseURL` — every request prepends `https://jsonplaceholder.typicode.com`, so you write `/posts` instead of the full URL.
- `headers` — set once, applied everywhere.
- `timeout` — requests fail after 10 seconds instead of hanging forever.

> JSONPlaceholder is a free public placeholder API. You can read posts, users, comments, etc. without signing up. We use it throughout the course because it is reliable and the response shapes are stable.

---

## 45.5 Define the `Post` Type

Always type your API responses. It costs nothing and gives you autocomplete everywhere.

```ts
// src/types/post.ts
export interface Post {
  id: number;
  userId: number;
  title: string;
  body: string;
}
```

You can verify the real shape by visiting `https://jsonplaceholder.typicode.com/posts/1` in your browser.

---

## 45.6 What is React Query?

Most state in your app is one of two kinds:

| Kind | Owned by | Examples |
|------|----------|----------|
| **Client state** | The browser, only this user | A dark-mode toggle, a modal's open/closed flag, a filter selection |
| **Server state** | The server, shared by everyone | The list of posts, the current user's profile, a single post by id |

`useState` and Context are perfect for **client** state. They are *not* great at **server** state because server state is asynchronous, can become stale, and often needs to be shared between many components.

React Query is built specifically to manage server state. It answers questions like:

- Is the data still loading?
- Did the request fail?
- Is this data fresh, or should we refetch it?
- How do we share the same data across pages without prop-drilling or duplicating fetches?

You get all of that for free by writing **two lines** instead of fifteen.

---

## 45.7 Setting Up the `QueryClient`

The `QueryClient` is the central cache that holds every query result. You create it once and provide it to your whole app, just like you would with a Context provider.

```tsx
// src/main.tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import App from "./App";
import "./index.css";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60,           // 1 minute — data is considered "fresh" for this long
      gcTime: 1000 * 60 * 5,           // 5 minutes — unused data stays in the cache this long
      retry: 1,                        // retry a failed request once
      refetchOnWindowFocus: true,      // refetch when the user returns to the tab
    },
  },
});

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  </React.StrictMode>,
);
```

The two important defaults:

- `staleTime` — how long a query result is considered fresh. While fresh, React Query will not refetch — it just serves the cached data. We use 1 minute as a sensible balance.
- `gcTime` — short for **garbage collection** time. How long a result stays in the cache after no component is using it. After this, it is dropped from memory.

`<ReactQueryDevtools />` adds a floating panel (only in development) where you can see every query, its current state, and force refetches. Use it constantly while learning.

---

## 45.8 Your First `useQuery`

`useQuery` is the hook for **reading** data. It takes two required things:

- `queryKey` — a unique identifier for this piece of data, written as an array.
- `queryFn` — an async function that returns the data.

```tsx
// src/components/PostsList.tsx
import { useQuery } from "@tanstack/react-query";
import api from "@/lib/api";
import type { Post } from "@/types/post";

function PostsList() {
  const { data, isLoading, isError, error } = useQuery({
    queryKey: ["posts"],
    queryFn: async (): Promise<Post[]> => {
      const response = await api.get<Post[]>("/posts");
      return response.data;
    },
  });

  if (isLoading) return <p>Loading posts...</p>;
  if (isError) return <p>Failed to load: {(error as Error).message}</p>;

  return (
    <ul className="space-y-2">
      {data?.map((post) => (
        <li key={post.id} className="border p-3 rounded">
          <h3 className="font-medium">{post.title}</h3>
          <p className="text-sm text-muted-foreground line-clamp-2">{post.body}</p>
        </li>
      ))}
    </ul>
  );
}

export default PostsList;
```

Read it slowly:

1. `useQuery` returns an object with `data`, `isLoading`, `isError`, `error`, and lots more.
2. `queryKey: ["posts"]` is the cache key. Every component that calls `useQuery({ queryKey: ["posts"], ... })` shares the same cached data.
3. `queryFn` is just an async function. Inside it, you can call Axios, Fetch — anything.
4. `data` is typed as `Post[] | undefined`. It is `undefined` while loading.

Open the React Query DevTools panel. You will see a `["posts"]` query, its **fresh / stale / fetching** badge, and the cached response. Click the **refetch** button to force a new request.

---

## 45.9 Fetching One Item: `usePost(id)`

To fetch a single post, the `queryKey` should include the id. That way each post has its own cache entry.

```tsx
import { useQuery } from "@tanstack/react-query";
import api from "@/lib/api";
import type { Post } from "@/types/post";

function PostDetail({ id }: { id: number }) {
  const { data, isLoading, isError, error } = useQuery({
    queryKey: ["posts", id],
    queryFn: async (): Promise<Post> => {
      const response = await api.get<Post>(`/posts/${id}`);
      return response.data;
    },
    enabled: id > 0, // skip the fetch if the id is missing
  });

  if (isLoading) return <p>Loading post...</p>;
  if (isError) return <p>{(error as Error).message}</p>;
  if (!data) return null;

  return (
    <article className="border p-4 rounded space-y-2">
      <h2 className="text-xl font-semibold">{data.title}</h2>
      <p>{data.body}</p>
    </article>
  );
}

export default PostDetail;
```

Two new things:

- `queryKey: ["posts", id]` — the cache key is now per-id. `["posts", 1]` and `["posts", 2]` are independent entries.
- `enabled: id > 0` — React Query will not run the query while this is `false`. Useful when the id is not ready yet.

---

## 45.10 Extracting Reusable Hooks

In any real app, you put your queries inside custom hooks. This keeps components clean and makes the query reusable from any page.

```ts
// src/hooks/usePosts.ts
import { useQuery } from "@tanstack/react-query";
import api from "@/lib/api";
import type { Post } from "@/types/post";

export function usePosts() {
  return useQuery({
    queryKey: ["posts"],
    queryFn: async (): Promise<Post[]> => {
      const { data } = await api.get<Post[]>("/posts");
      return data;
    },
  });
}

export function usePost(id: number) {
  return useQuery({
    queryKey: ["posts", id],
    queryFn: async (): Promise<Post> => {
      const { data } = await api.get<Post>(`/posts/${id}`);
      return data;
    },
    enabled: id > 0,
  });
}
```

Now your components shrink to a couple of lines:

```tsx
// src/components/PostsList.tsx
import { usePosts } from "@/hooks/usePosts";

function PostsList({ onSelect }: { onSelect: (id: number) => void }) {
  const { data, isLoading, isError } = usePosts();

  if (isLoading) return <p>Loading...</p>;
  if (isError) return <p>Failed to load posts.</p>;

  return (
    <ul className="space-y-2">
      {data?.slice(0, 10).map((post) => (
        <li key={post.id} className="border p-3 rounded">
          <button
            onClick={() => onSelect(post.id)}
            className="text-left w-full"
          >
            <h3 className="font-medium">{post.title}</h3>
          </button>
        </li>
      ))}
    </ul>
  );
}

export default PostsList;
```

---

## 45.11 Putting It All Together

A small page that lists posts and lets the user view a single post:

```tsx
// src/App.tsx
import { useState } from "react";
import { Button } from "@/components/ui/button";
import PostsList from "@/components/PostsList";
import PostDetail from "@/components/PostDetail";

function App() {
  const [selectedId, setSelectedId] = useState<number | null>(null);

  return (
    <div className="min-h-screen p-6 max-w-2xl mx-auto space-y-6">
      <header className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">Posts</h1>
        {selectedId !== null && (
          <Button variant="outline" onClick={() => setSelectedId(null)}>
            Back to list
          </Button>
        )}
      </header>

      {selectedId === null ? (
        <PostsList onSelect={setSelectedId} />
      ) : (
        <PostDetail id={selectedId} />
      )}
    </div>
  );
}

export default App;
```

(We swap this for real routes in Class 48.)

```tsx
// src/components/PostDetail.tsx
import { usePost } from "@/hooks/usePosts";

function PostDetail({ id }: { id: number }) {
  const { data, isLoading, isError, error } = usePost(id);

  if (isLoading) return <p>Loading post...</p>;
  if (isError) return <p className="text-destructive">{(error as Error).message}</p>;
  if (!data) return null;

  return (
    <article className="border p-4 rounded space-y-2">
      <h2 className="text-xl font-semibold">{data.title}</h2>
      <p className="whitespace-pre-line">{data.body}</p>
    </article>
  );
}

export default PostDetail;
```

Run the dev server and try it:

- Click a post — `usePost(id)` fires and the detail appears.
- Click **Back to list** — the list is still there, instant. React Query cached it.
- Click the same post again — the detail appears **instantly** from the cache. No spinner.
- Open the DevTools query panel — you can see both `["posts"]` and `["posts", 1]` sitting in the cache.

That instant second click is the moment most students "get it" — React Query is silently doing all the work you used to do by hand.

---

## 45.12 Helpful `useQuery` Return Values

You will use these constantly:

| Value | Meaning |
|-------|---------|
| `data` | The cached result. `undefined` while there is no data yet. |
| `isLoading` | `true` on the very first fetch when there is no cached data. |
| `isFetching` | `true` every time a request is in flight — including background refetches. |
| `isError` | `true` if the query failed. |
| `error` | The thrown error object (use `(error as Error).message`). |
| `isSuccess` | `true` once data is loaded successfully. |
| `refetch()` | Call to manually trigger a refetch (we use this in Class 47). |

A common pitfall: people use `isLoading` to drive a skeleton, but `isLoading` is `false` during background refetches. If you want the spinner to show every time there is a network request in flight, use `isFetching`.

---

## Practice Exercises

### Exercise 1: A `useUsers` hook
- Define a `User` interface matching `https://jsonplaceholder.typicode.com/users` (fields: `id`, `name`, `username`, `email`)
- Create a `useUsers()` hook in `src/hooks/useUsers.ts`
- Build a `<UsersList>` component that renders each user's name and email inside a `<Card>` (shadcn)
- Verify in the DevTools panel that the query is cached

### Exercise 2: A `useUser(id)` hook
- Add `useUser(id: number)` to the same file
- Build a `<UserDetail id={id} />` component that shows name, email, phone and website
- Toggle between list and detail using a `selectedUserId` piece of state (just like the posts example)

### Exercise 3: Tune the cache
- Set `staleTime: 0` on `useUsers` (just for this query — pass it as an option to `useQuery`)
- Open the DevTools panel. Switch away from the tab and back — see the query refetch every time
- Now set it to `1000 * 60 * 10` (10 minutes) — see the query stay fresh and not refetch
- In your own words, write one sentence under your code explaining what `staleTime` controls

---

## Key Takeaways
- Axios is a small HTTP-client library that fixes Fetch's rough edges: automatic JSON, automatic errors on 4xx/5xx, a configured `baseURL`, and typed responses.
- Create **one** Axios instance in `src/lib/api.ts` and import it everywhere.
- React Query manages **server state** — loading, errors, caching, sharing data across components.
- `<QueryClientProvider>` wraps the whole app; mount `<ReactQueryDevtools />` next to it while learning.
- `useQuery({ queryKey, queryFn })` reads data. The `queryKey` is also the cache key.
- Use array keys: `["posts"]` for a list, `["posts", id]` for a single item.
- Put queries inside `useThing()` custom hooks for reuse.
- `staleTime` decides how long data is fresh; `gcTime` decides how long unused data lingers in memory.
- **Next class**: changing data — `useMutation`, query keys factories, and cache invalidation.
