# 05 — Functions & Scope

::question-wrapper
### Q1. Which statement correctly describes the difference between how function declarations and function expressions are hoisted?

- [ ] Function declarations are hoisted complete with their implementation and can be called before their textual position; function expressions are only as hoisted as the variable they're assigned to (or not hoisted at all for `const`/`let`)
- [ ] Neither declarations nor expressions are hoisted; both must appear in the source before they're used
- [ ] Both are fully hoisted with their implementation, so both can be called before their textual position in the code
- [ ] Function expressions are hoisted complete with their implementation, but declarations are not hoisted at all

<details>
<summary>Show Answer</summary>

**Answer:** A — Function declarations are hoisted complete with their implementation and can be called before their textual position; function expressions are only as hoisted as the variable they're assigned to (or not hoisted at all for `const`/`let`)

**Explanation:** During the creation phase, the JS engine hoists an entire `function foo() {}` declaration — name and body — to the top of its enclosing scope, so it's fully callable before its line in the source. A function *expression*, like `const foo = function () {}` or `var foo = function () {}`, is just a variable assignment; only the variable binding follows that variable's own hoisting rules (`var` → hoisted and `undefined`, `let`/`const` → hoisted into the temporal dead zone), and the function value itself only exists once the assignment line runs. Options C and D describe hoisting behavior that doesn't apply to either form.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does this log?

::code-wrapper{language="javascript"}
```javascript
console.log(typeof greet);
greet();

function greet() {
  console.log("hello from a declaration");
}
```
::

- [ ] `"undefined"` then a `TypeError`
- [ ] `"function"` then `"hello from a declaration"`
- [ ] a `ReferenceError` before anything logs
- [ ] `"function"` then a `ReferenceError`

<details>
<summary>Show Answer</summary>

**Answer:** B — `"function"` then `"hello from a declaration"`

**Explanation:** Because `greet` is a function *declaration*, both its name and its full implementation are hoisted to the top of the module/script scope during the creation phase. By the time `console.log(typeof greet)` runs, `greet` already refers to a callable function, so `typeof greet` is `"function"`, and calling `greet()` immediately afterward executes normally even though the call appears above the declaration in the source.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What does this log?

::code-wrapper{language="javascript"}
```javascript
console.log(typeof sayHi);
sayHi();

var sayHi = function () {
  console.log("hi from an expression");
};
```
::

- [ ] `"function"` then `"hi from an expression"`
- [ ] `"undefined"` then `"hi from an expression"`
- [ ] `"undefined"` then `TypeError: sayHi is not a function`
- [ ] `ReferenceError: sayHi is not defined`

<details>
<summary>Show Answer</summary>

**Answer:** C — `"undefined"` then `TypeError: sayHi is not a function`

**Explanation:** **Debug:** only the `var sayHi` declaration is hoisted here, not the function it's assigned — `var` bindings are hoisted and pre-initialized to `undefined`, so `typeof sayHi` is `"undefined"` at the top. The actual function value isn't attached until the assignment line executes, so calling `sayHi()` before that line tries to invoke `undefined`, throwing a `TypeError`. This is the key contrast with Q2: swapping a declaration for a `var`-assigned expression turns "works before its position" into a runtime crash.

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does this log?

::code-wrapper{language="javascript"}
```javascript
function reportStatus() {
  console.log(status);
  var status = "pending";
  console.log(status);
}

reportStatus();
```
::

- [ ] `"pending"` then `"pending"`
- [ ] a `ReferenceError` on the first `console.log`
- [ ] `undefined` then `undefined`
- [ ] `undefined` then `"pending"`

<details>
<summary>Show Answer</summary>

**Answer:** D — `undefined` then `"pending"`

**Explanation:** `var status` is hoisted to the top of `reportStatus`'s function body and initialized to `undefined` before any code runs, so the first `console.log(status)` reads that placeholder value rather than throwing. Only the assignment `status = "pending"` happens where it's textually written, so the second `console.log` sees the updated value. Contrast this with Q5 below, where `let` behaves very differently for the same shape of code.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does this log?

