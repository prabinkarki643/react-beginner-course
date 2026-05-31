# Class 37: Custom Hooks

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 33 (useState), Class 35 (useEffect)

## What You Will Learn
- What a custom hook is and why we extract logic into one
- The `use*` naming convention and the **rules of hooks**
- Building `useToggle()` — your first custom hook
- Building `useLocalStorage<T>(key, initial)` — typed with generics
- Building `useDebounce<T>(value, delay)` — useful for search boxes
- Using the same hook in two different components and watching it just work

---

## 37.1 What is a Custom Hook?

A **custom hook** is a function whose name starts with `use` and which can call other React hooks (`useState`, `useEffect`, etc.) inside it.

That is the entire definition. There is no special API — a custom hook is just a function. What makes it a hook is the **convention** that it uses React hooks internally and follows the rules of hooks.

### Why do we want them?

Consider a component that holds a boolean and a function to flip it:

```tsx
const [open, setOpen] = useState<boolean>(false);
const toggle = () => setOpen((o) => !o);
```

Now imagine four different components in your app that each need a toggle (a modal, a sidebar, a dropdown, a hint). You would duplicate that snippet four times.

A custom hook lets you write it **once** and reuse it everywhere:

```tsx
const [open, toggle] = useToggle(false);
```

The same idea applies to anything stateful: a counter, a form field, a fetch result, a value synced with `localStorage`, a debounced value. Whenever you find yourself copying state-plus-effect logic between components, that is the moment to extract a hook.

---

## 37.2 The Rules of Hooks

There are two rules that apply to **all** hooks — built-in and custom:

1. **Only call hooks at the top level.** Never inside loops, conditions, or nested functions. The first call must be a hook, the second call must be a hook, and so on. React relies on the **order** of hook calls to remember which value is which.
2. **Only call hooks from React functions.** From components, or from other custom hooks. Never from regular utility functions, event handlers, or class components.

The `use` prefix is not just a style choice — it tells the React linter ("react-hooks/rules-of-hooks") that this function plays by the rules. Always name your custom hooks `useSomething`.

---

## 37.3 Your First Custom Hook — `useToggle`

```tsx
// src/hooks/useToggle.ts
import { useState, useCallback } from 'react';

export function useToggle(initial: boolean = false): [boolean, () => void] {
  const [value, setValue] = useState<boolean>(initial);
  const toggle = useCallback(() => setValue((v) => !v), []);
  return [value, toggle];
}
```

That is it. A custom hook is a function that calls other hooks and returns whatever the consumer needs.

We use `useCallback` so that `toggle` keeps the same identity between renders. This is not strictly necessary here, but it is good practice when returning functions from hooks — it prevents unnecessary re-renders in children that receive the function as a prop.

### Using it

```tsx
// src/Sidebar.tsx
import { useToggle } from './hooks/useToggle';

function Sidebar() {
  const [open, toggle] = useToggle(false);

  return (
    <div>
      <button onClick={toggle}>{open ? 'Hide' : 'Show'} sidebar</button>
      {open && (
        <aside style={{ marginTop: '1rem', padding: '1rem', border: '1px solid #ddd' }}>
          <p>I am the sidebar content.</p>
        </aside>
      )}
    </div>
  );
}

export default Sidebar;
```

Two lines and the sidebar has its toggle behaviour. Use the same hook in a modal, a dropdown, an accordion — anywhere you need a boolean that flips.

### Notes on the type signature

```tsx
function useToggle(initial: boolean = false): [boolean, () => void]
```

The return type is a **tuple** — exactly two items: the current value and the toggle function. This mirrors the shape of `useState` and makes destructuring at the call site read naturally.

---

## 37.4 Generic Hooks — `useLocalStorage<T>`

Sometimes a hook needs to work with **any** type. Generics make that possible without losing type safety.

