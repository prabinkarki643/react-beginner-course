# Class 42: Theming, Dark Mode & Customisation

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 37 (`useLocalStorage`), Class 38 (Context API), Class 40 (shadcn init), Class 41 (Sonner mounted in `App.tsx`)

## What You Will Learn
- How shadcn/ui's theme is **driven by CSS variables** in `src/index.css`
- The **`dark` class strategy** — toggling one class on `<html>` to swap the entire palette
- Building a typed **`<ThemeProvider>`** that persists the choice via `useLocalStorage`
- A **`<ThemeToggle>`** button using `<Button variant="ghost">` + a Lucide icon
- Customising the colour palette by editing CSS variables (with a quick **hex → HSL** note)
- A short tour of `tailwind.config.ts` and the official **shadcn theme generator**

---

## 42.1 How shadcn theming actually works

Open `src/index.css` after running `npx shadcn@latest init` (from Class 40). You will see something close to this:

```css
/* src/index.css */
@import "tailwindcss";

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;

    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;

    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;

    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;

    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;

    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;

    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;

    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;

    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    /* ...the dark equivalents for every variable above... */
  }
}
```

### The mental model

There are two layers:

1. **Tailwind classes** (e.g. `bg-background`, `text-foreground`, `bg-primary`) — these are **semantic** classes shadcn ships. They don't refer to a hard-coded colour like `bg-blue-600`. Instead they look up the `--background` / `--foreground` / `--primary` CSS variable.
2. **CSS variables** in `:root` (light mode defaults) and `.dark` (dark mode overrides).

So when you write:

```tsx
<div className="bg-background text-foreground">Hello</div>
```

…the actual colour comes from whichever set of variables is currently active. Toggle the `dark` class on `<html>` and **every** shadcn component re-themes itself at once. You change nothing in your components.

### Why HSL, not hex?

The numbers like `222.2 47.4% 11.2%` are **HSL** (Hue Saturation Lightness) values **without** the `hsl(...)` wrapper. shadcn wraps them in `hsl(var(--primary))` inside the component styles. There are two reasons to use HSL:

- **Lightness is one number.** You can easily darken a button on hover by tweaking the L value, without changing the hue.
- **It makes generating shades trivial** — a topic the official theme generator leans on heavily.

---

## 42.2 The `dark` class strategy

There are two common ways to enable dark mode:

| Strategy | How it works |
|---|---|
| `media` | `@media (prefers-color-scheme: dark)` — follows the OS automatically. No JS. |
| `class` | A `.dark` class on `<html>` (or `<body>`). You control it from JS. |

shadcn defaults to the **class strategy** because it lets *the user* override the OS choice (e.g. they have OS dark mode, but want this one app in light mode). It also makes a persistent toggle possible.

So to turn dark mode **on**, we add `class="dark"` to the `<html>` element. To turn it **off**, we remove it. That's it.

Try it manually right now: open your browser DevTools, find `<html>`, add a class `dark`, and watch the page flip. Remove it and watch it flip back.

> **Tailwind v4 note.** In v4 the dark variant defaults to **`@media (prefers-color-scheme: dark)`**. To use the `.dark` class strategy that shadcn relies on, we explicitly declare a custom variant in `src/index.css`. shadcn's init does this for you — you should see a line like `@custom-variant dark (&:is(.dark *));` near the top of the file. If it isn't there, add it just below `@import "tailwindcss";`.

---

## 42.3 Building a typed `<ThemeProvider>`

We want:

- A `theme` value that's `"light" | "dark"` (we'll keep it simple — a `"system"` option is an easy add later).
- A `setTheme` function any component can call.
- The choice **persisted** via `useLocalStorage` so it survives a page reload.
- The `dark` class added to / removed from `<html>` whenever `theme` changes.

We have all the ingredients from earlier classes — custom hooks (37) and Context (38).

### Step 1 — Confirm `useLocalStorage` from Class 37

If you have it in `src/hooks/useLocalStorage.ts` already, skip this. Otherwise, here's the version we'll assume:

```ts
// src/hooks/useLocalStorage.ts
import { useEffect, useState } from "react";

export function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    try {
      const stored = window.localStorage.getItem(key);
      return stored ? (JSON.parse(stored) as T) : initialValue;
    } catch {
      return initialValue;
    }
  });

  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch {
      /* ignore quota / private-browsing errors */
    }
  }, [key, value]);

  return [value, setValue] as const;
}
```

### Step 2 — Write `<ThemeProvider>`

