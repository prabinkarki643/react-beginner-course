# Class 19: Functions

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 18 — loops

A **function** is a named block of reusable code. Once you write it, you can run it from anywhere, as many times as you like, with different inputs. Functions are the most important building block in JavaScript — and the entire React phase of this course is built on them.

## What You Will Learn
- Function declarations vs function expressions
- Arrow functions and when to prefer them
- Parameters, default values and rest parameters (`...args`)
- `return` and the difference between expression and statement
- Scope (function vs block) and a gentle introduction to closures

---

## 19.1 Why functions?

Without functions, repetition lives in your code:

```js
const a = 10, b = 5;
console.log("Area: " + (a * b));

const c = 4, d = 6;
console.log("Area: " + (c * d));

const e = 7, f = 8;
console.log("Area: " + (e * f));
```

With a function, the logic lives in **one place**:

```js
function area(width, height) {
    return width * height;
}

console.log("Area: " + area(10, 5));
console.log("Area: " + area(4, 6));
console.log("Area: " + area(7, 8));
```

**Expected output:**
```
Area: 50
Area: 24
Area: 56
```

If we ever need to change how `area` is calculated, we change it once and every caller is fixed.

---

## 19.2 Function declarations

A **function declaration** uses the `function` keyword and a name.

```js
function greet(name) {
    return "Hello, " + name + "!";
}

console.log(greet("Aarya")); // Hello, Aarya!
console.log(greet("Prabin")); // Hello, Prabin!
```

Anatomy:

```
function greet(name) {
//       ↑      ↑
//      name  parameter (the input)
    return "Hello, " + name + "!";
//  ↑                            ↑
// return                        the output
}
```

### Hoisting (a nice quirk)

Function declarations are **hoisted** — they are available throughout their scope, even **before** the line they appear on. This is fine:

```js
console.log(greet("Aarya")); // works!

function greet(name) {
    return "Hello, " + name + "!";
}
```

This is unique to function declarations. The other forms below behave differently.

---

## 19.3 Function expressions

A **function expression** stores a function in a variable. We almost always use `const`.

```js
const greet = function (name) {
    return "Hello, " + name + "!";
};

console.log(greet("Aarya")); // Hello, Aarya!
```

Function expressions are **not** hoisted — you must define them before you call them.

```js
// ❌ Crashes: "Cannot access 'greet' before initialisation"
console.log(greet("Aarya"));

const greet = function (name) {
    return "Hello, " + name + "!";
};
```

Why bother with function expressions at all? Because they lead naturally to the most common modern form: **arrow functions**.

---

## 19.4 Arrow functions

Arrow functions are a shorter syntax for function expressions, introduced in ES6 (2015). They are the dominant style in modern JavaScript and the entire React world.

```js
// Step 1: a function expression
const greet = function (name) {
    return "Hello, " + name + "!";
};

// Step 2: drop "function", add => after the parameters
const greetArrow = (name) => {
    return "Hello, " + name + "!";
};

// Step 3: one expression, no braces — implicit return
const greetShort = (name) => "Hello, " + name + "!";

// Step 4: single parameter? parentheses are optional
const greetTiny = name => "Hello, " + name + "!";

console.log(greetTiny("Aarya")); // Hello, Aarya!
```

### Important arrow rules

- **No parameters?** You still need empty parentheses: `() => 42`.
- **More than one parameter?** Parentheses required: `(a, b) => a + b`.
- **One-line return?** Drop the braces and `return` — the value of the expression is returned.
- **Multi-line body?** You need braces, and you must write `return` yourself.

```js
const square = n => n * n;
const sum = (a, b) => a + b;
const now = () => new Date().toLocaleTimeString();

const describe = (name, age) => {
    const label = age >= 18 ? "adult" : "minor";
    return name + " is a " + label;
};

console.log(square(5));            // 25
console.log(sum(3, 4));            // 7
console.log(now());                // "14:32:18" (or whatever the time is)
console.log(describe("Aarya", 20)); // "Aarya is a adult"
```

### When to use which

| Form | When |
|------|------|
| Arrow function | **Default choice** — short, common in React |
| Function declaration | When you want **hoisting** (helpful for top-level helpers) |
| Function expression with `function` | Almost never — arrow is the modern replacement |

There is also one subtle difference around the `this` keyword — arrow functions do not have their own `this`. We will explore that only when it becomes relevant (in React, it almost never bites you).

---

## 19.5 Parameters and arguments

A **parameter** is what the function declares (the placeholder).
An **argument** is what you actually pass when you call it.

```js
function welcome(name, course) { // name and course are parameters
    return name + " is studying " + course;
}

console.log(welcome("Aarya", "BCA")); // "Aarya" and "BCA" are arguments
```

### Default parameter values

You can give parameters a default so callers can skip them:

```js
const greet = (name = "friend") => "Hello, " + name + "!";

console.log(greet());        // "Hello, friend!"
console.log(greet("Aarya")); // "Hello, Aarya!"
```

Defaults can also use other parameters:

```js
const order = (item, qty = 1, message = "Buying " + qty + " " + item) => {
    return message;
};

console.log(order("apple"));          // "Buying 1 apple"
console.log(order("apple", 3));       // "Buying 3 apple"
console.log(order("apple", 3, "x!")); // "x!"
```

### Rest parameters `...args`

If you want a function to take an **unknown number** of arguments, use `...`:

```js
const sum = (...numbers) => {
    let total = 0;
    for (const n of numbers) {
        total += n;
    }
    return total;
};

console.log(sum(1, 2));        // 3
console.log(sum(1, 2, 3, 4));  // 10
console.log(sum());            // 0
```

`numbers` is a real array — you can use any array method on it.

