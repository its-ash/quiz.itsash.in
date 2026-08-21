# 03 — Variables & Mutability

::question-wrapper{language="rust"}
### Q1. What happens when this code is compiled?

::code-wrapper{language="rust"}
```rust
fn main() {
    let x = 5;
    x = 6;
    println!("{x}");
}
```
::

- [ ] It compiles and prints `6`
- [ ] Compile error: cannot assign twice to immutable variable `x`
- [ ] It compiles and prints `5`, since the second assignment is silently ignored
- [ ] Runtime panic: attempted reassignment of immutable variable

<details>
<summary>Show Answer</summary>

**Answer:** B — Compile error: cannot assign twice to immutable variable `x`

**Explanation:** Variables in Rust are immutable by default; `let x = 5;` without `mut` forbids any further assignment, and the compiler catches this at compile time with `error[E0384]`, not at runtime (ruling out D, since immutability is a compile-time property, not a runtime check). Nothing is silently ignored (C) — the code simply fails to compile.

</details>
::

::question-wrapper
### Q2. What is required to make the reassignment in Q1 valid?

- [ ] Wrapping the value in `Cell::new(5)`
- [ ] Declaring the variable with `let mut x = 5;`
- [ ] Declaring the variable with `const mut x: i32 = 5;`
- [ ] Nothing — the original code already compiles

<details>
<summary>Show Answer</summary>

**Answer:** B — Declaring the variable with `let mut x = 5;`

**Explanation:** The `mut` keyword explicitly opts a binding into mutability, after which `x = 6;` is a legal reassignment. `const` (C) can never be combined with `mut` — constants are always immutable and this is a syntax error, not a valid alternative. `Cell::new` (A) provides interior mutability for shared references, which is a different, more advanced mechanism not needed for a simple owned local variable.

</details>
::

::question-wrapper{language="rust"}
### Q3. What is the difference between shadowing and mutation in this snippet?

::code-wrapper{language="rust"}
```rust
fn main() {
    let x = 5;
    let x = x + 1;
    let x = x * 2;
    println!("{x}");
}
```
::

- [ ] This is a compile error because `x` is redeclared without `mut`
- [ ] This is valid shadowing: each `let x = ...` creates a brand-new binding that temporarily hides the previous one; it prints `12`
- [ ] This mutates the original `x` in place, requiring `mut`, and prints `12`
- [ ] Only the first two `let x` lines are valid; the third causes a compile error for redefining twice

<details>
<summary>Show Answer</summary>

**Answer:** B — This is valid shadowing: each `let x = ...` creates a brand-new binding that temporarily hides the previous one; it prints `12`

**Explanation:** Shadowing with repeated `let` is explicitly allowed any number of times and does not require `mut`, because each `let` introduces a distinct variable (`(5+1)*2 = 12`) rather than mutating existing storage — unlike mutation, which changes a value in place through the same binding. This is a common point of confusion: shadowing *looks* like reassignment but is a completely different mechanism (C incorrectly claims `mut` is required).

</details>
::

::question-wrapper
### Q4. Which of the following is only possible with shadowing (`let`), not with plain mutation (`mut`)?

- [ ] Increasing a numeric value
- [ ] Changing the *type* of the variable binding, e.g. from `&str` to `usize`
- [ ] Using the variable inside a loop
- [ ] Passing the variable to a function

<details>
<summary>Show Answer</summary>

**Answer:** B — Changing the *type* of the variable binding, e.g. from `&str` to `usize`

**Explanation:** A `mut` binding keeps a fixed type for its entire lifetime — `let mut x = "5"; x = 5;` is a compile error (type mismatch: expected `&str`, found integer). Shadowing, however, creates an entirely new variable each time, so its type can freely differ: `let x = "5"; let x: usize = x.parse().unwrap();` is valid and idiomatic, commonly used to convert a value while reusing a meaningful name.

</details>
::

