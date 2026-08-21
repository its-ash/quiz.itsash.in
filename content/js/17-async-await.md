# 17 — Async/Await

::question-wrapper{language="javascript"}
### Q1. What does the following code actually log?

::code-wrapper{language="javascript"}
```javascript
async function getNum() {
  return 42;
}

const result = getNum();
console.log(result);
```
::

- [ ] 42
- [ ] A `Promise` that resolves to `42`
- [ ] `undefined`
- [ ] A `Promise` that resolves to a `Promise` that resolves to `42`

<details>
<summary>Show Answer</summary>

**Answer:** B — A `Promise` that resolves to `42`

**Explanation:** An `async function` always returns a `Promise`, no matter what its body does. A plain `return value;` is automatically wrapped, so `getNum()` immediately returns a (soon-to-be-fulfilled) `Promise` object, not the raw number — `console.log(result)` prints something like `Promise { 42 }`. Option A is the classic beginner mistake of forgetting the auto-wrap. Option D is wrong because JS never nests promises like that — returned promises/thenables are flattened, not stacked (see Q4).

</details>
::

::question-wrapper{language="javascript"}
### Q2. What order do the logs print in?

::code-wrapper{language="javascript"}
```javascript
async function delayedLog() {
  console.log("A");
  await new Promise(resolve => setTimeout(resolve, 0));
  console.log("B");
}

delayedLog();
console.log("C");
```
::

- [ ] A, C, B
- [ ] A, B, C
- [ ] C, A, B
- [ ] Order can't be determined — it's a race condition

<details>
<summary>Show Answer</summary>

**Answer:** A — A, C, B

**Explanation:** **Debug** — `await` only pauses the *async function itself*, not the whole program. `delayedLog()` runs synchronously up to the `await`, logging "A", then immediately hands control back to the caller — so `console.log("C")` runs next. Only after the current synchronous code and the microtask queue drain does the `setTimeout(..., 0)` macrotask fire, resuming `delayedLog` and logging "B" last. This is the core misconception people bring from single-threaded, blocking languages: `await` does not freeze the whole thread.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What does `fetchUser(1).then(console.log)` print?

::code-wrapper{language="javascript"}
```javascript
async function fetchUser(id) {
  try {
    const res = await Promise.reject(new Error("Network down"));
    return res;
  } catch (err) {
    return err.message;
  }
}

fetchUser(1).then(console.log);
```
::

- [ ] Throws an uncaught `Error: Network down`
- [ ] Logs `undefined`
- [ ] Logs `"Network down"`
- [ ] Logs the `Error` object itself

<details>
<summary>Show Answer</summary>

**Answer:** C — Logs `"Network down"`

**Explanation:** `await` on a rejected promise throws the rejection reason right at the `await` expression, exactly like a synchronous `throw`. Here that's the `Error` object, which the `catch (err)` block receives; `err.message` pulls out the string `"Network down"`, which is what `fetchUser` returns (and therefore resolves its own promise with). Option D is wrong because `.message` extracts the string, not the `Error` instance. Option A is wrong because the rejection is caught locally, so it never escapes as uncaught.

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does this log?

::code-wrapper{language="javascript"}
```javascript
async function inner() {
  return Promise.resolve("done");
}

async function outer() {
  const value = await inner();
  return value;
}

outer().then(v => console.log(typeof v, v));
```
::

- [ ] `object` (an unresolved `Promise`, `done` never appears)
- [ ] `string done`
- [ ] `undefined undefined`
- [ ] Throws — you can't `return` a `Promise` from an `async function`

<details>
<summary>Show Answer</summary>

**Answer:** B — `string done`

**Explanation:** Returning a promise (or any thenable) from an `async function` doesn't create a promise-wrapped-in-a-promise — the outer promise is flattened to adopt the state and value of the returned one. So `inner()` effectively resolves with `"done"`, and `await inner()` in `outer` unwraps that down to the plain string. Option A represents the common "double-wrapping" misconception. Option D is false: returning a promise from an async function is completely normal and is exactly how flattening is triggered.

