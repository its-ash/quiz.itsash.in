# Rust: Zero to Hero

A complete quiz-based curriculum covering Rust from first principles to expert-level, production-grade knowledge. Ownership and the borrow checker are the spine of the language — the curriculum is sequenced so every later quiz assumes you can reason about them.

## How to Use This Quiz Track

- Work through the parts in order; Parts 3 onward lean heavily on the ownership model from Part 2.
- Each quiz has 20 multiple-choice questions, ordered from basic recall to edge cases to best-practice judgment calls.
- Click **Show Answer** only after committing to a choice — many questions hinge on *why* the compiler accepts or rejects code, which only the explanation covers.
- Expect borrow-checker "gotchas," lifetime elision surprises, and `unsafe` footguns — these are where real Rust competency is built.

## Prerequisites

None for Part 1, though familiarity with a statically-typed language (C++, Java, Swift) makes the ownership model click faster.

## Curriculum

### Part 1 — Foundations

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 01 | [Introduction & Setup](/rust/01-introduction-and-setup) | `rustup`, toolchains, and editions set the stage for every later quiz. |
| 02 | [Hello World & Cargo](/rust/02-hello-world-and-cargo) | Cargo is Rust's build tool, package manager, and test runner — non-negotiable daily tooling. |
| 03 | [Variables & Mutability](/rust/03-variables-and-mutability) | Immutability by default and shadowing are core Rust idioms that surprise newcomers. |
| 04 | [Data Types](/rust/04-data-types) | Integer overflow behavior differs sharply between debug and release builds. |
| 05 | [Functions](/rust/05-functions) | Expression-vs-statement semantics (no `;` = return value) is a classic first-week trap. |
| 06 | [Control Flow](/rust/06-control-flow) | `if let`, `loop` with break-values, and exhaustive `match` are distinctly Rust-flavored. |

### Part 2 — The Ownership Model

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 07 | [Ownership](/rust/07-ownership) | The single idea that makes Rust memory-safe without a garbage collector. |
| 08 | [References & Borrowing](/rust/08-references-and-borrowing) | The borrow checker's aliasing rules are the most common source of "fighting the compiler." |
| 09 | [Slices](/rust/09-slices) | String slices and array slices avoid needless allocation but come with lifetime constraints. |
| 10 | [Lifetimes](/rust/10-lifetimes) | Lifetime elision rules and explicit annotations are the most-feared, most-misunderstood feature. |

### Part 3 — Data & Abstraction

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 11 | [Structs](/rust/11-structs) | Struct update syntax and method receivers (`self` vs `&self` vs `&mut self`) shape ergonomic APIs. |
| 12 | [Enums](/rust/12-enums) | `Option` and `Result` are enums — understanding variants unlocks idiomatic error handling. |
| 13 | [Pattern Matching](/rust/13-pattern-matching) | Exhaustiveness checks and guard clauses catch bugs at compile time other languages miss. |
| 14 | [Collections](/rust/14-collections) | `Vec`, `HashMap`, and entry APIs are the daily-driver data structures. |
| 15 | [Iterators](/rust/15-iterators) | Laziness, zero-cost abstraction claims, and adapter chaining are frequently misunderstood. |
| 16 | [Traits & Generics](/rust/16-traits-and-generics) | Trait bounds, `dyn` vs `impl Trait`, and monomorphization drive Rust's polymorphism model. |
| 17 | [Closures](/rust/17-closures) | `Fn`/`FnMut`/`FnOnce` and capture-by-move are common sources of compiler errors. |

### Part 4 — Robustness & Concurrency

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 18 | [Error Handling](/rust/18-error-handling) | `?` propagation, `panic!` vs `Result`, and error trait objects define idiomatic reliability. |
| 19 | [Smart Pointers](/rust/19-smart-pointers) | `Box`, `Rc`, `RefCell`, and interior mutability trade compile-time checks for runtime ones. |
| 20 | [Modules & Crates](/rust/20-modules-and-crates) | Visibility rules (`pub`, `pub(crate)`) and the module tree shape API design. |
| 21 | [Testing](/rust/21-testing) | Built-in `#[test]`, integration tests, and doctests are core to Rust's reliability culture. |
| 22 | [Concurrency](/rust/22-concurrency) | `Send`/`Sync` and ownership transfer make Rust's "fearless concurrency" claim concrete. |
| 23 | [Async/Await](/rust/23-async-await) | Futures are lazy and runtime-agnostic — a frequent source of "why didn't this run" bugs. |

### Part 5 — Metaprogramming & Systems

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 24 | [Macros](/rust/24-macros) | Declarative (`macro_rules!`) vs. procedural macros power much of the ecosystem's ergonomics. |
| 25 | [Unsafe Rust](/rust/25-unsafe-rust) | Raw pointers and manual invariants are where Rust's safety guarantees can be broken. |
| 26 | [FFI](/rust/26-ffi) | Calling into C and `#[repr(C)]` layout guarantees are essential for systems interop. |
| 27 | [Attributes & Cfg](/rust/27-attributes-and-cfg) | Conditional compilation and derive macros are pervasive in real crates. |
| 28 | [Cargo Features](/rust/28-cargo-features) | Feature flags and unification pitfalls affect every multi-crate workspace. |

### Part 6 — Expert & Professional

| # | Topic | Why It Matters |
|---|-------|-----------------|
| 29 | [Advanced Type System](/rust/29-advanced-type-system) | Associated types, GATs, and variance explain why some generic code won't compile. |
| 30 | [Design Patterns](/rust/30-design-patterns) | Builder, newtype, and typestate patterns are how Rust expresses OOP-style designs. |
| 31 | [Performance](/rust/31-performance) | Allocation patterns, `#[inline]`, and benchmarking pitfalls matter for systems-level code. |
| 32 | [Documentation](/rust/32-documentation) | Doc comments and doctests double as both documentation and executable tests. |
| 33 | [Ecosystem](/rust/33-ecosystem) | Knowing `serde`, `tokio`, and `clippy` conventions is table stakes for real Rust work. |
| 34 | [Common Pitfalls](/rust/34-common-pitfalls) | A curated set of the mistakes that trip up developers moving from GC'd languages. |
| 35 | [Exercises & Projects (Capstone)](/rust/35-exercises-and-projects) | A mixed-review capstone spanning ownership, traits, error handling, and concurrency. |
