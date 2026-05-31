# Class 17: Operators & Conditionals

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 16 — variables and primitive types

Variables hold values. Today, we start **doing things** with those values: adding them, comparing them, and choosing different paths based on the result. This is how programs make decisions.

## What You Will Learn
- Arithmetic, assignment and comparison operators
- Logical operators `&&`, `||`, `!` and the ternary `? :`
- The crucial difference between `==` and `===`
- `if`, `else if`, `else` and the `switch` statement
- The truthy and falsy values you must memorise

---

## 17.1 Arithmetic operators

JavaScript supports all the maths you would expect.

```js
console.log(10 + 3);  // 13
console.log(10 - 3);  // 7
console.log(10 * 3);  // 30
console.log(10 / 3);  // 3.3333333333333335
console.log(10 % 3);  // 1   (remainder — also called "modulo")
console.log(2 ** 8);  // 256 (exponent — 2 to the power 8)
```

The remainder operator `%` is more useful than it looks. It is the easiest way to check if a number is even:

```js
const n = 14;
console.log(n % 2); // 0  → even
const m = 7;
console.log(m % 2); // 1  → odd
```

### String concatenation with `+`

The `+` operator does two jobs depending on its operands:

```js
console.log(2 + 3);          // 5    (number + number = addition)
console.log("Hi " + "there"); // "Hi there" (string + string = concatenation)
console.log("Age: " + 21);    // "Age: 21" (mixed = string wins)
console.log(21 + " years");   // "21 years"
```

> Watch out: `"5" + 5` gives `"55"`, not `10`. We will see why in section 17.4.

### Increment and decrement

```js
let count = 0;
count++;   // count is now 1 (shorthand for count = count + 1)
count++;   // 2
count--;   // 1 (shorthand for count = count - 1)
console.log(count); // 1
```

---

## 17.2 Assignment operators

The plain `=` puts a value in a variable. There are also combined operators that update in place:

```js
let score = 10;

score += 5;  // score = score + 5  → 15
score -= 3;  // score = score - 3  → 12
score *= 2;  // score = score * 2  → 24
score /= 4;  // score = score / 4  → 6
score %= 4;  // score = score % 4  → 2

console.log(score); // 2
```

These are not new powers — they are just shorter ways to write `score = score + 5` and so on.

---

## 17.3 Comparison operators

Comparisons return `true` or `false`.

```js
console.log(5 > 3);    // true
console.log(5 < 3);    // false
console.log(5 >= 5);   // true
console.log(5 <= 4);   // false
console.log(5 === 5);  // true   (strict equal)
console.log(5 !== 3);  // true   (strict not equal)
```

---

## 17.4 `==` vs `===` — the most important rule in JS

JavaScript has **two** equality operators:

| Operator | Name | What it does |
|----------|------|--------------|
| `===` | **strict** equality | Must be the **same type** AND same value |
| `==`  | loose equality | Tries to **convert** the types, then compares |

```js
console.log(5 === 5);      // true
console.log(5 === "5");    // false  (number vs string)

console.log(5 == 5);       // true
console.log(5 == "5");     // true   ← string "5" gets converted to number 5

console.log(0 == false);   // true   ← false gets converted to 0
console.log("" == false);  // true   ← empty string is also "falsy 0"
console.log(null == undefined); // true ← special case
```

The `==` rules are so confusing that there are flowcharts to explain them. **Do not learn them.** Instead, follow this rule:

> **Always use `===` and `!==`. Never use `==` or `!=`.**

Strict equality means "what you wrote is what you compare." Your future self will thank you.

---

## 17.5 Logical operators

Logical operators combine `true`/`false` values.

### AND `&&` — both must be true

```js
const isLoggedIn = true;
const isAdmin = false;

console.log(isLoggedIn && isAdmin); // false (both must be true)
console.log(isLoggedIn && true);    // true

const age = 22;
console.log(age >= 18 && age < 60); // true (in working-age range)
```

### OR `||` — at least one must be true

```js
const hasCash = false;
const hasCard = true;
console.log(hasCash || hasCard); // true (one is enough)

const day = "Sunday";
console.log(day === "Saturday" || day === "Sunday"); // true (weekend)
```

### NOT `!` — flips a boolean

```js
const isHungry = true;
console.log(!isHungry);  // false

const list = [];
console.log(list.length === 0);  // true
console.log(!list.length);       // true (cleaner way to say "is empty")
```

You can combine them. Parentheses make intent clear:

```js
const isWeekend = true;
const isRaining = false;
const hasUmbrella = true;

const canGoOut = isWeekend && (!isRaining || hasUmbrella);
console.log(canGoOut); // true
```

---

## 17.6 The ternary operator `? :`

The ternary is a compact `if/else` that **returns a value**.

Syntax:

```
condition ? valueIfTrue : valueIfFalse
```

Example:

```js
const age = 17;
const status = age >= 18 ? "adult" : "minor";
console.log(status); // "minor"
```

Equivalent `if/else`:

```js
let status;
if (age >= 18) {
    status = "adult";
} else {
    status = "minor";
}
```

The ternary is brilliant for short, one-decision values. Avoid nesting ternaries inside ternaries — that gets unreadable fast.

```js
// Fine
const fee = isMember ? 0 : 50;

// Avoid — too hard to read
const fee = isMember ? 0 : isStudent ? 25 : isPensioner ? 10 : 50;
```

---

## 17.7 `if` / `else if` / `else`

