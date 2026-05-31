# Class 31: Components, Props & Composition

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 30 (JSX & components)

## What You Will Learn
- What props are and why they are the bedrock of reusable components
- How to type props with a `ComponentNameProps` interface
- How to give props default values cleanly with destructuring defaults
- The special `children` prop and its type `React.ReactNode`
- How to compose components together (the "container" pattern)
- How to split a page-sized UI into small, well-typed pieces

---

## 31.1 Props — Inputs to a Component

In the last class our `Greeting` component always said the same thing. That is fine for a single static greeting, but if we want to greet ten different people we need a way to pass **data into** the component.

That mechanism is called **props** (short for "properties"). Think of props as **function arguments for a component**.

A function:

```tsx
function greet(name: string): string {
  return `Hello, ${name}!`;
}
greet('Aarav');
```

A component is the same idea, but it returns JSX and the arguments come in as a single object called `props`:

```tsx
function Greeting(props: { name: string }) {
  return <h2>Hello, {props.name}!</h2>;
}

// Use it
<Greeting name="Aarav" />
<Greeting name="Sita" />
```

In JSX, props look exactly like HTML attributes. `name="Aarav"` is the same as calling `Greeting({ name: 'Aarav' })`.

### The destructuring shortcut

Writing `props.name` everywhere gets repetitive. Almost every React codebase destructures the props object right in the parameter list:

```tsx
function Greeting({ name }: { name: string }) {
  return <h2>Hello, {name}!</h2>;
}
```

This is exactly the same component, just neater. From now on we will always destructure.

---

## 31.2 Typing Props with an Interface

Inline types like `{ name: string }` are fine for one prop, but for components with several props it gets messy. The convention is to define a named `interface` called `ComponentNameProps`.

```tsx
// src/Greeting.tsx
interface GreetingProps {
  name: string;
  age: number;
  isAdmin: boolean;
}

function Greeting({ name, age, isAdmin }: GreetingProps) {
  return (
    <div style={{ padding: '1rem', border: '1px solid #ccc', borderRadius: '8px' }}>
      <h2>Hello, {name}!</h2>
      <p>Age: {age}</p>
      <p>Role: {isAdmin ? 'Administrator' : 'Member'}</p>
    </div>
  );
}

export default Greeting;
```

Using it:

```tsx
// src/App.tsx
import Greeting from './Greeting';

function App() {
  return (
    <div style={{ padding: '2rem', fontFamily: 'system-ui' }}>
      <Greeting name="Aarav" age={21} isAdmin={true} />
      <Greeting name="Sita"  age={22} isAdmin={false} />
    </div>
  );
}

export default App;
```

Note the difference between **strings** and **other types** in JSX:

- Strings can use quotes: `name="Aarav"`
- Numbers, booleans, arrays and objects must use braces: `age={21}`, `isAdmin={true}`

If you try `age="21"` TypeScript will immediately complain — `age` is supposed to be a number, not a string. That is the safety net at work.

### Shorthand for booleans

If a boolean prop is `true`, you can omit the value:

```tsx
<Greeting name="Sita" age={22} isAdmin />  {/* same as isAdmin={true} */}
```

### Optional props

Add a `?` after the name to mark a prop as optional:

```tsx
interface GreetingProps {
  name: string;
  age?: number;       // optional
  isAdmin?: boolean;  // optional
}
```

Now `<Greeting name="Sita" />` is valid. Inside the component, `age` will be `number | undefined`. Always handle the `undefined` case — usually with a default value (next section).

---

## 31.3 Default Values for Props

You can give props sensible defaults right in the destructuring. This is the cleanest pattern in modern React.

```tsx
interface GreetingProps {
  name: string;
  greeting?: string;     // optional
  exclaim?: boolean;     // optional
}

function Greeting({ name, greeting = 'Hello', exclaim = false }: GreetingProps) {
  const punctuation: string = exclaim ? '!' : '.';
  return <h2>{greeting}, {name}{punctuation}</h2>;
}
```

Usage:

```tsx
<Greeting name="Aarav" />                            {/* Hello, Aarav. */}
<Greeting name="Sita" greeting="Namaste" />          {/* Namaste, Sita. */}
<Greeting name="Ravi" greeting="Hi" exclaim />       {/* Hi, Ravi! */}
```

