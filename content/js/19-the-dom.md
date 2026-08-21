# 19 — The DOM

::question-wrapper
### Q1. A page has multiple `<li class="item">` elements inside a `<ul>`. What is the key structural difference between the results of `document.querySelectorAll('.item')` and `document.getElementsByClassName('item')`?

- [ ] Both return a live `HTMLCollection` that updates automatically as the DOM changes
- [ ] `querySelectorAll` returns a static `NodeList` snapshot taken at call time; `getElementsByClassName` returns a live `HTMLCollection` that reflects later DOM changes
- [ ] `querySelectorAll` returns a live `HTMLCollection`; `getElementsByClassName` returns a static `NodeList`
- [ ] Both return static arrays that never update, regardless of which method is used

<details>
<summary>Show Answer</summary>

**Answer:** B — `querySelectorAll` returns a static `NodeList` snapshot taken at call time; `getElementsByClassName` returns a live `HTMLCollection` that reflects later DOM changes

**Explanation:** `querySelectorAll` (and `querySelector`) always builds a static `NodeList` — a frozen snapshot of what matched at the moment the query ran, unaffected by later DOM mutations. The older `getElementsByClassName`, `getElementsByTagName`, and `getElementsByName` methods return live `HTMLCollection`s, which behave like a saved query that automatically re-evaluates as the document changes. Option C reverses the two — a very common mix-up. Options A and D collapse a real, load-bearing distinction that later questions build on.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does the second `console.log` print?

::code-wrapper{language="javascript"}
```javascript
const items = document.querySelectorAll('.item');
console.log(items.length); // 3

const li = document.createElement('li');
li.className = 'item';
document.querySelector('ul').appendChild(li);

console.log(items.length);
```
::

- [ ] `3` — `items` is a static `NodeList` captured at query time and does not reflect the element added afterward
- [ ] `4` — `querySelectorAll` always returns a live collection, just like `getElementsByClassName`
- [ ] `3`, then it throws a `TypeError` on the second access because the DOM changed underneath it
- [ ] `undefined` — reassigning `document.querySelector('ul')` invalidates the earlier `items` reference

<details>
<summary>Show Answer</summary>

**Answer:** A — `3` — `items` is a static `NodeList` captured at query time and does not reflect the element added afterward

**Explanation:** `querySelectorAll` freezes its result at call time. The new `<li>` appended afterward matches the original `.item` selector conceptually, but `items` was never wired up to re-run that query — it's just an array-like snapshot, so its `.length` stays `3` forever unless you call `querySelectorAll` again. Option B is the classic live-collection assumption bleeding over from `getElementsByClassName` (contrasted directly in Q3). Options C and D invent errors that don't occur — reading a stale static `NodeList` is completely safe, just potentially outdated.

</details>
::

::question-wrapper{language="javascript"}
### Q3. Using `getElementsByClassName` instead of `querySelectorAll` for the same scenario, what does the second `console.log` print?

::code-wrapper{language="javascript"}
```javascript
const items = document.getElementsByClassName('item');
console.log(items.length); // 3

const li = document.createElement('li');
li.className = 'item';
document.querySelector('ul').appendChild(li);

console.log(items.length);
```
::

- [ ] `3` — `HTMLCollection` becomes static once it's assigned to a variable
- [ ] `undefined` — `items` becomes stale as soon as the DOM mutates
- [ ] `4` — `getElementsByClassName` returns a live `HTMLCollection` that automatically reflects DOM changes matching the query
- [ ] `3`, then it throws because `items` was declared with `const`

<details>
<summary>Show Answer</summary>

**Answer:** C — `4` — `getElementsByClassName` returns a live `HTMLCollection` that automatically reflects DOM changes matching the query

**Explanation:** A live `HTMLCollection` isn't a snapshot at all — it's a view maintained by the DOM implementation that re-evaluates its matching criteria whenever the tree changes, so appending a new `.item` node immediately shows up in `items.length` with zero extra code. Option D confuses `const`'s restriction (you can't *reassign* the `items` binding) with the collection's *internal* mutability, which `const` has no effect on — the object itself is still free to change. Option A is exactly the incorrect assumption this question tests. Option B invents behavior that doesn't happen.

</details>
::

::question-wrapper{language="javascript"}
### Q4. Given 4 elements with class `"flagged"`, what actually happens when this loop runs?

