# 21 — Web Storage & APIs

::question-wrapper
### Q1. Which statement correctly distinguishes `localStorage` from `sessionStorage`?

- [ ] `localStorage` data is cleared when the browser closes; `sessionStorage` persists indefinitely
- [ ] `localStorage` persists across browser restarts until explicitly cleared; `sessionStorage` is cleared when the tab closes
- [ ] Both persist indefinitely, but `sessionStorage` is shared across all tabs of the same origin
- [ ] `localStorage` is per-tab; `sessionStorage` is shared across the whole browser

<details>
<summary>Show Answer</summary>

**Answer:** B — `localStorage` persists across browser restarts until explicitly cleared; `sessionStorage` is cleared when the tab closes

**Explanation:** `localStorage` has no expiration and survives browser restarts, page reloads, and navigation — it's only removed by explicit `removeItem`/`clear()`, the user clearing site data, or the browser evicting it. `sessionStorage` is scoped to a single tab's lifetime: closing that tab destroys it, even though a page reload within the same tab preserves it. Option A inverts the two. Option C is wrong because `sessionStorage` is explicitly *not* shared between tabs, even same-origin ones opened to the same URL — each tab gets its own isolated storage area. Option D is backwards.

</details>
::

::question-wrapper
### Q2. A team stores a JWT auth token in a cookie versus `localStorage`. Which behavior is unique to cookies (not `localStorage`)?

- [ ] The value is limited to strings
- [ ] The value is automatically sent with every same-origin HTTP request to the server (unless flagged otherwise)
- [ ] The value can be read via JavaScript's `document.cookie` or storage APIs
- [ ] The value is scoped to a single origin

<details>
<summary>Show Answer</summary>

**Answer:** B — The value is automatically sent with every same-origin HTTP request to the server (unless flagged otherwise)

**Explanation:** Cookies are attached to outgoing HTTP requests automatically by the browser (governed by `Domain`, `Path`, `SameSite`, and `Secure` attributes), which is why they're used for session identifiers the server needs to see. `localStorage` is never transmitted automatically — a page must explicitly read it and attach it (e.g., as an `Authorization` header). Option A is wrong because both cookies and `localStorage` only store strings. Option C is wrong because both are readable via JS unless a cookie is marked `HttpOnly`, which then hides it from `document.cookie`. Option D is wrong because both are origin-scoped (cookies also add `Domain`/`Path` scoping on top).

</details>
::

::question-wrapper{language="javascript"}
### Q3. What happens when you run the following code?

::code-wrapper{language="javascript"}
```javascript
const user = { name: "Ravi", age: 30 };
localStorage.setItem("user", user);
console.log(localStorage.getItem("user"));
```
::

- [ ] It logs `{ name: "Ravi", age: 30 }` as a live object
- [ ] It throws a `TypeError` because objects can't be stored
- [ ] It logs the string `"[object Object]"`
- [ ] It logs `undefined`

<details>
<summary>Show Answer</summary>

**Answer:** C — It logs the string `"[object Object]"`

**Explanation:** The Web Storage API only stores strings. `setItem` coerces any non-string value using `String(value)`, and the default string conversion of a plain object is `"[object Object]"` (via `Object.prototype.toString`), silently discarding the actual data. **Idiom:** always `JSON.stringify` before storing and `JSON.parse` after reading — there's no error to warn you otherwise, which makes this a common silent-data-loss bug. Option B is tempting because it feels like it *should* error, but `setItem` never validates the value's type, it just stringifies it.

</details>
::

::question-wrapper
### Q4. When does the `storage` event fire on the `window` object?

- [ ] In every tab, including the one that made the change
- [ ] Only in other tabs/windows of the same origin, not the tab that made the change
- [ ] Only when `sessionStorage` changes, never for `localStorage`
- [ ] Only when the page is reloaded

<details>
<summary>Show Answer</summary>

**Answer:** B — Only in other tabs/windows of the same origin, not the tab that made the change

