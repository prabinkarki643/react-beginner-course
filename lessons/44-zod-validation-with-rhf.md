# Class 44: Zod Validation with React Hook Form

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 43 (React Hook Form basics)

## What You Will Learn
- Why Zod is the standard validation library — one source of truth for schema **and** types
- Defining schemas with `z.object`, `z.string`, `z.number`, `.min`, `.max`, `.email`, `.optional`, `.refine`
- Deriving TypeScript types from a Zod schema with `z.infer<typeof schema>`
- Connecting Zod to React Hook Form using `zodResolver`
- The full shadcn `<Form>` primitive: `<Form>`, `<FormField>`, `<FormItem>`, `<FormLabel>`, `<FormControl>`, `<FormDescription>`, `<FormMessage>`
- Building a complete "create post" form with title, body and tags

---

## 44.1 Why Zod?

In Class 43 you validated forms with RHF's built-in rules (`required`, `minLength`, `pattern`). That is fine for tiny forms, but it has a few weaknesses for anything larger:

1. The validation rules live **inside the JSX**, mixed in with the inputs.
2. There is no way to reuse the same rules across a "create" form and an "edit" form.
3. TypeScript only knows what fields exist — it does not know they have been validated.
4. Cross-field rules (like "password and confirm must match") are clumsy.

**Zod** is a TypeScript-first validation library. You describe the shape of your data as a **schema**, and Zod gives you back:

- A function that validates data at runtime (catches bad input).
- A TypeScript type derived from that schema (catches mistakes at compile time).

```
        Zod schema (one source of truth)
       /                                \
      v                                  v
  Runtime validation              TypeScript type
  (the user typed junk)           (Code knows the shape)
```

> Analogy: a Zod schema is like a passport application form. It says *which* fields exist, *what* shape they have, and *which rules* they must obey. Use the same passport definition for runtime checks **and** for the type system.

---

## 44.2 Installing Zod and the Resolver

```bash
npm install zod @hookform/resolvers
```

- `zod` — the schema library itself.
- `@hookform/resolvers` — the bridge that lets React Hook Form use a Zod schema for validation.

You will also need the shadcn `<Form>` primitive. Add it (alongside the input/button/textarea you already have):

```bash
npx shadcn@latest add form
```

This adds `src/components/ui/form.tsx` to your project, which gives you the `<Form>`, `<FormField>`, `<FormItem>`, `<FormLabel>`, `<FormControl>`, `<FormDescription>` and `<FormMessage>` components.

---

## 44.3 Your First Zod Schema

A Zod schema describes a piece of data. For a form, that "piece of data" is the whole form object.

```ts
import { z } from "zod";

const userSchema = z.object({
  name: z.string().min(2, "Name must be at least 2 characters"),
  age: z.number().min(18, "You must be 18 or older"),
});
```

That schema says: an object with `name` (a string of at least 2 characters) and `age` (a number of at least 18). Each rule takes an optional message — that message is what the user sees if validation fails.

### Common building blocks

```ts
// Strings
z.string()                                // must be a string
z.string().min(3, "Too short")            // minimum length
z.string().max(100, "Too long")           // maximum length
z.string().email("Invalid email")         // must be an email
z.string().url("Invalid URL")             // must be a URL
z.string().trim()                         // trim whitespace before validating

// Numbers
z.number()                                // must be a number
z.number().min(0)                         // minimum value
z.number().max(120)                       // maximum value
z.number().int("Must be a whole number")  // integer only

// Booleans
z.boolean()

// Pick from a fixed list
z.enum(["draft", "published", "archived"])

// Optional field (can be missing or undefined)
z.string().optional()

// Coerce — convert a string input to a number
z.coerce.number().min(1)
```

### Object schemas

For forms, you almost always use `z.object({...})`:

```ts
const loginSchema = z.object({
  email: z.string().email("Enter a valid email"),
  password: z.string().min(8, "At least 8 characters"),
});
```

---

## 44.4 Deriving TypeScript Types with `z.infer`

This is the killer feature. Instead of writing an `interface` by hand and then keeping it in sync with your validation rules, you derive the type from the schema:

```ts
import { z } from "zod";

const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

type LoginFormValues = z.infer<typeof loginSchema>;
// LoginFormValues is automatically:
// {
//   email: string;
//   password: string;
// }
```

If you add `rememberMe: z.boolean()` to the schema, `LoginFormValues` updates automatically. No manual sync, no drift, no duplicated work.

---

## 44.5 Cross-field Rules with `.refine`

Some validations span multiple fields. Classic example: "password and confirm-password must match." That is what `.refine` is for. You chain it to the object schema:

```ts
const registerSchema = z
  .object({
    email: z.string().email("Enter a valid email"),
    password: z.string().min(8, "At least 8 characters"),
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "Passwords do not match",
    path: ["confirmPassword"], // attach the error to confirmPassword
  });
```