::code-wrapper{language="javascript"}
```javascript
function reportStatus() {
  console.log(status);
  let status = "pending";
  console.log(status);
}

reportStatus();
```
::

- [ ] `ReferenceError: Cannot access 'status' before initialization`
- [ ] `undefined` then `"pending"`
- [ ] `"pending"` then `"pending"`
- [ ] `undefined` then `undefined`

<details>
<summary>Show Answer</summary>

**Answer:** A — `ReferenceError: Cannot access 'status' before initialization`

**Explanation:** **Safety:** `let` (and `const`) bindings are hoisted too, but unlike `var` they are not initialized to `undefined` — they sit in the "temporal dead zone" (TDZ) from the top of the block until their declaration line actually runs. Reading `status` anywhere in that window throws a `ReferenceError` rather than silently returning `undefined`. This is a deliberate safety improvement over `var`: it turns an easy-to-miss ordering bug into a loud, immediate failure.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What does this log?

::code-wrapper{language="javascript"}
```javascript
const factorial = function calc(n) {
  return n <= 1 ? 1 : n * calc(n - 1);
};

console.log(factorial(5));
console.log(typeof calc);
```
::

- [ ] `120` then `"function"`
- [ ] `120` then `"undefined"`
- [ ] a `ReferenceError` on the `factorial(5)` call
- [ ] `120` then it throws `ReferenceError: calc is not defined`

<details>
<summary>Show Answer</summary>

**Answer:** B — `120` then `"undefined"`

**Explanation:** This is a *named function expression*: the name `calc` is bound only inside the function's own body (useful for recursion), it is never added to the enclosing scope. So `factorial(5)` works fine and recurses via `calc` internally, correctly producing `120`. But outside, `calc` was never declared, so `typeof calc` safely evaluates to `"undefined"` — `typeof` never throws on an unresolvable identifier, it only returns `"undefined"`, unlike directly referencing `calc` (which *would* throw a `ReferenceError`).

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does this log?

::code-wrapper{language="javascript"}
```javascript
let compute = function run(n) {
  if (n <= 0) return 0;
  return n + run(n - 1);
};

const backup = compute;
compute = null;

console.log(backup(3));
```
::

- [ ] `TypeError: run is not a function`
- [ ] `6`
- [ ] `null`
- [ ] `0`

<details>
<summary>Show Answer</summary>

**Answer:** B — `6`

**Explanation:** Because `compute` is a *named* function expression, the recursive calls inside its body reference the stable internal name `run`, not the outer `compute` variable. Reassigning `compute = null` has no effect on what `run` points to inside the function — `run` always refers back to the function itself. So `backup(3)` still recurses correctly: `3 + 2 + 1 + 0 = 6`. This is exactly why named function expressions are the safer choice for recursion over relying on an outer, reassignable binding.

</details>
::

::question-wrapper{language="javascript"}
### Q8. What does this log?

::code-wrapper{language="javascript"}
```javascript
let count = 10;

function addCount(base, extra = count) {
  return base + extra;
}

console.log(addCount(1));
count = 100;
console.log(addCount(1));
```
::

- [ ] `11` then `11`
- [ ] `101` then `101`
- [ ] `NaN` then `NaN`
- [ ] `11` then `101`

<details>
<summary>Show Answer</summary>

**Answer:** D — `11` then `101`

**Explanation:** **Debug:** a default parameter expression is not evaluated once and cached when the function is defined — it's re-evaluated fresh every time the function is called with that argument omitted. The first call reads the current `count` (`10`), giving `1 + 10 = 11`. After `count` is reassigned to `100`, the second call re-evaluates `extra = count` and picks up the new value, giving `1 + 100 = 101`. Treating default values as "baked in once" is a common but incorrect assumption.

</details>
::

::question-wrapper{language="javascript"}
### Q9. What does this log?

