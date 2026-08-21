# 16 — Promises

::question-wrapper{language="javascript"}
### Q1. What are the three states a Promise can be in?

- [ ] waiting, resolved, errored
- [ ] pending, fulfilled, rejected
- [ ] created, running, finished
- [ ] idle, loading, settled

<details>
<summary>Show Answer</summary>

**Answer:** B — pending, fulfilled, rejected

**Explanation:** Every Promise starts in the **pending** state, then transitions exactly once to either **fulfilled** (success, carries a value) or **rejected** (failure, carries a reason). There's no fourth state and no "loading" or "waiting" terminology in the spec — those are the informal names some libraries use, but the ECMAScript spec and every debugger/DevTools panel use pending/fulfilled/rejected.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does this log?

::code-wrapper{language="javascript"}
```javascript
const p = new Promise((resolve, reject) => {
  resolve('first');
  reject('second');
  resolve('third');
});

p.then(console.log).catch(console.log);
```
::

- [ ] 'third'
- [ ] Throws a TypeError for calling resolve/reject more than once
- [ ] 'first'
- [ ] 'second'

<details>
<summary>Show Answer</summary>

**Answer:** C — 'first'

**Explanation:** A promise can settle **only once**. The first call — `resolve('first')` — locks the promise into the fulfilled state with value `'first'`; every subsequent call to `resolve` or `reject` is silently ignored, no error is thrown. This is a deliberate spec guarantee: once settled, a promise's outcome is immutable, so downstream `.then`/`.catch` handlers never have to worry about a value changing out from under them.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What does this log?

::code-wrapper{language="javascript"}
```javascript
Promise.resolve(1)
  .then(n => n + 1)
  .then(n => n + 1)
  .then(console.log);
```
::

- [ ] 1
- [ ] [1, 2, 3]
- [ ] undefined
- [ ] 3

<details>
<summary>Show Answer</summary>

**Answer:** D — 3

**Explanation:** Each `.then()` call returns a **new promise** whose resolved value is whatever the callback returns — that's what makes chaining work. The first `.then` returns a promise resolving to `2`, the second returns one resolving to `3`, and the final `.then` logs that `3`. If a callback returns a plain (non-promise) value, it's simply wrapped and passed straight to the next link.

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does this log, and in what order does the value arrive?

::code-wrapper{language="javascript"}
```javascript
function getUser(id) {
  return Promise.resolve({ id, name: 'Ada' });
}

function getOrders(user) {
  return Promise.resolve([`order-1 for ${user.name}`]);
}

getUser(42)
  .then(user => getOrders(user))
  .then(orders => console.log(orders));
```
::

- [ ] ['order-1 for Ada'] — the chain automatically waits for the inner promise before continuing
- [ ] Promise { <pending> } — the second `.then` runs before `getOrders` settles
- [ ] [Promise] — a promise wrapped inside an array
- [ ] Throws a TypeError because `.then` can't return a promise from inside another `.then`

<details>
<summary>Show Answer</summary>

**Answer:** A — ['order-1 for Ada'] — the chain automatically waits for the inner promise before continuing

**Explanation:** When a `.then` callback returns a promise (rather than a plain value), the chain **auto-flattens**: the outer promise adopts the inner one's eventual state instead of resolving to "a promise containing a promise." So the second `.then` doesn't run until `getOrders`'s returned promise actually settles, and it receives the unwrapped array, not a `Promise` object. This is what lets you chain dependent async calls without manually nesting `.then` inside `.then`.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does this log?

::code-wrapper{language="javascript"}
```javascript
fetchStep1()
  .then(() => { throw new Error('boom in step 1'); })
  .then(() => console.log('step 2'))
  .then(() => console.log('step 3'))
  .catch(err => console.log('caught:', err.message));

function fetchStep1() {
  return Promise.resolve();
}
```
::

- [ ] 'step 2' then 'step 3'
- [ ] 'caught: boom in step 1'
- [ ] 'step 2' then 'caught: boom in step 1'
- [ ] Nothing — the error crashes the script before anything logs