**Explanation:** The `storage` event is the browser's cross-tab notification mechanism for `localStorage` changes — it fires on `window` in every *other* document sharing that origin, but never in the document that actually called `setItem`/`removeItem`/`clear()`. This is a common gotcha when trying to sync UI state within the same tab: you must update that tab's UI manually since it won't receive its own event. Option A is the tempting-but-wrong assumption. Option C is wrong — `sessionStorage` changes don't fire the storage event at all across tabs since it isn't shared. Option D is unrelated to the trigger condition.

</details>
::

::question-wrapper
### Q5. What does `fetch()` return?

- [ ] The parsed JSON body directly
- [ ] A `Promise` that resolves to a `Response` object
- [ ] A `Response` object synchronously
- [ ] An `XMLHttpRequest` instance

<details>
<summary>Show Answer</summary>

**Answer:** B — A `Promise` that resolves to a `Response` object

**Explanation:** `fetch(url)` immediately returns a `Promise<Response>`. The `Response` object wraps headers, status, and a body stream — you must call a method like `.json()` or `.text()` (which itself returns another `Promise`) to extract the actual payload, meaning a full fetch-and-parse is typically two `await`s. Option A is a common beginner shortcut mistake — assuming `fetch` parses JSON for you like some HTTP client libraries do. Option C ignores that network calls are inherently asynchronous. Option D confuses `fetch` with the older callback/event-based API it was designed to replace.

</details>
::

::question-wrapper{language="javascript"}
### Q6. A developer writes the following and is confused why the `catch` block never runs for a `404` response:

::code-wrapper{language="javascript"}
```javascript
fetch("/api/users/999")
  .then(res => res.json())
  .catch(err => console.error("Request failed:", err));
```
::

Why doesn't a `404` trigger the `catch` block?

- [ ] `fetch` only rejects on network failure (e.g., DNS error, offline), not on HTTP error status codes like 404 or 500
- [ ] `404` responses are automatically retried, so the error is swallowed
- [ ] `.json()` silently ignores error status codes
- [ ] This is a bug in the code; `fetch` should reject but doesn't due to a browser inconsistency

<details>
<summary>Show Answer</summary>

**Answer:** A — `fetch` only rejects on network failure (e.g., DNS error, offline), not on HTTP error status codes like 404 or 500

**Explanation:** By design, `fetch`'s promise only rejects for genuine network-level failures — a `4xx`/`5xx` response is still a *successful* HTTP exchange as far as `fetch` is concerned, so it resolves normally with `response.ok === false` and the appropriate `response.status`. This is one of the most common `fetch` gotchas and a real source of silently-swallowed errors in production: code that assumes "no exception means success" will happily try to parse an error page's body as JSON. **Debug:** the fix is to explicitly check `if (!res.ok) throw new Error(...)` before parsing. Option B and C are fabricated behaviors; option D incorrectly frames intentional spec behavior as a bug.

</details>
::

::question-wrapper
### Q7. What is the key advantage of `structuredClone(obj)` over `JSON.parse(JSON.stringify(obj))` for deep-copying data?

- [ ] It is always faster in every browser
- [ ] It can clone richer types like `Date`, `Map`, `Set`, and `ArrayBuffer` correctly, and preserves circular references
- [ ] It can clone functions and DOM nodes without error
- [ ] It converts numbers to strings for safe transport

<details>
<summary>Show Answer</summary>

**Answer:** B — It can clone richer types like `Date`, `Map`, `Set`, and `ArrayBuffer` correctly, and preserves circular references

**Explanation:** `JSON.stringify` silently mangles many types: a `Date` becomes a string, a `Map`/`Set` becomes `{}`, `undefined` values are dropped, and a circular reference throws a `TypeError`. `structuredClone` uses the structured clone algorithm (the same one browsers use for `postMessage`), which correctly round-trips these types and can even clone objects containing circular references back into an equivalent circular structure. Option C is the tempting trap — `structuredClone` explicitly *cannot* clone functions, DOM nodes, or objects with property accessors/prototypes beyond plain data, and throws a `DataCloneError` if you try. Option A overstates it — performance varies by payload shape. Option D is fabricated.

</details>
::

::question-wrapper
### Q8. A page stores several megabytes of data over time via repeated `localStorage.setItem` calls until the origin's quota (commonly ~5–10MB depending on browser) is exceeded. What happens on the call that exceeds it?

