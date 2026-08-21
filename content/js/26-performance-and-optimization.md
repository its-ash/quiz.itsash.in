# 26 — Performance & Optimization

::question-wrapper{language="javascript"}
### Q1. After the code below runs, is the removed button eligible for garbage collection?

::code-wrapper{language="javascript"}
```javascript
function attachHandler() {
  const btn = document.getElementById("submit");
  let clickCount = 0;
  btn.addEventListener("click", () => {
    clickCount++;
    console.log(clickCount);
  });
}
attachHandler();
document.getElementById("submit").remove();
```
::

- [ ] No — the closure passed to `addEventListener` still holds a reference to `btn` (and `clickCount`), keeping the detached node alive until the listener is removed or the reference is released
- [ ] Yes, once removed from the DOM it's eligible immediately regardless of listeners
- [ ] Yes, because `clickCount` is a primitive and doesn't affect garbage collection
- [ ] No, because `getElementById` caches all elements internally forever

<details>
<summary>Show Answer</summary>

**Answer:** A — No — the closure passed to `addEventListener` still holds a reference to `btn` (and `clickCount`), keeping the detached node alive until the listener is removed or the reference is released

**Explanation:** **Debug:** Removing an element from the DOM tree does not remove any event listeners attached to it, and the closure created inside `attachHandler` still captures `btn` in its scope for as long as the listener function itself is reachable — which it is, since the DOM's internal listener registry keeps a reference to it. This is the classic "detached DOM node" leak: the node is invisible on the page but still resident in memory. Option B is the common misconception that DOM removal alone triggers collection. Option C is irrelevant — a captured primitive doesn't change how `btn` is retained. Option D fabricates caching behavior `getElementById` doesn't have. The fix is to call `btn.removeEventListener(...)` (or use an `AbortController` signal) before/when discarding the node.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What's the memory/behavior consequence of removing `clockEl` without clearing the interval?

::code-wrapper{language="javascript"}
```javascript
function startPolling(el) {
  setInterval(() => {
    el.textContent = new Date().toLocaleTimeString();
  }, 1000);
}
const clockEl = document.getElementById("clock");
startPolling(clockEl);
clockEl.remove();
```
::

- [ ] The interval automatically stops once its target element is detached from the DOM
- [ ] The interval keeps firing forever, and its closure keeps a live reference to `clockEl`, preventing the detached node from ever being garbage collected — a classic leak
- [ ] `clockEl` becomes eligible for garbage collection immediately since it no longer has a parent
- [ ] `setInterval` throws once `el` is detached, alerting you to the leak

<details>
<summary>Show Answer</summary>

**Answer:** B — The interval keeps firing forever, and its closure keeps a live reference to `clockEl`, preventing the detached node from ever being garbage collected — a classic leak

**Explanation:** **Debug:** `setInterval` has no awareness of the DOM — it keeps invoking its callback on schedule until `clearInterval` is explicitly called, regardless of whether the elements it touches are still attached. The arrow function's closure over `el` keeps `clockEl` reachable forever, so the detached node (and everything it references) leaks for the lifetime of the page. Option A and D invent automatic cleanup/error behavior that doesn't exist. Option C ignores that the interval's closure is still a live reference even after DOM removal. The fix is to store the interval id and call `clearInterval(id)` whenever the element is torn down (e.g., in a component's cleanup/unmount logic).

</details>
::

::question-wrapper{language="javascript"}
### Q3. Over a long-running session with thousands of unique ids, what's the risk with this cache, and how should it be fixed?

::code-wrapper{language="javascript"}
```javascript
const cache = new Map();
function getUser(id) {
  if (cache.has(id)) return cache.get(id);
  const user = fetchUserSync(id);
  cache.set(id, user);
  return user;
}
```
::

- [ ] None — `Map` automatically evicts old entries once memory pressure is detected
- [ ] This can never leak because `id` is typically a primitive number
- [ ] The cache grows unbounded since entries are never removed, steadily increasing memory use; fix by adding an eviction policy (LRU, TTL, or a max-size cap) — a `WeakMap` would not help here since the keys are primitive ids, not objects being tracked elsewhere
- [ ] `Map` keys are always garbage collected once the function returns, so this is safe