::code-wrapper{language="javascript"}
```javascript
const items = document.getElementsByClassName('flagged');
for (let i = 0; i < items.length; i++) {
  items[i].classList.remove('flagged');
}
```
::

- [ ] It removes the class from all 4 elements as expected, then exits cleanly
- [ ] It throws a `RangeError` because `items.length` changes while the loop is running
- [ ] It removes the class from only the last 2 elements, leaving the first 2 unchanged
- [ ] It removes the class from only every other element — removing `flagged` from `items[0]` shrinks the live collection and shifts the next element into index 0, which the incrementing loop then skips over

<details>
<summary>Show Answer</summary>

**Answer:** D — It removes the class from only every other element — removing `flagged` from `items[0]` shrinks the live collection and shifts the next element into index 0, which the incrementing loop then skips over

**Explanation:** `getElementsByClassName` returns a live `HTMLCollection` (Q3), so the moment `items[0]` loses the `flagged` class, it drops out of the collection entirely — every remaining matched element shifts down one index, and `items.length` shrinks by one. But the loop's `i` still increments to `1`, which now points at what used to be `items[2]`, silently skipping the element that shifted into slot `0`. **Debug:** this is the canonical live-collection mutation bug — fix it by iterating a static copy (`Array.from(items)`), snapshotting the length up front, or walking backwards from the end. Options A and B are the naive assumptions; C names a plausible-sounding but incorrect specific outcome.

</details>
::

::question-wrapper
### Q5. A developer writes `document.getElementsByTagName('img').forEach(img => img.loading = 'lazy')` and gets `TypeError: items.forEach is not a function`. Why, and what's the correct fix?

- [ ] `forEach` was removed from all DOM collections in newer browsers; a `for...of` loop is now required universally
- [ ] `HTMLCollection` (returned by `getElementsByTagName`) never implemented `forEach`, unlike `NodeList` from `querySelectorAll`; convert it first with `Array.from(collection)` or the spread `[...collection]`
- [ ] The code should call `.values().forEach(...)`, because `HTMLCollection` only exposes an iterator, not array methods
- [ ] `img` elements are specifically excluded from `forEach` for security reasons; other tags work fine

<details>
<summary>Show Answer</summary>

**Answer:** B — `HTMLCollection` (returned by `getElementsByTagName`) never implemented `forEach`, unlike `NodeList` from `querySelectorAll`; convert it first with `Array.from(collection)` or the spread `[...collection]`

**Explanation:** `NodeList.prototype` was given `forEach` directly, so a `querySelectorAll` result supports it out of the box. `HTMLCollection` is a different, older interface exposing only `length`, indexed access, and `namedItem` — it was never extended with array methods, so calling `.forEach` on it throws. `Array.from(collection)` or `[...collection]` produces a real array with the full method set (`forEach`, `map`, `filter`, and so on). Option A is false — `NodeList` still has `forEach`. Option C invents a nonexistent API surface. Option D is nonsensical; there's no tag-based restriction.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What does the second `console.log` print, and why does it complicate the "`NodeList` = static" takeaway from Q2?

::code-wrapper{language="javascript"}
```javascript
const container = document.querySelector('#box');
const nodes = container.childNodes;
console.log(nodes.length); // 2

container.appendChild(document.createElement('span'));
console.log(nodes.length);
```
::

- [ ] `3` — unlike `querySelectorAll`'s result, `.childNodes` returns a *live* `NodeList` that updates as children are added or removed
- [ ] `2` — every `NodeList`, including `.childNodes`, is a static snapshot
- [ ] `3` — because `.childNodes` actually returns an `HTMLCollection`, not a `NodeList`, despite its name
- [ ] It throws, because `nodes` was assigned before the DOM mutation occurred

<details>
<summary>Show Answer</summary>

**Answer:** A — `3` — unlike `querySelectorAll`'s result, `.childNodes` returns a *live* `NodeList` that updates as children are added or removed

**Explanation:** "Static vs. live" is a property of *how a collection is produced*, not of the `NodeList` type itself. `querySelectorAll` is specified to always build a static snapshot, but other DOM properties — `.childNodes` chief among them — are specified to return a live `NodeList` that tracks the tree in real time. So "`NodeList`" and "static" are not synonyms, which trips up anyone who over-generalizes from `querySelectorAll` alone. Option C is wrong on the interface type — it genuinely is a `NodeList`, just a live one. Option B is the over-generalization this question targets. Option D fabricates an error that doesn't occur.