- [ ] The call silently does nothing and older data is evicted automatically
- [ ] It throws a `DOMException` (commonly named `QuotaExceededError`) synchronously
- [ ] The `Promise` returned by `setItem` rejects
- [ ] The browser prompts the user to grant more space

<details>
<summary>Show Answer</summary>

**Answer:** B — It throws a `DOMException` (commonly named `QuotaExceededError`) synchronously

**Explanation:** `localStorage.setItem` is a synchronous API, so quota overflow is reported synchronously by throwing — it does not return a rejected promise (there is no promise at all) and it does not silently evict old data like an LRU cache would. **Debug:** production code writing to `localStorage` in a loop or with user-generated content should wrap `setItem` in a `try/catch` to handle this gracefully instead of crashing the calling code. Option A describes cache-eviction behavior that Web Storage doesn't have. Option C is tempting because so many modern Web APIs are promise-based, but Web Storage predates that convention and stayed synchronous. Option D describes permission-prompt UX that some other APIs (like persistent storage) use, not quota overflow itself.

</details>
::

::question-wrapper
### Q9. Why is heavy, repeated use of `localStorage` inside a hot code path (e.g., on every `scroll` or `mousemove` event) considered a performance anti-pattern?

- [ ] `localStorage` operations are always executed on a background thread, so they don't block, but they leak memory over time
- [ ] `localStorage` reads and writes are synchronous and block the main thread, so frequent calls can cause jank
- [ ] `localStorage` triggers a full page reload on every write
- [ ] `localStorage` writes are asynchronous but rate-limited to one per second by the spec

<details>
<summary>Show Answer</summary>

**Answer:** B — `localStorage` reads and writes are synchronous and block the main thread, so frequent calls can cause jank

**Explanation:** Every `localStorage` read/write happens synchronously on the main thread and, depending on the browser, may involve disk I/O — calling it dozens of times per second inside a scroll or resize handler competes directly with rendering work and can visibly stutter the UI. **Performance:** the idiomatic fix is to debounce/throttle the writes, batch them, or move to `IndexedDB` (which is asynchronous) for high-frequency or large-payload storage needs. Option A and D invent async/threading behavior Web Storage doesn't have. Option C is simply false — writes don't reload the page.

</details>
::

::question-wrapper
### Q10. A user opens the same web app in two separate tabs. They interact with Tab A, which writes to `sessionStorage`. What does Tab B see?

- [ ] Tab B sees the same `sessionStorage` values immediately, since both tabs share the same origin
- [ ] Tab B has its own independent `sessionStorage`, unaffected by Tab A's writes
- [ ] Tab B sees the values only after Tab A is closed
- [ ] Tab B sees the values only after both tabs are refreshed

<details>
<summary>Show Answer</summary>

**Answer:** B — Tab B has its own independent `sessionStorage`, unaffected by Tab A's writes

**Explanation:** Unlike `localStorage`, which is shared across every tab/window of the same origin, `sessionStorage` is scoped per top-level browsing context (roughly: per tab). Even two tabs pointed at the identical URL of the identical origin get separate, isolated `sessionStorage` areas — the only exception is that a duplicated tab (e.g., "duplicate tab" from the browser menu) inherits a copy of the original's `sessionStorage` at the moment of duplication. Option A confuses it with `localStorage`'s sharing behavior. Options C and D fabricate a sync mechanism that doesn't exist for `sessionStorage`.

</details>
::

::question-wrapper
### Q11. Why do teams generally avoid putting large amounts of data in cookies, beyond the ~4KB per-cookie size limit?

- [ ] Cookies are readable by any origin, not just the one that set them
- [ ] Every cookie for a domain is sent with every matching HTTP request to that domain, adding latency and bandwidth overhead to unrelated requests (e.g., image loads)
- [ ] Cookies cannot store string data, only numbers
- [ ] Browsers cap total cookies per domain at 3

<details>
<summary>Show Answer</summary>

**Answer:** B — Every cookie for a domain is sent with every matching HTTP request to that domain, adding latency and bandwidth overhead to unrelated requests (e.g., image loads)

