# Class 34: Controlled Forms in React

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 33 (useState & events)

## What You Will Learn
- The difference between a **controlled** and an **uncontrolled** input
- The "single source of truth" pattern — React state owns the value
- How to manage many fields with one state object (instead of one per field)
- Simple inline validation: required fields, minimum length, matching values
- How to handle form submission cleanly with `event.preventDefault()`
- How to type the form submit event: `React.FormEvent<HTMLFormElement>`

---

## 34.1 Controlled vs Uncontrolled Inputs

There are two ways an HTML input can live inside a React app.

### Uncontrolled (the browser owns the value)

```tsx
function UncontrolledInput() {
  return <input type="text" defaultValue="Hello" />;
}
```

The browser stores the current value internally. React does not know what it is until you read it (usually with a `ref`). This is fine for very simple cases but harder to validate or transform.

### Controlled (React owns the value)

```tsx
import { useState } from 'react';

function ControlledInput() {
  const [name, setName] = useState<string>('');

  return (
    <input
      type="text"
      value={name}
      onChange={(e) => setName(e.target.value)}
    />
  );
}
```

The `value` prop is bound to state. The `onChange` handler updates state on every keystroke. React re-renders, the input shows the new value.

Why is controlled the right default in React?

- You can **validate on the fly** (length, format, regex).
- You can **transform the value** (force uppercase, strip spaces).
- You can **disable submit** when the value is invalid.
- The state is your single source of truth — easy to inspect, save, restore.

> **Rule**: in this course, every input is controlled. Always pair `value` with `onChange`.

---

## 34.2 The Single Source of Truth

In a controlled input, the **state is the truth**. The DOM input is just a window onto that state.

```
   User types
        ↓
   onChange fires
        ↓
   setName(newValue)         ← state updated
        ↓
   Component re-renders
        ↓
   <input value={name} />    ← DOM input now shows the new state
```

The cycle is: **state → DOM**, never the other way round. Because of this, the DOM input cannot drift out of sync with state. There is only one place to look when something is wrong: the state.

---

## 34.3 A Tiny Single-Field Form

Let us build a "search box" — one input and a submit button.

```tsx
// src/SearchBox.tsx
import { useState } from 'react';

function SearchBox() {
  const [query, setQuery] = useState<string>('');

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();          // stop the page reloading
    console.log('Searching for', query);
    // ...do the search here...
  };

  return (
    <form
      onSubmit={handleSubmit}
      style={{ display: 'flex', gap: '0.5rem', padding: '1rem' }}
    >
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search…"
        style={{ flex: 1, padding: '0.5rem' }}
      />
      <button type="submit">Search</button>
    </form>
  );
}

export default SearchBox;
```

Two things to notice:

1. **`event.preventDefault()`** stops the browser from doing its default behaviour (a full-page reload when a form is submitted). In a React app we almost always want to handle the submission ourselves.
2. The submit handler is typed as `React.FormEvent<HTMLFormElement>`. The generic tells TypeScript that the form element is an `HTMLFormElement`.

---

## 34.4 Multi-field Forms — One State Object

A sign-up form might have four fields: name, email, password, confirm-password. You could use four separate `useState` calls, but a single state object is cleaner.

```tsx
// src/SignupForm.tsx
import { useState } from 'react';

interface SignupValues {
  name: string;
  email: string;
  password: string;
  confirm: string;
}

const initialValues: SignupValues = {
  name: '',
  email: '',
  password: '',
  confirm: '',
};

function SignupForm() {
  const [values, setValues] = useState<SignupValues>(initialValues);

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = event.target;
    setValues((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    console.log('Submitting', values);
    setValues(initialValues);    // clear the form
  };

  return (
    <form onSubmit={handleSubmit} style={{ display: 'grid', gap: '0.75rem', maxWidth: 320 }}>
      <label>
        Name
        <input
          name="name"
          type="text"
          value={values.name}
          onChange={handleChange}
        />
      </label>

      <label>
        Email
        <input
          name="email"
          type="email"
          value={values.email}
          onChange={handleChange}
        />
      </label>

      <label>
        Password
        <input
          name="password"
          type="password"
          value={values.password}
          onChange={handleChange}
        />
      </label>

      <label>
        Confirm password
        <input
          name="confirm"
          type="password"
          value={values.confirm}
          onChange={handleChange}
        />
      </label>

      <button type="submit">Sign up</button>
    </form>
  );
}

export default SignupForm;
```

