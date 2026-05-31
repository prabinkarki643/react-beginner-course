# Class 46: Mutations and Cache Invalidation

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 45 (Axios + React Query intro)

## What You Will Learn
- Why `useMutation` exists and how it differs from `useQuery`
- The "one hook per action" pattern: `useCreatePost`, `useUpdatePost`, `useDeletePost`
- Query keys **factories** for type-safe, hierarchical cache invalidation
- `queryClient.invalidateQueries({ queryKey })` and what it actually does
- A gentle introduction to optimistic updates
- Building full CRUD for posts against JSONPlaceholder

---

## 46.1 Queries Read, Mutations Write

In Class 45 you used `useQuery` to **read** data: list of posts, single post, list of users. Reading is idempotent — running the same query twice does the same thing, and React Query can safely cache it.

**Writing** data is different. When the user clicks "create post", you do not want React Query to call that endpoint twice or cache the result. You want a one-off action that you trigger when the user clicks a button. That is `useMutation`.

| Action | Hook |
|--------|------|
| GET (read) | `useQuery` |
| POST (create) | `useMutation` |
| PUT / PATCH (update) | `useMutation` |
| DELETE (remove) | `useMutation` |

> Analogy: queries are like checking the menu on a restaurant table — you do it any time and the menu does not change. Mutations are like placing an order — you do it once, intentionally, and it changes the kitchen's state.

---

## 46.2 Shape of `useMutation`

`useMutation` returns an object with two crucial pieces:

- `mutate(variables)` — call this from your event handler to fire the request.
- `isPending` — `true` while the request is in flight (used to disable buttons).

It accepts an object with `mutationFn`, `onSuccess`, `onError`, etc.

```ts
const mutation = useMutation({
  mutationFn: (data: CreatePostData) => api.post("/posts", data).then((r) => r.data),
  onSuccess: (created) => {
    console.log("Created:", created);
  },
  onError: (error) => {
    console.error(error);
  },
});

// Later, in a click handler:
mutation.mutate({ title: "Hello", body: "World", userId: 1 });
```

The `onSuccess` and `onError` callbacks are where we plug in cache invalidation, toasts, and form resets.

> JSONPlaceholder note: the writes (POST / PUT / DELETE) are *mocked*. The server replies as if the post was created, but it is not really saved. That is perfectly fine for learning — you see the mutation flow working end-to-end without needing a backend.

---

## 46.3 The Types We Need

```ts
// src/types/post.ts
export interface Post {
  id: number;
  userId: number;
  title: string;
  body: string;
}

export interface CreatePostData {
  title: string;
  body: string;
  userId: number;
}

export interface UpdatePostData {
  title?: string;
  body?: string;
}
```

Three separate types — one for the full server-side object, one for creating, one for updating. This matches how real APIs are usually shaped.

---

## 46.4 The API Service Layer

Let us extend our `api.ts` usage with a thin service that wraps every HTTP call. This keeps Axios calls out of components.

```ts
// src/services/postApi.ts
import api from "@/lib/api";
import type { Post, CreatePostData, UpdatePostData } from "@/types/post";

export const postApi = {
  async getAll(): Promise<Post[]> {
    const { data } = await api.get<Post[]>("/posts");
    return data;
  },

  async getById(id: number): Promise<Post> {
    const { data } = await api.get<Post>(`/posts/${id}`);
    return data;
  },

  async create(payload: CreatePostData): Promise<Post> {
    const { data } = await api.post<Post>("/posts", payload);
    return data;
  },

  async update(id: number, payload: UpdatePostData): Promise<Post> {
    const { data } = await api.put<Post>(`/posts/${id}`, payload);
    return data;
  },

  async delete(id: number): Promise<void> {
    await api.delete(`/posts/${id}`);
  },
};
```

---

## 46.5 Query Keys Factories

Before we write the hooks, let us tidy up our cache keys. Sprinkling raw arrays like `["posts"]` and `["posts", id]` across the codebase is fragile — typos quietly break invalidation. The fix is a **query keys factory**: an object that holds every key for a given resource.

