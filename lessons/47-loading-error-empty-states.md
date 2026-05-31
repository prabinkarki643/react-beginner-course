# Class 47: Loading, Error and Empty States

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 46 (Mutations and cache invalidation)

## What You Will Learn
- The five UI states every data screen must handle: idle, loading, error, empty, data
- Skeleton loaders with the shadcn `<Skeleton>` component
- Friendly error screens with a **Retry** button (`refetch()` and `refetchOnWindowFocus`)
- Empty states with a clear call-to-action
- Sonner toasts on `onSuccess` / `onError` in mutations
- Disabling buttons during pending mutations to stop double-clicks

---

## 47.1 The Five States of Every Screen

Whenever you display data fetched from a server, your screen has five possible states. Beginners often only think about two ("loading" and "data") and ship a broken-feeling experience because of it.

| State | Meaning | UI |
|-------|---------|----|
| **Idle** | No request has been made yet. Rare with React Query but possible. | A blank placeholder or call-to-action |
| **Loading** | First request is in flight, no cached data. | Skeleton rows / spinner |
| **Error** | The request failed. | Friendly message + **Retry** button |
| **Empty** | The request succeeded but the list is empty. | "No results yet" with a CTA |
| **Data** | The happy path — render the items. | Your normal UI |

> Analogy: a restaurant menu has five matching modes — closed, kitchen preparing, kitchen on strike, sold out, ready to serve. Skipping any of them creates a bad customer experience.

Today we polish the CRUD page from Class 46 so it handles all five gracefully.

---

## 47.2 Skeleton Loaders

A skeleton is a grey placeholder shape that hints at the layout. It gives the user a sense of progress and stops the page from "jumping" when the real data lands.

Add the shadcn component:

```bash
npx shadcn@latest add skeleton
```

Skeletons are just `<div>`s with a pulsing animation. A reusable list-skeleton component:

```tsx
// src/components/PostsListSkeleton.tsx
import { Skeleton } from "@/components/ui/skeleton";

function PostsListSkeleton({ rows = 5 }: { rows?: number }) {
  return (
    <ul className="space-y-2" aria-label="Loading posts">
      {Array.from({ length: rows }).map((_, i) => (
        <li key={i} className="border p-3 rounded space-y-2">
          <Skeleton className="h-4 w-3/4" />
          <Skeleton className="h-3 w-full" />
          <Skeleton className="h-3 w-5/6" />
        </li>
      ))}
    </ul>
  );
}

export default PostsListSkeleton;
```

Key points:

- Match the **shape** of the real row — same padding, same border, same vertical rhythm. The transition from skeleton to data should be seamless.
- Use multiple lines (`h-4`, `h-3`, varied widths) — a single grey block looks lazy.
- The `aria-label` tells assistive technology what is happening.

---

## 47.3 Error State with Retry

When a request fails, the user needs three things:

1. A **clear message** ("Failed to load posts").
2. A **why** when available (the error message from the server).
3. A **way out** — usually a **Retry** button.

React Query gives us `refetch()` directly from `useQuery`:

```tsx
// src/components/PostsListError.tsx
import { Button } from "@/components/ui/button";

interface PostsListErrorProps {
  message: string;
  onRetry: () => void;
}

function PostsListError({ message, onRetry }: PostsListErrorProps) {
  return (
    <div
      role="alert"
      className="border border-destructive/30 bg-destructive/5 rounded p-4 space-y-2"
    >
      <p className="text-destructive font-medium">Failed to load posts</p>
      <p className="text-sm text-muted-foreground">{message}</p>
      <Button variant="outline" size="sm" onClick={onRetry}>
        Try again
      </Button>
    </div>
  );
}

export default PostsListError;
```

### Configuring retry behaviour

React Query also retries failed requests automatically. You can tune this per-query or globally on the `QueryClient`.

```ts
// src/main.tsx (excerpt)
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60,
      gcTime: 1000 * 60 * 5,
      retry: 2,                      // try the request up to 2 more times
      retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 30_000),
      refetchOnWindowFocus: true,    // refetch when the tab regains focus
    },
  },
});
```

- `retry: 2` — after the first failure, React Query tries twice more before reporting an error. Set this to `false` if you want errors to surface immediately during development.
- `retryDelay` — wait longer between retries (exponential backoff is a common pattern).
- `refetchOnWindowFocus` — when the user switches back to your tab, fetch any stale data so they see the latest.

