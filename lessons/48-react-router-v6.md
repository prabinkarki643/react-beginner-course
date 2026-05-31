# Class 48: React Router v6

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 47 (Loading, error and empty states)

## What You Will Learn
- Why client-side routing matters and how it differs from a normal `<a>` link
- Setting up `<BrowserRouter>`, `<Routes>` and `<Route>` in a Vite + TypeScript project
- Navigating with `<Link>` and `<NavLink>` (with active-state styling)
- Dynamic URL parameters (`/posts/:id`) and reading them with `useParams<{ id: string }>()`
- Programmatic navigation with `useNavigate()`
- Nested routes and the `<Outlet />` layout pattern with a shadcn navbar
- A catch-all 404 route

---

## 48.1 What is Client-Side Routing?

In a classic HTML website, clicking a link causes the browser to throw away the current page and ask the server for a brand-new HTML document. That works, but it has two costs: a flash of white between pages and the loss of any state your JavaScript was holding.

A **single-page application** (SPA) does it differently. The browser loads `index.html` once. After that, JavaScript watches the URL bar — when it changes, JavaScript swaps which components are rendered, without contacting the server for a new page.

**React Router** is the standard library for this. It maps URLs to components.

| Without React Router | With React Router |
|----------------------|-------------------|
| `<a href="/about">` triggers a full reload | `<Link to="/about">` swaps components instantly |
| Back button reloads the page | Back button is instant |
| You manage "which page is shown" with `useState` | Router reads the URL for you |
| Cannot bookmark a specific view | Every view has its own URL |

> Analogy: HTML links are like leaving the building, going to reception, and being shown to a new room. React Router is like a smart receptionist standing inside the building — they just swap the door's contents in front of you.

---

## 48.2 Installing React Router

In your Vite project:

```bash
npm install react-router-dom
```

That single package gives you everything: the components, the hooks, and the TypeScript types.

---

## 48.3 Wrap the App in `<BrowserRouter>`

The very first step is to wrap the app so that every component below has access to the router context. Open `main.tsx` and add `<BrowserRouter>` just inside `<QueryClientProvider>` (you set that up in Class 45):

```tsx
// src/main.tsx
import React from "react";
import ReactDOM from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import { Toaster } from "@/components/ui/sonner";
import App from "./App";
import "./index.css";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60,
      gcTime: 1000 * 60 * 5,
      retry: 1,
      refetchOnWindowFocus: true,
    },
  },
});

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <BrowserRouter>
      <QueryClientProvider client={queryClient}>
        <App />
        <Toaster richColors position="top-right" />
        <ReactQueryDevtools initialIsOpen={false} />
      </QueryClientProvider>
    </BrowserRouter>
  </React.StrictMode>,
);
```

Order matters: `<BrowserRouter>` provides routing context to React Query (and to the toaster). It must be **outside** the app.

---

## 48.4 The Page Components

Routes need pages. Make a `src/pages/` folder and add a few component files. We are going to wire `/`, `/posts`, `/posts/:id`, and a 404.

```tsx
// src/pages/HomePage.tsx
function HomePage() {
  return (
    <div className="space-y-3">
      <h1 className="text-3xl font-bold">Welcome</h1>
      <p className="text-muted-foreground">
        This is a small posts dashboard powered by JSONPlaceholder. Use the
        navigation above to browse.
      </p>
    </div>
  );
}

export default HomePage;
```

```tsx
// src/pages/PostsPage.tsx
import PostsList from "@/components/PostsList";

function PostsPage() {
  return (
    <div className="space-y-4">
      <h1 className="text-2xl font-bold">All posts</h1>
      <PostsList />
    </div>
  );
}

export default PostsPage;
```

```tsx
// src/pages/PostDetailPage.tsx
import { useParams, useNavigate, Link } from "react-router-dom";
import { Button } from "@/components/ui/button";
import { usePost } from "@/hooks/usePosts";

function PostDetailPage() {
  // Read the :id segment of the URL
  const { id } = useParams<{ id: string }>();
  const navigate = useNavigate();
  const numericId = Number(id);

  const { data, isLoading, isError, error } = usePost(numericId);

  if (Number.isNaN(numericId)) {
    return <p className="text-destructive">Invalid post id.</p>;
  }

  if (isLoading) return <p>Loading post...</p>;
  if (isError) {
    return (
      <div className="space-y-2">
        <p className="text-destructive">{(error as Error).message}</p>
        <Link to="/posts" className="underline">Back to posts</Link>
      </div>
    );
  }
  if (!data) return null;

  return (
    <article className="space-y-4">
      <Button variant="outline" size="sm" onClick={() => navigate(-1)}>
        ← Back
      </Button>
      <h1 className="text-2xl font-bold">{data.title}</h1>
      <p className="whitespace-pre-line">{data.body}</p>
    </article>
  );
}

export default PostDetailPage;
```

