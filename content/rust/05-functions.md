# 05 — Functions

::question-wrapper{language="rust"}
### Q1. What happens when you try to compile this function?

::code-wrapper{language="rust"}
```rust
fn add(a, b) -> i32 {
    a + b
}
```
::

- [ ] It compiles, but `a` and `b` are inferred as generic `T: Add` parameters.
- [ ] It compiles only in debug builds; release builds require explicit types.
- [ ] It fails to compile — Rust requires every function parameter to have an explicit type annotation; type inference like this only applies to closures and `let` bindings, never to `fn` parameter lists.
- [ ] It compiles fine — Rust infers `a` and `b` as `i32` from the return type and usage, just like closures do.

<details>
<summary>Show Answer</summary>

**Answer:** C — It fails to compile — Rust requires every function parameter to have an explicit type annotation.

**Explanation:** **Idiom:** the parser itself rejects this before type-checking even begins — every `fn` parameter must be written as `name: Type`, full stop. This surprises people who've just seen closures like `|a, b| a + b` compile fine with types inferred from context: closures and `let` bindings support inference, but plain `fn` items never do, by design, since a function's signature is part of its public contract and shouldn't silently shift based on how it's called. Rules out D (no such inference exists for `fn`) and B (nothing in `fn` syntax implies generics without explicit `<T>` bounds); A's debug/release split is invented — this is a hard parse error present in every build profile.

</details>
::

::question-wrapper{language="rust"}
### Q2. This function is declared to return `i32`. What happens when you try to compile it?

::code-wrapper{language="rust"}
```rust
fn plus_one(x: i32) -> i32 {
    x + 1;
}
```
::

- [ ] It fails to compile with a type mismatch: expected `i32`, found `()` — the trailing semicolon turns `x + 1` into a statement, so the block's value becomes the unit type instead of the computed `i32`.
- [ ] It compiles and returns `x + 1` as expected.
- [ ] It compiles and always returns `0`.
- [ ] It compiles with a warning but panics at runtime when called.

<details>
<summary>Show Answer</summary>

**Answer:** A — It fails to compile with a type mismatch: expected `i32`, found `()`.

**Explanation:** **Debug:** this is the single most common first-week Rust trap. A function body's value is determined by its final expression *if and only if* that expression has no trailing semicolon; adding `;` after `x + 1` converts it into a statement, and statements always evaluate to `()`. The block's overall value is then `()`, which doesn't match the declared `-> i32`, so `rustc` rejects it with `mismatched types: expected i32, found ()`. It's not a runtime issue (rules out D — this is caught entirely at compile time) and there's no fallback to a default value (rules out C). The fix is either deleting the semicolon or writing `return x + 1;` explicitly.

</details>
::

::question-wrapper{language="rust"}
### Q3. Is this function valid, and what does it demonstrate about `return`?

::code-wrapper{language="rust"}
```rust
fn abs_diff(a: i32, b: i32) -> i32 {
    if a > b {
        return a - b;
    }
    b - a
}
```
::

- [ ] It fails to compile — every code path must end in an explicit `return` statement, not a bare tail expression.
- [ ] It fails to compile — mixing `return` and implicit returns in the same function is not allowed.
- [ ] It compiles, but the final `b - a` line is unreachable dead code that gets a compiler warning.
- [ ] It compiles fine — `return` is only required to exit a function before reaching its final expression; the last line `b - a`, with no semicolon, is just as valid an implicit return.

<details>
<summary>Show Answer</summary>

**Answer:** D — It compiles fine — `return` is only required to exit a function before its final expression.

**Explanation:** Rust supports two equally valid ways to produce a function's return value: an explicit `return expr;` for early exits (as in the `if` branch here), and an implicit tail expression with no semicolon for the "falls through to the end" case. Both can coexist freely in the same function — there's no rule requiring consistency between them (rules out B), and a bare tail expression is not merely tolerated but idiomatic (rules out A). The final line is very much reachable whenever `a <= b`, since the `return` only fires inside the `if` block (rules out C).

</details>
::

::question-wrapper{language="rust"}
### Q4. `square` is called in `main` before its definition appears in the file. What happens?

::code-wrapper{language="rust"}
```rust
fn main() {
    println!("{}", square(5));
}

fn square(x: i32) -> i32 {
    x * x
}
```
::

