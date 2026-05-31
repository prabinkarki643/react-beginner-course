# Class 40: shadcn/ui — Setup & Core Components

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 39 (Tailwind CSS installed in our Vite + React + TS project)

## What You Will Learn
- What shadcn/ui is — and, just as importantly, what it is **not**
- How to initialise shadcn/ui in our existing Vite + React + TS project
- How to add the components we will use today: `button`, `input`, `card`, `badge`, `separator`, `label`
- Where these components live, why we "own" them, and how the `@/` import alias works
- The `cn()` helper that shadcn drops into `src/lib/utils.ts`
- Building a polished **profile card** using `<Card>`, `<Button>`, and `<Badge>`

---

## 40.1 What shadcn/ui is — and what it is NOT

shadcn/ui is one of the most popular UI toolkits in the modern React world, but it works very differently from the libraries you may have heard of (Material UI, Chakra UI, Ant Design, etc.).

| Traditional UI library (e.g. MUI) | shadcn/ui |
|---|---|
| You `npm install` a package | You **copy components into your project** |
| The library owns the styles | **You own** every component file |
| Updates require upgrading a dependency | You upgrade by re-copying when you choose to |
| Limited customisation without overrides | Full edit rights — it's your code |
| Adds the whole library to your bundle | You only have what you've added |

> **The headline:** shadcn/ui is **not an npm package**. It is a CLI that **copies real, readable, editable component files into `src/components/ui/`** in your project. After it runs, those files are yours.

### Why this matters for you

When you reach a tricky design requirement ("our button needs a loading spinner inside it"), you don't fight an opaque library API. You open `src/components/ui/button.tsx` and edit it. It is just a React component with Tailwind classes — the exact things you learned in Classes 29–39.

### What's actually under the hood

Each shadcn component is built on:

- **Radix UI** primitives — accessible, unstyled behaviour (dialogs that trap focus, dropdowns that handle keyboard navigation, etc.).
- **Tailwind CSS** — the visual styling.
- **CVA (`class-variance-authority`)** — a tidy way to express variants (`variant="default"` vs `variant="destructive"`).
- **The `cn()` helper** — combines `clsx` + `tailwind-merge`.

You don't need to memorise any of those names. You will see the imports appear when we add components.

---

## 40.2 Initialising shadcn/ui in our project

We will run the modern `npx shadcn@latest init` flow in the same Vite + React + TS project we used in Class 39.

> **Important command note.** The package is now called `shadcn` (not `shadcn-ui`, which is deprecated). Always use `npx shadcn@latest …`.

### Run init

From the project root:

```bash
npx shadcn@latest init
```

You will be asked a series of questions. Sensible answers for this course:

| Prompt | Choose |
|---|---|
| Which style would you like to use? | **New York** (or Default — both are fine) |
| Which color would you like to use as the base color? | **Slate** |
| Would you like to use CSS variables for theming? | **Yes** |

The CLI will then:

