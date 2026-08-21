# 12 — Closures & Lexical Scope

::question-wrapper{language="javascript"}
### Q1. What does this log?

::code-wrapper{language="javascript"}
```javascript
function makeCounter() {
  let count = 0;
  return function increment() {
    count++;
    return count;
  };
}

const counter = makeCounter();
console.log(counter(), counter(), counter());
```
::

- [ ] 1 2 3
- [ ] 1 1 1
- [ ] undefined undefined undefined
- [ ] NaN NaN NaN

<details>
<summary>Show Answer</summary>

**Answer:** A — 1 2 3

**Explanation:** `increment` forms a **closure** over `count` — it keeps a live reference to that variable binding even after `makeCounter` has returned. Each call mutates the same `count`, so it increments across calls rather than resetting. "1 1 1" is the trap a beginner falls into by assuming `count` re-initializes to `0` on every call, as if it were a local variable reset each time — but it's only initialized once, when `makeCounter()` runs.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does this log?

::code-wrapper{language="javascript"}
```javascript
function outer() {
  const secret = 42;
  return () => secret;
}

const getSecret = outer();
console.log(getSecret());
```
::

- [ ] 42
- [ ] undefined
- [ ] Throws a ReferenceError because `outer` has already returned
- [ ] null

<details>
<summary>Show Answer</summary>

**Answer:** A — 42

**Explanation:** Even though `outer()` has finished executing, the arrow function returned from it keeps `secret` alive via closure — the variable isn't garbage collected while a reachable function still references it. This is exactly what makes closures useful for private state: `secret` is inaccessible from outside except through the returned function.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What does this log?

::code-wrapper{language="javascript"}
```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```
::

- [ ] 3 3 3
- [ ] 0 1 2
- [ ] 0 1 2 3
- [ ] 2 2 2

<details>
<summary>Show Answer</summary>

**Answer:** A — 3 3 3

**Explanation:** `var` is function/global-scoped, not block-scoped, so there is only **one** `i` binding shared by the whole loop and every callback closure. By the time any `setTimeout` callback actually runs (after the synchronous loop has finished), `i` has already reached `3`, and all three callbacks read that same final value. "0 1 2" is what people expect from intuition — it's the correct answer only once `var` is swapped for `let` (see next question).

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does this log?

::code-wrapper{language="javascript"}
```javascript
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```
::

- [ ] 0 1 2
- [ ] 3 3 3
- [ ] 0 0 0
- [ ] 1 2 3

<details>
<summary>Show Answer</summary>

**Answer:** A — 0 1 2

**Explanation:** The `for (let ...)` form is special-cased by the spec: it creates a **fresh binding of `i` for each iteration**, copying the previous iteration's value into the new binding before running the loop body. Each `setTimeout` callback closes over its own iteration's `i`, so they log `0`, `1`, `2` in order — this is the standard fix for the classic `var` loop bug in Q3.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What is logged?

::code-wrapper{language="javascript"}
```javascript
function memoize(fn) {
  const cache = new Map();
  return function (n) {
    if (cache.has(n)) return cache.get(n);
    const result = fn(n);
    cache.set(n, result);
    return result;
  };
}

let calls = 0;
const square = memoize(n => { calls++; return n * n; });

square(4);
square(4);
square(5);
console.log(calls);
```
::

- [ ] 2
- [ ] 3
- [ ] 1
- [ ] 0

<details>
<summary>Show Answer</summary>

**Answer:** A — 2

**Explanation:** The memoized wrapper closes over `cache` so it persists across calls. The first `square(4)` is a miss (computes, increments `calls` to 1, caches it), the second `square(4)` is a **hit** and returns the cached value without calling the original function, and `square(5)` is a new miss (increments `calls` to 2). Expecting `3` assumes memoization isn't working — that's the whole point of the closure-backed cache.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What does this log?

::code-wrapper{language="javascript"}
```javascript
function createBankAccount(balance) {
  return {
    deposit(amount) { balance += amount; return balance; },
    getBalance() { return balance; }
  };
}

const acc = createBankAccount(100);
acc.deposit(50);
console.log(acc.balance);
```
::