```tsx
// src/theme/ThemeProvider.tsx
import { createContext, useContext, useEffect, type ReactNode } from "react";
import { useLocalStorage } from "@/hooks/useLocalStorage";

export type Theme = "light" | "dark";

interface ThemeContextValue {
  theme: Theme;
  setTheme: (theme: Theme) => void;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | undefined>(undefined);

interface ThemeProviderProps {
  children: ReactNode;
  defaultTheme?: Theme;
}

export function ThemeProvider({
  children,
  defaultTheme = "light",
}: ThemeProviderProps) {
  // 1. Persist the user's choice across reloads
  const [theme, setTheme] = useLocalStorage<Theme>("ui-theme", defaultTheme);

  // 2. Sync the <html> class whenever `theme` changes
  useEffect(() => {
    const root = document.documentElement;
    root.classList.remove("light", "dark");
    root.classList.add(theme);
  }, [theme]);

  const toggleTheme = () => setTheme(theme === "light" ? "dark" : "light");

  return (
    <ThemeContext.Provider value={{ theme, setTheme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. A typed consumer hook with a friendly error
export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) {
    throw new Error("useTheme must be used inside a <ThemeProvider>");
  }
  return ctx;
}
```

### Step 3 — Wrap the app

```tsx
// src/main.tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./index.css";
import App from "./App";
import { ThemeProvider } from "@/theme/ThemeProvider";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <ThemeProvider defaultTheme="light">
      <App />
    </ThemeProvider>
  </StrictMode>
);
```

That's the engine. Now we need a button to switch it.

---

## 42.4 Building `<ThemeToggle>`

A clean, accessible toggle using a shadcn ghost button and Lucide icons:

```tsx
// src/components/ThemeToggle.tsx
import { Moon, Sun } from "lucide-react";
import { Button } from "@/components/ui/button";
import { useTheme } from "@/theme/ThemeProvider";

export function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();
  const isDark = theme === "dark";

  return (
    <Button
      variant="ghost"
      size="icon"
      onClick={toggleTheme}
      aria-label={isDark ? "Switch to light mode" : "Switch to dark mode"}
      title={isDark ? "Light mode" : "Dark mode"}
    >
      {isDark ? <Sun className="h-5 w-5" /> : <Moon className="h-5 w-5" />}
    </Button>
  );
}
```

Drop it into `App.tsx`:

```tsx
// src/App.tsx
import { Toaster } from "@/components/ui/sonner";
import { ThemeToggle } from "@/components/ThemeToggle";
import { Home } from "./pages/Home";

function App() {
  return (
    <div className="min-h-screen bg-background text-foreground">
      <header className="flex items-center justify-between border-b px-6 py-3">
        <h1 className="text-lg font-semibold">My Dashboard</h1>
        <ThemeToggle />
      </header>

      <Home />

      <Toaster richColors position="top-right" />
    </div>
  );
}

export default App;
```

Notice how the header uses `bg-background` and `text-foreground` — the **semantic** classes. When dark mode flips on, the header re-themes itself without you touching a thing.

### A nice flourish — smooth colour transitions

Add this to the root element so theme swaps feel polished instead of jarring:

```tsx
<div className="min-h-screen bg-background text-foreground transition-colors duration-200">
```

---

## 42.5 Customising the colour palette

Now the fun bit. Let's change the **primary** colour to a deep teal. Open `src/index.css` and find these two lines (one in `:root`, one in `.dark`):

```css
:root {
  --primary: 222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;
}

.dark {
  --primary: 210 40% 98%;
  --primary-foreground: 222.2 47.4% 11.2%;
}
```

Replace the light-mode primary with a teal:

```css
:root {
  --primary: 173 80% 35%;        /* teal */
  --primary-foreground: 0 0% 100%;
}
```

Save. Every `<Button>` without a `variant` is now teal. Every `bg-primary`, every `ring-primary`, every "primary" thing across your app — instantly teal. No find-and-replace, no component edits.

Tweak dark mode too:

```css
.dark {
  --primary: 173 70% 60%;        /* a lighter teal for dark backgrounds */
  --primary-foreground: 222 47% 11%;
}
```

### A practical hex → HSL note

Designers often give you a hex like `#0EA5E9` (sky blue). The shadcn variables need **HSL without the wrapper**, like `199 89% 48%`. Quick conversion options:

