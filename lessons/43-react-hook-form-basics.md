# Class 43: React Hook Form Basics

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 42 (Theming, dark mode and customisation) — and a working Vite + React + TypeScript + Tailwind + shadcn/ui project.

## What You Will Learn
- Why we use a forms library instead of writing `useState` for every input
- Installing React Hook Form (RHF) into a Vite project
- The four building blocks: `useForm`, `register`, `handleSubmit`, `formState`
- Setting `defaultValues` and resetting the form with `reset()`
- Disabling the submit button while the form is sending
- Wiring an RHF form to shadcn `<Input>` and `<Button>` directly (without the `<Form>` primitive yet)

---

## 43.1 Why a Forms Library?

In **Class 34** you built a controlled form by hand. The recipe looked like this:

```tsx
const [name, setName] = useState("");
const [email, setEmail] = useState("");
const [message, setMessage] = useState("");
// ...one useState per field
// ...one onChange per field
// ...validation logic mixed into the JSX
```

That works for one or two inputs. But real applications have forms with ten, fifteen, sometimes thirty fields — login, registration, profile editing, checkout, settings. The hand-rolled approach has three real problems:

1. **Performance** — every keystroke re-renders the whole form component. With twenty fields and a heavy parent, typing feels sluggish.
2. **Boilerplate** — you write the same `useState` + `onChange` pattern over and over.
3. **Validation is scattered** — `if (!email.includes("@"))` lives in the submit handler, the error message lives in the JSX, the error state lives in another `useState`. The logic is spread across the file.

**React Hook Form** (RHF) solves all three. Think of it as a tiny library whose entire job is to manage form state efficiently:

- It uses **uncontrolled inputs under the hood** (the input remembers its own value), so the parent does **not** re-render on every keystroke.
- It gives you **one hook** (`useForm`) that returns everything you need.
- It exposes a single `formState.errors` object so all your errors live in one place.

> Analogy: hand-rolled forms are like writing every recipe from scratch. RHF is the cookbook — same ingredients, far less work.

---

## 43.2 Installing React Hook Form

Open your Vite project and run:

```bash
npm install react-hook-form
```

That is the only install for this class. We add Zod and the shadcn `<Form>` primitive in Class 44.

Quick sanity check — open `package.json` and you should see something like:

```json
"dependencies": {
  "react-hook-form": "^7.x.x"
}
```

---

## 43.3 The Four Building Blocks of `useForm`

Almost every RHF form uses the same four pieces:

| Piece | What it does |
|-------|--------------|
| `useForm<T>()` | The hook. Returns everything you need. `<T>` is the shape of your form data. |
| `register("fieldName")` | Connects an input to RHF. You spread it onto the input. |
| `handleSubmit(onValid)` | Wraps your submit handler. Runs validation first, then calls `onValid(data)`. |
| `formState` | An object with `errors`, `isSubmitting`, `isValid`, `isDirty`, etc. |

Here is the smallest possible RHF form:

```tsx
import { useForm } from "react-hook-form";

interface FormValues {
  name: string;
}

function TinyForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<FormValues>();

  const onSubmit = (data: FormValues) => {
    console.log("Submitted:", data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register("name", { required: "Name is required" })}
        placeholder="Your name"
      />
      {errors.name && <p>{errors.name.message}</p>}
      <button type="submit">Send</button>
    </form>
  );
}

export default TinyForm;
```

Read it slowly:

1. `useForm<FormValues>()` creates the form and types every field.
2. `{...register("name", { required: "..." })}` is the magic. It returns `name`, `ref`, `onChange`, and `onBlur` — you spread them onto the input.
3. `handleSubmit(onSubmit)` runs the validators. If they pass, it calls `onSubmit` with a typed `data` object.
4. `errors.name` is automatically populated when validation fails.

> Try changing `required: "Name is required"` to `required: "Tell us your name"` and submit empty — the message updates.

---

## 43.4 Wiring RHF to shadcn `<Input>` and `<Button>`

shadcn's `<Input>` is just a styled HTML input — it accepts every standard prop, including the ones `register()` returns. That means RHF works with shadcn inputs straight out of the box.

Make sure you have the components added (you installed them in Classes 40 and 41):

```bash
npx shadcn@latest add input button label textarea
```

Now a clean version with shadcn:

