# 23 — Functional Programming

::question-wrapper{language="javascript"}
### Q1. Which function below is a **pure function**?

::code-wrapper{language="javascript"}
```javascript
let taxRate = 0.08;

function priceWithTaxA(price) {
  return price + price * taxRate;
}

function priceWithTaxB(price, rate) {
  return price + price * rate;
}
```
::

- [ ] `priceWithTaxA`, because it performs a calculation
- [ ] `priceWithTaxB`, because its output depends only on its arguments and it causes no observable side effects
- [ ] Both are pure, since neither mutates its input
- [ ] Neither is pure, since both perform arithmetic

<details>
<summary>Show Answer</summary>

**Answer:** B — `priceWithTaxB`, because its output depends only on its arguments and it causes no observable side effects

**Explanation:** A pure function's return value must depend *only* on its input parameters, with no reads of external mutable state and no side effects. `priceWithTaxA` reads the outer `taxRate` variable — if that variable changes between calls, the same `price` argument produces a different result, which violates purity even though the function never mutates anything itself. `priceWithTaxB` closes over nothing external; every input it needs is passed explicitly, so it's deterministic and pure. This distinction matters in practice: `priceWithTaxA` is harder to test (you must control global state) and harder to reason about in concurrent/async code.

</details>
::

::question-wrapper{language="javascript"}
### Q2. Why is the following function considered impure, beyond just "it uses `push`"?

::code-wrapper{language="javascript"}
```javascript
function addItem(cart, item) {
  cart.push(item);
  return cart;
}
```
::

- [ ] It's not impure — returning a value makes any function pure
- [ ] It mutates the `cart` array argument in place, producing a side effect visible to any other code holding a reference to that same array
- [ ] It's impure only because it takes two arguments instead of one
- [ ] It's impure because `push` is asynchronous

<details>
<summary>Show Answer</summary>

**Answer:** B — It mutates the `cart` array argument in place, producing a side effect visible to any other code holding a reference to that same array

**Explanation:** Purity isn't just about avoiding *global* state — mutating an argument that was passed in by reference (arrays and objects are reference types in JS) is itself a side effect, because any other part of the program holding a reference to that same `cart` array sees the change too, even though this function never touched a variable outside its own scope. Option A is the common misconception that "has a return value" automatically implies purity; a function can return something *and* still mutate its inputs, which is precisely the trap here. The pure version would return a new array: `return [...cart, item]`.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What is the idiomatic, immutable way to add a property to an object without mutating the original?

::code-wrapper{language="javascript"}
```javascript
const user = { name: "Priya", role: "admin" };
```
::

- [ ] `user.active = true;`
- [ ] `Object.assign(user, { active: true });`
- [ ] `const updated = { ...user, active: true };`
- [ ] `user["active"] = true; return user;`

<details>
<summary>Show Answer</summary>

**Answer:** C — `const updated = { ...user, active: true };`

**Explanation:** The spread operator creates a *new* object, copying all of `user`'s own enumerable properties into it and then overwriting/adding `active`, leaving the original `user` completely untouched. Options A and D directly mutate `user` in place. Option B is the sneaky trap: `Object.assign(target, source)` mutates `target` (the first argument) — passing `user` as the target still mutates it, even though `Object.assign` is often reached for as an "immutable-looking" utility; producing a new object with `Object.assign` requires `Object.assign({}, user, { active: true })`, with an empty object literal as the target.

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does "currying" a function mean?

::code-wrapper{language="javascript"}
```javascript
function add(a, b, c) {
  return a + b + c;
}

function curriedAdd(a) {
  return (b) => (c) => a + b + c;
}
```
::

- [ ] Transforming a function so it can accept its arguments one at a time, each call returning a new function until all arguments are supplied
- [ ] Making a function run faster by caching its results
- [ ] Converting a function to accept an array of arguments instead of separate parameters
- [ ] Binding a function's `this` value permanently

<details>
<summary>Show Answer</summary>

**Answer:** A — Transforming a function so it can accept its arguments one at a time, each call returning a new function until all arguments are supplied