</details>
::

::question-wrapper{language="javascript"}
### Q5. `fetchUser()` and `fetchStats()` are independent — neither depends on the other's result, and each takes ~300ms.

::code-wrapper{language="javascript"}
```javascript
async function loadDashboard() {
  const user = await fetchUser();
  const stats = await fetchStats();
  return { user, stats };
}
```
::

- [ ] ~150ms
- [ ] ~300ms
- [ ] Indeterminate — depends on the event loop
- [ ] ~600ms

<details>
<summary>Show Answer</summary>

**Answer:** D — ~600ms

**Explanation:** **Performance** — `fetchStats()` isn't even *called* until the `await` on `fetchUser()` finishes, so the two 300ms operations run back-to-back instead of overlapping, even though nothing about them requires that. This is one of the most common async/await performance footguns: writing two independent operations as consecutive `await` lines silently serializes them. The fix is to start both before awaiting either (Q6) or use `Promise.all` (Q7).

</details>
::

::question-wrapper{language="javascript"}
### Q6. Same independent, ~300ms calls as Q5, but written like this instead:

::code-wrapper{language="javascript"}
```javascript
async function loadDashboard() {
  const userPromise = fetchUser();
  const statsPromise = fetchStats();
  const user = await userPromise;
  const stats = await statsPromise;
  return { user, stats };
}
```
::

- [ ] ~300ms total — both calls start immediately and run concurrently
- [ ] ~600ms — identical timing to Q5
- [ ] Throws a race-condition error since both are in flight at once
- [ ] ~300ms, but only if `fetchUser` happens to resolve first

<details>
<summary>Show Answer</summary>

**Answer:** A — ~300ms total — both calls start immediately and run concurrently

**Explanation:** **Performance** — Calling `fetchUser()` and `fetchStats()` without immediately awaiting them kicks off both underlying operations right away; they run concurrently while the async function is paused. The later `await` calls just wait on promises that are already in flight, so total wall time is bounded by the slower of the two (~300ms), not their sum. Option D is a trap: correctness here doesn't depend on which resolves first, since each `await` targets its own dedicated promise variable.

</details>
::

::question-wrapper{language="javascript"}
### Q7. How does this version compare to Q6's manual "start both, then await both" pattern?

::code-wrapper{language="javascript"}
```javascript
async function loadDashboard() {
  const [user, stats] = await Promise.all([fetchUser(), fetchStats()]);
  return { user, stats };
}
```
::

- [ ] It's slower — `Promise.all` adds meaningful overhead
- [ ] It behaves exactly like the sequential version in Q5
- [ ] It runs the same operations concurrently as Q6, as one idiomatic expression, and fails fast if any promise rejects
- [ ] It only works when there are exactly two promises in the array

<details>
<summary>Show Answer</summary>

**Answer:** C — It runs the same operations concurrently as Q6, as one idiomatic expression, and fails fast if any promise rejects

**Explanation:** **Idiom** — `Promise.all([...])` is the standard, idiomatic way to fan out independent async work: `fetchUser()` and `fetchStats()` are both invoked synchronously when the array literal is built, so timing matches Q6, just more concisely. Its key extra behavior is fail-fast semantics: the combined promise rejects as soon as *any* input promise rejects, without waiting for the rest (use `Promise.allSettled` if you need every result regardless of individual failures).

</details>
::

::question-wrapper{language="javascript"}
### Q8. `riskyOperation()` returns a promise that may reject. What happens if it does?

::code-wrapper{language="javascript"}
```javascript
async function process() {
  const promise = riskyOperation();
  try {
    doSomethingElse();
  } catch (err) {
    console.log("caught:", err.message);
  }
  const result = await promise;
  return result;
}
```
::

- [ ] It's caught by the `catch` block, since `promise` was created before the `try`
- [ ] It is NOT caught by the `catch` block — the rejection propagates out of `process()` as a rejected promise
- [ ] The program throws synchronously and crashes immediately
- [ ] The runtime automatically retries `riskyOperation()`

