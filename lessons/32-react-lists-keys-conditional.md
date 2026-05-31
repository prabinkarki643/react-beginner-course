# Class 32: Lists, Keys & Conditional Rendering

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 31 (props & composition)

## What You Will Learn
- How to render an array of data with `.map()` inside JSX
- Why every rendered list item needs a `key` and how to choose a good one
- Three ways to show or hide content: `&&`, the ternary `? :` and early `return`
- How to write a clean empty-state UI for lists with no data
- The classic bug: using the array index as a key when items can reorder

---

## 32.1 Rendering Arrays with `.map()`

In Class 20 you learned that `.map()` transforms an array into another array. In React we use it to transform an **array of data** into an **array of JSX elements**.

### Static example

```tsx
function CourseList() {
  const courses: string[] = ['HTML', 'CSS', 'JavaScript', 'TypeScript', 'React'];

  return (
    <ul>
      {courses.map((course) => (
        <li key={course}>{course}</li>
      ))}
    </ul>
  );
}

export default CourseList;
```

Open the rendered page and you will see a five-item bulleted list.

What is happening:

1. `courses` is an array of five strings.
2. `.map()` calls a function for each string and returns a new array — this time of `<li>` elements.
3. JSX is happy to render an array directly: each `<li>` is placed in the DOM in order.

The curly braces around `{courses.map(...)}` are just the usual "switch to TS" braces from Class 30.

### Realistic example — typed objects

A real list rarely contains plain strings. It usually contains objects. Let us model a list of users with TypeScript.

```tsx
// src/types/user.ts
export interface User {
  id: number;
  name: string;
  email: string;
  isAdmin: boolean;
}
```

```tsx
// src/UserList.tsx
import type { User } from './types/user';

interface UserListProps {
  users: User[];
}

function UserList({ users }: UserListProps) {
  return (
    <ul style={{ listStyle: 'none', padding: 0 }}>
      {users.map((user) => (
        <li
          key={user.id}
          style={{
            padding: '0.75rem 1rem',
            border: '1px solid #ddd',
            borderRadius: '8px',
            marginBottom: '0.5rem',
          }}
        >
          <strong>{user.name}</strong> — {user.email}
        </li>
      ))}
    </ul>
  );
}

export default UserList;
```

```tsx
// src/App.tsx
import UserList from './UserList';
import type { User } from './types/user';

function App() {
  const users: User[] = [
    { id: 1, name: 'Aarav Shrestha', email: 'aarav@example.com', isAdmin: true  },
    { id: 2, name: 'Sita Sharma',    email: 'sita@example.com',  isAdmin: false },
    { id: 3, name: 'Ravi Karki',     email: 'ravi@example.com',  isAdmin: false },
    { id: 4, name: 'Maya Rai',       email: 'maya@example.com',  isAdmin: true  },
    { id: 5, name: 'Bikash Lama',    email: 'bikash@example.com', isAdmin: false },
  ];

  return (
    <div style={{ padding: '2rem', maxWidth: '600px', fontFamily: 'system-ui' }}>
      <h1>Our team</h1>
      <UserList users={users} />
    </div>
  );
}

export default App;
```

Because `users` is typed as `User[]`, TypeScript automatically knows that `user` inside `.map()` is a `User` — you get autocompletion on `user.name`, `user.email`, etc. Free safety.

---

## 32.2 The `key` Prop — Why It Matters

Every element rendered in a list **must have a `key` prop**, and that key must be **unique among its siblings**.

```tsx
{users.map((user) => (
  <li key={user.id}>{user.name}</li>
))}
```

If you forget the key, React will log a warning in the console:

```
Warning: Each child in a list should have a unique "key" prop.
```

### What is the key for?

React uses keys to track which list items are the same between re-renders. When the data changes — items added, removed or reordered — React looks at the keys and works out the minimum amount of DOM work to do.

Without keys (or with bad keys), React falls back to comparing items by position. That is fine for a static list but causes very strange bugs the moment the list changes.

### What makes a good key?

A good key is:

1. **Unique** within the list.
2. **Stable** — does not change between renders for the same item.

In order of preference:

| Preference | Source of the key | When to use |
|-----------:|-------------------|-------------|
| 1 | A unique `id` field from the data (`user.id`, `post.id`) | Almost always |
| 2 | A unique natural string (an email, a slug, an ISBN) | When you genuinely have no `id` |
| 3 | The array index (`(item, index) => index`) | Only for **static**, never-changing lists |

**Never** use `Math.random()` as a key — it changes every render and defeats the whole point.

---

## 32.3 The "Index as Key" Bug

It is tempting to write:

```tsx
{users.map((user, index) => (
  <li key={index}>{user.name}</li>
))}
```

For a static list, that works. But the moment the list can reorder, add or remove items, you get subtle bugs.

### Why? A worked example

Imagine a to-do list rendered with checkbox inputs:

```tsx
const tasks = [
  { id: 'a', label: 'Buy milk' },
  { id: 'b', label: 'Walk dog' },
  { id: 'c', label: 'Do laundry' },
];
```

The user ticks "Walk dog" (the second item). The checkbox is now ticked at **position 1**.

The user then deletes "Buy milk". The new array is:

```tsx
[
  { id: 'b', label: 'Walk dog' },
  { id: 'c', label: 'Do laundry' },
]
```

If we used `key={index}`, React thinks:

- Position 0 is the same as before → reuse the DOM node → it shows "Walk dog" but the **ticked state at position 0 is still empty** (because position 0 used to be unticked "Buy milk").
- Position 1 is the same as before → it shows "Do laundry" but with the **tick that was originally on "Walk dog"**.

The ticks have moved to the wrong rows. The user is rightly confused.

If we use `key={task.id}`, React correctly matches "Walk dog" to "Walk dog" by id, keeps its tick, removes "Buy milk", and life is good.

**Rule:** if the list is dynamic, use a stable `id` as the key.

---

## 32.4 Conditional Rendering — Three Patterns

A real UI must decide what to show based on state. There are three common patterns in React.

### Pattern 1 — Logical AND `&&`

`A && B` evaluates to `B` if `A` is truthy, and to `A` (a falsy value) if `A` is falsy. React renders falsy values as nothing. So this reads almost like English: "if the user is admin, show the badge".

```tsx
function UserRow({ user }: { user: User }) {
  return (
    <li>
      {user.name}
      {user.isAdmin && (
        <span style={{ marginLeft: '0.5rem', color: 'crimson' }}>(admin)</span>
      )}
    </li>
  );
}
```

If `user.isAdmin` is `true`, the badge is shown. If `false`, nothing is rendered.

> **Watch out**: `&&` evaluates from left to right and returns the first falsy value. So `{tasks.length && <Tasks />}` will render the literal `0` on screen when the list is empty — because `0` is falsy. Use `{tasks.length > 0 && <Tasks />}` instead, or a ternary.

### Pattern 2 — Ternary `condition ? a : b`

Use a ternary when you want to choose **between two things** to render.

```tsx
function StatusBadge({ online }: { online: boolean }) {
  return (
    <span style={{ color: online ? 'green' : 'grey' }}>
      {online ? 'Online' : 'Offline'}
    </span>
  );
}
```

You can nest ternaries, but be careful — readability suffers fast. If you have more than two branches, an `if/else` block above the `return` is usually clearer.

### Pattern 3 — Early `return`

When the whole component should render something completely different based on a condition, return early.

```tsx
interface UserListProps {
  users: User[];
}

function UserList({ users }: UserListProps) {
  if (users.length === 0) {
    return <p style={{ color: '#666' }}>No users yet. Add one above.</p>;
  }

  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

This is the standard pattern for **empty states**, which we will look at next.

### Quick chooser

| Situation | Pattern |
|-----------|---------|
| Show something only if a condition is true | `condition && <Thing />` |
| Choose between exactly two things | `condition ? <A /> : <B />` |
| Render a completely different layout when a condition holds | `if (...) return <Other />;` then the main `return` |

---

## 32.5 Empty-State UI

A polished app does not show an empty `<ul>` when there is no data. It shows a helpful message — sometimes with an icon or a "create your first item" button.

```tsx
// src/UserList.tsx
import type { User } from './types/user';

interface UserListProps {
  users: User[];
}