**Explanation:** `curriedAdd(1)` returns a function waiting for `b`, and calling *that* with `2` returns a function waiting for `c` — only `curriedAdd(1)(2)(3)` produces the final sum `6`. This enables partial application: `curriedAdd(1)` can be saved and reused as a specialized function that always adds `1` to whatever comes next. Option B describes memoization, a related but distinct technique (see Q17). Option C describes a different pattern sometimes called "argument spreading" or using a single options object, not currying. Option D describes `Function.prototype.bind`'s `this`-binding behavior, unrelated to currying's argument-splitting purpose.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does function composition (`compose`) typically achieve?

::code-wrapper{language="javascript"}
```javascript
const compose = (f, g) => (x) => f(g(x));

const shout = (s) => s.toUpperCase() + "!";
const exclaim = (s) => s + "!!!";

const shoutExclaim = compose(exclaim, shout);
shoutExclaim("hello");
```
::

- [ ] `"HELLO!!!!!"` — `g` (`shout`) runs first on the input, then `f` (`exclaim`) runs on `g`'s result
- [ ] `"hello!!!"` then uppercased separately
- [ ] It throws a `TypeError` because `compose` only accepts one function
- [ ] `f` (`exclaim`) runs first, then `g` (`shout`) runs on its result

<details>
<summary>Show Answer</summary>

**Answer:** A — `"HELLO!!!!!"` — `g` (`shout`) runs first on the input, then `f` (`exclaim`) runs on `g`'s result

**Explanation:** By convention, `compose(f, g)(x)` evaluates as `f(g(x))` — the *rightmost* function runs first, and results flow right-to-left through the chain, mirroring mathematical function composition notation. Here `shout("hello")` produces `"HELLO!"`, and `exclaim("HELLO!")` produces `"HELLO!!!!!"`. This right-to-left order is a frequent point of confusion (option D reverses it), which is exactly why many functional libraries also offer a `pipe` helper that runs left-to-right instead, matching the order functions are visually listed — see Q11 for the mixup this causes.

</details>
::

::question-wrapper{language="javascript"}
### Q6. Which snippet demonstrates "point-free" (tacit) style?

::code-wrapper{language="javascript"}
```javascript
const names = ["ravi", "priya", "sam"];

const a = names.map((name) => name.toUpperCase());
const b = names.map(String.prototype.toUpperCase.call.bind(String.prototype.toUpperCase));
const c = names.map((name) => name.trim().toUpperCase());
```
::

- [ ] `a`, because it uses an arrow function
- [ ] `b`, because the mapping function is referenced directly without explicitly naming or wrapping the data it operates on (no intermediate named argument like `name`)
- [ ] `c`, because it chains two methods
- [ ] None of these are point-free

<details>
<summary>Show Answer</summary>

**Answer:** B — `b`, because the mapping function is referenced directly without explicitly naming or wrapping the data it operates on (no intermediate named argument like `name`)

**Explanation:** Point-free style means defining a function without mentioning the arguments ("points") it operates on — you compose existing functions directly rather than writing `(x) => f(x)`. `a` and `c` both explicitly name `name` as an intermediate variable, which is the opposite of point-free. `b` (admittedly contrived and hard to read here, which illustrates a real trade-off) passes a function reference derived purely from existing methods, with no named parameter appearing in the mapping logic itself. In practice, simpler point-free examples look like `names.map(s => s.toUpperCase())` → `names.map(String.prototype.toUpperCase.call, String.prototype)` style tricks, or more commonly just passing a named utility function directly: `names.map(toUpperCase)`.

</details>
::

::question-wrapper{language="javascript"}
### Q7. Why is shared mutable state considered risky in a codebase, especially with async code?

::code-wrapper{language="javascript"}
```javascript
let requestCount = 0;

async function handleRequest(req) {
  requestCount++;
  const result = await process(req);
  requestCount--;
  return result;
}
```
::

- [ ] It isn't risky — `let` variables are always safe to share
- [ ] Multiple concurrent calls all read and write the same `requestCount` variable, so their increments/decrements interleave in ways that are hard to trace, and any bug in one call path can corrupt state visible to every other call
- [ ] `requestCount` will overflow after 2^53 requests, causing incorrect counts
- [ ] `async` functions cannot access variables declared outside their own body

<details>
<summary>Show Answer</summary>

**Answer:** B — Multiple concurrent calls all read and write the same `requestCount` variable, so their increments/decrements interleave in ways that are hard to trace, and any bug in one call path can corrupt state visible to every other call