::question-wrapper{language="rust"}
### Q5. What is the correct syntax and semantics of a `const`?

::code-wrapper{language="rust"}
```rust
const MAX_RETRIES: u32 = 3;
```
::

- [ ] `const` values must have an explicit type annotation, are evaluated at compile time, and can never be mutated
- [ ] `const` values are inferred automatically like `let`, and type annotations are optional
- [ ] `const` behaves exactly like `let` but with a different keyword for style purposes
- [ ] `const` values are computed once at program startup (runtime), similar to a lazily initialized static

<details>
<summary>Show Answer</summary>

**Answer:** A — `const` values must have an explicit type annotation, are evaluated at compile time, and can never be mutated

**Explanation:** Unlike `let`, `const` requires an explicit type annotation and its initializer must be a compile-time constant expression — the compiler effectively inlines its value wherever it's used, and `mut` is never permitted on a `const`. Option D describes lazy runtime initialization, which is not how plain `const` works (that pattern would need something like `std::sync::LazyLock`/`once_cell` for genuinely runtime-computed constants).

</details>
::

::question-wrapper
### Q6. How does `static` differ from `const`?

- [ ] They are identical in every respect; `static` is just older syntax
- [ ] A `static` has a fixed memory address and lives for the entire program (`'static` lifetime), and can be declared `mut` (though accessing a mutable static requires `unsafe`); a `const` is inlined at each use site and has no fixed address
- [ ] `const` can be mutable with the `mut` keyword, but `static` cannot
- [ ] `static` values must be recomputed every time they're accessed, while `const` is cached

<details>
<summary>Show Answer</summary>

**Answer:** B — A `static` has a fixed memory address and lives for the entire program, and can be declared `mut` (requiring `unsafe` to access); a `const` is inlined at each use site and has no fixed address

**Explanation:** `const` is conceptually a compile-time substitution with no guaranteed single memory location (the compiler may duplicate its value at each usage), whereas `static` reserves one fixed, program-lifetime memory location — this matters when you need a stable address (e.g., for FFI) or genuine shared mutable global state, which is only possible (and only unsafely) via `static mut` (option C has the mutability rule backwards).

</details>
::

::question-wrapper{language="rust"}
### Q7. What does this program print?

::code-wrapper{language="rust"}
```rust
fn main() {
    let spaces = "   ";
    let spaces = spaces.len();
    println!("{spaces}");
}
```
::

- [ ] Compile error: mismatched types between `&str` and `usize`
- [ ] `3`
- [ ] It prints the string `"   "` itself
- [ ] Compile error: `spaces` was already defined

<details>
<summary>Show Answer</summary>

**Answer:** B — `3`

**Explanation:** This is the canonical shadowing example: the second `let spaces` shadows the first with a new binding of a completely different type (`usize`, holding the length of the original three-space string), which compiles fine specifically because shadowing (unlike mutation) permits type changes. Option A is the tempting mistake of assuming this is a `mut` reassignment, where a type mismatch really would occur.

</details>
::

::question-wrapper{language="rust"}
### Q8. What happens with this nested-scope shadowing?

::code-wrapper{language="rust"}
```rust
fn main() {
    let x = 1;
    {
        let x = x * 100;
        println!("inner: {x}");
    }
    println!("outer: {x}");
}
```
::

- [ ] Prints `inner: 100` then `outer: 100` — the inner shadow permanently changes `x`
- [ ] Prints `inner: 100` then `outer: 1` — the inner shadow only applies within its block scope; once the block ends, the outer `x` is unaffected
- [ ] Compile error: `x` used before being fully shadowed
- [ ] Prints `inner: 1` then `outer: 100`, since block scoping is evaluated bottom-up

<details>
<summary>Show Answer</summary>

**Answer:** B — Prints `inner: 100` then `outer: 1` — the inner shadow only applies within its block scope

