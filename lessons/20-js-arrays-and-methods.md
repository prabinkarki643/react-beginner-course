# Class 20: Arrays & Array Methods

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 19 — functions (especially arrow functions)

Arrays are how we store **lists of things**. Today's content is some of the most important material in the whole course. Almost every React component you ever write will use `.map()`, and ideas like "do not mutate" come straight from these methods. Take your time.

## What You Will Learn
- Creating, indexing and measuring arrays
- Mutating methods: `push`, `pop`, `shift`, `unshift`, `splice`
- Non-mutating methods: `map`, `filter`, `find`, `some`, `every`, `reduce`, `includes`, `slice`, `concat`
- Method chaining
- The crucial difference: mutating vs non-mutating

---

## 20.1 Creating an array

An array is an ordered list, written with square brackets:

```js
const empty = [];
const fruits = ["apple", "banana", "cherry"];
const numbers = [10, 20, 30, 40];
const mixed = ["Aarya", 20, true, null]; // any types allowed
```

### Indexing

Each item has a numeric **index**, starting at **0**.

```js
const fruits = ["apple", "banana", "cherry"];

console.log(fruits[0]); // "apple"
console.log(fruits[1]); // "banana"
console.log(fruits[2]); // "cherry"
console.log(fruits[3]); // undefined  (no such index)
```

### `length` and the last item

```js
const fruits = ["apple", "banana", "cherry"];

console.log(fruits.length);             // 3
console.log(fruits[fruits.length - 1]); // "cherry" (last item)

// ES2023 sugar:
console.log(fruits.at(-1));             // "cherry"
console.log(fruits.at(-2));             // "banana"
```

`.at(-1)` is a tidy way to get the last item without writing `length - 1`.

---

## 20.2 Mutating methods — they **change the original**

These methods modify the array in place. After they run, the original array is different.

### `push` — add to the end

```js
const fruits = ["apple", "banana"];
fruits.push("cherry");
console.log(fruits); // ["apple", "banana", "cherry"]
```

### `pop` — remove from the end

```js
const fruits = ["apple", "banana", "cherry"];
const removed = fruits.pop();
console.log(removed); // "cherry"
console.log(fruits);  // ["apple", "banana"]
```

### `unshift` — add to the start

```js
const fruits = ["banana", "cherry"];
fruits.unshift("apple");
console.log(fruits); // ["apple", "banana", "cherry"]
```

### `shift` — remove from the start

```js
const fruits = ["apple", "banana", "cherry"];
const removed = fruits.shift();
console.log(removed); // "apple"
console.log(fruits);  // ["banana", "cherry"]
```

### `splice` — surgical insert/remove anywhere

`splice(start, deleteCount, ...itemsToInsert)` is the swiss-army knife — it can remove, insert, or both.

```js
const colours = ["red", "green", "blue", "yellow"];

// Remove 2 items starting at index 1
colours.splice(1, 2);
console.log(colours); // ["red", "yellow"]

// Insert items at index 1 (delete count = 0)
colours.splice(1, 0, "green", "blue");
console.log(colours); // ["red", "green", "blue", "yellow"]

// Replace: remove 1, insert 1
colours.splice(2, 1, "purple");
console.log(colours); // ["red", "green", "purple", "yellow"]
```

> A memorable trap: people confuse `splice` (mutating) with `slice` (non-mutating). One letter, opposite behaviour.

---

## 20.3 Non-mutating methods — they **return a new array**

These methods leave the original array alone and return a new one. **This is the React-friendly style** — React expects you to never modify state directly.

### `slice` — copy a section

`slice(start, end)` returns items from `start` up to (but not including) `end`.

```js
const fruits = ["apple", "banana", "cherry", "date", "elderberry"];

const middle = fruits.slice(1, 3);
console.log(middle); // ["banana", "cherry"]

const last2 = fruits.slice(-2);
console.log(last2);  // ["date", "elderberry"]

const copy = fruits.slice();
console.log(copy);   // ["apple", "banana", "cherry", "date", "elderberry"]

console.log(fruits); // unchanged
```

### `concat` — join arrays together

```js
const a = [1, 2];
const b = [3, 4];
const c = [5, 6];

const all = a.concat(b, c);
console.log(all); // [1, 2, 3, 4, 5, 6]
console.log(a);   // [1, 2] — unchanged
```

### `includes` — does the array contain this value?

