# JavaScript: Zero to Hero

A complete quiz-based curriculum covering JavaScript from first principles to expert-level, production-grade knowledge. Every quiz is self-contained — read the explanations, not just the answers, to build a full mental model of the language.

## How to Use This Quiz Track

- Work through the parts in order; later quizzes assume knowledge from earlier ones.
- Each quiz has 20 multiple-choice questions, ordered from basic recall to edge cases to best-practice judgment calls.
- Click **Show Answer** only after committing to a choice — the explanations cover why distractors are wrong, which is where the real quizing happens.
- Questions favor "gotchas" (coercion traps, event-loop ordering, `this` binding surprises) over rote recall, so expect to be wrong sometimes on the first pass.

## Prerequisites

None for Part 1. Basic command-line comfort helps for the tooling and Node.js-flavored questions in later parts.

## Curriculum

### Part 1 — Foundations

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 01 | [Introduction & Setup](/js/01-introduction-and-setup) | Where JS runs, engines, and the ECMAScript version treadmill — context for every quirk that follows. |
| 02 | [Variables & Data Types](/js/02-variables-and-data-types) | `var`/`let`/`const` scoping and the primitive vs. reference type split drive most beginner bugs. |
| 03 | [Operators & Expressions](/js/03-operators-and-expressions) | `==` vs `===`, coercion rules, and operator precedence are classic interview traps. |
| 04 | [Control Flow](/js/04-control-flow) | Truthy/falsy edge cases and loop scoping (`let` vs `var` in closures) trip up even experienced devs. |
| 05 | [Functions & Scope](/js/05-functions-and-scope) | Hoisting, the TDZ, and function vs. arrow semantics underpin everything else in JS. |
| 06 | [Strings & Template Literals](/js/06-strings-and-template-literals) | Immutability, Unicode surrogate pairs, and tagged templates are common gotchas in string handling. |
| 07 | [Objects & Properties](/js/07-objects-and-properties) | Property descriptors, shallow vs. deep copies, and key coercion shape real-world bugs. |
| 08 | [Arrays & Array Methods](/js/08-arrays-and-array-methods) | Mutating vs. non-mutating methods and sparse arrays cause subtle production issues. |
| 09 | [Destructuring & Spread](/js/09-destructuring-and-spread) | Default values, nested patterns, and spread vs. rest are everywhere in modern codebases. |

### Part 2 — Core Language Mastery

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 10 | [Classes & Prototypes](/js/10-classes-and-prototypes) | Classes are sugar over the prototype chain — understanding both explains inheritance bugs. |
| 11 | [The `this` Keyword](/js/11-this-keyword) | `this` binding rules are the single most-misunderstood part of JavaScript. |
| 12 | [Closures & Lexical Scope](/js/12-closures-and-lexical-scope) | Closures power memoization, private state, and are a top source of loop-variable bugs. |
| 13 | [Higher-Order Functions](/js/13-higher-order-functions) | `map`/`filter`/`reduce` composition is the backbone of idiomatic modern JS. |
| 14 | [Modules](/js/14-modules) | ESM vs. CommonJS interop and circular imports are frequent real-world pain points. |
| 15 | [Callbacks & the Event Loop](/js/15-callbacks-and-the-event-loop) | Microtask vs. macrotask ordering explains why `Promise.then` beats `setTimeout(0)`. |
| 16 | [Promises](/js/16-promises) | Chaining, error propagation, and `Promise.all` vs. `allSettled` are daily async concerns. |
| 17 | [Async/Await](/js/17-async-await) | Await ordering, parallel vs. sequential awaits, and try/catch scope are frequent bugs. |
| 18 | [Error Handling](/js/18-error-handling) | Custom errors, `finally` semantics, and swallowed rejections shape reliability. |

### Part 3 — Browser & Application Layer

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 19 | [The DOM](/js/19-the-dom) | Reflow/repaint costs and live vs. static node lists affect UI performance. |
| 20 | [Events](/js/20-events) | Bubbling, capturing, and delegation are essential for building efficient UIs. |
| 21 | [Web Storage & APIs](/js/21-web-storage-and-apis) | `localStorage` quotas, synchronous cost, and Fetch API quirks matter in production. |
| 22 | [Timers & Scheduling](/js/22-timers-and-scheduling) | `setTimeout` drift, `requestAnimationFrame`, and debounce/throttle are UI staples. |
| 23 | [Functional Programming](/js/23-functional-programming) | Currying, composition, and immutability patterns are increasingly idiomatic in JS. |

### Part 4 — Advanced & Professional

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 24 | [Design Patterns](/js/24-design-patterns) | Module, singleton, observer, and factory patterns recur across every large JS codebase. |
| 25 | [Testing](/js/25-testing) | Mocking, spies, and async test pitfalls determine whether a test suite is trustworthy. |
| 26 | [Performance & Optimization](/js/26-performance-and-optimization) | Memory leaks, debouncing, and V8 optimization quirks matter at scale. |
| 27 | [Tooling & Build Systems](/js/27-tooling-and-build-systems) | Bundlers, tree-shaking, and source maps are unavoidable in modern JS delivery. |
| 28 | [TypeScript Essentials](/js/28-typescript-essentials) | Structural typing and type narrowing are now baseline JS-adjacent knowledge. |
| 29 | [Security](/js/29-security) | XSS, prototype pollution, and unsafe `eval` usage are the most common JS vulnerabilities. |
| 30 | [Exercises & Projects (Capstone)](/js/30-exercises-and-projects) | A mixed-review capstone pulling together async, closures, `this`, and performance. |
