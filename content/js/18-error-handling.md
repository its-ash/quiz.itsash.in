# 18 — Error Handling

::question-wrapper
### Q1. Which statement about the `finally` block in a `try/catch/finally` statement is true?

- [ ] `finally` only runs if the `try` block completes without throwing
- [ ] `finally` runs after every `try`/`catch` path, including when `try` or `catch` contains a `return`
- [ ] `finally` is skipped if the `catch` block itself throws a new error
- [ ] `finally` only runs when there is no `catch` block present

<details>
<summary>Show Answer</summary>

**Answer:** B — `finally` runs after every `try`/`catch` path, including when `try` or `catch` contains a `return`

**Explanation:** `finally` always executes regardless of how `try`/`catch` exits — normal completion, `return`, `throw`, or even `break`/`continue`. Even if `catch` throws a new error, `finally` still runs before that error propagates further, which rules out C. A is wrong because `finally` runs on both success and failure paths, not just success. D is wrong because `try/finally` (with no `catch` at all) is perfectly valid, and `finally` still runs.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does this log?

::code-wrapper{language="javascript"}
```javascript
function getStatus() {
  try {
    return "success";
  } finally {
    return "overridden";
  }
}

console.log(getStatus());
```
::

- [ ] "success"
- [ ] `undefined`
- [ ] A `SyntaxError` is thrown because you cannot `return` twice in one function
- [ ] "overridden"

<details>
<summary>Show Answer</summary>

**Answer:** D — "overridden"

**Explanation:** A `return` inside `finally` silently overrides the pending return value from `try`. The engine has already set up `"success"` as the function's about-to-be-returned value, but before control actually leaves the function, `finally` runs — and its own `return "overridden"` replaces that pending value entirely. No error is thrown; this is legal but dangerous JavaScript (ESLint's `no-unsafe-finally` rule exists specifically to flag it). A is the intuitive-but-wrong guess that `try`'s `return` should win. **Debug**

</details>
::

::question-wrapper{language="javascript"}
### Q3. What does this log?

::code-wrapper{language="javascript"}
```javascript
function loadConfig() {
  try {
    throw new Error("config missing");
  } finally {
    return "default-config";
  }
}

console.log(loadConfig());
```
::

- [ ] "default-config" is logged, and the thrown error is completely swallowed
- [ ] The function throws "config missing" and the `console.log` line never runs
- [ ] Both the error message and the return value are logged
- [ ] A `TypeError` is thrown because `finally` cannot contain a `return` after a `throw` in `try`

<details>
<summary>Show Answer</summary>

**Answer:** A — "default-config" is logged, and the thrown error is completely swallowed

**Explanation:** Just like it overrides a pending `return`, a `return` inside `finally` also overrides a pending *exception*. The `throw new Error("config missing")` in `try` never propagates because `finally`'s `return "default-config"` becomes the completion value of the whole statement instead. This silently discards a real error with no trace it ever happened — a genuinely dangerous pattern, which is why linters flag `return`/`break`/`continue` inside `finally`. **Debug** / **Safety**

</details>
::

::question-wrapper{language="javascript"}
### Q4. What happens when this runs?

::code-wrapper{language="javascript"}
```javascript
class ApiError extends Error {
  constructor(message, statusCode) {
    this.statusCode = statusCode;
    this.message = message;
  }
}

const err = new ApiError("Not Found", 404);
```
::

- [ ] `err` is created successfully and `err.message` is "Not Found"
- [ ] `err` is created but `err instanceof Error` is `false`
- [ ] A `ReferenceError` is thrown because `this` is accessed before `super()` is called
- [ ] `err.statusCode` is `undefined` because it was set before `message`

<details>
<summary>Show Answer</summary>

**Answer:** C — A `ReferenceError` is thrown because `this` is accessed before `super()` is called

**Explanation:** In a derived class, `this` is not initialized until `super()` runs. Writing `this.statusCode = statusCode` before calling `super()` throws "Must call super constructor in derived class before accessing 'this' or returning from derived constructor." The fix is `super(message)` first, then assign any extra custom fields like `statusCode`. A is the outcome you'd get *after* fixing the bug, not what actually happens here. **Safety**

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does this log?

