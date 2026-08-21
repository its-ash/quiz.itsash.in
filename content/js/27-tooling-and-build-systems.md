# 27 — Tooling & Build Systems

::question-wrapper{language="javascript"}
### Q1. Why do bundlers like webpack, Vite, esbuild, or Rollup exist in a modern JavaScript project?

- [ ] They are required by the JavaScript language specification for any project that uses more than one file
- [ ] They combine and resolve a project's module graph into fewer output files, cutting down HTTP requests, and enable whole-program optimizations (like tree-shaking) that aren't possible when analyzing files one at a time
- [ ] They exist only to add TypeScript support to a project
- [ ] They replace the browser's JavaScript engine with a faster one

<details>
<summary>Show Answer</summary>

**Answer:** B — They combine and resolve a project's module graph into fewer output files, cutting down HTTP requests, and enable whole-program optimizations (like tree-shaking) that aren't possible when analyzing files one at a time

**Explanation:** Before bundlers, shipping many separate `<script>` files meant many separate HTTP round trips and no way to statically analyze which code was actually reachable. A bundler walks the `import`/`require` graph starting from an entry point, resolves every dependency, and emits a small number of optimized output files — while also unlocking cross-file analysis like tree-shaking and code splitting. Nothing in the ECMAScript spec requires a bundler (option A) — Node and modern browsers can load ESM natively — and bundlers operate on the source code, not the JS engine itself (option D). TypeScript support (option C) is a separate, optional concern usually handled by a transpiler step.

</details>
::

::question-wrapper{language="javascript"}
### Q2. Why can a bundler safely remove `multiply` from the production bundle below, but couldn't perform the same removal if `mathUtils.js` used CommonJS (`module.exports` / `require`) instead?

::code-wrapper{language="javascript"}
```javascript
// mathUtils.js
export function add(a, b) { return a + b; }
export function multiply(a, b) { return a * b; }

// app.js
import { add } from './mathUtils.js';
console.log(add(2, 3));
```
::

- [ ] Because ESM import/export bindings are static and analyzable at build time, so the bundler can prove `multiply` is never referenced anywhere in the graph; `require()` is an ordinary function call that can execute conditionally or dynamically, so the bundler can't prove a given export is unused
- [ ] Because ESM files are always smaller in byte size than CommonJS files
- [ ] Because CommonJS modules cannot be included in a bundle at all
- [ ] Because tree-shaking only works when the project also uses TypeScript

<details>
<summary>Show Answer</summary>

**Answer:** A — Because ESM import/export bindings are static and analyzable at build time, so the bundler can prove `multiply` is never referenced anywhere in the graph; `require()` is an ordinary function call that can execute conditionally or dynamically, so the bundler can't prove a given export is unused

**Explanation:** ESM `import`/`export` statements must appear at the top level with a fixed, literal set of names, so a bundler can build a complete, static picture of what's exported and what's actually imported and used — anything unreferenced is provably dead code. CommonJS's `require()` and `module.exports` are just regular JavaScript values and function calls; they can be wrapped in conditionals, computed with variables, or reassigned at runtime, so a bundler generally cannot prove a given property will never be accessed and must keep the whole module. This is the core mechanical reason "tree-shaking needs ESM" — it's not a licensing or file-size issue (options B, C are false), and it has nothing to do with TypeScript (option D).

</details>
::

::question-wrapper{language="json"}
### Q3. What does setting `"sideEffects": false` in a library's `package.json` tell a bundler like webpack, and what risk does it introduce if the claim is wrong?

::code-wrapper{language="json"}
```json
{
  "name": "date-utils",
  "sideEffects": false
}
```
::

- [ ] It disables minification for that specific package during the build
- [ ] It forces the bundler to always include the entire package in the output, regardless of whether it's used
- [ ] It tells the bundler that importing any module in this package triggers no meaningful top-level side effects, so unused exports (and even entire unused modules) can be safely dropped; if a module secretly does something at import time — e.g. `window.foo = ...` executed at the top level — the bundler may strip it out anyway, silently breaking behavior that depended on it running
- [ ] It tells npm to skip running the package's install scripts

<details>
<summary>Show Answer</summary>

**Answer:** C — It tells the bundler that importing any module in this package triggers no meaningful top-level side effects, so unused exports (and even entire unused modules) can be safely dropped; if a module secretly does something at import time — e.g. `window.foo = ...` executed at the top level — the bundler may strip it out anyway, silently breaking behavior that depended on it running

