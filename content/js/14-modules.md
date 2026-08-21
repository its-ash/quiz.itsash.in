# 14 — Modules

::question-wrapper{language="javascript"}
### Q1. Which pair correctly matches each module system to its export/import syntax?

- [ ] Both use `require`/`module.exports`, but ESM adds static analysis on top
- [ ] ESM uses `import`/`export`; CommonJS uses `require`/`module.exports`
- [ ] ESM uses `require`/`module.exports`; CommonJS uses `import`/`export`
- [ ] Both use `import`/`export`, but CommonJS resolves them synchronously

<details>
<summary>Show Answer</summary>

**Answer:** B — ESM uses `import`/`export`; CommonJS uses `require`/`module.exports`

**Explanation:** ECMAScript Modules (ESM) are the standardized, browser-and-Node-native system built around the `import`/`export` keywords. CommonJS is Node's original module system, built around the `require()` function and the `module.exports` object. They are not interchangeable syntaxes for the same mechanism — ESM is statically analyzable at compile time, while CommonJS resolves `require` calls dynamically at runtime, which is the root of most interop friction between the two.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What is the fundamental difference between how ESM and CommonJS resolve imports?

- [ ] CommonJS imports are resolved statically at compile time; ESM imports are resolved dynamically at runtime
- [ ] Both are resolved at compile time, but ESM caches the result and CommonJS doesn't
- [ ] ESM imports are resolved statically at compile time; CommonJS `require` calls are resolved dynamically at runtime
- [ ] There is no meaningful difference — both are just syntax sugar over the same loader

<details>
<summary>Show Answer</summary>

**Answer:** C — ESM imports are resolved statically at compile time; CommonJS `require` calls are resolved dynamically at runtime

**Explanation:** ESM `import`/`export` declarations must appear at the top level of a module with a literal specifier, so a tool can determine the entire dependency graph before running any code — this is what enables tree-shaking and hoisting. CommonJS `require()` is just a function call, so it can appear inside `if` blocks, loops, or be built from a dynamic string, meaning the dependency graph can only be known by actually executing the code. **Idiom:** this is the core reason ESM is described as "static" and CommonJS as "dynamic."

</details>
::

::question-wrapper{language="javascript"}
### Q3. A module has this export statement. Which import correctly consumes both exports?

::code-wrapper{language="javascript" filename="math.js"}
```javascript
export const PI = 3.14159;
export default function square(n) {
  return n * n;
}
```
::

- [ ] `import square, { PI } from './math.js';`
- [ ] `import { square, PI } from './math.js';`
- [ ] `import { default as square, PI } from './math.js';` only — the shorthand form is invalid
- [ ] `import PI, { square } from './math.js';`

<details>
<summary>Show Answer</summary>

**Answer:** A — `import square, { PI } from './math.js';`

**Explanation:** A module can have any number of named exports (`PI` here) plus at most one default export (the `square` function). The default import binding comes first with no braces, followed by a comma and the named imports in braces. Option B incorrectly treats `square` as a named export. Option D swaps the positions. Option C is unnecessarily verbose — `import square, { PI }` is valid shorthand for exactly that `default as square` form.

</details>
::

::question-wrapper{language="javascript"}
### Q4. `counter.js` exports a mutable binding. What does `main.js` log?

::code-wrapper{language="javascript" filename="counter.js"}
```javascript
export let count = 0;
export function increment() {
  count++;
}
```
::

::code-wrapper{language="javascript" filename="main.js"}
```javascript
import { count, increment } from './counter.js';

console.log(count);
increment();
console.log(count);
```
::

- [ ] 0 then 0
- [ ] 1 then 1
- [ ] Throws a TypeError on the second `console.log` because `count` was reassigned externally
- [ ] 0 then 1

<details>
<summary>Show Answer</summary>

**Answer:** D — 0 then 1

**Explanation:** ESM named imports are **live bindings**, not copied values — `count` in `main.js` is a read-only view directly onto `counter.js`'s own `count` variable. When `increment()` reassigns `count` inside the module that owns it, every importer observing that binding sees the new value immediately, without re-importing anything. **Debug:** this is a major behavioral difference from CommonJS, where `require` copies the value (or reference) present at the moment of the require call, so a later reassignment in the source module would NOT be visible to something that already required it.

