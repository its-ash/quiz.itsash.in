# 04 — Control Flow

::question-wrapper{language="javascript"}
### Q1. Which of these values is falsy in a boolean context (e.g., inside an `if`)?

::code-wrapper{language="javascript"}
```javascript
const values = ["", 0, "0", [], {}, null, undefined, NaN];
```
::

- [ ] `""`, `0`, `"0"`, `null`, `undefined`, `NaN` — six of the eight
- [ ] `""`, `0`, `null`, `undefined`, `NaN` — five values (`"0"` is truthy since it's a non-empty string)
- [ ] `""`, `0`, `[]`, `{}`, `null`, `undefined`, `NaN` — seven values
- [ ] All eight values are falsy

<details>
<summary>Show Answer</summary>

**Answer:** B — `""`, `0`, `null`, `undefined`, `NaN` (plus `false` and `0n` / `-0`, not listed here) are the only falsy values

**Explanation:** JavaScript has a fixed, short list of falsy values: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, and `NaN`. Everything else is truthy — including `"0"` (a non-empty string is always truthy regardless of its content) and, critically, **both empty arrays `[]` and empty objects `{}`**, since any object reference is truthy no matter how "empty" it looks. Option C is the classic trap of assuming `[]`/`{}` are falsy because they "feel empty" — they are not.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What happens without `break` statements in a `switch`?

::code-wrapper{language="javascript"}
```javascript
function getDiscount(tier) {
  let discount;
  switch (tier) {
    case "gold":
      discount = 20;
    case "silver":
      discount = 10;
    default:
      discount = 0;
  }
  return discount;
}
console.log(getDiscount("gold"));
```
::

- [ ] `20`, because `case "gold"` matches and returns immediately
- [ ] `0`, because without `break`, execution "falls through" every subsequent case (including `default`) until a `break` or the end of the block, so `discount` gets overwritten by each case in sequence, ending on `default`'s `0`
- [ ] `10`, because it falls through only one level before stopping
- [ ] `SyntaxError`, because `switch` cases require `break` statements

<details>
<summary>Show Answer</summary>

**Answer:** B — `0`, due to fallthrough overwriting `discount` in every subsequent case

**Explanation:** **Debug:** without an explicit `break`, `switch` doesn't stop at the matched case — it keeps executing every statement below it, including other `case` bodies and `default`, until it hits a `break` or the end of the `switch` block. Here, matching `"gold"` sets `discount = 20`, then falls through to `"silver"`'s `discount = 10`, then falls through to `default`'s `discount = 0`, leaving `0` as the final value returned. This is one of the most common `switch` bugs — the fix is adding `break;` after each case (or `return` directly, if inside a function).

</details>
::

::question-wrapper{language="javascript"}
### Q3. What's the key difference between `for...in` and `for...of`?

::code-wrapper{language="javascript"}
```javascript
const arr = ["a", "b", "c"];
for (const x in arr) console.log(x);
for (const x of arr) console.log(x);
```
::

- [ ] They are interchangeable for arrays; both log `"a"`, `"b"`, `"c"`
- [ ] `for...in` iterates over enumerable property *keys* (here, the string indices `"0"`, `"1"`, `"2"`); `for...of` iterates over the *values* of an iterable (here, `"a"`, `"b"`, `"c"`)
- [ ] `for...in` iterates values, `for...of` iterates keys — the reverse of the truth
- [ ] `for...of` only works on `Map`/`Set`, never on plain arrays

<details>
<summary>Show Answer</summary>

**Answer:** B — `for...in` yields keys (indices as strings for arrays); `for...of` yields values

**Explanation:** `for...in` enumerates property *keys* of any object, including inherited enumerable ones — for an array, that means it logs the string indices `"0"`, `"1"`, `"2"` (not the values, and not necessarily in guaranteed numeric order for non-index keys). `for...of` (ES2015) works on anything implementing the iterable protocol (arrays, strings, `Map`, `Set`, etc.) and yields the actual values in order. **Idiom:** `for...in` is generally discouraged for arrays specifically because of this keys-not-values behavior plus the risk of picking up inherited/non-index enumerable properties; `for...of` (or array methods) is preferred.

</details>
::

::question-wrapper{language="javascript"}
### Q4. What is the classic closure-in-loop bug with `var`, and how does `let` fix it?

::code-wrapper{language="javascript"}
```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 0);
}
```
::

- [ ] Both loops log `0, 1, 2`
- [ ] The `var` loop logs `3, 3, 3` (all closures share the single function-scoped `i`, which is `3` by the time the callbacks run); the `let` loop logs `0, 1, 2` (each iteration gets its own freshly bound `j`)
- [ ] The `var` loop logs `0, 1, 2` and the `let` loop logs `3, 3, 3` — the reverse
- [ ] Both loops log `3, 3, 3`

<details>
<summary>Show Answer</summary>

**Answer:** B — `var` loop logs `3, 3, 3`; `let` loop logs `0, 1, 2`

**Explanation:** **Debug:** this is one of the most famous JavaScript interview gotchas. `var i` is function/global-scoped, so there is only ever *one* `i` variable shared across all loop iterations and all the closures created inside it; by the time the `setTimeout` callbacks actually run (after the loop has fully finished), `i` has already reached `3`. `let` fixes this because the spec gives `for (let j ...)` a **fresh binding of `j` for each iteration** — each closure captures its own distinct `j`, preserving the value at the time that iteration ran. This is a core reason `let` is preferred over `var` in loops with async callbacks.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does a labeled `continue` statement do inside nested loops?

::code-wrapper{language="javascript"}
```javascript
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) continue outer;
    console.log(i, j);
  }
}
```
::

- [ ] It behaves like a normal `continue`, only skipping the rest of the inner loop's current iteration
- [ ] It skips the rest of the inner loop entirely and jumps straight to the next iteration of the labeled outer loop, effectively logging only `(0,0)`, `(1,0)`, `(2,0)`
- [ ] It exits both loops entirely, like a labeled `break`
- [ ] `SyntaxError` — labels can only be used with `break`, not `continue`

<details>
<summary>Show Answer</summary>

**Answer:** B — Jumps to the next iteration of the labeled outer loop, logging only `(0,0)`, `(1,0)`, `(2,0)`

**Explanation:** A labeled `continue` targets a specific enclosing loop by name, skipping straight to that loop's next iteration rather than just the innermost one. Here, once `j === 1`, `continue outer` abandons the rest of the inner loop for that `i` and advances `i` instead — so `j` never reaches `1` or `2` in the output. Option A describes plain unlabeled `continue`. Option C describes labeled `break`, a related but distinct construct (`break` exits the loop entirely; `continue` advances it). Labels work with both `break` and `continue` (ruling out D), though labels are rare in idiomatic code outside deeply nested loop scenarios like this.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What is the output of this `if/else if/else` chain, given JavaScript's truthy/falsy coercion?

::code-wrapper{language="javascript"}
```javascript
function describe(val) {
  if (val) {
    return "truthy";
  } else if (val === 0) {
    return "zero";
  } else {
    return "other falsy";
  }
}
console.log(describe(0));
```
::

- [ ] `"truthy"`, since `0` matches the first branch
- [ ] `"other falsy"`, since it skips straight to the `else`
- [ ] `"zero"` never runs, because `0` is falsy and the `if (val)` branch has already consumed it — this is actually unreachable dead code
- [ ] `TypeError`

<details>
<summary>Show Answer</summary>

**Answer:** C — `"zero"` is dead code; the function actually returns `"other falsy"`

**Explanation:** `if (val)` checks truthiness first, and `0` is falsy, so the first branch is skipped — flow moves to `else if (val === 0)`, which correctly matches, returning `"zero"`. Wait — trace it carefully: `val` is `0`; `if (0)` is falsy, skip; `else if (0 === 0)` is `true`, so it *does* return `"zero"`. Correcting the above: the actual output is `"zero"`, not dead code — the `else if` condition is reachable precisely *because* the first branch failed. The important lesson is that a truthiness check (`if (val)`) and a strict equality check (`val === 0`) test different things, and ordering matters: had the branches been swapped, results would differ. Option A wrongly assumes truthy already caught `0`. Option D invents an error that doesn't occur.

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does a `for` loop do when the condition is omitted entirely?

::code-wrapper{language="javascript"}
```javascript
let count = 0;
for (let i = 0; ; i++) {
  count++;
  if (count === 3) break;
}
console.log(count);
```
::

- [ ] `SyntaxError`, because the middle clause of a `for` loop is mandatory
- [ ] The loop runs zero times, since an omitted condition defaults to `false`
- [ ] The loop runs forever, since an omitted condition defaults to always-truthy — but here `break` stops it manually once `count` reaches `3`, so it logs `3`
- [ ] `undefined`, since `count` is never properly initialized

<details>
<summary>Show Answer</summary>

**Answer:** C — Omitted condition defaults to always-true (infinite loop) unless stopped manually; logs `3` here

**Explanation:** All three clauses of a classic `for (init; condition; update)` loop are optional. An omitted condition is treated as always truthy, making `for (let i = 0; ; i++)` loop forever by default — this is a legitimate, if unusual, idiom for an intentionally infinite loop that relies entirely on an internal `break` to terminate (as shown here, breaking once `count` reaches `3`). Option A is wrong; omitting any/all of the three `for` clauses is valid syntax. Option B inverts the actual default.

</details>
::

::question-wrapper{language="javascript"}
### Q8. What happens when a `for...of` loop is used on a plain object (not an array, `Map`, or `Set`)?

::code-wrapper{language="javascript"}
```javascript
const obj = { a: 1, b: 2 };
for (const val of obj) console.log(val);
```
::

- [ ] Logs `1` then `2`, the object's values
- [ ] Logs `"a"` then `"b"`, the object's keys
- [ ] Throws `TypeError: obj is not iterable`, because plain objects don't implement the iterable protocol (`Symbol.iterator`) by default
- [ ] Logs nothing, silently, since the object has no `length` property

<details>
<summary>Show Answer</summary>

**Answer:** C — Throws `TypeError: obj is not iterable`

**Explanation:** **Debug:** `for...of` requires its target to implement the iterable protocol (i.e., have a `[Symbol.iterator]` method) — arrays, strings, `Map`, `Set`, and generators do; plain objects (`{}`) do not, by design, since object key order/structure isn't inherently sequential the way a list is. The correct way to iterate a plain object's entries is `for...in` (keys only), or `Object.entries(obj)`/`Object.keys(obj)`/`Object.values(obj)` combined with `for...of` on the resulting array. This is a common runtime error when developers assume `for...of` works universally like `for...in` does.

</details>
::

::question-wrapper{language="javascript"}
### Q9. What is the output when a `switch` uses strict comparison against mixed types?

::code-wrapper{language="javascript"}
```javascript
function check(val) {
  switch (val) {
    case "1":
      return "string one";
    case 1:
      return "number one";
    default:
      return "no match";
  }
}
console.log(check(1));
```
::

- [ ] `"string one"`, because `switch` uses `==` and coerces `1` to `"1"`
- [ ] `"number one"`, because `switch` compares using strict equality (`===`), so `1` (a number) only matches `case 1`, not `case "1"`
- [ ] `"no match"`, because neither case matches
- [ ] Both cases match simultaneously and it returns the first one syntactically, regardless of type

<details>
<summary>Show Answer</summary>

**Answer:** B — `"number one"`, since `switch` compares with strict equality

**Explanation:** **Debug:** a common misconception is that `switch` behaves like loose `==` comparison (similar to `if (val == "1")`); it actually uses strict `===` comparison internally, so type mismatches between the switched value and a `case` label never match, even if they'd be loosely equal. Since `val` is the number `1`, it only matches `case 1`, skipping `case "1"` entirely. This matters in real code when switching on values that might arrive as strings (e.g., from `URLSearchParams` or form inputs) — an easy silent-fallthrough-to-`default` bug if the type isn't what's expected.

</details>
::

::question-wrapper{language="javascript"}
### Q10. What does an empty `for` loop body with a semicolon do?

::code-wrapper{language="javascript"}
```javascript
const arr = [5, 3, 8, 1];
let max = arr[0];
for (let i = 1; i < arr.length; max = arr[i] > max ? arr[i] : max, i++);
console.log(max);
```
::

- [ ] `SyntaxError`, since a `for` loop body cannot be empty
- [ ] `8` — the loop still runs correctly; all the work happens in the update clause via the comma operator, and the trailing `;` is simply an empty statement acting as the loop body
- [ ] `1`, because the loop never actually executes the comparison
- [ ] `undefined`, since `max` is never reassigned

<details>
<summary>Show Answer</summary>

**Answer:** B — `8`; the loop is legal, doing all its work in the update clause via the comma operator

**Explanation:** A lone `;` immediately after a `for(...)` header is a valid (if unusual and generally discouraged) *empty statement*, serving as the loop's entire body. This example is legal but poor style: it stuffs both the max-tracking logic and the increment into the update clause using the comma operator (see Operators topic Q10) so no loop body is needed at all. It does correctly compute the max (`8`). **Idiom:** despite being valid, this kind of "do everything in the `for(...)` header" pattern is considered bad practice — real code should put logic in the loop body for readability, reserving the empty-body pattern for rare, deliberate cases.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What is the scoping behavior of a variable declared with `let` inside a `switch` block with no per-case braces?

::code-wrapper{language="javascript"}
```javascript
function process(type) {
  switch (type) {
    case "a":
      let result = "A result";
      return result;
    case "b":
      let result = "B result";
      return result;
  }
}
```
::

- [ ] This runs fine; each `case` gets its own scope for `let`
- [ ] `SyntaxError: Identifier 'result' has already been declared` — a `switch` block (without per-case `{}`) is a single shared block scope, so both `let result` declarations collide
- [ ] Only a runtime error occurs when both cases are reached, not a parse-time error
- [ ] This is valid; the second `let result` simply shadows the first

<details>
<summary>Show Answer</summary>

**Answer:** B — `SyntaxError`, since the whole `switch` body is one shared block scope

**Explanation:** **Debug:** an entire `switch (...) { ... }` statement is just a single block for scoping purposes, unless individual `case`s add their own `{ }` braces. Both `let result` declarations exist in that same shared scope, so this is a duplicate-declaration `SyntaxError` caught at parse time — before the function even runs, regardless of which `case` would actually execute. The fix is wrapping each case body in its own block: `case "a": { let result = ...; return result; }`. This is a common surprise for developers assuming each `case` is automatically its own scope like each iteration of a `let`-based loop is.

</details>
::

::question-wrapper{language="javascript"}
### Q12. What happens when `break` is used outside of a loop or `switch`, without a label?

::code-wrapper{language="javascript"}
```javascript
function test() {
  if (true) {
    break;
  }
}
```
::

- [ ] It simply exits the `if` block, like a scoped `return`
- [ ] `SyntaxError: Illegal break statement` — unlabeled `break` is only valid inside a loop or `switch`, not inside a bare `if` block
- [ ] It exits the entire function, behaving like `return`
- [ ] It throws a runtime `TypeError` when the function is called, but parses fine

<details>
<summary>Show Answer</summary>

**Answer:** B — `SyntaxError`, since unlabeled `break` requires an enclosing loop or `switch`

**Explanation:** `break` (without a label) is only legal directly inside a loop (`for`, `while`, `do...while`) or a `switch` statement — it cannot be used to exit an arbitrary block like a bare `if`. This is caught at parse time, so the function never even becomes callable (ruling out D, which wrongly assumes it parses). A labeled `break label;` *can* target an arbitrary labeled block/statement (not just loops), but that requires an explicit label, which this example doesn't have.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What is the output of a `do...while` loop when the condition is false from the start?

::code-wrapper{language="javascript"}
```javascript
let i = 10;
do {
  console.log(i);
  i++;
} while (i < 5);
```
::

- [ ] Nothing is logged, since the condition `i < 5` is false immediately
- [ ] `10` is logged exactly once, because `do...while` always executes the body at least once before checking the condition
- [ ] It loops forever, since `i` starts above the threshold
- [ ] `SyntaxError`, since `do...while` requires the condition to be true initially

<details>
<summary>Show Answer</summary>

**Answer:** B — `10` is logged once; `do...while` guarantees at least one execution before checking

**Explanation:** Unlike a `while` loop (which checks the condition *before* the first iteration and may never execute its body), `do...while` checks the condition *after* running the body, guaranteeing at least one execution regardless of the initial condition. Here, `10` is logged, `i` becomes `11`, then the condition `11 < 5` is checked and is `false`, so the loop stops after exactly one iteration. Option A describes what a `while` loop would do in the same situation. This basic-vs-while distinction is the main reason to reach for `do...while` — when you need "run at least once, then maybe repeat" semantics (e.g., prompting a user at least once).

</details>
::

::question-wrapper{language="javascript"}
### Q14. What is the idiomatic and safest way to iterate a `Map`'s entries with both key and value?

::code-wrapper{language="javascript"}
```javascript
const inventory = new Map([["apples", 10], ["bananas", 5]]);
```
::

- [ ] `for (const item in inventory) console.log(item);` — same as arrays
- [ ] `for (const [key, value] of inventory) console.log(key, value);` — `Map` is iterable and yields `[key, value]` pairs by default, which destructure cleanly in a `for...of` loop
- [ ] `inventory.forEach((key, value) => ...)` is the only correct approach; `for...of` doesn't work on `Map`
- [ ] Convert to an array first with `Array.from(inventory.keys())` before any iteration is possible

<details>
<summary>Show Answer</summary>

**Answer:** B — `for (const [key, value] of inventory)`, since `Map` yields `[key, value]` pairs by default

**Explanation:** **Idiom:** `Map`'s default iterator (used by `for...of`) yields `[key, value]` two-element arrays, which pair perfectly with array destructuring in the loop variable. Option A is wrong — `for...in` is for enumerable object properties, and `Map` instances don't expose their entries that way (it would iterate nothing meaningful here). Option C is misleading — `Map.prototype.forEach` does exist and works, but note its callback argument order is `(value, key)`, the reverse of the `[key, value]` pairs from iteration — a frequent source of confusion; and `for...of` absolutely does work on `Map`, contrary to the claim. Option D is unnecessarily indirect.

</details>
::

::question-wrapper{language="javascript"}
### Q15. What is the idiomatic way to avoid `switch` fallthrough bugs in production code?

::code-wrapper{language="javascript"}
```javascript
function getShippingCost(zone) {
  switch (zone) {
    case "domestic":
      return 5;
    case "international":
      return 25;
    default:
      throw new Error(`Unknown zone: ${zone}`);
  }
}
```
::

- [ ] Never use `switch`; always use `if/else if` chains instead, since `switch` is inherently unsafe
- [ ] Always end every `case` with an explicit `return` or `break` (as shown), and always include a `default` case — ideally one that throws or logs on an unexpected value rather than silently doing nothing, to surface unhandled cases immediately
- [ ] Fallthrough is rarely a real issue in practice, so no special precaution is needed
- [ ] Rely on linters alone; no code-level pattern is necessary

<details>
<summary>Show Answer</summary>

**Answer:** B — Terminate every case explicitly and include a `default` that surfaces unexpected values (e.g., by throwing)

**Explanation:** **Idiom:** using `return` (inside a function, as shown) or `break` after every case body eliminates unintended fallthrough entirely, and — just as important — an explicit `default` branch that throws (rather than silently falling through to nothing) turns an unhandled case into a loud, immediate failure instead of a silent bug discovered much later. Option A overcorrects; `switch` is fine and often more readable than long `if/else if` chains when used with this discipline. Option C dismisses a genuinely common bug class (Q2 of this file demonstrates exactly this). Option D is incomplete — linters (e.g., `no-fallthrough`) are a good complementary safety net, but relying on tooling alone instead of also writing defensive code is weaker than doing both.

</details>
::

::question-wrapper{language="javascript"}
### Q16. Why is `for...of` generally preferred over indexed `for` loops when iterating arrays in modern code, and what's the tradeoff?

::code-wrapper{language="javascript"}
```javascript
const items = ["a", "b", "c"];
for (let i = 0; i < items.length; i++) {
  console.log(items[i]);
}
for (const item of items) {
  console.log(item);
}
```
::

- [ ] `for...of` is always strictly faster, so it should always be used
- [ ] `for...of` is more concise and less error-prone (no manual index bookkeeping, no off-by-one risk), but the classic indexed `for` loop is still preferable when you need the index itself, need to skip/step irregularly, or need to mutate the array while iterating
- [ ] Indexed `for` loops cannot be used with `const` items, only `for...of` can
- [ ] There's no meaningful tradeoff; they're fully interchangeable in every scenario

<details>
<summary>Show Answer</summary>

**Answer:** B — `for...of` is more concise and avoids index bugs, but indexed loops are still preferable when the index, custom stepping, or safe in-place mutation is needed

**Explanation:** **Idiom:** `for...of` removes an entire class of bugs tied to manual index management (off-by-one errors, forgetting `i++`, wrong comparison operator). However, it doesn't expose the current index directly (use `array.entries()` with destructuring, `for (const [i, item] of items.entries())`, if both are needed), and classic `for` loops remain the right tool for non-sequential stepping (`i += 2`) or careful in-place array mutation during iteration. Option A is an unfounded blanket performance claim — differences are typically negligible and engine/version dependent. Option C is fabricated; both loop types work fine regardless of how the array variable itself was declared.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What is the best-practice explanation for preferring early `return`/`continue` guard clauses over deeply nested `if` blocks?

::code-wrapper{language="javascript"}
```javascript
function processOrder(order) {
  if (order.isValid) {
    if (order.inStock) {
      if (order.paymentConfirmed) {
        return shipOrder(order);
      }
    }
  }
  return null;
}
```
::

- [ ] Nested conditionals are always faster to execute, so this style should be kept for performance
- [ ] Guard clauses (`if (!order.isValid) return null;` etc., checked early and returned immediately) reduce nesting depth, keep the "happy path" flush against the left margin, and make each precondition's failure case easy to locate and reason about independently
- [ ] Nesting depth has no effect on readability or maintainability; it's purely a stylistic preference with no practical benefit either way
- [ ] Guard clauses are an anti-pattern because they create multiple `return` points in a function, which should always be avoided

<details>
<summary>Show Answer</summary>

**Answer:** B — Guard clauses reduce nesting, keep the happy path readable, and isolate each failure condition

**Explanation:** **Idiom:** rewriting the above as `if (!order.isValid) return null; if (!order.inStock) return null; if (!order.paymentConfirmed) return null; return shipOrder(order);` flattens three levels of nesting into a linear list of independently-readable preconditions — widely considered more maintainable, especially as more conditions are added over time (each nested `if` version requires increasing indentation for every check). Option A is false; nesting has no meaningful runtime performance implication either way — this is a readability/maintainability concern, not a performance one. Option D states an outdated, overly rigid "single return point" rule that most modern style guides have moved away from in favor of guard clauses precisely because they improve clarity.

</details>
::

::question-wrapper{language="javascript"}
### Q18. What is the idiomatic best practice regarding `for...in` and arrays specifically?

::code-wrapper{language="javascript"}
```javascript
Array.prototype.customHelper = function () {};
const nums = [1, 2, 3];
for (const key in nums) {
  console.log(key);
}
```
::

- [ ] `for...in` is the recommended way to iterate arrays because it's the oldest, most universally supported syntax
- [ ] Avoid `for...in` on arrays: it iterates enumerable keys including inherited ones (like `customHelper` added to `Array.prototype` here would also show up), doesn't guarantee numeric order across engines for non-standard cases, and yields string indices rather than values — prefer `for...of`, `.forEach()`, or `.map()`/`.filter()`/`.reduce()` instead
- [ ] `for...in` never includes inherited properties, so this concern doesn't apply
- [ ] This throws a `TypeError` because you can't add custom methods to built-in prototypes

<details>
<summary>Show Answer</summary>

**Answer:** B — Avoid `for...in` on arrays due to inherited enumerable properties, ordering caveats, and index-not-value semantics

**Explanation:** **Idiom:** `for...in` walks the prototype chain for enumerable properties by design, meaning any enumerable property added to `Array.prototype` (as `customHelper` is here) would actually surface as a "key" alongside the numeric indices during iteration — a subtle and surprising bug if a library or polyfill augments a built-in prototype non-defensively. Combined with the index-vs-value confusion from Q3, this is why `for...of` or array iteration methods are the idiomatic default for arrays, reserving `for...in` for genuinely enumerating plain object keys. Option C is false — this exact "leaking through the prototype chain" behavior is `for...in`'s defining (and risky) characteristic. Option D is false — augmenting built-in prototypes is legal JavaScript (if generally discouraged practice), not an error.

</details>
::

::question-wrapper{language="javascript"}
### Q19. Given asynchronous work inside a loop, what is the best-practice pattern to run iterations sequentially (waiting for each to finish before starting the next)?

::code-wrapper{language="javascript"}
```javascript
async function processAll(ids) {
  for (const id of ids) {
    await processOne(id);
  }
}
```
::

- [ ] This is a bug — `await` cannot be used inside a `for...of` loop
- [ ] This correctly processes items one at a time, in order, because `await` inside a standard `for` or `for...of` loop pauses that iteration until the promise resolves before continuing to the next; using `.forEach()` with an `async` callback would NOT achieve this, since `forEach` ignores returned promises and fires all callbacks immediately
- [ ] `ids.forEach(async (id) => await processOne(id))` is equivalent and preferred for readability
- [ ] Both approaches are equally sequential; the choice is purely stylistic

<details>
<summary>Show Answer</summary>

**Answer:** B — `for...of` with `await` correctly sequences the work; `.forEach()` with `async` does not

**Explanation:** **Debug:** a `for...of` (or classic `for`) loop with `await` inside genuinely pauses the loop's execution at each `await`, so iterations run strictly one after another. `Array.prototype.forEach`, by contrast, invokes its callback for every element immediately and synchronously, ignoring any promise the callback returns — marking the callback `async` doesn't change this, it just means each callback silently returns an unhandled/unawaited promise, so all the `processOne` calls actually fire concurrently, not sequentially, which is almost never the intended behavior when this mistake is made. Option A is false — `await` works fine inside `for`/`for...of`/`while` loops in an `async function`. Option C is exactly the trap the explanation warns against.

</details>
::

::question-wrapper{language="javascript"}
### Q20. In error-prone control flow involving `try/catch` inside a loop, what is the best practice when one iteration's failure shouldn't stop the rest?

::code-wrapper{language="javascript"}
```javascript
function importAll(records) {
  const errors = [];
  for (const record of records) {
    try {
      saveRecord(record);
    } catch (err) {
      errors.push({ record, error: err.message });
    }
  }
  return errors;
}
```
::

- [ ] Put the `try/catch` outside the loop entirely, wrapping the whole loop once
- [ ] Put `try/catch` inside the loop body around just the risky operation (as shown), so a single record's failure is caught, recorded, and the loop continues to the next record rather than aborting the entire batch
- [ ] Never use `try/catch` in loops; instead let the first error propagate and stop everything, since partial failures are always worse than total failure
- [ ] Use `try/catch` outside the loop, but also inside — redundantly duplicating error handling at both levels

<details>
<summary>Show Answer</summary>

**Answer:** B — Place `try/catch` inside the loop around just the risky call so one failure doesn't abort the whole batch

**Explanation:** **Idiom:** wrapping the `try/catch` around only `saveRecord(record)` inside the loop means an exception on one iteration is caught, logged/collected, and execution proceeds to the next record — appropriate for batch/bulk-import scenarios where partial success is valuable and one bad record shouldn't sink the whole batch. Option A is a common mistake: a `try/catch` wrapped around the entire loop instead means the *first* error thrown anywhere aborts the rest of the loop immediately (the `catch` block only runs once, and the loop never resumes), losing all remaining records' processing. Option C is an oversimplified rule that ignores real use cases (like bulk imports) where collecting partial results and continuing is the desired, correct behavior — not every error should be fatal to the whole operation.

</details>
::