```ts
// src/hooks/postKeys.ts
export const postKeys = {
  all: ["posts"] as const,
  lists: () => [...postKeys.all, "list"] as const,
  list: (filters: Record<string, unknown>) =>
    [...postKeys.lists(), filters] as const,
  details: () => [...postKeys.all, "detail"] as const,
  detail: (id: number) => [...postKeys.details(), id] as const,
};
```

This looks complicated but the idea is simple — each level **nests** inside the previous one:

```
postKeys.all              => ["posts"]
postKeys.lists()          => ["posts", "list"]
postKeys.list({})         => ["posts", "list", {}]
postKeys.details()        => ["posts", "detail"]
postKeys.detail(1)        => ["posts", "detail", 1]
```

### Why does the hierarchy matter?

React Query treats query keys **as prefixes**. When you invalidate `["posts"]`, it invalidates **every** query whose key starts with `"posts"` — lists, details, all of them. When you invalidate `["posts", "list"]`, only the lists are invalidated, leaving the details alone.

| You invalidate | What gets refetched |
|----------------|---------------------|
| `postKeys.all` | Every posts query: list, detail, anything posts-related |
| `postKeys.lists()` | Only the list query |
| `postKeys.detail(5)` | Only the detail for post id 5 |

This is the foundation of clean cache management.

---

## 46.6 The Query Hooks

Update `usePosts.ts` to use the new factory:

```ts
// src/hooks/usePosts.ts
import { useQuery } from "@tanstack/react-query";
import { postApi } from "@/services/postApi";
import { postKeys } from "@/hooks/postKeys";

export function usePosts() {
  return useQuery({
    queryKey: postKeys.list({}),
    queryFn: () => postApi.getAll(),
  });
}

export function usePost(id: number) {
  return useQuery({
    queryKey: postKeys.detail(id),
    queryFn: () => postApi.getById(id),
    enabled: id > 0,
  });
}
```

Same behaviour as Class 45, but now the keys are centralised.

---

## 46.7 One Hook Per Mutation

This is the industry-standard pattern: one hook per action. Each hook owns its own success/error logic and invalidates exactly the right keys.

```ts
// src/hooks/usePosts.ts (continued)
import { useMutation, useQueryClient } from "@tanstack/react-query";
import type { CreatePostData, UpdatePostData } from "@/types/post";

export function useCreatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreatePostData) => postApi.create(data),
    onSuccess: () => {
      // The list is now stale — refetch it
      queryClient.invalidateQueries({ queryKey: postKeys.lists() });
    },
    onError: (error: Error) => {
      console.error("Create failed:", error.message);
    },
  });
}

export function useUpdatePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: number; data: UpdatePostData }) =>
      postApi.update(id, data),
    onSuccess: (_updated, variables) => {
      // The list AND the detail for this id are now stale
      queryClient.invalidateQueries({ queryKey: postKeys.lists() });
      queryClient.invalidateQueries({ queryKey: postKeys.detail(variables.id) });
    },
    onError: (error: Error) => {
      console.error("Update failed:", error.message);
    },
  });
}

export function useDeletePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: number) => postApi.delete(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: postKeys.lists() });
    },
    onError: (error: Error) => {
      console.error("Delete failed:", error.message);
    },
  });
}
```

Read each hook carefully — they all follow the same recipe:

1. `mutationFn` calls the API service.
2. `onSuccess` invalidates the queries that **depend** on the data we just changed.
3. `onError` logs the error (we add toasts in Class 47).

The key insight: invalidation is React Query's way of saying *"the data behind these keys is now stale — please refetch whichever ones are currently being displayed."*

---

## 46.8 Using a Mutation in a Component

We will build a small "create post" form using everything from Classes 43, 44 and 45.

### The schema

```ts
// src/schemas/postSchema.ts (reuse from Class 44, plus userId)
import { z } from "zod";

export const createPostSchema = z.object({
  title: z.string().trim().min(3, "Title must be at least 3 characters"),
  body: z.string().trim().min(10, "Body must be at least 10 characters"),
  userId: z.coerce.number().int().min(1).max(10),
});

export type CreatePostFormValues = z.infer<typeof createPostSchema>;
```

### The form

