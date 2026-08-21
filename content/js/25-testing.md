# 25 — Testing

::question-wrapper{language="javascript"}
### Q1. What best characterizes the difference between unit, integration, and end-to-end (E2E) tests?

- [ ] Unit tests exercise the whole app through the UI, so they give the highest confidence but run slowest
- [ ] Unit tests verify a single function/module in isolation and run fastest; integration tests verify multiple units working together; end-to-end tests drive the real app like a user and give the highest confidence but are slowest and most brittle
- [ ] Integration tests are always slower than end-to-end tests, because they spin up a full browser
- [ ] End-to-end tests replace the need for unit tests once a project has enough of them

<details>
<summary>Show Answer</summary>

**Answer:** B — Unit tests verify a single function/module in isolation and run fastest; integration tests verify multiple units working together; end-to-end tests drive the real app like a user and give the highest confidence but are slowest and most brittle

**Explanation:** This is the classic testing pyramid tradeoff: isolation and speed decrease as confidence increases. Unit tests are cheap and fast because they touch no real collaborators; E2E tests are expensive and slow because they exercise the real browser, network, and database, but they catch integration bugs unit tests structurally cannot see. Option A reverses the definitions — unit tests never touch the UI. Option C gets the ordering backwards; E2E is typically the slowest tier because it pays for a real browser and real I/O, with integration tests in between. Option D is wrong because the pyramid recommends many unit tests, fewer integration tests, and the fewest E2E tests as complements, not replacements — E2E tests alone are too slow and flaky to run on every change.

</details>
::

::question-wrapper{language="javascript"}
### Q2. You have `processOrder(cart, paymentGateway)`, which totals a cart and charges a payment gateway. A test constructs an in-memory `InMemoryCart` and a `FakePaymentGateway`, calls `processOrder`, and asserts the resulting order's `status` is `"paid"`. What kind of test is this?

::code-wrapper{language="javascript"}
```javascript
test("processOrder marks the order as paid", async () => {
  const cart = new InMemoryCart([{ sku: "A1", price: 20 }]);
  const gateway = new FakePaymentGateway({ willSucceed: true });

  const order = await processOrder(cart, gateway);

  expect(order.status).toBe("paid");
});
```
::

- [ ] End-to-end test — it exercises the full application exactly the way a real user would, including the actual UI and a live payment gateway
- [ ] Unit test — because `processOrder` is a single function, calling it in a test always makes the test a unit test regardless of what it depends on
- [ ] Integration test — it exercises `processOrder` together with its collaborators (the cart and the payment gateway, even though the gateway is a fake), verifying that these pieces work correctly together rather than testing `processOrder` in full isolation or driving the app through a real UI
- [ ] Snapshot test — because the assertion checks the resulting `order` object's shape

<details>
<summary>Show Answer</summary>

**Answer:** C — Integration test — it exercises `processOrder` together with its collaborators (the cart and the payment gateway, even though the gateway is a fake), verifying that these pieces work correctly together rather than testing `processOrder` in full isolation or driving the app through a real UI

**Explanation:** What makes this an integration test is that the assertion depends on the correct interaction between multiple collaborating pieces (cart totaling, order state transitions, gateway charging), even though the gateway is faked rather than real. Option A is wrong because there's no browser, UI, or live network call involved — using fakes instead of the real UI is exactly what keeps this out of E2E territory. Option B is the common misconception that "one function call" automatically means "unit test"; what matters is whether the collaborators are isolated out (unit) or exercised together (integration), not how many function calls appear in the test body. Option D confuses this with snapshot testing, which serializes and diffs output rather than asserting on a specific field.

</details>
::

::question-wrapper
### Q3. Which statement accurately distinguishes a spy, a stub, and a mock?

- [ ] A spy wraps a real function, recording calls (arguments, call count) while still invoking the original implementation; a stub replaces a function with fixed, canned behavior; a mock is a stub that also has built-in expectations/assertions about how it should be called
- [ ] A stub wraps a real function and records how it was called while still invoking the original implementation
- [ ] A mock always requires a real network connection to verify calls
- [ ] Spies, stubs, and mocks are interchangeable terms with no meaningful difference between testing libraries

<details>
<summary>Show Answer</summary>

**Answer:** A — A spy wraps a real function, recording calls (arguments, call count) while still invoking the original implementation; a stub replaces a function with fixed, canned behavior; a mock is a stub that also has built-in expectations/assertions about how it should be called