</details>
::

::question-wrapper{language="javascript"}
### Q5. The equivalent scenario using CommonJS. What does `main.js` log?

::code-wrapper{language="javascript" filename="counter.js"}
```javascript
let count = 0;
function increment() { count++; }
module.exports = { count, increment };
```
::

::code-wrapper{language="javascript" filename="main.js"}
```javascript
const { count, increment } = require('./counter.js');

console.log(count);
increment();
console.log(count);
```
::

- [ ] 0 then 1
- [ ] 0 then 0
- [ ] 1 then 1
- [ ] undefined then undefined

<details>
<summary>Show Answer</summary>

**Answer:** B — 0 then 0

**Explanation:** `module.exports = { count, increment }` copies the *current primitive value* of `count` (`0`) into a new property on the exports object at the moment `module.exports` is assigned. `main.js`'s destructured `count` is just a local `const` holding that snapshot — it has no ongoing connection to `counter.js`'s internal `count` variable. Calling `increment()` mutates the module-internal `count`, but `main.js`'s copy never updates. **Debug:** this exact gap — CommonJS copies values, ESM shares live bindings — is why porting CommonJS "mutable exported counter" patterns to ESM (or vice versa) silently changes behavior.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What happens when this code runs?

::code-wrapper{language="javascript"}
```javascript
import { PI } from './constants.js';

PI = 3;
console.log(PI);
```
::

- [ ] SyntaxError / TypeError — named imports are read-only bindings and cannot be reassigned
- [ ] Logs 3 — the local binding is reassigned but the export is untouched
- [ ] Logs 3 — and the exporting module's `PI` is also updated to 3, since imports are live
- [ ] Logs `undefined` because `PI` was never actually imported

<details>
<summary>Show Answer</summary>

**Answer:** A — SyntaxError / TypeError — named imports are read-only bindings and cannot be reassigned

**Explanation:** ESM named imports are read-only views onto the exporting module's binding — "live" means updates flow *from* the exporter *to* every importer, never the reverse. Attempting to assign to an imported binding directly is rejected (a `TypeError: Assignment to constant variable`-style failure, enforced at the binding level regardless of how the exporter declared it). If you need to change shared state from the importing side, the exporting module must expose a function (like `increment()` in Q4) that performs the mutation internally.

</details>
::

::question-wrapper{language="javascript"}
### Q7. `a.js` and `b.js` import each other. Using ESM, what does running `main.js` log?

::code-wrapper{language="javascript" filename="a.js"}
```javascript
import { bValue } from './b.js';
export const aValue = 'A';
console.log('a.js sees bValue:', bValue);
```
::

::code-wrapper{language="javascript" filename="b.js"}
```javascript
import { aValue } from './a.js';
export const bValue = 'B';
console.log('b.js sees aValue:', aValue);
```
::

::code-wrapper{language="javascript" filename="main.js"}
```javascript
import './a.js';
```
::

- [ ] `a.js sees bValue: undefined` then `b.js sees aValue: A`
- [ ] Throws a ReferenceError immediately due to the circular dependency
- [ ] Both log their values correctly because ESM resolves circular imports out of order
- [ ] `b.js sees aValue: undefined` then `a.js sees bValue: B`

<details>
<summary>Show Answer</summary>

**Answer:** D — `b.js sees aValue: undefined` then `a.js sees bValue: B`