**Explanation:** Shadowing is scoped like any other `let` binding: the inner `let x = x * 100;` creates a new variable visible only inside the `{ }` block; once that block ends, that shadow goes out of scope and the original outer `x` (still `1`) becomes visible again. This is a key edge case distinguishing shadowing from mutation — a `mut` variable's value change would persist outside a nested block that merely reassigns it (assuming no re-`let`), whereas a shadow's effect is strictly scoped.

</details>
::

::question-wrapper{language="rust"}
### Q9. What happens when an immutable variable holding a `Vec` is used like this?

::code-wrapper{language="rust"}
```rust
fn main() {
    let v = vec![1, 2, 3];
    v.push(4);
    println!("{v:?}");
}
```
::

- [ ] Compiles fine — `push` only needs `&self`, so mutability of the binding doesn't matter
- [ ] Compile error: cannot borrow `v` as mutable, as it is not declared as mutable
- [ ] Compiles but panics at runtime because `v` is immutable
- [ ] Compiles fine because `Vec` has interior mutability by default

<details>
<summary>Show Answer</summary>

**Answer:** B — Compile error: cannot borrow `v` as mutable, as it is not declared as mutable

**Explanation:** `Vec::push` takes `&mut self`, and obtaining a mutable borrow of `v` requires `v` itself to be declared `mut` — immutability is not just about direct reassignment (`v = ...`) but about disallowing any mutable access to the value's contents, including through method calls that need `&mut self`. `Vec` does not have interior mutability (D describes types like `RefCell`/`Cell`, not `Vec`), so the fix is simply `let mut v = vec![1, 2, 3];`.

</details>
::

::question-wrapper{language="rust"}
### Q10. What is the effect of shadowing inside a loop, as shown here?

::code-wrapper{language="rust"}
```rust
fn main() {
    let mut count = 0;
    for i in 0..3 {
        let count = count + i;
        println!("{count}");
    }
    println!("final: {count}");
}
```
::

- [ ] Prints `0`, `1`, `2` and `final: 0` — the shadowed `count` inside the loop body never affects the outer `mut count`, which is never actually mutated anywhere in this program
- [ ] Prints `0`, `1`, `3` and `final: 3`, since the shadow accumulates across iterations
- [ ] Compile error: `count` conflicts between `mut` and shadowed declarations
- [ ] Prints `0`, `1`, `2` and `final: 3`

<details>
<summary>Show Answer</summary>

**Answer:** A — Prints `0`, `1`, `2` and `final: 0` — the shadowed `count` inside the loop body never affects the outer `mut count`, which is never actually mutated anywhere in this program