1. Detect that you are using Vite + React + TypeScript + Tailwind v4.
2. Install a handful of small helper packages (`clsx`, `tailwind-merge`, `class-variance-authority`, `lucide-react`, and the Radix primitives needed by the components you'll add later).
3. Create **`components.json`** at the project root (the shadcn config file).
4. Create **`src/lib/utils.ts`** with the `cn()` helper.
5. Update **`src/index.css`** with the CSS variables it uses for theming (we'll explore these in Class 42).
6. Add a path alias `@/*` so you can `import { Button } from "@/components/ui/button"`.

### What if Vite asks me about path aliases?

The CLI will offer to configure the `@/` alias in both `tsconfig.json` (for TypeScript) and `vite.config.ts` (for the bundler). Say **yes** to both. Afterwards your `vite.config.ts` will look something like this:

```ts
// vite.config.ts
import path from "node:path"
import { defineConfig } from "vite"
import react from "@vitejs/plugin-react"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
})
```

…and your `tsconfig.json` (or the referenced `tsconfig.app.json`) will get:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### Check it worked

Open the project. You should now have:

```
src/
├── lib/
│   └── utils.ts          ← the cn() helper lives here
├── components/           ← we'll add UI components into a `ui/` subfolder
├── index.css             ← now contains shadcn's CSS variables + @import "tailwindcss"
└── ...
components.json           ← shadcn's config (at the project root)
```

Run `npm run dev` once. The app should still work exactly as it did at the end of Class 39 — we have only added wiring, not changed any visible behaviour.

---

## 40.3 The `cn()` helper

Open `src/lib/utils.ts`. You will see roughly this:

```ts
// src/lib/utils.ts
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

This is the function we previewed in Class 39. From now on, whenever a component needs **conditional** or **combined** Tailwind classes, we use `cn()`:

```tsx
import { cn } from "@/lib/utils";

function Tag({ active }: { active: boolean }) {
  return (
    <span
      className={cn(
        "inline-flex items-center rounded-full px-3 py-1 text-xs font-medium",
        active
          ? "bg-blue-600 text-white"
          : "bg-slate-100 text-slate-700"
      )}
    >
      Tag
    </span>
  );
}
```

Why bother instead of a template literal?

- **`clsx`** lets us drop in `false`/`null`/`undefined` safely (`active && "..."`).
- **`twMerge`** resolves Tailwind conflicts: if both sides try to set padding, the **last one wins** (`"p-2 p-4"` → `"p-4"`), which prevents subtle bugs when you forward `className` from a parent.

We will reach for `cn()` in nearly every component from here on.

---

## 40.4 Adding our first batch of components

Components are added one at a time (or as a space-separated list) using the CLI. Let's add the set we'll use today:

```bash
npx shadcn@latest add button input card badge separator label
```

When this finishes, you'll have:

```
src/components/ui/
├── button.tsx
├── input.tsx
├── card.tsx
├── badge.tsx
├── separator.tsx
└── label.tsx
```

Open `button.tsx`. You will see a real, readable React component built on Radix Slot + CVA + Tailwind, exporting a typed `Button` and a `buttonVariants` helper. **This file is yours.** You can edit, extend, or simplify it.

### The `@/components/ui/...` import path

Thanks to the alias from Section 40.2, you import these from anywhere in `src/` like this:

```tsx
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Card, CardContent } from "@/components/ui/card"
import { Badge } from "@/components/ui/badge"
import { Separator } from "@/components/ui/separator"
import { Label } from "@/components/ui/label"
```

No more `../../components/...` zig-zagging.

---

## 40.5 The `<Button>` component

```tsx
import { Button } from "@/components/ui/button";

function ButtonDemo() {
  return (
    <div className="flex flex-wrap gap-2">
      {/* Variants */}
      <Button>Default</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="destructive">Delete</Button>
      <Button variant="outline">Outline</Button>
      <Button variant="ghost">Ghost</Button>
      <Button variant="link">Link style</Button>

      {/* Sizes */}
      <Button size="sm">Small</Button>
      <Button size="default">Default</Button>
      <Button size="lg">Large</Button>
      <Button size="icon" aria-label="Settings">⚙</Button>

      {/* Disabled */}
      <Button disabled>Disabled</Button>

      {/* Behaves like a normal button */}
      <Button onClick={() => alert("Clicked!")}>Click me</Button>
    </div>
  );
}
```

`<Button>` accepts every HTML `<button>` prop (`onClick`, `type`, `disabled`, `aria-*`) plus shadcn's `variant` and `size`.

---

## 40.6 The `<Input>` and `<Label>` components

```tsx
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

function EmailField() {
  return (
    <div className="grid gap-2 max-w-sm">
      <Label htmlFor="email">Email</Label>
      <Input
        id="email"
        type="email"
        placeholder="you@example.com"
        onChange={(e) => console.log(e.target.value)}
      />
    </div>
  );
}
```

A `<Label>` paired with an `<Input>` (via `htmlFor`/`id`) is the accessible default and the one we'll use throughout the course.

---

## 40.7 The `<Card>` family

The `<Card>` is a small kit of subcomponents you compose together:

| Subcomponent | Purpose |
|---|---|
| `<Card>` | Outer container with border, padding, rounded corners |
| `<CardHeader>` | Top section — typically title + description |
| `<CardTitle>` | The card's heading |
| `<CardDescription>` | A subtitle / supporting line |
| `<CardContent>` | The main body |
| `<CardFooter>` | Bottom row — typically actions |

```tsx
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Button } from "@/components/ui/button";