`path: ["confirmPassword"]` tells Zod which field the error message should appear next to.

---

## 44.6 Connecting Zod to React Hook Form

Now we glue the two together. The bridge is `zodResolver` from `@hookform/resolvers/zod`:

```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const loginSchema = z.object({
  email: z.string().email("Enter a valid email"),
  password: z.string().min(8, "At least 8 characters"),
});

type LoginFormValues = z.infer<typeof loginSchema>;

function LoginForm() {
  const form = useForm<LoginFormValues>({
    resolver: zodResolver(loginSchema),
    defaultValues: { email: "", password: "" },
  });

  const onSubmit = (data: LoginFormValues) => {
    console.log("Valid:", data);
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      {/* fields ... */}
    </form>
  );
}
```

Three things changed compared to Class 43:

1. The `resolver: zodResolver(loginSchema)` option tells RHF to use the Zod schema for validation.
2. The form values are typed using `z.infer<typeof loginSchema>`.
3. You no longer pass `required` or `minLength` to `register` — the schema owns all the rules.

---

## 44.7 The shadcn `<Form>` Primitive

Up to now we wrote the label, input, and error message by hand for every field. The shadcn `<Form>` primitive gives us reusable building blocks for that pattern. Here are the seven pieces:

| Component | Purpose |
|-----------|---------|
| `<Form>` | The wrapper. It is just a typed provider for the RHF `form` object. |
| `<FormField>` | Binds **one** field to RHF. Takes `control`, `name`, and a `render` prop. |
| `<FormItem>` | The visual wrapper for a single field (label + control + message). |
| `<FormLabel>` | The accessible label. Automatically linked to the input. |
| `<FormControl>` | Wraps the actual input so RHF and shadcn can hook into it. |
| `<FormDescription>` | Helper text under the label. |
| `<FormMessage>` | Renders the validation error message for this field automatically. |

The big win is that `<FormMessage>` reads the error from RHF for you — no more `{errors.name && <p>...</p>}`. And `<FormLabel>` turns red automatically when the field is invalid.

### Anatomy of one field

```tsx
<FormField
  control={form.control}
  name="email"
  render={({ field }) => (
    <FormItem>
      <FormLabel>Email</FormLabel>
      <FormControl>
        <Input type="email" placeholder="you@example.com" {...field} />
      </FormControl>
      <FormDescription>We will never share your email.</FormDescription>
      <FormMessage />
    </FormItem>
  )}
/>
```

Read it carefully:

- `control={form.control}` and `name="email"` tell RHF which field this is.
- The `render` callback gives you `field` — spread it onto your input, exactly like the props `register()` used to return.
- `<FormMessage />` shows the Zod error for this field. You write nothing.

---

## 44.8 The Full "Create Post" Form

Now we build the lesson goal: a post-creation form with title, body and tags, fully validated by Zod and rendered with the shadcn `<Form>` primitive.

### Step 1 — define the schema

Keep schemas in their own file. This is industry standard — schemas are reusable, and types derive from them.

```ts
// src/schemas/postSchema.ts
import { z } from "zod";

export const postSchema = z.object({
  title: z
    .string()
    .trim()
    .min(3, "Title must be at least 3 characters")
    .max(100, "Title must be at most 100 characters"),
  body: z
    .string()
    .trim()
    .min(10, "Body must be at least 10 characters")
    .max(1000, "Body must be at most 1000 characters"),
  tags: z
    .string()
    .trim()
    .optional()
    .refine(
      (val) => !val || val.split(",").every((t) => t.trim().length > 0),
      "Tags must be comma-separated and non-empty",
    ),
});

export type PostFormValues = z.infer<typeof postSchema>;
```

A few touches worth pointing out:

- `.trim()` removes accidental leading/trailing spaces **before** the length check.
- `tags` is `optional()` — the user can leave it blank.
- The `.refine` checks the structure of the tags string (a comma-separated list with no empty entries).

### Step 2 — build the form