</details>
::

::question-wrapper
### Q7. In browser rendering, what is the actual difference between a "reflow" (layout) and a "repaint"?

- [ ] Reflow only affects `<canvas>` elements; repaint affects every other element
- [ ] They are two names for the exact same browser operation
- [ ] Reflow recalculates element geometry and position, and can cascade to affect ancestors, descendants, and siblings; repaint only redraws pixels (e.g., a color change) without recomputing any layout, which makes it cheaper
- [ ] Repaint happens on every `scroll` event; reflow only ever happens once, on initial page load

<details>
<summary>Show Answer</summary>

**Answer:** C — Reflow recalculates element geometry and position, and can cascade to affect ancestors, descendants, and siblings; repaint only redraws pixels (e.g., a color change) without recomputing any layout, which makes it cheaper

**Explanation:** Reflow (layout) computes size and position for elements in the render tree, and because layout is fundamentally a tree-wide computation, a change to one element's box can ripple outward to affect its neighbors. Repaint (paint) only redraws the *appearance* of already-laid-out pixels — `background-color`, `visibility`, and similar properties — reusing the existing geometry, so it skips the expensive remeasurement step entirely. Changing something like `width` or `left` triggers a reflow (and a subsequent repaint); changing only `color` triggers just a repaint. Options A and D fabricate rules that don't reflect how the rendering pipeline actually works; option B collapses a distinction that matters a great deal for performance, as the next questions show.

</details>
::

::question-wrapper{language="javascript"}
### Q8. Why is this loop considered "layout thrashing," and what makes it especially expensive?

::code-wrapper{language="javascript"}
```javascript
const boxes = document.querySelectorAll('.box');
boxes.forEach(box => {
  box.style.width = '200px';
  console.log(box.offsetHeight);
});
```
::

- [ ] `offsetHeight` is deprecated and logs a console warning on every access
- [ ] `.style.width` writes are batched automatically by the browser, so this pattern is actually efficient as written
- [ ] `querySelectorAll` re-runs the CSS selector engine on every loop iteration, which is the real cost here
- [ ] Writing `.style.width` invalidates the cached layout, and immediately reading `offsetHeight` forces the browser to synchronously recompute layout right then instead of deferring it — repeating this write-then-read pattern for every element forces a full synchronous reflow on *every single iteration*

<details>
<summary>Show Answer</summary>

**Answer:** D — Writing `.style.width` invalidates the cached layout, and immediately reading `offsetHeight` forces the browser to synchronously recompute layout right then instead of deferring it — repeating this write-then-read pattern for every element forces a full synchronous reflow on *every single iteration*

**Explanation:** Browsers normally batch layout-invalidating writes and defer the actual recalculation until it's genuinely needed (typically right before the next paint). But certain "layout-dependent" reads — `offsetHeight`, `offsetWidth`, `getBoundingClientRect()`, `scrollTop`, computed styles — force an immediate, synchronous flush of any pending layout work so the browser can hand back an accurate number. Interleaving a write then a read inside a loop defeats that batching completely, forcing *N* synchronous reflows for *N* elements instead of just one. **Performance:** this "layout thrashing" pattern is one of the most common real-world sources of janky UI code. Options A and C are fabricated costs; option B is the literal opposite of what happens in this snippet.

</details>
::

::question-wrapper{language="javascript"}
### Q9. How does this rewrite of the Q8 pattern avoid layout thrashing?

::code-wrapper{language="javascript"}
```javascript
const boxes = document.querySelectorAll('.box');

const heights = [];
boxes.forEach(box => heights.push(box.offsetHeight));

boxes.forEach((box, i) => {
  box.style.width = heights[i] > 100 ? '200px' : '100px';
});
```
::

- [ ] It doesn't actually help — reading `offsetHeight` in the first loop still forces a reflow for every element, regardless of write timing
- [ ] It separates every layout-triggering *read* into one pass, before any layout-invalidating *write* happens; the browser only needs a single reflow to satisfy the whole read pass, and the later write pass never needs to be flushed synchronously because nothing reads from layout afterward
- [ ] It works because `querySelectorAll` automatically caches each element's `offsetHeight` the first time it's accessed
- [ ] It works because `.forEach` runs asynchronously, giving the browser idle time to repaint between the reads and the writes

