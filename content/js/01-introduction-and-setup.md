# 01 — Introduction & Setup

::question-wrapper
### Q1. What is V8, in the context of JavaScript?

- [ ] A JavaScript runtime that provides `fs`, `http`, and other OS-level APIs
- [ ] A JavaScript engine that parses and executes JavaScript source code
- [ ] A package manager bundled with Node.js
- [ ] A transpiler that converts modern JavaScript into ES5

<details>
<summary>Show Answer</summary>

**Answer:** B — A JavaScript engine that parses and executes JavaScript source code

**Explanation:** V8 (built by Google) is an engine — it parses, compiles (via JIT), and executes JS code, and implements the ECMAScript spec plus the language's core objects (`Array`, `Promise`, etc.). It does **not** provide `fs` or `http`; those come from a *runtime* built around the engine. Option A describes Node.js, which embeds V8 but adds its own APIs. Option C confuses V8 with `npm`. Option D describes tools like Babel — V8 runs JS natively, it doesn't transpile.

</details>
::

::question-wrapper
### Q2. Which statement correctly distinguishes Node.js from V8?

- [ ] They are the same thing — "Node.js" is just another name for the V8 engine
- [ ] Node.js is a runtime that embeds V8 and adds APIs like `fs`, `http`, and `process` that aren't part of JavaScript itself
- [ ] V8 is a superset of Node.js that adds browser-specific APIs
- [ ] Node.js replaces V8 with its own custom-built engine

<details>
<summary>Show Answer</summary>

**Answer:** B — Node.js is a runtime that embeds V8 and adds APIs like `fs`, `http`, and `process`

**Explanation:** Node.js embeds the V8 engine and layers on host APIs (filesystem, networking, `process`, `Buffer`) that are not defined by the ECMAScript spec at all. **Portability:** because `fs` and `window` are host APIs, not language features, code using them isn't portable between Node and the browser even though both run "JavaScript." Option A collapses the runtime/engine distinction. Option C reverses the relationship. Option D is false — Node has always used V8, not a custom engine.

</details>
::

::question-wrapper
### Q3. "ES6" and "ES2015" refer to:

- [ ] Two different, incompatible versions of JavaScript
- [ ] The same ECMAScript edition — ES6 is the older name, ES2015 is the year-based name adopted afterward
- [ ] ES6 is the browser implementation, ES2015 is the Node.js implementation
- [ ] ES2015 is a strict superset of ES6 released a year later

<details>
<summary>Show Answer</summary>

**Answer:** B — The same edition; ES6 is the older name, ES2015 is the year-based name

**Explanation:** TC39 switched to annual, year-based release naming starting with the edition that introduced `let`, `const`, classes, arrow functions, and Promises. That edition is interchangeably called "ES6" or "ES2015" — they are not different versions. Options A, C, and D all invent a distinction that doesn't exist.

</details>
::

::question-wrapper{language="javascript"}
### Q4. Where must a `"use strict"` directive appear to enable strict mode for an entire script?

::code-wrapper{language="javascript"}
```javascript
function example() {
  "use strict";
  x = 10;
}
```
::

- [ ] Anywhere in the file, even after other executable statements
- [ ] As the very first statement in the file (or function body), before any other statements
- [ ] Inside a comment at the top of the file
- [ ] It must be passed as a CLI flag to Node; it cannot appear in source

<details>
<summary>Show Answer</summary>

**Answer:** B — As the very first statement in the file (or function body), before any other statements

**Explanation:** `"use strict"` is a *directive prologue* — the parser only recognizes it as special when it is literally the first statement. **Debug:** if any other statement (even a variable declaration) precedes it, it's silently treated as a harmless, no-op string literal expression instead of enabling strict mode — a common silent-failure trap. Option A is the tempting wrong answer for exactly that reason. It's a plain string literal, not a comment (C), and works in any JS file, not just via a Node flag (D).

</details>
::

::question-wrapper
### Q5. By default, how does a browser handle a plain `<script src="app.js"></script>` tag with no `async`, `defer`, or `type="module"` attributes?

- [ ] It downloads and executes the script without blocking HTML parsing
- [ ] It blocks HTML parsing while the script downloads and executes, then resumes parsing
- [ ] It waits until the entire document is parsed before downloading the script
- [ ] It downloads the script in parallel but always executes it after `DOMContentLoaded`

<details>
<summary>Show Answer</summary>