<details>
<summary>Show Answer</summary>

**Answer:** B — 'caught: boom in step 1'

**Explanation:** A single `.catch()` at the end of a chain catches a rejection from **any earlier link**, not just the one immediately before it. Once the first `.then` throws, the promise it returns rejects, and every subsequent `.then` in the chain is **skipped** — rejections propagate past `.then` handlers (they only have fulfillment callbacks registered) until a handler with a rejection callback (`.catch`, or the second argument to `.then`) is found. `'step 2'` and `'step 3'` never run.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What does this log?

::code-wrapper{language="javascript"}
```javascript
Promise.reject(new Error('network down'))
  .then(
    value => console.log('fulfilled:', value),
    err => console.log('rejected in 2nd arg:', err.message)
  )
  .catch(err => console.log('caught:', err.message));
```
::

- [ ] 'caught: network down'
- [ ] Nothing — passing two arguments to `.then` is a TypeError
- [ ] 'rejected in 2nd arg: network down'
- [ ] Both lines log, in that order

<details>
<summary>Show Answer</summary>

**Answer:** C — 'rejected in 2nd arg: network down'

**Explanation:** `.then(onFulfilled, onRejected)` registers the second argument as a rejection handler for the promise `.then` is called ON. Since the original promise is already rejected, that second-argument handler catches it directly, logs the message, and — because it doesn't throw or return a rejected promise — the promise `.then` returns **resolves** (not rejects). So the trailing `.catch` has nothing to catch and never runs.

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does this log?

::code-wrapper{language="javascript"}
```javascript
Promise.resolve('ok')
  .then(
    value => { throw new Error('thrown inside onFulfilled'); },
    err => console.log('rejected handler:', err.message)
  )
  .catch(err => console.log('caught by catch:', err.message));
```
::

- [ ] 'rejected handler: thrown inside onFulfilled'
- [ ] Nothing logs — the throw is swallowed since a rejection handler was already supplied
- [ ] Both the rejected handler and the catch run
- [ ] 'caught by catch: thrown inside onFulfilled'

<details>
<summary>Show Answer</summary>

**Answer:** D — 'caught by catch: thrown inside onFulfilled'

**Explanation:** This is the classic two-arg-`.then` gotcha: the `onRejected` second argument only catches a rejection of the promise `.then` was called on — it does **not** catch an error thrown from inside the `onFulfilled` callback sitting right next to it. Since the original promise was fulfilled, `onFulfilled` runs, throws, and that throw becomes a rejection of the *new* promise `.then` returns — which is exactly what the chained `.catch()` picks up. Using `.then(fn).catch(handler)` instead of `.then(fn, handler)` is the idiomatic fix precisely because `.catch` sits downstream of `.then`'s own callback and can catch throws from within it.

</details>
::

::question-wrapper{language="javascript"}
### Q8. What does this log, and in what order?

::code-wrapper{language="javascript"}
```javascript
Promise.resolve('data')
  .then(val => {
    console.log('handler:', val);
    return val.toUpperCase();
  })
  .finally(() => console.log('finally ran'))
  .then(val => console.log('after finally:', val));
```
::

- [ ] 'handler: data' then 'finally ran' then 'after finally: DATA'
- [ ] 'handler: data' then 'finally ran' then 'after finally: undefined'
- [ ] 'finally ran' then 'handler: data' then 'after finally: DATA'
- [ ] 'handler: data' then 'after finally: DATA' then 'finally ran'

<details>
<summary>Show Answer</summary>

**Answer:** A — 'handler: data' then 'finally ran' then 'after finally: DATA'

**Explanation:** `.finally()` runs after the preceding link settles, regardless of whether it fulfilled or rejected, but its callback receives **no argument** — it can't see the value or reason, and it doesn't need to, since its job is cleanup (closing a spinner, releasing a resource) rather than transforming data. Critically, `.finally()` passes the original outcome **through unchanged** to the next `.then`, so `'DATA'` survives past the `finally` untouched — it does not become `undefined` just because the `finally` callback returned nothing.