<details>
<summary>Show Answer</summary>

**Answer:** C — The cache grows unbounded since entries are never removed, steadily increasing memory use; fix by adding an eviction policy (LRU, TTL, or a max-size cap) — a `WeakMap` would not help here since the keys are primitive ids, not objects being tracked elsewhere

**Explanation:** A `Map` never evicts entries on its own — it holds a strong reference to every key and value for as long as the entry exists, so an ever-growing set of unique ids means an ever-growing cache with no upper bound. Option A invents automatic eviction that `Map` does not provide. Option B and D wrongly assume primitive keys are somehow exempt from retention — primitives are stored and retained just like any other value. The real fix is bounding growth explicitly (an LRU cache, a TTL-based expiry, or a max-entry cap that evicts the oldest/least-used entry).

</details>
::

::question-wrapper
### Q4. You're implementing search-as-you-type autocomplete that should fire an API request only once the user pauses typing for 300ms, not on every keystroke. Which technique fits, and why?

- [ ] Throttle — it guarantees the function runs at least once every 300ms even during continuous typing
- [ ] Throttle — it runs the function immediately on the first keystroke and ignores the rest
- [ ] Debounce — it runs the function on every keystroke but batches the results
- [ ] Debounce — it delays execution until 300ms have passed with no new keystrokes, so an idle pause is what finally triggers the call; throttle would still fire repeatedly at intervals *while* the user keeps typing, which isn't what's wanted here

<details>
<summary>Show Answer</summary>

**Answer:** D — Debounce — it delays execution until 300ms have passed with no new keystrokes, so an idle pause is what finally triggers the call; throttle would still fire repeatedly at intervals *while* the user keeps typing, which isn't what's wanted here

**Explanation:** Debounce resets its timer on every call and only fires once the calls stop for the configured wait — exactly what "wait until the user pauses" needs. Throttle (options A and B) instead guarantees execution on a steady cadence *during* continuous activity, which would fire requests mid-typing rather than waiting for a pause — the two are frequently confused because both "limit" how often a function runs, but they solve opposite problems. Option C misdescribes debounce as running on every keystroke, which defeats its entire purpose.

</details>
::

::question-wrapper
### Q5. A scroll handler recalculates a sticky header's position and needs to run at most once every 100ms while the user is actively scrolling, not only after scrolling stops. Which is correct?

- [ ] Throttle with a 100ms interval — the handler executes on a steady cadence during continuous scrolling instead of only after it stops, keeping the header visually in sync while scrolling happens
- [ ] Debounce with a 100ms wait — ensures the handler only runs after scrolling stops
- [ ] `requestIdleCallback` — runs the handler only when the browser is completely idle
- [ ] Neither — scroll handlers should never be rate-limited since browsers already throttle them to the display refresh rate automatically

<details>
<summary>Show Answer</summary>

**Answer:** A — Throttle with a 100ms interval — the handler executes on a steady cadence during continuous scrolling instead of only after it stops, keeping the header visually in sync while scrolling happens

**Explanation:** Throttle guarantees the function runs at a capped rate *while* events keep firing, which is exactly what a sticky header needs — visual updates during the scroll, not just at the end. Option B (debounce) would make the header appear frozen until scrolling fully stops, which looks broken for a "stays in sync" requirement. Option C is wrong because `requestIdleCallback` only runs during idle gaps and offers no guarantee of running during continuous scrolling at all. Option D is a myth — native `scroll` events can fire far more often than needed for visual updates, and manual rate-limiting is standard practice, not redundant.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What gets logged, and at roughly what time?

::code-wrapper{language="javascript"}
```javascript
function debounce(fn, wait) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), wait);
  };
}

const log = debounce((x) => console.log(x), 200);
log("a");
setTimeout(() => log("b"), 100);
setTimeout(() => log("c"), 250);
```
::

- [ ] `a`, `b`, and `c` all log, at 200ms, 300ms, and 450ms respectively — since each call schedules its own timer independently
- [ ] Only `c` logs, at ~450ms — each new call clears the still-pending timer from the previous call before it can fire, and since no further call arrives within 200ms of the last one, only its timer survives to completion
- [ ] Only `a` logs, at 200ms — later calls are ignored once a timer is already pending
- [ ] `b` and `c` both log, at 300ms and 450ms — only the very first call's timer gets cancelled