We want a hook that behaves like `useState`, but persists the value to `localStorage` so it survives page reloads.

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
      // ignore quota errors
    }
  }, [key, value]);

  return [value, setValue];
}
```

Things to notice:

- `<T>` is a **generic type parameter**. The caller decides what `T` is when they call the hook — `string`, `number`, an array, an object, anything serialisable.
- `useState(() => ...)` is the **lazy initialiser** form. The function only runs on the first render, which keeps the `localStorage.getItem` call out of subsequent re-renders.
- The `useEffect` writes the value back to storage whenever it changes. Whenever you set a new value, storage updates automatically.
- We wrap both `getItem` and `setItem` in `try/catch` — `localStorage` can throw on quota errors, in private windows, etc.

### Using it

```tsx
// src/UsernameForm.tsx
import { useLocalStorage } from './hooks/useLocalStorage';

function UsernameForm() {
  const [name, setName] = useLocalStorage<string>('username', '');

  return (
    <div style={{ padding: '1rem' }}>
      <label>
        Your name&nbsp;
        <input value={name} onChange={(e) => setName(e.target.value)} />
      </label>
      <p>Hello, {name || '(stranger)'}!</p>
      <p style={{ color: '#666', fontSize: '0.85rem' }}>
        Refresh the page — your name will still be here.
      </p>
    </div>
  );
}

export default UsernameForm;
```

Type a name, refresh the browser, and the value is still there. Open the browser DevTools → Application → Local Storage and you can see the key.

### Using the SAME hook in two components

Let us build a tiny example where two unrelated components both read and write the same key. Whatever happens in one, the other sees on the next render.

```tsx
// src/ThemePicker.tsx
import { useLocalStorage } from './hooks/useLocalStorage';

function ThemePicker() {
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');

  return (
    <div style={{ padding: '1rem' }}>
      <p>Current theme: <strong>{theme}</strong></p>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Switch theme
      </button>
    </div>
  );
}

export default ThemePicker;
```

```tsx
// src/ThemeBadge.tsx
import { useLocalStorage } from './hooks/useLocalStorage';

function ThemeBadge() {
  const [theme] = useLocalStorage<'light' | 'dark'>('theme', 'light');

  return (
    <span
      style={{
        padding: '0.25rem 0.5rem',
        borderRadius: '999px',
        background: theme === 'light' ? '#fffae6' : '#222',
        color: theme === 'light' ? '#333' : '#fff',
        fontSize: '0.85rem',
      }}
    >
      Theme is {theme}
    </span>
  );
}

export default ThemeBadge;
```

```tsx
// src/App.tsx
import ThemePicker from './ThemePicker';
import ThemeBadge from './ThemeBadge';

function App() {
  return (
    <div style={{ padding: '2rem', fontFamily: 'system-ui' }}>
      <ThemeBadge />
      <ThemePicker />
    </div>
  );
}

export default App;
```

Two completely separate components, both using `useLocalStorage('theme', ...)`. They stay in sync through `localStorage` (and a refresh).

> **Caveat**: two `useLocalStorage` calls in different components do not share state in memory automatically — they each hold their own React state and write to the same storage key. They sync on **refresh**. For live syncing across components without prop drilling, use Context (Class 38) or wire up a `storage` event listener. Plain `useLocalStorage` is enough for most beginner use cases.

---

## 37.5 `useDebounce<T>` — A Hook with an Effect

A **debounced** value is one that only updates after the input has been still for a while. It is the right tool for search boxes that hit an API on every keystroke — you do not actually want to fire a request for every letter.

```tsx
// src/hooks/useDebounce.ts
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number = 300): T {
  const [debounced, setDebounced] = useState<T>(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);   // cancel the previous timer on every change
  }, [value, delay]);

  return debounced;
}
```

The magic is in the cleanup function. Each time `value` changes, the previous `setTimeout` is cancelled and a new one is scheduled. The debounced value only updates when `value` has been stable for `delay` milliseconds.

### Using it in a search box

```tsx
// src/UserSearch.tsx
import { useEffect, useState } from 'react';
import { useDebounce } from './hooks/useDebounce';

interface ApiUser {
  id: number;
  name: string;
  email: string;
}