</details>
::

::question-wrapper{language="javascript"}
### Q9. What does this log?

::code-wrapper{language="javascript"}
```javascript
Promise.reject(new Error('save failed'))
  .finally(() => console.log('cleanup'))
  .catch(err => console.log('caught:', err.message));
```
::

- [ ] 'caught: save failed' then 'cleanup'
- [ ] 'cleanup' then 'caught: save failed'
- [ ] Nothing logs — `.finally` can't be chained after a rejected promise
- [ ] Only 'cleanup' — `.finally` consumes the rejection so `.catch` never runs

<details>
<summary>Show Answer</summary>

**Answer:** B — 'cleanup' then 'caught: save failed'

**Explanation:** `.finally()` runs its callback on rejection too, but since it doesn't return a value that replaces the outcome, the original rejection **propagates through** to the next handler in the chain — it does not get swallowed. So `'cleanup'` logs first (as the finally callback fires), and the still-pending rejection then reaches `.catch`, which logs the error message. The one exception (not shown here) is if the `finally` callback itself throws or returns a rejected promise — then *that* new rejection would override the original.

</details>
::

::question-wrapper{language="javascript"}
### Q10. Three API calls each take a different amount of time and one of them fails. What does this log?

::code-wrapper{language="javascript"}
```javascript
const fast = new Promise(res => setTimeout(() => res('fast done'), 10));
const slow = new Promise(res => setTimeout(() => res('slow done'), 100));
const broken = new Promise((_, rej) => setTimeout(() => rej(new Error('broke')), 50));

Promise.all([fast, slow, broken])
  .then(results => console.log('all:', results))
  .catch(err => console.log('all failed:', err.message));
```
::

- [ ] 'all failed: broke' — but only after all three have settled at 100ms
- [ ] 'all: [\'fast done\', \'slow done\', undefined]'
- [ ] 'all failed: broke' — logged around the 50ms mark, without waiting for `slow`
- [ ] Nothing logs because one promise in the array rejected

<details>
<summary>Show Answer</summary>

**Answer:** C — 'all failed: broke' — logged around the 50ms mark, without waiting for `slow`

**Explanation:** `Promise.all` is **fail-fast**: it rejects as soon as any single input promise rejects, immediately propagating that rejection to the `.catch`, even though `slow` is still pending at that moment (and will keep running in the background, its eventual result simply discarded by this code). This is the key difference from `Promise.allSettled`, which would wait for every promise to settle no matter what and never short-circuit.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What does this log?

::code-wrapper{language="javascript"}
```javascript
const results = await Promise.allSettled([
  Promise.resolve('ok-1'),
  Promise.reject(new Error('bad')),
  Promise.resolve('ok-2'),
]);
```
::

- [ ] Throws because one of the promises rejected
- [ ] ['ok-1', 'ok-2'] — rejected entries are filtered out automatically
- [ ] [{status: 'fulfilled', value: 'ok-1'}, undefined, {status: 'fulfilled', value: 'ok-2'}]
- [ ] [{status: 'fulfilled', value: 'ok-1'}, {status: 'rejected', reason: Error('bad')}, {status: 'fulfilled', value: 'ok-2'}]

<details>
<summary>Show Answer</summary>

**Answer:** D — [{status: 'fulfilled', value: 'ok-1'}, {status: 'rejected', reason: Error('bad')}, {status: 'fulfilled', value: 'ok-2'}]

**Explanation:** `Promise.allSettled` never short-circuits and never rejects — it waits for **every** input promise to settle (whether fulfilled or rejected) and resolves with an array of outcome objects, one per input, in the original order. Fulfilled entries get `{status: 'fulfilled', value}`, rejected entries get `{status: 'rejected', reason}`. This makes it the right tool when you need results from independent operations even if some of them fail, unlike `Promise.all`'s all-or-nothing behavior.

</details>
::