- **Browser DevTools.** Hover the colour swatch next to any CSS colour value and click the small icon — Chrome and Firefox let you cycle through hex / RGB / HSL.
- **Pasting into an online converter** such as the one bundled with the [shadcn theme generator](https://ui.shadcn.com/themes) (more on this in §42.7).
- **In code**, `color()` and `hsl()` modern CSS functions can also do this conversion at runtime if you really need to.

> **Crucial gotcha.** Write `199 89% 48%`, **not** `hsl(199, 89%, 48%)`. The `hsl(...)` wrapper is added by shadcn's components when they consume the variable.

---

## 42.6 A short tour of `tailwind.config.ts`

In Tailwind **v4**, a `tailwind.config.ts` file is **optional** — most configuration moves into your CSS using `@theme`. shadcn may or may not generate one for you depending on the version of the CLI. If you do have it, here's what to look for:

```ts
// tailwind.config.ts (only if present)
import type { Config } from "tailwindcss";

const config: Config = {
  darkMode: ["class"],
  content: [
    "./index.html",
    "./src/**/*.{ts,tsx}",
  ],
  theme: {
    extend: {
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      colors: {
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        /* ...and so on for every variable */
      },
    },
  },
};

export default config;
```

The important bits to understand:

- `darkMode: ["class"]` — pins us to the `.dark` class strategy (matching what we did in §42.2).
- The `colors` block is what makes `bg-primary` *mean* "use `--primary`". This is the bridge between **Tailwind utilities** and the **CSS variables** we edited.

In Tailwind v4-only setups, the equivalent lives inside `src/index.css` under `@theme { ... }`. The shadcn CLI handles either case. **You should rarely need to edit this file by hand** unless you're adding a brand-new design token (e.g. a custom radius scale or a brand colour).

---

## 42.7 Tooling tip — the official shadcn theme generator

When you want a polished palette without doing colour theory in your head, visit:

**[https://ui.shadcn.com/themes](https://ui.shadcn.com/themes)**

You pick a base colour and a radius, preview the entire UI in light and dark, and the page gives you a ready-to-paste CSS block — the exact same `--background`, `--primary`, `--ring` lines you've been editing. Drop it into `src/index.css`, save, and your whole app reskins.

This is the fastest way to give a client demo a unique identity without writing a single line of CSS yourself.

---

## 42.8 Putting it together — the full picture

Quick sanity checklist for the end of class. Everything below should now be true in your project:

- [ ] `src/hooks/useLocalStorage.ts` exists (from Class 37).
- [ ] `src/theme/ThemeProvider.tsx` exists and wraps the app in `main.tsx`.
- [ ] `src/components/ThemeToggle.tsx` exists and is rendered in your header.
- [ ] Clicking the toggle flips the entire app between light and dark.
- [ ] Reloading the page **keeps** the theme — proving `useLocalStorage` is working.
- [ ] Changing `--primary` in `src/index.css` re-skins every `<Button>` without further edits.

If all of those check out, you've genuinely **earned** the right to put "Tailwind + shadcn/ui + theming" on a CV.

---

## Practice Exercises

1. **Persist across browsers.** Open your project in two browser tabs. Toggle the theme in tab A, then reload tab B. Confirm both end up in the same mode (proving the persistence is keyed to `localStorage`, not memory).

2. **Add a "system" mode (stretch).** Extend `Theme` to `"light" | "dark" | "system"`. When the value is `"system"`, read `window.matchMedia("(prefers-color-scheme: dark)")` and apply the right class. Bonus: react to OS changes live by attaching `change` listeners.

3. **Rebrand the dashboard.** Pick a colour (your favourite, your college's, anything). Use the [shadcn theme generator](https://ui.shadcn.com/themes) to produce a new palette and paste it into `src/index.css`. Confirm both light and dark modes still look good.

4. **Animated toggle.** Add a smooth crossfade between the `<Sun />` and `<Moon />` icons in `<ThemeToggle>`. Hint: stack both icons absolutely inside the button, give each `opacity` + `rotate` classes that flip based on the current theme, and add a `transition-all duration-300`.

---

## Key Takeaways
- shadcn's theme lives in **CSS variables** (`--background`, `--primary`, …) inside `src/index.css`. The values are **HSL without the `hsl(...)` wrapper**.
- The `.dark` class on `<html>` swaps the entire variable set — change *one* class and the whole UI re-themes.
- A `<ThemeProvider>` + `useLocalStorage` + a `useEffect` that toggles `document.documentElement.classList` is all you need for a persistent dark-mode toggle.
- Lucide icons (`Sun`, `Moon`, `Settings`, etc.) are the default — `npm install lucide-react` was done for you during `init`.
- Customising the palette is **find variable → change HSL value → save**. The official theme generator at `ui.shadcn.com/themes` does this visually.
- `tailwind.config.ts` is the **bridge** between utility classes like `bg-primary` and your CSS variables — usually you don't need to touch it.
- **Next class** we leave UI behind and start serious form handling with **React Hook Form** (Class 43), then add **Zod** validation (Class 44).