- [ ] undefined
- [ ] 150
- [ ] 100
- [ ] Throws a TypeError

<details>
<summary>Show Answer</summary>

**Answer:** A — undefined

**Explanation:** `balance` is never assigned as a property of the returned object — it only exists as a variable in `createBankAccount`'s scope, accessible to `deposit` and `getBalance` through closure. `acc.balance` looks for an own property named `balance` on `acc`, which doesn't exist, so it's `undefined`. This is precisely the **private state pattern**: the real value (now `150` internally) is only reachable through the exposed methods, never by direct property access.

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does this log?

::code-wrapper{language="javascript"}
```javascript
function makeCounter() {
  let c = 0;
  return () => ++c;
}

const a = makeCounter();
const b = makeCounter();

a();
a();
console.log(a(), b());
```
::

- [ ] 3 1
- [ ] 2 1
- [ ] 3 3
- [ ] 1 1

<details>
<summary>Show Answer</summary>

**Answer:** A — 3 1

**Explanation:** Each call to `makeCounter()` creates a **brand-new, independent** `c` binding — `a` and `b` do not share state even though they came from the same factory function. `a` has been invoked three times total by the time it's logged (two discarded calls, then a third inside the `console.log`), reaching `3`; `b` is invoked for the first time inside the same line, giving `1`.

</details>
::

::question-wrapper{language="javascript"}
### Q8. What does this log?

::code-wrapper{language="javascript"}
```javascript
function createLogger(config) {
  return function log(msg) {
    console.log(`[${config.level}] ${msg}`);
  };
}

const config = { level: 'info' };
const logger = createLogger(config);
config.level = 'error';

logger('test');
```
::

- [ ] "[error] test"
- [ ] "[info] test"
- [ ] Throws a TypeError because `config` was reassigned
- [ ] "[undefined] test"

<details>
<summary>Show Answer</summary>

**Answer:** A — "[error] test"

**Explanation:** **Debug:** a closure captures the *variable binding*, not a snapshot of its value at creation time — and for an object, that binding holds a reference. Mutating `config.level` after `createLogger` runs is visible to `log` the next time it reads `config.level`, because both `config` (outer) and the closure's captured reference point to the same object. This surprises people who expect closures to "freeze" values the way default arguments do.

</details>
::

::question-wrapper{language="javascript"}
### Q9. Given modern JS engines (like V8), what typically happens to `bigArray` here?

::code-wrapper{language="javascript"}
```javascript
function heavyClosure() {
  const bigArray = new Array(1_000_000).fill('x');
  return function tiny() {
    return 'hi';
  };
}

const fn = heavyClosure();
```
::

- [ ] `bigArray` is eligible for garbage collection because `tiny` never references it — modern engines only keep alive the specific bindings an escaping closure actually uses
- [ ] `bigArray` is retained in memory forever as long as `fn` exists, regardless of whether `tiny` uses it
- [ ] `bigArray` gets copied into `tiny`'s closure, doubling memory usage
- [ ] `heavyClosure` throws an error because `bigArray` is unused

<details>
<summary>Show Answer</summary>

**Answer:** A — `bigArray` is eligible for garbage collection because `tiny` never references it — modern engines only keep alive the specific bindings an escaping closure actually uses

**Explanation:** **Performance:** it's a common myth that a surviving closure keeps the *entire* enclosing scope alive. Modern engines perform static analysis and only retain the variables an inner function actually touches; since `tiny` never reads `bigArray`, V8 can typically collect it once `heavyClosure` returns. That said, this is an engine optimization, not a spec guarantee — the safe, portable practice is still to avoid holding unnecessary large references in scope alongside a closure you intend to keep around, since older or simpler engines may retain the whole variable environment.

</details>
::

::question-wrapper{language="javascript"}
### Q10. What does this log?

::code-wrapper{language="javascript"}
```javascript
for (var i = 0; i < 3; i++) {
  (function (j) {
    setTimeout(() => console.log(j), 0);
  })(i);
}
```
::

- [ ] 0 1 2
- [ ] 3 3 3
- [ ] undefined undefined undefined
- [ ] 0 0 0

<details>
<summary>Show Answer</summary>

