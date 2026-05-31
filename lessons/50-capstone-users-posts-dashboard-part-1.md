# Class 50: Capstone Part 1 — Users & Posts Dashboard

> **Time budget**: 1 hour (≈ 50 min build + 10 min discussion)
> **Prerequisites**: Classes 1–49 (everything you have learnt so far)

This is the moment. Over the next two classes you will combine **every single tool** from this course into one small, production-style application and then put it on the live internet. Take a breath — you are ready.

## What You Will Build

A small admin dashboard called **Users & Posts** that talks to the public JSONPlaceholder API. By the end of Class 51 it will be deployed to Vercel with a real `.vercel.app` URL you can share.

- App shell with a navbar, theme toggle and a mock user menu
- A mock login screen (RHF + Zod, stores a fake token in localStorage)
- A **Posts** module: list page, detail page, "create post" form
- A **Users** module: list page, detail page (we replace the list with a TanStack Table in Class 51)
- Loading skeletons, error UI, success toasts
- Dark mode that persists across reloads
- Deployed live to Vercel and pushed to GitHub

We will not invent a single new pattern in these two classes. Everything you write today, you have already done in an earlier class.

## What You Will Learn / Do Today
- Scaffold the full project: Vite + React + TypeScript + Tailwind + shadcn
- Set up the folder structure, Axios instance and React Router
- Build the app layout (navbar + theme toggle + mock user menu)
- Build the mock login page with RHF + Zod
- Build the Posts module end-to-end (list, detail, create)
- Wire React Query, query-key factories and Sonner toasts

---

## 50.1 The Project Brief

Imagine a junior developer's first task at a small SaaS company: "Build an admin dashboard so the support team can see all users and all posts in one place. Also let them create a post on behalf of a user."

That is our brief. We will use **JSONPlaceholder** (`https://jsonplaceholder.typicode.com`) as the placeholder back-end — it serves real-looking data for users and posts, and it accepts `POST` requests (though it pretends to save and does not actually persist). That is fine for a learning capstone.

### Feature checklist

| Feature | Tool from this course |
|---|---|
| Routing | React Router v6 (Class 48) |
| Layout & navbar | shadcn `<Card>`, `<Button>`, `<Avatar>` (Classes 40–41) |
| Dark mode | `dark:` class strategy + `useLocalStorage` (Class 42) |
| Mock login | RHF + Zod + shadcn `<Form>` (Class 44) |
| Posts list | `useQuery` (Class 45) |
| Post detail | `useQuery` with URL param (Classes 45 + 48) |
| Create post | `useMutation` + RHF + Zod + Sonner (Classes 44 + 46) |
| Users table | TanStack Table with URL-synced state (Class 49 — done in Class 51) |
| Loading / Error / Empty | Skeletons + retry buttons (Class 47) |
| Toasts | Sonner from shadcn (Class 41) |

---

## 50.2 Scaffold the Project (5 minutes)

In your projects folder, run:

```bash
npm create vite@latest dashboard -- --template react-ts
cd dashboard
npm install
```

Install the data, forms and router libraries:

```bash
npm install axios @tanstack/react-query @tanstack/react-query-devtools \
            react-router-dom react-hook-form @hookform/resolvers zod \
            @tanstack/react-table
```

Initialise Tailwind and shadcn using the preset flow you already know from Class 40:

```bash
npx shadcn@latest init
```

When prompted, pick the **New York** style and your preferred base colour (Slate works well). Now add every shadcn component we will need across both classes — getting them all in one go saves time:

```bash
npx shadcn@latest add button input card badge separator label \
                       form dialog alert-dialog sonner skeleton \
                       table avatar dropdown-menu
```

> **If you see a peer dependency warning** about React 19, accept it — shadcn supports it.

Start the dev server in a separate terminal so you can see your changes live:

```bash
npm run dev
```

---

## 50.3 Folder Structure

Create the following folders inside `src/`. We are using a **feature-based** layout — everything to do with posts lives in `features/posts`, everything to do with users lives in `features/users`. This is the same pattern professional teams use.

