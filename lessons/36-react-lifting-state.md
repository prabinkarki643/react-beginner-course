# Class 36: Lifting State Up

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 33 (useState & events), Class 34 (controlled forms)

## What You Will Learn
- The "shared state" problem — when two components need the same value
- The rule: move state to the **closest common ancestor**
- How to pass values **down** as props and changes **up** as callback functions
- A worked example: a filter input in a parent that filters a list in a child
- Composing parents with `children` so they stay general and reusable
- When **not** to lift state (preview of Context, coming in Class 38)

---

## 36.1 The "Shared State" Problem

Consider a tiny app with two components:

- A `<SearchInput />` that has a text box.
- A `<UserList />` that shows users filtered by what the user has typed.

If we naively give each component its own state:

```tsx
function SearchInput() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}

function UserList() {
  const [query, setQuery] = useState('');     // a *different* query
  // ... how do we read SearchInput's query? We can't.
}
```

…the two components each own a `query`, but they cannot see each other's. There is no way for `<UserList />` to know what the user has typed.

We need **one** `query` that both can use.

### The rule

When two or more sibling components need the same piece of data, **move the state to the closest common ancestor** and pass it down to the siblings as props. This is called **lifting state up**.

```
       App (owns query)
        │
   ┌────┴────┐
   ▼         ▼
SearchInput  UserList
(reads +    (reads
 changes    query)
 query)
```

- Data flows **down** through props.
- Changes flow **up** through callback props (functions the parent gives the child to call).

---

## 36.2 Worked Example — Filtering a List

Let us build the example end-to-end. We will model a small contact list.

### Step 1 — types

```tsx
// src/types/contact.ts
export interface Contact {
  id: number;
  name: string;
  email: string;
}
```

### Step 2 — the search input child

`SearchInput` does not own the query any more. It receives the current value as a prop and a callback to report changes back up.

```tsx
// src/SearchInput.tsx
interface SearchInputProps {
  value: string;
  onChange: (next: string) => void;
}

function SearchInput({ value, onChange }: SearchInputProps) {
  return (
    <input
      type="search"
      placeholder="Search by name or email…"
      value={value}
      onChange={(e) => onChange(e.target.value)}
      style={{ padding: '0.5rem', width: '100%', maxWidth: '320px' }}
    />
  );
}

export default SearchInput;
```

A couple of things to notice:

- The prop is called `value`, not `query`. The component is general — it just receives a string. The name `query` is the parent's concern.
- `onChange` is typed as `(next: string) => void`. The child translates the raw `event.target.value` into a clean string before calling the parent — the parent does not need to know about DOM events.

### Step 3 — the user list child

`UserList` receives the filtered contacts already prepared, or it can filter itself. Either is fine; we will pass the raw list and the query and let the list filter.

For learning purposes it is cleaner to let the **parent** do the filtering and just pass an already-filtered list to the child. That keeps the child completely dumb.

```tsx
// src/ContactList.tsx
import type { Contact } from './types/contact';

interface ContactListProps {
  contacts: Contact[];
}

function ContactList({ contacts }: ContactListProps) {
  if (contacts.length === 0) {
    return <p style={{ color: '#666' }}>No matching contacts.</p>;
  }

  return (
    <ul style={{ listStyle: 'none', padding: 0 }}>
      {contacts.map((c) => (
        <li
          key={c.id}
          style={{
            padding: '0.5rem 0.75rem',
            border: '1px solid #ddd',
            borderRadius: '6px',
            marginBottom: '0.5rem',
          }}
        >
          <strong>{c.name}</strong> — {c.email}
        </li>
      ))}
    </ul>
  );
}

export default ContactList;
```

### Step 4 — the parent owns the state

```tsx
// src/App.tsx
import { useState } from 'react';
import type { Contact } from './types/contact';
import SearchInput from './SearchInput';
import ContactList from './ContactList';

const allContacts: Contact[] = [
  { id: 1, name: 'Aarav Shrestha', email: 'aarav@example.com' },
  { id: 2, name: 'Sita Sharma',    email: 'sita@example.com' },
  { id: 3, name: 'Ravi Karki',     email: 'ravi@example.com' },
  { id: 4, name: 'Maya Rai',       email: 'maya@example.com' },
  { id: 5, name: 'Bikash Lama',    email: 'bikash@example.com' },
];

function App() {
  const [query, setQuery] = useState<string>('');

  const filtered: Contact[] = allContacts.filter((c) => {
    const q = query.trim().toLowerCase();
    if (q === '') return true;
    return c.name.toLowerCase().includes(q) || c.email.toLowerCase().includes(q);
  });

  return (
    <div style={{ padding: '2rem', maxWidth: 480, fontFamily: 'system-ui' }}>
      <h1>Contacts</h1>
      <SearchInput value={query} onChange={setQuery} />
      <p style={{ color: '#666', fontSize: '0.9rem' }}>
        Showing {filtered.length} of {allContacts.length}
      </p>
      <ContactList contacts={filtered} />
    </div>
  );
}

export default App;
```