**Answer:** B — It blocks HTML parsing while the script downloads and executes, then resumes parsing

**Explanation:** A classic synchronous `<script>` tag is parser-blocking: the browser must fetch and run it before continuing to parse the rest of the HTML. **Performance:** this is why render-blocking scripts placed in `<head>` are a common cause of slow page loads, and why `defer`/`async`/placing scripts before `</body>` are recommended. Option A describes `async`. Option C is closer to `defer`'s execution timing but still wrong about parsing being blocked. Option D is incorrect — plain scripts run as soon as they're evaluated, not after `DOMContentLoaded`.

</details>
::

::question-wrapper
### Q6. A `<script type="module">` tag behaves like which combination of attributes by default?

- [ ] Like a plain script — blocking, synchronous
- [ ] Like `async` — non-blocking, execution order not guaranteed
- [ ] Like `defer` — non-blocking, and execution is deferred until after parsing, in document order
- [ ] Modules cannot be loaded via `<script>` tags at all; they require a bundler

<details>
<summary>Show Answer</summary>

**Answer:** C — Like `defer` — non-blocking, deferred, and in document order

**Explanation:** ES module scripts are deferred by default: the browser fetches them without blocking parsing, and executes them in document order after parsing completes (unless `async` is also explicitly added, which then allows out-of-order execution). Option A is the common beginner assumption — that "module" means "just a normal script with import support" — but the loading semantics are different. Option B is wrong unless `async` is explicitly added. Option D is false; native `<script type="module">` works without any bundler in modern browsers.

</details>
::

::question-wrapper
### Q7. What is a defining characteristic of Deno compared to Node.js?

- [ ] Deno cannot execute TypeScript without a separate compile step
- [ ] Deno runs scripts with no filesystem, network, or environment access by default, requiring explicit permission flags
- [ ] Deno does not support ES modules
- [ ] Deno uses the SpiderMonkey engine instead of V8

<details>
<summary>Show Answer</summary>

**Answer:** B — Deno runs scripts with no filesystem, network, or environment access by default, requiring explicit permission flags

**Explanation:** **Safety:** Deno is secure-by-default — a script needs `--allow-read`, `--allow-net`, `--allow-env`, etc. explicitly granted, unlike Node where any script has full system access the moment it runs. Deno actually supports TypeScript natively without a separate build step (making A false), supports ES modules as its primary module system (making C false), and uses V8 just like Node (making D false).

</details>
::

::question-wrapper{language="javascript"}
### Q8. What is the key syntactic/behavioral difference between CommonJS and ES modules in Node.js?

::code-wrapper{language="javascript"}
```javascript
const fs = require('fs');
module.exports = { readConfig };
```
::

- [ ] CommonJS uses `require`/`module.exports` and loads synchronously; ESM uses `import`/`export` and is loaded asynchronously with static analysis
- [ ] There is no real difference — Node treats both identically at runtime
- [ ] CommonJS supports `import` statements but ESM does not
- [ ] ESM is only available in the browser, never in Node.js

<details>
<summary>Show Answer</summary>

**Answer:** A — CommonJS uses `require`/`module.exports` synchronously; ESM uses `import`/`export`, resolved asynchronously and statically analyzable

