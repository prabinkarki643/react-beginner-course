# Class 38: Context API for Global State

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 36 (lifting state), Class 37 (custom hooks)

## What You Will Learn
- The **prop drilling** problem and why it is painful in deep trees
- How `createContext`, `<Provider>` and `useContext` solve it
- Typing context properly with `createContext<T | null>(null)`
- Building a custom typed hook (`useTheme`) that throws if used outside the provider
- A complete typed `ThemeContext` (light/dark) wired up to `useLocalStorage`
- Re-render concerns and when Context is the wrong tool (server state → React Query later)

---

## 38.1 The Prop Drilling Problem

By Class 36 we learned to lift state to the **closest common ancestor** when siblings share data. That works beautifully for small trees. But what about a value that nearly every component needs — say, the **current theme**?

```
App (theme state)
 ├── Header
 │    ├── Logo
 │    └── ThemeToggle      ← needs theme
 ├── Sidebar
 │    └── NavLinks
 │         └── NavItem     ← needs theme
 └── Main
      └── Content
           └── Article
                └── Footnote ← needs theme
```

To get `theme` to every deep leaf, every component in between must accept and forward the prop:

```tsx
<App theme={theme}>
  <Header theme={theme}>
    <ThemeToggle theme={theme} setTheme={setTheme} />
  </Header>
  <Main theme={theme}>
    <Content theme={theme}>
      <Article theme={theme}>
        <Footnote theme={theme} />
      </Article>
    </Content>
  </Main>
</App>
```

`Sidebar` and `Content` do not care about the theme but are forced to carry it. This is **prop drilling** — and it is tedious, error-prone, and makes refactoring painful.

> If only a couple of leaves need the value, lift to the nearest ancestor. If many components scattered across the tree need it, reach for **Context**.

---

## 38.2 How Context Works — The Radio Analogy

Context is like a radio broadcast:

1. **Create** a radio frequency: `createContext<T | null>(null)`.
2. **Broadcast** a signal: wrap part of the tree in a `<Provider value={...}>`.
3. **Tune in** from any component inside that part: call `useContext(MyContext)`.

```
Provider (broadcasts the theme value)
   │
   ├─ Header
   │   └─ ThemeToggle ── useContext → reads theme directly
   ├─ Sidebar
   │   └─ NavLinks
   └─ Main
       └─ Article
           └─ Footnote ── useContext → reads theme directly
```

No intermediate component has to know or care.

---

## 38.3 A Minimal Context Example

```tsx
// src/context/CounterContext.tsx
import { createContext, useContext, useState } from 'react';

interface CounterContextValue {
  count: number;
  increment: () => void;
  reset: () => void;
}

const CounterContext = createContext<CounterContextValue | null>(null);

export function CounterProvider({ children }: { children: React.ReactNode }) {
  const [count, setCount] = useState<number>(0);
  const value: CounterContextValue = {
    count,
    increment: () => setCount((c) => c + 1),
    reset: () => setCount(0),
  };
  return <CounterContext.Provider value={value}>{children}</CounterContext.Provider>;
}

export function useCounter(): CounterContextValue {
  const ctx = useContext(CounterContext);
  if (ctx === null) {
    throw new Error('useCounter must be used inside a <CounterProvider>');
  }
  return ctx;
}
```

That one file contains the three things you need every time you build a context:

1. **The context value type** (`CounterContextValue`).
2. **The provider** (`CounterProvider`) that owns the state and broadcasts a value.
3. **A custom hook** (`useCounter`) that wraps `useContext` and throws if called in the wrong place.

### Why `T | null` and a custom hook?

`createContext` needs a **default value**. If a component reads the context without a provider above it, that default is returned. Using `null` and checking for it inside a custom hook gives us two big wins:

- The error message is **explicit**: "useCounter must be used inside a `<CounterProvider>`" — far better than a mysterious `undefined.count`.
- After the `if (ctx === null) throw ...` check, TypeScript **narrows** the type from `CounterContextValue | null` down to `CounterContextValue`. Every caller gets the real type with no extra checks.

### Using it

```tsx
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import { CounterProvider } from './context/CounterContext';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <CounterProvider>
      <App />
    </CounterProvider>
  </React.StrictMode>
);
```

```tsx
// src/CountDisplay.tsx
import { useCounter } from './context/CounterContext';

function CountDisplay() {
  const { count } = useCounter();
  return <p>The count is {count}.</p>;
}

export default CountDisplay;
```