::code-wrapper{language="javascript"}
```javascript
class AppError extends Error {}
class ValidationError extends AppError {}

function validate(input) {
  if (!input) throw new ValidationError("input required");
}

try {
  validate(null);
} catch (err) {
  if (err instanceof AppError) {
    console.log("handled:", err instanceof ValidationError);
  }
}
```
::

- [ ] Nothing is logged because `ValidationError` does not match `AppError`
- [ ] "handled: true", because `instanceof` walks the entire prototype chain
- [ ] "handled: false", because `instanceof` only checks the direct class
- [ ] A `TypeError` is thrown since you cannot `throw` a subclass of a subclass of `Error`

<details>
<summary>Show Answer</summary>

**Answer:** B — "handled: true", because `instanceof` walks the entire prototype chain

**Explanation:** `instanceof` checks the full prototype chain, not just the exact constructor. A `ValidationError` instance is also an `instanceof AppError` and an `instanceof Error`, so both checks in the snippet are `true`. This lets a broad `instanceof` check higher up a hierarchy catch a whole family of related errors while still allowing narrower checks. C describes a common misconception that `instanceof` only matches the exact class it was constructed with.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What does this log?

::code-wrapper{language="javascript"}
```javascript
class TimeoutError extends Error {
  constructor(message) {
    super(message);
  }
}

const err = new TimeoutError("request timed out");
console.log(err.name, err instanceof TimeoutError);
```
::

- [ ] "TimeoutError" and `true`
- [ ] "TimeoutError" and `false`
- [ ] `undefined` and `true`
- [ ] "Error" and `true`

<details>
<summary>Show Answer</summary>

**Answer:** D — "Error" and `true`

**Explanation:** `Error`'s own constructor sets `this.name` to `"Error"` by default; simply extending the class does not update `.name` to match the subclass — you must explicitly set `this.name = "TimeoutError"` (typically right after `super(message)`) for logs and error-reporting tools that read `.name` to show the right label. Meanwhile `instanceof` is still `true` because it's driven entirely by the prototype chain, a completely independent mechanism from the `.name` string — which is exactly what trips people up. **Debug**

</details>
::

::question-wrapper{language="javascript"}
### Q7. What's the problem with this code, and the best fix?

::code-wrapper{language="javascript"}
```javascript
async function saveUser(user) {
  try {
    await db.insert(user);
  } catch (err) {
    console.error("saveUser failed:", err.message);
  }
}
```
::

- [ ] The `catch` block swallows the error; callers of `saveUser` have no way to know it failed unless the error is rethrown after logging
- [ ] Nothing — logging the error is sufficient error handling
- [ ] `await` cannot be used inside a `try` block
- [ ] The error should be logged with `console.log` instead of `console.error`

<details>
<summary>Show Answer</summary>

**Answer:** A — The `catch` block swallows the error; callers of `saveUser` have no way to know it failed unless the error is rethrown after logging

**Explanation:** Logging without rethrowing lets `saveUser`'s returned promise resolve successfully even though the insert failed — any caller doing `await saveUser(user)` has no way to detect the failure and proceeds as if it succeeded. The log-and-rethrow pattern (log for diagnostics, then `throw err` or a wrapped error) keeps both observability and correct control flow intact. **Idiom**

</details>
::

::question-wrapper{language="javascript"}
### Q8. Compare the two functions below. Which statement is correct?

::code-wrapper{language="javascript"}
```javascript
function readFile(path) {
  try {
    return fs.readFileSync(path);
  } catch (err) {
    throw err;
  }
}

function readFileWrapped(path) {
  try {
    return fs.readFileSync(path);
  } catch (err) {
    throw new Error("read failed");
  }
}
```
::

- [ ] Both functions produce an error whose `.stack` points to the original `fs.readFileSync` failure
- [ ] Neither function preserves any stack information once the error is caught
- [ ] `readFile`'s rethrown error keeps its original `.stack` (captured when it was first constructed), but `readFileWrapped` creates a brand-new error whose `.stack` starts at the `new Error("read failed")` line, losing the original failure site unless passed via `cause`
- [ ] `readFileWrapped` automatically appends the original error's stack to the new error's `.stack` property