```
src/
├── components/
│   ├── ui/                 # shadcn components (auto-generated)
│   └── layout/
│       ├── Navbar.tsx
│       ├── Layout.tsx
│       └── ThemeToggle.tsx
├── features/
│   ├── posts/
│   │   ├── api.ts          # query hooks: usePosts, usePost, useCreatePost
│   │   ├── schema.ts       # Zod schema for the create-post form
│   │   ├── PostsList.tsx
│   │   ├── PostDetail.tsx
│   │   └── CreatePostForm.tsx
│   └── users/
│       ├── api.ts
│       ├── UsersList.tsx
│       └── UserDetail.tsx
├── hooks/
│   ├── useLocalStorage.ts  # from Class 37
│   └── useAuth.ts          # tiny mock-auth helper
├── lib/
│   ├── api.ts              # Axios instance
│   └── queryClient.ts      # React Query client config
├── pages/
│   ├── HomePage.tsx
│   ├── LoginPage.tsx
│   └── NotFoundPage.tsx
├── providers/
│   └── ThemeProvider.tsx
├── App.tsx
├── main.tsx
└── index.css
```

Create the empty folders now (`mkdir -p src/components/layout src/features/posts src/features/users src/hooks src/lib src/pages src/providers`). We will fill them in over the next sections.

---

## 50.4 The Axios Instance and React Query Client

These two files are tiny but they are the backbone of every network call in the app. You wrote both in Class 45 — they are the same pattern.

```ts
// src/lib/api.ts
import axios from 'axios'

export const api = axios.create({
  baseURL: 'https://jsonplaceholder.typicode.com',
  headers: { 'Content-Type': 'application/json' },
})
```

```ts
// src/lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query'

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,        // 1 minute — fresh enough for a dashboard
      gcTime: 5 * 60_000,       // keep cached for 5 minutes
      retry: 1,                 // retry failed requests once
      refetchOnWindowFocus: false,
    },
  },
})
```

---

## 50.5 The Theme Provider and `useLocalStorage`

You wrote both of these in Classes 37 and 42. Drop them in.

```ts
// src/hooks/useLocalStorage.ts
import { useEffect, useState } from 'react'

export function useLocalStorage<T>(key: string, initial: T) {
  const [value, setValue] = useState<T>(() => {
    try {
      const raw = localStorage.getItem(key)
      return raw ? (JSON.parse(raw) as T) : initial
    } catch {
      return initial
    }
  })

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value))
  }, [key, value])

  return [value, setValue] as const
}
```

```tsx
// src/providers/ThemeProvider.tsx
import { createContext, useContext, useEffect, type ReactNode } from 'react'
import { useLocalStorage } from '@/hooks/useLocalStorage'

type Theme = 'light' | 'dark'

interface ThemeContextValue {
  theme: Theme
  toggle: () => void
}

const ThemeContext = createContext<ThemeContextValue | undefined>(undefined)

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useLocalStorage<Theme>('theme', 'light')

  useEffect(() => {
    document.documentElement.classList.toggle('dark', theme === 'dark')
  }, [theme])

  const toggle = () => setTheme(theme === 'light' ? 'dark' : 'light')

  return (
    <ThemeContext.Provider value={{ theme, toggle }}>
      {children}
    </ThemeContext.Provider>
  )
}

export function useTheme() {
  const ctx = useContext(ThemeContext)
  if (!ctx) throw new Error('useTheme must be used inside a ThemeProvider')
  return ctx
}
```

---

## 50.6 The Mock Auth Helper

There is no real auth server in this course, so we mimic one. The user "logs in", we generate a fake token and put it in localStorage. The navbar reads it to decide whether to show the user menu.

```ts
// src/hooks/useAuth.ts
import { useLocalStorage } from './useLocalStorage'

interface AuthState {
  token: string | null
  user: { name: string; email: string } | null
}

const empty: AuthState = { token: null, user: null }

export function useAuth() {
  const [auth, setAuth] = useLocalStorage<AuthState>('auth', empty)

  const login = (name: string, email: string) => {
    setAuth({ token: `mock-${Date.now()}`, user: { name, email } })
  }

  const logout = () => setAuth(empty)

  return {
    isLoggedIn: Boolean(auth.token),
    user: auth.user,
    login,
    logout,
  }
}
```

---

## 50.7 The App Shell — Layout, Navbar, Theme Toggle

The layout wraps every page. It has a sticky navbar at the top and an `<Outlet />` underneath where the routed page renders.