```tsx
// src/CountButtons.tsx
import { useCounter } from './context/CounterContext';

function CountButtons() {
  const { increment, reset } = useCounter();
  return (
    <div style={{ display: 'flex', gap: '0.5rem' }}>
      <button onClick={increment}>+1</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

export default CountButtons;
```

```tsx
// src/App.tsx
import CountDisplay from './CountDisplay';
import CountButtons from './CountButtons';

function App() {
  return (
    <div style={{ padding: '2rem', fontFamily: 'system-ui' }}>
      <h1>Counter via Context</h1>
      <CountDisplay />
      <CountButtons />
    </div>
  );
}

export default App;
```

`CountDisplay` and `CountButtons` are siblings. Neither receives the count as a prop. Both read it directly from context. Adding a `<CountSummary />` ten levels deep tomorrow would Just Work.

---

## 38.4 The Full Example — A Typed `ThemeContext`

Let us build a real, typed theme context that persists across page refreshes by re-using the `useLocalStorage` hook from Class 37.

### Step 1 — the hook (recap)

```tsx
// src/hooks/useLocalStorage.ts
import { useState, useEffect } from 'react';

export function useLocalStorage<T>(key: string, initialValue: T): [T, (next: T) => void] {
  const [value, setValue] = useState<T>(() => {
    try {
      const raw = window.localStorage.getItem(key);
      return raw !== null ? (JSON.parse(raw) as T) : initialValue;
    } catch {
      return initialValue;
    }
  });

  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch {
      /* ignore */
    }
  }, [key, value]);

  return [value, setValue];
}
```

### Step 2 — the theme context

```tsx
// src/context/ThemeContext.tsx
import { createContext, useContext } from 'react';
import { useLocalStorage } from '../hooks/useLocalStorage';

export type Theme = 'light' | 'dark';

interface ThemeContextValue {
  theme: Theme;
  setTheme: (next: Theme) => void;
  toggle: () => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);

interface ThemeProviderProps {
  children: React.ReactNode;
}

export function ThemeProvider({ children }: ThemeProviderProps) {
  const [theme, setTheme] = useLocalStorage<Theme>('theme', 'light');

  const toggle = () => setTheme(theme === 'light' ? 'dark' : 'light');

  const value: ThemeContextValue = { theme, setTheme, toggle };

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

export function useTheme(): ThemeContextValue {
  const ctx = useContext(ThemeContext);
  if (ctx === null) {
    throw new Error('useTheme must be used inside a <ThemeProvider>');
  }
  return ctx;
}
```

### Step 3 — components that consume it

```tsx
// src/ThemeToggle.tsx
import { useTheme } from './context/ThemeContext';

function ThemeToggle() {
  const { theme, toggle } = useTheme();
  return (
    <button onClick={toggle}>
      Switch to {theme === 'light' ? 'dark' : 'light'} mode
    </button>
  );
}

export default ThemeToggle;
```

```tsx
// src/ThemedPage.tsx
import { useTheme } from './context/ThemeContext';
import ThemeToggle from './ThemeToggle';

function ThemedPage() {
  const { theme } = useTheme();

  const palette = theme === 'light'
    ? { bg: '#fff', fg: '#111', border: '#ddd' }
    : { bg: '#111', fg: '#eee', border: '#333' };

  return (
    <div
      style={{
        minHeight: '100vh',
        background: palette.bg,
        color: palette.fg,
        padding: '2rem',
        fontFamily: 'system-ui',
      }}
    >
      <header
        style={{
          padding: '0.75rem 1rem',
          border: `1px solid ${palette.border}`,
          borderRadius: '8px',
          marginBottom: '1.5rem',
          display: 'flex',
          justifyContent: 'space-between',
          alignItems: 'center',
        }}
      >
        <strong>My themed app</strong>
        <ThemeToggle />
      </header>
      <main>
        <h1>Welcome</h1>
        <p>Current theme: {theme}</p>
        <p>Refresh the page — the theme is remembered.</p>
      </main>
    </div>
  );
}

export default ThemedPage;
```

### Step 4 — wire it all in `main.tsx`

```tsx
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { ThemeProvider } from './context/ThemeContext';
import ThemedPage from './ThemedPage';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <ThemeProvider>
      <ThemedPage />
    </ThemeProvider>
  </React.StrictMode>
);
```

Click the toggle. The theme switches. Refresh the page — it stays. Any component anywhere inside `<ThemeProvider>` can call `useTheme()` and just work.

### Why this pattern works so well

