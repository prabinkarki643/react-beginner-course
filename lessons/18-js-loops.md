# Class 18: Loops

> **Time budget**: 1 hour (≈ 45 min teach + 15 min practice)
> **Prerequisites**: Class 17 — operators and conditionals

If conditionals let our programs **make decisions**, loops let our programs **repeat work**. Without loops, you would have to write `console.log` a hundred times to print 1 to 100. With a loop, three lines do the job.

## What You Will Learn
- The classic `for` loop and when to use it
- `while` and `do…while` loops
- `for…of` for arrays and `for…in` for object keys
- The `break` and `continue` keywords
- Common pitfalls — infinite loops and off-by-one errors

---

## 18.1 The repetition problem

Suppose we want to greet five students:

```js
console.log("Hello, student 1");
console.log("Hello, student 2");
console.log("Hello, student 3");
console.log("Hello, student 4");
console.log("Hello, student 5");
```

Tedious — and what if there are 500 students? Loops solve this.

---

## 18.2 The `for` loop

The `for` loop is the workhorse of JS. It has three parts inside its parentheses, separated by semicolons:

```
for (initialise; condition; update) {
    // body
}
```

```js
for (let i = 1; i <= 5; i++) {
    console.log("Hello, student " + i);
}
```

**Expected output:**
```
Hello, student 1
Hello, student 2
Hello, student 3
Hello, student 4
Hello, student 5
```

Let us decode the three parts:

| Part | Code | Meaning |
|------|------|---------|
| Initialise | `let i = 1` | runs **once**, at the start |
| Condition | `i <= 5` | checked **before each iteration**; the loop runs while this is true |
| Update | `i++` | runs **after each iteration** |

The variable `i` (short for "index" or "iterator") is a convention. You can call it anything, but `i`, `j`, `k` are traditional for nested loops.

### Counting down

```js
for (let i = 10; i > 0; i--) {
    console.log(i);
}
console.log("Lift-off!");
```

**Expected output:**
```
10
9
8
7
6
5
4
3
2
1
Lift-off!
```

### Stepping by more than one

```js
for (let i = 0; i <= 20; i += 5) {
    console.log(i);
}
```

**Expected output:**
```
0
5
10
15
20
```

### Looping through an array (the old way)

```js
const fruits = ["apple", "banana", "cherry"];

for (let i = 0; i < fruits.length; i++) {
    console.log(i, fruits[i]);
}
```

**Expected output:**
```
0 apple
1 banana
2 cherry
```

Notice we use `<` (not `<=`) here because array indices start at 0 and go up to `length - 1`. More on this in 18.7.

---

## 18.3 The `while` loop

A `while` loop runs **as long as a condition is true**. Use it when you do not know in advance how many times you will need to loop.

```js
let battery = 100;

while (battery > 0) {
    console.log("Battery at " + battery + "%");
    battery -= 25;
}
console.log("Battery dead!");
```

**Expected output:**
```
Battery at 100%
Battery at 75%
Battery at 50%
Battery at 25%
Battery dead!
```

The condition is checked **before** each pass. If it is false from the start, the body never runs:

```js
let x = 10;
while (x < 5) {
    console.log("Never printed");
}
```

---

## 18.4 The `do…while` loop

A `do…while` loop is the same as `while`, except it checks the condition **after** the body runs. This guarantees at least one iteration.

```js
let attempts = 0;
let password;

do {
    // pretend we ask the user
    password = "letmein";
    attempts++;
    console.log("Attempt " + attempts);
} while (password !== "secret" && attempts < 3);

console.log("Stopped after " + attempts + " attempts");
```

**Expected output:**
```
Attempt 1
Attempt 2
Attempt 3
Stopped after 3 attempts
```

`do…while` is less common than `while`. Reach for it when you need the body to run **at least once** before any check (e.g. "ask the user, then check if their answer is valid").

---

## 18.5 `for…of` — looping through arrays cleanly

The classic `for` loop works, but it is wordy. `for…of` gives you each item directly:

```js
const fruits = ["apple", "banana", "cherry"];

for (const fruit of fruits) {
    console.log(fruit);
}
```

**Expected output:**
```
apple
banana
cherry
```

No index, no `length`, no `[i]` — much nicer. Use `for…of` when you only care about the values.

If you need the index too, use `.entries()`:

```js
const fruits = ["apple", "banana", "cherry"];

for (const [index, fruit] of fruits.entries()) {
    console.log(index, fruit);
}
```

**Expected output:**
```
0 apple
1 banana
2 cherry
```

(That `[index, fruit]` syntax is destructuring — we will cover it properly in Class 21.)

---

## 18.6 `for…in` — looping through object keys

`for…in` is the equivalent for **objects**: it gives you each key.