**Explanation:** `main.js` imports `a.js` first, so `a.js` starts executing and immediately hits its `import` of `b.js`, suspending `a.js` to run `b.js`. `b.js` then imports `aValue` from `a.js` — but `a.js`'s `export const aValue = 'A'` line hasn't executed yet (it's after the import), so the *binding* exists (hoisted) but is still in its temporal dead zone / unset state, read as `undefined` at this point. `b.js` finishes, setting `bValue = 'B'`, and control returns to `a.js`, which now sees the fully-set `bValue`. **Debug:** ESM's live-binding model means circular imports don't throw outright, but the order in which each side finishes initializing determines which values are visible when — the fix is usually to reference the circularly-imported binding lazily (inside a function) rather than at module top level.

</details>
::

::question-wrapper{language="javascript"}
### Q8. The same circular scenario, but using CommonJS. What's the key difference in outcome versus the ESM version?

::code-wrapper{language="javascript" filename="a.js"}
```javascript
const { bValue } = require('./b.js');
exports.aValue = 'A';
console.log('a.js sees bValue:', bValue);
```
::

::code-wrapper{language="javascript" filename="b.js"}
```javascript
const { aValue } = require('./a.js');
exports.bValue = 'B';
console.log('b.js sees aValue:', aValue);
```
::

- [ ] `aValue` inside `b.js` is `'A'` because CommonJS eagerly resolves the whole dependency graph before running any module
- [ ] CommonJS throws a `RequireError` on any circular `require`, unlike ESM
- [ ] `aValue` inside `b.js` is `undefined` because CommonJS destructures a snapshot of `a.js`'s (still-incomplete) `exports` object at require-time, and no later update to `a.js`'s exports is ever reflected
- [ ] The behavior is identical to ESM — both produce `undefined` in `b.js` and the correct value in `a.js`

<details>
<summary>Show Answer</summary>

**Answer:** C — `aValue` inside `b.js` is `undefined` because CommonJS destructures a snapshot of `a.js`'s (still-incomplete) `exports` object at require-time, and no later update to `a.js`'s exports is ever reflected

**Explanation:** When `b.js` calls `require('./a.js')`, Node returns whatever `a.js`'s `module.exports` object currently contains — but `a.js` is mid-execution (it's paused at its own `require('./b.js')` call, before reaching `exports.aValue = 'A'`), so the returned object is incomplete/empty at that point. Because `const { aValue } = require(...)` destructures once and copies that value out immediately, `b.js`'s local `aValue` stays `undefined` forever, even after `a.js` later finishes and sets `exports.aValue`. **Debug:** this is functionally similar in symptom to the ESM case (Q7) but for a different mechanical reason — CommonJS copies a snapshot of an incomplete object, while ESM shares a live binding that is simply unset at read time; the practical fix in CommonJS is to `require` inside a function body (deferring the read) rather than destructuring at the top.

</details>
::

::question-wrapper{language="javascript"}
### Q9. Which statement about renaming a default export at its export site is correct?

- [ ] A default export has no name to rename at the export site — you write `export default <value>`, and the importer chooses any local name it wants
- [ ] A default export can be renamed at the export site using `export default as myName`
- [ ] A default export must share its name with the file name
- [ ] A default export can be renamed only if it's a named export first, then re-exported as default

<details>
<summary>Show Answer</summary>

**Answer:** A — A default export has no name to rename at the export site — you write `export default <value>`, and the importer chooses any local name it wants

**Explanation:** `export default` binds to the reserved identifier `default` internally; there's nothing to "rename" on the export side because there was never a chosen name to begin with — you're exporting a value, not a named binding. This is exactly why the importer has total freedom: `import Foo from './x.js'`, `import Bar from './x.js'`, and `import whatever123 from './x.js'` are all equally valid regardless of what the exporting file called the value internally. Contrast with named exports, which do have a fixed export-side name unless explicitly aliased with `export { x as y }`.

</details>
::

::question-wrapper{language="javascript"}
### Q10. Two files import the same default-exported function under different local names. Is this a problem?

::code-wrapper{language="javascript" filename="fileA.js"}
```javascript
import formatDate from './dateUtils.js';
```
::

::code-wrapper{language="javascript" filename="fileB.js"}
```javascript
import fmtDate from './dateUtils.js';
```
::

- [ ] Yes — a default export must be imported under the same name everywhere or the module cache breaks
- [ ] No — default import names are purely local aliases and can legally differ across importing files
- [ ] No, but it silently creates two separate instances of `dateUtils.js`
- [ ] Yes — this throws a `SyntaxError: mismatched default import name`

<details>
<summary>Show Answer</summary>