**Explanation:** By default, bundlers must conservatively assume any module might do something important just by being imported (register a polyfill, patch a global, run an analytics ping), so they keep it even if none of its exports are used. `"sideEffects": false` is the package author's explicit promise that no module in the package needs to run "just for its side effects," which lets the bundler drop whole unused files, not just unused exports. It's a real footgun: if that promise is false for even one file, tree-shaking can delete code the app silently relied on. It has nothing to do with minification (A), it does the opposite of "always include everything" (B), and it's unrelated to npm lifecycle scripts (D).

</details>
::

::question-wrapper{language="javascript"}
### Q4. What problem do source maps solve when debugging a production JavaScript app in the browser's devtools?

- [ ] They speed up the browser's JavaScript engine execution at runtime
- [ ] They map positions in the minified/transpiled output back to the corresponding location in the original source, so devtools can display original file names, line numbers, and variable names instead of one unreadable minified line
- [ ] They automatically fix runtime errors before they can occur
- [ ] They replace the need for a transpilation step entirely

<details>
<summary>Show Answer</summary>

**Answer:** B — They map positions in the minified/transpiled output back to the corresponding location in the original source, so devtools can display original file names, line numbers, and variable names instead of one unreadable minified line

**Explanation:** A `.map` file (or an inline base64 data-URI version of it) encodes a position-by-position mapping between the shipped output and the original authored source. When an error is thrown or a breakpoint is set in the minified bundle, devtools consult the source map to show you the original, human-readable source instead of a single 200KB line of renamed variables. It's purely a debugging aid — it has no effect on runtime speed (A), can't prevent errors from happening (C), and is unrelated to whether transpilation is needed (D).

</details>
::

::question-wrapper{language="bash"}
### Q5. A production bundle ends with the line shown below, and the corresponding `.map` file is deployed alongside it on the public web server.

::code-wrapper{language="javascript"}
```javascript
//# sourceMappingURL=app.min.js.map
```
::

What is the practical risk of doing this on a public-facing deployment?

- [ ] There is no risk — browsers silently ignore this comment outside of development mode
- [ ] It causes the bundled JavaScript to execute twice on page load
- [ ] It prevents Hot Module Replacement from working correctly in the next local development session
- [ ] Anyone can open devtools, let it fetch `app.min.js.map`, and reconstruct the full original, unminified source — including original file structure and variable names — because the map file embeds the complete original source content; teams that want minification's obfuscation benefit typically exclude source maps from public deploys, or upload them privately to an error-tracking service instead

<details>
<summary>Show Answer</summary>

**Answer:** D — Anyone can open devtools, let it fetch `app.min.js.map`, and reconstruct the full original, unminified source — including original file structure and variable names — because the map file embeds the complete original source content; teams that want minification's obfuscation benefit typically exclude source maps from public deploys, or upload them privately to an error-tracking service instead

**Explanation:** **Debug** / **Security** — A source map's whole job is to reverse minification, so if it's publicly reachable it fully defeats any obfuscation minification provided: the `sourcesContent` field literally embeds the original file text. This comment is *not* ignored outside dev (option A is false) — any browser's devtools will follow it whenever devtools are open, in any environment. It has no effect on double execution (B) or on a *future* dev session's HMR (C); those are unrelated mechanisms. The common production pattern is to still generate source maps (for your own error-tracking pipeline, e.g. Sentry) but upload them out-of-band and keep them off the public server, or omit the `sourceMappingURL` comment from the publicly served file.

</details>
::

::question-wrapper{language="javascript"}
### Q6. A teammate says "we don't need Babel anymore since we already have webpack in the build." What's the flaw in that reasoning?

- [ ] Bundling (resolving and combining a module graph, enabling tree-shaking and code splitting) and transpiling (rewriting newer syntax like optional chaining or class fields into syntax older engines can parse) are different concerns; webpack orchestrates the build and can invoke Babel as a loader, but without Babel (or an equivalent transform) newer syntax passes through unchanged and will throw a `SyntaxError` in engines that don't support it
- [ ] Nothing is wrong — webpack and Babel perform the exact same job, so either one alone is sufficient
- [ ] Babel is only relevant for TypeScript projects, so it was never needed here regardless
- [ ] webpack cannot process any JavaScript file at all without Babel installed

<details>
<summary>Show Answer</summary>

**Answer:** A — Bundling (resolving and combining a module graph, enabling tree-shaking and code splitting) and transpiling (rewriting newer syntax like optional chaining or class fields into syntax older engines can parse) are different concerns; webpack orchestrates the build and can invoke Babel as a loader, but without Babel (or an equivalent transform) newer syntax passes through unchanged and will throw a `SyntaxError` in engines that don't support it