> **Best practice**: Make optional props really feel optional by giving them sensible defaults. Then the component just works.

---

## 31.4 The `children` Prop

So far we have used self-closing components: `<Greeting />`. But what if you want to put **content inside** a component, the way you put `<p>` inside `<div>`?

That content is passed as a special prop called `children`. The type for "anything React can render" is `React.ReactNode`.

```tsx
// src/Card.tsx
interface CardProps {
  title: string;
  children: React.ReactNode;
}

function Card({ title, children }: CardProps) {
  return (
    <section
      style={{
        border: '1px solid #ddd',
        borderRadius: '8px',
        padding: '1rem',
        marginBottom: '1rem',
      }}
    >
      <h3 style={{ marginTop: 0 }}>{title}</h3>
      <div>{children}</div>
    </section>
  );
}

export default Card;
```

Using it:

```tsx
// src/App.tsx
import Card from './Card';

function App() {
  return (
    <div style={{ padding: '2rem', maxWidth: '600px' }}>
      <Card title="About me">
        <p>I am a BCA student at Triton College.</p>
        <p>I am learning React this semester.</p>
      </Card>

      <Card title="Contact">
        <p>Email: prabin@example.com</p>
        <p>Phone: +977 9800000000</p>
      </Card>
    </div>
  );
}

export default App;
```

Whatever you put between the opening and closing `<Card>` tags becomes the `children` prop inside the component. The `Card` decides where to render those children (in our case, inside the inner `<div>`).

This is called **composition** — building bigger things by nesting smaller ones.

> **The container pattern**: A component that wraps content (a `Card`, a `Modal`, a `Layout`, a `Panel`) almost always uses `children` so the caller can put anything inside it.

### What `React.ReactNode` actually accepts

`React.ReactNode` is React's "anything renderable" type. It allows:

- A string or number (`"hello"`, `42`)
- A JSX element (`<p>hi</p>`)
- An array of JSX elements (`[<li>a</li>, <li>b</li>]`)
- `null`, `undefined`, `false` (these render as nothing)
- Any combination of the above

That is why you can write whatever you like between `<Card>` and `</Card>` and it just works.

---

## 31.5 Composing Components

Once you have `Card` and `Greeting`, you can mix them freely:

```tsx
import Card from './Card';
import Greeting from './Greeting';

function App() {
  return (
    <div style={{ padding: '2rem', maxWidth: '600px' }}>
      <Card title="Welcome">
        <Greeting name="Aarav" greeting="Namaste" exclaim />
        <p>Glad to have you on board.</p>
      </Card>

      <Card title="Today's agenda">
        <ul>
          <li>Learn about props</li>
          <li>Learn about children</li>
          <li>Build a layout from small components</li>
        </ul>
      </Card>
    </div>
  );
}

export default App;
```

Notice that `Card` does not know — or care — that there is a `Greeting` inside it. It just renders whatever children it gets. That separation is what makes React UIs scale.

---

## 31.6 Splitting a Page into Components

Let us see this in action with a small "profile page" example. We will start with one big component and split it up.

### Before — one big `App`

```tsx
function App() {
  return (
    <div style={{ padding: '2rem', maxWidth: '700px', fontFamily: 'system-ui' }}>
      <header style={{ borderBottom: '1px solid #eee', paddingBottom: '1rem' }}>
        <h1>Sita Sharma</h1>
        <p>BCA Student · Triton College</p>
      </header>

      <section style={{ marginTop: '1.5rem' }}>
        <h2>About</h2>
        <p>I enjoy building web applications and reading detective novels.</p>
      </section>

      <section style={{ marginTop: '1.5rem' }}>
        <h2>Skills</h2>
        <ul>
          <li>HTML & CSS</li>
          <li>JavaScript / TypeScript</li>
          <li>React (learning now)</li>
        </ul>
      </section>

      <footer style={{ marginTop: '2rem', borderTop: '1px solid #eee', paddingTop: '1rem' }}>
        <p>© 2026 Sita Sharma</p>
      </footer>
    </div>
  );
}

export default App;
```

This works but it is long, hard to read, and not reusable.

### After — small typed components