::code-wrapper{language="javascript"}
```javascript
function createRange(start, end = start + 10, step = (end - start) / 5) {
  return { start, end, step };
}

console.log(createRange(0));
```
::

- [ ] `{ start: 0, end: 10, step: 2 }`
- [ ] `{ start: 0, end: undefined, step: NaN }`
- [ ] `ReferenceError: Cannot access 'end' before initialization`
- [ ] `{ start: 0, end: 10, step: undefined }`

<details>
<summary>Show Answer</summary>

**Answer:** A — `{ start: 0, end: 10, step: 2 }`

**Explanation:** Default parameters are evaluated left to right, and each one can freely reference any parameter to its left that has already been initialized. `end` defaults to `start + 10 = 10`, and by the time `step`'s default runs, both `start` and `end` are already initialized, so `step = (10 - 0) / 5 = 2`. This only works in this left-to-right direction — see Q10 for what happens when a default tries to reference a parameter declared *after* it.

</details>
::

::question-wrapper{language="javascript"}
### Q10. What happens when this runs?

::code-wrapper{language="javascript"}
```javascript
function build(a = b + 1, b = 5) {
  return a + b;
}

console.log(build());
```
::

- [ ] Logs `6`
- [ ] Throws `ReferenceError: Cannot access 'b' before initialization`
- [ ] Logs `NaN`
- [ ] Logs `11`

<details>
<summary>Show Answer</summary>

**Answer:** B — Throws `ReferenceError: Cannot access 'b' before initialization`

**Explanation:** **Safety:** parameters form their own scope, evaluated in declaration order, and each parameter sits in a temporal dead zone until its own initialization runs — just like `let` inside a block. When `a`'s default tries to read `b`, `b` hasn't been initialized yet, so it's still in the TDZ and the read throws. Defaults are not hoisted or resolved out of order; they only work looking backward at already-initialized parameters (Q9), never forward.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What does this log?

::code-wrapper{language="javascript"}
```javascript
function connect(host, port = 8080) {
  return `${host}:${port}`;
}

console.log(connect("db", null));
```
::

- [ ] `"db:8080"`
- [ ] `"db:undefined"`
- [ ] `"db:null"`
- [ ] `TypeError: cannot use null as port`

<details>
<summary>Show Answer</summary>

**Answer:** C — `"db:null"`

**Explanation:** **Debug:** default parameters only kick in when the argument is exactly `undefined` (including simply being omitted) — not for any other "empty-ish" value. `null` is a perfectly valid, explicitly passed value, so `port` is set to `null` and the default `8080` is never used; the template literal then stringifies `null` to `"db:null"`. This is a common trap for anyone assuming defaults behave like a general null-coalescing fallback.

</details>
::

::question-wrapper{language="javascript"}
### Q12. What does this log?

::code-wrapper{language="javascript"}
```javascript
function createTimer(label, retries = 3) {
  return `${label} retries=${retries}`;
}

console.log(createTimer("job", 0));
```
::

- [ ] `"job retries=3"`
- [ ] `"job retries=NaN"`
- [ ] `TypeError`
- [ ] `"job retries=0"`

<details>
<summary>Show Answer</summary>

**Answer:** D — `"job retries=0"`

**Explanation:** **Debug:** default parameters check specifically for `undefined`, not general falsiness. `0` is a legitimate, explicitly passed argument, so it's kept as-is and `3` is never substituted. This is the crucial difference from the common `retries = retries || 3` fallback idiom, which *would* incorrectly replace a passed `0` (or `""`, or `false`) with the fallback — a classic source of bugs when zero is a meaningful value.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What does this log?

::code-wrapper{language="javascript"}
```javascript
let calls = 0;
function nextId() {
  calls += 1;
  return calls;
}

function createUser(name, id = nextId()) {
  return { name, id };
}

createUser("alice", 5);
createUser("bob");
createUser("carol", 9);

console.log(calls);
```
::

- [ ] `1`
- [ ] `3`
- [ ] `0`
- [ ] `2`

<details>
<summary>Show Answer</summary>