```tsx
import { useForm } from "react-hook-form";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Label } from "@/components/ui/label";

interface FormValues {
  name: string;
}

function ShadcnTinyForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<FormValues>();

  const onSubmit = (data: FormValues) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-3 max-w-sm">
      <div className="space-y-1">
        <Label htmlFor="name">Name</Label>
        <Input
          id="name"
          placeholder="Your name"
          {...register("name", { required: "Name is required" })}
        />
        {errors.name && (
          <p className="text-sm text-destructive">{errors.name.message}</p>
        )}
      </div>

      <Button type="submit">Send</Button>
    </form>
  );
}

export default ShadcnTinyForm;
```

Notice the **order matters** on the input: `id="name"` first, then `{...register(...)}`. `register` returns its own `name` prop, so spreading it last is fine — but never put `value` or `onChange` on an RHF-registered input. RHF owns them.

---

## 43.5 `defaultValues` — Always Set Them

A common beginner trap: leaving `defaultValues` undefined. Without it, React Hook Form starts each field as `undefined`, which makes TypeScript and controlled-component warnings unhappy.

**Rule**: always give every field a `defaultValue`, even if it is an empty string.

```tsx
const form = useForm<FormValues>({
  defaultValues: {
    name: "",
    email: "",
    message: "",
  },
});
```

`defaultValues` is also how you pre-fill an edit form:

```tsx
const form = useForm<UserFormValues>({
  defaultValues: {
    name: existingUser.name,
    email: existingUser.email,
  },
});
```

---

## 43.6 Resetting the Form with `reset()`

After a successful submit, you usually want the inputs to clear. RHF's `reset()` does exactly that — it puts every field back to its `defaultValues`.

```tsx
const {
  register,
  handleSubmit,
  reset,
  formState: { errors },
} = useForm<FormValues>({
  defaultValues: { name: "", email: "", message: "" },
});

const onSubmit = (data: FormValues) => {
  console.log("Sending:", data);
  reset(); // back to empty
};
```

You can also reset to specific values:

```tsx
reset({ name: "Prabin", email: "", message: "" });
```

---

## 43.7 `isSubmitting` — Disable the Button While Sending

When the user clicks **Send**, you do **not** want them to click it five more times. RHF tracks submission state in `formState.isSubmitting`. As long as your `onSubmit` is an `async` function, RHF flips this to `true` until the promise resolves.

```tsx
const {
  register,
  handleSubmit,
  reset,
  formState: { errors, isSubmitting },
} = useForm<FormValues>({ defaultValues: { name: "", email: "", message: "" } });

const onSubmit = async (data: FormValues) => {
  // Pretend this is a real API call (we cover that in Class 45)
  await new Promise((r) => setTimeout(r, 1000));
  console.log("Sent:", data);
  reset();
};

return (
  <form onSubmit={handleSubmit(onSubmit)}>
    {/* fields ... */}
    <Button type="submit" disabled={isSubmitting}>
      {isSubmitting ? "Sending..." : "Send"}
    </Button>
  </form>
);
```

Two small details that make a real difference:

- `disabled={isSubmitting}` stops double-submits.
- The label change (`"Sending..."`) reassures the user something is happening.

---

## 43.8 Built-in Validation (Without a Library)

RHF has a small set of built-in validators that you pass as the second argument to `register`. They are enough for simple cases — you only reach for Zod (Class 44) when the rules get more complex.

```tsx
{...register("email", {
  required: "Email is required",
  pattern: {
    value: /^\S+@\S+\.\S+$/,
    message: "Enter a valid email address",
  },
})}
```

| Validator | Example |
|-----------|---------|
| `required` | `required: "This field is required"` |
| `minLength` | `minLength: { value: 3, message: "Too short" }` |
| `maxLength` | `maxLength: { value: 100, message: "Too long" }` |
| `pattern` | `pattern: { value: /regex/, message: "..." }` |
| `min` / `max` | for `type="number"` inputs |
| `validate` | a function returning `true` or an error string |

Built-in validators are perfectly fine for this class. We upgrade to Zod in the next class for cleaner, reusable schemas.

---

## 43.9 The Full "Contact Us" Form

Time to put everything together. We will build a contact form with three fields — `name`, `email`, and `message` — using only RHF's built-in validation, wired to shadcn primitives.