- **One source of truth.** `theme` lives in `ThemeProvider`. Every consumer reads from the same place.
- **No prop drilling.** No intermediate component knows the theme exists.
- **Typed strictly.** `Theme = 'light' | 'dark'` means you cannot accidentally set `'darrk'`.
- **Persisted automatically.** Because we use `useLocalStorage`, the theme survives reloads.
- **Safe by construction.** `useTheme()` throws if used outside the provider, so misuse is caught immediately in development.

---

## 38.5 Multiple Contexts

You can have as many contexts as you want, each providing a different value. A typical real app might have:

- `ThemeContext` — light/dark mode
- `AuthContext` — the current user
- `LocaleContext` — language preferences
- `ToastContext` — a global toast queue

Each is its own file, its own provider, its own typed hook. Compose them in `main.tsx`:

```tsx
<ThemeProvider>
  <AuthProvider>
    <LocaleProvider>
      <App />
    </LocaleProvider>
  </AuthProvider>
</ThemeProvider>
```

Order matters only if an inner provider reads from an outer one.

---

## 38.6 Re-render Concerns — When Context Is the Wrong Tool

Context is wonderful, but it has one trap worth knowing about now and budgeting for later.

**Every consumer re-renders whenever the provider's value changes.** If the value is an object literal computed in the provider, even a tiny change can trigger a re-render in every component that calls `useContext` — even those that only read one field.

```tsx
const value = { theme, setTheme, toggle };   // new object every render
```

For a value that changes rarely (like the theme) this is fine. For values that change many times per second (mouse position, scroll position) or for **server data** (lists fetched from an API) it becomes a performance problem.

**Three rules to remember:**

1. **Use context for low-frequency, app-wide UI state**: theme, locale, current user, modal open/close.
2. **Do not use context for server data**. Lists of posts, users, products and so on should live in **TanStack Query** — we cover this from Class 45. React Query was built to solve exactly this problem (caching, deduplication, refetching) without abusing Context.
3. **If a context value changes often**, split it into two contexts (one for state, one for the setter), or memoise the object — we will look at `useMemo` properly later. For our beginner course, the rule of thumb above is enough.

---

## 38.7 Putting It Together — A Quick Recap

To build a typed context the right way:

1. Define a value type: `interface FooContextValue { ... }`.
2. Create the context: `createContext<FooContextValue | null>(null)`.
3. Build the provider: own the state inside, broadcast it with `<Foo.Provider value={...}>`.
4. Build the typed hook: `useFoo()` that calls `useContext`, throws on `null`, returns the value.
5. Wrap the part of the tree that should have access to the value in the provider.
6. Anywhere inside, call `useFoo()` to read or change the value.

That recipe is the same for every context you will ever build. Memorise it.

---

## Practice Exercises

1. **Locale context.** Build a `LocaleContext` that holds `locale: 'en-GB' | 'ne-NP' | 'hi-IN'` and a `setLocale` function. Wire it up with a `LocaleProvider` and a custom `useLocale` hook (with the null-check). Build a `LanguageSwitcher` button that cycles through the three options and a `Greeting` component that shows "Hello", "नमस्ते" or "नमस्कार" based on the current locale.
2. **Auth-like context (mocked).** Build an `AuthContext` with `user: { id: number; name: string } | null` and two methods `login(name: string)` and `logout()`. Render a `<Navbar />` that shows "Log in" or "Hi, {name} (log out)" depending on whether a user is logged in. No real backend — `login` just sets a fake user object.
3. **Theme context + your custom hook.** Repeat section 38.4 from memory. Then add a third theme `'high-contrast'`. Update the type, the toggle (cycle through all three), and the palette in `ThemedPage`. Confirm the typed union prevents typos.
4. **Provider misuse.** In one of your contexts, deliberately render a consumer **outside** the provider. Confirm that the error from your custom hook fires (`useFoo must be used inside a <FooProvider>`). Then wrap it correctly and confirm it works.

---

## Key Takeaways
- **Context** solves prop drilling: a value broadcast from a provider can be read anywhere below it.
- The pattern is always the same: a value type, a provider that owns the state, a typed `useFoo()` hook with a null-check.
- Type the context as `T | null` and throw inside the hook so TypeScript narrows correctly and misuse is loud.
- Context is great for **app-wide UI state**: theme, locale, current user. It is the wrong tool for high-frequency state or server data.
- For server state (lists fetched from an API) we will use **TanStack Query** from Class 45 — caching, deduplication and refetching come for free.
- That wraps up React Fundamentals. Next we move to **Tailwind CSS** in Class 39 and start styling our apps properly.