<details>
<summary>Show Answer</summary>

**Answer:** B — Only `c` logs, at ~450ms — each new call clears the still-pending timer from the previous call before it can fire, and since no further call arrives within 200ms of the last one, only its timer survives to completion

**Explanation:** **Debug:** Trace it: `log("a")` at t=0 schedules a fire at t=200. `log("b")` at t=100 calls `clearTimeout` on that pending timer (cancelling `a` before it ever fires) and schedules its own fire at t=300. `log("c")` at t=250 again clears the still-pending timer (cancelling `b`, since 250 < 300) and schedules its own fire at t=450. No further call arrives before t=450, so `fn("c")` finally runs — nothing else does. Option A ignores that `clearTimeout` cancels the previous pending call entirely rather than letting each accumulate. Option C stops tracing too early. Option D misidentifies which timer survives.

</details>
::

::question-wrapper{language="javascript"}
### Q7. What perf benefit does V8 get from `a` and `b` being built with the same property order and types?

::code-wrapper{language="javascript"}
```javascript
function makePoint(x, y) {
  const p = {};
  p.x = x;
  p.y = y;
  return p;
}
const a = makePoint(1, 2);
const b = makePoint(3, 4);
```
::

- [ ] Nothing — V8 treats every object as a fully dynamic hash map regardless of creation pattern
- [ ] V8 merges `a` and `b` into a single object in memory to save space
- [ ] `a` and `b` end up sharing the same underlying "hidden class" (shape), letting V8 use fast, offset-based property access instead of a slower dictionary lookup for both objects
- [ ] Hidden classes only apply to objects created with `class` syntax, not object literals

<details>
<summary>Show Answer</summary>

**Answer:** C — `a` and `b` end up sharing the same underlying "hidden class" (shape), letting V8 use fast, offset-based property access instead of a slower dictionary lookup for both objects

**Explanation:** **Performance:** V8 assigns objects an internal "hidden class" (a.k.a. "shape" or "map") that describes their property layout. Because `makePoint` always adds `x` then `y`, in that order, with consistent value types, both `a` and `b` transition through the exact same sequence of hidden classes and end up sharing one — letting V8 compile property access as a fixed memory offset instead of a hash-map lookup. Option A describes the slow, generic fallback path that hidden classes exist to avoid. Option B fabricates object merging. Option D is false — hidden classes apply to any object, `class`-based or not.

</details>
::

::question-wrapper{language="javascript"}
### Q8. Building on the previous scenario, what happens to the shared hidden-class optimization once this runs?

::code-wrapper{language="javascript"}
```javascript
b.z = 99;
if (Math.random() > 0.9) a.y = "surprise";
```
::

- [ ] Nothing changes — hidden classes are assigned once at creation and never affected by later mutation
- [ ] V8 automatically adds a `z` property to `a` as well to keep the shapes in sync
- [ ] This only matters for arrays, not plain objects
- [ ] Adding `z` only to `b` gives `b` a different hidden-class transition than `a`, and changing `a.y` from a number to a string can force V8 to fall back to a slower representation for `a` — both objects lose the shared fast shape, and functions operating on both become polymorphic

<details>
<summary>Show Answer</summary>

**Answer:** D — Adding `z` only to `b` gives `b` a different hidden-class transition than `a`, and changing `a.y` from a number to a string can force V8 to fall back to a slower representation for `a` — both objects lose the shared fast shape, and functions operating on both become polymorphic

**Explanation:** **Performance:** Hidden classes are dynamic, not frozen at creation — every property addition or type change is itself a transition to a (possibly new) hidden class. Adding `z` only to `b` diverges `b`'s shape from `a`'s; changing `a.y`'s type mid-flight can force V8 out of the fast tracked-shape path entirely for `a`. Any function that previously saw only one shared shape now sees two different shapes, becoming polymorphic and losing its fast inline-cached path. Option A is false — mutation absolutely affects hidden-class assignment. Option B invents synchronization behavior V8 doesn't perform. Option C is wrong — this applies to any plain object, not just arrays.

