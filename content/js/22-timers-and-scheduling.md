# 22 — Timers & Scheduling

::question-wrapper
### Q1. What does the second argument to `setTimeout(fn, delay)` actually guarantee?

- [ ] `fn` will run exactly `delay` milliseconds later, to the millisecond
- [ ] `fn` will run no earlier than `delay` milliseconds later — the actual delay may be longer
- [ ] `fn` will run before `delay` milliseconds have passed
- [ ] `delay` is ignored in all modern browsers

<details>
<summary>Show Answer</summary>

**Answer:** B — `fn` will run no earlier than `delay` milliseconds later — the actual delay may be longer

**Explanation:** `delay` is a *minimum* wait time, not a scheduling guarantee. The callback is only queued as a macrotask once `delay` elapses; it still has to wait for the call stack to be empty and for any tasks already ahead of it in the queue to finish, so a busy main thread (long synchronous work, other pending callbacks) pushes the actual execution later. Option A is the common misconception that timers behave like a real-time scheduler. Option C is backwards. Option D is false — the delay still matters as a floor.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What happens when you call `setTimeout(fn, 0)`?

::code-wrapper{language="javascript"}
```javascript
console.log("1");
setTimeout(() => console.log("2"), 0);
console.log("3");
```
::

- [ ] Logs `1, 2, 3` — the timeout runs immediately since delay is 0
- [ ] Logs `1, 3, 2` — `fn` is still deferred to a macrotask that runs after the current synchronous code finishes
- [ ] Throws an error, since 0 is not a valid delay
- [ ] Logs `2, 1, 3` — zero-delay timers jump the queue

<details>
<summary>Show Answer</summary>

**Answer:** B — Logs `1, 3, 2` — `fn` is still deferred to a macrotask that runs after the current synchronous code finishes

**Explanation:** Even with a `0`ms delay, `setTimeout` always schedules its callback as a macrotask for a *future* iteration of the event loop — it never runs synchronously inline, no matter how small the delay. The current call stack (`console.log("1")` then `console.log("3")`) always finishes executing first. Option A is the classic beginner assumption that `0` means "right now." This also means a zero-delay timer runs *after* any pending microtasks (like resolved Promise `.then` callbacks), which is a related, frequently-tested ordering gotcha.

</details>
::

::question-wrapper
### Q3. What does `clearTimeout(id)` do if `id` refers to a timer that has already fired?

- [ ] It throws a `ReferenceError`
- [ ] It does nothing — calling it on an already-fired (or invalid/nonexistent) id is a silent no-op
- [ ] It re-schedules the callback to run again
- [ ] It throws a `TypeError` because the timer is no longer active

<details>
<summary>Show Answer</summary>

**Answer:** B — It does nothing — calling it on an already-fired (or invalid/nonexistent) id is a silent no-op