**Explanation:** Even though JavaScript is single-threaded and each individual `requestCount++`/`requestCount--` is atomic, the *sequence* of operations across many concurrent `handleRequest` calls (each suspended and resumed at different `await` points) means the shared counter's value at any given moment depends on unpredictable interleaving — an early return, a thrown error skipping the decrement, or a bug in one request handler can leave `requestCount` permanently wrong for the entire process, affecting every other concurrent request. Isolating state per-call (e.g., not sharing a mutable counter at all, or using request-scoped state) avoids this entire category of bug, which is the core motivation behind functional programming's emphasis on avoiding shared mutable state.

</details>
::

::question-wrapper{language="javascript"}
### Q8. `Object.freeze` is often used for immutability. What's the gotcha with the following code?

::code-wrapper{language="javascript"}
```javascript
const config = Object.freeze({
  api: { retries: 3, timeout: 5000 },
});

config.api.retries = 10;
console.log(config.api.retries);
```
::

- [ ] It throws a `TypeError` in strict mode because `config` is frozen
- [ ] It logs `10` — `Object.freeze` is shallow, so nested objects (`config.api`) remain fully mutable
- [ ] It logs `3` — the mutation silently fails because `Object.freeze` is deep by default
- [ ] `Object.freeze` only works on arrays, not plain objects

<details>
<summary>Show Answer</summary>

**Answer:** B — It logs `10` — `Object.freeze` is shallow, so nested objects (`config.api`) remain fully mutable

**Explanation:** **Safety:** `Object.freeze` only locks the *direct* properties of the object it's called on — it prevents reassigning `config.api` itself to a different object, but the nested `api` object was never frozen, so its own properties remain fully writable. This is one of the most common `Object.freeze` gotchas: developers assume "frozen" means "deeply immutable," write `config.api.retries = 10`, and are surprised it silently succeeds (in non-strict contexts) or throws only for the *direct* level. Achieving true deep immutability requires recursively freezing every nested object, or using a library/utility that does so.

</details>
::

::question-wrapper{language="javascript"}
### Q9. What is the surprising problem with using `Array.prototype.sort` inside code that's meant to follow functional, immutable patterns?

::code-wrapper{language="javascript"}
```javascript
function topScores(scores) {
  return scores.sort((a, b) => b - a).slice(0, 3);
}

const original = [5, 2, 9, 1, 7];
const top3 = topScores(original);
console.log(original);
```
::

- [ ] `sort` returns a new array, so `original` is unaffected — this code is already pure
- [ ] `sort` mutates the array in place and also returns it, so `original` itself ends up reordered as a side effect, even though the function "looks" like it just derives a new value
- [ ] `sort` throws an error when used with a custom comparator
- [ ] `slice` mutates `original` by removing the first three elements

<details>
<summary>Show Answer</summary>

**Answer:** B — `sort` mutates the array in place and also returns it, so `original` itself ends up reordered as a side effect, even though the function "looks" like it just derives a new value

**Explanation:** `Array.prototype.sort` (and `reverse`, `splice`, `fill`, `copyWithin`) sorts the array *in place* and returns the same reference — it does not create a new array. Chaining `.slice()` afterward returns a fresh array for `top3`, which masks the fact that `original` was already silently reordered by the time `sort` ran. This is a classic trap for functional-style code, since `sort` sits right next to genuinely non-mutating array methods like `map`/`filter`/`slice` and is easy to assume behaves the same way. **Idiom:** the fix is to copy first — `[...scores].sort(...)` or `scores.toSorted(...)` (the newer, non-mutating ES2023 method) — before sorting.

</details>
::

::question-wrapper{language="javascript"}
### Q10. A curried function is built using `Function.prototype.length` to determine when enough arguments have been collected. What's the pitfall with this specific implementation?

::code-wrapper{language="javascript"}
```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn(...args);
    return (...more) => curried(...args, ...more);
  };
}

function sum(a, b, ...rest) {
  return a + b + rest.reduce((x, y) => x + y, 0);
}

const curriedSum = curry(sum);
```
::

