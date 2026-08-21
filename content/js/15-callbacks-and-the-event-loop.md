# 15 — Callbacks & the Event Loop

::question-wrapper
### Q1. What does it actually mean that "JavaScript is single-threaded," in terms of the call stack and the event loop?

- [ ] JS can only execute one function at a time on its single call stack; the event loop, task queues, and the browser/Node runtime's own APIs work alongside that one thread to schedule async callbacks, without ever running two pieces of JS code simultaneously
- [ ] JS spawns a new OS thread for every async operation, such as `setTimeout` or `fetch`
- [ ] Single-threaded means JavaScript cannot use callbacks at all
- [ ] Single-threaded only describes Node.js; browsers run JS on multiple threads

<details>
<summary>Show Answer</summary>

**Answer:** A — JS can only execute one function at a time on its single call stack; the event loop, task queues, and the browser/Node runtime's own APIs work alongside that one thread to schedule async callbacks, without ever running two pieces of JS code simultaneously

**Explanation:** There is exactly one call stack per JS realm, and only one frame ever executes at a time. Timers, network requests, and file I/O are handled by the surrounding environment (browser Web APIs, or Node's `libuv`), which may internally use threads, but the JS callback that eventually reports the result always gets funneled back onto the single call stack, one at a time, via the event loop. Option B is a common misconception — `setTimeout` itself doesn't spawn a thread for your callback; only some underlying runtime plumbing might. Option C is nonsensical. Option D is wrong — the main JS thread in a browser tab is just as single-threaded as Node's.

</details>
::

::question-wrapper
### Q2. Which of the following is classified as a **microtask** rather than a macrotask?

- [ ] A `setTimeout` callback
- [ ] A `setInterval` callback
- [ ] A Promise's `.then()` callback
- [ ] A UI `click` event handler callback

<details>
<summary>Show Answer</summary>

**Answer:** C — A Promise's `.then()` callback

**Explanation:** The microtask queue holds promise reaction callbacks (`.then`/`.catch`/`.finally`) and anything scheduled with `queueMicrotask`. Timers (`setTimeout`, `setInterval`), I/O completions, and UI events like `click` are all macrotasks (sometimes just called "tasks"). This distinction matters because the event loop always fully drains the microtask queue between one macrotask and the next — a rule tested throughout this quiz.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What does this code log, and in what order?

::code-wrapper{language="javascript"}
```javascript
console.log("start");
setTimeout(() => console.log("timeout"), 0);
Promise.resolve().then(() => console.log("promise"));
console.log("end");
```
::

- [ ] `start, promise, end, timeout`
- [ ] `start, end, timeout, promise`
- [ ] `start, promise, timeout, end`
- [ ] `start, end, promise, timeout`

<details>
<summary>Show Answer</summary>

**Answer:** D — `start, end, promise, timeout`

**Explanation:** All synchronous code runs first to completion (`start`, then `end`), regardless of what async work was scheduled in between. Once the call stack is empty, the event loop drains the microtask queue completely before touching the macrotask queue, so the resolved promise's `.then` callback (`promise`) runs next. Only after the microtask queue is empty does the loop pick up the next macrotask, running the timeout callback (`timeout`) last. Option A wrongly assumes an already-resolved promise interrupts synchronous code — it never does, `.then` callbacks are always deferred at least to the microtask queue. Option B reverses microtask/macrotask priority.

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does this code log, and in what order?

::code-wrapper{language="javascript"}
```javascript
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve()
  .then(() => console.log("C"))
  .then(() => console.log("D"));

console.log("E");
```
::

- [ ] `A, E, B, C, D`
- [ ] `A, E, C, D, B`
- [ ] `A, C, D, E, B`
- [ ] `A, E, C, B, D`

<details>
<summary>Show Answer</summary>

**Answer:** B — `A, E, C, D, B`

**Explanation:** **Debug:** synchronous code runs first (`A`, `E`). The first `.then` then logs `C`, and in the process of resolving it, the *second* `.then` gets queued as a brand-new microtask — but the event loop doesn't care that it was queued "late"; it keeps draining the microtask queue as long as anything is in it, so `D` runs before the loop is even allowed to consider the next macrotask. Only once the microtask queue is truly empty does `B` (the timeout) finally run. Option D wrongly inserts the macrotask between the two chained `.then` calls, which would only happen if microtasks queued *during* draining were deferred to the next loop turn — they are not.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does this code log, and in what order?

::code-wrapper{language="javascript"}
```javascript
function first() {
  console.log("first");
}
function second() {
  setTimeout(() => console.log("second"), 0);
}
function third() {
  console.log("third");
}

first();
second();
third();
```
::

- [ ] `first, second, third`
- [ ] `second, first, third`
- [ ] `first, third, second`
- [ ] `first, third`, then a thrown error

<details>
<summary>Show Answer</summary>

**Answer:** C — `first, third, second`

**Explanation:** **Debug:** calling `second()` doesn't log anything itself — it only *registers* a macrotask via `setTimeout`; the string `"second"` lives inside the deferred arrow function, not in `second()`'s own body. So the synchronous call order (`first()`, `second()`, `third()`) produces `first`, then `third` (since `second()` produces no immediate output), and only after the stack is empty does the deferred callback finally run, logging `second` last. The trap here is conflating "the function named `second` was called second" with "the value `\"second\"` logs second" — they're unrelated once a timer is involved.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What does this code log, and in what order?

::code-wrapper{language="javascript"}
```javascript
function loopMicrotasks(n) {
  if (n <= 0) return;
  queueMicrotask(() => {
    console.log("microtask", n);
    loopMicrotasks(n - 1);
  });
}

console.log("sync");
loopMicrotasks(3);
setTimeout(() => console.log("timeout"), 0);
```
::

- [ ] `sync, microtask 3, microtask 2, microtask 1, timeout`
- [ ] `sync, timeout, microtask 3, microtask 2, microtask 1`
- [ ] `sync, microtask 3, timeout, microtask 2, microtask 1`
- [ ] `timeout, sync, microtask 3, microtask 2, microtask 1`

<details>
<summary>Show Answer</summary>

**Answer:** A — `sync, microtask 3, microtask 2, microtask 1, timeout`

**Explanation:** **Performance:** each queued microtask here schedules *another* microtask before returning, and the spec says the event loop keeps processing the microtask queue until it is completely empty — including microtasks added while draining — before it's allowed to move to the next macrotask. So all three self-chaining microtasks run back-to-back, and only then does the `timeout` macrotask get a turn. This is the mechanism behind a real production hazard: a runaway chain of self-queuing microtasks (or promises) can starve macrotasks — including timers and even browser rendering — indefinitely, since the loop never "gets around" to them while microtasks keep refilling the queue.

</details>
::

::question-wrapper{language="javascript"}
### Q7. Setting aside whether it works, what is the primary maintainability problem with this pattern?

::code-wrapper{language="javascript"}
```javascript
getUser(id, (err, user) => {
  getPosts(user.id, (err, posts) => {
    getComments(posts[0].id, (err, comments) => {
      render(user, posts, comments);
    });
  });
});
```
::

- [ ] It runs slower than equivalent synchronous code
- [ ] JavaScript enforces a maximum callback nesting depth of 3
- [ ] Nested callbacks cannot access variables from outer scopes
- [ ] Each nested callback adds indentation and couples unrelated steps into one deeply nested closure, making the code hard to read, test, or modify in isolation — the classic "pyramid of doom"

<details>
<summary>Show Answer</summary>

**Answer:** D — Each nested callback adds indentation and couples unrelated steps into one deeply nested closure, making the code hard to read, test, or modify in isolation — the classic "pyramid of doom"

**Explanation:** This is a structural readability problem, not a functional or performance one: every step is defined inline inside the previous step's callback, so the logic marches rightward and none of the steps can be tested, named, or reused independently. Option B invents a nonexistent engine limit — nesting can go arbitrarily deep (until you run out of patience or screen width). Option C is backwards — nested callbacks *can* close over outer-scope variables (that's exactly how `user` and `posts` stay accessible inward), which is what makes this pattern work at all, just not cleanly. Option A is unrelated; this code's async cost is the same regardless of nesting style.