**Explanation:** `clearTimeout`/`clearInterval` are deliberately forgiving: passing an id that already fired, was already cleared, or was never valid at all simply does nothing and never throws. This makes it safe to call `clearTimeout` defensively (e.g., in cleanup code that isn't sure whether the timer already ran) without needing to guard it in a `try/catch`. Options A, C, and D invent error/re-trigger behavior that the spec explicitly avoids for ergonomic reasons.

</details>
::

::question-wrapper
### Q4. What is the defining behavior of `setInterval(fn, delay)` compared to a single `setTimeout`?

- [ ] It runs `fn` once after `delay`, then automatically clears itself
- [ ] It repeatedly schedules `fn` to run roughly every `delay` milliseconds until explicitly cleared with `clearInterval`
- [ ] It runs `fn` synchronously and blocks the thread until cleared
- [ ] It only works for functions with no arguments

<details>
<summary>Show Answer</summary>

**Answer:** B — It repeatedly schedules `fn` to run roughly every `delay` milliseconds until explicitly cleared with `clearInterval`

**Explanation:** `setInterval` is the repeating counterpart to `setTimeout`: after the initial `delay`, it keeps re-queuing `fn` at that interval indefinitely, and only `clearInterval(id)` stops it — it will otherwise keep firing for the lifetime of the page. Option A describes `setTimeout`'s one-shot behavior instead. Option C is false since all timer callbacks run asynchronously on the event loop, never blocking synchronously. Option D is fabricated — you can pass extra arguments to both `setTimeout` and `setInterval` as trailing parameters after the delay.

</details>
::

::question-wrapper
### Q5. Why is `requestAnimationFrame` (rAF) preferred over `setTimeout`/`setInterval` for JavaScript-driven animations?

- [ ] rAF runs animations on a separate thread, avoiding the main thread entirely
- [ ] rAF schedules the callback to run right before the browser's next repaint, syncing animation updates to the display's actual refresh rate
- [ ] rAF guarantees exactly 60 callbacks per second on every device
- [ ] rAF has no minimum delay, so it always fires faster than `setTimeout(fn, 0)`

<details>
<summary>Show Answer</summary>

**Answer:** B — rAF schedules the callback to run right before the browser's next repaint, syncing animation updates to the display's actual refresh rate

**Explanation:** **Performance:** `requestAnimationFrame` ties its callback to the browser's paint cycle, so updates happen exactly once per frame with no wasted work (no updates when the tab isn't visible) and no risk of scheduling a visual update at a moment the browser is about to throw it away before the next paint. `setTimeout`/`setInterval` have no awareness of paint timing, so an interval like `16.67ms` can easily drift out of sync with the actual refresh rate, causing jank. Option C is wrong because refresh rate varies (90Hz, 120Hz, etc.) and rAF adapts to the actual display, not a fixed 60. Option A is false — rAF still runs on the main thread.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What does this code log, and why?

::code-wrapper{language="javascript"}
```javascript
function tick(n) {
  console.log(n);
  if (n < 3) setTimeout(() => tick(n + 1), 0);
}
tick(0);
```
::

- [ ] `0, 1, 2, 3`, each firing essentially instantly since delay is 0
- [ ] `0, 1, 2, 3`, but after 5+ levels of nesting the HTML spec clamps timeouts to a minimum of ~4ms — irrelevant here since nesting is shallow
- [ ] Only `0` is logged; the recursive `setTimeout` calls never fire
- [ ] It causes a stack overflow due to unbounded recursion

<details>
<summary>Show Answer</summary>

**Answer:** B — `0, 1, 2, 3`, but after 5+ levels of nesting the HTML spec clamps timeouts to a minimum of ~4ms — irrelevant here since nesting is shallow

**Explanation:** Each call logs its number, then schedules the next tick as a macrotask, so all four values print in order across separate event-loop turns. The clamping rule (relevant background, not the trigger here) is that once `setTimeout` calls are nested more than 5 levels deep, browsers enforce a minimum delay of about 4ms regardless of the requested delay, specifically to prevent runaway zero-delay recursive loops from starving the event loop. With only 4 total levels of nesting, this clamp never actually kicks in for this snippet, which is why A is tempting but imprecise — it's not "no clamp exists," it's that the threshold isn't reached. Option D is wrong because each recursive call happens in a fresh, unwound stack (a new macrotask), not synchronous recursion, so there's no stack growth.

</details>
::

::question-wrapper
### Q7. What is the core conceptual difference between debouncing and throttling a function?

- [ ] They are the same technique with different names
- [ ] Debounce delays execution until a pause in calls; throttle guarantees execution at most once per fixed time window, even during continuous calls
- [ ] Debounce runs the function on a background thread; throttle runs it on the main thread
- [ ] Throttle only works with `setInterval`; debounce only works with `setTimeout`

<details>
<summary>Show Answer</summary>

**Answer:** B — Debounce delays execution until a pause in calls; throttle guarantees execution at most once per fixed time window, even during continuous calls