You can override these per-query too:

```ts
useQuery({
  queryKey: postKeys.list({}),
  queryFn: () => postApi.getAll(),
  retry: 1,
  refetchOnWindowFocus: false, // off for this one query
});
```

---

## 47.4 Empty State

An empty list and a loading list look almost identical to the user — both are blank. Make the empty state explicit and helpful:

```tsx
// src/components/PostsListEmpty.tsx
import { Button } from "@/components/ui/button";

interface PostsListEmptyProps {
  onCreate: () => void;
}

function PostsListEmpty({ onCreate }: PostsListEmptyProps) {
  return (
    <div className="border border-dashed rounded p-8 text-center space-y-3">
      <p className="text-base font-medium">No posts yet</p>
      <p className="text-sm text-muted-foreground">
        Get started by creating your very first post.
      </p>
      <Button onClick={onCreate}>Create a post</Button>
    </div>
  );
}

export default PostsListEmpty;
```

The pattern: **headline** → **explanation** → **call to action**. Never leave the user stuck.

---

## 47.5 Putting It All Together in `<PostsList>`

```tsx
// src/components/PostsList.tsx
import { useRef } from "react";
import { usePosts } from "@/hooks/usePosts";
import PostRow from "@/components/PostRow";
import PostsListSkeleton from "@/components/PostsListSkeleton";
import PostsListError from "@/components/PostsListError";
import PostsListEmpty from "@/components/PostsListEmpty";

function PostsList() {
  const titleInputRef = useRef<HTMLInputElement | null>(null);

  const { data, isLoading, isError, error, refetch } = usePosts();

  if (isLoading) return <PostsListSkeleton rows={5} />;

  if (isError) {
    return (
      <PostsListError
        message={(error as Error).message ?? "Unknown error"}
        onRetry={() => refetch()}
      />
    );
  }

  if (!data || data.length === 0) {
    return (
      <PostsListEmpty
        onCreate={() => {
          titleInputRef.current?.focus();
          titleInputRef.current?.scrollIntoView({ behavior: "smooth" });
        }}
      />
    );
  }

  return (
    <ul className="space-y-2">
      {data.slice(0, 10).map((post) => (
        <PostRow key={post.id} post={post} />
      ))}
    </ul>
  );
}

export default PostsList;
```

Notice the **order** of the checks: loading first, then error, then empty, then data. Get this order wrong and you might render a "no posts" message during the first loading frame.

### Testing each state

A nice technique: temporarily force each state by editing the query for a moment.

| State | How to force it |
|-------|-----------------|
| Loading | Throttle the network to "Slow 3G" in DevTools |
| Error | Change the `baseURL` to a nonsense URL for a moment |
| Empty | Filter the data: `data: data?.filter(() => false)` |
| Data | Normal flow |

---

## 47.6 Toasts on Mutations with Sonner

Visual feedback after a mutation is part of feeling polished. You used the shadcn Sonner toaster in Class 41 — now it really earns its place inside `useMutation`'s success/error callbacks.

If you have not added it yet:

```bash
npx shadcn@latest add sonner
```

Mount the `<Toaster />` once at the root, alongside `QueryClientProvider`:

```tsx
// src/main.tsx
import { Toaster } from "@/components/ui/sonner";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
      <Toaster richColors position="top-right" />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  </React.StrictMode>,
);
```

Now update each mutation hook to fire a toast:

```ts
// src/hooks/usePosts.ts (excerpt)
import { toast } from "sonner";

export function useCreatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreatePostData) => postApi.create(data),
    onSuccess: () => {
      toast.success("Post created");
      queryClient.invalidateQueries({ queryKey: postKeys.lists() });
    },
    onError: (error: Error) => {
      toast.error(error.message || "Failed to create post");
    },
  });
}

export function useUpdatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: number; data: UpdatePostData }) =>
      postApi.update(id, data),
    onSuccess: (_updated, variables) => {
      toast.success("Post updated");
      queryClient.invalidateQueries({ queryKey: postKeys.lists() });
      queryClient.invalidateQueries({ queryKey: postKeys.detail(variables.id) });
    },
    onError: (error: Error) => {
      toast.error(error.message || "Failed to update post");
    },
  });
}

export function useDeletePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: number) => postApi.delete(id),
    onSuccess: () => {
      toast.success("Post deleted");
      queryClient.invalidateQueries({ queryKey: postKeys.lists() });
    },
    onError: (error: Error) => {
      toast.error(error.message || "Failed to delete post");
    },
  });
}
```