**Answer:** A — 0 1 2

**Explanation:** This is the classic pre-`let` fix for the `var` loop bug: the **IIFE** (immediately invoked function expression) creates a new function scope on every iteration, and `i`'s current value is passed in as the argument `j`. Each `setTimeout` callback then closes over its own private `j`, not the shared loop `i`, reproducing the same effect `let` gives you automatically.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What does this log?

::code-wrapper{language="javascript"}
```javascript
const fns = [];
for (let i = 0; i < 3; i++) {
  fns.push(() => i);
}
console.log(fns.map(fn => fn()));
```
::

- [ ] [0, 1, 2]
- [ ] [3, 3, 3]
- [ ] [2, 2, 2]
- [ ] [undefined, undefined, undefined]

<details>
<summary>Show Answer</summary>

**Answer:** A — [0, 1, 2]

**Explanation:** This confirms the per-iteration binding of `let` has nothing to do with timing or asynchrony — even called synchronously (no `setTimeout` involved), each pushed arrow function closes over its own iteration's `i`. It's a common misconception that `let`'s loop fix is specifically an "async trick"; it's really about scoping, and this example proves that by removing the timer entirely.

</details>
::

::question-wrapper{language="javascript"}
### Q12. What happens when `show()` is called?

::code-wrapper{language="javascript"}
```javascript
let value = 'outer';

function show() {
  console.log(value);
  let value = 'inner';
}

show();
```
::

- [ ] Throws `ReferenceError: Cannot access 'value' before initialization`
- [ ] Logs "outer"
- [ ] Logs "inner"
- [ ] Logs undefined

<details>
<summary>Show Answer</summary>

**Answer:** A — Throws `ReferenceError: Cannot access 'value' before initialization`

**Explanation:** **Debug:** the inner `let value` declaration is hoisted to the top of `show`'s function scope (though not initialized), so it **shadows** the outer `value` for the entire function body — including the `console.log(value)` line that appears before the declaration. That reference falls into the **temporal dead zone** and throws, rather than falling back to the outer variable the way people expect closures/scoping to "skip over" an inner declaration that hasn't run yet.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What does this log?

::code-wrapper{language="javascript"}
```javascript
function createValidator(min) {
  return function validate(value) {
    return value >= min;
  };
}

let minValue = 10;
const validate = createValidator(minValue);
minValue = 100;

console.log(validate(50));
```
::

- [ ] true
- [ ] false
- [ ] Throws a ReferenceError
- [ ] undefined

<details>
<summary>Show Answer</summary>

**Answer:** A — true

**Explanation:** **Debug:** unlike Q8's object example, `min` here is a **parameter**, and primitives are passed **by value** — `createValidator(minValue)` copies the number `10` into the parameter `min` at call time. Reassigning the outer `minValue` variable afterward has no effect on that already-copied parameter, so `validate`'s closure still sees `min = 10`, and `50 >= 10` is `true`. This is the mirror image of Q8: closures capture bindings by reference, but a primitive value copied into a parameter is disconnected from the variable it came from.

</details>
::

::question-wrapper{language="javascript"}
### Q14. What does this log?

::code-wrapper{language="javascript"}
```javascript
function createButtons(labels) {
  return labels.map(label => () => console.log(`Clicked ${label}`));
}

const labels = ['A', 'B'];
const handlers = createButtons(labels);
labels.push('C');

handlers.forEach(fn => fn());
```
::

- [ ] "Clicked A" then "Clicked B" (2 lines total)
- [ ] "Clicked A", "Clicked B", "Clicked C" (3 lines total)
- [ ] "Clicked C" three times
- [ ] Throws a TypeError because `labels` was mutated after `.map`

<details>
<summary>Show Answer</summary>

**Answer:** A — "Clicked A" then "Clicked B" (2 lines total)

**Explanation:** `.map` runs synchronously and finishes building `handlers` (with exactly 2 functions) before `labels.push('C')` ever executes — pushing to `labels` afterward doesn't retroactively add a third handler. Additionally, each arrow function closes over its own `label` **parameter**, a primitive string copied per iteration by `.map`, not a shared reference into the `labels` array — so even mutating `labels` further wouldn't change what the existing handlers log.