**Explanation:** Because the browser auto-attaches all applicable cookies to *every* request to that origin/path — including static asset requests like images and stylesheets — bloated cookies add real, repeated overhead to traffic that has nothing to do with the cookie's purpose. This is exactly why session identifiers (small) belong in cookies while bulk data belongs in `localStorage`/`IndexedDB` (never auto-sent). Option A is false — cross-origin cookie reads are blocked by the same-origin policy (modern `SameSite` rules restrict this further). Option C is nonsensical since cookies only ever store strings. Option D fabricates a specific count; the real limit is typically around 50–180 cookies per domain depending on the browser, not 3.

</details>
::

::question-wrapper{language="javascript"}
### Q12. What happens when you call `structuredClone()` on an object that contains a function property?

::code-wrapper{language="javascript"}
```javascript
const config = {
  name: "widget",
  onClick: () => console.log("clicked"),
};
structuredClone(config);
```
::

- [ ] It clones the object and silently drops the function
- [ ] It clones the object and replaces the function with `null`
- [ ] It throws a `DataCloneError` (`DOMException`)
- [ ] It clones the function by reference, so both objects share it

<details>
<summary>Show Answer</summary>

**Answer:** C — It throws a `DataCloneError` (`DOMException`)

**Explanation:** The structured clone algorithm has a defined, limited set of cloneable types (primitives, plain objects/arrays, `Date`, `RegExp`, `Map`, `Set`, typed arrays, `Blob`, and a few others); functions are explicitly unsupported and cause the call to throw synchronously. This differs from `JSON.stringify`, which silently *drops* function properties instead of throwing (making option A the tempting-but-wrong answer, since it describes `JSON.stringify`'s behavior, not `structuredClone`'s). **Safety:** if you need to clone an object that might contain callbacks, strip them out first or use a targeted manual copy instead of a blanket deep-clone utility.

</details>
::

::question-wrapper{language="javascript"}
### Q13. A developer writes `JSON.stringify` on an object with a self-reference:

::code-wrapper{language="javascript"}
```javascript
const node = { value: 1 };
node.self = node;
JSON.stringify(node);
```
::

What happens with `JSON.stringify` versus `structuredClone(node)`?

- [ ] Both throw an error for circular references
- [ ] `JSON.stringify` throws a `TypeError` ("Converting circular structure to JSON"); `structuredClone` successfully clones it, preserving the circular reference
- [ ] `JSON.stringify` succeeds by cloning the circular part as `null`; `structuredClone` throws
- [ ] Both succeed silently, but only `structuredClone` preserves the cycle

<details>
<summary>Show Answer</summary>

**Answer:** B — `JSON.stringify` throws a `TypeError` ("Converting circular structure to JSON"); `structuredClone` successfully clones it, preserving the circular reference

**Explanation:** JSON has no representation for cycles, so `JSON.stringify` detects the recursion and throws. The structured clone algorithm, by contrast, is graph-aware — it tracks already-visited objects during the clone and correctly reconstructs the same cyclical shape in the copy, which is exactly the capability referenced in Q7. This is a common trap for anyone who reaches for `JSON.parse(JSON.stringify(x))` as a "deep clone" one-liner without realizing it fails hard on data shapes like linked lists, trees with parent pointers, or event emitters that reference their own listeners.

</details>
::

::question-wrapper{language="javascript"}
### Q14. A response body has already been consumed once via `.json()`. What happens on a second call to `.json()` or `.text()` on the same `Response` object?

::code-wrapper{language="javascript"}
```javascript
const res = await fetch("/api/data");
const data = await res.json();
const dataAgain = await res.json();
```
::

- [ ] It returns the same parsed data again, from an internal cache
- [ ] It throws a `TypeError` because the body stream has already been read (a `Response` body can only be consumed once)
- [ ] It returns `undefined`
- [ ] It re-fetches the URL automatically

<details>
<summary>Show Answer</summary>

**Answer:** B — It throws a `TypeError` because the body stream has already been read (a `Response` body can only be consumed once)