- [ ] Compile error — Rust requires functions to be declared or prototyped before use, like C.
- [ ] It compiles and runs fine, printing `25` — Rust resolves item names (functions, structs, and so on) within a scope regardless of their textual order, so forward references to later-defined functions are completely normal.
- [ ] Compile error — only `main` may be defined first; all helper functions must precede their first call site.
- [ ] It compiles but only if `square` is marked `pub`.

<details>
<summary>Show Answer</summary>

**Answer:** B — It compiles and runs fine, printing `25`.

**Explanation:** **Idiom:** unlike C, which needs a prototype (or the definition itself) to appear before first use, Rust's compiler does a full pass over a scope's items — collecting function signatures, struct definitions, and so on — before checking any bodies, so the textual order of top-level items is irrelevant to name resolution. This lets you put `main` at the top of a file with helpers defined below it, a very common and idiomatic layout. `pub` (option D) controls cross-module/cross-crate visibility, not same-file resolution order, so it's irrelevant here since both functions are in the same module (rules out D and A/C, which both invent a C-like ordering requirement that doesn't exist in Rust).

</details>
::

::question-wrapper{language="rust"}
### Q5. `log` has no `-> Type` in its signature. What happens when compiling `main`?

::code-wrapper{language="rust"}
```rust
fn log(msg: &str) {
    println!("{msg}");
}

fn main() {
    let x = log("hi") + 1;
}
```
::

- [ ] It compiles — `log` implicitly returns `i32` `0` when no return type is given, so `x` becomes `1`.
- [ ] It compiles — Rust automatically discards `log`'s return value and treats the expression as just `1`.
- [ ] It fails to compile — `log`'s implicit return type is the unit type `()`, and `()` does not implement the `Add` trait needed for `() + 1`.
- [ ] It fails to compile because `log` must have an explicit `-> ()` annotation to be callable at all.

<details>
<summary>Show Answer</summary>

**Answer:** C — It fails to compile — `log`'s implicit return type is `()`, which has no `Add` implementation.

**Explanation:** Omitting `-> Type` entirely is exactly equivalent to writing `-> ()` — a function with no arrow always implicitly returns the unit type (rules out D, since the arrow is optional shorthand, not mandatory). `()` is a real, if trivial, value — but it's a zero-sized marker type with no arithmetic trait implementations, so `log("hi") + 1` fails to type-check with an error to the effect of "cannot add `{integer}` to `()`." Nothing about calling a `()`-returning function discards it into a usable numeric default (rules out A and B) — the unit value has to be explicitly used or ignored as `()`, not silently coerced into a number.

</details>
::

::question-wrapper{language="rust"}
### Q6. What does this program print?

::code-wrapper{language="rust"}
```rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };
    println!("{y}");
}
```
::

- [ ] `4`, because the block's trailing expression `x + 1` (no semicolon) is the value the whole block evaluates to, and that value is assigned to `y`.
- [ ] Compile error — a `{ }` block cannot be assigned directly to a `let` binding.
- [ ] `3`, because `y` binds to the inner `x`.
- [ ] It prints nothing — blocks used as expressions always evaluate to `()`.

<details>
<summary>Show Answer</summary>

**Answer:** A — `4`, because the block's trailing expression is its value.

**Explanation:** A bare `{ ... }` block is itself an expression, governed by the exact same rule as function bodies (Q2, Q3): its value is its own trailing expression without a semicolon. Here that's `x + 1`, which evaluates to `4` inside the block, so `y` binds to `4`. The inner `let x = 3;` is scoped entirely to the block and never leaks into the outer scope (rules out C — there is no outer `x` to shadow or bind to). This is a genuine departure from C-family languages, where `{ }` is purely a scoping construct with no value of its own (rules out B and D, which both assume block-as-value doesn't work or defaults to unit — it only defaults to unit if the block's last line *does* end in a semicolon or has no trailing expression).

</details>
::

::question-wrapper{language="rust"}
### Q7. What does this program print?

::code-wrapper{language="rust"}
```rust
fn factorial(n: u64) -> u64 {
    if n == 0 {
        1
    } else {
        n * factorial(n - 1)
    }
}

fn main() {
    println!("{}", factorial(5));
}
```
::

- [ ] `100`
- [ ] `24`
- [ ] Compile error — Rust does not support recursive function calls without an explicit `#[recursive]` attribute.
- [ ] `120`

<details>
<summary>Show Answer</summary>

**Answer:** D — `120`.