function UserSearch() {
  const [query, setQuery] = useState<string>('');
  const debounced = useDebounce(query, 400);
  const [results, setResults] = useState<ApiUser[]>([]);

  useEffect(() => {
    const controller = new AbortController();

    async function load() {
      if (debounced.trim() === '') {
        setResults([]);
        return;
      }
      const res = await fetch(
        `https://jsonplaceholder.typicode.com/users?q=${encodeURIComponent(debounced)}`,
        { signal: controller.signal }
      );
      if (!res.ok) return;
      const data: ApiUser[] = await res.json();
      const lower = debounced.toLowerCase();
      setResults(
        data.filter((u) => u.name.toLowerCase().includes(lower) || u.email.toLowerCase().includes(lower))
      );
    }

    load();
    return () => controller.abort();
  }, [debounced]);

  return (
    <div style={{ padding: '1rem', maxWidth: 480, fontFamily: 'system-ui' }}>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search users…"
        style={{ width: '100%', padding: '0.5rem' }}
      />
      <p style={{ color: '#666', fontSize: '0.85rem' }}>
        Typing: <code>{query}</code> &nbsp;·&nbsp; Searching for: <code>{debounced}</code>
      </p>
      <ul style={{ listStyle: 'none', padding: 0 }}>
        {results.map((u) => (
          <li key={u.id} style={{ padding: '0.5rem 0', borderBottom: '1px solid #eee' }}>
            <strong>{u.name}</strong> — {u.email}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default UserSearch;
```

Type fast: `query` updates on every key, but `debounced` only catches up 400ms after you stop. The effect that fires the fetch depends on `debounced`, so the network is only hit when typing pauses. This pattern is the basis of every well-behaved search box on the web.

> JSONPlaceholder does not actually filter on the server, so we also filter on the client. The point is the debouncing pattern; real APIs would do the filtering for you.

---

## 37.6 Where to Put Hooks in the Project

A common convention:

```
src/
├── hooks/
│   ├── useToggle.ts
│   ├── useLocalStorage.ts
│   └── useDebounce.ts
├── components/
│   └── ...
└── App.tsx
```

One hook per file, each `export`-ed by name. If a hook is only used by one component, it can live next to that component — but most hooks are general enough to deserve their own home.

---

## 37.7 A Mental Checklist for a Good Custom Hook

Before you extract a hook, ask yourself:

- Does the logic involve **state** or an **effect**? If neither, it is probably just a plain function — keep it as one.
- Will it be **reused** in two or more places? Or does it isolate complicated logic that clutters a component? Either is a fine reason to extract.
- Does the **name** describe what it gives you? `useToggle` (returns a toggle), `useLocalStorage` (returns a stored value), `useDebounce` (returns a debounced value). Avoid vague names like `useHelper`.
- Are the inputs and outputs **typed** clearly, using generics where the type can vary?

---

## Practice Exercises

1. **`useCounter`.** Build a `useCounter(initial: number = 0)` hook that returns `{ count, increment, decrement, reset }`. Type the return as an interface. Use it in two different components in the same page; confirm each has its own count.
2. **`usePrevious<T>`.** Build a generic hook `usePrevious<T>(value: T): T | undefined` that returns the value from the previous render. Hint: store the value in a `useRef`, update it in a `useEffect`. Use it in a counter to display "Now: 3, before: 2".
3. **`useLocalStorage` for an object.** Use `useLocalStorage<UserPrefs>('prefs', { theme: 'light', fontSize: 14 })` where `UserPrefs` is an interface. Build a small "Settings" page that edits the object and saves it. Reload to confirm persistence.
4. **Debounced live search.** Take the `UserSearch` example from 37.5 and add a second component on the same page, `<SearchHeader />`, that reads the same `query` (lifted up to `App`) and shows the current search count. The hook stays general — the components share state via the parent.

---

## Key Takeaways
- A custom hook is a function whose name starts with `use` and which calls React hooks inside it.
- Hooks must be called at the top level of a component or another hook — no loops, no conditions.
- Extract a hook when the same state-plus-effect logic appears in more than one component, or when it clutters a single component.
- Use **generics** (`useLocalStorage<T>`, `useDebounce<T>`) to keep your hooks type-safe across many value types.
- The same hook used in two components keeps **its own state** in each — they do not magically share. For shared state, lift up (Class 36) or use Context (Class 38).
- Next class: **Context API** — sharing values across the whole tree without prop drilling.