</details>
::

::question-wrapper{language="javascript"}
### Q9. `getArea(shape)` reads `shape.width * shape.height` and runs thousands of times per frame. Why is it faster when every `shape` has the same property layout than when shapes vary?

::code-wrapper{language="javascript"}
```javascript
function getArea(shape) {
  return shape.width * shape.height;
}
```
::

- [ ] Consistent shapes make the call site monomorphic (one hidden class seen), letting the JIT inline a fast, specialized path; varying shapes make it polymorphic/megamorphic, forcing V8 to fall back to slower generic property lookups and inline-cache misses
- [ ] It doesn't matter — the JIT re-optimizes on every call regardless of shape
- [ ] Only `TypedArray`s benefit from consistent shapes; plain objects are unaffected
- [ ] Polymorphic call sites are always faster because V8 can choose the best strategy per call

<details>
<summary>Show Answer</summary>

**Answer:** A — Consistent shapes make the call site monomorphic (one hidden class seen), letting the JIT inline a fast, specialized path; varying shapes make it polymorphic/megamorphic, forcing V8 to fall back to slower generic property lookups and inline-cache misses

**Explanation:** **Performance:** V8's inline caches record which hidden class(es) a call site has seen for `shape`. If it's always the same shape (monomorphic), the JIT can specialize `getArea` to read `width`/`height` at fixed offsets directly. Once the call site sees several distinct shapes (polymorphic) or too many (megamorphic), V8 gives up on a specialized fast path and falls back to a generic, slower lookup on every call. Option B and D contradict how inline caches actually behave — variety is a cost, not a benefit. Option C is false; this optimization applies broadly to plain objects, not just typed arrays.

</details>
::

::question-wrapper{language="javascript"}
### Q10. In a hot path, what's the perf concern with `delete cfg.debug`, and what's the preferred alternative?

::code-wrapper{language="javascript"}
```javascript
const config = { debug: true, retries: 3, timeout: 500 };
function disableDebug(cfg) {
  delete cfg.debug;
}
```
::

- [ ] `delete` is always fastest since it fully frees the property's memory immediately
- [ ] `delete` changes the object's hidden class (creating a new, often "dictionary mode" shape), which can deoptimize property access for that object; setting `cfg.debug = undefined` (or restructuring so the property was never conditionally needed) preserves the original shape and avoids the transition
- [ ] There's no concern — `delete` behaves identically to setting a property to `undefined`
- [ ] `delete` is disallowed in strict mode, so this code throws a `SyntaxError`

<details>
<summary>Show Answer</summary>

**Answer:** B — `delete` changes the object's hidden class (creating a new, often "dictionary mode" shape), which can deoptimize property access for that object; setting `cfg.debug = undefined` (or restructuring so the property was never conditionally needed) preserves the original shape and avoids the transition

