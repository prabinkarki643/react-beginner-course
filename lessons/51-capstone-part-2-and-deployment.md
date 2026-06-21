# Class 51: Capstone Part 2 — Polish & Firebase Deployment

> **Time budget**: 1 hour (≈ 50 min build + deploy + 10 min closing)
> **Prerequisites**: Class 50 (the dashboard must be running locally)

This is the final class of the course. By the end of the hour you will have a polished application, a public GitHub repository and a live URL on **Firebase Hosting** that you can put on your CV. Let's finish strong.

## What You Will Build
- Replace the users list with a proper `<UsersTable>` (TanStack Table, URL-synced pagination + search)
- Add proper loading skeletons, error UI with retry, and empty states
- Polish the create-post form (disabled-while-submitting, reset on success)
- Build for production with `npm run build` and preview locally
- Push the project to GitHub
- Deploy the built app to Firebase Hosting on a free `.web.app` URL
- Look at "what's next" in your developer journey

---

## 51.1 The Users Table — TanStack Table (15 minutes)

Replace the simple card list from Class 50 with a real admin table. We are using exactly the pattern from Class 49: client-side pagination, a debounced search box, and a company filter — all synced to the URL so the page is shareable and the back button works.

Create the file:

```tsx
// src/features/users/UsersTable.tsx
import { useMemo } from 'react'
import { Link, useSearchParams } from 'react-router-dom'
import {
  flexRender,
  getCoreRowModel,
  getFilteredRowModel,
  getPaginationRowModel,
  useReactTable,
  type ColumnDef,
} from '@tanstack/react-table'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Badge } from '@/components/ui/badge'
import {
  Table, TableBody, TableCell, TableHead, TableHeader, TableRow,
} from '@/components/ui/table'
import { Skeleton } from '@/components/ui/skeleton'
import { useUsers, type User } from './api'

const columns: ColumnDef<User>[] = [
  {
    accessorKey: 'name',
    header: 'Name',
    cell: ({ row }) => (
      <Link
        to={`/users/${row.original.id}`}
        className="font-medium text-primary hover:underline"
      >
        {row.original.name}
      </Link>
    ),
  },
  { accessorKey: 'email', header: 'Email' },
  { accessorKey: 'phone', header: 'Phone' },
  {
    id: 'company',
    accessorFn: (row) => row.company.name,
    header: 'Company',
    cell: ({ getValue }) => <Badge variant="secondary">{String(getValue())}</Badge>,
  },
  {
    id: 'city',
    accessorFn: (row) => row.address.city,
    header: 'City',
  },
]

export function UsersTable() {
  const [params, setParams] = useSearchParams()
  const search = params.get('q') ?? ''
  const page = Math.max(0, Number(params.get('page') ?? '0'))
  const { data, isLoading, isError, refetch } = useUsers()

  // Client-side filter — JSONPlaceholder gives us all users in one request
  const filtered = useMemo(() => {
    if (!data) return []
    const q = search.toLowerCase().trim()
    if (!q) return data
    return data.filter((u) =>
      [u.name, u.email, u.company.name, u.address.city]
        .join(' ').toLowerCase().includes(q),
    )
  }, [data, search])

  const table = useReactTable({
    data: filtered,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    state: { pagination: { pageIndex: page, pageSize: 5 } },
    onPaginationChange: (updater) => {
      const next = typeof updater === 'function'
        ? updater({ pageIndex: page, pageSize: 5 })
        : updater
      setParams((prev) => {
        prev.set('page', String(next.pageIndex))
        return prev
      })
    },
  })

  const updateSearch = (value: string) => {
    setParams((prev) => {
      if (value) prev.set('q', value); else prev.delete('q')
      prev.set('page', '0')
      return prev
    })
  }

  if (isError) {
    return (
      <div className="space-y-2">
        <p className="text-destructive">Failed to load users.</p>
        <Button size="sm" onClick={() => refetch()}>Retry</Button>
      </div>
    )
  }

  return (
    <div className="space-y-4">
      <div className="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-3">
        <h1 className="text-2xl font-bold">Users</h1>
        <Input
          value={search}
          onChange={(e) => updateSearch(e.target.value)}
          placeholder="Search name, email, company, city…"
          className="sm:max-w-xs"
        />
      </div>

      <div className="rounded-md border">
        <Table>
          <TableHeader>
            {table.getHeaderGroups().map((hg) => (
              <TableRow key={hg.id}>
                {hg.headers.map((h) => (
                  <TableHead key={h.id}>
                    {flexRender(h.column.columnDef.header, h.getContext())}
                  </TableHead>
                ))}
              </TableRow>
            ))}
          </TableHeader>
          <TableBody>
            {isLoading ? (
              Array.from({ length: 5 }).map((_, i) => (
                <TableRow key={i}>
                  {columns.map((_, j) => (
                    <TableCell key={j}><Skeleton className="h-4 w-full" /></TableCell>
                  ))}
                </TableRow>
              ))
            ) : table.getRowModel().rows.length === 0 ? (
              <TableRow>
                <TableCell colSpan={columns.length} className="h-24 text-center text-muted-foreground">
                  No users match "{search}".
                </TableCell>
              </TableRow>
            ) : (
              table.getRowModel().rows.map((row) => (
                <TableRow key={row.id}>
                  {row.getVisibleCells().map((cell) => (
                    <TableCell key={cell.id}>
                      {flexRender(cell.column.columnDef.cell, cell.getContext())}
                    </TableCell>
                  ))}
                </TableRow>
              ))
            )}
          </TableBody>
        </Table>
      </div>

      <div className="flex items-center justify-between">
        <p className="text-sm text-muted-foreground">
          Page {table.getState().pagination.pageIndex + 1} of {Math.max(1, table.getPageCount())}
        </p>
        <div className="flex gap-2">
          <Button
            variant="outline" size="sm"
            onClick={() => table.previousPage()}
            disabled={!table.getCanPreviousPage()}
          >
            Previous
          </Button>
          <Button
            variant="outline" size="sm"
            onClick={() => table.nextPage()}
            disabled={!table.getCanNextPage()}
          >
            Next
          </Button>
        </div>
      </div>
    </div>
  )
}
```