**Answer:** A — `1`

**Explanation:** A default parameter expression — including a function call like `nextId()` — only runs when its argument is actually omitted (or explicitly `undefined`). `createUser("alice", 5)` and `createUser("carol", 9)` both pass an explicit `id`, so `nextId()` never executes for them. Only `createUser("bob")` omits `id`, triggering exactly one call to `nextId()`, leaving `calls` at `1`. This side-effect timing is easy to get wrong if you assume the default expression runs on every invocation regardless of whether it's needed.

</details>
::

::question-wrapper{language="javascript"}
### Q14. What does this log?

::code-wrapper{language="javascript"}
```javascript
function addItem(item, cart = []) {
  cart.push(item);
  return cart;
}

console.log(addItem("apple"));
console.log(addItem("banana"));
```
::

- [ ] `["apple"]` then `["apple", "banana"]`
- [ ] `["apple"]` then `["banana"]`
- [ ] `["apple", "banana"]` then `["apple", "banana"]`
- [ ] `TypeError: cart is not extensible`

<details>
<summary>Show Answer</summary>

**Answer:** B — `["apple"]` then `["banana"]`

**Explanation:** **Idiom:** unlike Python's infamous mutable-default-argument footgun (where a default `list` is created once and reused across every call), JavaScript re-evaluates a default parameter expression fresh every time it's needed. Each call that omits `cart` gets its own brand-new empty array — nothing accumulates or leaks between calls. Option A is exactly the Python-style behavior a learner might expect, but it does not happen in JavaScript.

</details>
::

::question-wrapper{language="javascript"}
### Q15. What happens when this runs?

::code-wrapper{language="javascript"}
```javascript
function sumAll() {
  return arguments.filter(n => n > 0).reduce((a, b) => a + b, 0);
}

console.log(sumAll(1, -2, 3));
```
::

- [ ] Logs `4`
- [ ] Logs `NaN`
- [ ] `TypeError: arguments.filter is not a function`
- [ ] Logs `1`

<details>
<summary>Show Answer</summary>

**Answer:** C — `TypeError: arguments.filter is not a function`

**Explanation:** **Debug:** `arguments` is *array-like* — it has a `length` and indexed properties — but it is not an actual `Array` instance, so it doesn't inherit `Array.prototype` methods like `filter` or `reduce`. Calling `.filter` directly on it throws a `TypeError`. To use array methods you'd need to convert it first (`Array.from(arguments)` or `[...arguments]`), or better yet, avoid `arguments` entirely in favor of a rest parameter (see Q17), which is a real array from the start.

</details>
::

::question-wrapper{language="javascript"}
### Q16. What does this log?

::code-wrapper{language="javascript"}
```javascript
function outer() {
  const inner = () => {
    console.log(arguments[0]);
  };
  inner("z");
}

outer("a", "b");
```
::

- [ ] `"z"`
- [ ] `undefined`
- [ ] `ReferenceError: arguments is not defined`
- [ ] `"a"`

<details>
<summary>Show Answer</summary>

**Answer:** D — `"a"`

**Explanation:** **Debug:** arrow functions never get their own `arguments` object. Any `arguments` reference inside an arrow function is resolved lexically, walking up to the nearest enclosing *regular* (non-arrow) function's `arguments` — here, `outer`'s. `outer` was called with `("a", "b")`, so `arguments[0]` is `"a"`; the `"z"` passed directly to `inner` is irrelevant because `inner` never captures its own arguments at all. This is a frequent source of confusion when refactoring a regular function into an arrow function.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What does this log?

::code-wrapper{language="javascript"}
```javascript
function sumAll(...nums) {
  return nums.filter(n => n > 0).reduce((a, b) => a + b, 0);
}

console.log(sumAll(1, -2, 3));
```
::

- [ ] `4`
- [ ] `TypeError: nums.filter is not a function`
- [ ] `NaN`
- [ ] `2`

<details>
<summary>Show Answer</summary>

**Answer:** A — `4`