**Explanation:** CommonJS `require()` calls are synchronous function calls that can happen conditionally anywhere in code. ESM `import`/`export` bindings are statically analyzed at parse time (enabling tree-shaking) and top-level `import` cannot be conditional. Node determines which system to use per-file based on `.mjs`/`.cjs` extensions or the `"type"` field in `package.json`. Option B ignores real interop pitfalls (e.g., CJS's `require` is unavailable by default in ESM files). Option C is backwards. Option D is false — Node has supported ESM natively since Node 12+.

</details>
::

::question-wrapper{language="javascript"}
### Q9. What happens when you assign to an undeclared variable inside strict-mode code?

::code-wrapper{language="javascript"}
```javascript
"use strict";
function setTotal() {
  total = 42;
}
setTotal();
```
::

- [ ] `total` is silently created as a global variable, same as in non-strict mode
- [ ] A `ReferenceError` is thrown because `total` was never declared
- [ ] A `TypeError` is thrown because `total` is `undefined`
- [ ] It works fine, but `total` is scoped only to `setTotal`

<details>
<summary>Show Answer</summary>

**Answer:** B — A `ReferenceError` is thrown because `total` was never declared

**Explanation:** In non-strict mode, assigning to an undeclared identifier silently creates a global variable — a notorious source of bugs. Strict mode closes this hole: it throws a `ReferenceError` instead. **Safety:** the correct fix is to declare the variable explicitly (`let total = 42;`) rather than rely on implicit globals. Option A describes the (bug-prone) non-strict behavior. Option C names the wrong error type. Option D is wrong because the assignment never succeeds at all.

</details>
::

::question-wrapper
### Q10. Do ES modules (`<script type="module">` or `.mjs` files) require an explicit `"use strict"` directive to run in strict mode?

- [ ] Yes — without it, modules run in sloppy (non-strict) mode
- [ ] No — ES modules are automatically strict mode, with no directive needed
- [ ] Only if the module also uses classes
- [ ] Only in Node.js, not in browsers

<details>
<summary>Show Answer</summary>

**Answer:** B — No, ES modules are automatically strict mode

**Explanation:** The ECMAScript spec mandates that all module code is implicitly strict — there's no opt-out and no need for the directive. **Idiom:** adding `"use strict"` at the top of a `.mjs` file is harmless but redundant. Option A is the common false assumption carried over from script-tag habits. Option C invents a class-specific rule (classes are *always* strict-mode bodies regardless of module status, but that's a separate rule). Option D is false — this is a language-level guarantee, not runtime-specific.

</details>
::

::question-wrapper
### Q11. What does top-level `this` refer to in each of: a classic (non-module) browser script, an ES module, and a Node.js CommonJS file?

- [ ] `window`, `window`, and `module.exports` respectively
- [ ] `undefined` in all three
- [ ] `window` (global object), `undefined`, and `module.exports` respectively
- [ ] `globalThis` in all three, since ES2020 unified them

<details>
<summary>Show Answer</summary>

**Answer:** C — `window`, `undefined`, and `module.exports` respectively

**Explanation:** In a classic script, top-level `this` is the global object (`window` in browsers). In an ES module, top-level `this` is `undefined` by spec — modules don't have an implicit global receiver. In a Node CommonJS file, top-level `this` refers to `module.exports` because the file is wrapped in a function by Node's module loader. **Debug:** relying on top-level `this` for global access is fragile precisely because it varies by context — use `globalThis` instead when you genuinely need the global object. Option D wrongly claims `globalThis` replaced these semantics; `globalThis` is a *new, separate* way to reliably reach the global object, it didn't change what `this` means.

</details>
::

::question-wrapper{language="html"}
### Q12. Given two scripts loaded with `async`, in what order do they execute relative to each other?

::code-wrapper{language="html"}
```html
<script src="a.js" async></script>
<script src="b.js" async></script>
```
::

- [ ] Always in document order: `a.js` then `b.js`
- [ ] In whichever order each script finishes downloading first — order is not guaranteed
- [ ] Always in reverse document order
- [ ] Simultaneously, on separate threads

<details>
<summary>Show Answer</summary>

**Answer:** B — In whichever order each script finishes downloading first; order is not guaranteed

**Explanation:** `async` scripts download in parallel and each executes immediately as soon as it finishes downloading, independent of the other or of document order. **Portability:** this means a smaller/faster-to-fetch `b.js` can easily execute before `a.js`, so `async` is unsafe for scripts with ordering dependencies (use `defer` for that instead). Option A describes `defer`'s guarantee, not `async`'s. Option C and D describe behaviors JavaScript's single-threaded execution model doesn't produce — script execution itself is never literally parallel/simultaneous even though downloads are.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What happens when a strict-mode function is declared with duplicate parameter names?

::code-wrapper{language="javascript"}
```javascript
"use strict";
function add(a, a, b) {
  return a + b;
}
```
::

- [ ] It runs fine; the last `a` argument silently shadows the first
- [ ] It throws a `SyntaxError` at parse time
- [ ] It throws a `TypeError` only when the function is called
- [ ] It works the same as in non-strict mode, using the first `a`

<details>
<summary>Show Answer</summary>

**Answer:** B — It throws a `SyntaxError` at parse time

**Explanation:** Non-strict mode silently allows duplicate parameter names (later ones shadow earlier ones) — a foot-gun strict mode explicitly forbids by making it a parse-time `SyntaxError`, so the code never even runs. **Safety:** the fix is simply to rename parameters uniquely. Option A and D describe the permissive non-strict behavior. Option C is wrong about timing — this is caught before execution, not at call time.

</details>
::