```tsx
// src/ProfileHeader.tsx
interface ProfileHeaderProps {
  name: string;
  tagline: string;
}

function ProfileHeader({ name, tagline }: ProfileHeaderProps) {
  return (
    <header style={{ borderBottom: '1px solid #eee', paddingBottom: '1rem' }}>
      <h1>{name}</h1>
      <p>{tagline}</p>
    </header>
  );
}

export default ProfileHeader;
```

```tsx
// src/Section.tsx
interface SectionProps {
  title: string;
  children: React.ReactNode;
}

function Section({ title, children }: SectionProps) {
  return (
    <section style={{ marginTop: '1.5rem' }}>
      <h2>{title}</h2>
      {children}
    </section>
  );
}

export default Section;
```

```tsx
// src/PageFooter.tsx
interface PageFooterProps {
  owner: string;
  year?: number;
}

function PageFooter({ owner, year = new Date().getFullYear() }: PageFooterProps) {
  return (
    <footer style={{ marginTop: '2rem', borderTop: '1px solid #eee', paddingTop: '1rem' }}>
      <p>© {year} {owner}</p>
    </footer>
  );
}

export default PageFooter;
```

```tsx
// src/App.tsx
import ProfileHeader from './ProfileHeader';
import Section from './Section';
import PageFooter from './PageFooter';

function App() {
  return (
    <div style={{ padding: '2rem', maxWidth: '700px', fontFamily: 'system-ui' }}>
      <ProfileHeader name="Sita Sharma" tagline="BCA Student · Triton College" />

      <Section title="About">
        <p>I enjoy building web applications and reading detective novels.</p>
      </Section>

      <Section title="Skills">
        <ul>
          <li>HTML & CSS</li>
          <li>JavaScript / TypeScript</li>
          <li>React (learning now)</li>
        </ul>
      </Section>

      <PageFooter owner="Sita Sharma" />
    </div>
  );
}

export default App;
```

The `App.tsx` file is now far shorter and reads almost like a sentence. Each small component has one job, is well typed, and can be reused on another page tomorrow.

---

## 31.7 Choosing Props vs `children`

A common question: when should something be a named prop, and when should it be `children`?

| Use a named prop when... | Use `children` when... |
|--------------------------|------------------------|
| The value is a simple primitive (string, number, boolean) | The value is JSX / a block of content |
| You need many such values, each with a different role (`title`, `subtitle`, `image`) | You want flexibility — the caller decides what goes inside |
| The component will format the value (uppercase it, add an icon, etc.) | The component is just a wrapper (`Card`, `Modal`, `Layout`) |

A rule of thumb: **named props for data, `children` for content**.

---

## Practice Exercises

1. **Typed `<UserCard />`.** Create a `UserCard` component with this interface:
   ```tsx
   interface UserCardProps {
     name: string;
     email: string;
     role: 'Admin' | 'Editor' | 'Viewer';
     avatarUrl?: string;
   }
   ```
   Render the four pieces of data. If `avatarUrl` is missing, fall back to a placeholder like `https://i.pravatar.cc/64`. Render three different users in `App.tsx`.
2. **A `<Button>` with defaults.** Build a `Button` component that takes `label: string`, `variant?: 'primary' | 'secondary'` (default `'primary'`) and `disabled?: boolean` (default `false`). Style the two variants differently using inline `style`. Render four buttons in `App.tsx`, one of each variant and one disabled.
3. **`<Card title>` with `children`.** Recreate the `Card` component from section 31.4 and use it to display three cards: one with text, one with a list, and one with another component inside it.
4. **Refactor.** Take your `App.tsx` from Class 30 Exercise 4 and refactor it so every section is its own component with a typed `Props` interface. The new `App.tsx` should be no longer than 15 lines.

---

## Key Takeaways
- Props are **inputs** to a component, written like HTML attributes but typed by an interface.
- Convention: name the interface `ComponentNameProps`.
- Use `?` for optional props and **destructuring defaults** to give them sensible values.
- The `children` prop is everything you put between the opening and closing tags — type it as `React.ReactNode`.
- **Composition**: build big components by nesting small ones. Wrapper components almost always use `children`.
- A good UI is a tree of small, well-typed components, each with one clear job.
- Next class: we will learn how to render **lists** of data with `.map()` and the all-important `key` prop.
