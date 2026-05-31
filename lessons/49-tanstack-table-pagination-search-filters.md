# Class 49: TanStack Table — Pagination, Search and Filters

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 48 (React Router v6), Class 47 (loading/error/empty states), Class 46 (mutations)

## What You Will Learn
- Why a real `<DataTable>` matters in admin-style apps and why we use a library for it
- Installing `@tanstack/react-table` and the shadcn `<Table>` primitives
- Building a reusable typed generic `<DataTable<TData, TValue>>` component
- Defining `ColumnDef<Post>[]` — text columns, badge cells, date cells, action cells
- Client-side pagination and sorting
- Server-style pagination with `manualPagination` and `pageCount`
- URL-synced search and filters using `useSearchParams`
- A debounced search input (300ms)
- Row actions: edit dialog and delete confirmation
- `placeholderData: keepPreviousData` to remove flicker between pages

---

## 49.1 Why a Real Table?

So far our list of posts has been a `space-y-2` of cards. That is fine for ten items on a personal page, but admin tools need much more:

- Sortable columns ("show me the newest first")
- Pagination (do not download 10,000 rows)
- Search and filters (find one row in thousands)
- Per-row actions (edit, delete, archive)
- Sticky headers, responsive overflow, column visibility

You *could* build all of that by hand. `@tanstack/react-table` v8 gives it to you for free, and it is **headless** — the library owns the *logic* and lets you control the *markup*. We pair it with shadcn's `<Table>` for the look.

| Hand-rolled list | TanStack Table |
|------------------|----------------|
| Manual `.map()` over items | Define columns once, library renders the rows |
| Build pagination buttons yourself | `table.previousPage()`, `table.nextPage()` |
| Track sort in component state | Built-in sorting state |
| Filter logic mixed with UI | Filter state lives separately |
| One-off for each entity | One `<DataTable>` reused for posts, users, anything |

We will build a **generic** `<DataTable>` so the same component renders a posts table today and a users table next week.

---

## 49.2 Installing the Dependencies

```bash
npm install @tanstack/react-table
npx shadcn@latest add table select badge alert-dialog dialog
```

- `@tanstack/react-table` — the headless table library (v8).
- `table`, `select`, `badge`, `alert-dialog`, `dialog` — shadcn components we will use for the markup, filters, and row actions.

You should already have `lucide-react` from earlier classes — if not, install it for the icons:

```bash
npm install lucide-react
```

---

## 49.3 The Generic `<DataTable>` Component

This is the centrepiece — a reusable component that knows nothing about posts. It takes columns and rows, renders the table, and surfaces pagination controls. We will use it later for users, todos, and anything else.

```tsx
// src/components/ui/data-table.tsx
import {
  flexRender,
  getCoreRowModel,
  getPaginationRowModel,
  getSortedRowModel,
  useReactTable,
  type ColumnDef,
  type SortingState,
} from "@tanstack/react-table";
import { useState } from "react";
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";
import { Skeleton } from "@/components/ui/skeleton";

interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[];
  data: TData[];
  isLoading?: boolean;
  emptyMessage?: string;
  // For server-side pagination
  manualPagination?: boolean;
  pageCount?: number;
  pageIndex?: number;
  pageSize?: number;
}

export function DataTable<TData, TValue>({
  columns,
  data,
  isLoading,
  emptyMessage = "No results.",
  manualPagination = false,
  pageCount,
  pageIndex = 0,
  pageSize = 10,
}: DataTableProps<TData, TValue>) {
  const [sorting, setSorting] = useState<SortingState>([]);

  const table = useReactTable({
    data,
    columns,
    state: {
      sorting,
      ...(manualPagination ? { pagination: { pageIndex, pageSize } } : {}),
    },
    onSortingChange: setSorting,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getPaginationRowModel: manualPagination ? undefined : getPaginationRowModel(),
    manualPagination,
    pageCount: manualPagination ? pageCount ?? -1 : undefined,
  });

  return (
    <div className="rounded-md border">
      <Table>
        <TableHeader>
          {table.getHeaderGroups().map((headerGroup) => (
            <TableRow key={headerGroup.id}>
              {headerGroup.headers.map((header) => (
                <TableHead key={header.id}>
                  {header.isPlaceholder
                    ? null
                    : flexRender(
                        header.column.columnDef.header,
                        header.getContext(),
                      )}
                </TableHead>
              ))}
            </TableRow>
          ))}
        </TableHeader>
        <TableBody>
          {isLoading ? (
            Array.from({ length: pageSize }).map((_, idx) => (
              <TableRow key={idx}>
                {columns.map((_col, colIdx) => (
                  <TableCell key={colIdx}>
                    <Skeleton className="h-5 w-full" />
                  </TableCell>
                ))}
              </TableRow>
            ))
          ) : table.getRowModel().rows.length ? (
            table.getRowModel().rows.map((row) => (
              <TableRow key={row.id}>
                {row.getVisibleCells().map((cell) => (
                  <TableCell key={cell.id}>
                    {flexRender(cell.column.columnDef.cell, cell.getContext())}
                  </TableCell>
                ))}
              </TableRow>
            ))
          ) : (
            <TableRow>
              <TableCell
                colSpan={columns.length}
                className="h-24 text-center text-muted-foreground"
              >
                {emptyMessage}
              </TableCell>
            </TableRow>
          )}
        </TableBody>
      </Table>
    </div>
  );
}
```