**Explanation:** **Idiom:** a rest parameter (`...nums`) collects the remaining arguments into a genuine `Array`, so `filter` and `reduce` work directly with no conversion step — `filter(n => n > 0)` keeps `1` and `3`, and `reduce` sums them to `4`. This is exactly why rest parameters are the recommended, more readable replacement for the legacy `arguments` object (contrast with Q15, where the equivalent code using `arguments` throws).

</details>
::

::question-wrapper{language="javascript"}
### Q18. What is `request.length` in this snippet?

::code-wrapper{language="javascript"}
```javascript
function request(url, method = "GET", headers = {}) {
  return method;
}

console.log(request.length);
```
::

- [ ] `3`
- [ ] `1`
- [ ] `2`
- [ ] `0`

<details>
<summary>Show Answer</summary>

**Answer:** B — `1`

**Explanation:** **Debug:** `Function.prototype.length` only counts the parameters that appear *before* the first one with a default value (and rest parameters are never counted either). Here `method` and `headers` both have defaults, so counting stops immediately after `url`, giving a length of `1` — not `3`, even though the function accepts up to three arguments. This surprises people who expect `.length` to reflect the full parameter list; it's really reporting the number of "required-looking" leading parameters.

</details>
::

::question-wrapper{language="javascript"}
### Q19. This entire file is in strict mode. What does it log?

::code-wrapper{language="javascript"}
```javascript
"use strict";

if (true) {
  function greet() {
    return "hi";
  }
}

console.log(typeof greet);
```
::

- [ ] `"function"`
- [ ] `"string"`
- [ ] `"undefined"`
- [ ] it throws a `ReferenceError`

<details>
<summary>Show Answer</summary>

**Answer:** C — `"undefined"`

**Explanation:** **Portability:** in strict mode, the ECMAScript spec requires a function declaration inside a block (like this `if`) to be properly block-scoped, behaving essentially like `let` — it does not exist outside the `if` block. So `typeof greet` outside is `"undefined"` (and `typeof` never throws on an unresolvable identifier, ruling out option D). In legacy non-strict ("sloppy") code, engines instead apply Annex B compatibility semantics, which additionally leak the function's name into the enclosing function/global scope initialized to `undefined` — a long-standing, easy-to-forget inconsistency between strict and sloppy mode that's best avoided by not declaring functions inside blocks at all.

</details>
::

::question-wrapper{language="javascript"}
### Q20. What does this log, and what is the pattern accomplishing?

::code-wrapper{language="javascript"}
```javascript
const cache = {};

(function initCache() {
  const secretSalt = "xyz123";
  cache.hash = str => str.length + secretSalt.length;
})();

console.log(typeof secretSalt);
console.log(cache.hash("abc"));
```
::

- [ ] `"string"` then `9` — the IIFE leaks its inner variables onto `cache` automatically
- [ ] it throws a `ReferenceError`, then nothing logs — IIFEs cannot reach outer variables like `cache`
- [ ] `"function"` then `9` — `secretSalt` is hoisted as a function
- [ ] `"undefined"` then `9` — the IIFE runs immediately and creates a private scope, so `secretSalt` never escapes, while `cache.hash` retains a reference to it

<details>
<summary>Show Answer</summary>

**Answer:** D — `"undefined"` then `9` — the IIFE runs immediately and creates a private scope, so `secretSalt` never escapes, while `cache.hash` retains a reference to it

**Explanation:** **Idiom:** an IIFE (immediately invoked function expression) is defined and called in one step specifically to create an isolated, private scope — a pattern that predates block scoping and is still useful for module-style encapsulation. `secretSalt` lives only inside `initCache`'s scope, so it's never visible outside, making `typeof secretSalt` `"undefined"`. The function assigned to `cache.hash` was created inside that same scope though, so it can still read `secretSalt` when it's called later, producing `3 + 6 = 9`. Option A gets the mechanism backwards — the whole point of an IIFE is to *prevent* leakage, not cause it.

</details>
::