```tsx
// src/components/layout/ThemeToggle.tsx
import { Button } from '@/components/ui/button'
import { useTheme } from '@/providers/ThemeProvider'

export function ThemeToggle() {
  const { theme, toggle } = useTheme()
  return (
    <Button variant="ghost" size="sm" onClick={toggle} aria-label="Toggle theme">
      {theme === 'light' ? 'Dark' : 'Light'} mode
    </Button>
  )
}
```

```tsx
// src/components/layout/Navbar.tsx
import { Link, NavLink, useNavigate } from 'react-router-dom'
import { Button } from '@/components/ui/button'
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu'
import { Avatar, AvatarFallback } from '@/components/ui/avatar'
import { ThemeToggle } from './ThemeToggle'
import { useAuth } from '@/hooks/useAuth'

export function Navbar() {
  const { isLoggedIn, user, logout } = useAuth()
  const navigate = useNavigate()

  const linkClass = ({ isActive }: { isActive: boolean }) =>
    `text-sm font-medium ${isActive ? 'text-primary' : 'text-muted-foreground hover:text-foreground'}`

  return (
    <header className="sticky top-0 z-30 border-b bg-background/80 backdrop-blur">
      <div className="mx-auto flex h-14 max-w-6xl items-center justify-between px-4">
        <Link to="/" className="text-lg font-semibold">
          Dashboard
        </Link>

        <nav className="flex items-center gap-6">
          <NavLink to="/posts" className={linkClass}>Posts</NavLink>
          <NavLink to="/users" className={linkClass}>Users</NavLink>
        </nav>

        <div className="flex items-center gap-2">
          <ThemeToggle />
          {isLoggedIn && user ? (
            <DropdownMenu>
              <DropdownMenuTrigger asChild>
                <Button variant="ghost" size="sm" className="gap-2">
                  <Avatar className="h-7 w-7">
                    <AvatarFallback>{user.name.charAt(0).toUpperCase()}</AvatarFallback>
                  </Avatar>
                  <span className="hidden sm:inline">{user.name}</span>
                </Button>
              </DropdownMenuTrigger>
              <DropdownMenuContent align="end">
                <DropdownMenuLabel>{user.email}</DropdownMenuLabel>
                <DropdownMenuSeparator />
                <DropdownMenuItem onClick={() => { logout(); navigate('/login') }}>
                  Sign out
                </DropdownMenuItem>
              </DropdownMenuContent>
            </DropdownMenu>
          ) : (
            <Button size="sm" onClick={() => navigate('/login')}>Sign in</Button>
          )}
        </div>
      </div>
    </header>
  )
}
```

```tsx
// src/components/layout/Layout.tsx
import { Outlet } from 'react-router-dom'
import { Navbar } from './Navbar'
import { Toaster } from '@/components/ui/sonner'

export function Layout() {
  return (
    <div className="min-h-screen bg-background text-foreground">
      <Navbar />
      <main className="mx-auto max-w-6xl px-4 py-8">
        <Outlet />
      </main>
      <Toaster richColors position="top-right" />
    </div>
  )
}
```

---

## 50.8 Routes and Entry Point

Wire React Router exactly as you did in Class 48.

```tsx
// src/App.tsx
import { Routes, Route, Navigate } from 'react-router-dom'
import { Layout } from '@/components/layout/Layout'
import { HomePage } from '@/pages/HomePage'
import { LoginPage } from '@/pages/LoginPage'
import { NotFoundPage } from '@/pages/NotFoundPage'
import { PostsList } from '@/features/posts/PostsList'
import { PostDetail } from '@/features/posts/PostDetail'
import { CreatePostForm } from '@/features/posts/CreatePostForm'
import { UsersList } from '@/features/users/UsersList'
import { UserDetail } from '@/features/users/UserDetail'
import { useAuth } from '@/hooks/useAuth'

function RequireAuth({ children }: { children: React.ReactElement }) {
  const { isLoggedIn } = useAuth()
  return isLoggedIn ? children : <Navigate to="/login" replace />
}

export default function App() {
  return (
    <Routes>
      <Route element={<Layout />}>
        <Route index element={<HomePage />} />
        <Route path="login" element={<LoginPage />} />

        <Route path="posts">
          <Route index element={<RequireAuth><PostsList /></RequireAuth>} />
          <Route path="new" element={<RequireAuth><CreatePostForm /></RequireAuth>} />
          <Route path=":id" element={<RequireAuth><PostDetail /></RequireAuth>} />
        </Route>

        <Route path="users">
          <Route index element={<RequireAuth><UsersList /></RequireAuth>} />
          <Route path=":id" element={<RequireAuth><UserDetail /></RequireAuth>} />
        </Route>

        <Route path="*" element={<NotFoundPage />} />
      </Route>
    </Routes>
  )
}
```