</details>
::

::question-wrapper{language="javascript"}
### Q15. Which approach actually prevents external code from directly overwriting the account balance?

::code-wrapper{language="javascript"}
```javascript
// Option A
class BankAccount {
  constructor(balance) { this.balance = balance; }
}

// Option B
function createAccount(balance) {
  return {
    deposit(amount) { balance += amount; return balance; },
    getBalance() { return balance; }
  };
}
```
::

- [ ] Option B — `balance` lives only in closure scope, so there's no property to overwrite from outside
- [ ] Option A — classes always enforce encapsulation
- [ ] Both are equally safe against `account.balance = 999999`
- [ ] Neither prevents it; JavaScript has no way to hide state

<details>
<summary>Show Answer</summary>

**Answer:** A — Option B — `balance` lives only in closure scope, so there's no property to overwrite from outside

**Explanation:** **Idiom:** in Option A, `this.balance` is a plain public property — anyone holding the instance can do `account.balance = 999999` directly, no method required. In Option B, `balance` never becomes a property of the returned object; it only exists as a variable the closures can reach, so there is no `account.balance` to assign to at all. (A class using a true private field, `#balance`, would achieve the same guarantee — the key distinction is private-by-closure or `#field` versus a plain public property, not "class vs. factory function" per se.)

</details>
::

::question-wrapper{language="javascript"}
### Q16. This click handler is attached to a large widget that later gets removed from the DOM. What's the best practice to prevent a memory leak?

::code-wrapper{language="javascript"}
```javascript
function setupWidget(el, largeCachedData) {
  el.addEventListener('click', function handler() {
    console.log(largeCachedData.summary);
  });
}
```
::

- [ ] Call `el.removeEventListener('click', handler)` (or use an `AbortController`/`signal`) before discarding `el`, so the closure holding `largeCachedData` can be released
- [ ] Nothing needs to change — removing `el` from the DOM automatically releases its listeners and closures
- [ ] Replace the named function with an arrow function to avoid the leak
- [ ] Avoid closures entirely by making `largeCachedData` a global variable

<details>
<summary>Show Answer</summary>

**Answer:** A — Call `el.removeEventListener('click', handler)` (or use an `AbortController`/`signal`) before discarding `el`, so the closure holding `largeCachedData` can be released

**Explanation:** **Debug:** as long as the listener reference exists, the `handler` closure keeps `largeCachedData` alive — and if something else (a framework, a detached-node cache, a reference cycle) keeps the element itself alive too, simply removing `el` from the DOM tree doesn't guarantee garbage collection. Explicitly removing the listener (or attaching it with an `AbortController`'s `signal` and calling `abort()`) breaks the reference chain so both the element and the closed-over data can be collected. Swapping to an arrow function changes nothing about what's captured.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What's the pitfall in this memoization helper?

::code-wrapper{language="javascript"}
```javascript
function memoize(fn) {
  const cache = {};
  return function (obj) {
    const key = JSON.stringify(obj);
    if (key in cache) return cache[key];
    return (cache[key] = fn(obj));
  };
}

const process = memoize(data => data.a + data.b);
process({ a: 1, b: 2 });
process({ b: 2, a: 1 });
```
::

- [ ] `JSON.stringify` is key-order-sensitive, so these logically-equal objects produce different cache keys (an unwanted cache miss), and the cache also grows unbounded with no eviction
- [ ] This code throws a TypeError on the second call because the key already exists
- [ ] `JSON.stringify` automatically sorts object keys, so this memoization works correctly and efficiently
- [ ] The cache is shared across all calls to `memoize`, causing collisions between unrelated functions

<details>
<summary>Show Answer</summary>

**Answer:** A — `JSON.stringify` is key-order-sensitive, so these logically-equal objects produce different cache keys (an unwanted cache miss), and the cache also grows unbounded with no eviction