<details>
<summary>Show Answer</summary>

**Answer:** B — It separates every layout-triggering *read* into one pass, before any layout-invalidating *write* happens; the browser only needs a single reflow to satisfy the whole read pass, and the later write pass never needs to be flushed synchronously because nothing reads from layout afterward

**Explanation:** This is the "read-then-write" batching idiom (the core idea behind libraries like FastDOM). Since nothing has invalidated layout yet when the first loop reads `offsetHeight`, at most one reflow satisfies the entire batch of reads. The second loop's writes can then all queue up freely, because no subsequent read in this code forces the browser to flush them synchronously. **Performance:** one reflow for a batch beats *N* reflows for *N* elements by a wide margin as the list grows. Option A misses that a single triggered reflow is vastly cheaper than repeating it per element. Options C and D invent mechanisms that don't exist — `querySelectorAll` doesn't cache layout metrics, and `forEach` is fully synchronous.

</details>
::

::question-wrapper{language="javascript"}
### Q10. How should this loop be rewritten to minimize reflows when inserting 1,000 items?

::code-wrapper{language="javascript"}
```javascript
const list = document.querySelector('#list');
for (let i = 0; i < 1000; i++) {
  const li = document.createElement('li');
  li.textContent = `Item ${i}`;
  list.appendChild(li);
}
```
::

- [ ] Build every `<li>` into a `DocumentFragment` first, then append the fragment to `list` once after the loop — the fragment lives outside the render tree, so populating it triggers no reflows, and only the single final append touches the live DOM
- [ ] Nothing needs to change — `appendChild` calls inside a loop are already batched by the browser automatically
- [ ] Replace `appendChild` with `insertAdjacentHTML('beforeend', ...)` inside the loop, since string-based insertion never triggers layout
- [ ] Wrap the loop body in `requestAnimationFrame` so each insertion happens on its own frame

<details>
<summary>Show Answer</summary>

**Answer:** A — Build every `<li>` into a `DocumentFragment` first, then append the fragment to `list` once after the loop — the fragment lives outside the render tree, so populating it triggers no reflows, and only the single final append touches the live DOM

**Explanation:** A `DocumentFragment` is a lightweight, in-memory container that is never part of the visible document tree, so appending 1,000 children to it costs no layout or paint work at all. Only the single fragment-into-`list` append potentially triggers a reflow — one, instead of up to 1,000. **Performance:** this is the standard batching idiom for bulk DOM insertion. Option B is false: each `appendChild` call directly onto a connected `list` element is a live-DOM mutation, each one a potential invalidation. Option C still mutates the live DOM on every iteration, with the added cost of re-parsing an HTML string each time. Option D would spread 1,000 insertions across 1,000 separate animation frames — far slower and visibly janky, not faster.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What is the primary risk with this function if `userSuppliedComment` comes from another user's input (e.g., a public comment form), and what's the safer alternative?

::code-wrapper{language="javascript"}
```javascript
function showComment(rawText) {
  const el = document.querySelector('#comment');
  el.innerHTML = rawText;
}
showComment(userSuppliedComment);
```
::

- [ ] `innerHTML` is merely slower than `textContent`, so the only real issue here is performance, not correctness
- [ ] There's no real risk — browsers automatically strip any `<script>` tags assigned via `innerHTML`
- [ ] Assigning untrusted input to `innerHTML` parses it as real HTML, so a comment like `<img src=x onerror="steal()">` executes arbitrary JavaScript in the page (a classic XSS vector); use `el.textContent = rawText` instead, which inserts the string as a raw text node with no HTML parsing at all
- [ ] The fix is to use `innerText` instead of `innerHTML`, since `innerText` automatically escapes any HTML in the string

<details>
<summary>Show Answer</summary>

**Answer:** C — Assigning untrusted input to `innerHTML` parses it as real HTML, so a comment like `<img src=x onerror="steal()">` executes arbitrary JavaScript in the page (a classic XSS vector); use `el.textContent = rawText` instead, which inserts the string as a raw text node with no HTML parsing at all