Type in the search box. The list filters live.

### How the data flows

```
App holds query
       │
       │ value={query}
       ▼
SearchInput ─── onChange ───▶  setQuery (in App)
                                   │
                                   │ triggers re-render
                                   ▼
                              filter(allContacts)
                                   │
                                   │ contacts={filtered}
                                   ▼
                              ContactList renders
```

`SearchInput` does not know anything about contacts, and `ContactList` does not know anything about searching. They are reusable pieces. `App` is the brain.

---

## 36.3 Filtering Logic — Parent or Child?

Both options work; the trade-off is reusability.

| Parent filters, child gets `contacts: Contact[]` | Child filters, child gets `contacts` + `query` |
|---|---|
| Child stays completely general — reuse it anywhere | Child is coupled to "filterable list" idea |
| Filtering logic is visible in `App` — easy to change | Filtering hidden inside child |
| Easy to switch to a derived "selected" or "sorted" list later | Slightly less code for one-off cases |

For this course, lean towards the first option. **Compute derived data in the parent, pass the result down.** Children should be as simple as possible.

---

## 36.4 Composing with `children`

Sometimes the lifted state and the lifted layout get tangled. The `children` prop helps you keep them separate.

Here is a `Panel` component that owns its own expanded/collapsed state and renders whatever you put inside it.

```tsx
// src/Panel.tsx
import { useState } from 'react';

interface PanelProps {
  title: string;
  children: React.ReactNode;
}

function Panel({ title, children }: PanelProps) {
  const [open, setOpen] = useState<boolean>(true);

  return (
    <section style={{ border: '1px solid #ddd', borderRadius: '8px', marginBottom: '1rem' }}>
      <header
        style={{ padding: '0.75rem 1rem', cursor: 'pointer', userSelect: 'none' }}
        onClick={() => setOpen((o) => !o)}
      >
        {open ? '▾' : '▸'} {title}
      </header>
      {open && <div style={{ padding: '1rem', borderTop: '1px solid #eee' }}>{children}</div>}
    </section>
  );
}

export default Panel;
```

Now any caller can drop arbitrary content inside, including more components that have lifted state:

```tsx
function App() {
  return (
    <div style={{ padding: '2rem', maxWidth: 480 }}>
      <Panel title="Contacts">
        <ContactListWithSearch />
      </Panel>
      <Panel title="Settings">
        <p>Theme, language, notifications…</p>
      </Panel>
    </div>
  );
}
```

`Panel` knows nothing about contacts or settings — it just provides the open/close behaviour. This is composition with state, and it scales beautifully.

---

## 36.5 When NOT to Lift

Lifting state is the right answer most of the time, but there are two cases where it starts to hurt.

### Problem 1 — Lifting too high

If only two leaves of a deep tree need a piece of data, do not lift it all the way to `App`. That forces every component in between to forward props it does not care about — the classic **prop drilling** problem.

```
            App
             │
       ┌─────┴─────┐
     Layout      Sidebar
       │
       Main
       │
       Section
       │
       Card
       │
       Inner
       │
       Leaf    ← only Leaf and Sidebar actually need the value
```

Lifting to `App` means `Layout`, `Main`, `Section`, `Card` and `Inner` all forward the prop. Painful.

### Problem 2 — Lifting state that does not need lifting

If only **one** component needs the value, keep the state inside that component. Lifting is for **sharing**.

### What helps

For values that many components across the tree need (like the current theme, the logged-in user, or the locale) we will use the **Context API** in Class 38. Context lets any component subscribe to a value without prop drilling. For everything else, lifting state to the closest common ancestor remains the right tool.

> **Mantra**: lift to the *closest* common ancestor, not the top of the tree.

---

## 36.6 Bigger Example — Lifting a Form

Let us lift state across three components so the same piece of data is read and written from different places.