**Explanation:** **Idiom:** these three terms describe increasing levels of replacement and assertion, and the distinction is universal test-double terminology, not specific to any one framework — Jest's `jest.fn()`/`jest.spyOn()` and Vitest's `vi.fn()`/`vi.spyOn()` are just two libraries' names for building spies, stubs, and mocks. A spy observes without necessarily replacing behavior; a stub replaces behavior with something canned so the test controls the return value; a mock goes further by encoding expectations about the calls themselves (e.g. "was called exactly once with these arguments") as part of the double. Option B swaps the spy/stub definitions. Option C and D are simply false — none of these concepts require real network access, and the distinctions materially affect how a test reads and what it actually proves.

</details>
::

::question-wrapper{language="javascript"}
### Q4. `jest.spyOn(console, "error")` is called with no `.mockImplementation(...)` chained after it. What happens when the spied method is invoked during the test?

::code-wrapper{language="javascript"}
```javascript
const spy = jest.spyOn(console, "error");

doSomethingThatLogsAnError();

expect(spy).toHaveBeenCalled();
```
::

- [ ] `console.error` is completely silenced; nothing is printed to the console
- [ ] `jest.spyOn` throws because `console.error` cannot be spied on
- [ ] The test fails immediately, because spies cannot wrap built-in global methods
- [ ] The spy records the call for the assertion, but `console.error` still executes its real implementation, so the error is still printed to the console

<details>
<summary>Show Answer</summary>

**Answer:** D — The spy records the call for the assertion, but `console.error` still executes its real implementation, so the error is still printed to the console

**Explanation:** **Debug:** a bare `jest.spyOn(obj, "method")` call, by default, calls through to the real implementation — it only adds tracking of calls, arguments, and return values on top. To also replace the behavior (e.g. to silence noisy `console.error` output in a test), you must explicitly chain `.mockImplementation(() => {})`. Option A is the mistake many people make: assuming spying automatically suppresses output, when in fact suppression requires an explicit mock implementation. Options B and C are false — `jest.spyOn` works on object methods including built-ins like `console`, as long as the property is configurable.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What is the problem with this test?

::code-wrapper{language="javascript"}
```javascript
test("fetchUser resolves with the right name", () => {
  fetchUser(1).then((user) => {
    expect(user.name).toBe("Ada");
  });
});
```
::

- [ ] This test always fails, because `fetchUser` returns a promise instead of a value
- [ ] This test can report as passing even if `user.name` is not `"Ada"`, because the test function returns `undefined` synchronously and the test runner considers the test done before the `.then()` callback — and its `expect` — ever runs
- [ ] This test throws a syntax error, because `.then()` cannot be used inside a `test()` callback
- [ ] This test correctly verifies the resolved value; the test runner automatically waits for any promise created inside a test

<details>
<summary>Show Answer</summary>

**Answer:** B — This test can report as passing even if `user.name` is not `"Ada"`, because the test function returns `undefined` synchronously and the test runner considers the test done before the `.then()` callback — and its `expect` — ever runs

**Explanation:** **Debug:** this is one of the most common silent-false-positive bugs in async testing. The test callback here is a plain synchronous function that returns `undefined` immediately; Jest and Vitest both mark a test complete as soon as its callback returns (unless it returns a promise or uses a `done` parameter). The `.then()` callback is scheduled as a microtask that runs *after* the synchronous test function has already returned, so if the `expect` inside it fails, that failure either gets reported as an unhandled rejection after the test has already "passed," or is missed entirely depending on runner configuration. The fix is to `return fetchUser(1).then(...)` or, more idiomatically, use `async () => { const user = await fetchUser(1); expect(user.name).toBe("Ada"); }`.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What is wrong with this test?

::code-wrapper{language="javascript"}
```javascript
test("all users are validated", async () => {
  const ids = [1, 2, 3];

  ids.forEach(async (id) => {
    const user = await fetchUser(id);
    expect(user.isValid).toBe(true);
  });
});
```
::

- [ ] `Array.prototype.forEach` does not await the async callbacks it's given, so the outer test function returns (and the test is marked done) before any of the `fetchUser` calls resolve or their `expect` calls actually run
- [ ] This is correct — `async` inside `forEach` makes the whole loop asynchronous, and `await` on the outer `test` waits for every iteration to finish before the test completes
- [ ] This throws a `TypeError`, because `forEach` cannot accept an async function
- [ ] This is equivalent to using `Promise.all` with `map`, so all assertions are guaranteed to run before the test finishes

<details>
<summary>Show Answer</summary>

**Answer:** A — `Array.prototype.forEach` does not await the async callbacks it's given, so the outer test function returns (and the test is marked done) before any of the `fetchUser` calls resolve or their `expect` calls actually run