- [ ] It works perfectly for any function, including variadic ones like `sum`
- [ ] `fn.length` does not count rest parameters (or parameters after the first default-valued one), so `sum.length` is `2` — `curriedSum` will call `sum` as soon as 2 arguments arrive, silently ignoring the ability to curry in any of the `rest` values
- [ ] `curry` throws a `TypeError` immediately because `sum` uses rest parameters
- [ ] `fn.length` always equals the number of arguments actually passed at call time, so this is safe

<details>
<summary>Show Answer</summary>

**Answer:** B — `fn.length` does not count rest parameters (or parameters after the first default-valued one), so `sum.length` is `2` — `curriedSum` will call `sum` as soon as 2 arguments arrive, silently ignoring the ability to curry in any of the rest values

**Explanation:** `Function.prototype.length` reports only the count of parameters *before* the first default-valued or rest parameter — it's a static count of the function's declared signature, not a runtime count of anything. For `sum(a, b, ...rest)`, `sum.length` is `2`, so a `length`-based `curry` implementation will invoke `sum(a, b)` the moment two arguments are collected, never giving the caller a chance to curry in additional values that `rest` was designed to accept. This is a real, easy-to-miss limitation of naive curry implementations, and is why production-grade curry utilities usually require an explicit arity argument (`curry(fn, arity)`) instead of trusting `fn.length` for any function that isn't a plain fixed-arity function.

</details>
::

::question-wrapper{language="javascript"}
### Q11. A developer mixes up `compose` and `pipe` and gets an unexpected result. Given:

::code-wrapper{language="javascript"}
```javascript
const compose = (...fns) => (x) => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe = (...fns) => (x) => fns.reduce((acc, fn) => fn(acc), x);

const double = (n) => n * 2;
const addOne = (n) => n + 1;

const a = compose(double, addOne)(5);
const b = pipe(double, addOne)(5);
```
::

What are `a` and `b`?

- [ ] `a = 11`, `b = 11` — order doesn't matter for these particular functions
- [ ] `a = 12`, `b = 11` — `compose` applies `addOne` first then `double` (right-to-left); `pipe` applies `double` first then `addOne` (left-to-right)
- [ ] `a = 11`, `b = 12` — the reverse of the above
- [ ] Both throw an error because `reduceRight` cannot be used with functions

<details>
<summary>Show Answer</summary>

**Answer:** B — `a = 12`, `b = 11` — `compose` applies `addOne` first then `double` (right-to-left); `pipe` applies `double` first then `addOne` (left-to-right)

**Explanation:** `compose(double, addOne)` runs right-to-left (per Q5): `addOne(5) = 6`, then `double(6) = 12`, giving `a = 12`. `pipe(double, addOne)` runs left-to-right: `double(5) = 10`, then `addOne(10) = 11`, giving `b = 11`. This mismatch — same function list, same input, different results purely because of which combinator was used — is exactly the gotcha that makes `compose`/`pipe` mixups a real production bug source: swapping one for the other with the same argument order silently changes execution order rather than throwing an error, so the bug can slip past casual testing if the functions happen to be commutative for some inputs but not others.

</details>
::

::question-wrapper{language="javascript"}
### Q12. Is the following function pure?

::code-wrapper{language="javascript"}
```javascript
function createSession(userId) {
  return { userId, createdAt: Date.now(), token: Math.random().toString(36) };
}
```
::

- [ ] Yes — it doesn't read or mutate any external variable
- [ ] No — even though it never touches external variables, `Date.now()` and `Math.random()` make its output non-deterministic: the same `userId` input produces a different result on every call
- [ ] Yes — purity only requires the absence of argument mutation, which this function satisfies
- [ ] No — it's impure because it returns an object instead of a primitive

<details>
<summary>Show Answer</summary>

**Answer:** B — No — even though it never touches external variables, `Date.now()` and `Math.random()` make its output non-deterministic: the same `userId` input produces a different result on every call