::question-wrapper{language="javascript"}
### Q12. Three mirrors of the same resource are raced to find whichever responds first. What does this log?

::code-wrapper{language="javascript"}
```javascript
const mirrorA = new Promise(res => setTimeout(() => res('A responded'), 200));
const mirrorB = new Promise((_, rej) => setTimeout(() => rej(new Error('B timed out')), 30));
const mirrorC = new Promise(res => setTimeout(() => res('C responded'), 150));

Promise.race([mirrorA, mirrorB, mirrorC])
  .then(v => console.log('winner:', v))
  .catch(e => console.log('race rejected:', e.message));
```
::

- [ ] 'race rejected: B timed out'
- [ ] 'winner: C responded'
- [ ] 'winner: A responded'
- [ ] Nothing — `Promise.race` requires every input to fulfill, never reject

<details>
<summary>Show Answer</summary>

**Answer:** A — 'race rejected: B timed out'

**Explanation:** `Promise.race` settles with whichever input promise settles **first, period** — fulfillment and rejection are treated identically as "settling." `mirrorB` rejects at 30ms, which is earlier than `mirrorC`'s fulfillment at 150ms or `mirrorA`'s at 200ms, so the race settles as a rejection even though two of the three mirrors would have eventually succeeded. This trips people up because "race" sounds like it should mean "first success" — that behavior is actually `Promise.any`, covered next.

</details>
::

::question-wrapper{language="javascript"}
### Q13. Using the same three mirrors from the previous question, what does `Promise.any` produce?

::code-wrapper{language="javascript"}
```javascript
const mirrorA = new Promise(res => setTimeout(() => res('A responded'), 200));
const mirrorB = new Promise((_, rej) => setTimeout(() => rej(new Error('B timed out')), 30));
const mirrorC = new Promise(res => setTimeout(() => res('C responded'), 150));

Promise.any([mirrorA, mirrorB, mirrorC])
  .then(v => console.log('winner:', v))
  .catch(e => console.log('any rejected:', e));
```
::

- [ ] 'any rejected: Error: B timed out'
- [ ] 'winner: C responded'
- [ ] 'winner: B timed out'
- [ ] 'winner: A responded'

<details>
<summary>Show Answer</summary>

**Answer:** B — 'winner: C responded'

**Explanation:** `Promise.any` resolves with the first promise to **fulfill**, ignoring rejections along the way — it only rejects if *all* inputs reject (with an `AggregateError` collecting every individual reason). Here `mirrorB` rejects at 30ms but that's not enough to settle the race; `mirrorC` is the first to *fulfill*, at 150ms, so that's the winner, even though `mirrorB` technically settled earlier.

</details>
::

::question-wrapper{language="javascript"}
### Q14. All three mirrors are down. What does this log?

::code-wrapper{language="javascript"}
```javascript
Promise.any([
  Promise.reject(new Error('A down')),
  Promise.reject(new Error('B down')),
  Promise.reject(new Error('C down')),
]).catch(err => {
  console.log(err.name);
  console.log(err.errors.map(e => e.message));
});
```
::

- [ ] 'Error' then ['A down', 'B down', 'C down']
- [ ] Nothing logs — `.catch` never fires because `Promise.any` can't reject
- [ ] 'AggregateError' then ['A down', 'B down', 'C down']
- [ ] 'AggregateError' then ['A down']  — only the first rejection is kept

<details>
<summary>Show Answer</summary>

**Answer:** C — 'AggregateError' then ['A down', 'B down', 'C down']

**Explanation:** When **every** input to `Promise.any` rejects, it rejects with a single `AggregateError` whose `.errors` property is an array holding all the individual rejection reasons in input order — not just the first or last one. This lets a caller inspect exactly why every candidate failed, unlike a plain `Error` which could only carry one message.

</details>
::

::question-wrapper{language="javascript"}
### Q15. What does this log?

::code-wrapper{language="javascript"}
```javascript
const original = Promise.resolve('value');
const wrapped = Promise.resolve(original);

console.log(wrapped === original);
```
::

