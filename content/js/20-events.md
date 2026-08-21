# 20 — Events

::question-wrapper
### Q1. In the default event flow model used by the DOM, what happens when a user clicks a deeply nested `<span>` inside several wrapping `<div>` elements, assuming no listener calls `stopPropagation()`?

- [ ] The click event fires on the `<span>` first, then propagates upward, firing on each ancestor in turn up to `document`
- [ ] The click event fires only on the `<span>`; ancestor elements never see it
- [ ] The click event fires on `document` first, then downward to the `<span>`, then stops
- [ ] The click event fires on every ancestor and the `<span>` at the same time, in an unspecified order

<details>
<summary>Show Answer</summary>

**Answer:** A — The click event fires on the `<span>` first, then propagates upward, firing on each ancestor in turn up to `document`

**Explanation:** This is the bubbling phase, the default direction of event flow: the event originates at the deepest element the user interacted with (the "target phase"), then walks up the ancestor chain, firing any matching listener on each ancestor in turn, unless something calls `stopPropagation()`. Option B ignores bubbling entirely. Option C describes only the capturing phase (which does run top-down, but before the target phase, not instead of bubbling) and incorrectly claims it "stops." Option D is wrong because the phases are strictly sequential and deterministic, never simultaneous.

</details>
::

::question-wrapper
### Q2. By default, `element.addEventListener('click', handler)` registers the handler for which phase of the event flow?

- [ ] The capturing phase, so it runs before any bubbling-phase listeners on ancestors
- [ ] The bubbling phase, so it runs after the capturing phase has already passed through this element
- [ ] Both phases simultaneously, unless `capture: false` is explicitly set
- [ ] Neither phase — plain calls without `{capture: true}` only fire when the element is the exact `event.target`

<details>
<summary>Show Answer</summary>

**Answer:** B — The bubbling phase, so it runs after the capturing phase has already passed through this element

**Explanation:** `addEventListener`'s third argument defaults to `false` (or an options object with `capture` omitted, which also defaults to `false`), meaning the handler is registered for the bubbling phase. The full dispatch always runs capturing (root → target) first, regardless of what any individual listener requests, before bubbling-phase listeners get their turn. Option A describes what `{capture: true}` does. Option C is nonsensical — a listener is bound to one phase's behavior. Option D is too restrictive: a bubbling-phase listener on an ancestor fires too, once the event bubbles up to it, not only when it's the exact target.

</details>
::

::question-wrapper{language="javascript"}
### Q3. Given nested elements where `inner` is a direct child of `outer`, and the following listeners:

::code-wrapper{language="javascript"}
```javascript
outer.addEventListener("click", () => console.log("outer bubble"));
outer.addEventListener("click", () => console.log("outer capture"), { capture: true });

inner.addEventListener("click", () => console.log("inner capture"), { capture: true });
inner.addEventListener("click", () => console.log("inner bubble"));

inner.click();
```
::

What is the console output order?

- [ ] inner bubble, inner capture, outer bubble, outer capture
- [ ] outer bubble, outer capture, inner capture, inner bubble
- [ ] outer capture, inner capture, inner bubble, outer bubble
- [ ] inner capture, outer capture, outer bubble, inner bubble

<details>
<summary>Show Answer</summary>

**Answer:** C — outer capture, inner capture, inner bubble, outer bubble

**Explanation:** **Debug.** The capturing phase travels from the root down to (but not including firing bubble listeners at) the target, so `outer`'s capture-flagged listener fires first as the event descends toward `inner`. Once the event reaches the target element itself, the dispatch is in the "at target" phase — here, both of `inner`'s listeners fire regardless of their `capture` flag, in the order they were *registered*, which is why "inner capture" logs before "inner bubble" (it was added first in this snippet, not because capture flags dictate ordering at the target). Finally the bubbling phase carries the event back up, firing `outer`'s bubble-phase listener last. The key gotcha: at the target element, registration order — not the capture flag — decides the sequence.

</details>
::

