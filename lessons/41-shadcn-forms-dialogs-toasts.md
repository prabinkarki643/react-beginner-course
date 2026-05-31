# Class 41: shadcn — Form, Dialog, AlertDialog, Sonner Toasts

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 40 (shadcn/ui initialised, `cn()` available, `@/` alias working)

## What You Will Learn
- Adding the next batch of shadcn components: `form`, `dialog`, `alert-dialog`, `sonner`, `label`
- A **preview** of the shadcn `<Form>` primitive (we'll use it lightly today and wire up React Hook Form properly in Class 44)
- Building a **modal flow** with `<Dialog>` controlled by `useState`
- Building a **destructive confirmation** with `<AlertDialog>`
- Mounting `<Toaster />` once and firing `toast.success(...)` / `toast.error(...)` from anywhere with **Sonner**
- A full mini-feature: a **delete user** flow — button → confirm dialog → success toast

---

## 41.1 Adding the components

Carrying on in the same project from Class 40, run:

```bash
npx shadcn@latest add form dialog alert-dialog sonner label
```

(`label` may already exist from Class 40 — the CLI will simply skip it or ask whether to overwrite. Either choice is safe.)

You should now have these new files:

```
src/components/ui/
├── alert-dialog.tsx
├── dialog.tsx
├── form.tsx
├── sonner.tsx
└── label.tsx
```

You will also notice that `react-hook-form`, `@hookform/resolvers`, `zod`, and `sonner` were installed for you. We'll touch them briefly today and then study them properly in Classes 43–44.

---

## 41.2 The `<Form>` primitive — a gentle preview

The shadcn `<Form>` is a small set of components designed to **pair with React Hook Form (RHF)**. Today we'll use it without any validation library to keep the surface area small. **In Class 44** we will return and add Zod for proper schema validation.

Here is the simplest possible RHF-powered form using the shadcn primitives. Read the comments carefully — every line is doing one job.

```tsx
// src/components/SubscribeForm.tsx
import { useForm } from "react-hook-form";
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "@/components/ui/form";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";

interface SubscribeValues {
  email: string;
}

export function SubscribeForm() {
  // 1. RHF gives us a `form` object with methods & state
  const form = useForm<SubscribeValues>({
    defaultValues: { email: "" },
  });

  // 2. handleSubmit runs our function only if everything is valid
  const onSubmit = (values: SubscribeValues) => {
    console.log("Submitted:", values);
  };

  return (
    // 3. <Form {...form}> wires the shadcn primitive to RHF's context
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4 max-w-sm">
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input type="email" placeholder="you@example.com" {...field} />
              </FormControl>
              {/* Validation messages will appear here in Class 44 */}
              <FormMessage />
            </FormItem>
          )}
        />

        <Button type="submit">Subscribe</Button>
      </form>
    </Form>
  );
}
```

### What each piece is for

| Component | Role |
|---|---|
| `<Form>` | A wrapper that shares RHF's context with the fields below it |
| `<FormField>` | Connects a single named field to RHF and provides a render prop |
| `<FormItem>` | The visual row (label + control + message) — handles spacing |
| `<FormLabel>` | A label that automatically pairs with the control (accessibility) |
| `<FormControl>` | A slot that forwards the right props to the actual input |
| `<FormMessage>` | Renders validation errors (silent today, useful from Class 44) |

> **Why preview this now?** Because shadcn `<Dialog>`s very often contain a form, and we want you to recognise the shape when you see it. Don't worry if it feels a bit ceremonial — Class 44 will explain *why* every piece is there.

---

## 41.3 `<Dialog>` — modal flows

A **Dialog** is a modal that overlays the page. Use it for things like editing a record, viewing details, or running a quick task that needs full attention.

shadcn's `<Dialog>` has two ways to be opened:

1. **Uncontrolled** — wrap a trigger button in `<DialogTrigger>` and shadcn manages open/closed state internally.
2. **Controlled** — manage `open` with your own `useState`. Use this when you need to **close the dialog from code** (e.g. after a successful action).

We'll use the **controlled** pattern because it's the one you'll need 90% of the time.

```tsx
// src/components/EditNameDialog.tsx
import { useState } from "react";
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
} from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Label } from "@/components/ui/label";

interface EditNameDialogProps {
  initialName: string;
  onSave: (newName: string) => void;
}

export function EditNameDialog({ initialName, onSave }: EditNameDialogProps) {
  const [open, setOpen] = useState(false);
  const [name, setName] = useState(initialName);

  const handleSave = () => {
    onSave(name);
    setOpen(false); // close the dialog from code
  };

  return (
    <>
      <Button variant="outline" onClick={() => setOpen(true)}>
        Edit name
      </Button>

      <Dialog open={open} onOpenChange={setOpen}>
        <DialogContent>
          <DialogHeader>
            <DialogTitle>Edit your name</DialogTitle>
            <DialogDescription>
              This will update how your name appears across the app.
            </DialogDescription>
          </DialogHeader>

          <div className="grid gap-2 py-2">
            <Label htmlFor="name">Display name</Label>
            <Input
              id="name"
              value={name}
              onChange={(e) => setName(e.target.value)}
            />
          </div>

          <DialogFooter>
            <Button variant="ghost" onClick={() => setOpen(false)}>
              Cancel
            </Button>
            <Button onClick={handleSave}>Save</Button>
          </DialogFooter>
        </DialogContent>
      </Dialog>
    </>
  );
}
```

### Things to notice

- `<Dialog open={open} onOpenChange={setOpen}>` — we are giving the Dialog **two-way control**. Our `setOpen` is the same `useState` setter, so the dialog can close itself (e.g. when the user hits Escape) **and** we can close it from our `handleSave`.
- Pressing **Escape**, clicking the **X** (rendered automatically), or clicking the **dimmed overlay** all close the dialog — shadcn + Radix handle this for you, including accessible focus trapping.
- The form **state lives outside the dialog content** (in `name`/`setName`). That means it survives even if the user closes and re-opens the dialog. Whether that is what you want is up to you — sometimes you'll deliberately reset state when the dialog opens.

---

## 41.4 `<AlertDialog>` — destructive confirmations

`<Dialog>` is for *flows*. `<AlertDialog>` is for **dangerous, irreversible actions** where we want a clear "yes/no" choice.

Differences from `<Dialog>`:

- The user **must** click Cancel or Action — clicking outside or pressing Escape behaves more strictly.
- It's semantically marked up as a `role="alertdialog"` for assistive tech.
- It has paired `<AlertDialogCancel>` and `<AlertDialogAction>` buttons baked in.

```tsx
// src/components/DeleteUserButton.tsx
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
import { Button } from "@/components/ui/button";
import { Trash2 } from "lucide-react";

interface DeleteUserButtonProps {
  userName: string;
  onConfirm: () => void;
}

export function DeleteUserButton({ userName, onConfirm }: DeleteUserButtonProps) {
  return (
    <AlertDialog>
      <AlertDialogTrigger asChild>
        <Button variant="destructive" size="sm">
          <Trash2 className="mr-2 h-4 w-4" />
          Delete
        </Button>
      </AlertDialogTrigger>

      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>Delete {userName}?</AlertDialogTitle>
          <AlertDialogDescription>
            This will permanently remove the user and all their posts.
            This action cannot be undone.
          </AlertDialogDescription>
        </AlertDialogHeader>

        <AlertDialogFooter>
          <AlertDialogCancel>Cancel</AlertDialogCancel>
          <AlertDialogAction onClick={onConfirm}>
            Yes, delete
          </AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  );
}
```

### `asChild` — what is it?

`<AlertDialogTrigger asChild>` tells the trigger to **render its child as the trigger**, instead of wrapping it in a `<button>` of its own. Without `asChild`, you'd end up with a button inside a button — invalid HTML. The same `asChild` prop appears on `<DialogTrigger>` too. Remember it.

---

## 41.5 Sonner toasts — the polite messenger

Toasts are the small non-blocking messages that appear at the corner of the screen ("Saved", "Something went wrong", "Sent to printer"). shadcn uses the **Sonner** library, which is fast, accessible, and beautiful out of the box.

### Step 1 — Mount `<Toaster />` once at the app root

Open `App.tsx` and import the toaster shadcn added for you, then render it **once**:

```tsx
// src/App.tsx
import { Toaster } from "@/components/ui/sonner";
import { Home } from "./pages/Home";

function App() {
  return (
    <>
      <Home />
      <Toaster richColors position="top-right" />
    </>
  );
}

export default App;
```

The `<Toaster />` is invisible until a toast appears. Common props:

- `position="top-right"` (default `"bottom-right"`)
- `richColors` — colourful success/error/warning styling
- `closeButton` — add an X to each toast
- `expand` — show stacked toasts expanded by default

### Step 2 — Fire toasts from anywhere

```tsx
import { toast } from "sonner";

// Plain
toast("Profile updated");

// Variants
toast.success("Settings saved");
toast.error("Could not connect to server");
toast.info("Three new messages");
toast.warning("Battery low");

// With a description
toast.success("Post published", {
  description: "It's now visible to everyone.",
});

// With an action button
toast("New comment", {
  action: {
    label: "View",
    onClick: () => console.log("View clicked"),
  },
});
```

Notice we import `toast` **directly from `sonner`**, not from a shadcn file. The shadcn `sonner.tsx` only wraps `<Toaster />` so it picks up your theme — the `toast()` function itself is the Sonner library's.

---

## 41.6 Putting it together — a "Delete user" flow

Let's build the feature called for in the SUMMARY checklist: **button → confirm → toast on success**. This is the exact shape you'll use again in the capstone.

```tsx
// src/pages/Home.tsx
import { useState } from "react";
import { toast } from "sonner";
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
import {
  Card,
  CardContent,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Badge } from "@/components/ui/badge";
import { Trash2 } from "lucide-react";

interface User {
  id: number;
  name: string;
  email: string;
  role: "admin" | "user";
}

const initialUsers: User[] = [
  { id: 1, name: "Aanya Sharma", email: "aanya@example.com", role: "admin" },
  { id: 2, name: "Bibek Thapa", email: "bibek@example.com", role: "user" },
  { id: 3, name: "Sita Rai", email: "sita@example.com", role: "user" },
];

export function Home() {
  const [users, setUsers] = useState<User[]>(initialUsers);

  const handleDelete = (userToDelete: User) => {
    // Pretend we called an API — we'll do this for real in Classes 45–47
    setUsers((prev) => prev.filter((u) => u.id !== userToDelete.id));

    toast.success(`Deleted ${userToDelete.name}`, {
      description: "The user has been permanently removed.",
    });
  };

  return (
    <main className="min-h-screen bg-slate-50 p-6">
      <Card className="max-w-2xl mx-auto">
        <CardHeader>
          <CardTitle>Users</CardTitle>
        </CardHeader>
        <CardContent className="space-y-3">
          {users.length === 0 && (
            <p className="text-sm text-muted-foreground text-center py-8">
              No users left. Try refreshing the page to reset.
            </p>
          )}

          {users.map((user) => (
            <div
              key={user.id}
              className="flex items-center justify-between rounded-lg border p-3"
            >
              <div>
                <p className="font-medium">{user.name}</p>
                <p className="text-sm text-muted-foreground">{user.email}</p>
              </div>

              <div className="flex items-center gap-2">
                <Badge variant={user.role === "admin" ? "default" : "secondary"}>
                  {user.role}
                </Badge>

                {/* The destructive confirmation flow */}
                <AlertDialog>
                  <AlertDialogTrigger asChild>
                    <Button variant="ghost" size="icon" aria-label={`Delete ${user.name}`}>
                      <Trash2 className="h-4 w-4 text-destructive" />
                    </Button>
                  </AlertDialogTrigger>

                  <AlertDialogContent>
                    <AlertDialogHeader>
                      <AlertDialogTitle>Delete {user.name}?</AlertDialogTitle>
                      <AlertDialogDescription>
                        This will permanently remove the user. This action cannot
                        be undone.
                      </AlertDialogDescription>
                    </AlertDialogHeader>
                    <AlertDialogFooter>
                      <AlertDialogCancel>Cancel</AlertDialogCancel>
                      <AlertDialogAction onClick={() => handleDelete(user)}>
                        Yes, delete
                      </AlertDialogAction>
                    </AlertDialogFooter>
                  </AlertDialogContent>
                </AlertDialog>
              </div>
            </div>
          ))}
        </CardContent>
      </Card>
    </main>
  );
}
```

Wire it up by making `App.tsx` render `<Home />` and mount the `<Toaster />`:

```tsx
// src/App.tsx
import { Toaster } from "@/components/ui/sonner";
import { Home } from "./pages/Home";

function App() {
  return (
    <>
      <Home />
      <Toaster richColors position="top-right" />
    </>
  );
}

export default App;
```

Run the app, click the bin icon next to any user, confirm in the alert, and watch the user disappear with a green success toast in the top-right corner. This is the **exact pattern** you'll re-use in the capstone for deleting posts.

---

## 41.7 A useful side-note — error toasts from real failures

In Class 45 we'll start making real HTTP requests. The pattern there will be:

```tsx
try {
  await deleteUser(user.id);
  toast.success(`Deleted ${user.name}`);
} catch (error) {
  toast.error("Failed to delete user", {
    description: error instanceof Error ? error.message : "Unknown error",
  });
}
```

Keep this shape in mind. Success path → success toast. Catch block → error toast. Users always know what happened.

---

## Practice Exercises

1. **Edit-name dialog.** Take the controlled `<Dialog>` from §41.3, render it inside the `<Home>` page, and connect it to a piece of `useState` so changing the name updates a heading at the top of the page. Show a `toast.success("Name updated")` after a successful save.

2. **Cancel-order confirmation.** Build a `<CancelOrderButton>` using `<AlertDialog>`. The trigger should be a `variant="destructive"` button reading "Cancel order". The confirmation message should include the order ID prop. On confirm, log to the console and fire `toast.warning("Order cancelled")`.

3. **Toaster customisation.** In `App.tsx`, try at least three different combinations of `<Toaster />` props: top-left rich colours, bottom-right with close buttons, and top-centre expanded. Decide which feels best for the dashboard you'll build in the capstone.

4. **Reusable `<ConfirmDialog>` (optional but recommended).** Wrap the AlertDialog boilerplate from §41.6 into a `<ConfirmDialog>` component that takes `trigger`, `title`, `description`, `confirmLabel`, and `onConfirm` props. Use it twice on the page to feel the win.

---

## Key Takeaways
- Add the toolkit in one go: `npx shadcn@latest add form dialog alert-dialog sonner label`.
- The shadcn `<Form>` primitive pairs with **React Hook Form** — today it's a preview; Class 44 wires in Zod validation.
- `<Dialog>` is for editing/viewing flows. Drive it from `useState` (`open` + `onOpenChange`) when you need to close it from code.
- `<AlertDialog>` is for **destructive confirmations**: it forces a deliberate Cancel/Action choice.
- Always use `asChild` on `<DialogTrigger>` / `<AlertDialogTrigger>` when you supply your own `<Button>`.
- Mount `<Toaster />` once in `App.tsx`. Then fire toasts from anywhere with `toast.success(...)`, `toast.error(...)`, and friends.
- **Next class** we customise the look — CSS variables, the `dark` class, a `<ThemeProvider>` that remembers your choice, and a `<ThemeToggle>` button.