```tsx
// src/main.tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { BrowserRouter } from 'react-router-dom'
import { QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { ThemeProvider } from '@/providers/ThemeProvider'
import { queryClient } from '@/lib/queryClient'
import App from './App'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <ThemeProvider>
      <QueryClientProvider client={queryClient}>
        <BrowserRouter>
          <App />
        </BrowserRouter>
        <ReactQueryDevtools initialIsOpen={false} />
      </QueryClientProvider>
    </ThemeProvider>
  </React.StrictMode>,
)
```

> **A friendly tour through that file**: theme outside, so dark mode applies everywhere. React Query inside theme. Router inside React Query so pages can use both. The Devtools panel is invaluable while building.

---

## 50.9 The Home Page and 404

Two tiny pages so the router has something to render.

```tsx
// src/pages/HomePage.tsx
import { Link } from 'react-router-dom'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'

export function HomePage() {
  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-3xl font-bold">Users &amp; Posts Dashboard</h1>
        <p className="text-muted-foreground mt-2">
          A small admin dashboard built with everything from the course.
        </p>
      </div>

      <div className="grid gap-4 sm:grid-cols-2">
        <Card>
          <CardHeader><CardTitle>Posts</CardTitle></CardHeader>
          <CardContent className="flex justify-between">
            <p className="text-sm text-muted-foreground">Browse and create posts.</p>
            <Button asChild size="sm"><Link to="/posts">Open</Link></Button>
          </CardContent>
        </Card>
        <Card>
          <CardHeader><CardTitle>Users</CardTitle></CardHeader>
          <CardContent className="flex justify-between">
            <p className="text-sm text-muted-foreground">View all users.</p>
            <Button asChild size="sm"><Link to="/users">Open</Link></Button>
          </CardContent>
        </Card>
      </div>
    </div>
  )
}
```

```tsx
// src/pages/NotFoundPage.tsx
import { Link } from 'react-router-dom'
import { Button } from '@/components/ui/button'

export function NotFoundPage() {
  return (
    <div className="text-center py-20">
      <h1 className="text-5xl font-bold mb-2">404</h1>
      <p className="text-muted-foreground mb-6">That page does not exist.</p>
      <Button asChild><Link to="/">Back to home</Link></Button>
    </div>
  )
}
```

---

## 50.10 The Mock Login Page

This is your first chance to combine RHF, Zod and the shadcn `<Form>` primitive in this project. The "login" does no network call — it just calls `login(name, email)` from `useAuth`, which writes a fake token to localStorage.

```tsx
// src/pages/LoginPage.tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { useNavigate } from 'react-router-dom'
import { toast } from 'sonner'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import {
  Form, FormControl, FormField, FormItem, FormLabel, FormMessage,
} from '@/components/ui/form'
import { useAuth } from '@/hooks/useAuth'

const loginSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Enter a valid email'),
})

type LoginValues = z.infer<typeof loginSchema>

export function LoginPage() {
  const { login } = useAuth()
  const navigate = useNavigate()

  const form = useForm<LoginValues>({
    resolver: zodResolver(loginSchema),
    defaultValues: { name: '', email: '' },
  })

  const onSubmit = (values: LoginValues) => {
    login(values.name, values.email)
    toast.success(`Welcome, ${values.name}`)
    navigate('/posts')
  }

  return (
    <div className="max-w-sm mx-auto">
      <Card>
        <CardHeader>
          <CardTitle>Sign in (mock)</CardTitle>
        </CardHeader>
        <CardContent>
          <Form {...form}>
            <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
              <FormField
                control={form.control}
                name="name"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Name</FormLabel>
                    <FormControl><Input placeholder="Jane Smith" {...field} /></FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />
              <FormField
                control={form.control}
                name="email"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Email</FormLabel>
                    <FormControl><Input type="email" placeholder="jane@example.com" {...field} /></FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />
              <Button type="submit" className="w-full">Sign in</Button>
              <p className="text-xs text-muted-foreground text-center">
                No real account needed — this stores a mock token in localStorage.
              </p>
            </form>
          </Form>
        </CardContent>
      </Card>
    </div>
  )
}
```

