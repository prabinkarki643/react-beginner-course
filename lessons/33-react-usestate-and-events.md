# Class 33: useState & Event Handling

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 32 (lists & conditional rendering)

## What You Will Learn
- What "state" means in React and how it differs from a plain variable
- How to use the `useState` hook with an explicit TypeScript type argument
- The golden rule: **always update state immutably**
- The functional update form `setX(prev => ...)` and when you need it
- How to attach event handlers — `onClick`, `onChange`, `onSubmit`
- How to type event objects: `React.MouseEvent`, `React.ChangeEvent`, `React.FormEvent`

---

## 33.1 What is State?

Up to now, every value in our components has been **fixed**. We hard-coded an array of users and rendered them. The page never changed.

A real app needs values that **change over time** as the user interacts with it:

- The number of unread emails
- The text in a search box
- Whether a sidebar is open or closed
- The list of tasks the user has added

These changing values are called **state**. When state changes, React re-runs the component function and updates the screen to match.

> **The key insight**: in React, you do not say "update the DOM". You change the state, and React works out what the DOM should look like next.

### Why not just a normal variable?

```tsx
function Counter() {
  let count = 0;

  return (
    <button onClick={() => { count = count + 1; console.log(count); }}>
      Count is {count}
    </button>
  );
}
```

Click the button. The console prints `1, 2, 3, ...` but the button text stays at `Count is 0`. Why?

Because React only re-renders when state changes. A normal variable change is invisible to React — the function has finished running, the variable is gone, and the DOM was never told to update.

We need a special kind of variable that **tells React** when it changes. That is what `useState` gives us.

---

## 33.2 The `useState` Hook

`useState` is a function from React called a **hook**. (All hooks start with `use`.) You import it from `react`:

```tsx
import { useState } from 'react';
```

### Basic usage

```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState<number>(0);

  return (
    <div style={{ padding: '1rem', fontFamily: 'system-ui' }}>
      <p style={{ fontSize: '1.5rem' }}>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

export default Counter;
```

Let us unpack the magic line:

```tsx
const [count, setCount] = useState<number>(0);
//      ^        ^                    ^      ^
//      |        |                    |      └─ initial value
//      |        |                    └─────── the type held in state
//      |        └──────────────────────────── the setter — call to update
//      └───────────────────────────────────── the current value
```

`useState<number>(0)` returns an array of two things: the current value and a setter function. We destructure them into `count` and `setCount`. The TypeScript type argument `<number>` tells the compiler that `count` is a `number` and `setCount` only accepts a `number`.

Click the button:

1. `setCount(count + 1)` queues a new value.
2. React re-runs the `Counter` function.
3. This time `useState` returns the new value.
4. The new JSX is produced and React updates the DOM.

### Type inference vs explicit types

If TypeScript can work out the type from the initial value, you can omit the `<number>`:

```tsx
const [count, setCount] = useState(0);            // inferred as number
const [name, setName]   = useState('');           // inferred as string
const [open, setOpen]   = useState(false);        // inferred as boolean
```

But for **arrays, objects, or values that might be null**, always be explicit. Otherwise TypeScript will guess `never[]` (an array of nothing), and the first time you try to put a real item in, the compiler will complain.

```tsx
import type { User } from './types/user';

const [users, setUsers] = useState<User[]>([]);            // good
const [selected, setSelected] = useState<User | null>(null); // good
```

> **Rule of thumb**: always type your state explicitly when the initial value is `[]`, `{}` or `null`.

---

## 33.3 The Golden Rule: Update State Immutably

This is the single most common React beginner mistake. **Never mutate** the value held in state. Always create a **new** value and pass it to the setter.

### Numbers and strings are fine

Numbers and strings in JavaScript are already immutable, so `setCount(count + 1)` works. Easy.

### Objects — use the spread

```tsx
interface Profile {
  name: string;
  age: number;
}

const [profile, setProfile] = useState<Profile>({ name: 'Aarav', age: 21 });

// WRONG — mutates the object, React does not see a change
profile.age = 22;
setProfile(profile);

// RIGHT — new object with the updated field
setProfile({ ...profile, age: 22 });
```

The spread `...profile` copies all existing fields, and `age: 22` overrides one of them. The result is a brand-new object — React notices, re-renders, and the screen updates.

### Arrays — use `concat` / `map` / `filter`

Never `push`, `pop`, `splice` or `sort` an array in state. Use the non-mutating methods you learned in Class 20.