</details>
::

::question-wrapper{language="javascript"}
### Q8. Compared to the deeply nested version in Q7, what does refactoring into named top-level functions actually fix?

::code-wrapper{language="javascript"}
```javascript
function onComments(err, comments) {
  if (err) return handleError(err);
  render(comments);
}
function onPosts(err, posts) {
  if (err) return handleError(err);
  getComments(posts[0].id, onComments);
}
function onUser(err, user) {
  if (err) return handleError(err);
  getPosts(user.id, onPosts);
}
getUser(id, onUser);
```
::

- [ ] It makes the code run synchronously instead of asynchronously
- [ ] It flattens the visual nesting, gives each step a name, and lets each step be tested independently — but the underlying callback-based control flow (and its complexity) is otherwise unchanged
- [ ] It converts the callbacks into Promises automatically
- [ ] It eliminates the need for error handling entirely

<details>
<summary>Show Answer</summary>

**Answer:** B — It flattens the visual nesting, gives each step a name, and lets each step be tested independently — but the underlying callback-based control flow (and its complexity) is otherwise unchanged

**Explanation:** **Idiom:** naming each step as a top-level function is a genuine readability win over Q7's pyramid — each function can be reasoned about, unit-tested, and reused on its own — but it's a purely structural fix. Execution order, the error-first convention (checking `err` at every single step, as this snippet still does), and the overall async wiring are identical to the nested version. Option D is contradicted by the code itself, which explicitly checks `err` at every level. Option C is false — nothing here touches Promises; that's a separate refactor covered in the next quiz. Option A is false — none of this changes sync/async timing.