**Explanation:** **Debug:** `forEach` invokes its callback for each element and completely ignores any value — including a promise — that the callback returns; it does not chain or await them. Each `async (id) => {...}` call kicks off a promise and `forEach` immediately moves to the next element without waiting, so the outer `test` function's `await`-free body returns right after the loop, and the test is considered finished before any `fetchUser` resolves. This makes every `expect` inside the loop a fire-and-forget check that may never be seen by the test runner. The fix is `for (const id of ids) { const user = await fetchUser(id); expect(user.isValid).toBe(true); }`, or `await Promise.all(ids.map(async (id) => { ... }))`. Option D specifically names the correct fix pattern but wrongly claims it describes the code as written.

</details>
::

::question-wrapper{language="javascript"}
### Q7. Given this function, which is the most correct way to test that it rejects for an invalid amount?

::code-wrapper{language="javascript"}
```javascript
async function chargeCard(amount) {
  if (amount <= 0) throw new Error("Invalid amount");
  return gateway.charge(amount);
}
```
::

- [ ] `expect(chargeCard(-5)).toThrow("Invalid amount");`
- [ ] `const result = chargeCard(-5); expect(result).toBe(undefined);`
- [ ] `chargeCard(-5).catch((e) => console.log(e));`
- [ ] `await expect(chargeCard(-5)).rejects.toThrow("Invalid amount");`

<details>
<summary>Show Answer</summary>

**Answer:** D — `await expect(chargeCard(-5)).rejects.toThrow("Invalid amount");`

**Explanation:** **Idiom:** `.rejects` unwraps the promise's rejection so the matcher can assert on the thrown error, and awaiting the whole `expect(...)` call ensures the test runner actually waits for that assertion to run before the test is marked complete. (`rejects`/`resolves` are Jest and Vitest matcher-naming conventions specifically — other runners use different syntax, but the underlying need to both await and assert on the rejection is universal.) Option A is wrong because `toThrow` expects a synchronously-throwing function, not a promise — an async function's `throw` becomes a rejection, not a synchronous exception, so this assertion doesn't correctly observe it. Option B never awaits anything and doesn't test failure at all. Option C swallows the rejection with a `console.log` and asserts nothing — if `chargeCard` unexpectedly resolved instead of rejecting, this "test" would still report as passing.

</details>
::

::question-wrapper{language="javascript"}
### Q8. What is the risk in this test?

::code-wrapper{language="javascript"}
```javascript
test("chargeCard rejects for invalid amount", async () => {
  try {
    await chargeCard(-5);
  } catch (err) {
    expect(err.message).toBe("Invalid amount");
  }
});
```
::

- [ ] This is the recommended pattern for testing rejections; it is equivalent in safety to `rejects.toThrow`
- [ ] This throws a syntax error, because `expect` cannot be called inside a `catch` block
- [ ] This test can give a false positive: if `chargeCard(-5)` unexpectedly resolves instead of rejecting, the `catch` block — and its `expect` — never runs, and the test passes having executed zero assertions
- [ ] This test always fails, because `try`/`catch` cannot be combined with `async`/`await`

<details>
<summary>Show Answer</summary>

**Answer:** C — This test can give a false positive: if `chargeCard(-5)` unexpectedly resolves instead of rejecting, the `catch` block — and its `expect` — never runs, and the test passes having executed zero assertions

**Explanation:** **Debug:** this is the same silent-false-positive family as the missing-`await`/`.then()` bug, just wearing a `try`/`catch` disguise. If a future change accidentally makes `chargeCard` resolve for negative amounts instead of throwing, the `catch` block simply never executes, no `expect` ever runs, and the test still reports green — precisely when it should be failing loudest. The safer pattern is `await expect(chargeCard(-5)).rejects.toThrow("Invalid amount")`, where the absence of a rejection itself causes the assertion to fail; alternatively, guard the try/catch version with `expect.assertions(1)` at the top of the test so the runner fails the test if the expected number of assertions never ran.

</details>
::

::question-wrapper{language="javascript"}
### Q9. What tradeoff does snapshot testing introduce?

::code-wrapper{language="javascript"}
```javascript
test("renders the user card", () => {
  const tree = renderer.create(<UserCard name="Ada" />).toJSON();
  expect(tree).toMatchSnapshot();
});
```
::

- [ ] Snapshot tests replace the need for any other assertions, because they capture the entire output automatically
- [ ] Snapshot tests are useful for catching unintended changes to output, but they carry a real risk: when a snapshot fails, it's easy to reflexively run the "update snapshot" command without actually reviewing the diff, which can silently bake a real regression into the new snapshot as the new "expected" output
- [ ] Snapshots can only be used with React components, never with plain JSON or string output
- [ ] A snapshot test fails only if the component throws an error during render