<details>
<summary>Show Answer</summary>

**Answer:** B — It is NOT caught by the `catch` block — the rejection propagates out of `process()` as a rejected promise

**Explanation:** **Debug** — `try/catch` only catches errors from code that runs *textually inside* the `try` block. Here `await promise` sits after the `try/catch` entirely, so its rejection isn't caught locally at all — it makes `process()`'s own returned promise reject. Where the promise was *created* is irrelevant; what matters is where the `await` keyword itself sits relative to `try/catch`.

</details>
::

::question-wrapper{language="javascript"}
### Q9. Now the `await` has moved inside the `try` block, even though `riskyOperation()` is still called before it:

::code-wrapper{language="javascript"}
```javascript
async function process() {
  const promise = riskyOperation();
  try {
    const result = await promise;
    return result;
  } catch (err) {
    return "fallback";
  }
}
```
::

- [ ] The `catch` block never runs because the promise was created outside the `try`
- [ ] This is a syntax error — you can't `await` a promise defined outside a `try` block
- [ ] An unhandled rejection warning still fires even though `catch` runs
- [ ] The `catch` block correctly handles the rejection, because what matters is where `await` is written, not where the promise was created

<details>
<summary>Show Answer</summary>

**Answer:** D — The `catch` block correctly handles the rejection, because what matters is where `await` is written, not where the promise was created