**Answer:** B — No — default import names are purely local aliases and can legally differ across importing files

**Explanation:** Since a default export carries no name of its own (see Q9), every importer is free to bind it to whatever local identifier is convenient — `formatDate` and `fmtDate` both refer to the exact same underlying function object. **Idiom:** while this is legal, letting the local name drift across a codebase makes the export harder to grep for and reason about — many teams adopt a convention (e.g., always import a default as the exact export's canonical name) purely for readability, not because the language requires it.

</details>
::

::question-wrapper{language="javascript"}
### Q11. A CommonJS module is imported from an ESM file in Node.js. What typically happens to `module.exports`?

::code-wrapper{language="javascript" filename="legacy.cjs"}
```javascript
module.exports = { greet: () => 'hi' };
```
::

::code-wrapper{language="javascript" filename="main.mjs"}
```javascript
import legacy from './legacy.cjs';
console.log(legacy.greet());
```
::

- [ ] `legacy.cjs`'s properties automatically become named exports, so `import { greet } from './legacy.cjs'` is the only correct form
- [ ] Node refuses to import `.cjs` files from `.mjs` files under any circumstances
- [ ] `legacy` is `undefined` because CommonJS modules have no exports Node can see from ESM
- [ ] `module.exports` becomes the default export as a whole, so `legacy.greet()` works via the default import

<details>
<summary>Show Answer</summary>

**Answer:** D — `module.exports` becomes the default export as a whole, so `legacy.greet()` works via the default import

**Explanation:** Node's interop layer wraps a CommonJS module's entire `module.exports` object as the ESM default export when it's imported from an `.mjs` file (or an ESM-mode `.js` file). So `import legacy from './legacy.cjs'` gives you the whole `{ greet }` object, and `legacy.greet()` works as shown. Named-export interop (`import { greet } from './legacy.cjs'`) is sometimes synthesized too, via static analysis of the CommonJS source by Node/bundlers, but this is heuristic and not guaranteed — it's a well-known source of "why does this named import say undefined" bugs. **Portability:** relying on named-import interop for CommonJS packages is fragile across Node versions and bundlers; importing the default and destructuring afterward is the safer, more portable pattern.

</details>
::

::question-wrapper{language="javascript"}
### Q12. Why can bundlers reliably tree-shake ESM code but not CommonJS code?

- [ ] CommonJS doesn't support tree-shaking because Node.js disables minification
- [ ] ESM modules are always smaller in file size before bundling even begins
- [ ] ESM's `import`/`export` are static, top-level-only declarations a bundler can analyze without running any code, whereas `require()` is a plain function call that can be conditional or dynamically constructed, forcing the bundler to assume any export might be used
- [ ] Tree-shaking is a runtime, not build-time, distinction — it applies equally to both once code is executing

<details>
<summary>Show Answer</summary>

**Answer:** C — ESM's `import`/`export` are static, top-level-only declarations a bundler can analyze without running any code, whereas `require()` is a plain function call that can be conditional or dynamically constructed, forcing the bundler to assume any export might be used

**Explanation:** Tree-shaking means removing exports that are never imported anywhere. To do that safely, a bundler needs to prove, without executing the program, exactly which exports each file uses — ESM's syntactic restrictions (imports/exports must be literal, top-level statements) make that provable at build time. `require('./mod')` inside an `if` block, inside a loop, or built from `require(someVariable)` cannot be resolved without actually running the code, so a bundler must conservatively keep everything `module.exports` might contain. **Performance:** this is the primary real-world reason teams migrate legacy CommonJS libraries to ESM — smaller bundles, not stylistic preference.

</details>
::

::question-wrapper{language="javascript"}
### Q13. `state.js` is imported by both `pageA.js` and `pageB.js`. `pageA.js` mutates the shared state, then `pageB.js` reads it. What does `pageB.js` see?

::code-wrapper{language="javascript" filename="state.js"}
```javascript
export const store = { user: null };
```
::

::code-wrapper{language="javascript" filename="pageA.js"}
```javascript
import { store } from './state.js';
store.user = 'ashvini';
```
::

::code-wrapper{language="javascript" filename="pageB.js"}
```javascript
import { store } from './state.js';
console.log(store.user);
```
::

- [ ] `'ashvini'` — modules are singletons, so both files share the exact same `store` object instance
- [ ] `null` — each importing file gets its own fresh copy of `store.js`'s exports
- [ ] `undefined` — `store` isn't re-exported by `pageB.js` so it can't see the mutation
- [ ] It depends on import order, and is `null` if `pageB.js` runs first regardless of later mutation

<details>
<summary>Show Answer</summary>

**Answer:** A — `'ashvini'` — modules are singletons, so both files share the exact same `store` object instance

**Explanation:** A module is only ever evaluated once per module graph, no matter how many files import it — every importer receives references to the *same* exported bindings/objects, not independent copies. `store` is a single object living in `state.js`'s module scope; `pageA.js` mutates its `user` property, and since `pageB.js` holds a reference to that identical object, it observes the change. **Debug:** this singleton behavior is exactly what makes modules a common (if implicit) place to stash shared app state, but it's also a classic source of test-pollution bugs — module state persists across test files unless explicitly reset, because the module isn't re-evaluated between imports.

</details>
::

::question-wrapper{language="javascript"}
### Q14. What is the key behavioral difference between static `import` and dynamic `import()`?

- [ ] Static `import` returns a promise; dynamic `import()` is synchronous and hoisted
- [ ] Static `import` is hoisted and its target module is fully evaluated before the rest of the file runs; dynamic `import()` is a function call, evaluated in place, that returns a promise resolving to the module namespace object
- [ ] They behave identically — `import()` is just alternate syntax with no timing difference
- [ ] Dynamic `import()` can only be used inside Node.js, never in browsers

<details>
<summary>Show Answer</summary>

**Answer:** B — Static `import` is hoisted and its target module is fully evaluated before the rest of the file runs; dynamic `import()` is a function call, evaluated in place, that returns a promise resolving to the module namespace object

**Explanation:** Static `import` declarations are processed at parse time regardless of where they're textually written in the file — the imported module's code runs to completion before any of the importing module's own top-level code executes (see Q15). Dynamic `import('./mod.js')`, by contrast, is an ordinary expression that can appear anywhere a value is expected (inside an `if`, a click handler, a loop) and returns a promise, since fetching/compiling the module may happen asynchronously. This is the mechanism behind code-splitting and lazy-loading — a chunk is only fetched when the `import()` call actually runs. **Note:** consuming that returned promise (`.then()` or `await`) is covered in depth once Promises and async/await are introduced later in this track — for now, just recognize that dynamic `import()` always hands you a promise, never the module directly.

</details>
::

::question-wrapper{language="javascript"}
### Q15. What does this log, and in what order?

::code-wrapper{language="javascript" filename="logger.js"}
```javascript
console.log('logger.js running');
export function log(msg) { console.log(msg); }
```
::

::code-wrapper{language="javascript" filename="main.js"}
```javascript
console.log('main.js: before import line');
import { log } from './logger.js';
console.log('main.js: after import line');
log('hello');
```
::

- [ ] `main.js: before import line`, `logger.js running`, `main.js: after import line`, `hello`
- [ ] `main.js: before import line`, `main.js: after import line`, `logger.js running`, `hello`
- [ ] This throws a SyntaxError because the `import` statement appears after other code
- [ ] `logger.js running`, `main.js: before import line`, `main.js: after import line`, `hello`

<details>
<summary>Show Answer</summary>

**Answer:** D — `logger.js running`, `main.js: before import line`, `main.js: after import line`, `hello`

**Explanation:** `import` declarations are hoisted to the top of the module and always execute before any of the module's own top-level code, regardless of where the `import` line is textually positioned in the source. So even though `console.log('main.js: before import line')` appears *above* the `import` statement in the file, `logger.js` still finishes running first, because hoisting moves the import's effect to the very top of `main.js`'s execution. **Debug:** this surprises developers coming from CommonJS, where `require()` genuinely executes at its exact position in the file — writing code above a `require()` call really does run first there, unlike with ESM `import`.

</details>
::

::question-wrapper{language="javascript"}
### Q16. Which correctly re-exports `add` from `math.js` through `index.js`, without `index.js` needing a separate local binding?

::code-wrapper{language="javascript" filename="math.js"}
```javascript
export function add(a, b) { return a + b; }
```
::

- [ ] `import { add } from './math.js'; export add;`
- [ ] `export add from './math.js';`
- [ ] `export { add } from './math.js';`
- [ ] `import add from './math.js'; export default add;`

<details>
<summary>Show Answer</summary>

**Answer:** C — `export { add } from './math.js';`

**Explanation:** The `export { name } from './source.js'` syntax is a dedicated re-export form: it forwards `math.js`'s `add` export through `index.js` as `index.js`'s own named export, without ever creating a local `add` binding inside `index.js` that you'd have to import first. Option A is invalid syntax (`export add;` isn't a real form). Option B is also invalid — `export ... from` requires braces around named bindings. Option D would work but changes `add` into `index.js`'s *default* export rather than preserving it as a named export, which isn't equivalent unless that was the goal.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What is logged, and why is this preferred over importing then re-exporting manually when no transformation is needed?