---

## 50.11 The Posts Feature — Types, Schema and API Hooks

This is the heart of today's class. We co-locate the types, Zod schema and query hooks for the Posts feature in one file each, the way you did in Class 46.

```ts
// src/features/posts/schema.ts
import { z } from 'zod'

export interface Post {
  id: number
  userId: number
  title: string
  body: string
}

export const createPostSchema = z.object({
  title: z.string().min(3, 'Title must be at least 3 characters').max(120),
  body: z.string().min(10, 'Body must be at least 10 characters').max(1000),
  userId: z.coerce.number().int().min(1, 'Pick a user id between 1 and 10').max(10),
})

export type CreatePostValues = z.infer<typeof createPostSchema>
```

```ts
// src/features/posts/api.ts
import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query'
import { api } from '@/lib/api'
import type { CreatePostValues, Post } from './schema'

// Query key factory — see Class 46
export const postsKeys = {
  all: ['posts'] as const,
  list: () => [...postsKeys.all, 'list'] as const,
  detail: (id: number) => [...postsKeys.all, 'detail', id] as const,
}

export function usePosts() {
  return useQuery({
    queryKey: postsKeys.list(),
    queryFn: async () => {
      const { data } = await api.get<Post[]>('/posts')
      return data
    },
  })
}

export function usePost(id: number) {
  return useQuery({
    queryKey: postsKeys.detail(id),
    queryFn: async () => {
      const { data } = await api.get<Post>(`/posts/${id}`)
      return data
    },
    enabled: Number.isFinite(id),
  })
}

export function useCreatePost() {
  const qc = useQueryClient()
  return useMutation({
    mutationFn: async (values: CreatePostValues) => {
      const { data } = await api.post<Post>('/posts', values)
      return data
    },
    onSuccess: () => {
      // JSONPlaceholder fakes the write, but we still demonstrate cache invalidation
      qc.invalidateQueries({ queryKey: postsKeys.list() })
    },
  })
}
```

---

## 50.12 The Posts List Page

A grid of cards, each linking to the detail page. We use a simple `<p>` "Loading…" and an error message here — Class 51 polishes them into proper skeletons and a retry button.

```tsx
// src/features/posts/PostsList.tsx
import { Link } from 'react-router-dom'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { usePosts } from './api'

export function PostsList() {
  const { data, isLoading, isError } = usePosts()

  if (isLoading) return <p className="text-muted-foreground">Loading posts…</p>
  if (isError)   return <p className="text-destructive">Failed to load posts.</p>

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">Posts</h1>
        <Button asChild><Link to="/posts/new">New post</Link></Button>
      </div>

      <div className="grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
        {data?.slice(0, 24).map((post) => (
          <Link key={post.id} to={`/posts/${post.id}`} className="block">
            <Card className="h-full hover:border-primary/40 transition-colors">
              <CardHeader>
                <CardTitle className="line-clamp-2 text-base">{post.title}</CardTitle>
              </CardHeader>
              <CardContent>
                <p className="line-clamp-3 text-sm text-muted-foreground">{post.body}</p>
              </CardContent>
            </Card>
          </Link>
        ))}
      </div>
    </div>
  )
}
```

> JSONPlaceholder returns 100 posts. We `.slice(0, 24)` to keep the UI tidy. Try removing it and watch the grid grow.

---

## 50.13 The Post Detail Page

Reads `:id` from the URL with `useParams`, fetches via `usePost`, renders the post.