function SimpleCard() {
  return (
    <Card className="max-w-sm">
      <CardHeader>
        <CardTitle>Welcome back</CardTitle>
        <CardDescription>Sign in to continue to your dashboard.</CardDescription>
      </CardHeader>
      <CardContent>
        <p className="text-sm text-muted-foreground">
          You have 3 unread notifications.
        </p>
      </CardContent>
      <CardFooter className="flex justify-end gap-2">
        <Button variant="ghost">Skip</Button>
        <Button>Continue</Button>
      </CardFooter>
    </Card>
  );
}
```

> **Note the `text-muted-foreground` class.** shadcn defines a small set of **semantic colour classes** — `background`, `foreground`, `muted`, `muted-foreground`, `primary`, `destructive` — that map to CSS variables. We'll customise those in Class 42. For now, just trust them: they look right in both light and dark mode.

---

## 40.8 The `<Badge>` component

A `<Badge>` is a small inline tag — perfect for roles, statuses, or counts.

```tsx
import { Badge } from "@/components/ui/badge";

function BadgeDemo() {
  return (
    <div className="flex flex-wrap gap-2">
      <Badge>Default</Badge>
      <Badge variant="secondary">Secondary</Badge>
      <Badge variant="destructive">Destructive</Badge>
      <Badge variant="outline">Outline</Badge>
    </div>
  );
}
```

---

## 40.9 The `<Separator>` component

A tiny but useful one — a horizontal or vertical rule, accessibly marked.

```tsx
import { Separator } from "@/components/ui/separator";

function Example() {
  return (
    <div className="space-y-4">
      <p>Section one</p>
      <Separator />
      <p>Section two</p>
    </div>
  );
}
```

---

## 40.10 Putting it together — a "Profile card"

Let's build a real, polished profile card using everything we've added.

```tsx
// src/App.tsx
import { Mail, MapPin } from "lucide-react";
import {
  Card,
  CardContent,
  CardDescription,
  CardFooter,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Separator } from "@/components/ui/separator";

interface Profile {
  name: string;
  role: string;
  location: string;
  email: string;
  initials: string;
  skills: string[];
  isOnline: boolean;
}

const profile: Profile = {
  name: "Prabin Karki",
  role: "Senior Full-Stack Developer",
  location: "Kathmandu, Nepal",
  email: "prabin@example.com",
  initials: "PK",
  skills: ["React", "TypeScript", "Node.js", "AWS"],
  isOnline: true,
};