```js
const fruits = ["apple", "banana", "cherry"];

console.log(fruits.includes("banana"));  // true
console.log(fruits.includes("mango"));   // false
```

### `map` — transform every item

`map` is **the** array method. It creates a new array by running a function over every item.

```js
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(n => n * 2);

console.log(doubled); // [2, 4, 6, 8]
console.log(numbers); // [1, 2, 3, 4] — unchanged
```

A more realistic example — extracting a single field from a list of objects:

```js
const students = [
    { name: "Aarya", marks: 78 },
    { name: "Prabin", marks: 92 },
    { name: "Sita", marks: 65 }
];

const names = students.map(student => student.name);
console.log(names); // ["Aarya", "Prabin", "Sita"]
```

You will write `.map()` thousands of times. It is how React renders lists.

### `filter` — keep items that match

`filter` returns a new array with only the items for which your function returns `true`.

```js
const numbers = [1, 2, 3, 4, 5, 6];
const evens = numbers.filter(n => n % 2 === 0);

console.log(evens); // [2, 4, 6]
```

With objects:

```js
const students = [
    { name: "Aarya", marks: 78 },
    { name: "Prabin", marks: 92 },
    { name: "Sita", marks: 35 }
];

const passing = students.filter(s => s.marks >= 40);
console.log(passing);
// [ { name: "Aarya", marks: 78 }, { name: "Prabin", marks: 92 } ]
```

### `find` — first match (or `undefined`)

```js
const students = [
    { name: "Aarya", marks: 78 },
    { name: "Prabin", marks: 92 },
    { name: "Sita", marks: 35 }
];

const aarya = students.find(s => s.name === "Aarya");
console.log(aarya); // { name: "Aarya", marks: 78 }

const nobody = students.find(s => s.name === "Ram");
console.log(nobody); // undefined
```

Use `find` when you want one item; use `filter` when you want all matches.

### `some` — does **any** item match?

```js
const numbers = [1, 2, 3, 4];

console.log(numbers.some(n => n > 3)); // true
console.log(numbers.some(n => n > 9)); // false
```

### `every` — do **all** items match?

```js
const numbers = [1, 2, 3, 4];

console.log(numbers.every(n => n > 0)); // true
console.log(numbers.every(n => n > 2)); // false
```

### `reduce` — combine everything into one value

`reduce` is the most powerful (and most intimidating) of these. It takes two arguments:

1. A function that takes `(accumulator, currentItem)` and returns the new accumulator.
2. A starting value for the accumulator.

```js
const numbers = [10, 20, 30, 40];

const total = numbers.reduce((acc, n) => acc + n, 0);
console.log(total); // 100
```

Step by step:

| Step | acc (before) | n | result |
|---|---|---|---|
| 1 | 0 | 10 | 10 |
| 2 | 10 | 20 | 30 |
| 3 | 30 | 30 | 60 |
| 4 | 60 | 40 | 100 |

`reduce` can do far more than sums — it can build objects, group data, find max/min. For now, sum and product are great starting examples.

```js
const product = numbers.reduce((acc, n) => acc * n, 1);
console.log(product); // 240000
```

---

## 20.4 Mutating vs non-mutating — a side-by-side comparison

This is one of the most useful tables in JavaScript. Pin it on the wall.

| Mutating (changes original) | Non-mutating (returns new) | Use when |
|---|---|---|
| `arr.push(x)` | `[...arr, x]` (Class 21) | Adding to end |
| `arr.pop()` | `arr.slice(0, -1)` | Removing last |
| `arr.unshift(x)` | `[x, ...arr]` | Adding to start |
| `arr.shift()` | `arr.slice(1)` | Removing first |
| `arr.splice(...)` | `arr.filter(...)` / `arr.map(...)` | Inserting/removing anywhere |
| `arr.sort()` | `arr.toSorted()` | Sorting |
| `arr.reverse()` | `arr.toReversed()` | Reversing |

> Rule for React: **prefer non-mutating methods**. State must be updated by replacing arrays, not poking at them.

### A live example

```js
const original = [3, 1, 2];

// MUTATING — original is now [1, 2, 3]
original.sort();
console.log(original); // [1, 2, 3]

// NON-MUTATING — original stays as it was
const original2 = [3, 1, 2];
const sorted2 = original2.toSorted();
console.log(sorted2);    // [1, 2, 3]
console.log(original2);  // [3, 1, 2] — unchanged
```