```tsx
// src/components/CreatePostForm.tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { postSchema, type PostFormValues } from "@/schemas/postSchema";
import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "@/components/ui/form";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Button } from "@/components/ui/button";

function CreatePostForm() {
  const form = useForm<PostFormValues>({
    resolver: zodResolver(postSchema),
    defaultValues: {
      title: "",
      body: "",
      tags: "",
    },
  });

  const onSubmit = async (data: PostFormValues) => {
    // Pretend this is the API call — we wire real requests in Class 45
    await new Promise((r) => setTimeout(r, 800));
    console.log("Post created:", {
      ...data,
      tags: data.tags
        ? data.tags.split(",").map((t) => t.trim())
        : [],
    });
    form.reset();
  };

  return (
    <Form {...form}>
      <form
        onSubmit={form.handleSubmit(onSubmit)}
        className="space-y-5 max-w-xl mx-auto p-6 border rounded-lg"
      >
        <h2 className="text-2xl font-semibold">Create a post</h2>

        {/* Title */}
        <FormField
          control={form.control}
          name="title"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Title</FormLabel>
              <FormControl>
                <Input placeholder="A catchy headline" {...field} />
              </FormControl>
              <FormDescription>3 to 100 characters.</FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />

        {/* Body */}
        <FormField
          control={form.control}
          name="body"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Body</FormLabel>
              <FormControl>
                <Textarea
                  rows={6}
                  placeholder="What is on your mind?"
                  {...field}
                />
              </FormControl>
              <FormDescription>10 to 1000 characters.</FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />

        {/* Tags */}
        <FormField
          control={form.control}
          name="tags"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Tags</FormLabel>
              <FormControl>
                <Input placeholder="react, typescript, tutorial" {...field} />
              </FormControl>
              <FormDescription>
                Optional. Comma-separated, no empty entries.
              </FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />

        <Button
          type="submit"
          disabled={form.formState.isSubmitting}
          className="w-full"
        >
          {form.formState.isSubmitting ? "Publishing..." : "Publish post"}
        </Button>
      </form>
    </Form>
  );
}

export default CreatePostForm;
```

Drop `<CreatePostForm />` into `App.tsx`, run the dev server, and try:

- Submit empty — three red messages appear, each generated by Zod, rendered by `<FormMessage />` automatically.
- Type `hi` in title — the message becomes "Title must be at least 3 characters".
- Type `react,,typescript` in tags — the `refine` rule fires.
- Fill everything in correctly and submit — the button shows **Publishing...**, the form resets, and the console logs a properly typed object with tags already split into an array.

### Step 3 — read the data flow

```
User types in <Input>
        |
        v
RHF stores the value (no re-render of the parent)
        |
        v
User clicks Publish
        |
        v
form.handleSubmit -> zodResolver runs the Zod schema
        |                                    \
        |                                     v
        v                              schema fails
   schema passes                              |
        |                                     v
        v                            FormMessage renders the
   onSubmit(data) runs                Zod error next to the field
   data is fully typed (PostFormValues)
```

---

## 44.9 Why the `<Form>` Pattern Wins

| Hand-rolled (Class 43) | shadcn `<Form>` (this class) |
|------------------------|------------------------------|
| `register("name", { required: ... })` in JSX | Rules live in the Zod schema |
| `{errors.name && <p>...</p>}` for every field | `<FormMessage />` — one line, automatic |
| Manually link `<Label htmlFor>` to `<Input id>` | `<FormLabel>` and `<FormControl>` link automatically |
| Validation rules scattered across the file | One schema, easy to reuse |
| Type the form values by hand | `z.infer` derives them |

This is the pattern you will use for every form in the rest of the course — the capstone in Classes 50 and 51 uses it heavily.

---

## Practice Exercises

### Exercise 1: Sign-up form with confirm password
Build a `<SignUpForm>` with a Zod schema:
- `email` — required, must be an email
- `password` — required, minimum 8 characters, must contain at least one number
- `confirmPassword` — must match `password` (use `.refine` with `path: ["confirmPassword"]`)
- `acceptTerms` — `z.literal(true, { errorMap: () => ({ message: "You must accept the terms" }) })` paired with a shadcn `<Checkbox>`
- Render with the full `<Form>` primitive
- `console.log` the data on submit and reset the form

### Exercise 2: Edit-post form
Reuse the `postSchema` from this class. Build an `<EditPostForm>` that:
- Accepts `initialPost: PostFormValues` as a prop
- Pre-fills the form via `defaultValues`
- Includes a **Cancel** button (calls `form.reset()`)
- Includes a **Save** button disabled while submitting

### Exercise 3: Profile form with optional URL
A schema with:
- `displayName` — 2 to 30 characters
- `bio` — optional, up to 200 characters
- `website` — optional, but if provided must be a valid URL (hint: `z.string().url().optional().or(z.literal(""))` to allow empty strings as well)
- Show a live character count under the `bio` field using `form.watch("bio")`

---

## Key Takeaways
- A Zod schema is a single source of truth for both validation rules and TypeScript types.
- `z.infer<typeof schema>` derives the form's TypeScript type — no manual sync.
- `.refine()` handles cross-field rules like password confirmation.
- `zodResolver(schema)` is the only line you need to connect Zod to React Hook Form.
- The shadcn `<Form>` family (`<FormField>`, `<FormItem>`, `<FormLabel>`, `<FormControl>`, `<FormDescription>`, `<FormMessage>`) removes nearly all the boilerplate around labels, layout and error messages.
- Keep schemas in `src/schemas/` and export the inferred type alongside them.
- **Next class**: we leave the local console and start talking to a real API with Axios and React Query.