function App() {
  return (
    <div className="min-h-screen bg-slate-100 flex items-center justify-center p-4">
      <Card className="w-full max-w-md">
        <CardHeader>
          <div className="flex items-center gap-4">
            {/* Avatar circle */}
            <div className="relative">
              <div
                className="h-14 w-14 rounded-full bg-blue-600 text-white
                           flex items-center justify-center text-lg font-semibold"
              >
                {profile.initials}
              </div>
              {profile.isOnline && (
                <span
                  className="absolute bottom-0 right-0 h-3 w-3 rounded-full
                             bg-green-500 ring-2 ring-white"
                  aria-label="Online"
                />
              )}
            </div>

            <div className="flex-1">
              <CardTitle>{profile.name}</CardTitle>
              <CardDescription>{profile.role}</CardDescription>
            </div>

            <Badge variant={profile.isOnline ? "default" : "secondary"}>
              {profile.isOnline ? "Online" : "Away"}
            </Badge>
          </div>
        </CardHeader>

        <CardContent className="space-y-4">
          <div className="space-y-2 text-sm text-muted-foreground">
            <p className="flex items-center gap-2">
              <MapPin className="h-4 w-4" />
              {profile.location}
            </p>
            <p className="flex items-center gap-2">
              <Mail className="h-4 w-4" />
              {profile.email}
            </p>
          </div>

          <Separator />

          <div>
            <p className="text-sm font-medium mb-2">Skills</p>
            <div className="flex flex-wrap gap-2">
              {profile.skills.map((skill) => (
                <Badge key={skill} variant="outline">
                  {skill}
                </Badge>
              ))}
            </div>
          </div>
        </CardContent>

        <CardFooter className="flex gap-2">
          <Button className="flex-1">Message</Button>
          <Button variant="outline" className="flex-1">
            Follow
          </Button>
        </CardFooter>
      </Card>
    </div>
  );
}

export default App;
```

### What's happening here

- `lucide-react` was installed for us during `init`. It's the **default icon set** for shadcn. Browse all icons at [lucide.dev/icons](https://lucide.dev/icons).
- We're using the semantic colour `text-muted-foreground` for secondary text — this will adapt automatically when we add dark mode in Class 42.
- The `Badge` variant changes based on `isOnline` — a tiny example of *driving design from data*, a habit we'll lean on heavily later.
- The whole component is responsive thanks to `max-w-md` and `w-full`.

Run `npm run dev` and admire — that is a professional-looking card built with about ten minutes of typing.

---

## 40.11 What we did NOT install today (and that's deliberate)

We deliberately stopped at the components needed for a static card. In **Class 41** we will add:

- `form` (the shadcn `<Form>` primitive — preview only; full RHF wiring lands in Class 44)
- `dialog` (modals)
- `alert-dialog` (destructive confirmations)
- `sonner` (toast notifications)

And in **Class 42** we will explore the CSS variables in `src/index.css` to add dark mode and customise the colour palette.

---

## Practice Exercises

1. **Read the source.** Open `src/components/ui/button.tsx`, `card.tsx`, and `badge.tsx`. Identify (a) which import is from `@radix-ui/...`, (b) where the `cn()` helper is used, and (c) where `cva` defines the variants. You don't need to fully understand it — just see that it's normal React.

2. **Three profile cards in a grid.** Make an array of three different profiles and render a `<Card>` for each. Use `grid grid-cols-1 md:grid-cols-3 gap-4` for the layout.

3. **A new button variant.** Open `button.tsx` and add a fifth variant called `"success"` (green background, white text, darker green on hover). Use it: `<Button variant="success">Save</Button>`. This is the moment you feel the power of "you own the component".

4. **`cn()` practice.** Build a `<StatBadge>` component that takes `tone: "positive" | "neutral" | "negative"` and `children`. Use a single `cn(...)` call with three conditional branches to colour it.

---

## Key Takeaways
- shadcn/ui is **not an npm package** — it's a CLI that copies real components into `src/components/ui/`. You own them.
- Use the current command: `npx shadcn@latest init` and `npx shadcn@latest add <component>`.
- After init, the `@/` alias lets you write `import { Button } from "@/components/ui/button"`.
- The `cn()` helper in `src/lib/utils.ts` is now your default tool for combining Tailwind classes.
- The `<Card>` family composes (`Card` → `CardHeader` → `CardTitle` / `CardDescription` → `CardContent` → `CardFooter`) — learn this shape, it appears everywhere.
- **Next class** we add `form`, `dialog`, `alert-dialog`, and `sonner` to handle modals, confirmations, and toasts.