Now swap the route in `App.tsx`:

```tsx
// src/App.tsx — change just this line
import { UsersTable } from '@/features/users/UsersTable'
// ...
<Route index element={<RequireAuth><UsersTable /></RequireAuth>} />
```

> **Try it.** Type in the search box. Watch the URL change to `?q=fox`. Click "Next". The URL becomes `?q=fox&page=1`. Refresh — your search and page are still there. Open the link in a new tab — same result. This is the value of URL-synced state.

---

## 51.2 Polish — Skeletons, Error Retry, Empty States (10 minutes)

We had bland "Loading…" paragraphs in Class 50. Let's upgrade.

### Posts list skeletons

```tsx
// src/features/posts/PostsList.tsx — replace the loading branch
import { Skeleton } from '@/components/ui/skeleton'

// ...inside PostsList():
if (isLoading) {
  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">Posts</h1>
        <Button asChild><Link to="/posts/new">New post</Link></Button>
      </div>
      <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
        {Array.from({ length: 6 }).map((_, i) => (
          <Card key={i}>
            <CardHeader><Skeleton className="h-5 w-3/4" /></CardHeader>
            <CardContent className="space-y-2">
              <Skeleton className="h-3 w-full" />
              <Skeleton className="h-3 w-5/6" />
              <Skeleton className="h-3 w-2/3" />
            </CardContent>
          </Card>
        ))}
      </div>
    </div>
  )
}
```

### Error with retry

Add a tiny shared component once and reuse it everywhere:

```tsx
// src/components/layout/ErrorState.tsx
import { Button } from '@/components/ui/button'

interface ErrorStateProps {
  message?: string
  onRetry?: () => void
}

export function ErrorState({ message = 'Something went wrong.', onRetry }: ErrorStateProps) {
  return (
    <div className="rounded-md border border-destructive/30 bg-destructive/5 p-6 text-center space-y-3">
      <p className="text-destructive font-medium">{message}</p>
      {onRetry && <Button size="sm" variant="outline" onClick={onRetry}>Try again</Button>}
    </div>
  )
}
```

Use it in `PostsList`:

```tsx
// PostsList.tsx — pull refetch from the hook
const { data, isLoading, isError, refetch } = usePosts()
// ...
if (isError) return <ErrorState message="Failed to load posts." onRetry={() => refetch()} />
```

Do the same in `PostDetail` and `UserDetail`. The pattern is now consistent everywhere — every list and detail page has loading skeletons, error retry and an empty-state message.

---

## 51.3 Form Polish (5 minutes)

You already have `disabled={createPost.isPending}` and `form.reset()` from Class 50. Two more small touches make the form feel "finished":

1. Disable **all** inputs while submitting, not just the button.
2. Lock submit if the form has validation errors.