A few things to highlight for students:

| Concept | Meaning |
|---------|---------|
| `<TData, TValue>` | Generic types. The table works with any row shape (`Post`, `User`, etc.) |
| `useReactTable({...})` | The hook that builds the table instance |
| `getCoreRowModel()` | Tells the table how to compute its rows. Required. |
| `getSortedRowModel()` | Enables client-side sorting |
| `getPaginationRowModel()` | Enables client-side pagination (we skip this for server-side mode) |
| `manualPagination: true` | "The server handles pagination — just render what I give you" |
| `flexRender(...)` | Renders either a string header or a JSX cell from the column definition |

---

## 49.4 Defining Columns for Posts

Columns are **data**, not JSX. You describe each column once with a `ColumnDef`, and the table renders accordingly. Putting column definitions in their own hook keeps the parent component small.

```tsx
// src/components/posts/post-columns.tsx
import type { ColumnDef } from "@tanstack/react-table";
import { ArrowUpDown } from "lucide-react";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import { PostRowActions } from "./post-row-actions";
import type { Post } from "@/types/post";

// JSONPlaceholder posts do not include a date — invent one from the id
// so we can show students a date column. In a real app, the server returns it.
function fakeDateFromId(id: number) {
  const base = new Date("2025-01-01").getTime();
  return new Date(base + id * 24 * 60 * 60 * 1000);
}

export function usePostColumns(): ColumnDef<Post>[] {
  return [
    {
      accessorKey: "id",
      header: "ID",
      cell: ({ row }) => (
        <span className="font-mono text-xs text-muted-foreground">
          #{row.original.id}
        </span>
      ),
    },
    {
      accessorKey: "title",
      header: ({ column }) => (
        <Button
          variant="ghost"
          size="sm"
          className="-ml-3 h-8"
          onClick={() => column.toggleSorting(column.getIsSorted() === "asc")}
        >
          Title <ArrowUpDown className="ml-1 h-3 w-3" />
        </Button>
      ),
      cell: ({ row }) => (
        <span className="font-medium">{row.original.title}</span>
      ),
    },
    {
      accessorKey: "userId",
      header: "Author",
      cell: ({ row }) => (
        <Badge variant="secondary">User {row.original.userId}</Badge>
      ),
    },
    {
      id: "createdAt",
      header: "Created",
      cell: ({ row }) =>
        fakeDateFromId(row.original.id).toLocaleDateString("en-GB", {
          day: "2-digit",
          month: "short",
          year: "numeric",
        }),
    },
    {
      id: "actions",
      header: "",
      cell: ({ row }) => <PostRowActions post={row.original} />,
    },
  ];
}
```

The patterns to recognise:

| Pattern | Example | When |
|---------|---------|------|
| `accessorKey` only | Reads `row.id` directly | Plain field |
| Custom `cell` function | Returns JSX using `row.original` | Badges, dates, formatted text |
| `id` with no `accessorKey` | The action column | Columns not mapped to a field |
| Header as a function | Sortable header with a button | When you need sorting controls |

---

## 49.5 Row Actions: Edit Dialog and Delete Confirm

Each row gets an edit button (opens a shadcn `<Dialog>`) and a delete button (opens an `<AlertDialog>` to confirm).

