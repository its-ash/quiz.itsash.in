# Lua: Zero to Hero

A complete quiz-based curriculum covering Lua from first principles to expert-level, production-grade knowledge. Lua's minimalism is deceptive — nearly everything (objects, modules, namespaces) is built from one primitive, the table, and this track is sequenced around the handful of mechanisms (metatables, closures, coroutines) that make that possible.

## How to Use This Quiz Track

- Work through the parts in order; Part 2 onward assumes comfort with tables and closures from Part 1.
- Each quiz has 20 multiple-choice questions, ordered from basic recall to edge cases to best-practice judgment calls.
- Click **Show Answer** only after committing to a choice — the explanations cover the underlying mechanism (metamethods, upvalue sharing, coroutine state machines), not just the right letter.
- Expect classic Lua "wat" moments — 1-based indexing, the `#` operator on tables with holes, and `..` string coercion — turned into quiz questions.
- Questions target Lua 5.4 (the current reference implementation) and call out version differences (5.1/5.2/5.3, LuaJIT) where they matter in practice.

## Prerequisites

None for Part 1, though familiarity with any dynamically-typed scripting language helps the table/closure model click faster.

## Curriculum

### Part 1 — Foundations

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 01 | [Introduction & Setup](/lua/01-introduction-and-setup) | `lua` vs `luajit`, embedding vs standalone, and version fragmentation (5.1–5.4) shape which features and libraries are actually available. |
| 02 | [Variables & Data Types](/lua/02-variables-and-data-types) | Globals-by-default, `nil`, and the integer/float subtype split (5.3+) are frequent sources of subtle bugs. |
| 03 | [Operators & Expressions](/lua/03-operators-and-expressions) | String-to-number coercion via arithmetic operators and `..` concatenation are classic gotcha territory. |
| 04 | [Control Flow](/lua/04-control-flow) | `repeat...until`'s scoping quirk and the absence of `break`-with-label (only `goto`) surprise newcomers. |
| 05 | [Strings & Patterns](/lua/05-strings-and-patterns) | Lua patterns are not regex — the differences trip up anyone assuming PCRE-style syntax. |
| 06 | [Tables](/lua/06-tables) | The array/hash duality and the famously undefined `#` operator on tables with holes are the single biggest source of Lua bugs. |

### Part 2 — Functions & Scope

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 07 | [Functions](/lua/07-functions) | Multiple return values and varargs (`...`) are used pervasively and behave differently from most mainstream languages. |
| 08 | [Closures & Upvalues](/lua/08-closures-and-upvalues) | Shared upvalues across loop iterations are Lua's version of the classic JS `var`-in-a-loop bug. |
| 09 | [Metatables & Metamethods](/lua/09-metatables-and-metamethods) | `__index`/`__newindex` and operator metamethods are how Lua achieves everything from OOP to operator overloading. |
| 10 | [OOP with Tables & Metatables](/lua/10-oop-with-tables-and-metatables) | Colon-call sugar (`obj:method()`) hides an implicit `self` that's a frequent source of `attempt to call a nil value` errors. |

### Part 3 — Control & Errors

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 11 | [Error Handling](/lua/11-error-handling) | `pcall`/`xpcall` and non-string error objects are Lua's whole error model — there's no `try`/`catch`. |
| 12 | [Coroutines](/lua/12-coroutines) | `resume`/`yield` state machines power generators, cooperative schedulers, and game-engine update loops. |
| 13 | [Modules & require](/lua/13-modules-and-require) | `require` caching and `package.path` resolution order are essential for structuring any non-trivial project. |

### Part 4 — Advanced & Systems

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 14 | [Garbage Collection](/lua/14-garbage-collection) | Incremental vs. generational GC (5.4) and weak tables matter for anyone building caches or long-running processes. |
| 15 | [The C API & FFI](/lua/15-the-c-api-and-ffi) | Lua's defining use case is embedding — the stack-based C API and LuaJIT's FFI are how host applications actually integrate it. |
| 16 | [Advanced Metaprogramming](/lua/16-advanced-metaprogramming) | `__call`, `__tostring`, and proxy tables via `__index`/`__newindex` enable sandboxing and DSL-style APIs. |
| 17 | [Standard Library Deep Dive](/lua/17-standard-library-deep-dive) | `table.sort` stability, `os.time` granularity, and `io` buffering behavior matter in real scripts. |
| 18 | [Performance & LuaJIT](/lua/18-performance-and-luajit) | JIT trace compilation, deoptimization triggers, and table pre-sizing are where Lua performance work actually happens. |
| 19 | [Testing](/lua/19-testing) | Frameworks like `busted` and `luaunit`, plus common assertion/mocking patterns, determine test suite trustworthiness. |
| 20 | [Security](/lua/20-security) | `load`/`loadstring` on untrusted input and unsandboxed `os.execute` are the most common Lua vulnerabilities (notably in game modding and CDN edge scripts). |
| 21 | [Exercises & Projects (Capstone)](/lua/21-exercises-and-projects) | A mixed-review capstone spanning metatables, coroutines, closures, and error handling. |