**Explanation:** Each loop iteration re-declares a fresh shadow `let count = count + i`, reading the *outer* `count` (always `0`, since it's never reassigned with `count = ...` anywhere) plus the current `i`, then discarding that shadow at the end of the iteration's block scope. Despite being marked `mut`, the outer `count` is never actually mutated — only shadowed — which is a common gotcha: declaring `mut` doesn't mean a variable *is* being mutated, and shadowing inside a loop body does not accumulate like a running total would (contradicting the tempting B).

</details>
::

::question-wrapper{language="rust"}
### Q11. What is wrong, if anything, with this code?

::code-wrapper{language="rust"}
```rust
const BUFFER_SIZE: usize = compute_size();

fn compute_size() -> usize {
    42
}

fn main() {
    println!("{BUFFER_SIZE}");
}
```
::

- [ ] Nothing — this compiles and prints `42`, since `compute_size` is a simple const-evaluable function
- [ ] Compile error: `compute_size` is not marked `const fn`, so it cannot be called in a const-evaluation context
- [ ] Runtime panic because `const` initializers cannot call functions
- [ ] Compile error: `const` items cannot appear before `fn main`

<details>
<summary>Show Answer</summary>

**Answer:** B — Compile error: `compute_size` is not marked `const fn`, so it cannot be called in a const-evaluation context

**Explanation:** A `const` initializer must be evaluable entirely at compile time; calling an ordinary `fn` is not permitted because the compiler cannot generally guarantee it has no runtime-only behavior. The fix is `const fn compute_size() -> usize { 42 }`, which explicitly opts the function into compile-time (and still-usable-at-runtime) evaluation. Declaration order (D) is a red herring — Rust items are not order-dependent within a module, unlike `let` statements.

</details>
::

::question-wrapper{language="rust"}
### Q12. Given this snippet, what does `total` end up being, and why?

::code-wrapper{language="rust"}
```rust
fn main() {
    let total = 10;
    let total = total;
    let total = total + 5;
    println!("{total}");
}
```
::

- [ ] Compile error: cannot shadow a variable with itself (`let total = total;`)
- [ ] `15` — the second `let total = total;` is a valid (if redundant) shadow that simply moves/copies the value into a new binding, and shadowing continues to work normally afterward
- [ ] `10`, because the final shadow is ignored
- [ ] Compile error: `total` used in its own initializer is a circular definition

<details>
<summary>Show Answer</summary>

**Answer:** B — `15` — the second `let total = total;` is a valid (if redundant) shadow that simply moves/copies the value into a new binding

**Explanation:** Each `let total = <expr>` evaluates its right-hand side using whatever `total` was previously in scope *before* introducing the new binding — this is not circular (ruling out D) because the old and new `total` are different variables, not the same one referring to itself. This pattern (shadowing a variable with an expression referencing itself) is completely ordinary and compiles without any special-casing, ultimately yielding `10 + 5 = 15`.

</details>
::

::question-wrapper{language="rust"}
### Q13. What happens with this pattern-matched `let` binding?

::code-wrapper{language="rust"}
```rust
fn main() {
    let (a, mut b) = (1, 2);
    a = 10;
    b = 20;
    println!("{a} {b}");
}
```
::

- [ ] Both reassignments compile fine, since `mut` applies to the whole tuple pattern
- [ ] Compile error on `a = 10;` — mutability in a destructuring pattern is per-binding; `a` was not marked `mut`, so only `b` is reassignable
- [ ] Compile error on `b = 20;` — `mut` inside a tuple pattern is not valid syntax
- [ ] Compile error on both lines, since tuple destructuring never allows subsequent reassignment

<details>
<summary>Show Answer</summary>

**Answer:** B — Compile error on `a = 10;` — mutability in a destructuring pattern is per-binding; `a` was not marked `mut`, so only `b` is reassignable

**Explanation:** In pattern destructuring like `let (a, mut b) = ...`, `mut` attaches to the individual identifier it precedes, not to the tuple as a whole — a common gotcha for developers assuming `mut` on one part of a pattern applies globally. Since `a` lacks `mut`, `a = 10;` triggers `error[E0384]`, while `b = 20;` is perfectly valid.

</details>
::

::question-wrapper{language="rust"}
### Q14. What is the behavior of this code involving a shadowed reference?

::code-wrapper{language="rust"}
```rust
fn main() {
    let guess = "42";
    let guess: i32 = guess.trim().parse().expect("not a number");
    println!("{}", guess + 1);
}
```
::

- [ ] Compile error: `guess` cannot be both a `&str` and an `i32`
- [ ] Compiles and prints `43` — shadowing lets the string be parsed into a numeric type under the same name, a common idiom for input validation/conversion
- [ ] Compiles but panics at runtime because `"42"` is not trimmed first
- [ ] Compile error: `.expect()` cannot be called inside a `let` initializer

<details>
<summary>Show Answer</summary>

**Answer:** B — Compiles and prints `43` — shadowing lets the string be parsed into a numeric type under the same name, a common idiom for input validation/conversion

**Explanation:** This is the textbook idiomatic use of shadowing shown in the official Rust book: reusing the name `guess` avoids inventing a separate name like `guess_str`/`guess_num` for what is conceptually "the same value, converted." `.trim()` removes whitespace before `.parse::<i32>()`, which succeeds here since `"42"` parses cleanly, so `.expect()` never triggers its panic path, and the value is a genuine `i32` addition yielding `43`.

</details>
::

::question-wrapper
### Q15. When choosing between `mut` and shadowing to transform a value while keeping semantic continuity (e.g., trimming and parsing user input), what is the idiomatic best practice?

- [ ] Always prefer `mut` in every case, since shadowing is considered a deprecated pattern
- [ ] Prefer shadowing when the transformation also changes the type or represents a distinct logical value (e.g., raw string to parsed number), and reserve `mut` for genuine in-place accumulation/mutation of the same logical value (e.g., a running counter or a growing `Vec`)
- [ ] It never matters; both are functionally and stylistically interchangeable in all cases
- [ ] Always prefer shadowing over `mut`, even for loop counters and accumulators

<details>
<summary>Show Answer</summary>

**Answer:** B — Prefer shadowing when the transformation also changes the type or represents a distinct logical value, and reserve `mut` for genuine in-place accumulation/mutation

**Explanation:** **Idiom:** shadowing communicates "this is conceptually a new, immutable value derived from the old one" (great for type conversions or applying a series of independent transformations), while `mut` communicates "this variable's value genuinely changes over time" (ideal for counters, accumulators, or mutable collections). Defaulting to `mut` everywhere (A) loses the compiler's help in catching accidental unintended mutation, since immutable-by-default is one of Rust's core safety features, not a style preference to override universally.

</details>
::

::question-wrapper
### Q16. What is the idiomatic best-practice reason to prefer `const` over a "magic number" literal scattered throughout code, such as `const MAX_CONNECTIONS: u32 = 100;`?

- [ ] `const` values are faster at runtime than literals because they're cached in a register
- [ ] It gives the value a descriptive name (improving readability/intent), centralizes the value for easy updates, and lets the compiler enforce its type — literal magic numbers offer none of these and are error-prone to keep in sync across a codebase
- [ ] `const` is required by the compiler for any numeric literal used more than once
- [ ] There is no practical benefit; it's purely a stylistic preference with zero technical merit

<details>
<summary>Show Answer</summary>

**Answer:** B — It gives the value a descriptive name, centralizes the value for easy updates, and lets the compiler enforce its type — literal magic numbers offer none of these and are error-prone to keep in sync

**Explanation:** **Idiom:** since `const` values are inlined by the compiler, there's no meaningful runtime performance difference versus a literal (A is a myth) — the real benefit is entirely about maintainability and self-documenting code, e.g. changing `MAX_CONNECTIONS` in one place instead of hunting down every `100` in the codebase, some of which might mean something else entirely.

</details>
::

::question-wrapper
### Q17. Which naming convention does idiomatic Rust use for `const` and `static` items, as enforced by default Clippy/rustc style lints?

- [ ] camelCase, matching function and variable names
- [ ] SCREAMING_SNAKE_CASE
- [ ] PascalCase, matching type names
- [ ] snake_case, identical to regular variables

<details>
<summary>Show Answer</summary>

**Answer:** B — SCREAMING_SNAKE_CASE

**Explanation:** **Idiom:** Rust's naming conventions (enforced by the `non_upper_case_globals` lint) require `const`/`static` identifiers in `SCREAMING_SNAKE_CASE`, e.g. `MAX_RETRIES`, visually distinguishing compile-time constants from ordinary `snake_case` variables and `PascalCase` types — using `snake_case` for a `const` (D) will trigger a compiler warning by default, not silently pass unnoticed.

</details>
::

::question-wrapper{language="rust"}
### Q18. A function parameter needs to be modified locally within the function body without affecting the caller's original value (for `Copy` types). What is the idiomatic approach?

::code-wrapper{language="rust"}
```rust
fn double(mut n: i32) -> i32 {
    n *= 2;
    n
}
```
::

- [ ] This is invalid; parameters can never be declared `mut` in a function signature
- [ ] This is valid and idiomatic — `mut n` makes the local parameter binding mutable within the function body, and since `i32` is `Copy`, the caller's original value is entirely unaffected
- [ ] This mutates the caller's original variable as a side effect, since all integers are passed by reference
- [ ] `mut` here is redundant and should be removed, since function parameters are always mutable by default

<details>
<summary>Show Answer</summary>

**Answer:** B — This is valid and idiomatic — `mut n` makes the local parameter binding mutable within the function body, and since `i32` is `Copy`, the caller's original value is entirely unaffected

**Explanation:** Marking a by-value parameter `mut` is a common, idiomatic pattern when a function wants to use the parameter as a local scratch variable; because `i32` implements `Copy`, `n` is a completely independent copy of whatever the caller passed, so mutating it inside `double` has zero effect on the caller's variable (ruling out C, a mistaken assumption from languages that pass primitives by reference). Function parameters are immutable by default just like `let` bindings (contradicting D) — `mut` must be explicitly opted into here too.

</details>
::

::question-wrapper
### Q19. Best practice: a function receives configuration flags that should never change after being read at startup and are used throughout the entire program's lifetime across multiple modules. What is more idiomatic — a `static` or passing values explicitly through function parameters/structs?

- [ ] Always use `static mut` globals for any cross-module shared configuration, since it's the simplest to set up
- [ ] Prefer passing configuration explicitly (e.g., via a struct passed by reference or dependency injection) over global `static` state; reserve `static`/global state for cases with no reasonable alternative, since globals make code harder to test and reason about
- [ ] Use `const` for configuration that is only known at runtime (e.g., read from a file or environment variable)
- [ ] There is no idiomatic distinction; global mutable state is equally recommended in Rust as in any other language

<details>
<summary>Show Answer</summary>

**Answer:** B — Prefer passing configuration explicitly over global `static` state; reserve `static`/global state for cases with no reasonable alternative

**Explanation:** **Idiom:** explicit parameter/struct passing keeps dependencies visible and testable (you can construct different configs in different tests without global interference), whereas global mutable state (especially `static mut`, which requires `unsafe` and is a well-known source of data races and hard-to-trace bugs) is generally discouraged. `const` (C) cannot hold runtime-only values like file/env contents at all — its initializer must be evaluable at compile time, ruling that option out entirely, not just as bad practice.

</details>
::

::question-wrapper{language="rust"}
### Q20. A code reviewer flags this function for using `mut` where it isn't needed:

::code-wrapper{language="rust"}
```rust
fn describe(mut name: String) -> String {
    format!("Hello, {name}")
}
```
::

- [ ] The reviewer is right to flag it — `mut` is unused here since `name` is never reassigned or mutated in the body, and the compiler would emit an `unused_mut` warning; removing `mut` is the idiomatic fix
- [ ] The reviewer is wrong — `mut` is required whenever a `String` is taken by value
- [ ] The reviewer is wrong — removing `mut` would cause a compile error since `format!` internally mutates its arguments
- [ ] `mut` has no effect either way here and the compiler never warns about it

<details>
<summary>Show Answer</summary>

**Answer:** A — The reviewer is right to flag it; `mut` is unused, and removing it is the idiomatic fix

**Explanation:** **Idiom:** `mut` should only be applied when a binding is actually reassigned or mutated through a `&mut` borrow; here `name` is only read via `format!`, so `mut` is dead weight that the compiler flags with an `unused_mut` warning by default (contradicting D). Leaving unnecessary `mut` around is a minor but real code-smell — it misleads readers into thinking the parameter is modified, and idiomatic Rust code keeps mutability annotations minimal and accurate to signal actual intent.

</details>
::
