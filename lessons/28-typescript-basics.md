# Class 28: TypeScript Basics

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Classes 16–25 (JavaScript), Class 26 (npm)

## What You Will Learn
- What TypeScript is and the problem it solves
- The difference between **compile-time** and **runtime** errors
- Setting up a tiny TypeScript project with `tsc --init`
- Annotating primitives, arrays, tuples and objects
- When to use `interface` and when to use `type`
- Union (`|`) and intersection (`&`) types, plus `?` optional and `readonly`
- Narrowing with `typeof`
- Typing function parameters and return values
- A gentle first look at generics
- The four utility types you will use every day: `Partial`, `Pick`, `Omit`, `Record`
- Why you should reach for `unknown` instead of `any`

---

## 28.1 Why TypeScript exists

In JavaScript, this code runs perfectly happily:

```js
function add(a, b) {
  return a + b;
}

add(5, "3"); // "53" — string concatenation, not addition!
```

JavaScript never warned you. It just produced a wrong answer. In a small playground that is no big deal; in a real app it is the kind of bug that takes two hours to find.

**TypeScript is JavaScript with a type-checker bolted on.** It is the same language, the same syntax, but you can annotate your variables, parameters and return values with the **type of value they should hold**. The type-checker then refuses to let your code run if those types do not line up.

```ts
function add(a: number, b: number): number {
  return a + b;
}

add(5, "3"); // ❌ Error: Argument of type 'string' is not assignable to parameter of type 'number'
```

The error appears in red squiggles in VS Code **before you save the file**.

> **Analogy:** TypeScript is a spell-checker for code. It catches the typos before you publish, but it cannot tell you whether your essay is any good — that part is still up to you.

Every modern React project uses TypeScript. We will write `.tsx` (not `.jsx`) for every component from Class 29 onwards.

---

## 28.2 Compile-time vs runtime errors

This is the single most useful distinction in this whole class.

| When | Where it shows up | Example |
|------|--------------------|---------|
| **Compile-time** | While you are *writing* the code (the editor's red squiggles), and again when `tsc` builds the project. | "You passed a string where a number was expected." |
| **Runtime** | When the JavaScript actually runs in the browser or Node. | "Cannot read properties of undefined (reading 'name')." |

TypeScript catches a whole category of bugs at compile time so they never reach the user. **It does not, and cannot, catch every bug.** If your code asks an API for data and the API is offline, that is still a runtime problem — no amount of typing will fix it.

The flow is:

```
You write:    greeter.ts         (TypeScript, with annotations)
Tool runs:    tsc                (TypeScript compiler)
Output:       greeter.js         (plain JavaScript — what actually runs)
```

The annotations **vanish** after compilation. The browser and Node never see them; they only ever execute JavaScript.

---

## 28.3 Setting up a TypeScript project

You do not strictly need this when working in Vite (Vite handles TypeScript for you), but doing it once by hand demystifies the toolchain.

```bash
mkdir ts-playground
cd ts-playground
npm init -y
npm install --save-dev typescript
npx tsc --init
```

Three things happened:

1. `npm init -y` created `package.json`.
2. `npm install --save-dev typescript` added the compiler.
3. `npx tsc --init` generated a `tsconfig.json` — the file that tells `tsc` how to behave.

Open `tsconfig.json`. There is a lot in it, but the keys that matter to us are:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "strict": true,
    "moduleResolution": "bundler",
    "outDir": "./dist"
  }
}
```

- **`strict: true`** — turns on every type-safety check. Always leave it on.
- **`target`** — what JavaScript version to compile down to.
- **`outDir`** — where the compiled `.js` files go.

Create `index.ts`:

```ts
// index.ts
const greet = (name: string): string => `Hello, ${name}!`;
console.log(greet("Triton"));
```

Compile and run:

```bash
npx tsc            # produces dist/index.js
node dist/index.js
```

**Expected output:**

```
Hello, Triton!
```

For everything else in this class we will assume you are typing into the **TypeScript Playground** at [typescriptlang.org/play](https://www.typescriptlang.org/play) — no install needed, instant feedback.

---

## 28.4 Annotating primitives

A type annotation goes after the variable name with a colon:

```ts
const name: string = "Prabin";
const age: number = 25;
const isStudent: boolean = true;
const middleName: string | undefined = undefined;
```

| Type | Examples |
|------|----------|
| `string` | `"hello"`, `` `template ${x}` `` |
| `number` | `42`, `3.14`, `-1` |
| `boolean` | `true`, `false` |
| `null` | `null` |
| `undefined` | `undefined` |
| `any` | Anything — **avoid** |
| `unknown` | Anything — but safer (see § 28.13) |

### Type inference — TypeScript usually guesses for you

If you assign a value immediately, TypeScript already knows the type:

```ts
// You write:
const title = "Learn TypeScript";