```tsx
// src/PriceFilter.tsx
interface PriceFilterProps {
  min: number;
  max: number;
  onChange: (next: { min: number; max: number }) => void;
}

function PriceFilter({ min, max, onChange }: PriceFilterProps) {
  return (
    <div style={{ display: 'flex', gap: '0.5rem', alignItems: 'center', marginBottom: '1rem' }}>
      <label>
        Min&nbsp;
        <input
          type="number"
          value={min}
          onChange={(e) => onChange({ min: Number(e.target.value), max })}
          style={{ width: 80 }}
        />
      </label>
      <label>
        Max&nbsp;
        <input
          type="number"
          value={max}
          onChange={(e) => onChange({ min, max: Number(e.target.value) })}
          style={{ width: 80 }}
        />
      </label>
    </div>
  );
}

export default PriceFilter;
```

```tsx
// src/ProductGrid.tsx
export interface Product {
  id: number;
  name: string;
  price: number;
}

interface ProductGridProps {
  products: Product[];
}

function ProductGrid({ products }: ProductGridProps) {
  if (products.length === 0) {
    return <p style={{ color: '#666' }}>Nothing matches your price range.</p>;
  }
  return (
    <ul style={{ listStyle: 'none', padding: 0 }}>
      {products.map((p) => (
        <li
          key={p.id}
          style={{
            padding: '0.5rem 0.75rem',
            border: '1px solid #ddd',
            borderRadius: '6px',
            marginBottom: '0.5rem',
            display: 'flex',
            justifyContent: 'space-between',
          }}
        >
          <span>{p.name}</span>
          <strong>£{p.price}</strong>
        </li>
      ))}
    </ul>
  );
}

export default ProductGrid;
```

```tsx
// src/App.tsx
import { useState } from 'react';
import PriceFilter from './PriceFilter';
import ProductGrid, { type Product } from './ProductGrid';

const catalogue: Product[] = [
  { id: 1, name: 'Wired Mouse',     price: 12  },
  { id: 2, name: 'Wireless Mouse',  price: 28  },
  { id: 3, name: 'Mechanical KB',   price: 95  },
  { id: 4, name: 'USB Hub',         price: 18  },
  { id: 5, name: '4K Monitor',      price: 350 },
];

function App() {
  const [range, setRange] = useState<{ min: number; max: number }>({ min: 0, max: 1000 });

  const visible: Product[] = catalogue.filter((p) => p.price >= range.min && p.price <= range.max);

  return (
    <div style={{ padding: '2rem', maxWidth: 480, fontFamily: 'system-ui' }}>
      <h1>Shop</h1>
      <PriceFilter min={range.min} max={range.max} onChange={setRange} />
      <p style={{ color: '#666' }}>{visible.length} item(s) shown</p>
      <ProductGrid products={visible} />
    </div>
  );
}

export default App;
```

The single piece of state — the `{ min, max }` object — lives in `App`. The filter writes to it, the grid reads from it indirectly via the derived `visible` array. No duplicate sources of truth.

---

## Practice Exercises

1. **Light/dark theme.** Build an `App` that holds a `theme: 'light' | 'dark'` in state. Pass it to `<ThemeToggle />` (which has a button to flip it) and to `<Page />` (which uses inline `style` to colour its background and text). Both components must read the same theme state.
2. **Tabs.** Build a `<Tabs />` parent that owns `activeTab: string`. It renders a `<TabBar />` (three buttons that call back to set the active tab) and a `<TabContent />` (shows a different paragraph depending on the active tab). Type the callbacks properly.
3. **Filtered to-do list.** Build a parent that owns: `tasks: Task[]` and `filter: 'all' | 'active' | 'done'`. Pass `filter` and a setter to `<FilterBar />` and pass the **filtered tasks** to `<TaskList />`. The parent computes the visible list with a `.filter()`.
4. **Bug hunt.** Take Exercise 1 and deliberately give `<ThemeToggle />` its own local `useState<'light' | 'dark'>('light')` as well. Click the toggle and observe: the toggle's own UI flips, but the page does not. Explain in a comment what is happening. Then remove the local state and confirm the bug is gone.

---

## Key Takeaways
- When siblings need the same data, **move state up to the closest common ancestor**.
- Data flows **down** as props; changes flow **up** as callback props.
- Keep children dumb where possible — compute derived data (like filtered lists) in the parent.
- `children` lets parents stay general while their callers choose what goes inside.
- Do not lift further than necessary — too much lifting leads to prop drilling.
- For data that lives across the whole app, we will use **Context** in Class 38.
- Next class: extracting reusable stateful logic into our own **custom hooks**.