**Explanation:** **Idiom:** `JSON.stringify({a:1,b:2})` and `JSON.stringify({b:2,a:1})` produce different strings even though the objects are equivalent, so the second call recomputes instead of hitting the cache — a subtle correctness/perf bug, not a crash. Separately, nothing ever removes entries from `cache`, so a memoized function called with many distinct inputs grows the closure's cache indefinitely. Better practice: normalize keys (e.g. sort object keys before stringifying, or use a stable hash) and cap growth with an LRU eviction strategy or a `WeakMap` when keying by object identity is acceptable.

</details>
::

::question-wrapper{language="javascript"}
### Q18. What does this log?

::code-wrapper{language="javascript"}
```javascript
const curry = fn => (...args) =>
  args.length >= fn.length
    ? fn(...args)
    : (...more) => curry(fn)(...args, ...more);

const add3 = (a, b, c) => a + b + c;
const curried = curry(add3);

console.log(curried(1)(2)(3), curried(1, 2)(3), curried(1, 2, 3));
```
::

- [ ] 6 6 6
- [ ] Throws a TypeError because functions can't be called with zero, then more, arguments
- [ ] NaN NaN 6
- [ ] 6 NaN NaN

<details>
<summary>Show Answer</summary>

**Answer:** A — 6 6 6

**Explanation:** **Idiom:** each partial call returns a new closure that remembers (`args`) the arguments accumulated so far, comparing their count against `fn.length` (the declared arity of `add3`, which is `3`). Once enough arguments have accumulated across however many calls it took, the closure invokes the original function. All three calling styles — one at a time, in pairs, or all at once — reach the same total of three arguments and the same result, `6`. This accumulate-via-closure technique is the standard way to implement generic currying in JS.

</details>
::

::question-wrapper{language="javascript"}
### Q19. For a list of 10,000 rows that re-renders frequently, which is the more scalable click-handling strategy?

::code-wrapper{language="javascript"}
```javascript
// Approach A — one closure + one listener per row
items.forEach(item => {
  const li = document.createElement('li');
  li.addEventListener('click', () => selectItem(item.id));
  list.appendChild(li);
});

// Approach B — a single listener on the parent
list.addEventListener('click', e => {
  const li = e.target.closest('li');
  if (li) selectItem(li.dataset.id);
});
```
::

- [ ] Approach B — event delegation avoids allocating one closure and one listener per row, which matters a lot at scale
- [ ] Approach A — more closures means more granular control and better performance
- [ ] They perform identically; closures have no measurable cost
- [ ] Approach A, but only if each closure is wrapped in `.bind()` instead of an arrow function

<details>
<summary>Show Answer</summary>

**Answer:** A — Approach B — event delegation avoids allocating one closure and one listener per row, which matters a lot at scale

**Explanation:** **Performance:** Approach A allocates 10,000 separate closures and registers 10,000 separate listeners, all of which need to be torn down and recreated on every re-render — real memory and GC pressure at that scale. Approach B attaches a single listener once, using event bubbling and `closest()`/`dataset` to figure out which row was clicked, so re-rendering the list doesn't require touching listeners at all. This is the standard **event delegation** pattern for large or frequently-changing lists.

</details>
::

::question-wrapper{language="javascript"}
### Q20. Why must `timeoutId` be declared in the outer function here, rather than inside the returned function?

::code-wrapper{language="javascript"}
```javascript
function debounce(fn, delay) {
  let timeoutId;
  return function (...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}
```
::

- [ ] So its value persists across multiple calls to the debounced function, letting each new call cancel the previous pending timer
- [ ] It's purely stylistic — declaring it inside the returned function would behave identically
- [ ] To avoid a naming collision with the `delay` parameter
- [ ] Because `let` inside the returned function would throw a redeclaration error on the second call

<details>
<summary>Show Answer</summary>

**Answer:** A — So its value persists across multiple calls to the debounced function, letting each new call cancel the previous pending timer

**Explanation:** **Idiom:** debouncing fundamentally needs state (the id of the currently-pending timer) that outlives any single invocation and is shared across every call to the returned function — exactly what a variable in the enclosing closure scope provides. If `timeoutId` were declared inside the returned function instead, it would be reinitialized to `undefined` on every call, `clearTimeout(undefined)` would be a no-op, and the previous timer would never actually get cancelled — defeating the entire point of debouncing.

</details>
::
