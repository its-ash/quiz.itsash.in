# Python: Zero to Hero

A complete quiz-based curriculum covering Python from first principles to expert-level, production-grade knowledge. Python's simplicity hides a lot of nuance — this track is built around the surprises that trip up even experienced developers.

## How to Use This Quiz Track

- Work through the parts in order; later quizzes assume comfort with core data structures and OOP from earlier parts.
- Each quiz has 20 multiple-choice questions, ordered from basic recall to edge cases to best-practice judgment calls.
- Click **Show Answer** only after committing to a choice — the explanations cover the underlying mechanism (mutability, the GIL, MRO, descriptor protocol), not just the right letter.
- Expect classic "wat" moments — mutable default arguments, late-binding closures, and `is` vs `==` — turned into quiz questions.

## Prerequisites

None for Part 1. Comfort with basic programming concepts (variables, loops) helps but isn't required.

## Curriculum

### Part 1 — Foundations

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 01 | [Introduction & Setup](/python/01-introduction-and-setup) | CPython vs. other implementations and version differences (2 vs 3) shape everything downstream. |
| 02 | [Variables & Data Types](/python/02-variables-and-data-types) | Dynamic typing, object identity, and small-int caching produce classic `is`-vs-`==` gotchas. |
| 03 | [Operators & Expressions](/python/03-operators-and-expressions) | Chained comparisons, `//` vs `/`, and operator overloading via dunder methods are frequent traps. |
| 04 | [Control Flow](/python/04-control-flow) | `for...else`, walrus operator, and truthy/falsy rules for custom objects surprise newcomers. |
| 05 | [Functions](/python/05-functions) | Mutable default arguments are Python's most famous footgun. |
| 06 | [Strings & Text](/python/06-strings-and-text) | Immutability, encoding pitfalls, and f-string evaluation order matter in real code. |

### Part 2 — Core Data Structures

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 07 | [Lists & Tuples](/python/07-lists-and-tuples) | Shallow copies, list aliasing, and tuple immutability-of-reference-not-contents are daily bugs. |
| 08 | [Dictionaries & Sets](/python/08-dictionaries-and-sets) | Hashability rules and insertion-order guarantees (3.7+) affect correctness and performance. |
| 09 | [Comprehensions & Generators](/python/09-comprehensions-and-generators) | Lazy evaluation and comprehension scoping are common sources of subtle bugs. |
| 10 | [Functional Programming](/python/10-functional-programming) | `map`/`filter`/`reduce`, `lambda` limitations, and `functools` are idiomatic in data-heavy code. |
| 11 | [Closures & Decorators](/python/11-closures-and-decorators) | Late-binding closures in loops and decorator argument order are classic interview gotchas. |

### Part 3 — OOP & Design

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 12 | [Classes & Objects](/python/12-classes-and-objects) | Class vs. instance attributes and mutable class-level defaults cause shared-state bugs. |
| 13 | [Inheritance & Polymorphism](/python/13-inheritance-and-polymorphism) | Method Resolution Order (MRO) and `super()` in multiple inheritance are frequently misunderstood. |
| 14 | [Magic Methods & Protocols](/python/14-magic-methods-and-protocols) | `__eq__`/`__hash__` consistency and operator dunders define how objects behave idiomatically. |
| 15 | [Properties & Descriptors](/python/15-properties-and-descriptors) | The descriptor protocol underlies `@property`, ORMs, and much of Python's "magic." |

### Part 4 — Robustness & I/O

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 16 | [Error Handling](/python/16-error-handling) | Bare `except`, exception chaining, and `finally`-with-`return` interactions cause silent bugs. |
| 17 | [Modules & Packages](/python/17-modules-and-packages) | Circular imports and `__init__.py` semantics are frequent real-world stumbling blocks. |
| 18 | [File I/O & Serialization](/python/18-file-io-and-serialization) | Context managers, encoding defaults, and `pickle` security risks matter in production. |
| 19 | [Iterators & Context Managers](/python/19-iterators-and-context-managers) | The iterator protocol and `__enter__`/`__exit__` exception suppression are easy to misuse. |
| 20 | [Type Hints & Typing](/python/20-type-hints-and-typing) | `Optional`, generics, and runtime-vs-static-only enforcement are widely misunderstood. |

### Part 5 — Concurrency & Meta

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 21 | [Concurrency: Threading & Multiprocessing](/python/21-concurrency-threading-and-multiprocessing) | The GIL means threads don't parallelize CPU-bound work — a top source of design mistakes. |
| 22 | [Async/Await](/python/22-async-await) | Blocking calls inside `async def` silently kill concurrency without raising an error. |
| 23 | [Metaprogramming](/python/23-metaprogramming) | Metaclasses, `__init_subclass__`, and dynamic attribute access power frameworks like Django. |

### Part 6 — Professional & Expert

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 24 | [Testing](/python/24-testing) | Fixture scope, mocking pitfalls, and parametrization determine test suite trustworthiness. |
| 25 | [Packaging & Virtual Environments](/python/25-packaging-and-virtual-environments) | Dependency resolution and `pyproject.toml` conventions are essential for shipping real projects. |
| 26 | [Performance & Optimization](/python/26-performance-and-optimization) | Profiling before optimizing, and knowing where CPython is slow, prevents wasted effort. |
| 27 | [Security](/python/27-security) | `eval`/`pickle` risks and injection vectors are the most common Python vulnerabilities. |
| 28 | [Exercises & Projects (Capstone)](/python/28-exercises-and-projects) | A mixed-review capstone spanning generators, decorators, OOP, and concurrency. |