</details>
::

::question-wrapper
### Q9. In Node.js-style APIs like `fs.readFile(path, (err, data) => {...})`, what does the "error-first callback" convention actually mean?

- [ ] The callback's first parameter is reserved for an `Error` object (or `null`/`undefined` on success), and callers must check it before trusting later parameters — this is a community convention, not something the language enforces
- [ ] The callback function must be declared before any other code in the file
- [ ] Errors are automatically thrown first, before the callback ever runs
- [ ] The first argument is always a plain string describing the error message

<details>
<summary>Show Answer</summary>

**Answer:** A — The callback's first parameter is reserved for an `Error` object (or `null`/`undefined` on success), and callers must check it before trusting later parameters — this is a community convention, not something the language enforces

**Explanation:** **Idiom:** popularized by early Node.js core APIs, `callback(err, result)` puts the error (or `null` when there isn't one) in a fixed, predictable position. JavaScript itself does nothing to enforce this — a careless caller who forgets to check `err` first will happily read `result` even when it's `undefined` or garbage, which is a frequent real-world bug source. Option D is wrong because `err` is conventionally an `Error` instance (with a `.message` property), not a bare string. Options B and C describe behavior nothing in the convention or the language actually provides.

</details>
::

::question-wrapper{language="javascript"}
### Q10. What happens when this code runs?

::code-wrapper{language="javascript"}
```javascript
try {
  setTimeout(() => {
    throw new Error("boom");
  }, 0);
} catch (e) {
  console.log("caught:", e.message);
}
console.log("after try/catch");
```
::

- [ ] Logs `caught: boom` then `after try/catch`
- [ ] Nothing logs at all; the error is silently swallowed
- [ ] The program crashes immediately at the `setTimeout` line, before anything logs
- [ ] Logs `after try/catch`, and then the thrown error surfaces as an unhandled exception, because the callback runs in a separate macrotask turn, on a stack that no longer has the original `try` block active

<details>
<summary>Show Answer</summary>

**Answer:** D — Logs `after try/catch`, and then the thrown error surfaces as an unhandled exception, because the callback runs in a separate macrotask turn, on a stack that no longer has the original `try` block active

**Explanation:** **Debug:** `try/catch` only guards the code actively on the call stack while it's executing. By the time the `setTimeout` callback finally runs — a fresh call stack, in a later macrotask — the original `try` frame is long gone, so the `throw` propagates as an uncaught exception (visible as an unhandled error in the console, or a crash in Node without a global handler) rather than being caught. This is one of the most common async beginner traps. The fix is to put `try/catch` *inside* the callback itself, or to prefer error-first callbacks / promise rejections over `throw` in async code. Option A is the tempting-but-wrong assumption that a surrounding `try/catch` reaches into deferred callbacks.

</details>
::

::question-wrapper{language="javascript"}
### Q11. In Node.js, what does this code log, and in what order?

::code-wrapper{language="javascript"}
```javascript
console.log("start");

setTimeout(() => console.log("timeout"), 0);

Promise.resolve().then(() => console.log("promise"));

process.nextTick(() => console.log("nextTick"));

console.log("end");
```
::

- [ ] `start, end, promise, nextTick, timeout`
- [ ] `start, nextTick, end, promise, timeout`
- [ ] `start, end, nextTick, promise, timeout`
- [ ] `start, end, timeout, nextTick, promise`

<details>
<summary>Show Answer</summary>

**Answer:** C — `start, end, nextTick, promise, timeout`

**Explanation:** **Portability:** synchronous code always runs first (`start`, `end`). Node.js gives `process.nextTick` callbacks an even higher priority than the standard microtask (promise) queue — Node fully drains the `nextTick` queue before it drains promise microtasks, at the end of every phase. So `nextTick` logs before `promise`, and only after both queues are empty does the `timeout` macrotask run. This is explicitly a Node-specific, non-standard behavior — there is no `process.nextTick` in browsers, and code that needs a portable "run this as soon as possible, before other microtasks" primitive should generally reach for `queueMicrotask` instead, which behaves consistently across environments (though still after `nextTick` in Node).

</details>
::

::question-wrapper{language="javascript"}
### Q12. Given that the busy-wait loop below takes roughly 3 full seconds to finish, when does `"timeout fired"` actually log, relative to the requested 100ms delay?

::code-wrapper{language="javascript"}
```javascript
console.log("start");
setTimeout(() => console.log("timeout fired"), 100);

const end = Date.now() + 3000;
while (Date.now() < end) {} // busy-wait for ~3 seconds

console.log("loop done");
```
::

- [ ] Exactly 100ms after `start`, on a separate thread, unaffected by the loop
- [ ] Only after the ~3-second loop finishes and `"loop done"` logs — the timer became "due" around 100ms in, but its callback can't run until the call stack is empty
- [ ] Immediately after `start`, before the loop even begins
- [ ] Never — the busy-wait permanently cancels any pending timers

<details>
<summary>Show Answer</summary>

**Answer:** B — Only after the ~3-second loop finishes and `"loop done"` logs — the timer became "due" around 100ms in, but its callback can't run until the call stack is empty

**Explanation:** **Performance:** the `100ms` delay only controls when the callback becomes *eligible* to move from the macrotask queue onto the call stack — it says nothing about the stack actually being free at that moment. Since JS is single-threaded (Q1), the synchronous `while` loop occupies the only stack there is for the full 3 seconds, so even though the timer is "ready" after 100ms, the event loop has nowhere to run it until the loop finally releases the stack. This is the sharpest version of "delay is a minimum, not a guarantee": a blocked stack delays every queued callback, including timers that expired long ago. Option A invents a separate thread for the callback itself, which doesn't exist — only the *waiting* happens off-thread, not the callback execution.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What is the concrete risk in how `loadData` is written here, given that a callback's contract should specify how many times it can be invoked?

::code-wrapper{language="javascript"}
```javascript
function loadData(callback) {
  fetchFromCache((err, cached) => {
    if (cached) callback(null, cached);
  });
  fetchFromNetwork((err, fresh) => {
    callback(null, fresh);
  });
}
```
::

- [ ] If both the cache lookup and the network fetch succeed, `callback` runs twice — once with cached data, once with fresh data — which can cause consumers to double-render UI, double-submit data, or throw if they assumed single-invocation semantics
- [ ] It's guaranteed to call `callback` exactly once, because JavaScript functions can only be invoked once per registration
- [ ] This code throws a `SyntaxError`, because `callback` is referenced in two separate places
- [ ] Only the network fetch's callback actually runs; the cache branch is unreachable dead code

<details>
<summary>Show Answer</summary>

**Answer:** A — If both the cache lookup and the network fetch succeed, `callback` runs twice — once with cached data, once with fresh data — which can cause consumers to double-render UI, double-submit data, or throw if they assumed single-invocation semantics

**Explanation:** **Debug:** nothing in JavaScript stops a captured function reference from being called any number of times — each independent async operation here holds its own reference to `callback` and will invoke it on completion regardless of what the other one does. If both succeed, `callback` fires twice with two different results, which is a genuine, common bug: consumers that assume "called once" (a common implicit assumption for callback-based APIs) may double-render, double-submit, or crash on the second call. Fixing it requires an explicit guard (a "already called" flag, unregistering after the first call, or restructuring so only one source can win). Option B states an invented guarantee JS does not provide.

</details>
::

::question-wrapper{language="javascript"}
### Q14. What does this code log, and in what order?

::code-wrapper{language="javascript"}
```javascript
console.log(1);

setTimeout(() => console.log(2), 0);
setTimeout(() => console.log(3), 0);

Promise.resolve().then(() => {
  console.log(4);
  Promise.resolve().then(() => console.log(5));
});

console.log(6);
```
::

- [ ] `1, 6, 4, 2, 5, 3`
- [ ] `1, 4, 5, 6, 2, 3`
- [ ] `1, 6, 2, 3, 4, 5`
- [ ] `1, 6, 4, 5, 2, 3`

<details>
<summary>Show Answer</summary>

**Answer:** D — `1, 6, 4, 5, 2, 3`

**Explanation:** Synchronous code runs first: `1`, `6`. The microtask queue then runs: the first `.then` logs `4` and, while executing, schedules a nested `.then` — which joins the *same* still-draining microtask queue and therefore still runs before any macrotask, logging `5`. Only once the microtask queue is completely empty does the event loop move to the two queued timeouts, running them in the order they were queued (`2`, then `3`, since equal 0ms delays preserve FIFO order). Option A wrongly slots a macrotask between `4` and `5`. Option C wrongly runs both macrotasks before any microtask.

</details>
::

::question-wrapper{language="javascript"}
### Q15. What does this code log, and in what order?

::code-wrapper{language="javascript"}
```javascript
console.log("A");
queueMicrotask(() => console.log("B"));
setTimeout(() => console.log("C"), 0);
queueMicrotask(() => console.log("D"));
console.log("E");
```
::

- [ ] `A, B, D, E, C`
- [ ] `A, B, E, D, C`
- [ ] `A, E, B, D, C`
- [ ] `A, E, C, B, D`

<details>
<summary>Show Answer</summary>

**Answer:** C — `A, E, B, D, C`

**Explanation:** `queueMicrotask` schedules its callback on the exact same priority tier as promise `.then` callbacks — it runs after all synchronous code finishes, in the order the microtasks were queued, and always before the next macrotask. So the synchronous log lines finish first (`A`, `E`), then the two microtasks run in FIFO order (`B`, `D`), and only then does the timeout macrotask fire (`C`). Options A and B both interleave a microtask before synchronous code (`E`) has finished, which never happens — synchronous execution always runs to completion first.

</details>
::

::question-wrapper
### Q16. For sequencing several dependent async steps (fetch a user, then their posts, then comments on the first post), why do modern codebases generally avoid deeply nested callbacks in favor of promise chains (or `async`/`await`)?

- [ ] Promise-based sequencing flattens the code into a linear chain, and centralizes error handling — a single rejection can propagate through the whole chain — removing the pyramid-of-doom indentation and the per-step `if (err) return handleError(err)` boilerplate that manual callbacks require
- [ ] Callbacks are deprecated and no longer supported in modern JS engines
- [ ] Promises execute synchronously, bypassing the event loop entirely, which makes them inherently faster
- [ ] Callbacks cannot access outer-scope variables, so promises are required for closures to work

<details>
<summary>Show Answer</summary>

**Answer:** A — Promise-based sequencing flattens the code into a linear chain, and centralizes error handling — a single rejection can propagate through the whole chain — removing the pyramid-of-doom indentation and the per-step `if (err) return handleError(err)` boilerplate that manual callbacks require

**Explanation:** **Idiom:** the win is structural, not about raw speed or capability — promises are, under the hood, still built on callbacks and the microtask queue (Q2), so option C's "bypasses the event loop" claim is false. Option D is also false; callbacks close over outer scope perfectly well, which is precisely what made the nested pattern in Q7 possible in the first place. Option B is false — callbacks remain foundational and are not deprecated. The genuine benefit, covered in depth in the next quiz on Promises, is that `.then()` chains read top-to-bottom instead of nesting rightward, and a single trailing `.catch()` can handle rejection from any step in the chain instead of requiring a manual `err` check at every level.

</details>
::

::question-wrapper{language="javascript"}
### Q17. The loop below takes roughly 200ms to finish synchronously. Given that, in what order do `"A"` and `"B"` log, and why?

::code-wrapper{language="javascript"}
```javascript
setTimeout(() => console.log("A"), 50);
setTimeout(() => console.log("B"), 10);

for (let i = 0; i < 1_000_000_000; i++) {} // ~200ms of synchronous work

console.log("done looping");
```
::

- [ ] `A`, then `B` — later-registered timers always run first
- [ ] Only `A` fires; `B`'s shorter delay means it gets discarded as "missed"
- [ ] They fire at the exact same moment, since both delays elapsed during the loop
- [ ] `B`, then `A` — both delays have long since elapsed by the time the loop finishes, so both callbacks are already queued, in the order their delays expired (`B`'s 10ms elapsed before `A`'s 50ms)

<details>
<summary>Show Answer</summary>

**Answer:** D — `B`, then `A` — both delays have long since elapsed by the time the loop finishes, so both callbacks are already queued, in the order their delays expired (`B`'s 10ms elapsed before `A`'s 50ms)

**Explanation:** As established in Q12, a busy synchronous loop blocks the stack for its entire duration regardless of what timers become "due" in the meantime. By the time the ~200ms loop finally releases the stack, both the 10ms and 50ms timers are well overdue and sitting in the macrotask queue, ordered by when each became eligible — `B` (10ms) became due before `A` (50ms), so it's queued first and runs first. Option A wrongly uses source-order registration as the tiebreaker instead of elapsed-delay order. Option C is impossible — the event loop only ever runs one macrotask at a time. Option B fabricates a "missed timer" behavior that doesn't exist; a late timer still fires, just later than requested.

</details>
::

::question-wrapper{language="javascript"}
### Q18. What does this code log, and specifically, where does `"executor"` appear in the output?

::code-wrapper{language="javascript"}
```javascript
setTimeout(() => console.log("timeout 1"), 0);

new Promise((resolve) => {
  console.log("executor");
  resolve();
}).then(() => console.log("then 1"));

setTimeout(() => console.log("timeout 2"), 0);

console.log("sync end");
```
::

- [ ] `sync end, executor, then 1, timeout 1, timeout 2`
- [ ] `executor, sync end, then 1, timeout 1, timeout 2`
- [ ] `executor, then 1, sync end, timeout 1, timeout 2`
- [ ] `timeout 1, timeout 2, executor, sync end, then 1`

<details>
<summary>Show Answer</summary>

**Answer:** B — `executor, sync end, then 1, timeout 1, timeout 2`

**Explanation:** **Debug:** the function passed to `new Promise(...)` — the "executor" — runs **synchronously and immediately** the moment the constructor is called; it is not deferred at all. So `"executor"` logs right where the `new Promise(...)` line sits in the normal synchronous flow, before `"sync end"`. Only the `.then()` reaction is deferred, to the microtask queue, running after all synchronous code (`then 1`), and the two `setTimeout` callbacks run last, in registration order. The common misconception (option A) treats everything inside `new Promise(...)` as automatically async — it isn't; only the resolution *handlers* attached via `.then`/`.catch` get queued.

</details>
::

::question-wrapper
### Q19. A codebase's `saveUser(user, cb)` internally calls three callback-based APIs in sequence, each depending on the previous result. A teammate proposes wrapping only the *outermost* call to `saveUser` in a `try/catch` to handle errors from the whole chain. Why is this unsound, given the error-first callback convention from Q9?

- [ ] It's a fine approach — `try/catch` works identically across synchronous and asynchronous code in JavaScript
- [ ] `try/catch` is unnecessary here, because callback-based code never fails
- [ ] Errors from callback-based async APIs are delivered (by convention) as the `err` argument to each individual callback, not thrown onto the call stack that made the initiating call — a surrounding `try/catch` cannot intercept them, so each nested callback must check its own `err`
- [ ] The outer `try/catch` will catch errors from the first callback only, not the second or third

<details>
<summary>Show Answer</summary>

**Answer:** C — Errors from callback-based async APIs are delivered (by convention) as the `err` argument to each individual callback, not thrown onto the call stack that made the initiating call — a surrounding `try/catch` cannot intercept them, so each nested callback must check its own `err`

**Explanation:** This is Q10's lesson applied at the API-design level: each nested async step runs in its own later callback invocation, on a call stack that no longer has the outer `try` frame active, so wrapping the initiating call provides zero protection for errors surfaced via the `err`-first convention — every single callback has to check its own `err` and explicitly handle or forward it. Option D sounds like reasonable partial credit but is still wrong — the outer `try/catch` catches *none* of the async errors, not just the later ones. This exact pain point — manually re-checking `err` at every level with no way to centralize it — is one of the strongest motivations for the promise-based `.catch()` model covered starting in the next quiz.

</details>
::

::question-wrapper{language="javascript"}
### Q20. What does this code log, and in what order?

::code-wrapper{language="javascript"}
```javascript
console.log("1");

setTimeout(() => {
  console.log("2");
  Promise.resolve().then(() => console.log("3"));
}, 0);

Promise.resolve().then(() => {
  console.log("4");
  setTimeout(() => console.log("5"), 0);
});

queueMicrotask(() => console.log("6"));

console.log("7");
```
::

- [ ] `1, 7, 4, 6, 2, 5, 3`
- [ ] `1, 7, 4, 6, 2, 3, 5`
- [ ] `1, 4, 6, 7, 2, 3, 5`
- [ ] `1, 7, 6, 4, 2, 3, 5`

<details>
<summary>Show Answer</summary>

**Answer:** B — `1, 7, 4, 6, 2, 3, 5`

**Explanation:** **Debug:** synchronous code runs first: `1`, `7`. That leaves two macrotasks queued (the first `setTimeout`, and — later — one more once it's created) and two microtasks queued, in this order: the `Promise.resolve().then(...)` handler, then the `queueMicrotask` callback. Draining the microtask queue: the promise handler logs `4` and, inside it, calls `setTimeout(...)` — this only registers a *new macrotask*, it does not add anything to the microtask queue — so draining continues to the `queueMicrotask` callback, logging `6`. With microtasks now empty, the event loop runs the oldest queued macrotask (the original timeout): it logs `2`, then schedules a fresh `Promise.resolve().then(...)`, which becomes a new microtask — and since microtasks are drained fully before the *next* macrotask, `3` logs immediately after `2`. Only then does the loop reach the second macrotask (queued during step `4`), logging `5` last. Options A and D each misplace one step by forgetting that microtasks queued mid-macrotask still drain before the following macrotask.

</details>
::