**Explanation:** `factorial(5)` unwinds as `5 * factorial(4) = 5 * 4 * factorial(3) = ... = 5 * 4 * 3 * 2 * 1 * factorial(0)`, and the base case `n == 0` returns `1`, giving `5 * 4 * 3 * 2 * 1 = 120`. Rust supports ordinary recursion with no special opt-in attribute (rules out C — there's no such thing as `#[recursive]`; any function may call itself, limited only by available stack space for very deep recursion, not by any annotation). B and A are simple arithmetic slips (`24` is `4!`, and `100` doesn't correspond to any factorial in this chain).

</details>
::

::question-wrapper{language="rust"}
### Q8. This is an attempt to overload `parse` based on parameter type, as is common in C++ or Java. What happens when this module is compiled?

::code-wrapper{language="rust"}
```rust
fn parse(input: &str) -> i32 {
    input.len() as i32
}

fn parse(input: i32) -> i32 {
    input * 2
}
```
::

- [ ] It compiles, and Rust picks the correct overload based on the argument type at each call site.
- [ ] It fails to compile with a "duplicate definitions" error — Rust has no function overloading; two items with the same name cannot coexist in the same scope regardless of differing parameter types. Idiomatic alternatives are distinct function names, generics, or trait methods.
- [ ] It compiles, but only the second definition is kept; the first is silently shadowed.
- [ ] It compiles only if the two functions have different return types.

<details>
<summary>Show Answer</summary>

**Answer:** B — It fails to compile with a "duplicate definitions" error.

**Explanation:** **Idiom:** unlike C++ or Java, Rust resolves function calls purely by name within a scope, with no overload-resolution step based on argument types — so defining two items named `parse` in the same module is simply a name collision (`the name 'parse' is defined multiple times`), regardless of how their signatures differ. There's no silent-shadowing behavior for top-level items the way there is for `let` bindings (rules out C), and differing return types don't rescue it either (rules out D) — the conflict is purely on the name. The idiomatic fixes are distinct names (`parse_str` / `parse_int`), a generic function with a trait bound, or implementing a trait like `FromStr` per type.

</details>
::

::question-wrapper{language="rust"}
### Q9. What happens when this code is compiled?

::code-wrapper{language="rust"}
```rust
fn connect(host: &str, port: u16 = 8080) {
    println!("{host}:{port}");
}
```
::

- [ ] It compiles, and calling `connect("localhost")` uses `port = 8080` automatically.
- [ ] It compiles, but the default is only applied in debug builds.
- [ ] It compiles, and `= 8080` is treated as an assertion that must hold for any caller-supplied port.
- [ ] It fails to compile — Rust has no default-parameter syntax at all; every parameter must be supplied explicitly at every call site. The idiomatic workarounds are an `Option<u16>` parameter, multiple clearly-named functions (e.g. `connect` and `connect_with_port`), or the builder pattern.

<details>
<summary>Show Answer</summary>

**Answer:** D — It fails to compile — Rust has no default-parameter syntax at all.

**Explanation:** **Idiom:** `param: Type = value` in a function signature isn't valid Rust syntax at all — this is a hard parse error, not merely a missing feature caught later. Coming from Python or JavaScript, where default parameter values are routine, this is a genuine gap: Rust deliberately requires every call site to supply every parameter explicitly, so the workarounds are an `Option<T>` parameter with an explicit fallback inside the body, multiple distinctly-named functions, or a builder for many optional settings. None of A, B, or C describe real Rust behavior — there is no build-profile-dependent default (B) and no assertion semantics for `=` in a parameter list (C).

</details>
::

::question-wrapper{language="rust"}
### Q10. What happens when this is compiled?

::code-wrapper{language="rust"}
```rust
fn print_owned(s: String) {
    println!("{s}");
}

fn main() {
    let greeting = String::from("hello");
    print_owned(greeting);
    println!("{greeting}");
}
```
::

- [ ] It fails to compile — passing `greeting` by value into `print_owned` moves ownership of the `String` into the function; `greeting` is no longer valid in `main` afterward, so the second `println!` triggers a "use of moved value" error.
- [ ] It compiles and prints `hello` twice.
- [ ] It compiles, but `greeting` is empty (`""`) by the time the second `println!` runs.
- [ ] It fails to compile because `String` cannot be passed as a function argument at all.

<details>
<summary>Show Answer</summary>

**Answer:** A — It fails to compile — passing `greeting` by value moves it, and the second `println!` uses a moved value.