```tsx
// src/pages/NotFoundPage.tsx
import { Link } from "react-router-dom";

function NotFoundPage() {
  return (
    <div className="text-center space-y-3 py-12">
      <p className="text-6xl font-bold text-muted-foreground">404</p>
      <p className="text-lg">We could not find that page.</p>
      <Link to="/" className="underline text-primary">Go home</Link>
    </div>
  );
}

export default NotFoundPage;
```

---

## 48.5 The Layout Component

Most apps have a layout that stays put while only the main content changes — usually a navbar at the top and a footer at the bottom. We build that as a `Layout` component using `<Outlet />`, which is the placeholder React Router renders the matched child route into.

First the navbar:

```tsx
// src/components/Navbar.tsx
import { NavLink } from "react-router-dom";
import { cn } from "@/lib/utils";

const links = [
  { to: "/", label: "Home", end: true },
  { to: "/posts", label: "Posts" },
];

function Navbar() {
  return (
    <header className="border-b">
      <div className="max-w-3xl mx-auto px-4 py-3 flex items-center justify-between">
        <NavLink to="/" className="font-bold text-lg">
          Posts Dashboard
        </NavLink>
        <nav className="flex gap-4">
          {links.map((link) => (
            <NavLink
              key={link.to}
              to={link.to}
              end={link.end}
              className={({ isActive }) =>
                cn(
                  "text-sm transition-colors",
                  isActive
                    ? "text-primary font-semibold"
                    : "text-muted-foreground hover:text-foreground",
                )
              }
            >
              {link.label}
            </NavLink>
          ))}
        </nav>
      </div>
    </header>
  );
}

export default Navbar;
```

Two things worth pointing out:

- `<NavLink>` is `<Link>` plus an `isActive` boolean. The `className` prop accepts a function that receives `{ isActive }` and returns a string — perfect for highlighting the current page.
- `end={true}` on `/` makes sure it is only active for the exact path `/`. Without `end`, `/` would be considered active on `/posts` too (because `/posts` also starts with `/`).

Now the layout itself:

```tsx
// src/components/Layout.tsx
import { Outlet } from "react-router-dom";
import Navbar from "@/components/Navbar";

function Layout() {
  return (
    <div className="min-h-screen flex flex-col">
      <Navbar />
      <main className="flex-1 max-w-3xl mx-auto px-4 py-6 w-full">
        <Outlet />
      </main>
      <footer className="border-t py-4 text-center text-xs text-muted-foreground">
        Built with React Router v6 and TanStack Query
      </footer>
    </div>
  );
}

export default Layout;
```

`<Outlet />` is where the matched child route's component appears. Think of it as a `{children}` slot that the router controls.

---

## 48.6 Defining the Routes

Now we glue everything together in `App.tsx`. The trick to the layout pattern: the layout route has **no path** but an `element`. All other routes nest inside it.

```tsx
// src/App.tsx
import { Routes, Route } from "react-router-dom";
import Layout from "@/components/Layout";
import HomePage from "@/pages/HomePage";
import PostsPage from "@/pages/PostsPage";
import PostDetailPage from "@/pages/PostDetailPage";
import NotFoundPage from "@/pages/NotFoundPage";

function App() {
  return (
    <Routes>
      <Route element={<Layout />}>
        <Route path="/" element={<HomePage />} />
        <Route path="/posts" element={<PostsPage />} />
        <Route path="/posts/:id" element={<PostDetailPage />} />
        <Route path="*" element={<NotFoundPage />} />
      </Route>
    </Routes>
  );
}

export default App;
```

That tiny file does a lot:

| URL | Renders |
|-----|---------|
| `/` | `Layout` → `HomePage` |
| `/posts` | `Layout` → `PostsPage` |
| `/posts/1` | `Layout` → `PostDetailPage` (id `"1"`) |
| `/posts/42` | `Layout` → `PostDetailPage` (id `"42"`) |
| `/anything-else` | `Layout` → `NotFoundPage` |

The `path="*"` is the **catch-all**. It matches any URL that the other routes did not match. Always include one.

---

## 48.7 Linking to Dynamic Routes

Inside `<PostsList>` (or wherever you render the list), each row should link to the detail page. Use `<Link>` with a template-literal `to`:

```tsx
// src/components/PostRow.tsx (relevant excerpt)
import { Link } from "react-router-dom";

<Link
  to={`/posts/${post.id}`}
  className="block hover:underline"
>
  <h3 className="font-medium">{post.title}</h3>
</Link>
```

`<Link>` renders an `<a>` under the hood, but its click handler tells React Router to handle the navigation — no full page reload.

> **Never** use a plain `<a href="/posts/1">` for internal links. Reserve `<a>` for **external** sites (`<a href="https://www.google.com">`).