```tsx
const [tasks, setTasks] = useState<string[]>(['Buy milk']);

// Add an item — use spread or concat
setTasks([...tasks, 'Walk dog']);

// Remove an item — use filter
setTasks(tasks.filter((t) => t !== 'Buy milk'));

// Update an item — use map
setTasks(tasks.map((t) => (t === 'Buy milk' ? 'Buy oat milk' : t)));
```

Each of these returns a **new** array. React compares the old and new array references, sees they differ, and re-renders.

---

## 33.4 Functional Updates — `setX(prev => ...)`

When the new state depends on the **previous** state, prefer the **functional form**:

```tsx
setCount((c) => c + 1);
```

This passes a callback to the setter. React calls the callback with the latest value of `count` and uses the return value as the new state.

### Why does this matter?

State updates in React are **batched and asynchronous**. If you write:

```tsx
setCount(count + 1);
setCount(count + 1);
setCount(count + 1);
```

…you might expect the count to jump by 3. It will only jump by 1, because all three calls read the same stale `count`.

Use the functional form when you want guaranteed correctness:

```tsx
setCount((c) => c + 1);  // c is the latest committed value
setCount((c) => c + 1);
setCount((c) => c + 1);  // now jumps by 3
```

**Rule of thumb**: if the new state depends on the previous state, use `setX(prev => ...)`. If it does not, the direct form is fine.

---

## 33.5 Event Handlers

In React, events are camelCase attributes that take a function:

```tsx
<button onClick={handleClick}>Click me</button>
```

Two important rules:

1. **Pass the function, do not call it.** `onClick={handleClick}` is right; `onClick={handleClick()}` would call it during render and pass the result (probably `undefined`) to React.
2. **Use an arrow function to pass arguments.** When you need to pass data to the handler, wrap it:
   ```tsx
   <button onClick={() => deleteUser(user.id)}>Delete</button>
   ```

### A counter with three buttons

```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState<number>(0);

  const handleIncrement = () => setCount((c) => c + 1);
  const handleDecrement = () => setCount((c) => Math.max(0, c - 1));
  const handleReset     = () => setCount(0);

  return (
    <div style={{ padding: '1rem', fontFamily: 'system-ui' }}>
      <p style={{ fontSize: '1.5rem' }}>Count: {count}</p>
      <button onClick={handleDecrement} style={{ marginRight: '0.5rem' }}>−</button>
      <button onClick={handleIncrement} style={{ marginRight: '0.5rem' }}>+</button>
      <button onClick={handleReset}>Reset</button>
    </div>
  );
}

export default Counter;
```

The decrement uses `Math.max(0, c - 1)` to keep `count` from going below zero.

---

## 33.6 Typing Event Objects

React passes an event object to your handler. With TypeScript we type the parameter precisely so we get autocomplete and safety.

### Click event

```tsx
const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
  console.log('Clicked at', event.clientX, event.clientY);
};

<button onClick={handleClick}>Click me</button>
```

The generic `<HTMLButtonElement>` tells TS that `event.currentTarget` is a button element (which has a `.disabled` property, for example).

### Input change event

```tsx
import { useState } from 'react';

function NameInput() {
  const [name, setName] = useState<string>('');

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    setName(event.target.value);
  };

  return (
    <div style={{ padding: '1rem', fontFamily: 'system-ui' }}>
      <label htmlFor="name">Your name</label><br />
      <input id="name" type="text" value={name} onChange={handleChange} />
      <p>Hello, {name || '(stranger)'}!</p>
    </div>
  );
}

export default NameInput;
```

This is the classic **controlled input** pattern: the input's `value` always reflects state, and every keystroke fires `onChange` which updates state, which re-renders the input. We will cover controlled forms properly in the next class.

### A handy table

| You write | Type for `event` |
|-----------|------------------|
| `onClick` on a `<button>` | `React.MouseEvent<HTMLButtonElement>` |
| `onClick` on a `<div>`    | `React.MouseEvent<HTMLDivElement>` |
| `onChange` on an `<input>` | `React.ChangeEvent<HTMLInputElement>` |
| `onChange` on a `<select>` | `React.ChangeEvent<HTMLSelectElement>` |
| `onChange` on a `<textarea>` | `React.ChangeEvent<HTMLTextAreaElement>` |
| `onSubmit` on a `<form>` | `React.FormEvent<HTMLFormElement>` |
| `onKeyDown` on an `<input>` | `React.KeyboardEvent<HTMLInputElement>` |

You will memorise the common ones with practice.