```tsx
// src/components/ContactForm.tsx
import { useForm } from "react-hook-form";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Button } from "@/components/ui/button";
import { Label } from "@/components/ui/label";
import { cn } from "@/lib/utils";

interface ContactFormValues {
  name: string;
  email: string;
  message: string;
}

function ContactForm() {
  const {
    register,
    handleSubmit,
    reset,
    formState: { errors, isSubmitting },
  } = useForm<ContactFormValues>({
    defaultValues: {
      name: "",
      email: "",
      message: "",
    },
  });

  const onSubmit = async (data: ContactFormValues) => {
    // Pretend this is a real API call. Class 45 introduces real requests.
    await new Promise((r) => setTimeout(r, 800));
    console.log("Contact form submitted:", data);
    reset();
  };

  return (
    <form
      onSubmit={handleSubmit(onSubmit)}
      className="space-y-4 max-w-md mx-auto p-6 border rounded-lg"
    >
      <h2 className="text-2xl font-semibold">Contact us</h2>

      {/* Name */}
      <div className="space-y-1">
        <Label htmlFor="name">Name</Label>
        <Input
          id="name"
          placeholder="Your name"
          className={cn(errors.name && "border-destructive")}
          {...register("name", {
            required: "Name is required",
            minLength: { value: 2, message: "Name must be at least 2 characters" },
          })}
        />
        {errors.name && (
          <p className="text-sm text-destructive">{errors.name.message}</p>
        )}
      </div>

      {/* Email */}
      <div className="space-y-1">
        <Label htmlFor="email">Email</Label>
        <Input
          id="email"
          type="email"
          placeholder="you@example.com"
          className={cn(errors.email && "border-destructive")}
          {...register("email", {
            required: "Email is required",
            pattern: {
              value: /^\S+@\S+\.\S+$/,
              message: "Enter a valid email address",
            },
          })}
        />
        {errors.email && (
          <p className="text-sm text-destructive">{errors.email.message}</p>
        )}
      </div>

      {/* Message */}
      <div className="space-y-1">
        <Label htmlFor="message">Message</Label>
        <Textarea
          id="message"
          rows={4}
          placeholder="How can we help?"
          className={cn(errors.message && "border-destructive")}
          {...register("message", {
            required: "Message is required",
            minLength: {
              value: 10,
              message: "Message must be at least 10 characters",
            },
          })}
        />
        {errors.message && (
          <p className="text-sm text-destructive">{errors.message.message}</p>
        )}
      </div>

      <Button type="submit" disabled={isSubmitting} className="w-full">
        {isSubmitting ? "Sending..." : "Send message"}
      </Button>
    </form>
  );
}

export default ContactForm;
```

Drop `<ContactForm />` into `App.tsx`, run `npm run dev`, and try it:

- Submit empty — every field shows its red error.
- Type a name shorter than 2 letters — only the name error appears.
- Type a junk email — the email error appears.
- Fill everything in and submit — the button reads **Sending...** for a moment, then resets.

Look at the browser DevTools "Components" panel: the parent does not re-render on every keystroke. That is the RHF performance benefit in action.

---

## 43.10 What We Did Not Cover (Yet)

This class deliberately left two things out. You will pick them up next.

| Topic | Coming in |
|-------|-----------|
| Zod schemas for cleaner, reusable validation rules | Class 44 |
| The shadcn `<Form>`, `<FormField>`, `<FormMessage>` primitives | Class 44 |
| Sending the form data to a real server | Class 45 |

For now, your contact form works, validates, and resets — and you have learned the four core RHF building blocks that everything else builds on.

---

## Practice Exercises

### Exercise 1: A login form
Build a `<LoginForm>` with two fields — `email` and `password`:
- `email` — required, must match an email pattern
- `password` — required, minimum 8 characters
- A submit button that says **Logging in...** while submitting (use a fake 1-second delay)
- Reset the form on successful submit and `console.log` the data

### Exercise 2: A newsletter sign-up
Single field — `email`. Add a small "subscribed!" message that appears for 3 seconds after a successful submit (hint: combine `useState` with `reset()`).

### Exercise 3: A feedback form
Three fields:
- `rating` — a `type="number"` input, required, `min: 1`, `max: 5`
- `comments` — required, between 10 and 200 characters
- `wouldRecommend` — a checkbox, no validation rules

Show all errors using the same `text-destructive` pattern as the contact form. After submit, log the data and reset.

---

## Key Takeaways
- React Hook Form replaces dozens of `useState` calls with a single `useForm` hook.
- The four building blocks are `useForm`, `register`, `handleSubmit`, and `formState`.
- Always provide `defaultValues` for every field.
- `reset()` clears the form back to its defaults — call it after a successful submit.
- `formState.isSubmitting` is your hook for disabling the submit button during async work.
- RHF works with shadcn `<Input>` and `<Textarea>` straight away — just spread `{...register("field")}` onto them.
- **Next class**: we add Zod for type-safe schema validation and integrate the full shadcn `<Form>` primitive.