// TypeScript infers: const title: string
// Hover over the name in VS Code to see the inferred type.
```

**Rule of thumb:** annotate function parameters and empty initialisations; let inference handle the rest.

```ts
// ✅ Inference is fine
const score = 95;
const isActive = true;

// ✅ Annotate — no initial value, TypeScript can't guess
let currentFilter: string;
currentFilter = "all";

// ✅ Always annotate function parameters
function greet(name: string) {
  return `Hi, ${name}`;
}
```

---

## 28.5 Arrays and tuples

There are **two** equivalent ways of writing an array type:

```ts
const names: string[] = ["Alice", "Bob"];
const names2: Array<string> = ["Alice", "Bob"];
```

Both mean the same thing. `string[]` is shorter and more common; `Array<string>` is occasionally clearer when the element type itself is complicated (e.g. `Array<{ id: number; title: string }>`).

### Tuples — a fixed-length array with typed positions

A tuple is an array where the **position** of each element matters and the **length** is fixed:

```ts
// First element is always a string, second is always a number
const user: [string, number] = ["Prabin", 25];

user[0].toUpperCase(); // ✅ "PRABIN"
user[1].toFixed(2);    // ✅ "25.00"
// user[2];            // ❌ Tuple has no index 2
```

You will meet tuples in React all the time — every `useState` call returns one:

```ts
// useState<number> returns a [number, function-to-set-it] tuple
const [count, setCount] = useState<number>(0);
```

---

## 28.6 Object types: `interface` vs `type`

When working with objects you need to describe their **shape** — what properties they have and what types those properties are. There are two tools for this: `interface` and `type`.

### Interfaces

```ts
interface User {
  id: number;
  name: string;
  email: string;
  isAdmin: boolean;
}

const me: User = {
  id: 1,
  name: "Prabin",
  email: "prabin@example.com",
  isAdmin: true,
};

// ❌ Missing 'email'
const broken: User = {
  id: 2,
  name: "Bob",
  isAdmin: false,
};
```

### Type aliases

`type` does the same job for objects, but can also name any other type:

```ts
type User = {
  id: number;
  name: string;
};

type Priority = "low" | "medium" | "high";   // union — can't do this with interface
type ID = string | number;                   // alias for a union
```

### Which one should you use?

**Convention used throughout this course (and most React codebases):**

| Use | For |
|-----|-----|
| **`interface`** | Describing the shape of an **object** (component props, an API response, a data record). |
| **`type`** | Everything else: **unions, intersections, primitives, function signatures, tuples**. |

```ts
// ✅ interface for an object
interface PostProps {
  title: string;
  body: string;
}

// ✅ type for a union
type Status = "loading" | "success" | "error";
```

The technical differences between the two are subtle and rarely matter for beginners; pick one rule and stay consistent.

### Optional and readonly

A `?` marks a property as **optional** — it may be missing.
A `readonly` marks it as **fixed once set**.

```ts
interface User {
  readonly id: number;        // can never be changed
  name: string;
  email: string;
  phone?: string;             // optional
}

const u: User = { id: 1, name: "Prabin", email: "p@x.com" };
u.name = "Prabin K.";   // ✅ allowed
// u.id = 99;           // ❌ Cannot assign to 'id' because it is a read-only property
```

---

## 28.7 Union and intersection types

### Union — `A | B` means "A or B"

```ts
type ID = string | number;

let postId: ID = 42;
postId = "abc-123"; // ✅ both allowed
// postId = true;   // ❌ booleans not allowed
```

The most useful union pattern in React is the **string literal union**, where you list the only allowed string values:

```ts
type Priority = "low" | "medium" | "high";
type Theme = "light" | "dark";
type Tab = "users" | "posts" | "settings";

let theme: Theme = "light";
theme = "dark";    // ✅
// theme = "blue"; // ❌
```

### Intersection — `A & B` means "A *and* B"

Intersection combines two object types into one that has **all** the properties of both:

```ts
interface HasName {
  name: string;
}
interface HasAge {
  age: number;
}

type Person = HasName & HasAge;