**Explanation:** A `Response`'s body is a one-shot readable stream; once a body-reading method (`.json()`, `.text()`, `.blob()`, `.arrayBuffer()`, `.formData()`) has consumed it, the stream is marked "disturbed" and any further read attempt throws. **Debug:** if you need the body in multiple forms or multiple places, call `res.clone()` *before* the first read to get an independent `Response` with its own body stream. Option A is the tempting assumption since many other JS APIs are idempotent on repeated calls, but streams are explicitly not.

</details>
::

::question-wrapper
### Q15. For storing a short-lived UI preference like "sidebar collapsed = true" that only matters for the current browsing session, which storage mechanism is most idiomatic?

- [ ] `localStorage`, so it survives forever
- [ ] A cookie, so the server can also see it
- [ ] `sessionStorage`, since the preference is only meaningful for the current tab session
- [ ] `IndexedDB`, for its transactional guarantees

<details>
<summary>Show Answer</summary>

**Answer:** C — `sessionStorage`, since the preference is only meaningful for the current tab session

**Explanation:** **Idiom:** match the storage lifetime to the data's actual lifetime — a value that should reset when the tab closes belongs in `sessionStorage`, not `localStorage` (which would leak stale state into unrelated future sessions) or a cookie (which adds unnecessary request overhead for something the server never needs to know). `IndexedDB` is overkill for a single boolean flag; it's meant for structured, queryable, potentially large datasets, not simple key-value preferences.

</details>
::

::question-wrapper
### Q16. For storing a sensitive authentication token, why do security-conscious teams generally prefer an `HttpOnly`, `Secure`, `SameSite` cookie over `localStorage`?

- [ ] Cookies are encrypted automatically by the browser
- [ ] `localStorage` is readable by any JavaScript running on the page, so it's fully exposed to XSS attacks; an `HttpOnly` cookie is inaccessible to JavaScript entirely
- [ ] Cookies have unlimited storage capacity, unlike `localStorage`
- [ ] `localStorage` values expire after 24 hours automatically, breaking long sessions

<details>
<summary>Show Answer</summary>

**Answer:** B — `localStorage` is readable by any JavaScript running on the page, so it's fully exposed to XSS attacks; an `HttpOnly` cookie is inaccessible to JavaScript entirely

**Explanation:** **Safety:** any successful XSS injection on the page can run `localStorage.getItem(...)` and exfiltrate a token stored there — there's no isolation between "your" code and injected code once script execution happens on the page. An `HttpOnly` cookie is deliberately hidden from `document.cookie` and any storage API, so even a successful XSS payload can't read it directly (though it could still be *used* via same-origin requests, which is why `SameSite` matters for CSRF — see Q20). Option A is false: cookies are plain text by default, not encrypted. Option C is false: cookies are far more size-constrained than `localStorage`. Option D is a fabricated default.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What is the idiomatic way to handle a non-2xx HTTP response with `fetch`, given that it doesn't reject the promise?

::code-wrapper{language="javascript"}
```javascript
async function getUser(id) {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) {
    throw new Error(`Request failed with status ${res.status}`);
  }
  return res.json();
}
```
::

- [ ] This pattern is unnecessary — `fetch` already throws on error statuses
- [ ] This is the correct idiom: explicitly check `res.ok` (or `res.status`) and throw before attempting to parse the body
- [ ] `res.ok` doesn't exist; you must compare `res.status === 200` exactly
- [ ] You should check `res.statusText` instead, since `res.ok` is deprecated

<details>
<summary>Show Answer</summary>

**Answer:** B — This is the correct idiom: explicitly check `res.ok` (or `res.status`) and throw before attempting to parse the body

**Explanation:** Since `fetch` treats any completed HTTP exchange as a resolved promise (per Q6), the standard, idiomatic pattern is to inspect `response.ok` (true for status `200–299`) immediately after awaiting the fetch and manually throw so downstream `.catch`/`try-catch` logic can treat it as an error. Skipping this check means error bodies (often HTML error pages or JSON error payloads with a different shape) get parsed as if they were success data. Option C is wrong because `res.ok` covers the whole 2xx range, not just exactly `200` (e.g., `201 Created` and `204 No Content` are also `ok`). Option D fabricates a deprecation that doesn't exist.