---

## 33.7 Putting It All Together — A Light/Dark Toggle

A small component that holds two pieces of state (an array of tasks and a colour mode) and demonstrates immutable updates.

```tsx
// src/TodoBoard.tsx
import { useState } from 'react';

interface Task {
  id: number;
  title: string;
  done: boolean;
}

function TodoBoard() {
  const [tasks, setTasks] = useState<Task[]>([
    { id: 1, title: 'Learn useState', done: true  },
    { id: 2, title: 'Build a counter', done: true  },
    { id: 3, title: 'Build a toggle',  done: false },
  ]);
  const [mode, setMode] = useState<'light' | 'dark'>('light');

  const toggleMode = () => setMode((m) => (m === 'light' ? 'dark' : 'light'));

  const toggleTask = (id: number) => {
    setTasks((prev) =>
      prev.map((t) => (t.id === id ? { ...t, done: !t.done } : t))
    );
  };

  const removeTask = (id: number) => {
    setTasks((prev) => prev.filter((t) => t.id !== id));
  };

  const styles = {
    background: mode === 'light' ? '#fff' : '#1f1f1f',
    color: mode === 'light' ? '#111' : '#eee',
    padding: '1.5rem',
    minHeight: '100vh',
    fontFamily: 'system-ui',
  };

  return (
    <div style={styles}>
      <button onClick={toggleMode}>
        Switch to {mode === 'light' ? 'dark' : 'light'} mode
      </button>

      <h2>Tasks ({tasks.filter((t) => !t.done).length} left)</h2>

      <ul style={{ listStyle: 'none', padding: 0 }}>
        {tasks.map((task) => (
          <li key={task.id} style={{ padding: '0.5rem 0' }}>
            <input
              type="checkbox"
              checked={task.done}
              onChange={() => toggleTask(task.id)}
            />
            <span
              style={{
                marginLeft: '0.5rem',
                textDecoration: task.done ? 'line-through' : 'none',
              }}
            >
              {task.title}
            </span>
            <button
              onClick={() => removeTask(task.id)}
              style={{ marginLeft: '0.75rem' }}
            >
              Remove
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default TodoBoard;
```

Use it in `App.tsx`:

```tsx
import TodoBoard from './TodoBoard';

function App() {
  return <TodoBoard />;
}

export default App;
```

Try clicking the toggle, ticking tasks, removing them. Every change goes through `setTasks` or `setMode` — we never mutate.

---

## Practice Exercises

1. **A clamped counter.** Build a counter component that:
   - starts at 0
   - has buttons for `−`, `+`, `Reset` and `×2`
   - never goes below 0 or above 20
   - uses the **functional update form** for `+` and `×2`
2. **Like button.** Build a `LikeButton` that holds a `liked: boolean` and a `count: number`. Clicking toggles `liked` and updates the count up by 1 when liking, down by 1 when unliking. The button label should be "♥ Like (3)" or "♡ Liked (4)" depending on state. Type all event handlers properly.
3. **Typed colour mode.** Build a `ThemeToggle` that holds `mode: 'light' | 'dark' | 'system'`. Show three buttons; clicking each sets the matching mode. Render the current mode as text below the buttons.
4. **Immutable update drill.** Given:
   ```tsx
   interface Book { id: number; title: string; read: boolean; }
   const [books, setBooks] = useState<Book[]>([
     { id: 1, title: 'Dune',        read: false },
     { id: 2, title: 'Brave New World', read: false },
   ]);
   ```
   Write three handlers:
   - `addBook(title: string)` — adds a new book with `read: false`
   - `toggleRead(id: number)` — flips the `read` field for that book
   - `removeBook(id: number)` — removes that book
   None of them may mutate `books`. Hook them up to buttons and an input in a small UI.

---

## Key Takeaways
- A component re-renders only when **state** changes — use `useState`, not plain variables.
- `useState<T>(initial)` returns `[value, setter]`. Type your state explicitly for arrays, objects and nullable values.
- **Never mutate** state. Use the spread for objects, `map`/`filter`/`concat` for arrays.
- Use `setX(prev => ...)` when the new state depends on the old one.
- Event handlers go in camelCase attributes: `onClick`, `onChange`, `onSubmit`. Pass the function, do not call it.
- Type event objects precisely: `React.MouseEvent<HTMLButtonElement>`, `React.ChangeEvent<HTMLInputElement>`, etc.
- Next class: we apply all of this to **controlled forms** with multiple fields and validation.