---

## 20.5 Method chaining

Because non-mutating methods **return a new array**, you can call another method on the result. Chaining reads top-to-bottom like a recipe.

```js
const students = [
    { name: "Aarya",  marks: 78 },
    { name: "Prabin", marks: 92 },
    { name: "Sita",   marks: 35 },
    { name: "Ram",    marks: 88 },
    { name: "Geeta",  marks: 41 }
];

const passingNamesUpper = students
    .filter(s => s.marks >= 40)        // keep those who passed
    .map(s => s.name)                  // pull out just the names
    .map(name => name.toUpperCase());  // shout them

console.log(passingNamesUpper);
// ["AARYA", "PRABIN", "RAM", "GEETA"]
```

Each step is a simple transformation, and the whole pipeline is easy to read.

### Top-3 by marks

```js
const top3 = students
    .toSorted((a, b) => b.marks - a.marks) // sort by marks descending
    .slice(0, 3);                          // first 3

console.log(top3);
// [
//   { name: "Prabin", marks: 92 },
//   { name: "Ram",    marks: 88 },
//   { name: "Aarya",  marks: 78 }
// ]
```

---

## 20.6 A complete worked example

```js
const orders = [
    { id: 1, item: "Notebook", qty: 3, price: 50,  paid: true },
    { id: 2, item: "Pen",      qty: 5, price: 20,  paid: false },
    { id: 3, item: "Bag",      qty: 1, price: 800, paid: true },
    { id: 4, item: "Eraser",   qty: 4, price: 10,  paid: true },
    { id: 5, item: "Ruler",    qty: 2, price: 30,  paid: false }
];

// 1. Total revenue from paid orders
const revenue = orders
    .filter(o => o.paid)
    .reduce((acc, o) => acc + o.qty * o.price, 0);

// 2. List of items that still need paying
const unpaidItems = orders
    .filter(o => !o.paid)
    .map(o => o.item);

// 3. Has anything over Rs. 500 been ordered?
const hasExpensiveOrder = orders.some(o => o.price >= 500);

// 4. Are all orders for at least 1 item?
const allValid = orders.every(o => o.qty >= 1);

console.log("Revenue:", revenue);                 // 1810
console.log("Unpaid items:", unpaidItems);        // ["Pen", "Ruler"]
console.log("Expensive order present?", hasExpensiveOrder); // true
console.log("All valid?", allValid);              // true
```

---

## Practice Exercises

1. **Marks list**: Given the array below, write code that:
   1. Returns just the names.
   2. Returns only students who scored 60 or above.
   3. Calculates the average marks.
   4. Returns the top 3 by marks (do not mutate the original).

   ```js
   const students = [
       { name: "Aarya",  marks: 78 },
       { name: "Prabin", marks: 92 },
       { name: "Sita",   marks: 35 },
       { name: "Ram",    marks: 88 },
       { name: "Geeta",  marks: 41 },
       { name: "Hari",   marks: 67 }
   ];
   ```

2. **Mutating vs non-mutating**: For each of these, predict whether the original array changes, then verify in the console:
   - `arr.push(99)`
   - `arr.slice(0, 2)`
   - `arr.splice(1, 1)`
   - `arr.map(n => n * 2)`
   - `arr.filter(n => n > 5)`
   - `arr.sort()`
   - `arr.toSorted()`

3. **Word count**: Given an array of words, use `reduce` to build an object of `{ word: count }`. (Hint: the accumulator starts as `{}`.)

   ```js
   const words = ["apple", "pear", "apple", "apple", "pear", "orange"];
   // Target: { apple: 3, pear: 2, orange: 1 }
   ```

4. **Chained pipeline**: Given a list of numbers, in one chain:
   - Keep only the positive numbers
   - Double each one
   - Sum them up

---

## Key Takeaways
- Arrays are zero-indexed; `arr.length - 1` (or `arr.at(-1)`) is the last item.
- **Mutating** methods change the original (`push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse`).
- **Non-mutating** methods return a new array (`map`, `filter`, `slice`, `concat`, `find`, `some`, `every`, `reduce`, `toSorted`, `toReversed`).
- Prefer non-mutating methods — they chain beautifully and are React-friendly.
- `map`, `filter` and `reduce` are the three you will use every single day.

Next class: objects, destructuring, and the spread operator that ties everything together.