The classic decision structure.

```js
const marks = 72;

if (marks >= 80) {
    console.log("Distinction");
} else if (marks >= 60) {
    console.log("First Division");
} else if (marks >= 40) {
    console.log("Pass");
} else {
    console.log("Fail");
}
```

**Expected output:**
```
First Division
```

A few rules:

1. The condition lives inside `( )`.
2. The body lives inside `{ }` — always use braces, even for one line.
3. JavaScript checks branches **top-down** and stops at the first match.
4. `else` (the catch-all) is optional. Same for `else if`.

### Nested `if`

You can put an `if` inside another `if`, but try to keep things flat where possible.

```js
const isLoggedIn = true;
const role = "admin";

if (isLoggedIn) {
    if (role === "admin") {
        console.log("Welcome, admin");
    } else {
        console.log("Welcome, user");
    }
} else {
    console.log("Please log in");
}
```

The same logic, flattened with `&&`:

```js
if (!isLoggedIn) {
    console.log("Please log in");
} else if (role === "admin") {
    console.log("Welcome, admin");
} else {
    console.log("Welcome, user");
}
```

Flatter code is usually easier to read.

---

## 17.8 The `switch` statement

When you have one variable being checked against many specific values, `switch` is cleaner than a long `else if` chain.

```js
const day = "Wed";

switch (day) {
    case "Mon":
        console.log("Start of the week");
        break;
    case "Tue":
    case "Wed":
    case "Thu":
        console.log("Midweek grind");
        break;
    case "Fri":
        console.log("Almost the weekend!");
        break;
    case "Sat":
    case "Sun":
        console.log("Weekend!");
        break;
    default:
        console.log("Unknown day");
}
```

**Expected output:**
```
Midweek grind
```

Important points:

- `switch` uses **strict equality** (`===`) to match cases.
- Cases without a `break` "fall through" to the next case. This is sometimes useful (see `Tue`/`Wed`/`Thu` above) but is often a bug.
- `default` is the `else` of `switch`.

> Rule: **always end each `case` with `break`** unless you mean to fall through.

---

## 17.9 Truthy and falsy values

In JavaScript, any value can be used in a boolean context (like an `if`). Values that act like `true` are **truthy**; values that act like `false` are **falsy**.

There are only **seven falsy values**. Memorise them.

| Falsy value | Notes |
|---|---|
| `false` | the literal `false` |
| `0` | the number zero |
| `-0` | yes, even negative zero |
| `0n` | bigint zero |
| `""` | empty string (single, double or backtick) |
| `null` | |
| `undefined` | |
| `NaN` | "Not a Number" |

**Everything else is truthy**, including:

- `"0"` — a string containing the character zero
- `"false"` — a string containing the word "false"
- `[]` — an empty array
- `{}` — an empty object

```js
if ("hello") console.log("yes");   // yes
if (0)       console.log("never"); // (skipped)
if ([])      console.log("yes");   // yes!  empty array is truthy
if ({})      console.log("yes");   // yes!  empty object is truthy
if ("")      console.log("never"); // (skipped)
```

This trips up everyone at first. To check whether an array is empty, do **not** rely on truthiness:

```js
const tasks = [];

if (tasks) console.log("truthy, but the list is empty!"); // truthy
if (tasks.length > 0) console.log("the list has items"); // safer
```

---

## 17.10 Putting it all together: a grade calculator

```js
const studentName = "Aarya";
const percentage = 76;

let grade;
if (percentage >= 80) {
    grade = "A";
} else if (percentage >= 70) {
    grade = "B";
} else if (percentage >= 60) {
    grade = "C";
} else if (percentage >= 40) {
    grade = "D";
} else {
    grade = "F";
}

const passed = percentage >= 40;
const message = passed
    ? `${studentName} passed with grade ${grade}.`
    : `${studentName} failed. Better luck next time.`;

console.log(message);
```

**Expected output:**
```
Aarya passed with grade B.
```

(Yes, that uses a template literal — sneak preview of Class 24.)

---

## Practice Exercises

1. **Grade calculator**: Write a program that takes a `percentage` variable and logs the letter grade using the scale above. Test it with 95, 82, 65, 41 and 12.

2. **Voting age**: Given a `country` (one of `"NP"`, `"UK"`, `"US"`) and an `age`, log whether the person can vote. The voting age is 18 in Nepal and the UK, and 18 in the US too — but make the program use a `switch` so it is easy to add other countries later.

3. **Truthy/falsy quiz**: Without running the code, predict what each of these logs. Then run them and check.

   ```js
   if ("0") console.log("a");
   if (0)   console.log("b");
   if ([])  console.log("c");
   if (null) console.log("d");
   if (" ") console.log("e");
   ```

4. **Ternary refactor**: Take this `if/else`:

   ```js
   let message;
   if (score >= 50) {
       message = "Pass";
   } else {
       message = "Fail";
   }
   ```

   Rewrite it as a one-line ternary.

---

## Key Takeaways
- `&&` means AND, `||` means OR, `!` means NOT.
- The ternary `cond ? a : b` returns a value — great for one-line decisions.
- **Always** use `===` and `!==`. Never `==` or `!=`.
- `if` / `else if` / `else` checks top-down; `switch` is for matching one variable against many literal values (don't forget `break`).
- The seven falsy values are: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. Everything else is truthy — including `[]` and `{}`.

Next class: loops — how to do something many times without writing it many times.