::question-wrapper{language="javascript"}
### Q14. How does `eval()` behave differently in strict mode versus non-strict mode regarding variable declarations?

::code-wrapper{language="javascript"}
```javascript
"use strict";
eval("var leaked = 1;");
console.log(typeof leaked);
```
::

- [ ] In strict mode, `eval` still leaks `leaked` into the surrounding scope, logging `"number"`
- [ ] In strict mode, `eval` gets its own variable scope, so `leaked` never escapes; this logs `"undefined"`
- [ ] `eval` is entirely disabled in strict mode and throws a `SyntaxError`
- [ ] Both strict and non-strict `eval` behave identically — there's no scoping difference

<details>
<summary>Show Answer</summary>

**Answer:** B — In strict mode, `eval` gets its own scope, so `leaked` never escapes; this logs `"undefined"`

**Explanation:** In non-strict mode, `var` declarations inside `eval()` leak into the calling scope — a well-known danger of `eval`. Strict mode contains this: code run via `eval` gets its own variable environment, so declarations inside it stay local. **Safety:** this is one of several reasons strict mode is recommended even when not using classes/modules. Option A describes the leaky non-strict behavior. Option C overstates it — `eval` still works in strict mode, it's just scoped. Option D ignores this real, spec-mandated difference.

</details>
::

::question-wrapper
### Q15. Is it necessary to add `"use strict"` at the top of a file that only contains ES `class` declarations and no other top-level code?

- [ ] Yes, classes are not strict by default and need the directive
- [ ] No — class bodies are always executed in strict mode regardless of any directive, though it's still good practice to add it for non-class code in the same file
- [ ] No, and adding it would cause a `SyntaxError` inside a class
- [ ] It depends on whether the class uses `extends`

<details>
<summary>Show Answer</summary>

**Answer:** B — No, class bodies are always strict regardless of a directive; still good practice for non-class code in the same file

**Explanation:** The ECMAScript spec mandates that the body of every `class` (constructor and methods) runs in strict mode automatically, independent of directives or module status. **Idiom:** if a file mixes classes with regular top-level function code, you may still want an explicit `"use strict"` (or use a module) so that *non-class* code in the same file also gets strict-mode protections. Option A misunderstands the spec rule. Option C is false — the directive is legal syntax anywhere a directive prologue is allowed. Option D invents an irrelevant condition.

</details>
::

::question-wrapper{language="html"}
### Q16. What is the idiomatic reason modern projects prefer `<script type="module">` over the older `<script nomodule>` fallback pattern?

::code-wrapper{language="html"}
```html
<script type="module" src="modern.js"></script>
<script nomodule src="legacy.js"></script>
```
::

- [ ] `nomodule` scripts always execute first regardless of load order
- [ ] The fallback pattern exists to serve modern ESM bundles to browsers that support modules and a transpiled/bundled fallback to those that don't; with legacy browser support largely dropped, many modern projects skip the fallback and ship `type="module"` only
- [ ] `type="module"` scripts cannot import other modules, so `nomodule` is required as a workaround
- [ ] `nomodule` is deprecated syntax that throws an error in all current browsers

<details>
<summary>Show Answer</summary>

**Answer:** B — The pattern serves modern ESM to capable browsers and a fallback bundle to legacy ones; many projects now skip the fallback entirely

**Explanation:** The dual-script pattern exploited the fact that browsers old enough to not understand `type="module"` also don't recognize `nomodule` as special and thus ignore it as an unknown attribute (loading the legacy bundle), while modern browsers understand `nomodule` and skip that script. **Idiom:** since evergreen browsers dominate today, many teams now ship ESM-only bundles and drop the legacy fallback and its build complexity entirely. Option A misdescribes the loading logic. Option C is false — modules import other modules constantly via `import`. Option D is false; `nomodule` isn't deprecated, it's just often unnecessary now.

</details>
::

::question-wrapper
### Q17. Why is `defer` generally preferred over placing multiple plain `<script>` tags right before `</body>`?

- [ ] `defer` scripts execute before the DOM is parsed, which is faster
- [ ] `defer` lets the browser download scripts in parallel with HTML parsing (non-blocking) while still guaranteeing document order execution after parsing — plain end-of-body scripts still block parsing when the parser reaches them
- [ ] There is no real difference; both approaches are functionally identical
- [ ] `defer` scripts run once per animation frame instead of once on load

<details>
<summary>Show Answer</summary>