<details>
<summary>Show Answer</summary>

**Answer:** B — Snapshot tests are useful for catching unintended changes to output, but they carry a real risk: when a snapshot fails, it's easy to reflexively run the "update snapshot" command without actually reviewing the diff, which can silently bake a real regression into the new snapshot as the new "expected" output

**Explanation:** Snapshot testing is good at flagging *that* output changed, but says nothing about whether the change is correct — that judgment call is left entirely to whoever reviews the failing diff. In practice, a failing snapshot in a busy CI run often gets "fixed" by blindly re-running the updater, which overwrites the stored snapshot with whatever the (possibly buggy) code now produces, turning a real regression into the new passing baseline. Option A overstates what snapshots verify — they detect *change*, not *correctness*. Option C is false; `toMatchSnapshot` works on any serializable value, including plain objects and strings. Option D is false; a snapshot test fails whenever the serialized output differs from the stored snapshot, render errors aside.

</details>
::

::question-wrapper
### Q10. Following on from snapshot testing: what is the correct discipline for handling a failing snapshot?

- [ ] Snapshots should be reviewed like a code diff on every failure — treating an unreviewed "update snapshot" run as automatically correct defeats the purpose of the test, since it just captures whatever the code currently outputs as the new "truth"
- [ ] It is always safe to run the snapshot-update command in CI on every failing build, to keep the suite green
- [ ] Snapshots should be committed as binary files, so they cannot be diffed or reviewed at all
- [ ] A growing snapshot file is a sign the test suite has too much coverage, and snapshots should be deleted regularly

<details>
<summary>Show Answer</summary>

**Answer:** A — Snapshots should be reviewed like a code diff on every failure — treating an unreviewed "update snapshot" run as automatically correct defeats the purpose of the test, since it just captures whatever the code currently outputs as the new "truth"

**Explanation:** A snapshot is only as trustworthy as the review it received when it was captured or updated; the entire value of the test collapses if updates are applied reflexively rather than read as a diff. Option B describes the exact anti-pattern that turns a regression detector into rubber-stamping machinery — automatically updating snapshots in CI removes the human review step that gives the test any signal at all. Option C is counterproductive; snapshots are stored as plain, diffable text (e.g. `.snap` files) specifically so they can be reviewed in pull requests. Option D confuses volume with quality — a large snapshot file isn't inherently a problem, but unreviewed updates are.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What is the problem with this pair of tests?

::code-wrapper{language="javascript"}
```javascript
let cache = [];

function addItem(item) {
  cache.push(item);
  return cache.length;
}

test("adds first item", () => {
  expect(addItem("a")).toBe(1);
});

test("adds second item", () => {
  expect(addItem("b")).toBe(1);
});
```
::

- [ ] Both tests pass reliably regardless of run order, because the test runner resets all module-level variables before each test
- [ ] This code cannot run at all, because top-level `let` declarations are forbidden in test files
- [ ] The second test always fails with a `TypeError`, because `cache` is undefined inside `addItem`
- [ ] The second test is order-dependent and will fail (or pass) unpredictably depending on execution order, because `cache` is module-level shared state that isn't reset between tests — a classic source of flaky, order-dependent failures

<details>
<summary>Show Answer</summary>

**Answer:** D — The second test is order-dependent and will fail (or pass) unpredictably depending on execution order, because `cache` is module-level shared state that isn't reset between tests — a classic source of flaky, order-dependent failures

**Explanation:** Test isolation means each test should start from a known, independent state. Here, `cache` lives outside any test and outside any setup hook, so it accumulates across the whole file: after the first test runs, `cache.length` is already `1`, so the second test's `addItem("b")` returns `2`, not `1`, and the assertion fails — but only if the tests run in that order. If a test runner or `--shuffle` flag reorders tests, or a test is run in isolation with `.only`, the outcome flips unpredictably. The fix is to reset `cache = []` in a `beforeEach`, or avoid module-level mutable state entirely by scoping it inside each test. Option A describes behavior no mainstream test runner provides by default — module state persists across tests in the same file unless explicitly reset.

</details>
::

::question-wrapper{language="javascript"}
### Q12. What do `beforeEach` and `afterEach` accomplish in this test file?

::code-wrapper{language="javascript"}
```javascript
describe("UserRepository", () => {
  let repo;

  beforeEach(() => {
    repo = new UserRepository();
  });

  afterEach(() => {
    repo.close();
  });

  test("starts empty", () => {
    expect(repo.count()).toBe(0);
  });
});
```
::