**Explanation:** Purity requires that calling the function with the same arguments *always* produces the same result — determinism is just as essential as "no side effects." `Date.now()` and `Math.random()` both read genuinely external, constantly-changing state (the system clock, the engine's PRNG state), so even though this function never mutates anything or reads a module-level variable, it's still impure by the "same input → same output" definition. This is a subtle gotcha: developers often equate purity purely with "doesn't mutate things" (option C), missing the determinism requirement entirely. Functions needing timestamps/randomness for testability are usually made pure by accepting them as injected parameters instead: `createSession(userId, now, randomToken)`.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What's the gotcha in this code, given that `const` is often (incorrectly) equated with "immutable"?

::code-wrapper{language="javascript"}
```javascript
const cart = [];
cart.push("item-1");
cart.push("item-2");
console.log(cart);
```
::

- [ ] This throws a `TypeError` because `cart` is declared `const`
- [ ] This works fine and logs `["item-1", "item-2"]` — `const` only prevents *reassigning* the `cart` binding to a different value, it does nothing to prevent mutating the object/array that binding currently points to
- [ ] `const` arrays are automatically frozen, so `push` is a no-op
- [ ] This is a syntax error since `const` variables cannot call methods

<details>
<summary>Show Answer</summary>

**Answer:** B — This works fine and logs `["item-1", "item-2"]` — `const` only prevents reassigning the `cart` binding to a different value, it does nothing to prevent mutating the object/array that binding currently points to

**Explanation:** `const` creates an immutable *binding* (you can't do `cart = []` again later), but it says nothing about the mutability of the value the binding refers to — arrays and objects remain fully mutable through their own methods (`push`, `pop`, direct property assignment) regardless of how the variable holding them was declared. This is a persistent beginner misconception (`const` = "constant value") that leads to real bugs when someone assumes a `const`-declared array or object is safe from mutation elsewhere in the code. True immutability requires `Object.freeze` (with the shallow caveat from Q8) or disciplined use of non-mutating operations like spread.

</details>
::

::question-wrapper{language="javascript"}
### Q14. What bug does this counter factory have, and why?

::code-wrapper{language="javascript"}
```javascript
function makeCounter() {
  let count = 0;
  return {
    increment: () => count++,
    reset: () => (count = 0),
  };
}

const counterA = makeCounter();
const counterB = makeCounter();
counterA.increment();
counterA.increment();
console.log(counterB.increment());
```
::

- [ ] `counterB.increment()` logs `2`, since both counters share the same closed-over `count` variable
- [ ] `counterB.increment()` logs `0`, because each call to `makeCounter()` creates a fresh closure with its own independent `count` variable — there's no bug here, this is correctly isolated state
- [ ] This throws a `ReferenceError` because `count` is not accessible outside `makeCounter`
- [ ] `counterA` and `counterB` are the same object reference, so both counters are actually identical

<details>
<summary>Show Answer</summary>

**Answer:** B — `counterB.increment()` logs `0`, because each call to `makeCounter()` creates a fresh closure with its own independent `count` variable — there's no bug here, this is correctly isolated state

**Explanation:** This question flips the expected gotcha: many developers assume closures created from the same factory function share state (confusing this with the module-scoped shared-state problem from Q7), but each *invocation* of `makeCounter()` creates a brand-new `count` variable and a brand-new pair of closures over it — `counterA` and `counterB` are fully independent, so incrementing one has zero effect on the other. This pattern (closures for private, per-instance state) is a deliberately safe alternative to the shared-mutable-module-variable trap from Q7, precisely because each call gets its own isolated scope rather than reading from one shared location.

</details>
::

::question-wrapper{language="javascript"}
### Q15. Which style is more idiomatic functional JavaScript for transforming an array of order objects into a total?

::code-wrapper{language="javascript"}
```javascript
const orders = [{ amount: 20 }, { amount: 35 }, { amount: 10 }];

let totalA = 0;
for (let i = 0; i < orders.length; i++) {
  totalA += orders[i].amount;
}

const totalB = orders.reduce((sum, order) => sum + order.amount, 0);
```
::

- [ ] `totalA`'s imperative loop, because it's more performant in all cases
- [ ] `totalB`'s `reduce`, because it declares *what* is being computed rather than *how* to iterate, avoids a mutable accumulator variable in the surrounding scope, and composes naturally with `map`/`filter` chains
- [ ] Both are equally idiomatic; functional style has no preference here
- [ ] Neither — the idiomatic approach is `orders.map(o => o.amount).sum()`

<details>
<summary>Show Answer</summary>

**Answer:** B — `totalB`'s `reduce`, because it declares what is being computed rather than how to iterate, avoids a mutable accumulator variable in the surrounding scope, and composes naturally with `map`/`filter` chains

**Explanation:** **Idiom:** `reduce` (along with `map`/`filter`) expresses the transformation declaratively and keeps the accumulator scoped entirely inside the call rather than as a mutable `let` variable in the enclosing function, which reduces the surface area for bugs (accidental reuse, forgetting to reset, off-by-one loop errors) and reads closer to "sum the amounts" than "manage an index and a running total." Option D is a tempting-looking but invalid API — plain arrays have no built-in `.sum()` method in JavaScript. Performance (option A) is not the deciding factor here; a simple `reduce` over a modest array is not meaningfully slower than a `for` loop in practice, and clarity/composability is the actual reason functional style is favored for this kind of transformation.

</details>
::

::question-wrapper{language="javascript"}
### Q16. In a Redux-style state management pattern, why is the following reducer considered buggy?

::code-wrapper{language="javascript"}
```javascript
function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM":
      state.items.push(action.item);
      return state;
    default:
      return state;
  }
}
```
::

- [ ] `switch` statements aren't allowed in reducers
- [ ] It mutates `state.items` directly instead of returning a new state object/array, which breaks reference-equality checks that UI frameworks rely on to detect changes and re-render
- [ ] `action.item` must be destructured before use
- [ ] Reducers cannot return the same `state` reference under any circumstance, including the `default` case

<details>
<summary>Show Answer</summary>

**Answer:** B — It mutates `state.items` directly instead of returning a new state object/array, which breaks reference-equality checks that UI frameworks rely on to detect changes and re-render

**Explanation:** **Idiom:** state management libraries typically detect changes with a cheap `oldState !== newState` reference comparison rather than a deep-equality check, for performance reasons. Mutating `state.items` in place means `state` (the object reference) never actually changes, so that comparison reports "nothing changed" even though the data did — this manifests as a UI that silently fails to re-render after an action that logically should have updated it. The fix follows the immutable-update pattern from Q3: `return { ...state, items: [...state.items, action.item] }`, creating new references at every level of the update path. Option D is a false generalization — returning the *same* reference for `default` (an unhandled/no-op action) is correct and expected, since nothing changed there.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What performance technique commonly pairs with pure functions, and why does purity make it safe?

::code-wrapper{language="javascript"}
```javascript
function memoize(fn) {
  const cache = new Map();
  return (arg) => {
    if (cache.has(arg)) return cache.get(arg);
    const result = fn(arg);
    cache.set(arg, result);
    return result;
  };
}
```
::

- [ ] Debouncing — pure functions are always slow, so debouncing their calls is necessary
- [ ] Memoization — because a pure function always returns the same output for the same input, caching results by argument is safe and never returns a stale/incorrect value
- [ ] Throttling — pure functions must be rate-limited to avoid excessive CPU use
- [ ] Currying — memoization only works on curried functions

<details>
<summary>Show Answer</summary>

**Answer:** B — Memoization — because a pure function always returns the same output for the same input, caching results by argument is safe and never returns a stale/incorrect value

**Explanation:** **Performance:** memoization trades memory for time by caching a function's output keyed by its input, but this is only *correct* to do when the function is pure — an impure function (like the `createSession` example from Q12, which depends on the current time/randomness) would return a stale, wrong cached value for a later call with the same argument, since its "true" output legitimately changes between calls. This is why memoization utilities are typically documented as safe only for pure functions, and why introducing impurity into a previously-pure function that's already memoized somewhere is a subtle way to introduce a correctness bug.

</details>
::

::question-wrapper{language="javascript"}
### Q18. Why do teams favor composing several small pure functions over one large function that does everything, from a testing/maintenance standpoint?

::code-wrapper{language="javascript"}
```javascript
const applyDiscount = (rate) => (price) => price - price * rate;
const applyTax = (rate) => (price) => price + price * rate;
const round2 = (price) => Math.round(price * 100) / 100;

const finalPrice = (price) => round2(applyTax(0.08)(applyDiscount(0.1)(price)));
```
::

- [ ] Small functions are always faster to execute than one large function
- [ ] Each small function can be tested and reasoned about independently with trivial input/output assertions, and bugs are easier to isolate to a single stage of the pipeline rather than buried inside one large, tangled function
- [ ] JavaScript engines require functions to be under a certain line count to JIT-compile them
- [ ] Composing small functions eliminates the need for any testing at all

<details>
<summary>Show Answer</summary>

**Answer:** B — Each small function can be tested and reasoned about independently with trivial input/output assertions, and bugs are easier to isolate to a single stage of the pipeline rather than buried inside one large, tangled function

**Explanation:** **Idiom:** because each of `applyDiscount`, `applyTax`, and `round2` is pure and does exactly one thing, each can be unit-tested with a handful of direct input/output examples with no setup or mocking required, and if `finalPrice` produces a wrong result, the bug can be isolated by checking each stage's output in the pipeline rather than debugging one monolithic calculation. This composability is one of the primary practical benefits of functional style in production codebases, independent of any specific execution-speed claim (option A is a fabricated, generally false performance claim — composing functions typically has function-call overhead, not a speed benefit).

</details>
::

::question-wrapper{language="javascript"}
### Q19. A codebase rewrites this straightforward function into "point-free" style. Is this a good idea?

::code-wrapper{language="javascript"}
```javascript
const isAdult = (person) => person.age >= 18;

const isAdultPointFree = compose(gte(18), prop("age"));
```
::

- [ ] Always — point-free style is strictly superior and should be used everywhere possible
- [ ] It depends — point-free style can be elegant for simple, well-named transformations, but overusing it (especially with unfamiliar helper functions like `gte`/`prop`) can hurt readability, making code harder to scan and debug compared to a plain, explicit arrow function
- [ ] Never — point-free style is always harder to read than named-parameter style
- [ ] It doesn't matter — both versions are byte-for-byte identical after minification, so there's no practical difference

<details>
<summary>Show Answer</summary>

**Answer:** B — It depends — point-free style can be elegant for simple, well-named transformations, but overusing it (especially with unfamiliar helper functions like `gte`/`prop`) can hurt readability, making code harder to scan and debug compared to a plain, explicit arrow function

**Explanation:** **Idiom:** point-free style trades an explicit, named parameter (`person`, `person.age`) for implicit data flow through composed helper functions — readable to someone fluent in the specific utility library's vocabulary (`gte`, `prop` are common in libraries like Ramda), but opaque to someone who isn't, and notably harder to debug since there's no intermediate named value to inspect in a debugger or log statement. The original `isAdult` is immediately clear to any JavaScript developer regardless of functional-programming background. Neither extreme (option A "always" or option C "never") reflects real engineering judgment — the right call depends on team familiarity, the complexity of the transformation, and whether the point-free version actually reads more clearly than the explicit one.

</details>
::

::question-wrapper{language="javascript"}
### Q20. A module exports a shared configuration object that multiple other modules import and occasionally mutate directly. What is the idiomatic functional-programming fix for this shared-mutable-state hazard?

::code-wrapper{language="javascript"}
```javascript
export const appConfig = { retries: 3, timeout: 5000 };

// elsewhere, in an unrelated module:
import { appConfig } from "./config.js";
appConfig.retries = 10;
```
::

- [ ] Nothing needs to change — ES module bindings are read-only by default, so this mutation already fails silently
- [ ] Freeze the exported object with `Object.freeze` (deeply, if it has nested data) so accidental mutation throws (in strict mode) or silently no-ops instead of corrupting shared state, and provide an explicit function like `updateConfig(patch)` that returns a new merged object for legitimate updates
- [ ] Export a `let appConfig` instead of `const` so reassignment is explicitly allowed
- [ ] Wrap every property access across the codebase in a `try/catch`

<details>
<summary>Show Answer</summary>

**Answer:** B — Freeze the exported object with `Object.freeze` (deeply, if it has nested data) so accidental mutation throws (in strict mode) or silently no-ops instead of corrupting shared state, and provide an explicit function like `updateConfig(patch)` that returns a new merged object for legitimate updates

**Explanation:** **Idiom:** ES module bindings only make the *binding itself* (`appConfig` as a name) immutable from an importing module's perspective — you can't reassign what `appConfig` refers to from outside the exporting module — but that says nothing about mutating the *object's properties*, which remain fully writable by default (option A's premise is false, echoing the `const` gotcha from Q13, just at module scope instead of function scope). `Object.freeze` closes that hole directly, and pairing it with an explicit, controlled update function keeps all legitimate state changes traceable to one place instead of scattered `appConfig.x = y` mutations anywhere in the codebase, directly addressing the shared-mutable-state risk described in Q7.

</details>
::
