# Class 21: Objects, Destructuring & Spread/Rest

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 20 — arrays and array methods

If arrays are **ordered lists**, objects are **labelled boxes**. They are the way we group related data together: a student, a product, a blog post. Today's class also covers destructuring and spread — two pieces of syntax that show up in nearly every React file you will ever read.

## What You Will Learn
- Object literals, dot vs bracket access and dynamic keys
- Methods and shorthand syntax
- Destructuring objects and arrays (with rename and defaults)
- The spread (`...`) and rest (`...`) operators for arrays and objects
- The crucial difference between value and reference (shallow copies)

---

## 21.1 Creating an object

An object is a collection of **key/value pairs**, written with curly braces:

```js
const student = {
    name: "Aarya",
    age: 20,
    course: "BCA",
    isEnrolled: true
};

console.log(student);
```

**Expected output:**
```
{ name: 'Aarya', age: 20, course: 'BCA', isEnrolled: true }
```

The "keys" are the labels (`name`, `age`); the "values" are whatever you store next to them.

### Dot access vs bracket access

```js
console.log(student.name);       // "Aarya"
console.log(student["course"]);  // "BCA"
```

Both work. When do you use which?

- **Dot notation** is shorter and what you will use 95% of the time.
- **Bracket notation** is required when the key is dynamic or has unusual characters:

```js
const field = "age";
console.log(student[field]); // 20  — the variable's value is used as the key

const weird = { "first name": "Aarya" };
console.log(weird["first name"]); // "Aarya"
// console.log(weird.first name); // SyntaxError
```

### Adding, changing and deleting properties

```js
const student = { name: "Aarya", age: 20 };

student.course = "BCA";        // add new property
student.age = 21;              // change existing
delete student.course;         // remove a property

console.log(student); // { name: 'Aarya', age: 21 }
```

Note: even though `student` is declared with `const`, we can still change its **contents**. `const` locks the variable to one object — it does not freeze that object's properties.

---

## 21.2 Dynamic keys (computed properties)

Sometimes you need a key whose name comes from a variable. Wrap the key in `[ ]` when writing the literal:

```js
const field = "score";
const points = 95;

const result = {
    [field]: points,
    timestamp: new Date().toISOString()
};

console.log(result); // { score: 95, timestamp: '2026-05-31T...' }
```

This is incredibly handy when keys come from user input or function arguments.

---

## 21.3 Methods and shorthand syntax

A **method** is just a function stored inside an object.

```js
const counter = {
    count: 0,
    increment: function () {
        this.count = this.count + 1;
    }
};

counter.increment();
counter.increment();
console.log(counter.count); // 2
```

Inside a method declared with the `function` keyword, `this` refers to the object that owns it.

### Shorthand syntax (modern, preferred)

```js
const counter = {
    count: 0,

    // Method shorthand — no need for ": function"
    increment() {
        this.count = this.count + 1;
    },

    reset() {
        this.count = 0;
    }
};

counter.increment();
counter.increment();
counter.increment();
console.log(counter.count); // 3
counter.reset();
console.log(counter.count); // 0
```

### Property shorthand

If a key and the variable holding its value have the same name, you can write the key just once:

```js
const name = "Aarya";
const age = 20;

// Old way
const personA = { name: name, age: age };

// Shorthand
const personB = { name, age };

console.log(personB); // { name: 'Aarya', age: 20 }
```

This shorthand is everywhere in modern JS.

---

## 21.4 Destructuring objects

**Destructuring** lets you pull values out of an object (or array) into separate variables in one line.

```js
const student = { name: "Aarya", age: 20, course: "BCA" };

// Old way
const oldName = student.name;
const oldAge = student.age;

// Destructuring
const { name, age, course } = student;
console.log(name, age, course); // "Aarya" 20 "BCA"
```

The variable names **must match the keys**. The order of variables does not matter.

### Renaming while destructuring

```js
const student = { name: "Aarya", age: 20 };

const { name: studentName, age: studentAge } = student;

console.log(studentName); // "Aarya"
console.log(studentAge);  // 20
```

Read it as "take `name`, call it `studentName` here".

### Defaults

If a key is missing (its value would be `undefined`), you can provide a default:

```js
const settings = { theme: "dark" };

const { theme, fontSize = 16, language = "en" } = settings;

console.log(theme, fontSize, language); // "dark" 16 "en"
```

### Combining rename and default

```js
const { language: lang = "en" } = settings;
console.log(lang); // "en"
```

### Destructuring function parameters

This is hugely common in React — props are destructured this way every day.

```js
const greet = ({ name, age }) => {
    return `Hi ${name}, you are ${age}.`;
};

console.log(greet({ name: "Aarya", age: 20, course: "BCA" }));
// "Hi Aarya, you are 20."
```

---

## 21.5 Destructuring arrays

The same idea works on arrays, but you use `[ ]` and the **order matters**.

```js
const colours = ["red", "green", "blue"];

const [first, second, third] = colours;
console.log(first, second, third); // "red" "green" "blue"

// Skip items with empty commas
const [, , last] = colours;
console.log(last); // "blue"

// With defaults
const [a, b, c, d = "yellow"] = colours;
console.log(d); // "yellow"
```

You will see array destructuring all the time in React hooks:

```js
// const [count, setCount] = useState(0);  // ← exactly this pattern
```

---

## 21.6 Spread (`...`) — unpack things

The spread operator takes an array or object and "unpacks" its contents.

### Spread with arrays

```js
const a = [1, 2, 3];
const b = [4, 5, 6];

const combined = [...a, ...b];
console.log(combined); // [1, 2, 3, 4, 5, 6]

const withZero = [0, ...a];
console.log(withZero); // [0, 1, 2, 3]

const withTen = [...a, 10];
console.log(withTen); // [1, 2, 3, 10]

const copy = [...a];
console.log(copy);   // [1, 2, 3] — a brand new array
console.log(copy === a); // false (different identity)
```

This is the non-mutating way to "push" — `[...arr, x]` instead of `arr.push(x)`.

### Spread with objects

```js
const base = { name: "Aarya", age: 20 };

// Copy
const copy = { ...base };
console.log(copy); // { name: "Aarya", age: 20 }

// Add a new key
const withCourse = { ...base, course: "BCA" };
console.log(withCourse); // { name: "Aarya", age: 20, course: "BCA" }

// Override a key (later keys win)
const updated = { ...base, age: 21 };
console.log(updated); // { name: "Aarya", age: 21 }

// Merge two objects
const defaults = { theme: "light", fontSize: 14 };
const userPrefs = { theme: "dark" };
const final = { ...defaults, ...userPrefs };
console.log(final); // { theme: "dark", fontSize: 14 }
```

This is **the** way to "update" state in React.

---

## 21.7 Rest (`...`) — gather things up

The rest operator looks identical to spread, but it does the opposite: it collects what's left into a new array or object.

### Rest in array destructuring

```js
const [first, ...others] = [10, 20, 30, 40, 50];

console.log(first);  // 10
console.log(others); // [20, 30, 40, 50]
```

### Rest in object destructuring

```js
const student = { name: "Aarya", age: 20, course: "BCA", city: "Kathmandu" };

const { name, ...details } = student;

console.log(name);    // "Aarya"
console.log(details); // { age: 20, course: "BCA", city: "Kathmandu" }
```

### Rest in function parameters (from Class 19)

```js
const sum = (...numbers) => numbers.reduce((acc, n) => acc + n, 0);
console.log(sum(1, 2, 3, 4)); // 10
```

> The rule: if the `...` is **on the left** of an `=`, it is rest (collecting). If it is **on the right**, it is spread (unpacking).

---

## 21.8 Value vs reference (the most important slide today)

This is the bit that catches every beginner. Read it twice.

### Primitives are copied by value

```js
let a = 5;
let b = a;
b = 10;

console.log(a); // 5    (unchanged)
console.log(b); // 10
```

`b` got its own copy of the value `5`. Changing `b` does not affect `a`.

### Objects and arrays are copied by reference

```js
const personA = { name: "Aarya", age: 20 };
const personB = personA;

personB.age = 99;

console.log(personA.age); // 99  ← !!
console.log(personB.age); // 99
console.log(personA === personB); // true (same object)
```

`personA` and `personB` are two labels pointing to the **same** object in memory. Changing one changes the other.

The same applies to arrays:

```js
const listA = [1, 2, 3];
const listB = listA;

listB.push(4);

console.log(listA); // [1, 2, 3, 4]  ← !!
console.log(listB); // [1, 2, 3, 4]
```

### How to actually copy

For a shallow copy, spread is your friend:

```js
const personA = { name: "Aarya", age: 20 };
const personB = { ...personA };

personB.age = 99;

console.log(personA.age); // 20  ✅
console.log(personB.age); // 99
```

For arrays:

```js
const listA = [1, 2, 3];
const listB = [...listA];

listB.push(4);

console.log(listA); // [1, 2, 3]  ✅
console.log(listB); // [1, 2, 3, 4]
```

### "Shallow" — what's the catch?

Spread copies the **top level** only. Nested objects are still shared.

```js
const userA = {
    name: "Aarya",
    address: { city: "Kathmandu", country: "Nepal" }
};

const userB = { ...userA };
userB.name = "Prabin";              // top-level — only userB changes
userB.address.city = "Pokhara";     // nested — both change!

console.log(userA.name);            // "Aarya"  ✅
console.log(userB.name);            // "Prabin"
console.log(userA.address.city);    // "Pokhara"  ⚠️
console.log(userB.address.city);    // "Pokhara"
```

To deep-update, spread at every level:

```js
const userC = {
    ...userA,
    address: { ...userA.address, city: "Pokhara" }
};

console.log(userA.address.city); // "Kathmandu" — safe
console.log(userC.address.city); // "Pokhara"
```

This pattern is everywhere in React state updates.

---

## 21.9 A bigger worked example

```js
const users = [
    { id: 1, name: "Aarya",  role: "student" },
    { id: 2, name: "Prabin", role: "tutor" },
    { id: 3, name: "Sita",   role: "student" }
];

// 1. Convert to a "by id" lookup object
const byId = users.reduce((acc, user) => {
    acc[user.id] = user;
    return acc;
}, {});

console.log(byId);
// {
//   1: { id: 1, name: "Aarya",  role: "student" },
//   2: { id: 2, name: "Prabin", role: "tutor" },
//   3: { id: 3, name: "Sita",   role: "student" }
// }

console.log(byId[2]); // quick lookup by id

// 2. Update user 1's role — without mutating the original
const updatedUsers = users.map(u =>
    u.id === 1 ? { ...u, role: "tutor" } : u
);

console.log(users[0].role);        // "student" — safe
console.log(updatedUsers[0].role); // "tutor"

// 3. Add a new user (non-mutating)
const withNew = [...users, { id: 4, name: "Ram", role: "student" }];
console.log(withNew.length); // 4
console.log(users.length);   // 3 — unchanged
```

---

## Practice Exercises

1. **Normalise a list**: Convert this array into a `byId` lookup object using `reduce`:

   ```js
   const posts = [
       { id: 101, title: "Hello" },
       { id: 102, title: "World" },
       { id: 103, title: "Again" }
   ];
   // Target: { 101: { id: 101, title: "Hello" }, 102: ..., 103: ... }
   ```

2. **Update one item without mutating**: Given the `users` array from 21.9, write a function `updateUser(users, id, updates)` that returns a new array where the matching user is updated. The original array must be unchanged.

3. **Destructure with defaults**: Given this object, destructure `name`, `theme` (default `"light"`) and `fontSize` (default `16`). Log each.

   ```js
   const settings = { name: "Aarya", theme: "dark" };
   ```

4. **Shallow copy gotcha**: Create a nested object with at least two levels. Make a shallow copy with `...`, change a top-level field, then change a nested field. Print both originals and copies to see the difference. Then fix it with a deep update.

---

## Key Takeaways
- Access object values with dot (`obj.key`) or bracket (`obj[var]`) notation.
- **Destructuring** unpacks values into variables; it supports renaming and defaults.
- **Spread (`...`)** unpacks; **rest (`...`)** gathers — same syntax, opposite roles.
- `{ ...obj, key: newValue }` is the React way to "update" — it returns a brand new object.
- Objects and arrays are **shared by reference**. Spread gives you a **shallow** copy. For nested updates, spread at every level.

Next class: we leave pure JavaScript and start manipulating real HTML with the DOM.