- [ ] `beforeEach`/`afterEach` run once total for the whole `describe` block, before/after all its tests combined
- [ ] `afterEach` runs before `beforeEach` on every test, so `repo` is closed before it's even created
- [ ] `beforeEach` creates a fresh `repo` before every test in the block and `afterEach` tears it down after every test, giving each test an isolated, known starting state instead of leaking state between tests
- [ ] Because `repo` is declared with `let` outside the hooks, every test shares the exact same `UserRepository` instance

<details>
<summary>Show Answer</summary>

**Answer:** C — `beforeEach` creates a fresh `repo` before every test in the block and `afterEach` tears it down after every test, giving each test an isolated, known starting state instead of leaking state between tests

**Explanation:** `beforeEach`/`afterEach` run around *every individual* `test`/`it` in their scope, not once for the whole `describe` block — that's what `beforeAll`/`afterAll` do instead, which is the mistake option A describes. Each test therefore gets a brand-new `UserRepository` assigned to the shared `repo` variable, and the previous instance is closed afterward, preventing the kind of cross-test state leakage seen in Q11. Option B reverses the actual execution order — setup always runs before the test, teardown after. Option D is wrong precisely because `beforeEach` *reassigns* `repo` to a new instance before each test, so the reference changes even though the variable binding itself is declared once.

</details>
::

::question-wrapper{language="javascript"}
### Q13. `save` is a mock with a canned return value. The first test passes; the second fails because `save` has already been called once by the time it runs. Which fix resets the call count between tests *without* discarding the `mockReturnValue(true)` behavior?

::code-wrapper{language="javascript"}
```javascript
const save = jest.fn().mockReturnValue(true);

test("first call", () => {
  save("a");
  expect(save).toHaveBeenCalledTimes(1);
});

test("second call", () => {
  save("b");
  expect(save).toHaveBeenCalledTimes(1);
});
```
::

- [ ] Add `jest.restoreAllMocks()` in an `afterEach` — it only affects mocks created with `jest.spyOn`, restoring their original (non-mocked) implementation, so it wouldn't touch a plain `jest.fn()` like `save`
- [ ] Add `jest.clearAllMocks()` in an `afterEach` — it resets `mock.calls`/`mock.instances` (so the recorded call count starts fresh before each test) but leaves the configured `mockReturnValue(true)` implementation intact
- [ ] Add `jest.resetAllMocks()` in an `afterEach` — it clears call history AND removes any configured `mockReturnValue`/`mockImplementation`, resetting `save` to a plain mock that returns `undefined`, so `save("b")` would no longer return `true`
- [ ] Do nothing — the test runner automatically clears mock call counts between tests by default in every configuration

<details>
<summary>Show Answer</summary>

**Answer:** B — Add `jest.clearAllMocks()` in an `afterEach` — it resets `mock.calls`/`mock.instances` (so the recorded call count starts fresh before each test) but leaves the configured `mockReturnValue(true)` implementation intact

**Explanation:** **Idiom:** `mockClear()` (and its bulk form `jest.clearAllMocks()`) only wipes recorded call history — arguments, call count, results — leaving any configured implementation untouched, which is exactly what's needed here. `mockReset()`/`jest.resetAllMocks()` goes further and also strips the mocked implementation, which is why option C's `save` would start returning `undefined`, breaking any test relying on `true`. `mockRestore()`/`jest.restoreAllMocks()` goes furthest, restoring the *original* un-mocked implementation, but that only applies to mocks created via `jest.spyOn` on a real method — a bare `jest.fn()` has no "original" to restore to, so option A wouldn't fix anything. (`clearAllMocks`/`resetAllMocks`/`restoreAllMocks` are Jest's specific method names; Vitest mirrors them as `vi.clearAllMocks()`/`vi.resetAllMocks()`/`vi.restoreAllMocks()` — the underlying three-tier distinction is the part worth remembering.) Option D is false; leftover mock state across tests is a common, real cause of flakiness, not something runners fix automatically.

</details>
::

::question-wrapper{language="javascript"}
### Q14. Why does this test complete instantly instead of taking 5 real seconds?

::code-wrapper{language="javascript"}
```javascript
function scheduleReminder(cb) {
  setTimeout(cb, 5000);
}

test("calls the callback after 5 seconds", () => {
  jest.useFakeTimers();
  const cb = jest.fn();
  scheduleReminder(cb);

  jest.advanceTimersByTime(5000);

  expect(cb).toHaveBeenCalledTimes(1);
});
```
::