<details>
<summary>Show Answer</summary>

**Answer:** C — `readFile`'s rethrown error keeps its original `.stack`, but `readFileWrapped` loses it unless passed via `cause`

**Explanation:** An `Error`'s `.stack` is captured once, at construction time — not at throw time. `throw err` in `readFile` rethrows the exact same object, so `.stack` still shows where `fs.readFileSync` originally failed. `readFileWrapped` constructs a brand-new `Error`, so its `.stack` starts fresh at the `new Error("read failed")` line; the original failure location is lost unless explicitly preserved, e.g. `new Error("read failed", { cause: err })`. **Debug**

</details>
::

::question-wrapper{language="javascript"}
### Q9. What does this log?

::code-wrapper{language="javascript"}
```javascript
function checkAge(age) {
  if (age < 0) throw "invalid age";
}

try {
  checkAge(-5);
} catch (err) {
  console.log(err.message, err.stack);
}
```
::

- [ ] "invalid age" and a full stack trace are logged
- [ ] `undefined undefined` is logged, because a thrown string has no `.message` or `.stack` property
- [ ] A `TypeError` is thrown because only `Error` instances can be thrown
- [ ] `err` is automatically wrapped into an `Error` object by the JS engine before reaching `catch`

<details>
<summary>Show Answer</summary>

**Answer:** B — `undefined undefined` is logged, because a thrown string has no `.message` or `.stack` property

**Explanation:** JavaScript allows `throw` on any value — strings, numbers, plain objects — not only `Error` instances. `throw "invalid age"` means `err` inside `catch` is literally the string `"invalid age"`, which has no `.message` or `.stack`, so both log as `undefined`. This is a real anti-pattern: always `throw new Error(...)` (or a subclass) so consuming code and tooling — debuggers, error trackers, logging libraries — can rely on `.message`/`.stack`/`.name` existing. **Safety**

</details>
::

::question-wrapper{language="javascript"}
### Q10. What happens when `handleRequest` runs?

::code-wrapper{language="javascript"}
```javascript
async function processOrder(order) {
  if (!order.items.length) throw new Error("empty order");
  return "processed";
}

function handleRequest(order) {
  try {
    processOrder(order);
    console.log("request accepted");
  } catch (err) {
    console.log("caught:", err.message);
  }
}

handleRequest({ items: [] });
```
::

- [ ] "caught: empty order" is logged
- [ ] Nothing is logged because `handleRequest` throws synchronously
- [ ] The `catch` block runs before "request accepted" is logged
- [ ] "request accepted" is logged, and the rejection from `processOrder` becomes an unhandled promise rejection instead of being caught

<details>
<summary>Show Answer</summary>

**Answer:** D — "request accepted" is logged, and the rejection from `processOrder` becomes an unhandled promise rejection instead of being caught

**Explanation:** `processOrder` is `async`, so calling it without `await` immediately returns a pending promise — the `throw` inside it rejects that promise rather than throwing synchronously. The surrounding `try/catch` only catches synchronous throws (or `await`ed rejections), so it never sees this one. `"request accepted"` logs right away, and the rejection later surfaces as an `unhandledrejection` event. The fix is to `await processOrder(order)` inside the `try`, or chain an explicit `.catch()`. **Debug**

</details>
::

::question-wrapper{language="javascript"}
### Q11. What happens if the request inside `fetchProfile` fails?

::code-wrapper{language="javascript"}
```javascript
function fetchProfile(id) {
  return api.get(`/users/${id}`)
    .catch(err => console.error("fetch failed:", err));
}

fetchProfile(42).then(profile => {
  console.log(profile.name);
});
```
::

- [ ] `profile` is `undefined` inside `.then`, causing a `TypeError` when accessing `profile.name`
- [ ] If the request fails, `.then` never runs
- [ ] The `.catch` rethrows automatically, so `.then` is skipped on failure
- [ ] `fetchProfile` returns a rejected promise on failure, which propagates correctly to the caller

<details>
<summary>Show Answer</summary>

**Answer:** A — `profile` is `undefined` inside `.then`, causing a `TypeError` when accessing `profile.name`