**Explanation:** Debounce resets a timer on every call and only fires once the calls *stop* for the configured wait period — ideal for "wait until the user stops typing." Throttle instead enforces a rate ceiling, letting the function run at regular intervals *while* calls keep coming — ideal for "update at most every 200ms while scrolling." They solve related but distinct problems, so treating them as interchangeable (option A) is a common but incorrect simplification that leads to picking the wrong one for a given UI interaction.

</details>
::

::question-wrapper{language="javascript"}
### Q8. A naive polling interval assumes each tick fires exactly on schedule:

::code-wrapper{language="javascript"}
```javascript
setInterval(() => {
  const start = Date.now();
  doExpensiveWork();
  console.log(Date.now() - start);
}, 100);
```
::

If `doExpensiveWork()` occasionally takes 150ms, what is the practical consequence of using `setInterval` here?

- [ ] `setInterval` automatically skips the next tick to compensate, keeping long-run timing perfectly accurate
- [ ] Ticks can effectively overlap or fire back-to-back with no gap once the callback finally returns, since the browser only queues the *next* invocation and doesn't wait for slow callbacks to "catch up" gracefully — leading to drift and, if the callback consistently runs long, calls stacking up as fast as the engine can process them
- [ ] `setInterval` throws an error once a callback exceeds the interval duration
- [ ] The interval pauses entirely until the slow callback completes, then resumes exactly on the original 100ms grid

<details>
<summary>Show Answer</summary>

**Answer:** B — Ticks can effectively overlap or fire back-to-back with no gap once the callback finally returns, since the browser only queues the next invocation and doesn't wait for slow callbacks to catch up gracefully — leading to drift and, if the callback consistently runs long, calls stacking up as fast as the engine can process them

**Explanation:** `setInterval` doesn't run callbacks concurrently (JS is single-threaded), but it also doesn't intelligently reschedule around slow callbacks — if a tick takes longer than the interval, the *next* tick is still just queued as soon as possible once the thread is free, meaning back-to-back execution with no actual pause, and the wall-clock spacing between ticks silently drifts away from the requested `100ms`. Over many iterations with an unreliable callback duration, this compounds into meaningful timing drift, which is why polling loops susceptible to variable-duration work are usually rewritten with self-rescheduling `setTimeout` (see Q15) instead. Option A and D describe smart compensation the API doesn't actually implement.

</details>
::

::question-wrapper
### Q9. A `setInterval` is running to poll a server every 5 seconds. The user switches to a different browser tab for 10 minutes. What commonly happens to the timer's firing rate in most modern browsers?

- [ ] It continues firing exactly every 5 seconds, unaffected by tab visibility
- [ ] It stops firing completely and never resumes, even after the tab becomes visible again
- [ ] Browsers throttle timers in background/inactive tabs (often to no more than once per second, and more aggressively for long-backgrounded tabs) to save power and CPU
- [ ] The interval speeds up to "catch up" once the tab regains focus

<details>
<summary>Show Answer</summary>

**Answer:** C — Browsers throttle timers in background/inactive tabs (often to no more than once per second, and more aggressively for long-backgrounded tabs) to save power and CPU