function UserList({ users }: UserListProps) {
  if (users.length === 0) {
    return (
      <div
        style={{
          padding: '2rem',
          textAlign: 'center',
          border: '1px dashed #ccc',
          borderRadius: '8px',
          color: '#666',
        }}
      >
        <p style={{ margin: 0, fontWeight: 600 }}>No users yet</p>
        <p style={{ margin: '0.5rem 0 0' }}>Add your first team member to get started.</p>
      </div>
    );
  }

  return (
    <ul style={{ listStyle: 'none', padding: 0 }}>
      {users.map((user) => (
        <li
          key={user.id}
          style={{
            padding: '0.75rem 1rem',
            border: '1px solid #ddd',
            borderRadius: '8px',
            marginBottom: '0.5rem',
          }}
        >
          <strong>{user.name}</strong> — {user.email}
          {user.isAdmin && (
            <span style={{ marginLeft: '0.5rem', color: 'crimson' }}>(admin)</span>
          )}
        </li>
      ))}
    </ul>
  );
}

export default UserList;
```

Try it both ways. Pass `users={[]}` from `App.tsx` and you should see the friendly empty-state card. Pass the array of five users and you should see the list. We will revisit empty/loading/error states properly in Class 47.

---

## 32.6 Putting It All Together

A small `UserList` that demonstrates `.map`, keys, `&&` for the admin badge, and an early return for the empty state.

```tsx
// src/types/user.ts
export interface User {
  id: number;
  name: string;
  email: string;
  isAdmin: boolean;
}
```

```tsx
// src/UserList.tsx (same as section 32.5)
```

```tsx
// src/App.tsx
import type { User } from './types/user';
import UserList from './UserList';

function App() {
  const users: User[] = [
    { id: 1, name: 'Aarav Shrestha', email: 'aarav@example.com', isAdmin: true  },
    { id: 2, name: 'Sita Sharma',    email: 'sita@example.com',  isAdmin: false },
    { id: 3, name: 'Ravi Karki',     email: 'ravi@example.com',  isAdmin: false },
    { id: 4, name: 'Maya Rai',       email: 'maya@example.com',  isAdmin: true  },
    { id: 5, name: 'Bikash Lama',    email: 'bikash@example.com', isAdmin: false },
  ];

  return (
    <div style={{ padding: '2rem', maxWidth: '600px', fontFamily: 'system-ui' }}>
      <h1>Our team ({users.length})</h1>
      <UserList users={users} />
    </div>
  );
}

export default App;
```

Notice we have not used `useState` yet — `users` is a plain const array. Everything we render is derived from that data. In the next class we will make this data **change over time**.

---

## Practice Exercises

1. **Books library.** Create an interface `Book { id: number; title: string; author: string; available: boolean; }`. In `App.tsx`, declare an array of 6 books. Build a `<BookList books={books} />` component that renders each book in a card. Show a green "Available" badge or a grey "Borrowed" badge using a ternary. Use `book.id` as the key.
2. **Empty state.** Modify your `BookList` so that if `books.length === 0` it returns an early "No books in the library yet." message. Test by passing an empty array.
3. **Filtered list with `&&`.** Above your `BookList`, render a small `<p>` that says "Some books are currently borrowed." — but **only** if at least one book has `available: false`. Use the `&&` pattern. Make sure you guard against the falsy-`0` trap (use `someExpression > 0`, not the raw count).
4. **Reorder bug demo.** Replace `key={book.id}` with `key={index}` in your `BookList`. Reorder the array (move the first book to the end). Open DevTools and inspect — what looks visually right? Now go back to `key={book.id}` and explain in a comment why ids are better. Save the comment.

---

## Key Takeaways
- Use `.map()` to turn an array of data into an array of JSX elements.
- Every list element needs a **stable, unique `key`** — prefer `item.id` over the array index.
- The "index as key" bug only shows up when the list changes. Avoid it from day one.
- For "show if true", use `condition && <X />` (mind the falsy-`0` trap).
- For "either A or B", use `condition ? <A /> : <B />`.
- For "different layout entirely", use an `if` block with an early `return`.
- Always design an **empty state** for any list — it is part of the UI.
- Next class: data that changes — we meet `useState` and event handlers.