- [ ] false
- [ ] undefined — comparing promises with `===` always throws
- [ ] false, but `wrapped` resolves to a Promise object rather than 'value'
- [ ] true

<details>
<summary>Show Answer</summary>

**Answer:** D — true

**Explanation:** `Promise.resolve()` has a special case: if you pass it a value that is **already a genuine promise**, it returns that exact same promise instance rather than wrapping it in a new one — there's no such thing as a "promise of a promise" in the spec's eyes. This differs from passing a plain **thenable** (a non-promise object with a `.then` method), where `Promise.resolve` creates a brand-new native promise that adopts the thenable's eventual state instead of returning the thenable itself.

</details>
::

::question-wrapper{language="javascript"}
### Q16. What order do these log in?

::code-wrapper{language="javascript"}
```javascript
console.log('1: script start');

new Promise(resolve => {
  console.log('2: executor runs');
  resolve('done');
}).then(val => console.log('4: then callback,', val));

console.log('3: script end');
```
::

- [ ] '1: script start', '2: executor runs', '3: script end', '4: then callback, done'
- [ ] '1: script start', '3: script end', '2: executor runs', '4: then callback, done'
- [ ] '1: script start', '2: executor runs', '4: then callback, done', '3: script end'
- [ ] '2: executor runs', '1: script start', '3: script end', '4: then callback, done'

<details>
<summary>Show Answer</summary>

**Answer:** A — '1: script start', '2: executor runs', '3: script end', '4: then callback, done'

**Explanation:** The function passed to `new Promise(...)` — the **executor** — runs **synchronously and immediately**, right when the constructor is called, not deferred to a later tick. That's why `'2: executor runs'` logs in the middle of the synchronous script, before `'3: script end'`. But `.then` callbacks are always deferred to the **microtask queue**, even when the promise is already resolved by the time `.then` is attached — so `'4:...'` can only run after the current synchronous run of the script finishes, landing last.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What does this log?

::code-wrapper{language="javascript"}
```javascript
const p = new Promise((resolve, reject) => {
  JSON.parse('{ this is not valid json');
  resolve('never reached');
});

p.catch(err => console.log('caught:', err.constructor.name));
```
::

- [ ] Nothing — the malformed JSON crashes the script since it's thrown outside a try/catch
- [ ] 'caught: SyntaxError'
- [ ] The promise stays pending forever because `reject` was never explicitly called
- [ ] 'caught: undefined'

<details>
<summary>Show Answer</summary>

**Answer:** B — 'caught: SyntaxError'

**Explanation:** Any synchronous throw inside a promise executor is **automatically caught by the Promise machinery itself** and converted into a rejection with that thrown value as the reason — you don't need your own try/catch inside the executor. `JSON.parse` throws a `SyntaxError` on malformed input, that throw never reaches `resolve('never reached')`, and the promise settles as rejected with the `SyntaxError` as its reason, which `.catch` then receives.

</details>
::

::question-wrapper{language="javascript"}
### Q18. This promise chain has no `.catch()` anywhere. What's the most accurate description of what happens when it runs (e.g., in a browser)?

::code-wrapper{language="javascript"}
```javascript
function loadUserProfile(id) {
  return fetch(`/api/users/${id}`)
    .then(res => res.json())
    .then(data => data.profile.avatarUrl.toUpperCase());
}

loadUserProfile(999);
```
::

- [ ] JavaScript automatically logs the error to the console and continues as if `.catch` had been attached
- [ ] The entire script halts immediately at `loadUserProfile(999)`, exactly like an uncaught synchronous throw
- [ ] If any link in the chain rejects, the rejection goes unhandled: the runtime fires an `unhandledrejection` event (and Node can be configured to crash the process on it), but the error is otherwise silently swallowed from the caller's perspective
- [ ] Nothing special happens — a promise chain without `.catch` behaves identically to one with `.catch`

<details>
<summary>Show Answer</summary>