```tsx
// src/components/CreatePostForm.tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { createPostSchema, type CreatePostFormValues } from "@/schemas/postSchema";
import { useCreatePost } from "@/hooks/usePosts";
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "@/components/ui/form";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Button } from "@/components/ui/button";

function CreatePostForm() {
  const form = useForm<CreatePostFormValues>({
    resolver: zodResolver(createPostSchema),
    defaultValues: { title: "", body: "", userId: 1 },
  });

  const { mutate: createPost, isPending } = useCreatePost();

  const onSubmit = (data: CreatePostFormValues) => {
    createPost(data, {
      onSuccess: () => {
        form.reset();
      },
    });
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4 max-w-xl">
        <FormField
          control={form.control}
          name="title"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Title</FormLabel>
              <FormControl><Input {...field} /></FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <FormField
          control={form.control}
          name="body"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Body</FormLabel>
              <FormControl><Textarea rows={4} {...field} /></FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <FormField
          control={form.control}
          name="userId"
          render={({ field }) => (
            <FormItem>
              <FormLabel>User id (1–10)</FormLabel>
              <FormControl>
                <Input type="number" min={1} max={10} {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit" disabled={isPending}>
          {isPending ? "Creating..." : "Create post"}
        </Button>
      </form>
    </Form>
  );
}

export default CreatePostForm;
```

Two things to notice:

- We destructure `{ mutate: createPost, isPending }` from the hook. This gives a clean, action-named function for the click handler.
- The component-specific behaviour (`form.reset()` on success) is passed as a callback to `mutate()`. The hook's own `onSuccess` still runs the invalidation — both fire.

Submit the form, open the DevTools panel — you will see `["posts", "list", {}]` flip to **stale**, then **fetching**, then **fresh** again. Your list refreshes automatically.

---

## 46.9 Update and Delete in a Row

Now let us add edit and delete buttons to each list item.

```tsx
// src/components/PostRow.tsx
import { useState } from "react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { useUpdatePost, useDeletePost } from "@/hooks/usePosts";
import type { Post } from "@/types/post";

function PostRow({ post }: { post: Post }) {
  const [isEditing, setIsEditing] = useState(false);
  const [title, setTitle] = useState(post.title);
  const [body, setBody] = useState(post.body);

  const { mutate: updatePost, isPending: isUpdating } = useUpdatePost();
  const { mutate: deletePost, isPending: isDeleting } = useDeletePost();

  if (isEditing) {
    return (
      <li className="border p-3 rounded space-y-2">
        <Input value={title} onChange={(e) => setTitle(e.target.value)} />
        <Textarea
          rows={3}
          value={body}
          onChange={(e) => setBody(e.target.value)}
        />
        <div className="flex gap-2">
          <Button
            size="sm"
            disabled={isUpdating}
            onClick={() =>
              updatePost(
                { id: post.id, data: { title, body } },
                { onSuccess: () => setIsEditing(false) },
              )
            }
          >
            {isUpdating ? "Saving..." : "Save"}
          </Button>
          <Button size="sm" variant="outline" onClick={() => setIsEditing(false)}>
            Cancel
          </Button>
        </div>
      </li>
    );
  }

  return (
    <li className="border p-3 rounded flex items-start justify-between gap-3">
      <div>
        <h3 className="font-medium">{post.title}</h3>
        <p className="text-sm text-muted-foreground line-clamp-2">{post.body}</p>
      </div>
      <div className="flex flex-col gap-1 shrink-0">
        <Button size="sm" variant="outline" onClick={() => setIsEditing(true)}>
          Edit
        </Button>
        <Button
          size="sm"
          variant="destructive"
          disabled={isDeleting}
          onClick={() => deletePost(post.id)}
        >
          {isDeleting ? "Deleting..." : "Delete"}
        </Button>
      </div>
    </li>
  );
}

export default PostRow;
```

And the list:

```tsx
// src/components/PostsList.tsx
import { usePosts } from "@/hooks/usePosts";
import PostRow from "@/components/PostRow";

function PostsList() {
  const { data, isLoading, isError } = usePosts();

  if (isLoading) return <p>Loading...</p>;
  if (isError) return <p>Failed to load posts.</p>;

  return (
    <ul className="space-y-2">
      {data?.slice(0, 10).map((post) => (
        <PostRow key={post.id} post={post} />
      ))}
    </ul>
  );
}

export default PostsList;
```