**Explanation:** `String` is a heap-allocated, non-`Copy` type, so passing it by value into `print_owned` transfers ownership into the function's parameter `s` — `greeting` in `main` is left uninitialized from the compiler's point of view, and any later use of it (the second `println!`) is rejected at compile time with `error[E0382]: use of moved value: 'greeting'`. This is a basic, unavoidable consequence of by-value parameter passing for owned types, not silent data loss (rules out C — the value isn't emptied at runtime, the code simply never runs because it doesn't compile) and `String` is of course a perfectly ordinary argument type otherwise (rules out D). Contrast this with a `Copy` type in the next question.

</details>
::

::question-wrapper{language="rust"}
### Q11. Unlike the `String` example, what happens here?

::code-wrapper{language="rust"}
```rust
fn print_value(n: i32) {
    println!("{n}");
}

fn main() {
    let count = 5;
    print_value(count);
    println!("{count}");
}
```
::

- [ ] It fails to compile with the same "use of moved value" error as passing a `String` by value.
- [ ] It compiles, but `count` becomes `0` in `main` after the call, since ownership still transfers even for `Copy` types.
- [ ] It compiles and prints `5` twice — `i32` implements the `Copy` trait, so passing it by value copies the bits into the function; the original `count` in `main` remains valid and usable afterward.
- [ ] It compiles only because `count` happens to be a small value; larger integers like `i128` would move instead.

<details>
<summary>Show Answer</summary>

**Answer:** C — It compiles and prints `5` twice, since `i32` is `Copy`.

**Explanation:** Simple, fixed-size scalar types like `i32`, `bool`, `char`, and `f64` implement the `Copy` trait, which changes what "passing by value" means: instead of moving ownership (Q10), the bits are duplicated, leaving both the original `count` and the function's local `n` as independently valid, usable values. This directly contrasts with the `String` case — the difference is entirely about the type implementing `Copy`, not about the specific value (rules out D — `i128` is just as `Copy` as `i32`; magnitude is irrelevant, only the type matters) and there's no ownership transfer to "reset" the original to a default (rules out B).

</details>
::

::question-wrapper{language="rust"}
### Q12. `bail` is declared to return `!` (the "never" type) and always panics. What does this let the compiler do with `classify`?

::code-wrapper{language="rust"}
```rust
fn bail(msg: &str) -> ! {
    panic!("{msg}");
}

fn classify(n: i32) -> i32 {
    if n >= 0 {
        n
    } else {
        bail("negative numbers are not supported")
    }
}
```
::

- [ ] It fails to compile — the two branches of the `if` return different types (`i32` and `!`), which normally must match exactly.
- [ ] It compiles — `!` is a special type that coerces to whatever type is expected in context (here `i32`), because a call to a function returning `!` never actually produces a value to be wrong about, so it can stand in for any branch type.
- [ ] It compiles, but only because `panic!` is a macro exempt from normal type-checking rules.
- [ ] It fails to compile unless `bail`'s return type is changed to match `i32` exactly.

<details>
<summary>Show Answer</summary>

**Answer:** B — It compiles — `!` coerces to whatever type is expected in context.

**Explanation:** **Idiom:** `!`, the "never" type, marks a function (or expression) that never returns control to its caller — it always panics, exits the process, or loops forever. Because a diverging call genuinely never produces a value, the compiler is free to let `!` unify with any type an `if`/`else` or `match` branch needs, here `i32`, since there's no risk of a mismatched value actually surfacing at runtime. This is a deliberate special case, not a general relaxation of the rule that `if`/`else` branches must agree in type (rules out A, which describes the *normal* rule that `!` is specifically exempt from) — and it has nothing to do with `panic!` being a macro (rules out C; the special-casing lives in the type system's treatment of `!` itself, and applies equally to any `-> !` function, not just ones using `panic!`).

</details>
::

::question-wrapper{language="rust"}
### Q13. What does this program print, and what does it demonstrate about plain `fn` items?

::code-wrapper{language="rust"}
```rust
fn square(x: i32) -> i32 {
    x * x
}

fn apply(f: fn(i32) -> i32, value: i32) -> i32 {
    f(value)
}

fn main() {
    println!("{}", apply(square, 4));
}
```
::