::question-wrapper
### Q4. A `<button>` has two separate `click` listeners attached via `addEventListener`. The first one calls `event.stopPropagation()`; the second one does nothing special. Both are registered directly on the button (no capture option, no `stopImmediatePropagation`). When the button is clicked, what happens to the second listener?

- [ ] It never runs — `stopPropagation()` also stops other listeners on the same element
- [ ] Neither listener runs — `stopPropagation()` cancels the event entirely
- [ ] Only the second listener runs — `stopPropagation()` blocks the listener that called it from completing
- [ ] It still runs — `stopPropagation()` only stops bubbling to ancestor listeners, not sibling listeners on the same element; `stopImmediatePropagation()` would be needed to block it too

<details>
<summary>Show Answer</summary>

**Answer:** D — It still runs — `stopPropagation()` only stops bubbling to ancestor listeners, not sibling listeners on the same element; `stopImmediatePropagation()` would be needed to block it too

**Explanation:** **Debug.** `stopPropagation()` only prevents the event from continuing to travel to ancestor (or, during capturing, descendant) elements — it has no effect on other listeners already queued on the *same* element, which all still run in registration order. Only `stopImmediatePropagation()` additionally halts remaining same-element listeners. Option A is the classic mix-up between the two methods. Option B overstates the effect — propagation control never cancels the event object itself. Option C inverts which listener is affected.

</details>
::

::question-wrapper
### Q5. Calling `event.preventDefault()` inside a `click` handler on an `<a href="/page">` link does which of the following?

- [ ] Prevents the browser's default navigation to `/page`, but the event still bubbles to ancestor listeners unless `stopPropagation()` is also called
- [ ] Prevents the browser's default navigation to `/page`, and also stops the event from bubbling to ancestor listeners
- [ ] Has no effect unless the listener is registered on the capturing phase
- [ ] Cancels the entire event, so no other listener on the anchor element runs

<details>
<summary>Show Answer</summary>

**Answer:** A — Prevents the browser's default navigation to `/page`, but the event still bubbles to ancestor listeners unless `stopPropagation()` is also called

**Explanation:** `preventDefault()` and propagation control are two independent mechanisms: `preventDefault()` only suppresses whatever built-in action the browser would otherwise take (navigating, submitting a form, toggling a checkbox), while propagation continues normally unless `stopPropagation()`/`stopImmediatePropagation()` is called separately. Option B conflates the two, a very common assumption. Option C is wrong — `preventDefault()` works from either phase. Option D is wrong — it has no effect on other listeners at all.

</details>
::

::question-wrapper
### Q6. A `<ul>` contains 500 `<li>` items that get added and removed dynamically. Which approach is most robust for handling clicks on any `<li>`?

- [ ] Attach a separate `click` listener to every `<li>` when it's created
- [ ] Attach a single `click` listener to the `<ul>` and inspect `event.target` (or use `.closest()`) to determine which `<li>` was clicked
- [ ] Use `element.onclick` on each `<li>` instead of `addEventListener` since it's faster
- [ ] Poll `document.activeElement` on an interval to detect which `<li>` was interacted with

<details>
<summary>Show Answer</summary>

**Answer:** B — Attach a single `click` listener to the `<ul>` and inspect `event.target` (or use `.closest()`) to determine which `<li>` was clicked

**Explanation:** This is event delegation: because clicks bubble up from any descendant, one listener on a stable ancestor catches interactions from every current *and future* `<li>`, with no per-item wiring or cleanup needed when rows are added or removed. Option A requires re-attaching a listener on every re-render and doesn't scale to large or frequently-changing lists. Option C is false — `onclick` isn't inherently faster, and it still needs to be set per-element. Option D is unrelated to click detection and would miss most interactions entirely.

</details>
::

::question-wrapper{language="javascript"}
### Q7. A delegated click listener on a `<ul id="list">` is written as:

::code-wrapper{language="javascript"}
```javascript
list.addEventListener("click", (event) => {
  const item = event.target.closest("li[data-id]");
  if (!item) return;
  console.log("Selected:", item.dataset.id);
});
```
::

Each `<li data-id="42">` contains a `<button><span class="icon">×</span> Delete</button>`. A user clicks directly on the `<span class="icon">`. Why is `.closest()` needed here instead of using `event.target` directly?

- [ ] `event.target` is always the `<li>`, so `.closest()` is redundant
- [ ] `.closest()` is required because `event.target` does not exist inside delegated listeners
- [ ] `event.target` will be the innermost clicked element (the `<span>`), not the `<li>`; `.closest()` walks up from it to find the nearest matching ancestor (or itself)
- [ ] `.closest()` searches the descendants of `event.target`, which is needed to find the `<li>`'s children

<details>
<summary>Show Answer</summary>

**Answer:** C — `event.target` will be the innermost clicked element (the `<span>`), not the `<li>`; `.closest()` walks up from it to find the nearest matching ancestor (or itself)

**Explanation:** In a delegated listener, `event.target` is whatever the user actually clicked — frequently a deeply nested decorative element like an icon `<span>`, not the semantic row you actually care about. `.closest(selector)` starts at the element itself (inclusive) and searches upward through ancestors for the first match, correctly landing on the `<li data-id>` no matter how deeply the click originated inside it. Option A is false — `target` is the deepest node, not the delegated container's child. Option B is false — `event.target` always exists on the event object. Option D reverses the search direction; `.closest()` goes up the tree, never down into descendants.

</details>
::

::question-wrapper
### Q8. A `<div id="outer">` contains a nested `<button>`. A single `click` listener is attached to `#outer`. When a user clicks the button, inside that listener `event.target` and `event.currentTarget` are compared. Which statement is correct?