Putting toasts inside the hook (rather than in every component) means **every** caller benefits automatically. One source of truth.

---

## 47.7 Disabling Buttons During Mutations

This was introduced in Class 43 (`isSubmitting`) and Class 46 (`isPending`). It is worth a reminder because it is the easiest polish you can add for a noticeably better feel:

```tsx
const { mutate: deletePost, isPending } = useDeletePost();

<Button
  variant="destructive"
  disabled={isPending}
  onClick={() => deletePost(post.id)}
>
  {isPending ? "Deleting..." : "Delete"}
</Button>
```

Two benefits:

1. Prevents the user clicking the button five times in a row.
2. The label change gives clear feedback that *something* is happening.

Add this to every action button on the CRUD page — create, save, delete.

---

## 47.8 The Polished CRUD Page

Here is the full layout after today's polish. Notice how every state is accounted for.

```tsx
// src/App.tsx
import CreatePostForm from "@/components/CreatePostForm";
import PostsList from "@/components/PostsList";

function App() {
  return (
    <div className="min-h-screen p-6 max-w-2xl mx-auto space-y-8">
      <header>
        <h1 className="text-2xl font-bold">Posts CRUD</h1>
        <p className="text-sm text-muted-foreground">
          Polished with skeletons, error retry and toasts.
        </p>
      </header>

      <section className="space-y-2">
        <h2 className="text-lg font-semibold">Create a post</h2>
        <CreatePostForm />
      </section>

      <section className="space-y-2">
        <h2 className="text-lg font-semibold">All posts</h2>
        <PostsList />
      </section>
    </div>
  );
}

export default App;
```

Try every path:

- Open the app — see the **skeleton** for a beat, then the data.
- Throttle the network to fail (DevTools → Network → Offline), refetch — see the **error** UI, click **Try again** to recover when you go back online.
- Create a post — green toast, list refreshes, **Creating...** label while pending.
- Delete the only visible post repeatedly until the list looks empty — see the **empty** state (in development you can fake this by editing the query to slice to 0).

Each of those moments is now intentional, not accidental.

---

## 47.9 A Useful Decision Tree

When you build a new data page, ask these questions in order:

```
Has the request ever been made?
   No  -> idle state
   Yes:
      Is it currently the first fetch?  (isLoading)
         Yes -> skeleton
         No:
            Did it fail?  (isError)
               Yes -> error UI + retry
               No:
                  Is the list empty?  (data.length === 0)
                     Yes -> empty state + CTA
                     No  -> render the data
```

Print this out, put it on your wall. Every screen.

---

## Practice Exercises

### Exercise 1: Polish a `<UsersList>`
- Use the `useUsers` hook from Class 45's exercises
- Build a `<UsersListSkeleton>`, `<UsersListError>` and `<UsersListEmpty>`
- Show the empty state by temporarily slicing the data to length 0

### Exercise 2: A retry counter
- In `<PostsListError>`, add a counter showing how many times the user has clicked **Try again**
- Hint: `useState(0)` and `setCount((c) => c + 1)` in `onRetry`

### Exercise 3: Different skeleton lengths
- Make `PostsListSkeleton` accept a `rows` prop (it already does) and randomise the widths of the placeholder lines on each render so it looks more natural
- Hint: `Math.random()` inside the map

### Exercise 4: Toasts on the form
- Edit `useCreatePost` so the success toast shows the **title** of the created post (e.g. `Post created: "Hello world"`)
- Hint: `onSuccess` receives the created object as its first argument

---

## Key Takeaways
- Every data screen has five states — idle, loading, error, empty, data. Plan for all of them.
- Skeletons should mirror the shape of the real content, not be a single grey block.
- Always offer a way out of an error state with a **Retry** button calling `refetch()`.
- Empty states need a clear call-to-action — never leave the user on a blank page.
- Configure `retry`, `retryDelay`, and `refetchOnWindowFocus` on the `QueryClient` for sensible defaults across the whole app.
- Put Sonner toasts inside the mutation hooks (`onSuccess` / `onError`) so every caller benefits.
- Always disable action buttons during pending mutations, and change the label to reassure the user.
- **Next class**: real client-side routing with React Router — turn the toggle into proper `/posts` and `/posts/:id` URLs.