- [ ] Compile error — `square` is a function, not a value, and cannot be passed as an argument without wrapping it in a closure first.
- [ ] Compile error — `apply`'s parameter type `fn(i32) -> i32` is invalid syntax; function pointer types don't exist in Rust.
- [ ] `16` — but only because `apply` implicitly converts `square` into a `Box<dyn Fn(i32) -> i32>` behind the scenes.
- [ ] `16` — a plain `fn` item like `square` coerces to a function pointer of type `fn(i32) -> i32`, a distinct concrete type from closures, and can be passed around, stored in variables, and called like any other value.

<details>
<summary>Show Answer</summary>

**Answer:** D — `16` — `square` coerces to a function pointer and is called like any other value.

**Explanation:** A named `fn` item can be referenced by its bare name and coerces to the concrete function-pointer type `fn(ArgTypes) -> RetType` — `apply(square, 4)` computes `square(4) = 16`. This is a real, lightweight, zero-allocation value type distinct from the `Fn`/`FnMut`/`FnOnce` closure traits (covered separately later) — no boxing or dynamic dispatch is involved here (rules out C), and function pointer types are entirely ordinary, valid Rust syntax used throughout callback-style APIs (rules out B). Functions absolutely are first-class values in this sense — passing `square` directly, with no closure wrapper needed, is exactly the point (rules out A).

</details>
::

::question-wrapper{language="rust"}
### Q14. What does this program print?

::code-wrapper{language="rust"}
```rust
const fn double(x: usize) -> usize {
    x * 2
}

const LEN: usize = double(4);

fn main() {
    let buffer = [0u8; LEN];
    println!("{}", buffer.len());
}
```
::

- [ ] `8` — marking `double` as `const fn` makes it eligible for compile-time evaluation, so `double(4)` can be used to compute `LEN`, which in turn can be used as a fixed array length.
- [ ] Compile error — function calls are never allowed in a `const` initializer, only literal values.
- [ ] Compile error — `double` would need to be `unsafe fn` to be evaluated at compile time.
- [ ] `8`, but only at runtime — `const fn` is purely a documentation hint and has no effect on when the function actually executes.

<details>
<summary>Show Answer</summary>

**Answer:** A — `8` — `const fn` makes `double` usable in compile-time contexts like an array length.

**Explanation:** **Performance:** the `const` keyword on a function is what makes it eligible for evaluation during compilation, not just at runtime — `double(4)` can appear in a `const` initializer and inside `[0u8; LEN]`'s array-length position because `LEN` is itself fully resolved to `8` before codegen. An *ordinary* (non-`const`) `fn` genuinely cannot be called from a const context (that restriction is real — it just doesn't apply here, which is what rules out B, since it describes plain `fn`, not `const fn`). There's no `unsafe` requirement for compile-time evaluation (rules out C — `const fn` is fully safe), and `const fn` is far from cosmetic: it directly controls whether the function *can* run at compile time when the context demands it, though it may still be called normally at runtime too (rules out D).

</details>
::

::question-wrapper{language="rust"}
### Q15. Why does `largest` require the `T: PartialOrd` bound instead of being written as `fn largest<T>(list: &[T]) -> T`?

::code-wrapper{language="rust"}
```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];
    println!("{}", largest(&numbers));
}
```
::

- [ ] It doesn't actually need the bound — `PartialOrd` is inferred automatically for any `T` used with `>`.
- [ ] Without a bound, `T` is completely unconstrained, and the compiler has no guarantee that the `>` operator (which requires `PartialOrd`) is implemented for whatever concrete type `T` ends up being — the bound is what makes `item > largest` valid to even write.
- [ ] The bound is only cosmetic documentation; removing it would still compile since `>` works on all types by default.
- [ ] The bound is required only because the list contains integers; generic functions over `String` or custom structs never need it.

<details>
<summary>Show Answer</summary>

**Answer:** B — Without the bound, the compiler can't guarantee `>` is implemented for `T`.

**Explanation:** A bare `T` with no bounds could be instantiated with *any* type, including ones with no comparison operators defined at all — so the compiler must reject `item > largest` unless the signature itself promises that whatever `T` is, it supports `PartialOrd` (which is what `>` desugars to). This is true regardless of which concrete type `largest` is eventually called with, including integers (rules out D, which mistakenly treats the requirement as integer-specific) — the same signature is called here with `i32` but the bound is what makes it work generically for *any* `PartialOrd` type. Nothing about trait bounds is automatic or optional cosmetic decoration (rules out A and C) — omitting `PartialOrd` produces a real compile error the moment `>` is used on an unconstrained `T`.