Try it:

- Click **Edit**, change the text, click **Save** — the row flips back to view mode, the list refetches, your edit appears.
- Click **Delete** — the row disappears after the refetch (JSONPlaceholder accepts the delete and returns 200, even though it does not actually remove anything).

> JSONPlaceholder always returns the same mocked data, so on a hard refresh your "created" post will be gone. That is expected — the point of this class is the **flow**, not the persistence. We connect to a real backend in the MERN course.

---

## 46.10 Optimistic Updates (Intro)

Right now, when you click **Delete**, the row stays visible for a fraction of a second until the request finishes and the list refetches. For most apps that is fine. For something that should feel instant, you can do an **optimistic update** — update the cache *before* the server responds, and roll back if the server says no.

The pattern uses three new mutation callbacks: `onMutate`, `onError` (with rollback) and `onSettled`.

```ts
export function useDeletePost() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: number) => postApi.delete(id),
    // Runs BEFORE the request — update the cache optimistically
    onMutate: async (id) => {
      await queryClient.cancelQueries({ queryKey: postKeys.lists() });
      const previous = queryClient.getQueryData<Post[]>(postKeys.list({}));

      queryClient.setQueryData<Post[]>(postKeys.list({}), (old) =>
        old?.filter((p) => p.id !== id) ?? [],
      );

      // Return a snapshot so we can roll back on error
      return { previous };
    },
    onError: (_err, _id, context) => {
      // Restore the cache to what it was before
      if (context?.previous) {
        queryClient.setQueryData(postKeys.list({}), context.previous);
      }
    },
    onSettled: () => {
      // Whether it succeeded or failed, refetch the list to sync with the server
      queryClient.invalidateQueries({ queryKey: postKeys.lists() });
    },
  });
}
```

You do not need to memorise this — bookmark it as the recipe and reach for it when you genuinely need that instant-delete feel. We come back to optimistic updates in Class 47.

---

## 46.11 Full CRUD Wired Up

A tidy page that uses everything:

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
          Powered by JSONPlaceholder. Writes are mocked.
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

Run it. Create a post, edit a post, delete a post — and watch the DevTools panel. You should see your `["posts", "list", {}]` query go through `stale → fetching → fresh` after every mutation. That, in one screen, is professional-grade data-fetching architecture.

---

## Practice Exercises

### Exercise 1: A `useToggleCompleted` for todos
- Use the JSONPlaceholder `/todos` endpoint
- Type a `Todo` interface (`id`, `userId`, `title`, `completed`)
- Create `useToggleTodo()` that calls `PATCH /todos/:id` with `{ completed }`
- Use a `todoKeys` factory
- Add a checkbox to each todo row that fires the mutation on change

### Exercise 2: Confirm before delete
- Wrap the **Delete** button in a confirmation step:
  - On first click, change the button to red and label it **Confirm delete?**
  - On second click within 3 seconds, actually run the mutation
  - Otherwise revert
- Hint: `useState` for the confirming flag, `setTimeout` to revert

### Exercise 3: Reuse the pattern for users
- Replicate the entire CRUD on `/users` (JSONPlaceholder supports it)
- Create `userKeys`, `useUsers`, `useUser(id)`, `useCreateUser`, `useUpdateUser`, `useDeleteUser`
- Notice how quickly the second resource flies together when you already know the pattern

---

## Key Takeaways
- `useMutation` is for actions that change data; `useQuery` is for reading it.
- Destructure `{ mutate, isPending }` for clean call sites and disabled-button UX.
- A **query keys factory** (`postKeys.all`, `postKeys.lists()`, `postKeys.detail(id)`) is the cleanest way to manage cache keys and supports hierarchical invalidation.
- After every mutation, invalidate the keys for any **list** or **detail** that the change might affect.
- One hook per action keeps every component subscribing to only what it needs.
- JSONPlaceholder is fine for learning: it accepts writes and pretends they worked.
- Optimistic updates use `onMutate` to update the cache, return a snapshot, and roll back in `onError`.
- **Next class**: polish — skeleton loaders, error states with retry, empty states, and Sonner toasts.