**Answer:** B — `defer` downloads in parallel with parsing, non-blocking, and still preserves document order after parsing completes

**Explanation:** **Performance:** even scripts placed at the bottom of `<body>` still block parsing for the moment the parser reaches them (they just do so after most content is already visible); `defer` avoids blocking entirely by downloading concurrently with parsing and only executing once parsing is fully done, in document order. Option A gets the timing backwards — deferred scripts run *after* parsing, not before. Option C ignores the real performance and blocking difference. Option D describes something scripts don't do at all.

</details>
::

::question-wrapper{language="json"}
### Q18. Setting `"type": "module"` in a Node.js `package.json` changes what?

::code-wrapper{language="json"}
```json
{
  "name": "app",
  "type": "module"
}
```
::

- [ ] It only affects TypeScript files, not `.js` files
- [ ] It makes Node interpret `.js` files in that package as ES modules (`import`/`export`) instead of the CommonJS default, while `.cjs` files remain CommonJS regardless
- [ ] It disables `require()` globally, even in `.cjs` files
- [ ] It has no effect unless a bundler like webpack is also configured

<details>
<summary>Show Answer</summary>

**Answer:** B — It makes `.js` files in that package parse as ES modules; `.cjs` files stay CommonJS regardless

**Explanation:** Node picks a module system per file: without `"type": "module"`, `.js` defaults to CommonJS; with it set, `.js` defaults to ESM. The explicit extensions `.mjs` (always ESM) and `.cjs` (always CommonJS) override the `package.json` setting entirely — this is the escape hatch for mixed codebases. Option A is wrong; this setting is about `.js` resolution, unrelated to TypeScript compilation. Option C is too broad — `.cjs` files are unaffected. Option D is false; this is native Node behavior with zero bundler involvement.

</details>
::

::question-wrapper{language="json"}
### Q19. What is the purpose of the `"engines"` field in `package.json`?

::code-wrapper{language="json"}
```json
{
  "name": "app",
  "engines": {
    "node": ">=18.0.0"
  }
}
```
::

- [ ] It automatically installs the specified Node.js version when `npm install` runs
- [ ] It declares the Node.js (and optionally npm) version range the package expects; by default `npm install` only warns (doesn't block) if the current version doesn't match, unless `engine-strict` is enabled
- [ ] It sets the JavaScript engine (V8, SpiderMonkey, etc.) the app requires
- [ ] It is required for ES modules to work

<details>
<summary>Show Answer</summary>

**Answer:** B — It declares the expected Node/npm version range; npm warns by default rather than blocking, unless `engine-strict` is set

**Explanation:** **Idiom:** teams add `engines` to document compatibility and catch version mismatches early, but by default npm treats a mismatch as a warning, not a hard failure — many developers are surprised the install still succeeds. Setting `engine-strict=true` in `.npmrc` (or using tools like `volta`/`nvm` with `.nvmrc`) is needed to actually enforce it. Option A confuses it with a version manager's job — npm doesn't install Node versions. Option C confuses "engine" as used in `package.json` (meaning runtime version) with a JS engine like V8. Option D is unrelated; ESM support depends on the `"type"` field and Node version, not `engines`.

</details>
::

::question-wrapper
### Q20. What is the recommended way to check whether a JavaScript runtime supports a given feature (e.g., optional chaining) before using it in production code?

- [ ] Parse the `navigator.userAgent` string and branch based on browser name/version
- [ ] Use feature detection — check for the existence of the API/behavior directly (or rely on a transpiler/polyfill pipeline), rather than inferring support from the browser or engine identity
- [ ] Assume all evergreen browsers support all features equally, so no check is needed
- [ ] Wrap all code in `try/catch` and silently ignore failures

<details>
<summary>Show Answer</summary>

**Answer:** B — Use feature detection rather than inferring support from browser/engine identity

**Explanation:** **Idiom:** UA sniffing (option A) is notoriously unreliable — user agents can be spoofed, forks and embedded webviews report misleading strings, and new browser versions ship features unpredictably. Feature detection (checking `typeof obj?.prop !== 'undefined'`, `'querySelector' in document`, etc.) or using build tooling (Babel/TypeScript targets, Browserslist-driven polyfills) checks reality directly. Option C is risky since even "evergreen" runtimes lag on brand-new proposals. Option D silently swallowing errors trades a clear compatibility failure for a much harder-to-debug silent one, which is the opposite of good practice.

</details>
::