```tsx
// src/components/posts/post-row-actions.tsx
import { useState } from "react";
import { Pencil, Trash2 } from "lucide-react";
import { Button } from "@/components/ui/button";
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from "@/components/ui/alert-dialog";
import { EditPostDialog } from "./edit-post-dialog";
import { useDeletePost } from "@/hooks/usePosts";
import type { Post } from "@/types/post";

export function PostRowActions({ post }: { post: Post }) {
  const [editOpen, setEditOpen] = useState(false);
  const { mutate: deletePost, isPending: isDeleting } = useDeletePost();

  return (
    <div className="flex items-center justify-end gap-1">
      <Button
        variant="ghost"
        size="icon"
        onClick={() => setEditOpen(true)}
        aria-label="Edit post"
      >
        <Pencil className="h-4 w-4" />
      </Button>

      <AlertDialog>
        <AlertDialogTrigger asChild>
          <Button
            variant="ghost"
            size="icon"
            disabled={isDeleting}
            aria-label="Delete post"
          >
            <Trash2 className="h-4 w-4 text-destructive" />
          </Button>
        </AlertDialogTrigger>
        <AlertDialogContent>
          <AlertDialogHeader>
            <AlertDialogTitle>Delete this post?</AlertDialogTitle>
            <AlertDialogDescription>
              "{post.title}" will be removed. This action cannot be undone.
            </AlertDialogDescription>
          </AlertDialogHeader>
          <AlertDialogFooter>
            <AlertDialogCancel>Cancel</AlertDialogCancel>
            <AlertDialogAction onClick={() => deletePost(post.id)}>
              Delete
            </AlertDialogAction>
          </AlertDialogFooter>
        </AlertDialogContent>
      </AlertDialog>

      <EditPostDialog
        post={post}
        open={editOpen}
        onOpenChange={setEditOpen}
      />
    </div>
  );
}
```

The edit dialog reuses the form pattern from Class 44 plus the mutation hook from Class 46:

```tsx
// src/components/posts/edit-post-dialog.tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";
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
import { useUpdatePost } from "@/hooks/usePosts";
import type { Post } from "@/types/post";

const editSchema = z.object({
  title: z.string().trim().min(3, "Title must be at least 3 characters"),
  body: z.string().trim().min(10, "Body must be at least 10 characters"),
});
type EditValues = z.infer<typeof editSchema>;

interface EditPostDialogProps {
  post: Post;
  open: boolean;
  onOpenChange: (open: boolean) => void;
}

export function EditPostDialog({ post, open, onOpenChange }: EditPostDialogProps) {
  const form = useForm<EditValues>({
    resolver: zodResolver(editSchema),
    defaultValues: { title: post.title, body: post.body },
  });

  const { mutate: updatePost, isPending } = useUpdatePost();

  const onSubmit = (data: EditValues) => {
    updatePost(
      { id: post.id, data },
      { onSuccess: () => onOpenChange(false) },
    );
  };

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Edit post #{post.id}</DialogTitle>
          <DialogDescription>Update the title and body, then save.</DialogDescription>
        </DialogHeader>

        <Form {...form}>
          <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-3">
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
            <DialogFooter>
              <Button type="button" variant="outline" onClick={() => onOpenChange(false)}>
                Cancel
              </Button>
              <Button type="submit" disabled={isPending}>
                {isPending ? "Saving..." : "Save"}
              </Button>
            </DialogFooter>
          </form>
        </Form>
      </DialogContent>
    </Dialog>
  );
}
```

---

## 49.6 URL-Synced Filters with `useSearchParams`

Real admin tables put filter state in the **URL**, not in `useState`. Why?

- Refreshing the page keeps the user's filters.
- Users can **bookmark** filtered views.
- Users can **share** a URL like `?q=react&userId=3` with a colleague.
- The browser's back and forward buttons work for free.

React Router gives us `useSearchParams`, which behaves like `useState` but is backed by the URL query string. We wrap it in a small hook so the rest of the code stays simple.