</details>
::

::question-wrapper
### Q16. A function `create_user` conceptually wants an optional `nickname` that defaults to the user's real name when not provided. Rust has no default-argument syntax. What's the idiomatic way to model this?

- [ ] Overload `create_user` with two versions, one taking a nickname and one without, and rely on the compiler to disambiguate by call site.
- [ ] Accept `nickname: &str` and require every caller to pass an empty string `""` to mean "no nickname."
- [ ] Accept `nickname: Option<&str>` (or provide a companion function like `create_user_with_nickname`), and resolve the fallback to the real name explicitly inside the function body — making the "no nickname supplied" case an explicit, visible part of the signature and logic rather than hidden default-argument magic.
- [ ] Use a macro to generate the missing default value at compile time, since only macros can simulate default arguments in Rust.

<details>
<summary>Show Answer</summary>

**Answer:** C — Accept `Option<&str>` (or a companion function) and resolve the fallback explicitly.

**Explanation:** **Idiom:** since Rust has neither default parameters nor overloading (Q8, Q9), the idiomatic way to express "this value is optional, with a computed fallback" is `Option<T>` in the signature — the caller passes `None` or `Some(value)` explicitly, and the function body handles both cases visibly, typically with `.unwrap_or_else(...)`. Overloading (A) isn't available in Rust at all, as established in Q8. An empty-string sentinel (B) is a classic anti-pattern: it conflates "no value was given" with "a valid-looking but empty value," and nothing stops a caller from passing `""` by mistake with a completely different intent. A macro (D) is unnecessary machinery for a problem `Option<T>` already solves cleanly.

</details>
::

::question-wrapper{language="rust"}
### Q17. Both match arms here have to produce a value of the same type for the `match` to type-check. Why does this compile even though the `Err` arm's `panic!(...)` never produces an `i32`?

::code-wrapper{language="rust"}
```rust
fn parse_or_die(input: &str) -> i32 {
    match input.parse::<i32>() {
        Ok(n) => n,
        Err(_) => panic!("invalid number: {input}"),
    }
}
```
::

- [ ] It doesn't compile — `panic!` returns `String`, which mismatches the `Ok` arm's `i32`.
- [ ] `panic!` is special-cased by the match exhaustiveness checker to be silently skipped during type-checking.
- [ ] It only compiles because `parse_or_die`'s return type is inferred from the `Ok` arm alone, and the `Err` arm's type is never checked.
- [ ] `panic!(...)` has type `!` (never), and like any diverging expression, `!` coerces to match whatever type the other arms settle on — here `i32` — because a diverging arm never actually returns control to produce a mismatched value.

<details>
<summary>Show Answer</summary>

**Answer:** D — `panic!(...)` has type `!`, which coerces to match the other arms' type.

**Explanation:** **Idiom:** this is the same never-type coercion from Q12, now shown in its most common real-world form — using `panic!` (or `unreachable!`, `todo!`, `.expect(...)`'s internal panic) as one arm of a `match` whose other arms produce a real value. `panic!(...)` expands to code with type `!`, and `!` unifies with any expected type, so the whole `match` expression's type comes out as `i32` even though one arm can never actually produce an `i32`. It's not that the arm goes unchecked (rules out B and C — every arm genuinely is type-checked; `!` participates in that check, it isn't exempted from it) and `panic!` certainly doesn't return `String` — it never returns at all (rules out A).

</details>
::

::question-wrapper
### Q18. A `Server::new` constructor conceptually could take up to six independent optional settings (timeout, retries, TLS, max connections, and so on). Given Rust has neither default arguments nor overloading, what's considered the more idiomatic design as the number of optional settings grows?

- [ ] For a small number of optional settings, `Option<T>` parameters (or a couple of named constructor functions) are fine; as the count grows, the builder pattern (a separate `ServerBuilder` with chainable setter methods and a final `.build()`) scales better by avoiding an unwieldy parameter list while keeping each call site self-documenting.
- [ ] A single function with six `Option<T>` parameters is always preferred, no matter how many optional settings exist.
- [ ] Always use the builder pattern, even for a single optional parameter, since it's the only idiomatic Rust pattern for optional values.
- [ ] Define the struct's fields as `pub` and let callers construct it directly with struct-update syntax, since this is strictly superior to both `Option<T>` parameters and the builder pattern in every case.

<details>
<summary>Show Answer</summary>