**Explanation:** A `.catch()` handler that only logs (and returns nothing) recovers the promise chain — its return value, `undefined`, becomes the resolved value of the promise `fetchProfile` returns. So on failure, `.then(profile => ...)` still runs, but `profile` is `undefined`, and `profile.name` throws a fresh `TypeError` far from the real cause. To propagate the failure instead of masking it, the `catch` handler must rethrow (or the caller shouldn't swallow it there at all). **Idiom**

</details>
::

::question-wrapper{language="javascript"}
### Q12. What does this log?

::code-wrapper{language="javascript"}
```javascript
async function loadUser(id) {
  try {
    return await db.query(id);
  } catch (err) {
    throw new Error("failed to load user", { cause: err });
  }
}

try {
  await loadUser(1);
} catch (e) {
  console.log(e.message);
  console.log(e.cause.message);
}
```
::

- [ ] Only the original database error's message is logged
- [ ] "failed to load user" then `undefined`, because `cause` is not a real `Error` property
- [ ] "failed to load user" then the original database error's message
- [ ] A `SyntaxError` because `Error` does not accept a second argument

<details>
<summary>Show Answer</summary>

**Answer:** C — "failed to load user" then the original database error's message

**Explanation:** The ES2022 `Error` `cause` option lets you construct a new, higher-level error while preserving the original one on `e.cause`, so downstream code (or a debugger) can inspect both the friendly outer message and the root technical cause without losing information the way plain re-wrapping does. B is wrong because `cause` has been a standard, widely-supported `Error` constructor option since ES2022. **Idiom**

</details>
::

::question-wrapper{language="javascript"}
### Q13. Is this valid JavaScript, and what does it demonstrate?

::code-wrapper{language="javascript"}
```javascript
try {
  JSON.parse(rawInput);
} catch (err) {
  if (err instanceof SyntaxError) {
    console.log("bad JSON");
  } else if (err instanceof TypeError) {
    console.log("bad input type");
  } else {
    throw err;
  }
}
```
::

- [ ] This code is invalid — JavaScript requires a separate `catch` clause per error type, like `catch (SyntaxError e)`
- [ ] JavaScript has only one `catch` clause per `try`; differentiating error types is done manually with `instanceof` checks inside it, as shown here
- [ ] `instanceof SyntaxError` will never be `true` because `JSON.parse` throws a plain `Error`
- [ ] The `else` branch causes an infinite loop by re-throwing inside a `catch`

<details>
<summary>Show Answer</summary>

**Answer:** B — JavaScript has only one `catch` clause per `try`; differentiating error types is done manually with `instanceof` checks inside it, as shown here

**Explanation:** Unlike languages such as Java or Python that support multiple typed `catch` clauses, JavaScript's `try` statement allows exactly one `catch` block. The idiomatic way to branch on error type is `instanceof` checks inside that single block, rethrowing (or handling generically) anything unrecognized. `JSON.parse` genuinely throws a `SyntaxError` on malformed JSON, so C is false, and A describes syntax that doesn't exist in JS.

</details>
::

::question-wrapper{language="javascript"}
### Q14. What is this syntax, and when is it appropriate?

::code-wrapper{language="javascript"}
```javascript
async function isPortAvailable(port) {
  try {
    await net.connect(port);
    return false;
  } catch {
    return true;
  }
}
```
::

- [ ] This is a syntax error — `catch` always requires a parenthesized parameter like `catch (err)`
- [ ] `catch` without a parameter silently ignores only `TypeError`s and lets everything else propagate
- [ ] This works, but `err` is still implicitly available as a global variable inside the block
- [ ] This is valid ES2019+ syntax called optional catch binding, used when the caught error's details aren't needed

<details>
<summary>Show Answer</summary>

**Answer:** D — This is valid ES2019+ syntax called optional catch binding, used when the caught error's details aren't needed

**Explanation:** Optional catch binding (`catch { ... }` with no `(err)`) has been valid syntax since ES2019 for exactly this situation — reacting to the fact that something failed without needing the error object itself. It's cleaner than `catch (_)` or an unused `catch (err) { ... }` parameter and avoids unused-variable lint warnings. **Idiom**

</details>
::

::question-wrapper
### Q15. Why are `window.onerror` and the `unhandledrejection` event described as "last resort" global error handlers rather than a replacement for local `try/catch`?

- [ ] By the time they fire, the error has already escaped its original context — you've lost the chance to retry, show inline UI feedback, or recover gracefully; they're best used for logging/reporting errors that were missed locally
- [ ] They only fire in browsers, not in Node.js, so relying on them breaks server-side code
- [ ] They automatically retry the failed operation, which is usually undesirable
- [ ] They can only catch errors thrown inside `try` blocks, not truly uncaught ones

<details>
<summary>Show Answer</summary>

**Answer:** A — By the time they fire, the error has already escaped its original context — you've lost the chance to retry, show inline UI feedback, or recover gracefully; they're best used for logging/reporting errors that were missed locally

**Explanation:** Global handlers are a safety net for observability, e.g. shipping errors to a monitoring service, but they run after the stack has already unwound, so there's no way to resume the original operation, show contextual UI, or apply targeted recovery logic. Meaningful recovery requires catching close to the failure. B is false (Node has its analogous `process.on('uncaughtException')`/`process.on('unhandledRejection')`); D is false — they exist specifically to catch what wasn't caught locally.

</details>
::

::question-wrapper{language="javascript"}
### Q16. What happens when `scheduleWork` runs?

::code-wrapper{language="javascript"}
```javascript
function scheduleWork() {
  try {
    setTimeout(() => {
      throw new Error("boom");
    }, 100);
  } catch (err) {
    console.log("caught:", err.message);
  }
}

scheduleWork();
```
::

- [ ] "caught: boom" is logged after 100ms
- [ ] The `try/catch` prevents `setTimeout` from ever calling the callback
- [ ] Nothing is logged by the `catch`; the error becomes an uncaught exception once the timer callback runs, in a later turn of the event loop
- [ ] A `TypeError` is thrown immediately because you cannot `throw` inside a `setTimeout` callback

<details>
<summary>Show Answer</summary>

**Answer:** C — Nothing is logged by the `catch`; the error becomes an uncaught exception once the timer callback runs, in a later turn of the event loop

**Explanation:** `try/catch` can only catch synchronous exceptions that occur while it's actively on the call stack. `setTimeout` schedules its callback to run later, in a completely separate event-loop turn — by the time it executes, the original `try/catch` has already finished and is off the stack, so it has no way to intercept the throw. The error surfaces as an uncaught exception (Node) or via `window.onerror` (browsers). The fix is to put the `try/catch` inside the callback itself. **Debug**

</details>
::

::question-wrapper{language="javascript"}
### Q17. If `save(item)` rejects for one item, what happens when `processAll` runs?

::code-wrapper{language="javascript"}
```javascript
async function processAll(items) {
  try {
    items.forEach(async (item) => {
      await save(item);
    });
    console.log("all done");
  } catch (err) {
    console.log("caught:", err.message);
  }
}
```
::

- [ ] "all done" logs immediately, and the rejection becomes an unhandled promise rejection, uncaught by the surrounding `try/catch`
- [ ] The `catch` block logs the rejection's error message
- [ ] `processAll` itself rejects, and the caller must add a `.catch()`
- [ ] `forEach` waits for all async callbacks to settle before moving to the next line

<details>
<summary>Show Answer</summary>

**Answer:** A — "all done" logs immediately, and the rejection becomes an unhandled promise rejection, uncaught by the surrounding `try/catch`

**Explanation:** `Array.prototype.forEach` does not await the promises returned by its (async) callback, nor does it propagate their rejections — it fires all callbacks and moves on immediately, so `"all done"` logs right away regardless of what `save` does. Any rejection inside one of those detached async callbacks becomes an unhandled promise rejection instead of surfacing in the surrounding `try/catch`. The idiomatic fix is a `for...of` loop with `await`, or `Promise.all(items.map(item => save(item)))` inside the `try`. **Performance** / **Debug**

</details>
::

::question-wrapper
### Q18. A function three layers deep in a call stack fails to write to a cache (a non-critical, recoverable failure), while a top-level function fails to authenticate a user (a critical failure the caller must react to). What's the best-practice approach to where each error is caught?

- [ ] Catch both errors at the very top of the application in a single global handler, to keep error-handling logic centralized
- [ ] Never catch errors inside deeply nested functions; always let every error bubble to the top
- [ ] Catch every error immediately where it's thrown and never rethrow, to avoid unhandled rejections
- [ ] Catch the cache-write failure close to where it happens (a fallback, like skipping the cache, is possible there), and let the authentication failure propagate up to a caller that can actually decide what to do, attaching context via `cause` or custom fields as it goes

<details>
<summary>Show Answer</summary>

**Answer:** D — Catch the cache-write failure close to where it happens, and let the authentication failure propagate up to a caller that can actually decide what to do, attaching context via `cause` or custom fields as it goes

**Explanation:** The right layer to catch an error is the layer that can do something meaningful about it. A recoverable, local failure (skip the cache, continue) should be handled where the fallback is known; a failure that requires a caller-level decision (redirect to login, retry, surface to the user) should propagate — ideally enriched with `cause` or extra fields at each hop — rather than being caught prematurely and silently discarded. A centralizes logic but loses recovery opportunities; C swallows everything, hiding real failures from callers who need to know. **Idiom**

</details>
::

::question-wrapper{language="javascript"}
### Q19. What does this log?

::code-wrapper{language="javascript"}
```javascript
function transferFunds(amount) {
  try {
    throw new Error("insufficient balance");
  } finally {
    validateAudit();
  }
}

function validateAudit() {
  throw new Error("audit log unavailable");
}

try {
  transferFunds(100);
} catch (err) {
  console.log(err.message);
}
```
::

- [ ] "audit log unavailable" is logged, and "insufficient balance" is discarded entirely
- [ ] "insufficient balance" is logged
- [ ] Both error messages are logged, joined together
- [ ] Nothing is logged because two errors in one call stack crash the process

<details>
<summary>Show Answer</summary>

**Answer:** A — "audit log unavailable" is logged, and "insufficient balance" is discarded entirely

**Explanation:** Just as an explicit `return` in `finally` overrides a pending return or throw, a new exception thrown inside `finally` also overrides whatever was propagating from `try`/`catch` — `"insufficient balance"` never reaches the outer `catch` because `finally`'s own `throw new Error("audit log unavailable")` replaces it as the statement's outcome. This is why code inside `finally` should be simple and defensive (or wrapped in its own `try/catch`) — an unrelated failure there can silently mask the real error. **Debug** / **Safety**

</details>
::

::question-wrapper{language="javascript"}
### Q20. Which statement best describes why this implementation follows good error-handling practice?

::code-wrapper{language="javascript"}
```javascript
async function loadSettings(raw) {
  try {
    const parsed = JSON.parse(raw);
    return await applyDefaults(parsed);
  } catch (err) {
    throw new Error("could not load settings", { cause: err });
  }
}
```
::

- [ ] It is flawed because `try/catch` cannot wrap both a synchronous call (`JSON.parse`) and an `await` expression in the same block
- [ ] It is flawed because throwing inside a `catch` block is not allowed in `async` functions
- [ ] It catches both the potential `SyntaxError` from `JSON.parse` and any rejection from `applyDefaults` in a single block, wraps them in a descriptive higher-level error, and preserves the original failure via `cause` — all without swallowing it
- [ ] It should use `catch {}` with no binding, since the original error is discarded anyway

<details>
<summary>Show Answer</summary>

**Answer:** C — It catches both the potential `SyntaxError` from `JSON.parse` and any rejection from `applyDefaults` in a single block, wraps them in a descriptive higher-level error, and preserves the original failure via `cause` — all without swallowing it

**Explanation:** A single `try` can legally mix synchronous statements and `await`ed expressions — either kind of failure lands in the same `catch`. Wrapping with a clearer message while attaching `{ cause: err }` gives callers a readable error and preserves the root cause for logs and debuggers, rather than either swallowing the original error or leaking a raw `SyntaxError`/rejection with no context. A and B describe restrictions that don't exist in JS; D is wrong because the original error is deliberately preserved via `cause`, not discarded. **Idiom**

</details>
::