```ts
// src/hooks/usePostsFilters.ts
import { useSearchParams } from "react-router-dom";

export interface PostsFilters {
  page: number;
  limit: number;
  q: string;
  userId: number | undefined;
}

export function usePostsFilters() {
  const [params, setParams] = useSearchParams();

  const filters: PostsFilters = {
    page: Number(params.get("page")) || 1,
    limit: Number(params.get("limit")) || 10,
    q: params.get("q") ?? "",
    userId: params.get("userId") ? Number(params.get("userId")) : undefined,
  };

  function setFilters(updates: Partial<PostsFilters>) {
    const next = new URLSearchParams(params);
    for (const [key, value] of Object.entries(updates)) {
      if (value === undefined || value === "" || value === null) {
        next.delete(key);
      } else {
        next.set(key, String(value));
      }
    }
    setParams(next, { replace: true });
  }

  // Reset the page to 1 whenever a non-page filter changes
  function setFilter<K extends keyof PostsFilters>(key: K, value: PostsFilters[K]) {
    setFilters({ [key]: value, page: 1 } as Partial<PostsFilters>);
  }

  function resetFilters() {
    setParams(new URLSearchParams(), { replace: true });
  }

  return { filters, setFilters, setFilter, resetFilters };
}
```

`replace: true` updates the URL **without** adding an entry to browser history — otherwise typing in a search box would create one history entry per keystroke.

---

## 49.7 The Debounced Search Input

JSONPlaceholder does not support a real search endpoint, so we will keep search **client-side** for the demo — filter the in-memory list by title. The URL still drives the input value so the search is shareable.

A debounced input updates a local state immediately (so the input feels responsive) but only writes to the URL 300ms after the user stops typing.

```tsx
// src/components/posts/posts-filters.tsx
import { useEffect, useState } from "react";
import { X } from "lucide-react";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import { usePostsFilters } from "@/hooks/usePostsFilters";

export function PostsFilters() {
  const { filters, setFilter, resetFilters } = usePostsFilters();
  const [searchInput, setSearchInput] = useState(filters.q);

  // Debounce: wait 300ms after the user stops typing, then write to the URL
  useEffect(() => {
    const handle = setTimeout(() => {
      if (searchInput !== filters.q) {
        setFilter("q", searchInput);
      }
    }, 300);
    return () => clearTimeout(handle);
  }, [searchInput]); // eslint-disable-line react-hooks/exhaustive-deps

  // Keep the input in sync if the URL changes externally (e.g. "Clear")
  useEffect(() => {
    setSearchInput(filters.q);
  }, [filters.q]);

  const hasActive = !!filters.q || filters.userId !== undefined;

  return (
    <div className="flex flex-wrap items-center gap-2">
      <Input
        placeholder="Search by title..."
        value={searchInput}
        onChange={(e) => setSearchInput(e.target.value)}
        className="max-w-xs"
      />

      <Select
        value={filters.userId !== undefined ? String(filters.userId) : "all"}
        onValueChange={(v) =>
          setFilter("userId", v === "all" ? undefined : Number(v))
        }
      >
        <SelectTrigger className="w-[160px]">
          <SelectValue placeholder="Author" />
        </SelectTrigger>
        <SelectContent>
          <SelectItem value="all">All authors</SelectItem>
          {[1, 2, 3, 4, 5].map((n) => (
            <SelectItem key={n} value={String(n)}>
              User {n}
            </SelectItem>
          ))}
        </SelectContent>
      </Select>

      {hasActive && (
        <Button variant="ghost" size="sm" onClick={resetFilters}>
          <X className="h-4 w-4 mr-1" /> Clear
        </Button>
      )}
    </div>
  );
}
```

> **How debouncing works**: every keystroke updates `searchInput` immediately. The `setTimeout` only fires when the user **stops** typing for 300ms. If they type again before then, we `clearTimeout` and start over. Net effect: at most one URL update per typing burst.

---

## 49.8 Pagination Controls

The pagination bar reads from filters and writes back through `setFilters`.