**Answer:** C — If any link in the chain rejects, the rejection goes unhandled: the runtime fires an `unhandledrejection` event (and Node can be configured to crash the process on it), but the error is otherwise silently swallowed from the caller's perspective

**Explanation:** Without a terminal `.catch()` (or a rejection handler somewhere downstream), a rejection anywhere in this chain — a network failure, a missing `profile` field making `.avatarUrl` throw — becomes an **unhandled rejection**. Browsers surface this via the `unhandledrejection` event on `window` (usually just a console warning); Node.js emits the same event and, depending on configuration, can terminate the process. Critically, the calling code (`loadUserProfile(999)` here) gets no synchronous exception and no return value indicating failure — the error simply vanishes from the caller's point of view unless something is listening for that event or a `.catch` exists in the chain.

</details>
::

::question-wrapper{language="javascript"}
### Q19. Which change makes this function follow best practice for promise-based error handling?

::code-wrapper{language="javascript"}
```javascript
function saveDraft(doc) {
  db.write(doc)
    .then(() => notifyAutosaveSuccess())
    .then(() => updateLastSavedTimestamp());
}
```
::

- [ ] Nothing needs to change — chains without `.catch` are fine as long as the individual functions don't throw
- [ ] Replace `.then` with `.finally` on every step so failures are automatically retried
- [ ] Wrap the whole function body in `try { } catch { }` instead — that's sufficient to catch promise rejections
- [ ] Add a `.catch()` at the end of the chain (and `return` the chain) so a failure in `db.write`, the notify step, or the timestamp update is handled instead of becoming an unhandled rejection

<details>
<summary>Show Answer</summary>

**Answer:** D — Add a `.catch()` at the end of the chain (and `return` the chain) so a failure in `db.write`, the notify step, or the timestamp update is handled instead of becoming an unhandled rejection

**Explanation:** **Idiom:** every promise chain should terminate in a `.catch()` (or otherwise have its rejection handled by the caller) — otherwise a failure at any link becomes an unhandled rejection that silently disappears instead of being logged, retried, or surfaced to the user. A plain synchronous `try/catch` wrapped around code that merely *starts* a promise chain does **not** catch asynchronous rejections that happen later, since the `try` block has already finished executing by the time `.then` callbacks run — that's a common misconception. `.finally` is for cleanup, not retries, and doesn't handle rejections either.

</details>
::

::question-wrapper{language="javascript"}
### Q20. What does this log?

::code-wrapper{language="javascript"}
```javascript
function chargeCard(amount) {
  return new Promise((resolve, reject) => {
    if (amount <= 0) {
      reject(new Error('invalid amount'));
      return;
    }
    resolve(`charged $${amount}`);
  });
}

chargeCard(-5)
  .then(receipt => receipt)
  .then(receipt => console.log('receipt:', receipt))
  .catch(err => console.log('charge failed:', err.message))
  .then(() => console.log('cleanup: closing dialog'));
```
::

- [ ] 'charge failed: invalid amount' then 'cleanup: closing dialog'
- [ ] 'receipt: undefined' then 'cleanup: closing dialog'
- [ ] 'charge failed: invalid amount' — the final `.then` never runs because the chain already rejected
- [ ] 'cleanup: closing dialog' then 'charge failed: invalid amount'

<details>
<summary>Show Answer</summary>

**Answer:** A — 'charge failed: invalid amount' then 'cleanup: closing dialog'

**Explanation:** `chargeCard(-5)` rejects, so both intermediate `.then(receipt => receipt)` calls are skipped (rejections skip past `.then` handlers with no rejection callback) until `.catch` picks it up and logs the message. Since that `.catch` callback returns normally (doesn't throw or return a rejected promise), the promise it produces **fulfills**, so the chain is back in the "happy path" — the final `.then` runs normally and logs the cleanup line. This is the same "catch resets the chain to fulfilled" mechanism that lets a single `.catch` recover from an error and let subsequent `.then` steps continue.

</details>
::