**Explanation:** **Debug** — This mirrors Q8 from the other direction: a promise can be constructed anywhere; only the location of the `await` expression relative to `try/catch` determines whether its rejection is caught. No unhandled-rejection warning fires here, because a catch handler is effectively attached (via `await`'s internal machinery) before the rejection is ever reported as unhandled. Options A and C represent the same "creation site matters" misconception this pair of questions is built to correct.

</details>
::

::question-wrapper{language="javascript"}
### Q10. `saveUser` is `async`. What order do the logs print in when `handleSubmit(user)` runs?

::code-wrapper{language="javascript"}
```javascript
async function saveUser(user) {
  await db.insert(user);
  console.log("saved");
}

function handleSubmit(user) {
  saveUser(user);
  console.log("submitted");
}
```
::

- [ ] "submitted" logs before "saved", because `saveUser(user)` isn't awaited so `handleSubmit` doesn't pause for it
- [ ] "saved" always logs first, because `saveUser` was called first
- [ ] Only "submitted" logs — "saved" never logs since `saveUser` wasn't awaited
- [ ] This throws a runtime error: "must await async function"

<details>
<summary>Show Answer</summary>

**Answer:** A — "submitted" logs before "saved", because `saveUser(user)` isn't awaited so `handleSubmit` doesn't pause for it

**Explanation:** **Debug** — Calling an async function without `await` still *runs* it — the call isn't skipped, only the pausing is. `handleSubmit` fires `saveUser(user)`, immediately gets back a promise it ignores, and moves straight to `console.log("submitted")`. Meanwhile `saveUser`'s own `await` yields to the microtask queue, so its `"saved"` log lands afterward. Option C is the common wrong guess — the function body absolutely still executes, just asynchronously and unobserved.

</details>
::

::question-wrapper{language="javascript"}
### Q11. `saveUser` is `async` and may reject internally. What happens when it does?

::code-wrapper{language="javascript"}
```javascript
async function handleSubmit(user) {
  try {
    saveUser(user);
    console.log("submitted");
  } catch (err) {
    console.log("error:", err.message);
  }
}
```
::

- [ ] The `catch` block logs `"error: ..."` as expected
- [ ] It throws synchronously and crashes `handleSubmit` before `"submitted"` logs
- [ ] The `catch` block never runs — instead an unhandled promise rejection occurs elsewhere, since nothing awaits or `.catch`es `saveUser`'s returned promise
- [ ] JavaScript silently converts the missed `await` into a synchronous call

<details>
<summary>Show Answer</summary>

**Answer:** C — The `catch` block never runs — instead an unhandled promise rejection occurs elsewhere, since nothing awaits or `.catch`es `saveUser`'s returned promise

**Explanation:** **Debug** — Because `saveUser(user)` isn't awaited, its returned promise is orphaned; there's no `await` linking its eventual rejection back into this `try/catch`, so the local `catch` simply never sees it. The rejection instead surfaces later as an unhandled promise rejection (Node's `unhandledRejection` event, or a browser console warning). This is one of the most common real-world async/await bugs — forgetting a single `await` silently defeats the surrounding error handling.

</details>
::

::question-wrapper{language="javascript"}
### Q12. `saveItem` is `async`. What happens when `processAll` runs?

::code-wrapper{language="javascript"}
```javascript
async function processAll(items) {
  items.forEach(async (item) => {
    await saveItem(item);
    console.log("saved", item.id);
  });
  console.log("all done");
}
```
::

- [ ] "all done" logs only after every item has been saved, in order
- [ ] "all done" logs immediately, before any "saved" messages, because `forEach` doesn't wait for the async callbacks it invokes
- [ ] This throws, because `forEach` doesn't accept async callbacks
- [ ] Each iteration automatically awaits the previous one since they share the same array

<details>
<summary>Show Answer</summary>

**Answer:** B — "all done" logs immediately, before any "saved" messages, because `forEach` doesn't wait for the async callbacks it invokes

**Explanation:** **Debug** — `Array.prototype.forEach` ignores whatever its callback returns, promise or not, and never awaits it — it just fires every callback invocation and moves straight on. Marking the callback `async` doesn't change that contract; it just means each call quietly starts its own promise chain in the background while `forEach` itself barrels ahead. So `"all done"` logs before any `"saved"` line, and the saves may finish in arbitrary order. This is an extremely common real-world bug — use a `for...of` loop (Q13) for sequential awaiting or `Promise.all` with `map` (Q14) for parallel awaiting.

</details>
::

::question-wrapper{language="javascript"}
### Q13. Same task, rewritten with `for...of`:

::code-wrapper{language="javascript"}
```javascript
async function processAll(items) {
  for (const item of items) {
    await saveItem(item);
    console.log("saved", item.id);
  }
  console.log("all done");
}
```
::

- [ ] Behaves identically to the `forEach` version in Q12
- [ ] This throws — `for...of` bodies can't contain `await`
- [ ] Items save concurrently, the same as `Promise.all`
- [ ] Each `await` genuinely pauses the loop until that item's save completes, so items save one at a time, in order, before "all done" logs

<details>
<summary>Show Answer</summary>

**Answer:** D — Each `await` genuinely pauses the loop until that item's save completes, so items save one at a time, in order, before "all done" logs

**Explanation:** **Idiom** — Unlike `forEach`, `for...of` is ordinary synchronous control flow wrapped around each iteration; the `await` inside the loop body genuinely suspends the enclosing `async function` until that iteration's promise settles before advancing. This makes `for...of` the right tool when you need strictly sequential, ordered, one-at-a-time processing — reach for `Promise.all` + `map` (Q14) instead when order doesn't matter and speed does.

</details>
::

::question-wrapper{language="javascript"}
### Q14. Same task again, this time written as:

::code-wrapper{language="javascript"}
```javascript
async function processAll(items) {
  await Promise.all(items.map(item => saveItem(item)));
  console.log("all done");
}
```
::

- [ ] All `saveItem` calls start essentially simultaneously, and "all done" logs only once every one has settled successfully
- [ ] This behaves exactly like the `forEach` version in Q12
- [ ] `items.map` isn't allowed to return promises
- [ ] Only the first `saveItem` call actually runs — the rest are discarded

<details>
<summary>Show Answer</summary>

**Answer:** A — All `saveItem` calls start essentially simultaneously, and "all done" logs only once every one has settled successfully

**Explanation:** **Performance**/**Idiom** — `items.map(item => saveItem(item))` synchronously invokes `saveItem` for every item up front, producing an array of promises; `Promise.all(...)` then awaits them all together, resolving only once every one has fulfilled (or rejecting fast on the first failure). This is the idiomatic way to process a collection concurrently when per-item ordering isn't required, and it directly fixes Q12's bug by actually awaiting the work instead of silently dropping it.

</details>
::

::question-wrapper{language="javascript"}
### Q15. In an ES module, a top-level `await` that takes 2 seconds to settle will...

- [ ] Only delay code within that same module file — importing modules are unaffected
- [ ] Cause a syntax error, since `await` is only legal inside `async` functions
- [ ] Delay evaluation of the entire module, and any module that imports it will also wait for that evaluation (including the top-level `await`) to finish before it can use the imports
- [ ] Run in the background without blocking anything, since ESM loading is inherently async already

<details>
<summary>Show Answer</summary>

**Answer:** C — Delay evaluation of the entire module, and any module that imports it will also wait for that evaluation (including the top-level `await`) to finish before it can use the imports

**Explanation:** **Debug**/**Performance** — Top-level `await` (ESM-only) pauses the *evaluation of the containing module itself* at that point until the awaited promise settles. Because the module graph is evaluated respecting dependency order, any module that imports the awaiting module must wait for it to fully finish evaluating — top-level `await` included — before the importer can proceed. This can cascade delays through an entire dependency graph, which is why it's best reserved for genuinely required async setup (like initializing a WASM module) rather than used casually.

</details>
::

::question-wrapper{language="javascript"}
### Q16. `getConfig` is called without `await`. What happens if `key` is falsy?

::code-wrapper{language="javascript"}
```javascript
const getConfig = async (key) => {
  if (!key) {
    throw new Error("key is required");
  }
  return await loadFromDisk(key);
};

try {
  const result = getConfig();
  console.log(result);
} catch (err) {
  console.log("caught:", err.message);
}
```
::

- [ ] `"caught: key is required"` logs, since `throw` always produces a synchronous exception
- [ ] The `catch` block does NOT run — `getConfig()` returns a rejected promise instead, and the rejection goes unhandled
- [ ] Both the `try` block's log and the `catch` block run
- [ ] A `ReferenceError` is thrown because `key` is undefined

<details>
<summary>Show Answer</summary>

**Answer:** B — The `catch` block does NOT run — `getConfig()` returns a rejected promise instead, and the rejection goes unhandled

**Explanation:** **Debug** — Marking a function `async` changes what `throw` does inside it: even a `throw` that happens before any `await` never becomes a synchronous JS-engine exception at the call site — it's converted into a rejected promise returned by `getConfig()`. Since the caller doesn't `await getConfig()`, the surrounding `try/catch` (which only sees synchronous throws and awaited rejections) never observes it, and the rejection surfaces later as unhandled. This trips up developers used to "validate then `throw`" patterns in plain synchronous functions.

</details>
::

::question-wrapper{language="javascript"}
### Q17. Which statement correctly describes `async function*` combined with `for await...of`?

- [ ] It's identical to a regular generator; `for await...of` is just syntax sugar for `for...of` with no behavioral difference
- [ ] Async generators can only be used with arrays, not streams or async iterables
- [ ] `for await...of` requires all values to be available synchronously upfront, unlike `for...of`
- [ ] An async generator's `yield` can produce values that are themselves promises (or come from awaited async work), and `for await...of` automatically awaits each yielded value before running the loop body

<details>
<summary>Show Answer</summary>

**Answer:** D — An async generator's `yield` can produce values that are themselves promises (or come from awaited async work), and `for await...of` automatically awaits each yielded value before running the loop body

**Explanation:** An `async function*` produces an async iterator; `for await...of` consumes it by awaiting each yielded item automatically, one at a time. This makes it a natural fit for representing sequences of values that arrive over time — paginated API results, or chunks read from a stream — which a plain generator plus `for...of` can't express since those only handle synchronous iteration. Options A, B, and C each misstate that relationship.

</details>
::

::question-wrapper{language="javascript"}
### Q18. Is this code valid, and is it good style?

::code-wrapper{language="javascript"}
```javascript
async function loadProfile(id) {
  const user = await fetchUser(id);
  return fetchPosts(user.id).then(posts => ({ user, posts }));
}
```
::

- [ ] It's valid, but mixing `await` and `.then()` chains in the same function is generally considered inconsistent style — sticking to one approach throughout improves readability
- [ ] It throws a syntax error — you can't `return` a `.then()` chain from an `async function`
- [ ] `fetchPosts(user.id).then(...)` runs synchronously before `fetchUser` resolves
- [ ] The `.then()` chain's returned promise is discarded — `loadProfile` always resolves to `undefined`

<details>
<summary>Show Answer</summary>

**Answer:** A — It's valid, but mixing `await` and `.then()` chains in the same function is generally considered inconsistent style — sticking to one approach throughout improves readability

**Explanation:** **Idiom** — Nothing here is actually broken: async functions can freely mix `await` with `.then()`/`.catch()` chains, and the promise returned by `.then()` is flattened into `loadProfile`'s own return value, same as Q4. The issue is purely readability/consistency — switching styles mid-function makes control flow harder to scan. The idiomatic fix is `const posts = await fetchPosts(user.id); return { user, posts };`, keeping the whole function in one style.

</details>
::

::question-wrapper{language="javascript"}
### Q19. What order do the logs print in?

::code-wrapper{language="javascript"}
```javascript
async function test() {
  console.log("1");
  await 42;
  console.log("2");
}

test();
console.log("3");
```
::

- [ ] 1, 2, 3 — awaiting a plain value like `42` doesn't actually pause anything
- [ ] 3, 1, 2 — the whole async function is deferred until the synchronous code finishes
- [ ] 1, 3, 2 — awaiting any value, even a non-promise, always yields to the microtask queue before resuming
- [ ] This throws a `TypeError` — you can only `await` actual `Promise` objects

<details>
<summary>Show Answer</summary>

**Answer:** C — 1, 3, 2 — awaiting any value, even a non-promise, always yields to the microtask queue before resuming

**Explanation:** **Debug**/**Performance** — `await` always suspends the async function and schedules its resumption as a microtask, even when the awaited value isn't a thenable at all — the engine internally treats it as `Promise.resolve(42)`. So `test()` logs `"1"`, then immediately yields back to the caller; `console.log("3")` runs next synchronously; only then does the microtask queue drain and `"2"` logs. This surprises people who assume "no real async work happening, so no delay" — even a trivial `await` costs at least one microtask tick.

</details>
::

::question-wrapper{language="javascript"}
### Q20. A function needs a user's profile, their settings, and their notification count — three independent API calls that don't depend on each other's results — combined into one object. Which approach is best practice?

- [ ] Await each call sequentially, one after another, for the clearest, most linear-reading code
- [ ] Use `.then()` chains for each call, since async/await can't combine multiple independent results
- [ ] Wrap each call in its own `try/catch` and await them one at a time so errors stay isolated per call
- [ ] Kick off all three calls without awaiting immediately (or use `Promise.all`), then await them together, since running independent operations concurrently minimizes total wait time

<details>
<summary>Show Answer</summary>

**Answer:** D — Kick off all three calls without awaiting immediately (or use `Promise.all`), then await them together, since running independent operations concurrently minimizes total wait time

**Explanation:** **Performance**/**Idiom** — When operations don't depend on each other, best practice is to start them concurrently — either call all three functions first and await the results afterward, or more idiomatically write `const [profile, settings, count] = await Promise.all([...])` — so total latency is bounded by the slowest call rather than their sum, avoiding Q5's footgun. Option A is the common but suboptimal sequential pattern; option C still serializes the calls despite sounding "safer" — per-call isolation with `try/catch` isn't required for independent operations (use `Promise.allSettled` instead if partial-failure tolerance is genuinely needed).

</details>
::