```tsx
// CreatePostForm.tsx — tweak the form open tag and submit button
<form
  onSubmit={form.handleSubmit(onSubmit)}
  className="space-y-4"
>
  <fieldset disabled={createPost.isPending} className="space-y-4">
    {/* ...all existing FormField components... */}
  </fieldset>

  <div className="flex gap-2">
    <Button type="submit" disabled={createPost.isPending || !form.formState.isValid}>
      {createPost.isPending ? 'Saving…' : 'Create post'}
    </Button>
    <Button type="button" variant="ghost" onClick={() => navigate(-1)}>
      Cancel
    </Button>
  </div>
</form>
```

Also add `mode: 'onChange'` to the `useForm` call so `isValid` updates as the user types:

```tsx
const form = useForm<CreatePostValues>({
  resolver: zodResolver(createPostSchema),
  defaultValues: { title: '', body: '', userId: 1 },
  mode: 'onChange',
})
```

---

## 51.4 Build for Production (5 minutes)

The `dev` server you have been using is for development only — it's slow and ships every file separately. For real users we need a **production build**.

```bash
npm run build
```

This compiles TypeScript, bundles every component into a few hashed files and writes them to a folder called `dist/`. Open `dist/` and you will see:

```
dist/
├── index.html                 # the only HTML page
├── assets/
│   ├── index-a1b2c3d4.js      # all of your TS/TSX, minified, ~200 KB gzipped
│   └── index-e5f6g7h8.css     # all of your CSS, minified
└── vite.svg
```

What happened:

1. **TypeScript was compiled to JavaScript** — browsers cannot run `.tsx` directly.
2. **All your modules were bundled** into a couple of files.
3. **The code was minified** — variable names shortened, whitespace removed.
4. **File names were hashed** so browsers know when to download a new version.

Test the production build locally before pushing to anything:

```bash
npm run preview
```

This serves `dist/` on `http://localhost:4173`. Walk through the app one more time. Sign in. Create a post. Switch theme. Open `/users` and search. If it works here, it will work on Firebase.

> **A real check before deploying:** in the production build, the React Query Devtools panel is gone, the bundle is tiny and the page paints in milliseconds. That is what your users will get.

---

## 51.5 Push to GitHub (5 minutes)

You did this in Class 27 — quick recap using the **modern GitHub CLI flow** you learned that day.

```bash
# In the dashboard/ folder
git init -b main
git add .
git commit -m "Capstone: Users & Posts dashboard"

# Create the GitHub repo AND push, in one command
gh repo create users-posts-dashboard --public --source . --remote origin --push
```

> If `gh` is not installed on the machine you are using, fall back to the traditional flow from Class 27 §27.9 Path B (create an empty repo on github.com, then `git remote add origin …` and `git push -u origin main`).