**Explanation:** **Performance:** Unlike simply overwriting a value, `delete` removes a property slot entirely, which forces a hidden-class transition and can push the object into a slower, dictionary-mode (hash-map-like) representation that loses the fast offset-based access other objects of the "same" original shape still enjoy. `cfg.debug = undefined` keeps the property (and the object's shape) intact — only the value changes — so it avoids the deopt. Option A gets the performance direction backwards. Option C ignores the shape-transition cost that makes the two meaningfully different. Option D is fabricated — `delete` on a configurable own property is valid in strict mode.

</details>
::

::question-wrapper{language="javascript"}
### Q11. Why can iterating/summing `dense` be significantly faster than doing the same over `sparse` or `alsoSparse`?

::code-wrapper{language="javascript"}
```javascript
const dense = [1, 2, 3, 4, 5];

const sparse = [1, 2, 3, 4, 5];
delete sparse[2];

const alsoSparse = new Array(5);
alsoSparse[0] = "x";
```
::

- [ ] There is no difference — all three are stored identically as contiguous memory blocks
- [ ] `sparse` and `alsoSparse` are actually faster because V8 pre-allocates extra memory for them
- [ ] `dense` stays on V8's fast, packed "elements kind" (contiguous backing store); creating holes with `delete` or `new Array(n)` transitions the array to a slower "holey"/dictionary-like representation, since engines must now check for and skip missing indices on every access
- [ ] `delete sparse[2]` shrinks the array's `length` by one, making it faster to iterate

<details>
<summary>Show Answer</summary>

**Answer:** C — `dense` stays on V8's fast, packed "elements kind" (contiguous backing store); creating holes with `delete` or `new Array(n)` transitions the array to a slower "holey"/dictionary-like representation, since engines must now check for and skip missing indices on every access

**Explanation:** **Performance:** V8 tracks an array's "elements kind" internally. A fully packed array like `dense` can use a tight, contiguous fast path. `delete sparse[2]` doesn't shrink the array or fill the gap — it leaves an actual hole at index 2 (the array's `length` stays 5), which downgrades it to a "holey" representation that must check for missing slots on every access. `new Array(5)` starts out entirely holey by construction, for the same reason. Option A and B invert the real cost. Option D is a common misconception — `delete` on an array index never changes `length`.

</details>
::

::question-wrapper
### Q12. An animation loop uses `setInterval(update, 16)` instead of `requestAnimationFrame(update)`. Beyond potential drift from the display's refresh rate, what's another concrete downside in production?

- [ ] `setInterval` cannot be cleared once started, so the animation runs forever even after the component unmounts
- [ ] `setInterval` runs on a separate thread, causing race conditions with the DOM
- [ ] `setInterval` cannot accept a callback that touches the DOM
- [ ] `setInterval` keeps firing at roughly the same rate even when the tab is backgrounded/hidden, wasting CPU and battery on invisible work, whereas `requestAnimationFrame` callbacks are automatically throttled/paused by the browser for hidden tabs

<details>
<summary>Show Answer</summary>

**Answer:** D — `setInterval` keeps firing at roughly the same rate even when the tab is backgrounded/hidden, wasting CPU and battery on invisible work, whereas `requestAnimationFrame` callbacks are automatically throttled/paused by the browser for hidden tabs

**Explanation:** **Performance:** Browsers specifically de-prioritize `requestAnimationFrame` for hidden/background tabs (heavily throttling or pausing it entirely, since there's nothing to paint), which saves CPU and battery for work the user can't even see. `setInterval` has no such built-in awareness of visibility and keeps ticking at roughly its configured rate regardless, silently burning resources in the background. Option A is false — `clearInterval` works identically to clearing any interval. Options B and C invent thread and DOM-access restrictions that don't exist; `setInterval` runs on the main thread just like `requestAnimationFrame`.

</details>
::

::question-wrapper{language="javascript"}
### Q13. Running this synchronous computation in a click handler freezes the UI for its duration. What's the correct fix, and why?

::code-wrapper{language="javascript"}
```javascript
function findPrimesUpTo(n) {
  const primes = [];
  for (let i = 2; i <= n; i++) {
    let isPrime = true;
    for (let j = 2; j * j <= i; j++) {
      if (i % j === 0) { isPrime = false; break; }
    }
    if (isPrime) primes.push(i);
  }
  return primes;
}
document.getElementById("go").addEventListener("click", () => {
  const result = findPrimesUpTo(5_000_000);
  render(result);
});
```
::

- [ ] Move the computation into a Web Worker — since JS is single-threaded, any synchronous CPU-bound work on the main thread blocks rendering and event handling no matter how it's scheduled; a worker runs on a separate thread so the main thread stays responsive
- [ ] Wrap the call in a `Promise` so it runs asynchronously without blocking
- [ ] Use `setTimeout(() => findPrimesUpTo(5_000_000), 0)` to defer it to a macrotask
- [ ] Mark the function `async` so it yields control back to the event loop periodically

<details>
<summary>Show Answer</summary>

**Answer:** A — Move the computation into a Web Worker — since JS is single-threaded, any synchronous CPU-bound work on the main thread blocks rendering and event handling no matter how it's scheduled; a worker runs on a separate thread so the main thread stays responsive

**Explanation:** **Performance:** JavaScript on the main thread is single-threaded and cooperative — once a synchronous function starts running, it monopolizes the thread (blocking layout, painting, and input handling) until it returns, no matter how it got invoked. A Web Worker executes on an entirely separate thread, so the loop runs without ever touching the thread responsible for rendering and interaction. Option B is wrong — wrapping in a `Promise` doesn't make the executor's synchronous body non-blocking; the loop still runs to completion inline. Option C only delays *when* the freeze starts, not whether it happens. Option D is wrong — `async` functions don't magically yield mid-loop; only an actual `await` (or moving off-thread) does, and there's no `await` anywhere in this loop.

</details>
::

::question-wrapper
### Q14. A single-page app ships one 3MB JS bundle containing code for every route, including an admin dashboard only 2% of users ever visit. What's the standard fix, and what does it primarily improve?

- [ ] Minify the bundle further — minification alone can typically cut a well-structured bundle to under 3MB total
- [ ] Split the bundle by route/feature (code splitting) and lazy-load the admin dashboard's code only when a user navigates to it — this reduces the initial bundle size, improving time-to-interactive for the 98% of users who never load that code
- [ ] Inline all JS into the HTML `<head>` so there's no separate network request
- [ ] Move all JS to inline `<script>` tags at the bottom of `<body>` instead of an external file

<details>
<summary>Show Answer</summary>

**Answer:** B — Split the bundle by route/feature (code splitting) and lazy-load the admin dashboard's code only when a user navigates to it — this reduces the initial bundle size, improving time-to-interactive for the 98% of users who never load that code

**Explanation:** Code splitting plus lazy loading (e.g., dynamic `import()`) lets the bundler emit separate chunks per route/feature, so the browser downloads and parses only what the current page needs, shrinking the critical bundle most users actually pay for. Option A overstates what minification alone can realistically achieve on an already-built, feature-complete bundle. Options C and D just relocate the same amount of JS without reducing what has to be downloaded and parsed before the page becomes interactive — they can make things worse by blocking `<head>` parsing or bloating the HTML document itself.

</details>
::

::question-wrapper
### Q15. A page renders a scrollable table of 50,000 rows by mapping the full dataset directly to 50,000 `<tr>` elements, causing visible jank. What's the standard fix, and why does it help?

- [ ] Replace the `<table>` with `<div>`s, since divs render faster than table elements
- [ ] Add `will-change: transform` to every row to force GPU acceleration of all 50,000 rows
- [ ] Use virtualization/windowing to render only the rows currently visible in the viewport (plus a small buffer), swapping their content as the user scrolls — this keeps the number of live DOM nodes small regardless of dataset size, avoiding the cost of creating/laying out/painting tens of thousands of nodes at once
- [ ] Use `list-style: none` to skip the browser's default list rendering

<details>
<summary>Show Answer</summary>

**Answer:** C — Use virtualization/windowing to render only the rows currently visible in the viewport (plus a small buffer), swapping their content as the user scrolls — this keeps the number of live DOM nodes small regardless of dataset size, avoiding the cost of creating/laying out/painting tens of thousands of nodes at once

**Explanation:** Virtualization decouples the number of live DOM nodes from the size of the underlying dataset — only the handful of rows actually in (or near) the viewport ever exist as real elements, with the rest represented purely as data until they scroll into view. This keeps DOM node count, layout cost, and paint cost roughly constant no matter how large the dataset grows. Option A is a myth — element tag choice has negligible impact next to node count. Option B actually makes things worse, since `will-change` reserves GPU compositing layers per element and applying it to 50,000 rows wastes memory. Option D is irrelevant to `<table>`/`<tr>` rendering, which has no list markers to begin with.

</details>
::

::question-wrapper{language="javascript"}
### Q16. A particle-based game loop allocates thousands of new particle objects per second and discards them once off-screen, running at 60fps. Players notice periodic stutters. What's a common cause and fix?

::code-wrapper{language="javascript"}
```javascript
function spawnParticles(n) {
  for (let i = 0; i < n; i++) {
    particles.push(new Particle(x, y, vx, vy));
  }
}
```
::

- [ ] The stutters are caused by `requestAnimationFrame` itself pausing periodically; switching to `setInterval` fixes it
- [ ] The stutters are unrelated to allocation; they're caused by using `class` instead of object literals for `Particle`
- [ ] Switching `Particle` fields from `let` to `const` prevents the stutter by making objects immutable
- [ ] Constantly allocating and discarding thousands of short-lived objects per second creates heavy garbage collection pressure, and the GC's collection pauses cause the stutters; an object pool that reuses a fixed set of pre-allocated particle objects (resetting their fields instead of reallocating) avoids the allocation churn

<details>
<summary>Show Answer</summary>

**Answer:** D — Constantly allocating and discarding thousands of short-lived objects per second creates heavy garbage collection pressure, and the GC's collection pauses cause the stutters; an object pool that reuses a fixed set of pre-allocated particle objects (resetting their fields instead of reallocating) avoids the allocation churn

**Explanation:** **Performance:** High-frequency allocation of short-lived objects fills the young generation heap quickly, triggering frequent minor GC cycles; even brief collection pauses are enough to drop frames in a 60fps loop, producing visible stutter. An object pool sidesteps this by allocating a fixed set of particles once up front and recycling them (resetting position/velocity fields) instead of creating and discarding new ones every frame, which drastically cuts allocation churn and GC pressure. Option A blames the wrong scheduler and proposes a strictly worse alternative (see Q12). Options B and C misattribute the cause to unrelated syntax choices that don't affect allocation behavior.

</details>
::

::question-wrapper
### Q17. A developer notices the app feels slow and immediately rewrites a `for` loop as `array.reduce()` because "functional style is faster," without measuring anything first. What's wrong with this approach?

- [ ] This is premature optimization based on assumption rather than data; the actual bottleneck could be anywhere (a layout thrash, an unbatched network waterfall, a memory leak), and swapping loop syntax alone rarely matters for performance — the correct approach is to profile first (e.g., the browser's Performance/Profiler panel) to find the actual hot path before changing code
- [ ] Nothing — `reduce()` is always faster than a `for` loop in every JS engine
- [ ] `reduce()` should always be avoided since it's slower than `for` loops in every case
- [ ] Rewriting loop syntax is the single most impactful optimization available in JavaScript

<details>
<summary>Show Answer</summary>

**Answer:** A — This is premature optimization based on assumption rather than data; the actual bottleneck could be anywhere (a layout thrash, an unbatched network waterfall, a memory leak), and swapping loop syntax alone rarely matters for performance — the correct approach is to profile first (e.g., the browser's Performance/Profiler panel) to find the actual hot path before changing code

**Explanation:** "Feels slow" is a symptom, not a diagnosis — the actual bottleneck is just as likely to be a network waterfall, layout thrashing, an unbounded cache, or a single hot function buried deep in a call stack as it is to be loop syntax, and guessing wastes effort on code that may not matter at all. Profiling tools (flame graphs, the Performance panel, `console.time`/`performance.mark` around suspected hot paths) point directly at where time is actually spent. Options B and D both overstate loop-syntax swaps as a universal win, and option C overstates the opposite — in reality, the difference between `for` and `reduce()` is usually negligible next to real bottlenecks and is engine/case-dependent.

</details>
::

::question-wrapper{language="javascript"}
### Q18. `el` is a DOM element passed in from various parts of the app; some are later removed from the DOM with no other references held. What's the memory problem, and what's the direct fix?

::code-wrapper{language="javascript"}
```javascript
const cache = new Map();
function getMetadata(el) {
  if (!cache.has(el)) {
    cache.set(el, computeExpensiveMetadata(el));
  }
  return cache.get(el);
}
```
::

- [ ] There's no problem — `Map` automatically drops entries whose keys are no longer referenced elsewhere
- [ ] `cache` (a `Map`) holds a strong reference to each `el` key, so as long as an entry exists in the cache, the removed DOM element can never be garbage collected, even though nothing else references it — replacing `Map` with `WeakMap` lets those keys (and their entries) be collected once the element has no other referrers
- [ ] The fix is to call `cache.clear()` after every use
- [ ] `WeakMap` would make this worse, since weak references are collected too aggressively and could drop entries while still in use elsewhere

<details>
<summary>Show Answer</summary>

**Answer:** B — `cache` (a `Map`) holds a strong reference to each `el` key, so as long as an entry exists in the cache, the removed DOM element can never be garbage collected, even though nothing else references it — replacing `Map` with `WeakMap` lets those keys (and their entries) be collected once the element has no other referrers

**Explanation:** A regular `Map` retains every key strongly, exactly like any other reference — so simply being a `Map` key is enough to keep an otherwise-unreferenced, detached DOM element alive indefinitely, leaking memory as elements accumulate over the session. `WeakMap` holds its keys *weakly*: once nothing outside the `WeakMap` references a given key object, the engine is free to collect it, and the corresponding entry disappears automatically. Option A invents automatic eviction `Map` doesn't have. Option C is a workaround that discards the entire cache's usefulness rather than fixing the leak surgically. Option D misunderstands weak references — they're only collected when truly unreachable elsewhere, not "aggressively."

</details>
::

::question-wrapper{language="javascript"}
### Q19. Having switched the metadata cache to a `WeakMap`, a developer tries to run this. What happens?

::code-wrapper{language="javascript"}
```javascript
console.log(cache.size);
for (const [key, value] of cache) {
  console.log(key, value);
}
```
::

- [ ] `cache.size` returns the count correctly, but `for...of` throws because `WeakMap` is not iterable
- [ ] Both work identically to `Map`, since `WeakMap` is just a `Map` with automatic cleanup
- [ ] `WeakMap` has neither `.size` nor `.keys()`/`.entries()`/iteration support at all — this is deliberate, since the collection's contents can change at any moment as the engine garbage-collects keys, so exposing enumeration would make behavior non-deterministic; if you need to enumerate or count entries, a regular `Map` (with manual cleanup) is the right tool instead
- [ ] `cache.size` throws a `TypeError`, but iteration is supported and returns only the currently-live entries

<details>
<summary>Show Answer</summary>

**Answer:** C — `WeakMap` has neither `.size` nor `.keys()`/`.entries()`/iteration support at all — this is deliberate, since the collection's contents can change at any moment as the engine garbage-collects keys, so exposing enumeration would make behavior non-deterministic; if you need to enumerate or count entries, a regular `Map` (with manual cleanup) is the right tool instead

**Explanation:** **Debug:** `console.log(cache.size)` logs `undefined` (no `.size` getter exists on `WeakMap.prototype`), and the `for...of` loop throws a `TypeError` because `WeakMap` implements neither `Symbol.iterator` nor `.entries()`. This is an intentional spec design: garbage collection timing is non-deterministic and implementation-defined, so if you could enumerate a `WeakMap`'s contents, the results (and even whether a collection pass happened mid-iteration) would be observable and inconsistent across engines — the spec avoids that entirely by disallowing enumeration. Options A and D each get one half right but the other half wrong. Option B conflates `WeakMap` with `Map`, which does support both `.size` and iteration.

</details>
::

::question-wrapper
### Q20. A profiler flame graph shows 80% of a slow interaction's time inside a single `recalculateLayout()` function, called synchronously once per scroll event (dozens of times per second). What's the most appropriate first fix?

- [ ] Rewrite the entire app in a different framework, since the framework is likely the root cause
- [ ] Add `console.time`/`console.timeEnd` calls throughout the codebase before making any change, since more logging is always the correct first step
- [ ] Convert all `var` declarations in the file to `const`/`let`, since modern syntax is inherently faster
- [ ] Throttle the scroll handler so `recalculateLayout()` runs at a capped rate (or move the work to `requestAnimationFrame`) — this directly targets the measured bottleneck (call frequency) without speculative, unrelated changes

<details>
<summary>Show Answer</summary>

**Answer:** D — Throttle the scroll handler so `recalculateLayout()` runs at a capped rate (or move the work to `requestAnimationFrame`) — this directly targets the measured bottleneck (call frequency) without speculative, unrelated changes

**Explanation:** The profiler already identified the exact bottleneck: an expensive function invoked far more often than necessary on a high-frequency event. The direct, evidence-backed fix is to cut the call frequency — throttling, or batching the work into a single `requestAnimationFrame` callback per visual frame — rather than guessing at unrelated changes. Option B misunderstands the situation: the profiling data already exists, so adding scattered manual timers is redundant busywork, not "the correct first step." Options A and C are classic cargo-cult "optimizations" — a framework rewrite and syntax modernization — that don't address the measured cause at all and risk introducing new bugs for no measured benefit.

</details>
::