- [ ] Both always refer to the div, since that's where the listener is attached
- [ ] `event.target` is the div (the listener's element); `event.currentTarget` is the button (the clicked element)
- [ ] Both always refer to the button, since that's what was clicked
- [ ] `event.target` is the button (the element that actually triggered the event); `event.currentTarget` is the div (the element the listener is attached to) — the two stay fixed to their own definitions throughout the dispatch, and differ whenever the click originates on a descendant

<details>
<summary>Show Answer</summary>

**Answer:** D — `event.target` is the button (the element that actually triggered the event); `event.currentTarget` is the div (the element the listener is attached to) — the two stay fixed to their own definitions throughout the dispatch, and differ whenever the click originates on a descendant

**Explanation:** `event.target` is set once, at the very start of dispatch, to the innermost element the event originated on, and never changes for that dispatch. `event.currentTarget` instead tracks whichever element's listener is *currently executing*, which changes as the event moves through capturing and bubbling — inside this listener it's always `#outer`, since that's where the listener lives. This distinction matters when the same handler function is shared across multiple elements: `currentTarget` reliably tells you "which element I'm attached to," while `target` tells you "what was actually interacted with." Options A and B collapse this distinction incorrectly, and Option C wrongly assumes both track the clicked element.

</details>
::

::question-wrapper{language="javascript"}
### Q9. A developer writes:

::code-wrapper{language="javascript"}
```javascript
button.addEventListener("click", () => console.log("clicked"));
button.removeEventListener("click", () => console.log("clicked"));

button.click();
```
::

What happens?

- [ ] "clicked" logs once; the anonymous arrow function passed to `removeEventListener` is a different function reference, so nothing is actually removed
- [ ] Nothing logs; `removeEventListener` successfully matches and removes the listener
- [ ] It throws a `TypeError` because you can't remove a listener that hasn't fired yet
- [ ] "clicked" logs twice, once for the add and once for the mismatched remove

<details>
<summary>Show Answer</summary>

**Answer:** A — "clicked" logs once; the anonymous arrow function passed to `removeEventListener` is a different function reference, so nothing is actually removed

**Explanation:** **Debug.** `removeEventListener` only removes a listener when it's called with the exact same function reference (plus matching `type` and `capture` flag) that was originally passed to `addEventListener`. Two separately-written arrow functions are never `===` to each other, even with byte-for-byte identical bodies, so the "removal" silently matches nothing and the original listener stays active — no error, no warning. To make a listener removable, store it in a named variable and pass that same reference to both calls. Option C invents an error that never occurs; `removeEventListener` fails silently on non-matches. Option D misunderstands that a non-matching remove call has zero effect on the listener count.

</details>
::

::question-wrapper
### Q10. What does passing `{ once: true }` as the options argument to `addEventListener` do?

- [ ] It makes the listener run once per animation frame, throttling rapid events
- [ ] It automatically removes the listener after it has been invoked a single time
- [ ] It prevents the event from bubbling after the first invocation
- [ ] It defers the listener to run once the main thread is idle

<details>
<summary>Show Answer</summary>

**Answer:** B — It automatically removes the listener after it has been invoked a single time

**Explanation:** `{ once: true }` tells the browser to invoke the listener at most once, then internally call the equivalent of `removeEventListener` for it — useful for one-shot interactions like a dismiss button or a first-scroll trigger, without needing to manually clean up. Option A confuses it with `requestAnimationFrame`-based throttling, an unrelated technique. Option C confuses it with `stopPropagation()`. Option D confuses it with `requestIdleCallback` scheduling semantics, which `once` has nothing to do with.

</details>
::

::question-wrapper{language="javascript"}
### Q11. A component sets up several listeners tied to one controller:

::code-wrapper{language="javascript"}
```javascript
const controller = new AbortController();
const { signal } = controller;

button.addEventListener("click", onClick, { signal });
window.addEventListener("resize", onResize, { signal });
document.addEventListener("keydown", onKeydown, { signal });

controller.abort();
```
::

What is the effect of calling `controller.abort()`?

- [ ] Only the most recently added listener (`keydown`) is removed
- [ ] Nothing — `AbortController` only cancels `fetch` requests, not event listeners
- [ ] All three listeners are removed simultaneously, since they all share the same `signal`
- [ ] It throws because `signal` can only be used with one `addEventListener` call at a time

<details>
<summary>Show Answer</summary>

**Answer:** C — All three listeners are removed simultaneously, since they all share the same `signal`

**Explanation:** Modern `addEventListener` accepts a `signal` option, and a single `AbortController` can be shared across any number of listeners on any number of elements. Calling `abort()` once removes every listener registered with that signal in one shot, replacing the older pattern of manually calling `removeEventListener` for each one with matching references. Option B is outdated — `AbortSignal` is now a general-purpose cancellation primitive used by `fetch`, `addEventListener`, and other APIs. Options A and D misdescribe the sharing behavior; nothing limits a signal to a single listener.

</details>
::

::question-wrapper
### Q12. Why would you add `{ passive: true }` to a `touchstart` or `wheel` event listener that never calls `preventDefault()`?

- [ ] It makes the listener fire before capturing-phase listeners
- [ ] It automatically debounces rapid touch/wheel events for you
- [ ] It silently ignores any code inside the handler
- [ ] It tells the browser it can start scrolling immediately without waiting to see if the handler cancels it, improving scroll smoothness

<details>
<summary>Show Answer</summary>

**Answer:** D — It tells the browser it can start scrolling immediately without waiting to see if the handler cancels it, improving scroll smoothness

**Explanation:** **Performance.** Normally the browser must wait for a touch/wheel handler to finish running — in case it calls `preventDefault()` to cancel native scrolling — before it can start scrolling, which can cause visible jank if the main thread is busy. `{ passive: true }` is a promise that the handler will never call `preventDefault()`, letting the browser begin the native scroll or fling in parallel immediately. Some browsers even default `touchstart`/`touchmove` listeners to passive for exactly this reason. Options B and C describe behaviors `passive` doesn't have. Option A confuses it with the unrelated `capture` option.

</details>
::

::question-wrapper{language="javascript"}
### Q13. A listener is registered like this:

::code-wrapper{language="javascript"}
```javascript
document.addEventListener(
  "touchstart",
  (event) => {
    event.preventDefault();
  },
  { passive: true }
);
```
::

What happens when a user touches the screen?

- [ ] `preventDefault()` is silently ignored (default scrolling still happens), and most browsers log a console warning
- [ ] `preventDefault()` successfully blocks the default touch behavior (e.g., scrolling)
- [ ] The listener throws a `TypeError` because `preventDefault` is disabled on passive listeners
- [ ] The event stops firing entirely because `passive` and `preventDefault` conflict

<details>
<summary>Show Answer</summary>

**Answer:** A — `preventDefault()` is silently ignored (default scrolling still happens), and most browsers log a console warning

**Explanation:** **Debug.** `{ passive: true }` is a contract with the browser that the listener will never cancel the default action. Calling `preventDefault()` inside it doesn't throw, but the call becomes a no-op — the browser has already committed to proceeding with the default scroll without waiting, so nothing is actually blocked. Most browsers (e.g., Chrome) log a console warning such as "Unable to preventDefault inside passive event listener" to surface the mistake. Option C overstates the failure mode — it's silent, not an exception. Option D is wrong; the rest of the handler's code still runs, only the cancellation is dropped.

</details>
::

::question-wrapper{language="javascript"}
### Q14. Given `child` is a descendant of `parent`:

::code-wrapper{language="javascript"}
```javascript
parent.addEventListener("cart:updated", (e) => {
  console.log(e.detail.itemCount);
});

const event = new CustomEvent("cart:updated", {
  detail: { itemCount: 3 },
  bubbles: true,
});

child.dispatchEvent(event);
```
::

What logs?

- [ ] Nothing — `CustomEvent` instances never trigger `addEventListener` listeners
- [ ] `3` — the event was dispatched on `child`, and because `bubbles: true` was set, it propagates up to the `parent` listener, which reads it via `e.detail`
- [ ] `undefined` — `detail` is only accessible via `event.data`, not `event.detail`
- [ ] A `TypeError` — `dispatchEvent` cannot be used with a `CustomEvent`, only with built-in events like `Event`

<details>
<summary>Show Answer</summary>

**Answer:** B — `3` — the event was dispatched on `child`, and because `bubbles: true` was set, it propagates up to the `parent` listener, which reads it via `e.detail`

**Explanation:** `CustomEvent` lets application code build its own named events carrying an arbitrary payload in `detail`, and `dispatchEvent` runs it through the exact same capture/target/bubble pipeline as a real user-triggered event. Note the gotcha: unlike most native UI events (which bubble by default), `CustomEvent` (like the base `Event` constructor) defaults `bubbles` to `false` — it had to be explicitly set to `true` here for `parent`'s listener to ever see it. Option A is wrong — `dispatchEvent` synchronously invokes any matching listeners. Option C invents a nonexistent property name. Option D is false; `dispatchEvent` accepts any `Event` subtype, including `CustomEvent`.

</details>
::

::question-wrapper
### Q15. A team wants to detect when focus leaves a form's container entirely, using event delegation with a single listener on the `<form>` element. Which event should they use, and why?

- [ ] `blur`, because it fires on the form itself whenever any child input loses focus
- [ ] `focus`, because it's the bubbling counterpart to `blur`
- [ ] `focusout`, because unlike `blur`, it bubbles, so a single listener on the form catches focus changes from any descendant input
- [ ] Either `blur` or `focusout` work identically for delegation; the choice is just style preference

<details>
<summary>Show Answer</summary>

**Answer:** C — `focusout`, because unlike `blur`, it bubbles, so a single listener on the form catches focus changes from any descendant input

**Explanation:** **Debug.** `focus` and `blur` do not bubble by design, so a listener on a container only fires when that exact container element gains or loses focus — not its descendants. `focusin` and `focusout` are the bubbling equivalents, standardized specifically to support delegation, and are the correct choice here. Option A is the classic mistake: `blur` on the form fires only if the `<form>` element itself is directly focused, which rarely happens. Option B confuses `focus` (non-bubbling) with `focusin`. Option D is false — the two behave completely differently for delegation.

</details>
::

::question-wrapper
### Q16. A script runs, in order: `btn.onclick = fnA;` then `btn.onclick = fnB;` then `btn.addEventListener("click", fnC)`. When the button is clicked, which handlers run, and in what order?

- [ ] fnA, then fnB, then fnC — all three coexist because they were assigned at different times
- [ ] fnC, then fnB — `addEventListener`-registered handlers always run before the `onclick` property
- [ ] A `TypeError` is thrown — `onclick` and `addEventListener` cannot both be used on the same element
- [ ] fnB, then fnC — the second assignment to `onclick` silently overwrites the first (fnA never runs), while `addEventListener` maintains its own independent list and runs alongside it

<details>
<summary>Show Answer</summary>

**Answer:** D — fnB, then fnC — the second assignment to `onclick` silently overwrites the first (fnA never runs), while `addEventListener` maintains its own independent list and runs alongside it

**Explanation:** **Debug.** `element.onclick` is a plain property, not a collection — each assignment simply replaces whatever function was stored there, so only `fnB` (the last assignment) survives, with no error or warning that `fnA` was discarded. `addEventListener` maintains a completely separate internal list of listeners that can hold any number of handlers without clobbering the `onclick` property or each other. Option A wrongly assumes property assignment stacks like `addEventListener` does. Option B invents a precedence rule that doesn't exist. Option C is false — the two mechanisms coexist without conflict.

</details>
::

::question-wrapper{language="javascript"}
### Q17. Given:

::code-wrapper{language="javascript"}
```javascript
function handleClick() {
  console.log("clicked");
}

button.addEventListener("click", handleClick);
button.addEventListener("click", handleClick);

button.click();
```
::

How many times does `"clicked"` log?

- [ ] Once — `addEventListener` silently ignores a call that has the same type, function reference, and capture/options as one already registered
- [ ] Twice — each `addEventListener` call registers an independent listener
- [ ] Zero — registering the same listener twice throws and prevents both from being added
- [ ] Twice, but only if the second call includes `{ once: true }`

<details>
<summary>Show Answer</summary>

**Answer:** A — Once — `addEventListener` silently ignores a call that has the same type, function reference, and capture/options as one already registered

**Explanation:** **Debug.** Per spec, a call to `addEventListener` is treated as a no-op duplicate — and simply skipped — only when the event `type`, the exact function reference, and the `capture` flag all match an entry already registered on that element; here all three match `handleClick`, so the second call adds nothing and `"clicked"` logs once. This surprises developers who assume every call adds a new entry. It stops being a duplicate the moment any of the three differs — e.g., a different `capture` value, or a fresh arrow function created on each call, which is exactly why inline anonymous handlers *do* get registered multiple times. Option B is the natural-but-wrong assumption; Option C invents an error that never happens.

</details>
::

::question-wrapper
### Q18. A single-page app renders a large `<canvas>` chart and attaches `chartCanvas.addEventListener("mousemove", handleHover)`, where `handleHover` closes over a large `dataset` array. When the user navigates away, the component removes the canvas from the DOM via `chartCanvas.remove()` but never calls `removeEventListener`. What's the consequence?

- [ ] None — once an element is detached from the DOM, all of its event listeners and closures are automatically garbage collected
- [ ] The `chartCanvas` element — and the `dataset` array `handleHover` closes over — can be kept alive in memory as long as something still references `chartCanvas` or the listener, even though it's no longer visible on the page
- [ ] The browser throws an error on the next `mousemove` because the element is detached
- [ ] `handleHover` stops being called, but `dataset` is freed immediately regardless of other references

<details>
<summary>Show Answer</summary>

**Answer:** B — The `chartCanvas` element — and the `dataset` array `handleHover` closes over — can be kept alive in memory as long as something still references `chartCanvas` or the listener, even though it's no longer visible on the page

**Explanation:** **Performance.** Removing an element from the DOM does not make it eligible for garbage collection by itself — reclamation only happens once nothing reachable still references it. If `handleHover` (and its closure over `dataset`) is still attached to `chartCanvas`, and `chartCanvas` itself is still referenced anywhere (a variable, a cache, another closure), the whole chain — including the large `dataset` array — stays resident in memory, invisible on the page but present in the heap. This is the classic "detached DOM node" leak, visible in DevTools' heap snapshot tool. The fix is to call `removeEventListener` (or use a shared `AbortController`) before or when discarding the element. Option A is the false assumption most developers make. Options C and D describe behavior that doesn't occur — detached elements keep functioning normally as long as they're referenced, and nothing is freed just because rendering stopped.

</details>
::

::question-wrapper
### Q19. A table renders 10,000 rows, each with a "Delete" button, and rows are frequently added and removed via re-renders. Which listener strategy is best practice, and why?

- [ ] Attach one `addEventListener` to each "Delete" button — it's the most explicit and easiest to reason about
- [ ] Use inline `onclick="..."` HTML attributes on each button so there's no JS wiring needed
- [ ] Attach a single listener to the table (or its container) and use `event.target.closest("button")` to identify the clicked row — avoids thousands of listener objects and needs no rewiring when rows change
- [ ] Use `{ once: true }` on each button's listener so memory is automatically reclaimed after the first click

<details>
<summary>Show Answer</summary>

**Answer:** C — Attach a single listener to the table (or its container) and use `event.target.closest("button")` to identify the clicked row — avoids thousands of listener objects and needs no rewiring when rows change

**Explanation:** **Performance.** With 10,000 per-row listeners, both memory and setup cost scale linearly with row count, and every re-render that adds or removes rows must carefully add/remove matching listeners or risk leaking them. A single delegated listener on a stable ancestor costs one listener object regardless of row count, automatically covers rows added later (bubbling doesn't care when a descendant was created), and needs zero cleanup when rows are removed. Option A is exactly the anti-pattern delegation exists to solve. Option B mixes markup with behavior, supports only one handler per element, and still costs one binding per row. Option D is irrelevant — `once: true` only affects a single button's own first click and does nothing to address the scaling problem.

</details>
::

::question-wrapper{language="javascript"}
### Q20. Given `child` is nested inside `parent`:

::code-wrapper{language="javascript"}
```javascript
child.addEventListener("click", (e) => {
  console.log("child A");
  e.stopImmediatePropagation();
});
child.addEventListener("click", () => {
  console.log("child B");
});
parent.addEventListener("click", () => {
  console.log("parent");
});

child.click();
```
::

What logs?

- [ ] child A, child B, parent
- [ ] child A, parent — sibling listeners on the same element aren't affected, only propagation is stopped
- [ ] child B, parent — listeners run in reverse registration order, so B always wins the race before A calls stop
- [ ] child A only — `stopImmediatePropagation()` cancels both the remaining sibling listener on `child` and propagation up to `parent`

<details>
<summary>Show Answer</summary>

**Answer:** D — child A only — `stopImmediatePropagation()` cancels both the remaining sibling listener on `child` and propagation up to `parent`

**Explanation:** **Debug.** `stopImmediatePropagation()` is the strictest of the three propagation controls: like `stopPropagation()`, it halts further travel to ancestors, but it additionally prevents any other listener still queued on the *same* element from running at all. Since "child A" runs first (listeners fire in registration order) and calls it immediately, "child B" never fires, and the event never reaches `parent`'s listener either. Option A ignores that `stopImmediatePropagation()` was called. Option B describes plain `stopPropagation()`'s behavior, not the stricter method actually used here — the exact distinction this question tests. Option C invents a "reverse order" execution rule that doesn't exist; listeners always run in the order they were registered.

</details>
::