**Answer:** A — Small counts favor `Option<T>` params or named functions; larger counts favor the builder pattern.

**Explanation:** **Idiom:** this is a judgment call that scales with complexity — a couple of `Option<T>` parameters (Q16) or a couple of clearly-named constructors stay readable, but a six-parameter function (positional or not) becomes error-prone and hard to read at the call site, which is exactly the problem the builder pattern solves by naming each setting via a chained method call. Absolutist answers are the tell for the wrong choice here: always using six `Option<T>` params (B) ignores how unwieldy that gets, always reaching for a builder even for one flag (C) is needless ceremony, and treating public struct-update construction as "strictly superior... in every case" (D) ignores that it bypasses validation and forces every field to be independently public, which a constructor or builder can avoid.

</details>
::

::question-wrapper{language="rust"}
### Q19. In JavaScript or Python, a nested function like `multiply` would close over `factor` from the enclosing scope automatically. What happens when this Rust code is compiled?

::code-wrapper{language="rust"}
```rust
fn main() {
    let factor = 10;

    fn multiply(x: i32) -> i32 {
        x * factor
    }

    println!("{}", multiply(5));
}
```
::

- [ ] It compiles and prints `50` — nested `fn` items behave exactly like closures and capture enclosing local variables automatically.
- [ ] It compiles, but `factor` is treated as `0` inside `multiply` since it wasn't explicitly passed in.
- [ ] It fails to compile — a plain `fn` item, even one nested inside another function, never implicitly captures variables from its enclosing scope; `factor` is simply not in scope inside `multiply`'s body, giving a "cannot find value `factor`" error. To capture `factor`, `multiply` would need to be rewritten as a closure (`|x| x * factor`) instead of a plain `fn`.
- [ ] It fails to compile because nested `fn` items cannot reference `i32` parameters at all.

<details>
<summary>Show Answer</summary>

**Answer:** C — It fails to compile — a plain `fn` never implicitly captures its enclosing scope.

**Explanation:** **Debug:** this is a frequent surprise for developers coming from JavaScript or Python, where a nested function definition is a closure by default. Rust draws a hard line between function *items* (`fn`) and closures (`|...| ...`): a `fn`, whether at module scope or nested inside another function, has no implicit environment — it can only see its own parameters, its own locals, and other items, never a local variable from an enclosing function body. `factor` genuinely isn't visible inside `multiply`, giving a compile-time "cannot find value" error, not a runtime `0` (rules out B, which imagines a silent fallback that doesn't exist) and definitely not automatic capture (rules out A). Nested `fn` items are completely legal and commonly used for local helpers (rules out D) — the restriction is specifically about capturing outer *variables*, not about referencing parameter types.

</details>
::

::question-wrapper
### Q20. Which statement correctly describes a key limitation of `const fn` compared to an ordinary `fn`?

- [ ] There is no difference at all — `const fn` is purely a marker with no effect on what the function body may contain.
- [ ] A `const fn` is restricted to operations the compiler can guarantee are evaluable at compile time (no arbitrary heap allocation, no calls to non-`const` functions, limited trait usage, and so on); it is a strict subset of what an ordinary `fn` may do, in exchange for being usable in const contexts like array lengths.
- [ ] A `const fn` can only be called at compile time; calling it at runtime with a non-constant argument is a compile error.
- [ ] A `const fn` automatically becomes `unsafe` because compile-time execution bypasses the borrow checker.

<details>
<summary>Show Answer</summary>

**Answer:** B — `const fn` is restricted to a subset of compile-time-evaluable operations, in exchange for const-context usability.

**Explanation:** **Performance:** `const fn` is a genuine trade-off, not a free-lunch marker (rules out A) — its body must stick to operations the compiler can prove are evaluable at compile time, which historically excludes things like arbitrary heap allocation or calling non-`const` functions (the exact boundary of what's permitted has grown across Rust editions, but it remains narrower than what an ordinary `fn` allows). In exchange, a `const fn` gains the ability to appear in const contexts such as array lengths (Q14) or `const`/`static` initializers. A common misconception is that this makes it compile-time-*only* (C) — in fact a `const fn` is dual-purpose: called with a `const`-eligible argument, it may run at compile time; called with an ordinary runtime value, it just runs as a normal function at runtime, no error involved. It also has nothing to do with `unsafe` or the borrow checker (rules out D) — const evaluation is fully safety-checked, just under a more restricted rule set.

</details>
::