You can mix fixed and rest parameters, but the rest **must come last**:

```js
const greetMany = (greeting, ...names) => {
    for (const name of names) {
        console.log(greeting + ", " + name + "!");
    }
};

greetMany("Hi", "Aarya", "Prabin", "Sita");
```

**Expected output:**
```
Hi, Aarya!
Hi, Prabin!
Hi, Sita!
```

---

## 19.6 `return` — sending a value back

`return` does two things:

1. Sends a value back to whoever called the function.
2. **Exits the function immediately** — anything after it is ignored.

```js
const checkAge = (age) => {
    if (age < 0) {
        return "Invalid age";
    }
    if (age < 18) {
        return "Minor";
    }
    return "Adult";
};

console.log(checkAge(-5)); // Invalid age
console.log(checkAge(15)); // Minor
console.log(checkAge(25)); // Adult
```

Each `return` is a possible exit. Using early returns ("guard clauses") often leads to cleaner code than nested `if/else`.

### Functions without an explicit return

If you do not write `return`, the function implicitly returns `undefined`.

```js
const shout = (msg) => {
    console.log(msg.toUpperCase());
};

const result = shout("hello");
console.log(result); // undefined  (shout never returned anything)
```

This is fine if the function exists only to **do something** (a "side effect" like logging). If it should produce a value, it needs to `return`.

---

## 19.7 Scope — where variables live

**Scope** is the rule that decides where a variable is visible.

### Function scope

A variable declared inside a function does not exist outside it.

```js
function play() {
    const score = 10;
    console.log(score); // 10
}

play();
console.log(score); // ReferenceError: score is not defined
```

### Block scope

`let` and `const` are also scoped to the **block** (`{ }`) they appear in — not just functions.

```js
if (true) {
    const message = "inside the block";
    console.log(message); // works
}

console.log(message); // ReferenceError
```

This is why we avoid `var` — `var` ignores block scope and leaks out of `if`/`for` blocks, which is confusing.

### Nested scope (the "scope chain")

Inner scopes can see outer variables, but not the other way around.

```js
const courseName = "BCA";

function showCourse() {
    const semester = 2;
    console.log(courseName + " - Semester " + semester); // can see both
}

showCourse();              // "BCA - Semester 2"
console.log(semester);     // ReferenceError
```

---

## 19.8 Closures — a gentle introduction

A **closure** is what happens when a function "remembers" the variables that were in scope when it was created, even after that outer code has finished running.

This sounds abstract, so let's see the classic example: a counter.

```js
function makeCounter() {
    let count = 0;

    return () => {
        count = count + 1;
        return count;
    };
}

const counter = makeCounter();

console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

What is going on?

1. `makeCounter` is called once. Inside, it creates `count = 0`.
2. It returns an arrow function that uses `count`.
3. Normally, `count` would disappear when `makeCounter` finishes — but the returned arrow function still references it, so JavaScript keeps `count` alive.
4. That "kept-alive" variable is the closure's private state.

Each new call to `makeCounter` creates a **fresh** `count`:

```js
const a = makeCounter();
const b = makeCounter();

console.log(a()); // 1
console.log(a()); // 2
console.log(b()); // 1 (separate count!)
```

We will rely on this idea heavily when we meet **React hooks** like `useState` — they use closures behind the scenes.

---

## 19.9 Putting it together: a tiny utility belt

```js
const isEven = n => n % 2 === 0;
const isOdd = n => !isEven(n);

const formatCurrency = (amount, currency = "GBP") => {
    return currency + " " + amount.toFixed(2);
};

const greet = (name, ...titles) => {
    const titleString = titles.length > 0 ? titles.join(" ") + " " : "";
    return "Hello, " + titleString + name;
};

console.log(isEven(4));                            // true
console.log(isOdd(7));                             // true
console.log(formatCurrency(19.5));                 // "GBP 19.50"
console.log(formatCurrency(19.5, "NPR"));          // "NPR 19.50"
console.log(greet("Aarya"));                       // "Hello, Aarya"
console.log(greet("Aarya", "Er.", "Dr."));         // "Hello, Er. Dr. Aarya"
```

---

## Practice Exercises

1. **`formatCurrency(amount, currency)`**: Write a function that returns a string like `"£ 199.99"`. Default currency should be `"GBP"`. Support `"USD"` (`"$"`), `"EUR"` (`"€"`) and `"NPR"` (`"Rs"`). Test it with several values.

2. **`average(...numbers)`**: Using rest parameters, write a function that accepts any number of arguments and returns their average. Return `0` if no arguments are passed.

3. **Early returns**: Rewrite this nested `if/else` using early returns ("guard clauses"):

   ```js
   function describe(person) {
       let result;
       if (person) {
           if (person.age) {
               if (person.age >= 18) {
                   result = "adult";
               } else {
                   result = "minor";
               }
           } else {
               result = "no age given";
           }
       } else {
           result = "no person given";
       }
       return result;
   }
   ```

4. **Closure counter**: Modify the `makeCounter` example so it accepts a starting value and a step size:

   ```js
   const c = makeCounter(10, 2);
   c(); // 12
   c(); // 14
   c(); // 16
   ```

---

## Key Takeaways
- Use **arrow functions** by default. Use function declarations when you want hoisting.
- Parameters can have **default values**; collect "everything else" with `...rest`.
- `return` sends a value back **and** ends the function — use early returns to flatten code.
- `let` and `const` are **block-scoped**; `var` is function-scoped (which is why we avoid it).
- A **closure** is a function that remembers variables from where it was defined. We will see closures again with React hooks.

Next class: arrays and the methods that make data wrangling beautiful — `map`, `filter`, `reduce` and friends.