const p: Person = { name: "Prabin", age: 25 };
// Must have BOTH name and age.
```

You will not use intersections often as a beginner; unions are the workhorse.

---

## 28.8 Narrowing with `typeof`

When a variable could be one of several types (a union), TypeScript will not let you use type-specific methods until you have **narrowed** it down. The classic tool for that is `typeof`:

```ts
function formatId(id: string | number): string {
  if (typeof id === "string") {
    // Inside this block, TypeScript knows id is a string
    return id.toUpperCase();
  }
  // Out here, TypeScript knows id must be a number
  return id.toFixed(0);
}

console.log(formatId("abc"));  // "ABC"
console.log(formatId(42));     // "42"
```

This is called **type narrowing**. The editor's "the type changes depending on the branch you are in" magic is one of TypeScript's most genuinely helpful features.

---

## 28.9 Function parameter and return types

We have already seen the pattern: types go on the parameters, and the return type goes after the closing parenthesis.

```ts
// Plain function
function multiply(a: number, b: number): number {
  return a * b;
}

// Arrow function
const greet = (name: string): string => `Hello, ${name}!`;

// No return value → use `void`
const log = (message: string): void => {
  console.log(message);
};

// Default parameters work the same as in JS
const power = (base: number, exponent: number = 2): number => base ** exponent;

// Rest parameters
const sum = (...nums: number[]): number => nums.reduce((a, b) => a + b, 0);
sum(1, 2, 3, 4); // 10
```

You can usually drop the return type and let inference figure it out — but writing it makes your **intent** clear and catches mistakes when the function accidentally returns the wrong shape. For anything more than a one-liner, leave the return type in.

---

## 28.10 Generics — types that take a parameter

You will not write many generics from scratch as a beginner, but you will **use** them all the time (every call to `useState` is a generic). It is worth understanding the shape.

### The problem

Imagine writing a function that returns its argument unchanged:

```ts
// Only works for strings
const identityString = (x: string): string => x;
// Only works for numbers
const identityNumber = (x: number): number => x;
```

We do not want a copy for every type. We want **"whatever type comes in, the same type comes out."**

### The generic solution

```ts
function identity<T>(x: T): T {
  return x;
}

const a = identity("hello");   // T inferred as string, a: string
const b = identity(42);        // T inferred as number, b: number
const c = identity<boolean>(true); // explicit
```

`<T>` is a **type parameter** — a placeholder that gets filled in when the function is called. By convention single-letter names are used: `T` for "Type", `K` for "Key", `V` for "Value".

### Where you will meet generics in React

```ts
const [count, setCount] = useState<number>(0);
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<Post[]>([]);
```

The `<number>` tells `useState`: "this state holds a number." Without it, TypeScript would not know what shape the state takes.

That is enough for now. Generics deserve a course of their own; we are happy if you can read them.

---

## 28.11 Utility types you will actually use

TypeScript ships with a small library of built-in **utility types** that transform other types into useful new ones. These four come up daily in React work.

```ts
interface User {
  id: number;
  name: string;
  email: string;
  isAdmin: boolean;
}
```

### `Partial<T>` — every property becomes optional

Great for "update" functions where only some fields change.

```ts
function updateUser(id: number, changes: Partial<User>) {
  // changes can have any subset of User's properties
}

updateUser(1, { name: "New Name" });           // ✅
updateUser(1, { email: "a@b.com", isAdmin: true }); // ✅
updateUser(1, {});                              // ✅ (no changes)
```

### `Pick<T, K>` — keep only the listed keys

Great for "summary" shapes.

```ts
type UserListItem = Pick<User, "id" | "name">;

const item: UserListItem = { id: 1, name: "Prabin" };
// item.email; // ❌ does not exist on UserListItem
```

### `Omit<T, K>` — drop the listed keys

The opposite of `Pick`. Good for "creating a new record" where the server will generate the id.

```ts
type NewUser = Omit<User, "id">;

const draft: NewUser = {
  name: "Carol",
  email: "carol@example.com",
  isAdmin: false,
};
```

### `Record<K, V>` — a typed dictionary

Good for "lookup tables" where every key has the same shape of value.

```ts
// A map from user-id to user
type UsersById = Record<number, User>;

const lookup: UsersById = {
  1: { id: 1, name: "Prabin", email: "p@x.com", isAdmin: true },
  2: { id: 2, name: "Bob",    email: "b@x.com", isAdmin: false },
};