::code-wrapper{language="javascript" filename="shapes.js"}
```javascript
export function circleArea(r) { return Math.PI * r * r; }
```
::

::code-wrapper{language="javascript" filename="geometry.js"}
```javascript
export { circleArea } from './shapes.js';
```
::

::code-wrapper{language="javascript" filename="main.js"}
```javascript
import { circleArea } from './geometry.js';
console.log(circleArea(2).toFixed(2));
```
::

- [ ] `12.57` — `geometry.js` forwards the export without ever binding `circleArea` into its own local scope, unlike `import { x } from ...; export { x };` which does create a local binding
- [ ] `12.57` — but this is functionally worse than importing-then-exporting because it defeats tree-shaking
- [ ] `NaN` — re-exported functions lose their closure over `Math.PI`
- [ ] Throws — `circleArea` was never locally declared inside `geometry.js`, so `export { circleArea }` has nothing to reference

<details>
<summary>Show Answer</summary>

**Answer:** A — `12.57` — `geometry.js` forwards the export without ever binding `circleArea` into its own local scope, unlike `import { x } from ...; export { x };` which does create a local binding

**Explanation:** `export { circleArea } from './shapes.js'` is purely a forwarding declaration at the module-linking level — `geometry.js` never actually creates a local `circleArea` variable, it just tells consumers "ask `shapes.js` for this." The computation is unaffected: `circleArea(2)` is `Math.PI * 4 ≈ 12.566...`, and `.toFixed(2)` rounds that to `'12.57'`. **Idiom:** this direct re-export form is preferred in "barrel" files (`index.js` aggregating a package's public API) specifically because it avoids an unnecessary local binding and stays just as tree-shakeable as a direct import, unlike patterns that route the value through an intermediate local variable.

</details>
::

::question-wrapper{language="javascript"}
### Q18. What does `Object.keys(utils)` log?

::code-wrapper{language="javascript" filename="utils.js"}
```javascript
export function trim(s) { return s.trim(); }
export function upper(s) { return s.toUpperCase(); }
export default function normalize(s) { return trim(s).toLowerCase(); }
```
::

::code-wrapper{language="javascript" filename="main.js"}
```javascript
import * as utils from './utils.js';
console.log(Object.keys(utils).sort());
console.log(utils.trim('  hi  '));
```
::

- [ ] `['trim', 'upper']` then `'hi'`
- [ ] `['default', 'trim', 'upper']` then throws because `utils.trim` isn't callable
- [ ] `['normalize', 'trim', 'upper']` then `'hi'`
- [ ] `['default', 'trim', 'upper']` then `'hi'`

<details>
<summary>Show Answer</summary>

**Answer:** D — `['default', 'trim', 'upper']` then `'hi'`

**Explanation:** `import * as utils` creates a **namespace object** exposing every export of the module as a property — every named export (`trim`, `upper`) under its own name, and the default export under the special key `'default'` (not `'normalize'`, since the function's local name doesn't become the property key). So `Object.keys(utils).sort()` yields `['default', 'trim', 'upper']`. Named exports remain directly callable as properties, so `utils.trim('  hi  ')` runs normally and returns `'hi'`. Accessing the default function itself would require `utils.default(...)`, not `utils.normalize(...)`.

</details>
::

::question-wrapper{language="javascript"}
### Q19. A team's codebase has some files doing `import * as api from './api.js'` and using `api.fetchUser()`, `api.postOrder()`, etc., while other files do `import { fetchUser } from './api.js'` for the exact same module. What's the best-practice concern here?

- [ ] There is no real concern — `api.fetchUser()` and `fetchUser()` are guaranteed to always behave identically with zero trade-offs
- [ ] Mixing namespace-style and named-import access for the same module hurts predictability and tooling — pick one convention (usually named imports, since they tree-shake better and are easier to grep) and apply it consistently
- [ ] Namespace imports are strictly forbidden by the ECMAScript spec when named imports exist elsewhere in the codebase
- [ ] The two forms cause the module to be evaluated twice, doubling any side effects in `api.js`

<details>
<summary>Show Answer</summary>

**Answer:** B — Mixing namespace-style and named-import access for the same module hurts predictability and tooling — pick one convention (usually named imports, since they tree-shake better and are easier to grep) and apply it consistently

**Explanation:** Both forms are legal and reference the same underlying live bindings (module singletons — see Q13), so there's no correctness bug per se. The issue is consistency: namespace imports (`import * as api`) pull in a reference to every export even if only one is used, which can undermine tree-shaking analysis in some bundlers, and make "who uses `fetchUser`" harder to grep for since call sites read as `api.fetchUser` in some files and `fetchUser` in others. **Idiom:** picking named imports as the default convention (reserving `import * as ns` for cases like re-exporting an entire module, or genuinely needing dozens of its exports) keeps call sites uniform and analysis-friendly across a codebase.

</details>
::

::question-wrapper{language="javascript"}
### Q20. A large CommonJS utility library is `require`d conditionally based on an environment check, then the app is bundled for production. What's the most accurate best-practice takeaway?

::code-wrapper{language="javascript"}
```javascript
let logger;
if (process.env.NODE_ENV === 'production') {
  logger = require('./prodLogger.js');
} else {
  logger = require('./devLogger.js');
}
```
::

- [ ] Bundlers always evaluate `process.env.NODE_ENV` at build time regardless of module system, so this pattern is equally tree-shakeable in both CommonJS and ESM
- [ ] This pattern is impossible to express in CommonJS at all — `require` cannot appear inside an `if` block
- [ ] The bundler can't statically prove which branch runs, so it typically must include both `prodLogger.js` and `devLogger.js` in the final bundle — an ESM equivalent using dynamic `import()` inside the conditional, or a build-time env substitution, avoids shipping the unused branch
- [ ] Switching this to ESM `import` syntax directly inside the `if` block would fix the bundle-size issue

<details>
<summary>Show Answer</summary>

**Answer:** C — The bundler can't statically prove which branch runs, so it typically must include both `prodLogger.js` and `devLogger.js` in the final bundle — an ESM equivalent using dynamic `import()` inside the conditional, or a build-time env substitution, avoids shipping the unused branch

**Explanation:** Because `require()` is an ordinary function call, a bundler generally cannot prove at build time which branch of the `if` will execute (even though `NODE_ENV` checks are common enough that some bundlers special-case and inline them via string replacement, that's a heuristic, not a language guarantee) — so worst case, both `prodLogger.js` and `devLogger.js` end up bundled, bloating output. Static ESM `import` is explicitly disallowed inside a block like this (it must be a top-level declaration — option D is a syntax error, not a fix), so the correct ESM-world fix is dynamic `import()`, which is a real expression usable inside conditionals and only fetches the branch actually reached at runtime. **Performance:** this is a concrete, production-relevant instance of the static-vs-dynamic distinction from Q2 and Q12 — it's not just theoretical, it directly affects shipped bundle size.

</details>
::