---

## 48.8 `useParams` — Reading the URL

In `PostDetailPage` we already used `useParams<{ id: string }>()`. A quick breakdown:

```tsx
const { id } = useParams<{ id: string }>();
```

- The generic `<{ id: string }>` tells TypeScript what parameter names to expect.
- Every value from `useParams` is `string | undefined` (because the URL could in theory be missing the segment).
- If you need a number, convert it explicitly: `const numericId = Number(id)` — and guard against `NaN` just like we did.

```tsx
if (Number.isNaN(numericId)) {
  return <p className="text-destructive">Invalid post id.</p>;
}
```

---

## 48.9 `useNavigate` — Navigating from Code

Sometimes you need to navigate from a function — after a form submits, after a confirm dialog, in response to a `useEffect`. `<Link>` does not fit; you need a function. That is `useNavigate`:

```tsx
import { useNavigate } from "react-router-dom";

const navigate = useNavigate();

// Go to a specific page
navigate("/posts");

// Go back one entry in the browser history
navigate(-1);

// Replace the current entry (back button will skip this page)
navigate("/posts", { replace: true });
```

Real-world example — after creating a post, take the user to the list:

```tsx
const navigate = useNavigate();
const { mutate: createPost } = useCreatePost();

const onSubmit = (data: CreatePostFormValues) => {
  createPost(data, {
    onSuccess: () => navigate("/posts"),
  });
};
```

When you use `replace: true`, you remove the current page from the history stack — useful after login or registration, so the back button does not take the user back to the form they just submitted.

---

## 48.10 Try the Whole Thing

Run `npm run dev` and walk through the experience:

1. Land on `/` — see the home page inside the layout.
2. Click **Posts** — URL becomes `/posts`, list appears, **Posts** is now highlighted in the navbar.
3. Click any row — URL becomes `/posts/1` (or whatever id), detail appears.
4. Click **Back** (your own button) — back to `/posts` instantly.
5. Type `/somewhere-random` in the URL bar — 404 page.
6. Hit the browser back button — you return to wherever you came from. Forward button works too.

That whole experience used to require either a server, a hash-router hack, or a lot of manual state. React Router gives it to you in roughly 50 lines.

---

## 48.11 Mental Model of the Route Tree

The routes you defined form a tree. Beginners often picture it correctly the first time:

```
<Routes>
  <Route element={<Layout />}>            <-- always renders, no path
      <Route path="/"             />      <-- HomePage in <Outlet />
      <Route path="/posts"        />      <-- PostsPage in <Outlet />
      <Route path="/posts/:id"    />      <-- PostDetailPage in <Outlet />
      <Route path="*"             />      <-- NotFoundPage in <Outlet />
  </Route>
</Routes>
```

Router walks down the tree, picks the most specific match, and renders Layout with that match plugged into `<Outlet />`.

---

## Practice Exercises

### Exercise 1: Add a `/users` route
- Build a `<UsersPage>` that uses your `useUsers` hook
- Build a `<UserDetailPage>` at `/users/:id` using `useUser(id)`
- Make each user row a `<Link>` to its detail page
- Add **Users** to the navbar

### Exercise 2: Authenticated layout
- Create a second layout `<AuthLayout>` with no navbar (just a centred card)
- Add a fake `/login` route under this layout (no real auth — just a placeholder page)
- Verify that `/login` does **not** show the navbar, while `/` and `/posts` do

### Exercise 3: Programmatic redirect after create
- Wire `<CreatePostForm>`'s success callback to call `navigate("/posts")`
- Pass `{ replace: true }` so the user cannot press **Back** to return to the form

### Exercise 4: Highlight the navbar correctly
- Note that the `Home` link uses `end={true}`. Try removing it and refreshing on `/posts`
- Explain in one sentence (as a comment in your code) what `end` does

---

## Key Takeaways
- React Router maps URLs to components, with no full page reloads.
- `<BrowserRouter>` wraps the entire app in `main.tsx`.
- `<Routes>` contains your `<Route>`s; each `<Route>` connects a `path` to an `element`.
- Use `<Link>` for internal navigation. Use `<a>` only for external URLs.
- `<NavLink>` is `<Link>` with an `isActive` boolean — pass `className` a function for active-state styling.
- Dynamic segments use `:name` and you read them with `useParams<{ name: string }>()`. Convert to numbers explicitly.
- `useNavigate()` is for programmatic navigation; `navigate(-1)` is **back**, and `{ replace: true }` removes the current entry from history.
- The layout pattern uses a parent `<Route>` with no `path` and an `<Outlet />` slot where the child route renders.
- Always include a catch-all `<Route path="*" element={<NotFoundPage />} />`.
- **Next class**: a professional admin table — TanStack Table with pagination, search, filters and row actions.