- [ ] `jest.useFakeTimers()` replaces the global timer functions with mock versions; `jest.advanceTimersByTime(5000)` then synchronously fast-forwards the mocked clock and fires any callbacks scheduled at or before that point — no real waiting occurs, so the test runs instantly
- [ ] This test actually waits 5 real seconds; `advanceTimersByTime` only affects `Date.now()`, not `setTimeout`
- [ ] `jest.useFakeTimers()` only works with `setInterval`, not `setTimeout`, so `cb` is never called
- [ ] `advanceTimersByTime(5000)` schedules the callback to run 5000 real milliseconds from now, instead of running it immediately

<details>
<summary>Show Answer</summary>

**Answer:** A — `jest.useFakeTimers()` replaces the global timer functions with mock versions; `jest.advanceTimersByTime(5000)` then synchronously fast-forwards the mocked clock and fires any callbacks scheduled at or before that point — no real waiting occurs, so the test runs instantly

**Explanation:** Fake timers swap out `setTimeout`/`setInterval`/`Date` (depending on configuration) with mock implementations that track scheduled callbacks against a fake internal clock instead of the real system clock. `advanceTimersByTime(ms)` moves that fake clock forward by `ms` and synchronously invokes any callbacks whose delay has now elapsed, letting time-based code be tested in milliseconds of real wall-clock time instead of actually waiting. (`jest.useFakeTimers()` is Jest's API name; Vitest's equivalent is `vi.useFakeTimers()` — the fake-clock concept itself is universal.) Options B and C misdescribe what fake timers intercept — they replace the timer functions themselves, working for both `setTimeout` and `setInterval`. Option D inverts what "advance" means; it moves the clock forward through the scheduled point, not away from it.

</details>
::

::question-wrapper{language="javascript"}
### Q15. This test combines fake timers with a real `async`/`await` chain. What subtlety can trip it up?

::code-wrapper{language="javascript"}
```javascript
async function delayedFetch(id) {
  await new Promise((resolve) => setTimeout(resolve, 1000));
  return fetch(`/users/${id}`);
}

test("resolves after the delay", async () => {
  jest.useFakeTimers();
  const promise = delayedFetch(1);

  jest.advanceTimersByTime(1000);

  await expect(promise).resolves.toBeDefined();
});
```
::

- [ ] This is completely safe — `advanceTimersByTime` always resolves every pending promise in the same synchronous call, so no extra step is ever needed
- [ ] `jest.useFakeTimers()` disables `async`/`await` entirely, so this test throws a `SyntaxError`
- [ ] `fetch` cannot be called inside a function that uses `setTimeout`, so this test always throws a `ReferenceError` regardless of timers
- [ ] Advancing fake timers fires the timer callback synchronously, but resuming the `await` inside `delayedFetch` still requires a separate microtask tick — advancing timers doesn't automatically flush pending microtasks from real Promises, so mixing fake timers with async code can need an extra `await Promise.resolve()` (or an async-aware timer-advance helper) before the promise actually settles

<details>
<summary>Show Answer</summary>

**Answer:** D — Advancing fake timers fires the timer callback synchronously, but resuming the `await` inside `delayedFetch` still requires a separate microtask tick — advancing timers doesn't automatically flush pending microtasks from real Promises, so mixing fake timers with async code can need an extra `await Promise.resolve()` (or an async-aware timer-advance helper) before the promise actually settles

**Explanation:** **Debug:** `advanceTimersByTime` operates on the macrotask/timer queue — it synchronously invokes the `setTimeout` callback (which calls `resolve()`), but resuming execution after an `await` happens on the microtask queue, on the next tick. If the mock-timer implementation doesn't also flush pending microtasks, the outer `await expect(promise).resolves...` can end up racing the still-pending microtask, leading to flaky or hanging assertions in some setups. Modern Jest/Vitest offer async-aware helpers (e.g. `jest.advanceTimersByTimeAsync` / `vi.advanceTimersByTimeAsync`, or awaiting a `Promise.resolve()`/`flushPromises()` tick) specifically to bridge this gap. Options B and C describe failures that don't occur — fake timers don't disable async/await or forbid calling other functions inside a timer-based one; option A overstates the guarantee and is the assumption that causes real intermittent failures in test suites mixing fake timers with real promise chains.

</details>
::

::question-wrapper{language="javascript"}
### Q16. What is wrong with testing this function by asserting on a hardcoded expected output?

::code-wrapper{language="javascript"}
```javascript
function generateOrderId() {
  return `ORD-${Date.now()}-${Math.floor(Math.random() * 1000)}`;
}

test("generateOrderId returns a predictable id", () => {
  expect(generateOrderId()).toBe("ORD-1700000000000-42");
});
```
::