The trick is the **shared `handleChange`**. Every input has:

- A `name` attribute (`name="email"`, `name="password"`).
- A `value` from state (`value={values.email}`).
- The same `onChange` callback.

Inside `handleChange` we read `event.target.name` and update the matching field with a computed property: `[name]: value`. One handler, any number of fields.

The pattern `setValues((prev) => ({ ...prev, [name]: value }))` is worth memorising — it is the canonical "update one field of a state object" move.

> **Why the extra parentheses around the object?** In an arrow function, `() => { ... }` is a function body. `() => ({ ... })` returns an object literal. The parentheses tell TS we mean the object, not a block.

---

## 34.5 Inline Validation

Validation comes in three flavours: **on submit**, **on change** (also called live validation) and **on blur** (when the field loses focus). For a first pass, on-submit and live-validation together work well.

Let us add live errors to the sign-up form.

```tsx
// src/SignupForm.tsx
import { useState } from 'react';

interface SignupValues {
  name: string;
  email: string;
  password: string;
  confirm: string;
}

interface SignupErrors {
  name?: string;
  email?: string;
  password?: string;
  confirm?: string;
}

const initialValues: SignupValues = {
  name: '',
  email: '',
  password: '',
  confirm: '',
};

function validate(values: SignupValues): SignupErrors {
  const errors: SignupErrors = {};

  if (values.name.trim().length === 0) {
    errors.name = 'Name is required.';
  }

  if (!values.email.includes('@')) {
    errors.email = 'Please enter a valid email address.';
  }

  if (values.password.length < 8) {
    errors.password = 'Password must be at least 8 characters.';
  }

  if (values.confirm !== values.password) {
    errors.confirm = 'Passwords do not match.';
  }

  return errors;
}

function SignupForm() {
  const [values, setValues] = useState<SignupValues>(initialValues);
  const [submitted, setSubmitted] = useState<boolean>(false);

  const errors = validate(values);
  const hasErrors = Object.keys(errors).length > 0;

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = event.target;
    setValues((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    setSubmitted(true);
    if (hasErrors) return;            // do not submit if invalid
    console.log('Submitting', values);
    setValues(initialValues);
    setSubmitted(false);
  };

  const errorStyle: React.CSSProperties = {
    color: 'crimson',
    fontSize: '0.85rem',
    marginTop: '0.25rem',
  };

  return (
    <form
      onSubmit={handleSubmit}
      noValidate
      style={{ display: 'grid', gap: '0.75rem', maxWidth: 320, fontFamily: 'system-ui' }}
    >
      <label>
        Name
        <input name="name" type="text" value={values.name} onChange={handleChange} />
        {submitted && errors.name && <div style={errorStyle}>{errors.name}</div>}
      </label>

      <label>
        Email
        <input name="email" type="email" value={values.email} onChange={handleChange} />
        {submitted && errors.email && <div style={errorStyle}>{errors.email}</div>}
      </label>

      <label>
        Password
        <input name="password" type="password" value={values.password} onChange={handleChange} />
        {submitted && errors.password && <div style={errorStyle}>{errors.password}</div>}
      </label>

      <label>
        Confirm password
        <input name="confirm" type="password" value={values.confirm} onChange={handleChange} />
        {submitted && errors.confirm && <div style={errorStyle}>{errors.confirm}</div>}
      </label>

      <button type="submit" disabled={submitted && hasErrors}>
        Sign up
      </button>
    </form>
  );
}

export default SignupForm;
```

Key decisions:

- `errors` is **derived state**: we compute it from `values` every render. We do not store it in `useState` because that would mean keeping it in sync manually.
- We show errors only after the user has tried to submit at least once (`submitted` flag). This avoids shouting at someone before they have even finished typing.
- `noValidate` on the `<form>` turns off the browser's built-in HTML5 validation tooltips so our custom messages take over.
- The submit button is disabled when there are errors.

In Classes 43–44 we will replace this hand-rolled approach with **React Hook Form + Zod**, which scales much better. For now, doing it by hand teaches you what is going on underneath.

---

## 34.6 Other Controlled Input Types

The same `value` + `onChange` pattern works for every form control. The key differences are which property holds the value and what type the event has.

### Checkbox

```tsx
const [agree, setAgree] = useState<boolean>(false);

<label>
  <input
    type="checkbox"
    checked={agree}
    onChange={(e) => setAgree(e.target.checked)}
  />
  I agree to the terms.
</label>
```

For checkboxes use `checked` (not `value`) and read `e.target.checked` (not `.value`).

### Radio buttons

```tsx
const [size, setSize] = useState<'S' | 'M' | 'L'>('M');

<label>
  <input type="radio" name="size" value="S" checked={size === 'S'} onChange={() => setSize('S')} /> Small
</label>
<label>
  <input type="radio" name="size" value="M" checked={size === 'M'} onChange={() => setSize('M')} /> Medium
</label>
<label>
  <input type="radio" name="size" value="L" checked={size === 'L'} onChange={() => setSize('L')} /> Large
</label>
```

### Select

```tsx
const [country, setCountry] = useState<string>('np');

<select
  value={country}
  onChange={(e: React.ChangeEvent<HTMLSelectElement>) => setCountry(e.target.value)}
>
  <option value="np">Nepal</option>
  <option value="in">India</option>
  <option value="uk">United Kingdom</option>
</select>
```

### Textarea

```tsx
const [bio, setBio] = useState<string>('');

<textarea
  value={bio}
  onChange={(e: React.ChangeEvent<HTMLTextAreaElement>) => setBio(e.target.value)}
  rows={4}
/>
```

In React (unlike HTML), `<textarea>` uses `value`, not text between the tags. This makes the controlled pattern consistent across all inputs.

---

## 34.7 A Quick Note on Accessibility

Two small things will make your forms much more usable:

1. **Every input needs a `<label>`.** Either wrap the input in `<label>` (like our examples) or use `<label htmlFor="id">` with a matching `id` on the input.
2. **Use the right `type`.** `type="email"` gives mobile users the right keyboard. `type="password"` masks the value. `type="number"` validates and shows a numeric keypad.

We will come back to accessibility throughout the course.

---

## Practice Exercises

1. **Length counter.** Build a `MessageBox` component with a `<textarea>` and a paragraph below it showing "X / 200 characters". When the user passes 200 characters, the text turns red and the submit button is disabled.
2. **Login form.** Build a `LoginForm` with `email` and `password` fields, controlled. On submit (with `preventDefault()`), log the values to the console and clear the form. Add an inline error "Email is required" if email is empty when submitted.
3. **Sign-up with full validation.** Reproduce the sign-up form from section 34.5 in your own project. Add one extra field, `age`, that must be a number ≥ 16. Show the error only after the first submit attempt.
4. **Mixed form.** Build a "register for an event" form with: name (text), email (email), number of guests (number, 1–10), preferred slot (radio: Morning / Afternoon / Evening), agree to terms (checkbox). Use **one state object** for all fields. Disable the submit button until all required fields are valid and the checkbox is ticked.

---

## Key Takeaways
- A **controlled input** has `value` driven by state and `onChange` to update it. React is the single source of truth.
- For multi-field forms, prefer **one state object** and a **shared `handleChange`** that uses the input's `name`.
- The classic update move: `setValues(prev => ({ ...prev, [name]: value }))`.
- Always call `event.preventDefault()` in the form submit handler.
- Compute **derived errors** every render instead of storing them in state — it cannot go out of sync.
- Type the events: `React.ChangeEvent<HTMLInputElement>` and `React.FormEvent<HTMLFormElement>`.
- Next class: we leave the world of pure UI and meet **side effects** with `useEffect`.