```tsx
// src/features/posts/PostDetail.tsx
import { Link, useParams } from 'react-router-dom'
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { usePost } from './api'

export function PostDetail() {
  const { id } = useParams<{ id: string }>()
  const postId = Number(id)
  const { data, isLoading, isError } = usePost(postId)

  if (isLoading) return <p className="text-muted-foreground">Loading post…</p>
  if (isError || !data) return <p className="text-destructive">Post not found.</p>

  return (
    <div className="max-w-2xl space-y-4">
      <Button asChild variant="ghost" size="sm">
        <Link to="/posts">← Back to posts</Link>
      </Button>

      <Card>
        <CardHeader className="space-y-2">
          <Badge variant="secondary">Post #{data.id}</Badge>
          <CardTitle className="text-xl">{data.title}</CardTitle>
        </CardHeader>
        <CardContent className="space-y-2">
          <p className="text-sm text-muted-foreground">By user {data.userId}</p>
          <p className="leading-relaxed">{data.body}</p>
        </CardContent>
      </Card>
    </div>
  )
}
```

---

## 50.14 The Create-Post Form

This is the single biggest payoff page of the entire course — every layer you have learnt is doing its job here at once:

- **shadcn `<Form>`** for layout and accessibility
- **React Hook Form** for state and submission
- **Zod** for validation
- **React Query mutation** for the network call
- **Sonner** for the success toast
- **React Router** to redirect to the new post

```tsx
// src/features/posts/CreatePostForm.tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { useNavigate } from 'react-router-dom'
import { toast } from 'sonner'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import {
  Form, FormControl, FormField, FormItem, FormLabel, FormMessage,
} from '@/components/ui/form'
import { createPostSchema, type CreatePostValues } from './schema'
import { useCreatePost } from './api'

export function CreatePostForm() {
  const navigate = useNavigate()
  const createPost = useCreatePost()

  const form = useForm<CreatePostValues>({
    resolver: zodResolver(createPostSchema),
    defaultValues: { title: '', body: '', userId: 1 },
  })

  const onSubmit = (values: CreatePostValues) => {
    createPost.mutate(values, {
      onSuccess: (post) => {
        toast.success('Post created')
        form.reset()
        navigate(`/posts/${post.id}`)
      },
      onError: () => toast.error('Could not create the post'),
    })
  }

  return (
    <div className="max-w-xl">
      <Card>
        <CardHeader><CardTitle>New post</CardTitle></CardHeader>
        <CardContent>
          <Form {...form}>
            <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
              <FormField
                control={form.control}
                name="title"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Title</FormLabel>
                    <FormControl><Input placeholder="A clear, short title" {...field} /></FormControl>
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
                    <FormControl>
                      <textarea
                        {...field}
                        rows={6}
                        className="flex w-full rounded-md border border-input bg-background px-3 py-2 text-sm"
                        placeholder="Write the post content here…"
                      />
                    </FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />
              <FormField
                control={form.control}
                name="userId"
                render={({ field }) => (
                  <FormItem>
                    <FormLabel>Author user id (1–10)</FormLabel>
                    <FormControl><Input type="number" min={1} max={10} {...field} /></FormControl>
                    <FormMessage />
                  </FormItem>
                )}
              />

              <div className="flex gap-2">
                <Button type="submit" disabled={createPost.isPending}>
                  {createPost.isPending ? 'Saving…' : 'Create post'}
                </Button>
                <Button type="button" variant="ghost" onClick={() => navigate(-1)}>
                  Cancel
                </Button>
              </div>
            </form>
          </Form>
        </CardContent>
      </Card>
    </div>
  )
}
```

> **JSONPlaceholder will respond with a brand-new id (101)** and pretend it saved — that's expected. The point is the *full pipeline* (form → schema → mutation → toast → redirect) runs end-to-end.

---

## 50.15 A Minimal Users List and Detail (for now)

We'll replace the list with a real TanStack Table in Class 51, but we need *something* at `/users` so the navbar works.

```ts
// src/features/users/api.ts
import { useQuery } from '@tanstack/react-query'
import { api } from '@/lib/api'

export interface User {
  id: number
  name: string
  username: string
  email: string
  phone: string
  website: string
  company: { name: string }
  address: { city: string }
}

export const usersKeys = {
  all: ['users'] as const,
  list: () => [...usersKeys.all, 'list'] as const,
  detail: (id: number) => [...usersKeys.all, 'detail', id] as const,
}

export function useUsers() {
  return useQuery({
    queryKey: usersKeys.list(),
    queryFn: async () => (await api.get<User[]>('/users')).data,
  })
}

export function useUser(id: number) {
  return useQuery({
    queryKey: usersKeys.detail(id),
    queryFn: async () => (await api.get<User>(`/users/${id}`)).data,
    enabled: Number.isFinite(id),
  })
}
```