**Explanation:** **Performance:** modern browsers deliberately deprioritize timers in inactive tabs — a 5-second interval might slow to firing once per second or even less frequently the longer the tab stays backgrounded, and mobile browsers can throttle even more aggressively. This is intentional battery/CPU conservation, not a bug, but it surprises developers who assume timers are exact and can cause "why did polling stop working" bug reports. **Portability:** the exact throttling thresholds are browser-specific and not standardized, so code with hard real-time requirements shouldn't rely on background timer precision at all — visibility-aware logic (`document.visibilityState`) or server push (WebSockets) is more appropriate. Option A ignores this well-documented throttling; option B and D describe behaviors that don't match how throttling actually works (it slows, it doesn't stop or "catch up").

</details>
::

::question-wrapper{language="javascript"}
### Q10. What does this loop log, and why?

::code-wrapper{language="javascript"}
```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```
::

- [ ] `0, 1, 2` — each callback closes over the value of `i` at the time it was scheduled
- [ ] `3, 3, 3` — all three callbacks share the same `var i`, which has finished looping to `3` by the time any of them run
- [ ] `0, 0, 0` — each closure captures `i` as it was on the first iteration
- [ ] It throws a `ReferenceError` because `i` is out of scope in the callback

<details>
<summary>Show Answer</summary>

**Answer:** B — `3, 3, 3` — all three callbacks share the same `var i`, which has finished looping to `3` by the time any of them run

**Explanation:** `var` is function-scoped, not block-scoped, so there is only *one* `i` binding shared by the loop and all three closures. Since `setTimeout` always defers execution to a later macrotask (per Q2), the loop finishes completely — incrementing `i` to `3` and failing the `i < 3` check — before any of the callbacks actually run, so every closure reads the same final value. This is one of the most famous JavaScript interview gotchas; the fix is replacing `var` with `let`, which creates a fresh binding per iteration, making option A the *correct* behavior for a `let`-based loop but not for this `var`-based one.

</details>
::

::question-wrapper
### Q11. What does `setTimeout(fn, -100)` or `setTimeout(fn, "abc")` do?

- [ ] Both throw a `TypeError` immediately
- [ ] A negative or non-numeric (NaN-producing) delay is treated as `0` — `fn` is still scheduled as a macrotask on the next available tick
- [ ] Negative delays run the function immediately and synchronously, before the current line finishes
- [ ] The call is silently ignored and `fn` never runs

<details>
<summary>Show Answer</summary>

**Answer:** B — A negative or non-numeric (NaN-producing) delay is treated as `0` — `fn` is still scheduled as a macrotask on the next available tick

**Explanation:** Per the HTML timer spec, an invalid delay (negative, `NaN`, or omitted) is clamped to the minimum allowed value, effectively `0` — it does not throw and does not run synchronously. This still goes through the normal macrotask-queuing behavior from Q2, so `fn` runs after the current synchronous code completes, not immediately inline. Option C is the tempting trap of assuming a negative number somehow means "run before now," which isn't possible in an event-loop model. Option D and A both invent stricter failure behavior than what actually happens.

</details>
::

::question-wrapper{language="javascript"}
### Q12. A component sets up an interval on mount and stores the id, but a bug elsewhere in the app accidentally calls `clearInterval` twice with the same id (once in a cleanup function, once in an unrelated handler). What happens on the second call?

::code-wrapper{language="javascript"}
```javascript
const id = setInterval(poll, 1000);
clearInterval(id);
clearInterval(id); // called again elsewhere
```
::

- [ ] The second call throws because the interval is already cleared
- [ ] The second call is a harmless no-op, consistent with `clearTimeout`/`clearInterval` accepting any id (valid, stale, or already-cleared) without error
- [ ] The second call restarts the interval
- [ ] The second call clears a different, unrelated timer due to id reuse

<details>
<summary>Show Answer</summary>

**Answer:** B — The second call is a harmless no-op, consistent with `clearTimeout`/`clearInterval` accepting any id (valid, stale, or already-cleared) without error

**Explanation:** As established in Q3, clearing an already-cleared (or otherwise invalid) id is always safe and silent — there's no double-clear error to worry about. The subtler real risk here isn't the double-clear itself, it's option D's premise: timer ids *are* recycled by the engine once freed, so in a codebase with sloppy id bookkeeping, holding onto a stale id and clearing it "late" could — in a pathological, contrived case — coincide with an id reused for a newer, unrelated timer. In practice this requires mismanaging ids across unrelated code paths and is a code-hygiene problem, not something `clearInterval` itself does wrong; keeping id ownership local to the code that created the timer avoids it entirely.

</details>
::

::question-wrapper
### Q13. Unlike `setInterval`, what happens to a pending `requestAnimationFrame` callback when its tab is backgrounded (not visible)?

- [ ] It fires at the same rate regardless of visibility, since rAF is not subject to browser throttling
- [ ] Browsers pause rAF callbacks entirely for hidden/inactive tabs (no repaint needed), only resuming once the tab becomes visible again
- [ ] rAF throws an error if called while the tab is hidden
- [ ] rAF automatically converts itself into a `setInterval` while hidden

<details>
<summary>Show Answer</summary>

**Answer:** B — Browsers pause rAF callbacks entirely for hidden/inactive tabs (no repaint needed), only resuming once the tab becomes visible again

**Explanation:** **Performance:** since `requestAnimationFrame`'s entire purpose is to sync with the next paint, and hidden tabs don't paint, browsers simply stop invoking rAF callbacks while a tab is not visible — this is actually a *feature*, not a limitation, since it means animation loops using rAF automatically pause and save CPU/battery without any extra code. This is a meaningful practical difference from `setInterval`-driven "animations," which keep running (heavily throttled per Q9, but not fully paused) in the background, silently wasting resources on work nobody will see. Code with logic that must keep running regardless of visibility (e.g., a countdown timer) should not rely on rAF for that reason.

</details>
::

::question-wrapper
### Q14. If a callback registered with `setInterval` takes *longer* than the interval every single time (e.g., a 500ms interval running an 800ms task) for an extended period, what is the realistic outcome, as opposed to a naive mental model of "queued calls piling up infinitely"?

- [ ] The engine queues every missed tick and eventually executes a huge backlog all at once
- [ ] The browser effectively runs the callback back-to-back as fast as the thread allows (since only one pending invocation is queued at a time), so the interval behaves closer to "run continuously" than "run every 500ms," and the requested cadence is lost
- [ ] `setInterval` automatically cancels itself after 3 consecutive overruns
- [ ] The browser spawns a new thread per overrun to keep up with the schedule

<details>
<summary>Show Answer</summary>

**Answer:** B — The browser effectively runs the callback back-to-back as fast as the thread allows (since only one pending invocation is queued at a time), so the interval behaves closer to "run continuously" than "run every 500ms," and the requested cadence is lost

**Explanation:** Browsers do not queue unlimited backlog invocations of a `setInterval` callback (option A is a common but incorrect assumption) — implementations generally avoid stacking up multiple pending calls for the same interval, so a persistently-overrunning callback effectively degrades into running immediately after the previous call finishes, with no actual idle gap. This is the practical, worse cousin of the drift problem in Q8: not just imprecise timing, but the interval's requested cadence becoming meaningless. **Idiom:** the standard fix is a self-rescheduling `setTimeout` that only queues the *next* call after the current one finishes, so the true gap is always at least the intended delay (see Q15).

</details>
::

::question-wrapper{language="javascript"}
### Q15. Why do experienced engineers often replace `setInterval(fn, delay)` with a recursive `setTimeout` for polling loops?

::code-wrapper{language="javascript"}
```javascript
function poll() {
  doWork();
  setTimeout(poll, 1000);
}
setTimeout(poll, 1000);
```
::

- [ ] Recursive `setTimeout` runs faster than `setInterval` in every engine
- [ ] It guarantees a minimum gap between the *end* of one call and the *start* of the next, avoiding the overlap/back-to-back execution risk that `setInterval` has when a callback runs long
- [ ] `setInterval` is deprecated in modern JavaScript
- [ ] It's purely stylistic; there's no functional difference

<details>
<summary>Show Answer</summary>

**Answer:** B — It guarantees a minimum gap between the end of one call and the start of the next, avoiding the overlap/back-to-back execution risk that `setInterval` has when a callback runs long

**Explanation:** **Idiom:** because the next `setTimeout` is only scheduled *after* `doWork()` finishes, the interval between invocations always includes both the requested delay and however long the previous call took — there's no risk of the pile-up behavior described in Q14, since a new timer is never queued until the current work is fully done. This makes recursive `setTimeout` the safer default for polling/retry loops with variable-duration work. `setInterval` remains perfectly fine for short, reliably-fast, fixed-duration callbacks. Option C is false — `setInterval` isn't deprecated, just less safe for this specific case.

</details>
::

::question-wrapper{language="javascript"}
### Q16. Which implementation correctly debounces `fn` so it only runs once, `wait` ms after the *last* call?

::code-wrapper{language="javascript"}
```javascript
function debounce(fn, wait) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), wait);
  };
}
```
::

- [ ] This is broken — it needs to call `fn` immediately, then block further calls for `wait` ms
- [ ] This is correct — clearing the previous timer on every call means only the last call's scheduled invocation ever actually fires
- [ ] This is broken — `clearTimeout` inside the returned function causes an infinite loop
- [ ] This is correct, but only works for functions that take no arguments

<details>
<summary>Show Answer</summary>

**Answer:** B — This is correct — clearing the previous timer on every call means only the last call's scheduled invocation ever actually fires

**Explanation:** Each invocation cancels whatever timer the *previous* call had queued (a harmless no-op per Q3 if it already fired) and starts a fresh one; as long as calls keep arriving faster than `wait`, no scheduled `fn` call ever survives long enough to execute, so only the final call — the one with no subsequent call to cancel it — actually runs, `wait` ms after that last call. Option A describes throttle-with-leading-edge behavior, a different (also valid) pattern, but not what "debounce" means. Option D is wrong — `fn.apply(this, args)` correctly forwards whatever arguments and `this` context the wrapped call received.

</details>
::

::question-wrapper{language="javascript"}
### Q17. Which implementation correctly throttles `fn` to run at most once every `wait` ms, even under continuous calls?

::code-wrapper{language="javascript"}
```javascript
function throttle(fn, wait) {
  let lastCall = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastCall >= wait) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}
```
::

- [ ] This is broken — it never calls `fn` more than once, ever
- [ ] This is correct — it only invokes `fn` when at least `wait` ms have passed since the last actual invocation, allowing calls at a steady rate rather than resetting on every call like debounce
- [ ] This is broken — `Date.now()` cannot be used for timing comparisons
- [ ] This is identical to the debounce implementation in Q16, just renamed

<details>
<summary>Show Answer</summary>

**Answer:** B — This is correct — it only invokes `fn` when at least `wait` ms have passed since the last actual invocation, allowing calls at a steady rate rather than resetting on every call like debounce

**Explanation:** Unlike debounce (which cancels and restarts a timer on every call, so continuous calls can starve execution indefinitely), this throttle checks elapsed time on every call and lets `fn` fire as soon as the window has passed, then immediately starts a new window — so during a sustained burst (e.g., a `scroll` handler), `fn` still executes periodically instead of waiting for a pause. Option D is a common conflation, but the two implementations are meaningfully different: debounce uses `setTimeout`/`clearTimeout` and fires *after* activity stops, while this throttle uses a timestamp comparison and fires *during* continuous activity, at most once per window.

</details>
::

::question-wrapper
### Q18. A developer builds a drag-to-reposition UI feature and updates the element's position inside a `mousemove` handler using `setTimeout(update, 16)` scheduled repeatedly, aiming for ~60fps. What's the more idiomatic approach?

- [ ] This is already correct — 16ms is exactly one frame at 60fps
- [ ] Use `requestAnimationFrame` instead, since it syncs the visual update to the actual paint cycle rather than an approximate, drift-prone timer interval, and automatically pauses when the tab is hidden
- [ ] Reduce the delay to `0` for maximum responsiveness
- [ ] Use `setInterval(update, 16)` instead, since intervals are more precise than repeated `setTimeout`

<details>
<summary>Show Answer</summary>

**Answer:** B — Use `requestAnimationFrame` instead, since it syncs the visual update to the actual paint cycle rather than an approximate, drift-prone timer interval, and automatically pauses when the tab is hidden

**Explanation:** **Idiom:** `16ms` is only an *approximation* of one frame at exactly 60Hz — actual refresh rates vary (90Hz, 120Hz, or throttled displays), and timer-based scheduling has no knowledge of the browser's actual paint schedule, so visual updates driven by `setTimeout`/`setInterval` can land at the wrong moment relative to a repaint, causing visible jank or wasted work. `requestAnimationFrame` (from Q5) is purpose-built to solve exactly this, and as a bonus pauses automatically in hidden tabs (Q13), unlike option D's `setInterval`, which keeps running (throttled) in the background for no visual benefit.

</details>
::

::question-wrapper{language="javascript"}
### Q19. A component adds a `setInterval` for auto-refreshing data when it mounts. What is the idiomatic cleanup requirement, and what bug results if it's skipped?

::code-wrapper{language="javascript"}
```javascript
useEffect(() => {
  const id = setInterval(fetchLatest, 5000);
  return () => clearInterval(id);
}, []);
```
::

- [ ] The `return () => clearInterval(id)` line is unnecessary boilerplate and can be removed safely
- [ ] Skipping the cleanup leaks the interval — it keeps firing after the component unmounts, potentially calling `fetchLatest` (and any state updates it triggers) against a component that no longer exists, wasting resources or throwing warnings
- [ ] `clearInterval` must be called synchronously, not inside a cleanup function
- [ ] `setInterval` automatically stops itself when the component unmounts, without explicit cleanup

<details>
<summary>Show Answer</summary>

**Answer:** B — Skipping the cleanup leaks the interval — it keeps firing after the component unmounts, potentially calling `fetchLatest` (and any state updates it triggers) against a component that no longer exists, wasting resources or throwing warnings

**Explanation:** **Idiom:** timers are entirely independent of any UI framework's component lifecycle — nothing automatically stops a `setInterval` just because the component that created it was removed from the tree (option D is false). Without the cleanup function calling `clearInterval(id)` on unmount, the interval keeps running indefinitely, continuing to invoke `fetchLatest` and any state-setting logic inside it, which is a classic source of memory leaks and "can't update state on an unmounted component" warnings in UI frameworks. This is a specific case of the general rule: any subscription-like resource (timers, event listeners, observers) needs matching teardown.

</details>
::

::question-wrapper
### Q20. A search box needs to hit an autocomplete API as the user types, and a separate feature needs to update a "scroll progress" indicator as the user scrolls the page. Which combination is the idiomatic choice?

- [ ] Debounce both — waiting for a pause makes sense for both features
- [ ] Throttle both — a steady rate limit is correct for both features
- [ ] Debounce the search input (fire the API call after typing pauses); throttle the scroll handler (update the indicator at a steady rate during continuous scrolling)
- [ ] Neither needs rate-limiting — modern browsers handle this automatically

<details>
<summary>Show Answer</summary>

**Answer:** C — Debounce the search input (fire the API call after typing pauses); throttle the scroll handler (update the indicator at a steady rate during continuous scrolling)

**Explanation:** **Idiom:** these are the textbook use cases for each technique, precisely because of the semantic difference established in Q7 and Q16/Q17. Search-as-you-type benefits from *waiting* until the user pauses — firing an API request after every keystroke wastes bandwidth and creates race conditions between in-flight requests (see Q19 in the Web Storage & APIs quiz on `AbortController`), so debounce is correct. A scroll progress indicator, by contrast, needs to visibly *keep updating* throughout continuous scrolling rather than going silent until scrolling stops — using debounce there would make the UI look frozen during the scroll and only "catch up" once the user stops, which is the wrong feel; throttle keeps it responsive at a bounded, performance-friendly update rate instead.

</details>
::