Check the repo loaded on [github.com](https://github.com) — your code is now public.

---

## 51.6 Deploy to Firebase Hosting (10 minutes)

[Firebase Hosting](https://firebase.google.com/products/hosting) is Google's free static hosting service. It serves your built `dist/` folder from a global CDN, gives you free SSL (HTTPS), and lives at a friendly URL like `https://users-posts-dashboard.web.app`.

### Step 1 — Install the Firebase CLI

Globally install the CLI once per machine:

```bash
npm install -g firebase-tools
```

Verify:

```bash
firebase --version
```

You should see something like `13.x.x`.

### Step 2 — Sign in

```bash
firebase login
```

Your browser opens. Sign in with the **same Google account** you used to create your Firebase console account in Class 1. Once you see `Success! Logged in as you@gmail.com` in the terminal, you are done.

### Step 3 — Create a Firebase project (web console)

1. Go to **https://console.firebase.google.com**.
2. Click **Add project** (or **Create a project**).
3. Name it `users-posts-dashboard` (or anything you like — this becomes part of your free URL).
4. Disable **Google Analytics** for now — we do not need it.
5. Click **Create project** and wait for it to provision.

### Step 4 — Initialise Firebase Hosting in the project

From your `dashboard/` folder:

```bash
firebase init hosting
```

You will be asked a series of questions. Answer them like this:

| Prompt | Answer |
|--------|--------|
| Please select an option | **Use an existing project** |
| Select a default Firebase project for this directory | Pick **`users-posts-dashboard`** (the one you just created) |
| What do you want to use as your public directory? | **`dist`** *(this is where Vite puts the build output)* |
| Configure as a single-page app (rewrite all URLs to /index.html)? | **Yes** *(critical — without this, `/posts/5` would 404 on refresh)* |
| Set up automatic builds and deploys with GitHub? | **No** *(we will deploy manually for the capstone; CI can be added later)* |
| File `dist/index.html` already exists. Overwrite? | **No** *(if asked — keep your built file)* |

Firebase writes two files into your project:

- `firebase.json` — hosting config (commit this)
- `.firebaserc` — links this folder to your Firebase project (commit this too)

### Step 5 — Build and deploy

```bash
npm run build
firebase deploy --only hosting
```

Firebase will:

1. Upload the contents of `dist/` to its global CDN.
2. Configure the SPA rewrite so React Router URLs work on refresh.
3. Give you a public URL like `https://users-posts-dashboard.web.app`.

The whole deploy takes about 30 seconds. Open the URL in your browser — your app is live.

### Step 6 — Test on the live URL

- Open the URL on your phone over mobile data.
- Sign in, create a post, switch theme, share the URL with a classmate.
- Make a small change to your code:
  ```bash
  git add . && git commit -m "Tweak hero text"
  git push
  npm run build
  firebase deploy --only hosting
  ```
  Firebase only redeploys what changed.

> **Want auto-deploy on every push to `main`?** Re-run `firebase init hosting` and answer **Yes** to *"Set up automatic builds and deploys with GitHub?"* — Firebase will write a GitHub Actions workflow for you. We skip this in class to keep the deploy story simple, but it is a great stretch goal.

### A note on environment variables

This capstone uses JSONPlaceholder, which is public, so **no environment variables are required**. For future projects, remember:

- All Vite env vars must start with `VITE_` to be readable in the browser.
- Example: `VITE_API_URL=https://my-backend.example.com`
- Read in code with `import.meta.env.VITE_API_URL`.
- Set them locally in `.env.local` (which is in `.gitignore`).
- For Firebase Hosting, env vars are **baked in at build time** — set them in the shell *before* running `npm run build`.

### Custom domain (optional, skip in class)

If you own a domain, Firebase Hosting can attach it for free with automatic HTTPS — **Hosting → Add custom domain** in the Firebase console. Full guide: [firebase.google.com/docs/hosting/custom-domain](https://firebase.google.com/docs/hosting/custom-domain).

### Common deployment issues

| Symptom | Likely cause | Fix |
|---|---|---|
| **Build fails on TypeScript errors** | Code has TS errors that `dev` was tolerating | Run `npx tsc --noEmit` locally and fix every error before deploying |
| **404 on refresh on `/posts/5`** | The SPA rewrite was set to "No" during `firebase init` | Edit `firebase.json` — make sure `"rewrites": [{ "source": "**", "destination": "/index.html" }]` is in the `hosting` block |
| **Blank page in production only** | Wrong asset paths from a custom `base` in `vite.config.ts` | Leave `base` as default unless deploying to a sub-path |
| **`Error: Not in a Firebase app directory`** | You ran `firebase deploy` from the wrong folder | `cd` into the folder that contains `firebase.json` |
| **API calls fail with CORS** | A custom API that does not allow the Firebase Hosting origin | Add your `.web.app` URL to the API's CORS allow-list (not relevant for JSONPlaceholder) |

---

## 51.7 What's Next — Your Developer Journey

You have finished the course. Here is the honest, ordered list of where to go next.

### 1. A real back-end → the MERN course

This course was **frontend only on purpose** — you needed to focus on React. To make your dashboard talk to a *real* database with real authentication, the natural next step is the companion course at:

```
../react-node-course/
```

It picks up where this one ends: Node.js, Express, MongoDB, JWT auth and deployment of a real REST API. The frontend skills you have transfer 100%.

### 2. Real authentication

The mock-login pattern in this capstone is fine for a portfolio, not for a product. Look into:

- **Auth.js (NextAuth)** for full sign-in flows with GitHub, Google, etc.
- **Clerk** or **Supabase Auth** for hosted auth-as-a-service.
- **JWT + refresh tokens** if you build your own (covered in the MERN course).

### 3. Testing — Vitest + Playwright

You skipped testing in this course on purpose, but you should add it next:

- **Vitest** for unit testing your hooks, schemas and utility functions.
- **React Testing Library** for component tests that mimic real user behaviour.
- **Playwright** for end-to-end tests that drive a real browser.

A reasonable goal: 80% coverage on business logic, smoke tests on the critical user flows.

### 4. Animations — Framer Motion

Want the polished feel of apps like Linear or Notion? Add [Framer Motion](https://www.framer.com/motion/) for page transitions, list reorder animations and micro-interactions. It composes beautifully with React.

### 5. Monorepos — Turborepo / pnpm workspaces

When your project grows into "frontend + backend + shared types + mobile" you'll want them in one repo. **Turborepo** + **pnpm workspaces** is the modern stack. Share your Zod schemas between the front-end and back-end so there is one source of truth for every shape.

### 6. Mobile — React Native + Expo

Once you know React, **React Native** is mostly the same thing for iOS and Android. **Expo** removes 90% of the setup pain. The components are different (`<View>` instead of `<div>`), but hooks, state, props and TypeScript are identical.

### 7. The boring-but-essential list

The things that separate junior from mid-level:

- **Accessibility** — keyboard navigation, screen readers, colour contrast
- **Performance** — code splitting with `React.lazy`, image optimisation, bundle analysis
- **Error monitoring** — Sentry or LogRocket so you know when users hit bugs
- **Analytics** — Plausible or PostHog to understand how the app is actually used
- **CI/CD** — GitHub Actions to run `npm run build` and tests on every PR
- **Internationalisation** — `react-i18next` if you ship to a non-English audience
- **PWA** — install your dashboard to a phone's home screen

You don't need them all. You need to know they exist and to pick the next one based on the project you are building.

---

## Stretch Goals (Take These Home)

1. **Add a "Delete post" button** to the post detail page using `useMutation`, an `<AlertDialog>` to confirm, and a Sonner toast on success. Invalidate `postsKeys.list()` afterwards.
2. **Add a company filter** to the users table — a `<Select>` of unique company names, also URL-synced.
3. **Add a debounced search** to the users table using `useDebounce` from Class 37 so the URL only updates after the user stops typing.
4. **Show post counts per user** on the users table by calling `/users/:id/posts` for each visible row (or a single `/posts` call and grouping client-side — which is faster?).
5. **Polish the home page** into something you would actually show off — hero, feature cards, a screenshot of the dashboard.
6. **Replace JSONPlaceholder with DummyJSON** for richer user data (images, gender, age) and update the table columns accordingly.
7. **Deploy a preview channel** — Firebase Hosting supports preview URLs out of the box. Run `firebase hosting:channel:deploy preview-users-filter` after building, and Firebase will give you a temporary public URL for that channel. This is how real teams ship features for review before promoting to production.

---

## Key Takeaways

- **TanStack Table + URL-synced state** is the production pattern for any admin list.
- **Skeletons, error retry and empty states** are the difference between a demo and a product.
- **`npm run build`** produces an optimised, minified `dist/` folder ready for any static host.
- **Firebase Hosting** gives you a free `.web.app` URL with global CDN and HTTPS. With the GitHub Actions option enabled, every push to `main` can ship to the live site automatically.
- **Environment variables in Vite must start with `VITE_`** to be readable in the browser.
- **The mock-auth + localStorage pattern** is enough for portfolios; real apps need a real auth server (next course).

---

## Course Completion

You have finished a 51-class journey that started with a single line of HTML and ends with a TypeScript React application deployed to a global CDN.

Look at what you can now do:

- Write semantic HTML5 and responsive CSS3 with Flexbox and Grid
- Write modern JavaScript and TypeScript
- Use Git, GitHub and a professional pull-request workflow
- Build React applications with Vite, hooks, context and custom hooks
- Style with Tailwind CSS and the shadcn/ui component system
- Build any form with React Hook Form and validate it with Zod
- Fetch, cache and mutate server data with Axios and React Query
- Build admin tables with pagination, search and filters using TanStack Table
- Route multi-page apps with React Router v6
- Build and deploy a production application to Firebase Hosting

You started this course as a complete beginner. You are leaving it as a **junior front-end developer** with a live portfolio piece on the internet.

A few last words from me:

- **Keep building.** Pick a small idea this week — a workout tracker, a movie watchlist, a college canteen menu app — and build it. The course gave you the tools; building real things turns them into instincts.
- **Read other people's code.** Clone open-source React projects on GitHub. Don't try to understand everything — pick one file and figure out what it does.
- **Ask better questions.** When you get stuck, write down what you tried and what error you got. Half the time the answer reveals itself in the writing.
- **Ship publicly.** Put your projects on Firebase Hosting. Share the URL. Feedback from real users teaches more than any tutorial.

It has been a privilege to teach you. Now go and build something the world has not seen yet.

**Congratulations — you have finished the course.**

— Er. Prabin Karki