```tsx
// src/components/posts/posts-pagination.tsx
import { ChevronLeft, ChevronRight, ChevronsLeft, ChevronsRight } from "lucide-react";
import { Button } from "@/components/ui/button";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import { usePostsFilters } from "@/hooks/usePostsFilters";

export function PostsPagination({
  totalPages,
  totalItems,
}: {
  totalPages: number;
  totalItems: number;
}) {
  const { filters, setFilters } = usePostsFilters();
  const { page, limit } = filters;

  const hasPrev = page > 1;
  const hasNext = page < totalPages;

  return (
    <div className="flex flex-wrap items-center justify-between gap-4 px-2">
      <div className="text-sm text-muted-foreground">
        Page {page} of {totalPages} ({totalItems} total)
      </div>

      <div className="flex items-center gap-2">
        <span className="text-sm text-muted-foreground">Rows</span>
        <Select
          value={String(limit)}
          onValueChange={(v) => setFilters({ limit: Number(v), page: 1 })}
        >
          <SelectTrigger className="h-8 w-[70px]">
            <SelectValue />
          </SelectTrigger>
          <SelectContent>
            {[5, 10, 20, 50].map((n) => (
              <SelectItem key={n} value={String(n)}>{n}</SelectItem>
            ))}
          </SelectContent>
        </Select>

        <div className="flex items-center gap-1">
          <Button variant="outline" size="icon" className="h-8 w-8"
            disabled={!hasPrev} onClick={() => setFilters({ page: 1 })}>
            <ChevronsLeft className="h-4 w-4" />
          </Button>
          <Button variant="outline" size="icon" className="h-8 w-8"
            disabled={!hasPrev} onClick={() => setFilters({ page: page - 1 })}>
            <ChevronLeft className="h-4 w-4" />
          </Button>
          <Button variant="outline" size="icon" className="h-8 w-8"
            disabled={!hasNext} onClick={() => setFilters({ page: page + 1 })}>
            <ChevronRight className="h-4 w-4" />
          </Button>
          <Button variant="outline" size="icon" className="h-8 w-8"
            disabled={!hasNext} onClick={() => setFilters({ page: totalPages })}>
            <ChevronsRight className="h-4 w-4" />
          </Button>
        </div>
      </div>
    </div>
  );
}
```

---

## 49.9 The Orchestrator: `<PostsTable>`

All the pieces come together here. Notice how small this file is — every responsibility lives in its own component or hook.

```tsx
// src/components/posts/posts-table.tsx
import { useMemo } from "react";
import { keepPreviousData, useQuery } from "@tanstack/react-query";
import { DataTable } from "@/components/ui/data-table";
import { PostsFilters } from "./posts-filters";
import { PostsPagination } from "./posts-pagination";
import { usePostColumns } from "./post-columns";
import { usePostsFilters } from "@/hooks/usePostsFilters";
import { postApi } from "@/services/postApi";
import { postKeys } from "@/hooks/postKeys";
import type { Post } from "@/types/post";

export function PostsTable() {
  const { filters } = usePostsFilters();
  const columns = usePostColumns();

  // Fetch all posts (JSONPlaceholder returns 100) and paginate/filter client-side.
  // In a real backend, you would send page/limit/q/userId as query params and let
  // the server return the right slice with a total count.
  const { data: allPosts = [], isLoading } = useQuery({
    queryKey: postKeys.list({}),
    queryFn: () => postApi.getAll(),
    placeholderData: keepPreviousData, // no flicker between page changes
  });

  // Apply search and userId filter
  const filtered = useMemo(() => {
    return allPosts.filter((p: Post) => {
      if (filters.q && !p.title.toLowerCase().includes(filters.q.toLowerCase())) {
        return false;
      }
      if (filters.userId !== undefined && p.userId !== filters.userId) {
        return false;
      }
      return true;
    });
  }, [allPosts, filters.q, filters.userId]);

  // Slice for the current page (server-side mode, but the slicing is local for the demo)
  const totalItems = filtered.length;
  const totalPages = Math.max(1, Math.ceil(totalItems / filters.limit));
  const start = (filters.page - 1) * filters.limit;
  const pageRows = filtered.slice(start, start + filters.limit);

  return (
    <div className="space-y-4">
      <PostsFilters />
      <DataTable
        columns={columns}
        data={pageRows}
        isLoading={isLoading}
        emptyMessage="No posts match your filters."
        manualPagination
        pageCount={totalPages}
        pageIndex={filters.page - 1}
        pageSize={filters.limit}
      />
      <PostsPagination totalPages={totalPages} totalItems={totalItems} />
    </div>
  );
}
```

Two important details:

- `placeholderData: keepPreviousData` keeps the previous rows visible while a refetch is happening. The table no longer flashes to a loading state every time the page changes.
- We pass `manualPagination`, `pageCount` and `pageIndex` to the table — that tells TanStack Table that **we** control the slice, and it should just render whatever rows we give it.

---

## 49.10 Wire It Into a Page