**Explanation:** **Safety:** `innerHTML` hands its string directly to the HTML parser, so any markup or event-handler attribute embedded in untrusted input becomes real, executing HTML/JS — this is exactly how stored and reflected XSS attacks work in practice. `textContent` never parses its argument as markup; the string becomes a literal text node, so `<img ...>` shows up as visible, inert text rather than an executed tag. Option B is a dangerous misconception — browsers do not sanitize `innerHTML` input by default; that's entirely the caller's responsibility. Option D is wrong: `innerText` is about *rendered* text and layout, not escaping — writing to it has the same "no HTML parsing" property as `textContent`, but it isn't the idiomatic choice here because of its own layout costs (see Q12).

</details>
::

::question-wrapper
### Q12. `element.textContent = 'Hello'` and `element.innerText = 'Hello'` both set visible text, but why is `textContent` generally preferred in performance-sensitive code?

- [ ] `innerText` cannot hold more than 255 characters without silently truncating
- [ ] `textContent` is deprecated in favor of `innerText` in every modern browser
- [ ] `innerText` parses its argument as HTML while `textContent` does not
- [ ] `innerText` is defined in terms of the *rendered* page — it must account for applied CSS (e.g., skipping `display: none` content, normalizing whitespace to match what's visually shown) — so reading or writing it can force a synchronous reflow; `textContent` operates purely on the raw text nodes in the DOM tree and never touches layout

<details>
<summary>Show Answer</summary>

**Answer:** D — `innerText` is defined in terms of the *rendered* page — it must account for applied CSS (e.g., skipping `display: none` content, normalizing whitespace to match what's visually shown) — so reading or writing it can force a synchronous reflow; `textContent` operates purely on the raw text nodes in the DOM tree and never touches layout

**Explanation:** **Performance:** because `innerText` reflects what a user would actually *see* — excluding hidden elements, collapsing whitespace the way rendering would, respecting line breaks introduced by CSS — the engine needs up-to-date layout information to compute or apply it, which can trigger the same kind of synchronous reflow flagged back in Q8. `textContent` simply walks the raw text-node tree, entirely layout-agnostic, making it both faster and more predictable. Option C is backwards — neither property parses HTML; that's `innerHTML`'s job exclusively. Options A and B are fabricated.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What happens when `copy.click()` runs?

::code-wrapper{language="javascript"}
```javascript
const original = document.querySelector('#save-btn');
original.addEventListener('click', () => console.log('saved'));

const copy = original.cloneNode(true);
document.body.appendChild(copy);
copy.click();
```
::

- [ ] It logs `"saved"` — `cloneNode(true)` performs a deep clone that includes all attached event listeners
- [ ] Nothing is logged — `cloneNode` copies the element's tag, attributes, and (with `true`) its descendant nodes, but it never copies listeners attached via `addEventListener`, since those live outside the node's cloneable attribute/child data
- [ ] It throws a `TypeError`, because a cloned node cannot dispatch synthetic events like `.click()`
- [ ] It logs `"saved"` twice — once for the original element's listener and once for a listener copy on the clone

<details>
<summary>Show Answer</summary>

**Answer:** B — Nothing is logged — `cloneNode` copies the element's tag, attributes, and (with `true`) its descendant nodes, but it never copies listeners attached via `addEventListener`, since those live outside the node's cloneable attribute/child data

**Explanation:** **Debug:** `cloneNode(true)` performs a deep clone, duplicating the tag, every attribute, and all descendant nodes recursively; `cloneNode(false)` (or `cloneNode()` with no argument) does the same but shallow, omitting children entirely. In neither case does cloning touch the internal listener list an `EventTarget` accumulates via `addEventListener` — that bookkeeping is stored on the live object, not in the serializable DOM structure the clone algorithm copies. Any `addEventListener`-based behavior has to be reattached to the clone manually. Option A is an extremely common wrong assumption. Options C and D fabricate behavior — `.click()` works fine on any connected element, and there's no listener present on the clone at all to fire twice.

</details>
::

::question-wrapper
### Q14. A toggle-visibility helper is implemented two ways. Version 1: `el.className = el.className.includes('hidden') ? el.className.replace('hidden', '') : el.className + ' hidden'`. Version 2: `el.classList.toggle('hidden')`. Why is Version 2 the idiomatic choice?

- [ ] `classList.toggle` handles the add/remove logic atomically and correctly regardless of existing whitespace or class order, while manual `className` string manipulation is error-prone — `.replace('hidden', '')` is a plain substring match that can corrupt an unrelated class like `"overhidden"`, and repeated concatenation without a duplicate check can pile up the same class many times over
- [ ] `className` was removed from the DOM specification in favor of `classList`, so Version 1 no longer runs in any browser
- [ ] `classList.toggle` is the only one of the two that triggers a repaint; assigning to `className` does not update the visible page at all
- [ ] Version 1 is actually preferable for performance, since it avoids the overhead of the `DOMTokenList` object that `classList` allocates

<details>
<summary>Show Answer</summary>

**Answer:** A — `classList.toggle` handles the add/remove logic atomically and correctly regardless of existing whitespace or class order, while manual `className` string manipulation is error-prone — `.replace('hidden', '')` is a plain substring match that can corrupt an unrelated class like `"overhidden"`, and repeated concatenation without a duplicate check can pile up the same class many times over

**Explanation:** **Idiom:** `classList` exposes a `DOMTokenList` with `add`/`remove`/`toggle`/`contains` that operate correctly on the space-separated token list — `toggle('hidden')` is guaranteed to leave the element in exactly one consistent state, with no malformed whitespace and no duplicate tokens. Direct `className` string manipulation is a classic source of subtle bugs: `.replace('hidden', '')` performs a bare substring match that can silently mangle an unrelated class name that merely *contains* "hidden" as a substring, and naive concatenation without checking `includes` first can accumulate the same class repeatedly if the function runs more than once. Option B is false — `className` remains fully valid, just less ergonomic. Options C and D are fabricated technical claims.

</details>
::

::question-wrapper{language="javascript"}
### Q15. Given `<div id="user-card" data-user-id="482" data-is-verified="true"></div>`, what is the correct way to read the `data-user-id` attribute via the `dataset` API, and what type comes back?

::code-wrapper{language="javascript"}
```javascript
// <div id="user-card" data-user-id="482" data-is-verified="true"></div>
const card = document.querySelector('#user-card');
console.log(card.dataset.userId, typeof card.dataset.userId);
```
::

- [ ] `card.dataset['data-user-id']`, and it comes back as a `Number`
- [ ] `card.dataset.data_user_id`, and it comes back as a `Boolean`
- [ ] `card.dataset.userId`, and it comes back as a `String` (`"482"`) — `dataset` always yields strings, so numeric-looking values need explicit conversion, e.g. `Number(card.dataset.userId)`
- [ ] `card.getAttribute('dataset.userId')`, and it comes back as a `String`

<details>
<summary>Show Answer</summary>

**Answer:** C — `card.dataset.userId`, and it comes back as a `String` (`"482"`) — `dataset` always yields strings, so numeric-looking values need explicit conversion, e.g. `Number(card.dataset.userId)`

**Explanation:** The `dataset` API automatically converts a hyphen-separated `data-*` attribute name into camelCase for property access: `data-user-id` becomes `dataset.userId`, and `data-is-verified` becomes `dataset.isVerified`. Every value comes back as a plain string regardless of how it looks — `"482"`, not `482`; `"true"`, not `true` — so callers must explicitly coerce with `Number(...)` or compare `=== 'true'` as needed. Option A uses the wrong key format (it should be camelCase, not the literal hyphenated attribute name as a bracket key) and the wrong type assumption. Option B uses invalid snake_case syntax that the conversion rule doesn't produce. Option D conflates `dataset` (a property) with `getAttribute` (a method that takes the literal attribute name, not a dataset key path).

</details>
::

::question-wrapper{language="javascript"}
### Q16. What does `el.outerHTML` show after this assignment?

::code-wrapper{language="javascript"}
```javascript
const el = document.createElement('div');
el.dataset.orderStatus = 'pending';
console.log(el.outerHTML);
```
::

- [ ] `<div dataset-order-status="pending"></div>`
- [ ] `<div data-orderStatus="pending"></div>`
- [ ] `<div orderStatus="pending"></div>`
- [ ] `<div data-order-status="pending"></div>` — writing to a camelCase `dataset` property converts it back into a hyphen-separated `data-*` attribute on the element

<details>
<summary>Show Answer</summary>

**Answer:** D — `<div data-order-status="pending"></div>` — writing to a camelCase `dataset` property converts it back into a hyphen-separated `data-*` attribute on the element

**Explanation:** The `dataset` conversion (introduced in Q15) is symmetric: reading turns `data-order-status` into `dataset.orderStatus`, and writing to `dataset.orderStatus` creates or updates the `data-order-status` attribute, inserting a hyphen before each formerly-uppercase letter and lowercasing it. Option A wrongly assumes the word `dataset` itself becomes part of the attribute name. Option B fails to convert the camelCase key back into kebab-case. Option C drops the required `data-` prefix entirely, which would just create a non-standard, untracked custom attribute that `dataset` would never pick back up.

</details>
::

::question-wrapper{language="javascript"}
### Q17. If `removeRow(row)` is called to remove a row from the visible table, but `cachedRows` still holds a reference to it, what happens?

::code-wrapper{language="javascript"}
```javascript
let cachedRows = [];

function removeRow(row) {
  row.remove();
}

function renderRow(data) {
  const row = document.createElement('tr');
  row.textContent = data.label;
  row.addEventListener('click', () => console.log(data.id));
  cachedRows.push(row);
  document.querySelector('tbody').appendChild(row);
  return row;
}
```
::

- [ ] The browser automatically clears `row` out of `cachedRows` once it's removed from the DOM, since detached nodes are garbage-collected immediately
- [ ] The row becomes a "detached" DOM node — no longer part of the visible document tree, but still reachable through the `cachedRows` array — so it, its closure over `data`, and its click listener can never be garbage-collected; repeating this pattern for many rows leaks memory
- [ ] `row.remove()` throws an error if any other references to that node still exist elsewhere in the program
- [ ] The click listener is automatically stripped by `.remove()`, so only the empty node itself leaks, which is negligible

<details>
<summary>Show Answer</summary>

**Answer:** B — The row becomes a "detached" DOM node — no longer part of the visible document tree, but still reachable through the `cachedRows` array — so it, its closure over `data`, and its click listener can never be garbage-collected; repeating this pattern for many rows leaks memory

**Explanation:** **Performance:** `.remove()` only detaches a node from its parent in the render tree — it does nothing to any *other* JavaScript reference pointing at that same node object. As long as something (an array, a closure, an event registry) still holds it, the garbage collector must keep the entire reachable object graph alive, including the `addEventListener` closure capturing `data`. This is a genuinely common leak pattern in single-page apps that cache row/item references without cleaning them up alongside DOM removal. Options A and C invent automatic cleanup that doesn't exist — the GC only reclaims objects that are truly unreachable. Option D wrongly assumes `.remove()` also strips listeners; it doesn't — they persist on the detached node and would even still fire if the node were ever re-appended.

</details>
::

::question-wrapper
### Q18. When inserting new UI content that includes some user-controlled text (e.g., a username), what's the key tradeoff between `document.createElement(...)` + `appendChild(...)` versus `element.insertAdjacentHTML('beforeend', markup)`?

- [ ] `createElement`/`appendChild` requires more code but is inherently safe for user-controlled values when the dynamic part is assigned via `textContent` (no HTML parsing occurs); `insertAdjacentHTML` is more concise, but it parses its argument as raw HTML, so any user-controlled value interpolated into that string must be escaped manually or it becomes an XSS vector
- [ ] Both approaches are equally safe by default — the only real difference is that `insertAdjacentHTML` is slightly slower due to extra function-call overhead
- [ ] `createElement` cannot set visible text at all, only attributes, so `insertAdjacentHTML` is required whenever text content is involved
- [ ] `insertAdjacentHTML` is always the safer choice, because the browser automatically escapes any interpolated template values before parsing the markup

<details>
<summary>Show Answer</summary>

**Answer:** A — `createElement`/`appendChild` requires more code but is inherently safe for user-controlled values when the dynamic part is assigned via `textContent` (no HTML parsing occurs); `insertAdjacentHTML` is more concise, but it parses its argument as raw HTML, so any user-controlled value interpolated into that string must be escaped manually or it becomes an XSS vector

**Explanation:** Building nodes programmatically and assigning the dynamic part through `textContent` keeps user data inert by construction — there's no parsing step where markup could ever be interpreted as tags. `insertAdjacentHTML` hands its string straight to the HTML parser, so a template like `` `<li>${username}</li>` `` lets a `username` containing `<img src=x onerror=...>` execute exactly like the `innerHTML` case in Q11. The real tradeoff: `createElement` is verbose (multiple statements per element), while `insertAdjacentHTML` is compact but pushes all escaping responsibility onto the caller. Options B and D understate or misstate the risk — neither API escapes anything automatically. Option C is simply false; elements built with `createElement` set visible text via `.textContent` (or `.innerText`) without issue.

</details>
::

::question-wrapper{language="javascript"}
### Q19. Given `<input id="email" type="text" value="default@example.com">`, after the user clears the field and types `"new@example.com"`, what do the two logs print?

::code-wrapper{language="javascript"}
```javascript
// <input id="email" type="text" value="default@example.com">
const input = document.querySelector('#email');
// user clears the field and types "new@example.com"
console.log(input.value);
console.log(input.getAttribute('value'));
```
::

- [ ] Both print `"new@example.com"`, since the property and the attribute are always kept in sync
- [ ] Both print `"default@example.com"`, since neither updates from user interaction, only from explicit JS assignment
- [ ] `input.value` prints `"new@example.com"` — the live property reflecting what the user actually typed; `input.getAttribute('value')` prints `"default@example.com"` — the original HTML attribute, which reflects only the initial/default value and does not track live user input
- [ ] `input.value` throws, because reading the property after user interaction requires calling `input.reportValidity()` first

<details>
<summary>Show Answer</summary>

**Answer:** C — `input.value` prints `"new@example.com"` — the live property reflecting what the user actually typed; `input.getAttribute('value')` prints `"default@example.com"` — the original HTML attribute, which reflects only the initial/default value and does not track live user input

**Explanation:** **Debug:** for form controls, the `value` *attribute* (what's literally written in the markup, and what `getAttribute('value')` reads) represents only the initial/default value — it stays frozen at the page's original state unless code explicitly calls `setAttribute`. The `value` *property* is live and tracks the control's current state continuously as the user types. Most reflected attributes (like `id`) stay perfectly in sync with their property, which is exactly why this divergence for a handful of stateful form properties (`value`, `checked`) catches people off guard — a common bug is resetting a form by reading `getAttribute('value')` and expecting it to reflect what's currently on screen. Options A and B both wrongly assume lockstep behavior; option D fabricates an error that doesn't exist.

</details>
::

::question-wrapper
### Q20. A widget needs to react whenever nodes are added to a specific container, without knowing in advance which code will trigger the change. Why is `MutationObserver` preferred over the older Mutation Events (e.g., `DOMNodeInserted`) or manual `setInterval` polling?

- [ ] `MutationObserver` runs synchronously on every single DOM mutation, giving instant, per-change callbacks
- [ ] Manual polling with `setInterval` is actually more efficient, since it avoids the overhead of registering an observer at all
- [ ] Mutation Events are still the recommended approach in every modern browser; `MutationObserver` was an experimental API that never actually shipped
- [ ] `MutationObserver` delivers batched, asynchronous notifications — queued as a microtask after the relevant DOM changes finish — instead of firing synchronously for every individual mutation the way the deprecated Mutation Events did; this avoids the severe performance cost that got Mutation Events deprecated, while still being far more efficient and timely than polling with `setInterval`

<details>
<summary>Show Answer</summary>

**Answer:** D — `MutationObserver` delivers batched, asynchronous notifications — queued as a microtask after the relevant DOM changes finish — instead of firing synchronously for every individual mutation the way the deprecated Mutation Events did; this avoids the severe performance cost that got Mutation Events deprecated, while still being far more efficient and timely than polling with `setInterval`

**Explanation:** **Performance:** the old Mutation Events fired a synchronous DOM event for *every single mutation*, which could itself trigger more mutations and more events, cascading into severe performance degradation on real pages — this is precisely why browsers deprecated them. `MutationObserver` instead collects mutations into a batch and delivers them together in one callback as a microtask, which is both cheaper (no per-change event-dispatch overhead, changes get coalesced) and safer (no synchronous reentrancy hazard). It also strictly beats `setInterval` polling, which either misses fast changes (interval too long) or burns CPU on redundant checks (interval too short) — the observer only fires when something actually changed. Option A misdescribes it as synchronous per-mutation, which is the old, deprecated behavior it was designed to replace. Options B and C are simply false.

</details>
::