```tsx
// src/features/users/UsersList.tsx
import { Link } from 'react-router-dom'
import { Card, CardContent } from '@/components/ui/card'
import { useUsers } from './api'

export function UsersList() {
  const { data, isLoading, isError } = useUsers()

  if (isLoading) return <p className="text-muted-foreground">Loading users…</p>
  if (isError) return <p className="text-destructive">Failed to load users.</p>

  return (
    <div className="space-y-4">
      <h1 className="text-2xl font-bold">Users</h1>
      <p className="text-sm text-muted-foreground">
        A real admin table replaces this list in the next class.
      </p>
      <div className="grid gap-3 sm:grid-cols-2">
        {data?.map((u) => (
          <Link key={u.id} to={`/users/${u.id}`}>
            <Card className="hover:border-primary/40 transition-colors">
              <CardContent className="py-4">
                <p className="font-medium">{u.name}</p>
                <p className="text-sm text-muted-foreground">{u.email}</p>
              </CardContent>
            </Card>
          </Link>
        ))}
      </div>
    </div>
  )
}
```

```tsx
// src/features/users/UserDetail.tsx
import { Link, useParams } from 'react-router-dom'
import { Button } from '@/components/ui/button'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Separator } from '@/components/ui/separator'
import { useUser } from './api'

export function UserDetail() {
  const { id } = useParams<{ id: string }>()
  const { data, isLoading, isError } = useUser(Number(id))

  if (isLoading) return <p className="text-muted-foreground">Loading user…</p>
  if (isError || !data) return <p className="text-destructive">User not found.</p>

  return (
    <div className="max-w-xl space-y-4">
      <Button asChild variant="ghost" size="sm">
        <Link to="/users">← Back to users</Link>
      </Button>
      <Card>
        <CardHeader>
          <CardTitle>{data.name}</CardTitle>
          <p className="text-sm text-muted-foreground">@{data.username}</p>
        </CardHeader>
        <CardContent className="space-y-2 text-sm">
          <p><strong>Email:</strong> {data.email}</p>
          <p><strong>Phone:</strong> {data.phone}</p>
          <p><strong>Website:</strong> {data.website}</p>
          <Separator className="my-2" />
          <p><strong>Company:</strong> {data.company.name}</p>
          <p><strong>City:</strong> {data.address.city}</p>
        </CardContent>
      </Card>
    </div>
  )
}
```

---

## 50.16 Run It

```bash
npm run dev
```

You should be able to:

1. Open `http://localhost:5173/` and see the home cards.
2. Click **Sign in**, complete the form, get a toast and land on `/posts`.
3. Browse 24 post cards, click one, read the detail.
4. Click **New post**, submit the form, get a toast, land on the new post.
5. Visit `/users` and see a card per user.
6. Click the **Dark mode** button — the whole UI flips. Reload the page — it stays dark.

If any of these don't work, open the React Query Devtools panel (bottom-left flower icon) and check the query states.

---

## Stretch Goals (Homework before Class 51)

These are optional but recommended — they make Class 51 smoother.

1. **Style the home page** with your own colour accents and a hero section.
2. **Cap the post body at 280 characters** in the list cards using a Tailwind `line-clamp`.
3. **Show the post author's name** on the detail page by also calling `useUser(post.userId)` — this is your first taste of a "dependent query".
4. **Refactor the empty/loading paragraphs** into shared `<LoadingMessage />` and `<ErrorMessage />` components — we'll formalise these into skeletons in Class 51.

---

## Key Takeaways

- A real app is not a single new idea — it is many old ideas **standing on each other's shoulders**.
- A **feature-based folder structure** (`features/posts`, `features/users`) scales better than grouping by file type.
- **Query-key factories** make cache invalidation predictable and refactor-safe.
- The **shadcn `<Form>` + RHF + Zod** trio is the production pattern for any non-trivial form.
- A mock auth layer + `useLocalStorage` is enough to demonstrate protected routes without a back-end.
- You now have a runnable, end-to-end React application. In Class 51 we polish it and put it on the internet.