console.log(lookup[1].name); // "Prabin"
```

Memorise these four names. You will use them every week.

---

## 28.12 `unknown` vs `any`

Both `any` and `unknown` mean "I do not know what type this is yet." The difference is what TypeScript allows you to do with them.

```ts
// `any` — opts out of type checking entirely. Anything goes. Dangerous.
const x: any = "hello";
x.toFixed(2);   // No error from TypeScript, but CRASHES at runtime!

// `unknown` — also "any type", but the compiler forces you to narrow it first.
const y: unknown = "hello";
// y.toFixed(2); // ❌ TypeScript stops you
if (typeof y === "number") {
  y.toFixed(2); // ✅ safe — narrowed to number
}
```

**Rule:** if you reach for `any`, stop and use `unknown` instead, then narrow it. It is the same flexibility with safety rails attached.

---

## 28.13 Putting it together

A short, realistic example tying everything in this lesson together:

```ts
// User shape
interface User {
  readonly id: number;
  name: string;
  email: string;
  role: "admin" | "editor" | "viewer";  // string literal union
  bio?: string;                          // optional
}

// Only the fields you can edit
type EditableUser = Omit<User, "id">;

// A lookup by id
type UsersById = Record<number, User>;

// A typed array
const users: User[] = [
  { id: 1, name: "Prabin", email: "p@x.com", role: "admin" },
  { id: 2, name: "Bob",    email: "b@x.com", role: "viewer" },
];

// Generic helper: get the first item of any array, or undefined if empty
function firstOf<T>(items: T[]): T | undefined {
  return items[0];
}

// Narrowing with typeof
function formatId(id: string | number): string {
  return typeof id === "string" ? id.toUpperCase() : id.toString();
}

// Partial<User> — perfect for an update payload
function updateUser(users: User[], id: number, patch: Partial<EditableUser>): User[] {
  return users.map((u) => (u.id === id ? { ...u, ...patch } : u));
}

const updated = updateUser(users, 1, { role: "editor", bio: "Senior dev" });
console.log(firstOf(updated));
console.log(formatId(42));
console.log(formatId("abc-123"));
```

If you can read every line of that and predict what TypeScript will allow, you are ready for React.

---

## Practice Exercises

### Exercise 1: Convert JavaScript to TypeScript
Take this JavaScript and add types. Aim for an `interface Book`, typed parameters, and a typed return value.

```js
function isLongTitle(book) {
  return book.title.length > 20;
}

function summarise(books) {
  return books.map(b => `${b.title} by ${b.author}`);
}

const books = [
  { id: 1, title: "Clean Code", author: "Robert Martin", pages: 464 },
  { id: 2, title: "You Don't Know JS", author: "Kyle Simpson", pages: 278 },
];
```

### Exercise 2: Design a Todo type
Define an `interface Todo` with: `id` (readonly number), `title` (string), `completed` (boolean), `priority` ("low" | "medium" | "high"), `description` (optional string). Then write **typed** functions:

```ts
const addTodo = (title: string, priority: "low" | "medium" | "high"): Todo => { /* ... */ };
const toggleTodo = (todos: Todo[], id: number): Todo[] => { /* ... */ };
const updateTodo = (todos: Todo[], id: number, patch: Partial<Todo>): Todo[] => { /* ... */ };
```

### Exercise 3: Break it on purpose
Open the TypeScript Playground and paste the example from § 28.13. Try each of these and read the error message TypeScript gives you. Understanding error messages is half the battle.

1. Add a fourth user with `role: "guest"`.
2. Try to reassign `users[0].id = 99`.
3. Call `updateUser(users, 1, { id: 999 })`.
4. Change `formatId(42)` to `formatId(true)`.

---

## Key Takeaways
- TypeScript catches a whole class of bugs at **compile time**, before the code ever runs.
- Annotate **function parameters** and **empty initialisations**; let **inference** handle the rest.
- Use **`interface`** for object shapes and **`type`** for unions, aliases and primitives.
- **Union (`|`)** = "either"; **intersection (`&`)** = "both"; **`?`** = optional; **`readonly`** = fixed.
- **Narrow** unions with `typeof` before using type-specific methods.
- **Generics** (`<T>`) are placeholders you will mostly *consume* in hooks like `useState<T>`.
- The four utility types to memorise: **`Partial<T>`**, **`Pick<T, K>`**, **`Omit<T, K>`**, **`Record<K, V>`**.
- Reach for **`unknown`** instead of **`any`** — same flexibility, safety rails on.
- Next class: React itself, Vite, and our first `.tsx` component.