- [ ] This test is fine, because the test runner automatically freezes `Date.now()` and seeds `Math.random()` for every test run
- [ ] `Math.random()` always returns the same value within a single test file, so only `Date.now()` needs mocking
- [ ] This test is inherently flaky, because `Date.now()` and `Math.random()` produce different real values on every run; to make it deterministic you need to mock them (e.g. `jest.spyOn(Date, "now").mockReturnValue(...)` and mock `Math.random`) or inject a clock/RNG dependency so the test controls their output
- [ ] This will pass consistently as long as the test runs on the same machine, because `Date.now()` is deterministic per machine

<details>
<summary>Show Answer</summary>

**Answer:** C — This test is inherently flaky, because `Date.now()` and `Math.random()` produce different real values on every run; to make it deterministic you need to mock them (e.g. `jest.spyOn(Date, "now").mockReturnValue(...)` and mock `Math.random`) or inject a clock/RNG dependency so the test controls their output

**Explanation:** Any function whose output depends on wall-clock time or randomness cannot be asserted against a fixed literal — the exact same code will produce a different string on every single run, so this test is guaranteed to fail almost immediately after being written (or on any machine other than the one that generated the literal). The fix is either to mock the nondeterministic sources directly (`jest.spyOn(Date, "now").mockReturnValue(1700000000000)`, `jest.spyOn(Math, "random").mockReturnValue(0.042)`), or better, to inject a clock and RNG as parameters/dependencies so the test can supply deterministic fakes without needing to patch globals. Options A, B, and D each invent a guarantee that doesn't exist in JavaScript or any mainstream test runner.

</details>
::

::question-wrapper
### Q17. A function under test has five collaborators, and the test mocks all five, then asserts only that each mock was called with the expected arguments in the expected order. What risk does this pattern introduce?

- [ ] Mocking more dependencies always makes a test suite more reliable, since fewer real code paths can fail
- [ ] Mocking so many of a function's collaborators that the test only verifies the mocks were called in the expected sequence — rather than exercising any real logic — can create false confidence: the test passes even if the real integration between those pieces is broken, because none of the real code paths actually ran
- [ ] Over-mocking is only a concern in end-to-end tests, never in unit tests
- [ ] A test with heavy mocking runs slower than one with no mocks, because mock setup always requires real I/O

<details>
<summary>Show Answer</summary>

**Answer:** B — Mocking so many of a function's collaborators that the test only verifies the mocks were called in the expected sequence — rather than exercising any real logic — can create false confidence: the test passes even if the real integration between those pieces is broken, because none of the real code paths actually ran

**Explanation:** When every collaborator is replaced with a mock, the "system under test" shrinks down to just the glue code wiring those mocks together — the test proves that glue code calls things in the right order, but proves nothing about whether the real implementations actually work together correctly. This is a genuine, common trap: a suite can be full of green tests like this while a refactor that changes real interaction contracts (argument shapes, error handling) sails through undetected, because nothing in the test exercised real behavior. Option A inverts the actual risk — more mocking generally means less real code is verified, not more reliability. Option C is false; over-mocking is very much a unit-test problem, arguably its most common failure mode. Option D confuses mock setup cost with I/O cost; mocks specifically avoid real I/O and are typically *faster*, not slower.

</details>
::

::question-wrapper{language="javascript"}
### Q18. This test achieves 100% line coverage of `divide`. What is still missing?

::code-wrapper{language="javascript"}
```javascript
function divide(a, b) {
  if (b === 0) throw new Error("Cannot divide by zero");
  return a / b;
}

test("divide runs without crashing", () => {
  divide(10, 2);
});
```
::

- [ ] This test can contribute to 100% line coverage for `divide` while asserting nothing about its behavior — it never checks that the returned value is actually `5`, so a bug that made `divide` always return `0` (or `NaN`) would still leave this test green. High coverage measures which lines executed, not whether the output was verified
- [ ] This test is equivalent to `expect(divide(10, 2)).toBe(5)`, because the test runner automatically asserts on the return value of the last expression in a test
- [ ] This test fails automatically, because a `test()` block must contain at least one `expect()` call
- [ ] Line coverage tools report this line as uncovered because no `expect()` was used, so the false sense of confidence described doesn't apply here

<details>
<summary>Show Answer</summary>

**Answer:** A — This test can contribute to 100% line coverage for `divide` while asserting nothing about its behavior — it never checks that the returned value is actually `5`, so a bug that made `divide` always return `0` (or `NaN`) would still leave this test green. High coverage measures which lines executed, not whether the output was verified