**Explanation:** webpack's core job is graph resolution and output generation; it has no built-in opinion about JavaScript *syntax* compatibility on its own. Babel's job is purely syntactic: parse newer JS, emit an equivalent AST expressed in older-compatible syntax (plus polyfills for missing runtime features, via something like `core-js`). Plenty of projects use webpack with zero syntax-lowering — perfectly fine if you only target evergreen browsers — but the moment you need to support an older runtime, dropping Babel means shipping syntax that engine can't parse, which fails at parse time, not gracefully. Options B, C, and D each conflate two genuinely separate tools with separate responsibilities.

</details>
::

::question-wrapper{language="javascript"}
### Q7. Before minification:

::code-wrapper{language="javascript"}
```javascript
const greet = (name) => `Hello, ${name}!`;
```
::

After minification:

::code-wrapper{language="javascript"}
```javascript
const greet=n=>`Hello, ${n}!`;
```
::

A developer assumed minifying this file would make it compatible with an ES5-only browser (which doesn't support arrow functions or template literals). Why is that assumption wrong?

- [ ] Minifiers remove all modern syntax by definition, so this specific case must be a bug in the minifier
- [ ] Minifiers and transpilers are simply two different marketing names for the same underlying tool
- [ ] Minification only shrinks code — renaming variables, stripping whitespace and comments, and similar size-focused rewrites — without changing which language-version features the code uses; the arrow function and template literal survive untouched, so an ES5-only engine still throws a `SyntaxError`. Only transpilation (e.g. via Babel) rewrites syntax down to an older-compatible form
- [ ] The minified code is actually less compatible with any browser than the unminified original

<details>
<summary>Show Answer</summary>

**Answer:** C — Minification only shrinks code — renaming variables, stripping whitespace and comments, and similar size-focused rewrites — without changing which language-version features the code uses; the arrow function and template literal survive untouched, so an ES5-only engine still throws a `SyntaxError`. Only transpilation (e.g. via Babel) rewrites syntax down to an older-compatible form

**Explanation:** **Idiom** — Notice the minified output still contains `=>` and a template-literal backtick — a minifier's job (tools like Terser) is purely about *output size*, not language-version compatibility. It happily shortens `name` to `n` and drops whitespace, but it has no concept of "rewrite this arrow function as a `function` expression for older engines" — that's transpilation's job, a semantically different transform. Confusing the two is a real production trap: teams sometimes ship a minified-but-not-transpiled bundle assuming "smaller" implies "more compatible," and it silently breaks on the exact old browsers they meant to support.

</details>
::

::question-wrapper{language="javascript"}
### Q8. What does Hot Module Replacement (HMR) in a dev server (like Vite's or webpack-dev-server's) do that a full page reload on save does not?

- [ ] It permanently deploys the current change straight to production automatically
- [ ] It swaps only the updated module(s) into the already-running application in the browser, preserving in-memory state — a form's typed-in text, a Redux store, an open modal's state — that a full page reload would otherwise wipe out
- [ ] It compiles the app faster, but the browser still performs a full page reload on every save regardless
- [ ] It works only for CSS files and has no effect on JavaScript module updates

<details>
<summary>Show Answer</summary>

**Answer:** B — It swaps only the updated module(s) into the already-running application in the browser, preserving in-memory state — a form's typed-in text, a Redux store, an open modal's state — that a full page reload would otherwise wipe out

**Explanation:** HMR relies on the dev server pushing just the changed module's new code over a persistent connection (typically a WebSocket), and a small runtime in the page replaces that module in place, re-running only what's needed to apply the update, without tearing down the whole JS execution context. That's precisely why it's valuable during development of stateful UI: you can tweak a component's rendering logic and see the change instantly without losing whatever state you'd navigated into. Option A confuses a dev-only mechanism with deployment. Option C describes what a fast *full reload* setup does, which is a different (and less state-preserving) feature. Option D is wrong — HMR is commonly used for JS component updates too, not just CSS.

</details>
::

::question-wrapper{language="json"}
### Q9. Two developers run `npm install` on different days, using only the `package.json` snippet below — with no `package-lock.json` present or committed to the repository.

::code-wrapper{language="json"}
```json
{
  "dependencies": {
    "left-pad": "^1.3.0"
  }
}
```
::

Why might they end up with different actual installed versions, and what does committing a lockfile (`package-lock.json`, `yarn.lock`, or `pnpm-lock.yaml`) fix?

- [ ] `npm install` always installs the exact same version regardless of lockfiles, so this scenario is impossible
- [ ] Lockfiles only pin versions for `devDependencies`, never for regular `dependencies`, so this scenario would happen with or without one
- [ ] The `^` symbol actually means "exact version only," so both installs are already guaranteed to be identical without any lockfile
- [ ] `^1.3.0` permits any `1.x.x` release equal to or newer than `1.3.0` (excluding `2.0.0`), so a new compatible version published between the two install dates — or a transitive dependency resolving its own range differently — can produce a different dependency tree on each machine; a committed lockfile pins the exact resolved version (and the full transitive tree) so every install reproduces identical versions

<details>
<summary>Show Answer</summary>

**Answer:** D — `^1.3.0` permits any `1.x.x` release equal to or newer than `1.3.0` (excluding `2.0.0`), so a new compatible version published between the two install dates — or a transitive dependency resolving its own range differently — can produce a different dependency tree on each machine; a committed lockfile pins the exact resolved version (and the full transitive tree) so every install reproduces identical versions

**Explanation:** `package.json` alone only expresses *acceptable ranges*, not exact versions — that's true by design, so authors can receive compatible patches automatically. The reproducibility problem is that "acceptable range" can resolve to a different concrete version depending on *when* you install and what else is in the dependency tree at that moment, including nested (transitive) dependencies with their own ranges. A lockfile freezes the entire resolved tree — exact versions and their exact sub-dependencies — so `npm ci` (or the equivalent) on any machine, at any later date, installs byte-for-byte the same tree. Option A is simply false in the absence of a lockfile; option B misdescribes lockfile scope (they cover all dependency types); option C misstates what `^` means (it is a *range*, not an exact pin).

</details>
::

::question-wrapper{language="javascript"}
### Q10. A library bumps its published version from `2.4.1` straight to `3.0.0`, with no other announcement. Under semantic versioning (semver) conventions, what should consumers assume?

- [ ] The release contains a backwards-incompatible ("breaking") change to the public API — code that worked against `2.x` may need modification to work against `3.0.0`; by contrast, a `2.4.1` → `2.5.0` bump would promise new backwards-compatible features, and → `2.4.2` would promise only backwards-compatible bug fixes
- [ ] Nothing meaningfully changed except bug fixes, so it's safe to upgrade blindly
- [ ] The major version number is purely cosmetic marketing and carries no contractual meaning under semver
- [ ] It means the package now requires a completely different JavaScript runtime, such as switching from Node.js to Deno

<details>
<summary>Show Answer</summary>

**Answer:** A — The release contains a backwards-incompatible ("breaking") change to the public API — code that worked against `2.x` may need modification to work against `3.0.0`; by contrast, a `2.4.1` → `2.5.0` bump would promise new backwards-compatible features, and → `2.4.2` would promise only backwards-compatible bug fixes

**Explanation:** Semver's `MAJOR.MINOR.PATCH` scheme is a contract: PATCH bumps promise backwards-compatible bug fixes only, MINOR bumps promise backwards-compatible new functionality, and MAJOR bumps are the *only* place a breaking change is allowed to happen under the convention — so a `2.x` → `3.0.0` jump is the explicit signal to consult a changelog/migration guide before upgrading, not to upgrade blindly (ruling out option B). The version number is meaningful, not cosmetic (ruling out C), and a major bump says nothing by itself about runtime requirements (ruling out D) — that would be called out separately, e.g. in an `engines` field.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What does a bundler typically do with the dynamic `import()` call below that it would not do with a static `import { openModal } from './modal.js'` placed at the top of the file?

::code-wrapper{language="javascript"}
```javascript
button.addEventListener('click', async () => {
  const { openModal } = await import('./modal.js');
  openModal();
});
```
::

- [ ] It ignores the dynamic import entirely and bundles `modal.js` into the main bundle exactly as if it had been statically imported
- [ ] It runs the code inside `modal.js` on a separate operating-system thread automatically
- [ ] It extracts `modal.js` (and its own dependencies) into a separate chunk file that's only fetched over the network when the click handler actually executes, instead of being downloaded as part of the initial page load — shrinking the initial bundle
- [ ] It silently converts every export inside `modal.js` from ESM syntax into CommonJS syntax

<details>
<summary>Show Answer</summary>

**Answer:** C — It extracts `modal.js` (and its own dependencies) into a separate chunk file that's only fetched over the network when the click handler actually executes, instead of being downloaded as part of the initial page load — shrinking the initial bundle

**Explanation:** A dynamic `import()` call returns a promise and is a recognized signal to essentially every modern bundler that the target module and its subgraph should become their own on-demand chunk, fetched lazily at runtime rather than being force-included in the entry bundle. This is the mechanism behind route-based and feature-based code splitting — code the user might never trigger (like a rarely opened modal) never has to be downloaded until it's needed. It doesn't run on a separate thread (B is a misconception conflating this with Web Workers), it's not ignored (A is simply the opposite of what code splitting is for), and it doesn't force a module-format conversion (D) — the output format is a build configuration choice, unrelated to whether the import is static or dynamic.

</details>
::

::question-wrapper{language="javascript"}
### Q12. A team's CI pipeline runs both ESLint and Prettier. A developer asks: "isn't that redundant — don't they both just check code style?" What's the accurate distinction?

- [ ] Prettier is the one that catches actual bugs, while ESLint only reformats whitespace
- [ ] ESLint performs static analysis to catch potential bugs and enforce code-quality rules (e.g. unused variables, unreachable code, misuse of `==`), while Prettier is purely an opinionated code formatter that rewrites whitespace, quote style, and line breaks without understanding code semantics; they're complementary tools, and teams commonly disable ESLint's own formatting-related rules (e.g. via `eslint-config-prettier`) so the two don't fight over the same lines
- [ ] Yes, it's redundant — either tool alone already provides everything the other one does
- [ ] Only one of the two tools is actually allowed to run inside a CI pipeline at a time

<details>
<summary>Show Answer</summary>

**Answer:** B — ESLint performs static analysis to catch potential bugs and enforce code-quality rules (e.g. unused variables, unreachable code, misuse of `==`), while Prettier is purely an opinionated code formatter that rewrites whitespace, quote style, and line breaks without understanding code semantics; they're complementary tools, and teams commonly disable ESLint's own formatting-related rules (e.g. via `eslint-config-prettier`) so the two don't fight over the same lines

**Explanation:** **Idiom** — ESLint parses code into an AST and applies rules that reason about *meaning* — "this variable is never read," "this promise is never awaited," "this `switch` case falls through unintentionally" — the kind of issue that can indicate an actual bug. Prettier has no opinion on any of that; it only reprints code in a single canonical style so diffs and reviews aren't cluttered by formatting bikeshedding. Running both isn't redundant, it's standard practice — but running ESLint's *formatting* rules alongside Prettier's formatting *would* be redundant (and can conflict), which is why teams disable that overlapping subset specifically.

</details>
::

::question-wrapper{language="javascript"}
### Q13. A team's CI pipeline for a JS project runs, in order: install dependencies → lint → unit tests → build. A new hire asks why the build step — often the slowest — runs last instead of first. What's the reasoning?

- [ ] The build must run last because bundlers are only able to process code that has already passed its unit tests
- [ ] Lint and test are required to run after the build because they need the already-compiled output to work against
- [ ] The step order is arbitrary and has no measurable effect on how quickly the pipeline reports a failure
- [ ] Running the fastest, cheapest checks first — lint typically finishes in seconds — means the pipeline fails fast on trivial issues before spending time on slower steps; if lint fails, there's no reason to wait through a multi-minute build (or a slower test suite) just to discover a problem that a quick static check already caught

<details>
<summary>Show Answer</summary>

**Answer:** D — Running the fastest, cheapest checks first — lint typically finishes in seconds — means the pipeline fails fast on trivial issues before spending time on slower steps; if lint fails, there's no reason to wait through a multi-minute build (or a slower test suite) just to discover a problem that a quick static check already caught

**Explanation:** CI ordering is usually optimized for "fail fast": cheap, high-signal checks run first so a broken commit is rejected in seconds rather than after minutes of unnecessary work. Lint is typically the fastest (pure static analysis, no execution), tests are next (they execute code but are usually scoped and parallelizable), and a full production build is often the slowest step because it involves bundling, minification, and sometimes type-checking a whole project. Neither lint nor tests structurally depend on build output in a typical JS setup (ruling out B), and the build doesn't require passing tests to *run* — teams just choose not to waste time building a commit that's already known to be broken (ruling out A). Ordering absolutely affects feedback latency, so C is false too.

</details>
::

::question-wrapper{language="javascript"}
### Q14. A frontend team changes the `VITE_API_URL` environment variable on their server after already deploying a built Vite app, expecting the running app to pick up the new URL on the next page load without a rebuild.

::code-wrapper{language="javascript"}
```javascript
console.log(import.meta.env.VITE_API_URL);
```
::

Why doesn't this work?

- [ ] `import.meta.env.VITE_API_URL` is resolved at build time — Vite statically replaces that expression with the literal value present in the environment when `vite build` ran, baking it directly into the generated static JS files; there's no runtime process left in the browser to re-read a server environment variable, so the already-deployed static files must be rebuilt (and redeployed) for a new value to take effect
- [ ] Environment variables are always read live from the server on every page load in any frontend framework, so this should already be working
- [ ] `import.meta.env` never reads from environment variables at all — it only reads from a static config file that has nothing to do with the server environment
- [ ] The value only updates the next time the browser itself is restarted, but otherwise updates automatically on every request

<details>
<summary>Show Answer</summary>

**Answer:** A — `import.meta.env.VITE_API_URL` is resolved at build time — Vite statically replaces that expression with the literal value present in the environment when `vite build` ran, baking it directly into the generated static JS files; there's no runtime process left in the browser to re-read a server environment variable, so the already-deployed static files must be rebuilt (and redeployed) for a new value to take effect

**Explanation:** **Debug** — This is a common point of confusion carried over from backend habits, where `process.env.X` really is read live at runtime by a running Node process. In a bundled frontend app, `import.meta.env.VITE_API_URL` (or Create React App's `process.env.REACT_APP_X`) is a compile-time substitution: the bundler literally replaces that expression with a hardcoded string in the emitted JS during the build step, because there is no server-side process running in the user's browser to consult a live environment variable. Changing the server's env var after the fact changes nothing about files that were already generated and shipped — a fresh build is required. Options B, C, and D each describe behavior that doesn't apply to a statically bundled frontend.

</details>
::

::question-wrapper{language="javascript"}
### Q15. A developer replaces a large date-handling library with a smaller alternative, expecting the production bundle to shrink as a result — but never actually checks. What's the best-practice next step before assuming the optimization worked?

- [ ] Trust the change without verification, since swapping to a "smaller" library logically implies a smaller bundle
- [ ] Only check the bundle size in a staging environment, and never verify it locally during development
- [ ] Inspect the actual build output — e.g. with a bundle analyzer (webpack-bundle-analyzer, or Vite's `rollup-plugin-visualizer`) or by diffing the output file sizes before and after — since the real bundle size depends on how well the bundler tree-shakes both libraries, whether other code still imports the old library transitively, and whether duplicate copies get pulled in through other dependencies; assumptions about size are frequently wrong until actually measured
- [ ] Assume bundle size no longer matters at all once gzip or Brotli compression is applied by the server

<details>
<summary>Show Answer</summary>

**Answer:** C — Inspect the actual build output — e.g. with a bundle analyzer (webpack-bundle-analyzer, or Vite's `rollup-plugin-visualizer`) or by diffing the output file sizes before and after — since the real bundle size depends on how well the bundler tree-shakes both libraries, whether other code still imports the old library transitively, and whether duplicate copies get pulled in through other dependencies; assumptions about size are frequently wrong until actually measured

**Explanation:** **Performance** — Swapping libraries feels like an obvious win, but the actual result depends on build-tool behavior you can't fully predict from source code alone: maybe another dependency still transitively pulls in the old library (so it never actually left the bundle), maybe the "smaller" library tree-shakes worse in practice, or maybe a duplicate version sneaks in through a nested dependency's own lockfile. A bundle analyzer visualizes exactly what ended up in the output and how large each module actually is, turning an assumption into a measurement. Compression (option D) reduces bytes over the wire but doesn't change parse/execution cost, so it doesn't make bundle size irrelevant, and skipping local checks (option B) just delays discovering a mistake.

</details>
::

::question-wrapper{language="javascript"}
### Q16. `app.js` imports `analytics.js` purely for its side effect and never calls `track`. Assuming the `analytics` package's `package.json` does *not* set `"sideEffects": false`, what happens during tree-shaking?

::code-wrapper{language="javascript"}
```javascript
// analytics.js
console.log('Analytics module loaded');
export function track(event) { /* ... */ }

// app.js
import './analytics.js';
```
::

- [ ] The bundler removes the entire `analytics.js` module, since `track` is never referenced anywhere
- [ ] The bundler must keep the whole module — including the unused `track` function — because it can't prove the top-level `console.log` call is free of observable side effects, and running it is required for correctness; tree-shaking only removes code the bundler can prove is both unused *and* side-effect-free to skip
- [ ] The build fails with an error, because top-level side effects are disallowed in ES modules
- [ ] Only the `console.log` line is stripped from the output, while `track` is kept regardless

<details>
<summary>Show Answer</summary>

**Answer:** B — The bundler must keep the whole module — including the unused `track` function — because it can't prove the top-level `console.log` call is free of observable side effects, and running it is required for correctness; tree-shaking only removes code the bundler can prove is both unused *and* side-effect-free to skip

**Explanation:** **Debug** — Tree-shaking's soundness guarantee requires proving a piece of code has *no observable effect* if removed. A statement like `console.log(...)` sitting directly at module scope executes the moment the module is evaluated, and removing it would change observable program behavior (the log line simply wouldn't appear) — so the bundler conservatively keeps the whole module, `track` included, even though `track` itself is never called. This is exactly the scenario `"sideEffects": false` (see Q3) exists to override: it lets the package author explicitly promise no module needs to run "just for its side effects," unlocking the removal the bundler otherwise can't safely perform on its own. ES modules impose no such restriction (ruling out C), and bundlers don't selectively remove single statements from a kept module while leaving unrelated exports (ruling out D).

</details>
::

::question-wrapper{language="javascript"}
### Q17. A project ends up with both a `package-lock.json` and a `pnpm-lock.yaml` committed, after a teammate ran a different package manager locally by mistake. What's the practical problem?

- [ ] `pnpm-lock.yaml` always silently takes priority over `package-lock.json`, in every package manager, so nothing actually breaks
- [ ] Lockfiles are optional metadata that npm, yarn, and pnpm all ignore by default during install, so committing more than one changes nothing
- [ ] No problem exists — every major package manager automatically merges and reconciles any lockfile format it finds
- [ ] Each package manager reads and trusts only its own lockfile format, so mixing formats causes inconsistent dependency resolution between teammates and CI depending on which tool happens to run, defeating the reproducibility guarantee lockfiles exist to provide; teams typically enforce a single package manager (e.g. via a `packageManager` field or a CI check) and `.gitignore` the rest

<details>
<summary>Show Answer</summary>

**Answer:** D — Each package manager reads and trusts only its own lockfile format, so mixing formats causes inconsistent dependency resolution between teammates and CI depending on which tool happens to run, defeating the reproducibility guarantee lockfiles exist to provide; teams typically enforce a single package manager (e.g. via a `packageManager` field or a CI check) and `.gitignore` the rest

**Explanation:** npm only consults `package-lock.json`, pnpm only consults `pnpm-lock.yaml`, and yarn only consults `yarn.lock` — none of them read or reconcile a rival tool's lockfile. If both files sit in the repo, whichever teammate (or CI job) happens to run `npm install` gets npm's resolution, while whoever runs `pnpm install` gets pnpm's — potentially different dependency trees, entirely defeating the point of having a lockfile at all. There's no automatic merging or precedence between formats (ruling out A and C), and lockfiles are very much respected by default, not ignored (ruling out B) — that's the entire reason this mismatch causes a real problem instead of a harmless no-op.

</details>
::

::question-wrapper{language="javascript"}
### Q18. Vite uses esbuild to pre-bundle dependencies and serve native ESM during development, but switches to Rollup for the production build. Given that esbuild is dramatically faster than Rollup, why not just use esbuild for production too?

- [ ] esbuild's plugin ecosystem and fine-grained output control (advanced code-splitting strategies, broader plugin compatibility, more mature tree-shaking edge-case handling) have historically lagged Rollup's, so Vite trades esbuild's raw speed for Rollup's more battle-tested, flexible production output; using native ESM plus esbuild's fast transforms in dev is fine there because dev-server responsiveness matters more than squeezing out the optimal final bundle shape
- [ ] esbuild is a CSS-only tool and cannot process JavaScript at all
- [ ] Rollup is required because esbuild is fundamentally incapable of running inside a Node.js process
- [ ] The two tools always produce byte-for-byte identical output, so the choice between them is arbitrary either way

<details>
<summary>Show Answer</summary>

**Answer:** A — esbuild's plugin ecosystem and fine-grained output control (advanced code-splitting strategies, broader plugin compatibility, more mature tree-shaking edge-case handling) have historically lagged Rollup's, so Vite trades esbuild's raw speed for Rollup's more battle-tested, flexible production output; using native ESM plus esbuild's fast transforms in dev is fine there because dev-server responsiveness matters more than squeezing out the optimal final bundle shape

**Explanation:** **Idiom** — This is a genuine, deliberate design trade-off in Vite's architecture: development mode optimizes for near-instant server start and updates (esbuild transforms individual files on the fly, and the browser handles module resolution natively via ESM `import`), while a production build optimizes for the *shape* and correctness of the final shipped bundle, where Rollup's more mature plugin API and configurable output (chunking strategy, more thorough dead-code elimination in edge cases) matter more than raw build speed. esbuild absolutely processes JavaScript, not just CSS (ruling out B), and runs fine under Node (ruling out C) — Vite itself uses it there. The two tools do not produce identical output (ruling out D); if they did, there'd be no reason for Vite to use two different bundlers in the first place.

</details>
::

::question-wrapper{language="javascript"}
### Q19. Importing the whole `lodash` package (published as CommonJS, exposed as one default-exported object) typically leaves the entire library in the production bundle, while importing from `lodash-es` and destructuring lets unused functions be tree-shaken away. Why?

::code-wrapper{language="javascript"}
```javascript
import _ from 'lodash';
console.log(_.debounce);
```
::

::code-wrapper{language="javascript"}
```javascript
import { debounce } from 'lodash-es';
```
::

- [ ] `lodash-es` is simply a smaller library that ships fewer functions than `lodash`
- [ ] Bundlers cannot process CommonJS packages at all, so the first snippet would actually fail to build
- [ ] CommonJS's `module.exports = { ... }` produces a single, dynamically-constructed export value that a bundler can't statically split apart, so `import _ from 'lodash'` pulls in that whole object as one indivisible unit; `lodash-es` ships genuine ESM named exports per function, so the bundler's static analysis can see exactly which named export (`debounce`) is actually referenced and drop the rest
- [ ] The difference only matters for local development builds and has no effect once a production build is generated

<details>
<summary>Show Answer</summary>

**Answer:** C — CommonJS's `module.exports = { ... }` produces a single, dynamically-constructed export value that a bundler can't statically split apart, so `import _ from 'lodash'` pulls in that whole object as one indivisible unit; `lodash-es` ships genuine ESM named exports per function, so the bundler's static analysis can see exactly which named export (`debounce`) is actually referenced and drop the rest

**Explanation:** **Performance** — This is the same underlying mechanism as Q2 and Q16, applied to a real-world library people actually hit in production. `lodash`'s CommonJS build exports one big object literal at runtime — from the bundler's static perspective, that's an opaque value, not a set of individually analyzable bindings, so accessing `_.debounce` gives no static guarantee the other ~300 functions are unused. `lodash-es` restructures the exact same functionality as individual ESM named exports, which *are* statically analyzable, so `import { debounce } from 'lodash-es'` lets the bundler prove every other export is dead code and drop it. `lodash` and `lodash-es` contain the same functions (ruling out A); bundlers handle CommonJS routinely, just without tree-shaking granularity (ruling out B); and this affects production bundle size specifically, since that's when tree-shaking is applied (ruling out D).

</details>
::

::question-wrapper{language="javascript"}
### Q20. After switching to a production build, a team notices their bundle size barely shrank despite the app only using a handful of functions from a large utility library. Based on everything above, which of the following is the **LEAST likely** actual cause?

- [ ] The library is published only as CommonJS, so its exports can't be statically split apart and tree-shaken by the bundler
- [ ] The library's `package.json` is missing `"sideEffects": false` (or declares it incorrectly), so the bundler conservatively keeps modules it can't prove are free of side effects
- [ ] Nobody on the team actually inspected the build output with a bundle analyzer, so an earlier attempted fix may not have taken effect, or an entirely different dependency is the real source of the bloat
- [ ] The bundler's minifier failed to rename any local variables during minification, inflating the file size

<details>
<summary>Show Answer</summary>

**Answer:** D — The bundler's minifier failed to rename any local variables during minification, inflating the file size

**Explanation:** Variable-renaming is a minification detail that shaves bytes off *already-included* code — it has essentially nothing to do with why unused exports from a library would fail to be removed from the bundle in the first place, which is squarely a tree-shaking question, not a minification one (see Q7's distinction between the two). By contrast, options A, B, and C are all genuine, common root causes of "tree-shaking didn't work": a CommonJS-only library defeats static analysis outright (Q2, Q19); a missing or incorrect `sideEffects` flag forces the bundler to keep modules it can't prove are safe to drop (Q3, Q16); and skipping verification with a real bundle analyzer (Q15) means the team is guessing rather than diagnosing, which is often itself the actual blocker to fixing the real cause.

</details>
::