</details>
::

::question-wrapper
### Q18. A page needs to cache tens of thousands of structured records (e.g., an offline product catalog) in the browser. Which storage choice is most appropriate, and why?

- [ ] `localStorage`, because it's the simplest key-value API
- [ ] Cookies, because they persist across sessions
- [ ] `IndexedDB`, because it's asynchronous, supports much larger storage quotas, and allows indexed queries over structured data
- [ ] `sessionStorage`, because it's the fastest storage mechanism available

<details>
<summary>Show Answer</summary>

**Answer:** C — `IndexedDB`, because it's asynchronous, supports much larger storage quotas, and allows indexed queries over structured data

**Explanation:** **Performance:** `localStorage`'s synchronous API and typically single-digit-megabyte quota make it unsuitable for large or frequently-read datasets — serializing/deserializing tens of thousands of records on the main thread on every read would cause visible jank (per Q9). `IndexedDB` is purpose-built for this: it's asynchronous (doesn't block rendering), commonly allows quotas in the hundreds of megabytes or more (subject to browser/device policy), and supports indexes for efficient querying instead of loading everything into memory at once. Cookies (option B) are capped at a few KB total and would blow the size limit almost immediately, plus they'd be sent needlessly with every request.

</details>
::

::question-wrapper{language="javascript"}
### Q19. A search-as-you-type feature fires a new `fetch` request on every keystroke. Why should the code use an `AbortController` to cancel the previous in-flight request when a new one starts?

::code-wrapper{language="javascript"}
```javascript
let controller;
async function search(query) {
  controller?.abort();
  controller = new AbortController();
  const res = await fetch(`/api/search?q=${query}`, { signal: controller.signal });
  return res.json();
}
```
::

- [ ] Without it, `fetch` automatically queues requests and only the last one's callback ever runs, so it's purely a style preference
- [ ] Without it, older slow responses can resolve *after* newer, faster ones, overwriting the UI with stale results (a race condition)
- [ ] `AbortController` is required by the Fetch spec for every request, or `fetch` throws a warning
- [ ] It prevents the browser from opening more than one TCP connection at a time

<details>
<summary>Show Answer</summary>

**Answer:** B — Without it, older slow responses can resolve *after* newer, faster ones, overwriting the UI with stale results (a race condition)

**Explanation:** Concurrent `fetch` calls resolve independently and in whatever order the network happens to deliver them — there's no built-in request cancellation or sequencing, so a request for `"ca"` typed early can easily resolve *after* a request for `"cat"` typed later if the network conditions vary, leaving stale results rendered last. Calling `controller.abort()` on the previous controller before issuing a new request cancels the outdated one (its promise rejects with an `AbortError`), preventing it from ever resolving and overwriting fresher data. Option A invents automatic queuing that doesn't exist — every `fetch` call runs independently and concurrently.

</details>
::

::question-wrapper
### Q20. Setting a cookie with `Set-Cookie: session=abc123; SameSite=Strict; Secure; HttpOnly` primarily helps mitigate which class of attack?

- [ ] SQL injection
- [ ] Cross-Site Request Forgery (CSRF), by preventing the cookie from being sent on cross-site requests
- [ ] Cross-Site Scripting (XSS) injection into the page's HTML
- [ ] DNS spoofing

<details>
<summary>Show Answer</summary>

**Answer:** B — Cross-Site Request Forgery (CSRF), by preventing the cookie from being sent on cross-site requests

**Explanation:** **Safety:** `SameSite=Strict` (or `Lax`) tells the browser not to attach this cookie to requests initiated from a different site, which directly defeats the classic CSRF pattern of a malicious page silently submitting a form or request to your app while relying on the browser auto-attaching the victim's session cookie. `HttpOnly` (from Q16) protects against XSS *reading* the cookie via JavaScript, but that's a separate, complementary protection, not what `SameSite` is for — so option C conflates the two attributes' purposes. `Secure` only ensures the cookie is sent solely over HTTPS, unrelated to injection attacks. Neither attribute has anything to do with SQL injection or DNS-layer attacks.

</details>
::