```js
const student = {
    name: "Aarya",
    age: 20,
    course: "BCA"
};

for (const key in student) {
    console.log(key + ": " + student[key]);
}
```

**Expected output:**
```
name: Aarya
age: 20
course: BCA
```

> Important: **`for…of` is for arrays. `for…in` is for object keys.** Mixing them up is a very common beginner bug. Using `for…in` on an array can give you surprising results, so do not do it.

---

## 18.7 `break` and `continue`

These two keywords give you fine control inside loops.

### `break` — get out of the loop immediately

```js
const numbers = [3, 8, 12, 21, 50, 77];

for (const n of numbers) {
    if (n > 20) {
        console.log("Found one bigger than 20: " + n);
        break; // stop the loop
    }
    console.log("checking " + n);
}
```

**Expected output:**
```
checking 3
checking 8
checking 12
Found one bigger than 20: 21
```

### `continue` — skip the rest of this iteration, move to the next

```js
for (let i = 1; i <= 6; i++) {
    if (i % 2 === 0) continue; // skip even numbers
    console.log(i);
}
```

**Expected output:**
```
1
3
5
```

---

## 18.8 Common pitfalls

### Pitfall 1: the infinite loop

If your condition never becomes false, your loop runs forever. The browser tab will freeze.

```js
// ⚠️ DO NOT RUN — infinite loop
let i = 0;
while (i < 5) {
    console.log(i);
    // forgot to update i — it stays at 0 forever
}
```

**Always make sure each iteration moves you closer to the exit condition.** If your browser freezes, close the tab.

### Pitfall 2: off-by-one errors

```js
const items = ["a", "b", "c"]; // indices 0, 1, 2  (length is 3)

// ❌ Wrong — tries to access items[3], which is undefined
for (let i = 0; i <= items.length; i++) {
    console.log(items[i]);
}
// Output: a b c undefined

// ✅ Correct
for (let i = 0; i < items.length; i++) {
    console.log(items[i]);
}
```

The rule of thumb: use `<` with `.length`, not `<=`. Or just use `for…of` and stop worrying about it.

### Pitfall 3: modifying the array while looping

Adding or removing items from the array you are looping over almost always causes bugs. Build a new array instead (we will see how with `.map` and `.filter` in Class 20).

---

## 18.9 A bigger example: multiplication table

```js
const number = 7;

console.log("Multiplication table for " + number);
console.log("-----------------------------");

for (let i = 1; i <= 10; i++) {
    const result = number * i;
    console.log(number + " x " + i + " = " + result);
}
```

**Expected output:**
```
Multiplication table for 7
-----------------------------
7 x 1 = 7
7 x 2 = 14
7 x 3 = 21
7 x 4 = 28
7 x 5 = 35
7 x 6 = 42
7 x 7 = 49
7 x 8 = 56
7 x 9 = 63
7 x 10 = 70
```

### Nested loops

You can put loops inside loops. Here we print the tables for 2, 3 and 4:

```js
for (let table = 2; table <= 4; table++) {
    console.log("--- Table of " + table + " ---");
    for (let i = 1; i <= 5; i++) {
        console.log(table + " x " + i + " = " + (table * i));
    }
}
```

**Expected output:**
```
--- Table of 2 ---
2 x 1 = 2
2 x 2 = 4
2 x 3 = 6
2 x 4 = 8
2 x 5 = 10
--- Table of 3 ---
3 x 1 = 3
...
```

A nested loop runs the inner loop **completely** for each step of the outer loop.

---

## Practice Exercises

1. **Sum 1 to 100**: Use a `for` loop to add the numbers from 1 to 100 and log the total. (Expected answer: `5050`.)

2. **First 100 primes**: A prime number is divisible only by 1 and itself. Write a program that prints the first 100 prime numbers. Hint: use a `while` loop that keeps going until you have collected 100 primes, and a helper `for` loop inside to test divisibility.

3. **FizzBuzz** (the classic): For numbers 1 to 30:
   - If divisible by 3 **and** 5, print `"FizzBuzz"`.
   - Else if divisible by 3, print `"Fizz"`.
   - Else if divisible by 5, print `"Buzz"`.
   - Otherwise print the number.

4. **Print object keys**: Use `for…in` to print every key and value of this object:

   ```js
   const profile = { name: "Aarya", age: 20, city: "Kathmandu", course: "BCA" };
   ```

---

## Key Takeaways
- Use `for` when you know how many times to loop, `while` when you do not.
- `do…while` always runs at least once.
- `for…of` iterates array values; `for…in` iterates object keys — do not mix them up.
- `break` exits the loop; `continue` jumps to the next iteration.
- Watch for infinite loops (forgot to update the counter) and off-by-one errors (used `<=` instead of `<`).

Next class: functions — packaging code so we can reuse it.