Replace the card list of posts on the posts page with the new table:

```tsx
// src/pages/PostsPage.tsx
import { PostsTable } from "@/components/posts/posts-table";

function PostsPage() {
  return (
    <div className="space-y-4">
      <h1 className="text-2xl font-bold">All posts</h1>
      <PostsTable />
    </div>
  );
}

export default PostsPage;
```

Run `npm run dev` and try it:

- Sort by title (click the header) — order changes instantly.
- Type in the search — URL becomes `?q=...&page=1` 300ms after you stop typing.
- Pick an author from the dropdown — URL adds `&userId=...`, table updates.
- Click **next page** — instant, no spinner, no flicker.
- Refresh the page — your filters are preserved.
- Copy the URL into another tab — same filtered view.
- Click **Edit** on a row — dialog opens with the form prefilled.
- Click **Delete** — confirmation, then the row disappears (toast appears thanks to Class 47).

That whole experience is the standard admin-table feel for modern web apps.

---

## 49.11 The Data Flow End-to-End

```
User types in search input
       |
       v
PostsFilters local state updates (input is responsive)
       |
       v
After 300ms of silence, setFilter("q", value)
       |
       v
URL becomes ?q=react&page=1
       |
       v
usePostsFilters re-reads the URL
       |
       v
PostsTable recomputes filtered/sliced rows
       |
       v
DataTable receives new data + new pageIndex
       |
       v
Browser back/forward jumps the URL, and the whole pipeline runs in reverse
```

The URL is the **single source of truth**. Every other component derives from it.

---

## 49.12 Switching to Real Server-Side Pagination

For JSONPlaceholder we filter client-side because it does not have a search endpoint. With a real backend you would do this instead:

1. Send the filters to the server as Axios query params:
   ```ts
   api.get<{ data: Post[]; meta: { totalPages: number; total: number } }>(
     "/posts",
     { params: filters },
   );
   ```
2. The server returns the right slice plus a `totalPages` and `total`.
3. Pass those to the table directly — no `useMemo`, no client-side slicing.

The hook signature `useTodos(filters)` from Class 46 already accepts filters, so the change is small. You will use this pattern in the capstone (Classes 50 and 51) and again in the MERN course.

---

## Practice Exercises

### Exercise 1: Build a `<UsersTable>`
- Reuse the generic `<DataTable>` directly
- Define `ColumnDef<User>[]` with columns: `id`, `name`, `username`, `email`, action
- Add a row action button that links to `/users/:id` (you built that route in Class 48's exercises)
- No filters needed — just sortable columns and pagination

### Exercise 2: A status badge column
- Add a fake `published: boolean` flag to your `Post` type (compute it from `id % 2 === 0` so half the rows are "Published")
- Add a column that renders a `<Badge>` — `default` for published, `secondary` for draft
- Add a filter dropdown: All / Published / Draft

### Exercise 3: Column visibility
- Add a `Select` (or a `<DropdownMenu>`) above the table with checkboxes for each column
- Wire it to TanStack Table's column visibility state (`table.getAllLeafColumns()`)
- Hint: read the TanStack Table docs section on **column visibility**

### Exercise 4: Bulk selection
- Add a checkbox column at the very left
- Enable row selection in TanStack Table
- Add a **Delete selected** button above the table that fires `useDeletePost` for every selected row

---

## Key Takeaways
- `@tanstack/react-table` is **headless** — it owns the table logic; you control the markup.
- A generic `<DataTable<TData, TValue>>` is reusable across every entity in your app.
- Columns are **data**, not JSX — define them with `ColumnDef<T>[]` and pass them in.
- Use `accessorKey` for simple fields, custom `cell` functions for badges/dates, and `id` for action columns.
- Server-side pagination: `manualPagination: true` plus `pageCount` from the server.
- Store filter state in the **URL** with `useSearchParams` — bookmarkable, shareable, refresh-proof.
- **Debounced** search (300ms) stops you hammering the URL or the API on every keystroke.
- `placeholderData: keepPreviousData` keeps the previous rows visible while the next page loads — no flicker.
- Changing any filter should reset `page` to 1, otherwise filtering on page 5 might show empty results.
- Row actions live in their own component; edit uses `<Dialog>`, delete uses `<AlertDialog>` for confirmation.
- **Next class**: the capstone — bring everything together into a Users & Posts dashboard.