**Explanation:** Line/branch coverage is purely a measure of which code *ran* during the test suite, not whether the test made any meaningful claim about correctness. Calling `divide(10, 2)` executes the `return a / b` line, satisfying coverage tooling, but with zero `expect()` calls the test can never fail no matter how wrong the computed value is — coverage percentage and correctness confidence are two different axes that can diverge sharply. Option B invents a behavior no mainstream test runner has; nothing is asserted automatically. Option C is false — most runners do not require an `expect()` per test by default (some can be configured with `expect.assertions()` to enforce it, but that's opt-in, not automatic). Option D is wrong for the reason stated in the correct answer: the line *does* count as covered by execution alone, regardless of whether it was asserted on.

</details>
::

::question-wrapper
### Q19. A test suite occasionally fails because a test makes a real HTTP call to a third-party API, which is sometimes slow or briefly unavailable. What is the best-practice fix?

- [ ] Add a retry mechanism so the test simply reruns until it passes
- [ ] Keep the real network call, since network-dependent tests are considered best practice — they prove the code works against the real production API
- [ ] Increase the test timeout, since that addresses the root cause of network-related flakiness
- [ ] A test that calls a real external API introduces flakiness from network latency, rate limits, and outages that have nothing to do with whether the code under test is correct; the best practice is to isolate the unit by mocking the network layer (or using a recorded fixture/fake server) so the test's pass/fail depends only on the code being tested

<details>
<summary>Show Answer</summary>

**Answer:** D — A test that calls a real external API introduces flakiness from network latency, rate limits, and outages that have nothing to do with whether the code under test is correct; the best practice is to isolate the unit by mocking the network layer (or using a recorded fixture/fake server) so the test's pass/fail depends only on the code being tested

**Explanation:** A test's job is to give a reliable, repeatable signal about the code under test; anything that ties its outcome to an external system's availability, latency, or rate limits undermines that signal, because the test can now fail for reasons that have nothing to do with a real bug. The standard fix is to isolate the unit from the network entirely — mocking the HTTP client, stubbing the fetch call, or replaying a recorded fixture — so the test exercises the code's logic deterministically, with a separate (typically smaller, explicitly-labeled) suite of true integration/E2E tests reserved for verifying the real network integration occasionally. Option A papers over the symptom without fixing the underlying nondeterminism, and can hide a real regression behind eventual retries. Option B mistakes an E2E concern for a unit-test virtue — real-API calls belong in a deliberately separate, smaller test tier, not scattered through the main suite. Option C doesn't address rate limits or outright outages, and just makes a slow, flaky test slower.

</details>
::

::question-wrapper{language="javascript"}
### Q20. Two tests check the same `increment` behavior on a `Counter` class that uses a real `#count` private field. Which approach is the better practice, and why?

::code-wrapper{language="javascript"}
```javascript
class Counter {
  #count = 0;
  increment() {
    this.#count += 1;
    return this.#count;
  }
}

test("increment increases the count (implementation-coupled)", () => {
  const c = new Counter();
  c.increment();
  expect(c["#count"]).toBe(1);
});

test("increment increases the count (behavior-based)", () => {
  const c = new Counter();
  expect(c.increment()).toBe(1);
});
```
::

- [ ] Both tests are equally good, because they both call `expect()` on the counter's state
- [ ] The first test is preferred, because it verifies the internal field directly, which is more precise than checking the return value
- [ ] The second test is the better practice: it asserts on the public, observable output (the return value) that consumers actually depend on, so refactoring `Counter`'s internals won't break it. The first test reaches for a private field by bracket-string key, which doesn't actually work with real `#`-syntax private fields and couples the test to implementation details that can change independently of behavior
- [ ] Neither test is valid, because private `#` fields cannot be tested at all, even indirectly

<details>
<summary>Show Answer</summary>

**Answer:** C — The second test is the better practice: it asserts on the public, observable output (the return value) that consumers actually depend on, so refactoring `Counter`'s internals won't break it. The first test reaches for a private field by bracket-string key, which doesn't actually work with real `#`-syntax private fields and couples the test to implementation details that can change independently of behavior

**Explanation:** Good tests verify a contract — given these inputs, what output or observable behavior should result — rather than reaching into how that output is produced internally. The second test does exactly that: it only relies on `increment()`'s public return value, so if `Counter` is later refactored to store its count differently, the test keeps passing as long as the behavior is unchanged. The first test tries to peek at `#count` via `c["#count"]`, which is not how JavaScript's true private class fields (`#`-prefixed) work at all — they are not accessible via bracket/string property access from outside the class, so this line either reads `undefined` or throws, depending on engine and strict-mode context, making the test broken independent of the design critique. Even a syntactically-valid version of implementation-coupled testing is still the worse practice, because it makes tests brittle against safe refactors that don't change behavior.

</details>
::
