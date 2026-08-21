# 12 — Enums

::question-wrapper
### Q1. Why does Rust use `Option<T>` instead of allowing references and values to be `null`?

- [ ] `null` is reserved as a keyword for future use
- [ ] `Option<T>` forces the compiler to make you explicitly handle the "no value" case at compile time, eliminating null-pointer-style runtime errors
- [ ] `Option<T>` is faster than a nullable pointer in every case
- [ ] Rust does support `null`, but only for raw pointers used in `unsafe` code, and `Option<T>` is just a convenience wrapper on top

<details>
<summary>Show Answer</summary>

**Answer:** B — `Option<T>` forces the compiler to make you explicitly handle the "no value" case at compile time, eliminating null-pointer-style runtime errors

**Explanation:** By encoding absence as a distinct type (`Option<T>` = `Some(T)` or `None`) rather than a special sentinel value any reference could silently hold, the compiler forces every caller to handle both cases (via `match`, `if let`, `?`, `.unwrap()`, etc.) before extracting the inner `T`. This eliminates an entire class of null-dereference bugs at compile time. A is a distractor — `null` isn't a reserved keyword tied to this. C overstates a general performance claim; the point is safety, not universal speed superiority (though `Option<&T>` is often the same size as a raw pointer thanks to niche optimization, addressed later). D is false — safe Rust references (`&T`) genuinely cannot be null; there is no hidden null state for them outside of raw pointers in `unsafe` code, which are a fundamentally different type.

</details>
::

::question-wrapper
### Q2. What are the two variants of `Result<T, E>`, and what does each represent?

- [ ] `Ok(T)` for success, `Err(E)` for failure
- [ ] `Some(T)` for success, `None` for failure
- [ ] `Pass(T)` and `Fail(E)`
- [ ] `Valid(T)` and `Invalid(String)`

<details>
<summary>Show Answer</summary>

**Answer:** A — `Ok(T)` for success, `Err(E)` for failure

**Explanation:** `Result<T, E>` is the standard library's two-variant enum for fallible operations: `Ok(T)` wraps the success value, `Err(E)` wraps the error value, and the generic `E` lets each API choose its own error type. B is the tempting mix-up with `Option<T>`'s variant names — a very common naming confusion for newcomers. C and D are plausible-sounding but fabricated variant names; the standard library specifically uses `Ok`/`Err`.

</details>
::

::question-wrapper{language="rust"}
### Q3. What's the memory representation implication of an enum with data-carrying variants of different sizes, like this one?

::code-wrapper{language="rust"}
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```
::

- [ ] Each variant gets its own independent memory allocation
- [ ] The enum's total size is at least large enough to hold its largest variant, plus a discriminant tag to identify which variant is active
- [ ] The enum is always exactly the size of a pointer, regardless of variant contents
- [ ] `Quit` (no data) makes the whole enum zero-sized

<details>
<summary>Show Answer</summary>

**Answer:** B — The enum's total size is at least large enough to hold its largest variant, plus a discriminant tag to identify which variant is active

**Explanation:** Rust enums are tagged unions: the compiler reserves enough space for the largest variant (here, `Write(String)` or `ChangeColor(i32,i32,i32)`, whichever is bigger) plus a discriminant to track which variant is currently stored, and every instance uses that same fixed size no matter which variant it holds. A describes a completely different allocation strategy Rust doesn't use for stack-resident enums. C is a fabricated simplification. D is wrong — one zero-sized variant among several data-carrying variants doesn't shrink the enum; the size is driven by the largest variant, not the smallest.

</details>
::

::question-wrapper{language="rust"}
### Q4. Given `let msg = Message::Write(String::from("hi"));`, how do you correctly destructure it in a `match` without moving `msg`?

::code-wrapper{language="rust"}
```rust
enum Message { Write(String), Quit }
let msg = Message::Write(String::from("hi"));
```
::

- [ ] `match msg { Message::Write(s) => println!("{s}"), Message::Quit => {} }`
- [ ] `match &msg { Message::Write(s) => println!("{s}"), Message::Quit => {} }`
- [ ] `match msg { Write(s) => println!("{s}"), Quit => {} }`
- [ ] `if msg == Message::Write { ... }`

<details>
<summary>Show Answer</summary>

**Answer:** B — `match &msg { Message::Write(s) => println!("{s}"), Message::Quit => {} }`

**Explanation:** Matching on `&msg` (a reference) makes the pattern match against borrowed data — `s` is then bound as `&String` via match ergonomics, and `msg` itself is not moved, so it remains usable afterward. Option A matches on `msg` by value, which moves the `String` out of `msg` into `s`, making `msg` (or at least the moved-out field) unusable afterward — a real gotcha covered further in the edge-case questions. C omits the `Message::` path qualifier, which only works if the variants were brought into scope via `use Message::*`, not by default — as bare code it fails to compile with "cannot find value `Write`". D confuses enums with `match` for a boolean-style `==` comparison; `Message::Write` isn't comparable this way without deriving `PartialEq`, and even then this particular syntax is invalid since `Write` alone (without its data) isn't a complete pattern for `==`.

</details>
::

::question-wrapper
### Q5. What does calling `.unwrap()` on `Option<T>` do when the value is `None`?

- [ ] Returns the type's default value
- [ ] Panics immediately with a message like "called `Option::unwrap()` on a `None` value"
- [ ] Returns `None` unchanged
- [ ] Silently converts to `Result::Err`

<details>
<summary>Show Answer</summary>

**Answer:** B — Panics immediately with a message like "called `Option::unwrap()` on a `None` value"

**Explanation:** `.unwrap()` is explicitly the "I am certain this is `Some`/`Ok`, and if I'm wrong, crash loudly" escape hatch — on `None` (or `Err` for `Result::unwrap()`) it triggers a panic that unwinds (or aborts, depending on panic strategy) the current thread. A and C invent silent fallback behaviors that `.unwrap()` specifically does not provide (that's what `.unwrap_or_default()` and `.unwrap_or(default)` are for). D is a fabricated cross-type conversion that doesn't happen automatically.

</details>
::

::question-wrapper
### Q6. What's the correct way to define an enum variant that carries multiple named fields, similar to a struct?

- [ ] `enum Shape { Circle(f64), Rectangle { width: f64, height: f64 } }`
- [ ] Named fields aren't allowed in enum variants; only tuple-style data is supported
- [ ] `enum Shape { Circle: f64, Rectangle: (f64, f64) }`
- [ ] You must define a separate `struct Rectangle` and reference it by name only

<details>
<summary>Show Answer</summary>

**Answer:** A — `enum Shape { Circle(f64), Rectangle { width: f64, height: f64 } }`

**Explanation:** Rust enum variants can mix tuple-style data (`Circle(f64)`) and struct-style named fields (`Rectangle { width: f64, height: f64 }`) within the same enum — each variant chooses its own shape independently. B is false; struct-like variants are a core, commonly used feature. C uses invalid syntax — variant definitions don't use `:` this way. D unnecessarily forces an external struct definition when inline struct-like variants are supported directly.

</details>
::

::question-wrapper{language="rust"}
### Q7. What does `?` do when applied to a `Result<T, E>` inside a function that itself returns `Result<T, E>`?

::code-wrapper{language="rust"}
```rust
fn read_count(path: &str) -> Result<u32, std::num::ParseIntError> {
    let contents = std::fs::read_to_string(path).unwrap_or_default();
    let n: u32 = contents.trim().parse()?;
    Ok(n)
}
```
::

- [ ] On `Err`, it panics immediately
- [ ] On `Ok(v)`, it unwraps to `v`; on `Err(e)`, it returns early from the function with `Err(e)` (converted via `From` if needed)
- [ ] It logs the error and continues with a default value
- [ ] It only works inside `main()`

<details>
<summary>Show Answer</summary>

**Answer:** B — On `Ok(v)`, it unwraps to `v`; on `Err(e)`, it returns early from the function with `Err(e)` (converted via `From` if needed)

**Explanation:** The `?` operator is sugar for "unwrap on success, early-return on failure," and it also calls `From::from` on the error to convert it into the function's declared error type if they differ. A is wrong — `?` explicitly does *not* panic; that's what distinguishes it from `.unwrap()`. C invents logging/default behavior that `?` does not perform. D is a common misconception — `?` works in any function whose return type implements the necessary `Try`/`FromResidual` mechanics (any `Result`- or `Option`-returning function, not just `main`, though `main` can also return `Result` since Rust 2018).

</details>
::

::question-wrapper{language="rust"}
### Q8. What happens when you call `.unwrap()` on a `Result::Err` in production code with no `catch_unwind` or panic hook configured?

::code-wrapper{language="rust"}
```rust
fn parse_port(s: &str) -> u16 {
    s.parse().unwrap()
}
let port = parse_port("not_a_number");
```
::

- [ ] `port` becomes `0`
- [ ] The thread panics and, by default, unwinds — if this happens on the main thread, the process terminates with a non-zero exit code and a panic message printed to stderr
- [ ] The compiler rejects this code because `parse_port`'s return type doesn't mention `Result`
- [ ] `parse()` silently returns `65535` (max `u16`) for invalid input

<details>
<summary>Show Answer</summary>

**Answer:** B — The thread panics and, by default, unwinds — if this happens on the main thread, the process terminates with a non-zero exit code and a panic message printed to stderr

**Explanation:** `"not_a_number".parse::<u16>()` returns `Err`, and `.unwrap()` on an `Err` panics with a message like `called \`Result::unwrap()\` on an \`Err\` value: ParseIntError { ... }`. In a real service, this is exactly the kind of `.unwrap()` that takes down a request handler (or the whole process, if on the main thread and not caught) on malformed input. **Debug:** the correct production handling is to propagate the error with `?` (changing the return type to `Result<u16, ParseIntError>`) or handle it explicitly with `match`/`unwrap_or` and a sensible fallback or error response, rather than trusting external input to always parse cleanly. A, C, and D all invent silent-failure behaviors; Rust does not silently coerce a failed parse into a default numeric value.

</details>
::

::question-wrapper
### Q9. What is the size, in bytes, of `Option<&i32>` on a 64-bit platform, compared to `&i32` alone, and why?

- [ ] 16 bytes for `Option<&i32>` vs 8 bytes for `&i32` — the `Option` tag doubles the size
- [ ] 8 bytes for both — the compiler uses "niche optimization," repurposing the fact that a valid reference can never be the all-zero bit pattern to represent `None` as that otherwise-impossible value
- [ ] `Option<&i32>` cannot be constructed since references can't be optional
- [ ] 9 bytes for `Option<&i32>` (8 for the pointer + 1 tag byte, unpadded) vs 8 bytes for `&i32`

<details>
<summary>Show Answer</summary>

**Answer:** B — 8 bytes for both — the compiler uses "niche optimization," repurposing the fact that a valid reference can never be the all-zero bit pattern to represent `None` as that otherwise-impossible value

**Explanation:** **Performance:** this is a genuinely surprising and important Rust optimization — because a real `&T` reference is guaranteed non-null, `Option<&T>` doesn't need a separate discriminant byte at all; the compiler encodes `None` as the bit pattern that a real reference could never have (all zeros), making `Option<&T>` exactly pointer-sized. This is why `Option<&T>` (and `Option<Box<T>>`, `Option<NonZeroU32>`, etc.) is often used as a zero-overhead nullable pointer. A is the naive "tag always adds size" assumption that's true for many enums but not this niche-optimized case. C is false — `Option<&T>` is a completely standard, common pattern. D invents padding math that doesn't reflect how niche optimization actually works here.

</details>
::

::question-wrapper{language="rust"}
### Q10. What does an empty match on `Option<T>` that omits the `None` arm do?

::code-wrapper{language="rust"}
```rust
fn describe(x: Option<i32>) -> String {
    match x {
        Some(n) => n.to_string(),
    }
}
```
::

- [ ] Compiles, returning an empty string for `None`
- [ ] Fails to compile — "non-exhaustive patterns: `None` not covered"
- [ ] Compiles, but panics at runtime if `x` is `None`
- [ ] Compiles only in debug builds

<details>
<summary>Show Answer</summary>

**Answer:** B — Fails to compile — "non-exhaustive patterns: `None` not covered"

**Explanation:** Rust's `match` requires exhaustiveness — every possible variant of the matched type must be covered (or a wildcard `_` catch-all provided) — and the compiler statically checks this against the enum's known variant set at compile time, so a missing `None` arm on `Option<T>` is a hard compile error, not a runtime concern. A and C both imagine runtime fallback behavior that never gets the chance to run because compilation fails first. D is false — exhaustiveness checking is not conditional on build profile; it's a fundamental part of type checking.

</details>
::

::question-wrapper
### Q11. `enum Never {}` — an enum with zero variants — what can you do with a value of this type?

- [ ] Construct it with `Never::default()`
- [ ] Nothing — it's impossible to construct a value of this type, since there are no variants to build; it's used to statically prove a code path is unreachable
- [ ] It behaves exactly like a unit struct
- [ ] It's a compile error to define an enum with no variants

<details>
<summary>Show Answer</summary>

**Answer:** B — Nothing — it's impossible to construct a value of this type, since there are no variants to build; it's used to statically prove a code path is unreachable

**Explanation:** An enum with zero variants (like the standard library's `std::convert::Infallible`, which is essentially this) has no possible values at all — it's an "uninhabited type." This is genuinely useful: e.g., `Result<T, Infallible>` tells the compiler (and a reader) that the error branch can never actually occur, and `match` exhaustiveness checking can then treat that branch as impossible to reach. A is wrong — there's no variant to construct, so no `default()` could exist. C is wrong — a unit struct *can* be constructed (it has exactly one, trivial value); a zero-variant enum has *no* values at all. D is false — this is valid, if unusual, Rust and compiles fine.

</details>
::

::question-wrapper{language="rust"}
### Q12. What's the pitfall in this match guard combined with binding, when matching against a mutable reference?

::code-wrapper{language="rust"}
```rust
enum Status { Pending, Active(u32), Done }
let mut status = Status::Active(5);
match &mut status {
    Status::Active(n) if *n > 3 => *n += 1,
    _ => {}
}
```
::

- [ ] This fails to compile — you cannot mutate through a match binding
- [ ] `n` binds as `&mut u32` due to match ergonomics on `&mut status`, so `*n += 1` correctly mutates the field in place, incrementing it to `6`
- [ ] The guard `if *n > 3` always evaluates to `false` because `n` hasn't been dereferenced yet at that point
- [ ] `status` must first be cloned before matching

<details>
<summary>Show Answer</summary>

**Answer:** B — `n` binds as `&mut u32` due to match ergonomics on `&mut status`, so `*n += 1` correctly mutates the field in place, incrementing it to `6`

**Explanation:** Match ergonomics automatically adjusts binding modes when matching on a reference: matching `&mut status` against `Status::Active(n)` binds `n` as `&mut u32` rather than requiring you to write `Status::Active(ref mut n)` manually. The guard `if *n > 3` correctly dereferences to compare the underlying value, and `*n += 1` mutates it through the reference — perfectly valid and idiomatic. A is wrong — mutation through a match binding is exactly what `&mut` matching enables. C misunderstands dereferencing — `*n` in the guard reads the current value just fine, it doesn't require a prior explicit step. D is unnecessary — no clone is needed since matching on `&mut status` borrows rather than moves.

</details>
::

::question-wrapper
### Q13. In `enum Shape { Circle(f64), Square(f64) }`, is `Circle(2.0) == Circle(2.0)` valid, and under what condition?

- [ ] Always valid — all enums support `==` by default
- [ ] Only valid if `Shape` derives (or manually implements) `PartialEq`; without it, `==` on enum values is a compile error
- [ ] Only valid for enums with no data-carrying variants
- [ ] Always invalid — enums can only be compared with `match`

<details>
<summary>Show Answer</summary>

**Answer:** B — Only valid if `Shape` derives (or manually implements) `PartialEq`; without it, `==` on enum values is a compile error

**Explanation:** Rust does not give any type — struct or enum — automatic equality comparison; `#[derive(PartialEq)]` (or a manual `impl`) is required before `==`/`!=` can be used, and the compiler will reject `Circle(2.0) == Circle(2.0)` with "binary operation `==` cannot be applied" if the derive is missing. A is the common false assumption carried over from languages with default structural equality. C is fabricated — data-carrying variants can absolutely be compared once `PartialEq` is derived (it compares variant tag and then inner data). D overstates things — `match`/`matches!` are alternatives for checking variant identity without full data equality, but `==` works fine too once derived.

</details>
::

::question-wrapper{language="rust"}
### Q14. What happens if you match on `&Option<String>` and try to move the inner `String` out inside the match arm?

::code-wrapper{language="rust"}
```rust
let maybe_name: Option<String> = Some(String::from("Ferris"));
match &maybe_name {
    Some(s) => {
        let owned: String = *s;
        println!("{owned}");
    }
    None => {}
}
```
::

- [ ] Compiles fine — `*s` copies the `String`
- [ ] Fails to compile — `s` is `&String` here, and `String` isn't `Copy`, so `*s` would attempt an illegal move out of a reference; use `s.clone()` instead
- [ ] `maybe_name` becomes `None` after this match
- [ ] This only fails in `unsafe` blocks

<details>
<summary>Show Answer</summary>

**Answer:** B — Fails to compile — `s` is `&String` here, and `String` isn't `Copy`, so `*s` would attempt an illegal move out of a reference; use `s.clone()` instead

**Explanation:** Because the match scrutinee is `&maybe_name`, match ergonomics binds `s` as `&String`, not `String`. Dereferencing with `*s` to produce an owned `String` would require moving data out from behind a reference, which the borrow checker forbids ("cannot move out of `*s` which is behind a shared reference") since the original `maybe_name` still owns that data. The fix is `s.clone()` to get an owned copy, or restructure to match on `maybe_name` by value if you intend to consume it. A is wrong — `String` does not implement `Copy` (it manages heap data), so `*s` cannot silently copy. C is fabricated. D is false — this is a plain compile-time borrow-checker rejection, unrelated to `unsafe`.

</details>
::

::question-wrapper
### Q15. When designing a function that can fail, why is returning `Result<T, MyError>` generally preferred over returning `Option<T>` and discarding the reason for failure?

- [ ] `Option<T>` cannot be used with the `?` operator at all
- [ ] `Result<T, E>` preserves *why* the operation failed, which callers (and logs/error messages) often need, whereas `Option<T>` only signals *that* it failed
- [ ] `Option<T>` is deprecated in modern Rust
- [ ] `Result<T, E>` is always smaller in memory than `Option<T>`

<details>
<summary>Show Answer</summary>

**Answer:** B — `Result<T, E>` preserves *why* the operation failed, which callers (and logs/error messages) often need, whereas `Option<T>` only signals *that* it failed

**Explanation:** **Idiom:** the general guideline is: use `Option<T>` when absence is a normal, expected, reason-less outcome (e.g., `HashMap::get` — the key simply wasn't there), and use `Result<T, E>` when failure needs an explanation a caller might inspect, log, or match on (e.g., parse errors, I/O errors, validation failures). Collapsing a rich error into `Option<T>` throws away diagnostic information callers may need later. A is false — `?` works on `Option<T>` too, propagating `None` in a function that itself returns `Option`. C is false — `Option<T>` is a completely standard, actively used type, not deprecated. D is not a real or relevant comparison; sizes depend entirely on `T` and `E`.

</details>
::

::question-wrapper
### Q16. Code review flags a public function returning `Result<User, String>` where `String` is just a human-readable error message. What's the idiomatic improvement, and why?

- [ ] Leave it as-is — `String` errors are always sufficient
- [ ] Define a proper error enum implementing `std::error::Error` (and typically `Display`) so callers can programmatically distinguish failure kinds (e.g., `UserError::NotFound` vs `UserError::InvalidEmail`) instead of parsing message text
- [ ] Switch to `Result<User, ()>` since the message isn't needed
- [ ] Change it to `Option<User>` since `String` errors aren't real errors

<details>
<summary>Show Answer</summary>

**Answer:** B — Define a proper error enum implementing `std::error::Error` (and typically `Display`) so callers can programmatically distinguish failure kinds (e.g., `UserError::NotFound` vs `UserError::InvalidEmail`) instead of parsing message text

**Explanation:** **Idiom:** `String`-typed errors are a common quick-and-dirty starting point, but they force any caller that needs to branch on failure kind to parse or `.contains()` a message string, which is brittle and breaks silently if the wording changes. A structured error enum lets callers `match` on variants directly and lets the type system enforce handling all known failure kinds. A dismisses a real, well-known anti-pattern. C throws away the diagnostic message entirely, making debugging harder. D conflates "has a reason" (an error) with "has no reason" (`Option`'s use case) — the message shows this function does have failure reasons, so `Option` is the wrong direction entirely.

</details>
::

::question-wrapper
### Q17. Why is `if let Some(x) = maybe_value { ... }` often preferred over a full `match` when you only care about one variant and want to ignore the rest?

- [ ] `match` cannot ignore variants; `if let` is required whenever you don't want every arm
- [ ] `if let` is more concise for the single-pattern case, at the cost of losing the compiler's exhaustiveness guarantee for any variants you're not handling
- [ ] `if let` is faster at runtime than `match`
- [ ] `if let` and `match` are interchangeable with no tradeoffs whatsoever

<details>
<summary>Show Answer</summary>

**Answer:** B — `if let` is more concise for the single-pattern case, at the cost of losing the compiler's exhaustiveness guarantee for any variants you're not handling

**Explanation:** **Idiom:** `if let Some(x) = maybe_value { ... }` desugars to a `match` with one meaningful arm and an implicit `_ => {}` — great ergonomics when you truly only care about one case, but it means if a new enum variant is added later, the compiler won't force you to reconsider this call site the way an exhaustive `match` would. This is a genuine tradeoff to weigh, not a strict upgrade. A is false — `match _ => {}` handles the "ignore the rest" case just fine within a full `match`. C is a fabricated performance claim; they compile to equivalent code. D dismisses the real exhaustiveness tradeoff just described.

</details>
::

::question-wrapper
### Q18. What's the best-practice reason to prefer `let Some(x) = opt else { return; };` ("let-else") over a `match` with an unreachable/panicking else-arm, when you need `x` bound in the *surrounding* scope rather than nested inside a block?

- [ ] `let else` is the only way to bind a variable from a pattern match at all
- [ ] `let else` keeps `x` usable in the rest of the enclosing function body without an extra nested block or indentation level, while still forcing the "else" (failure) path to diverge (return, break, panic, etc.)
- [ ] `match` cannot diverge in its arms
- [ ] `let else` automatically logs a warning when the else branch runs

<details>
<summary>Show Answer</summary>

**Answer:** B — `let else` keeps `x` usable in the rest of the enclosing function body without an extra nested block or indentation level, while still forcing the "else" (failure) path to diverge (return, break, panic, etc.)

**Explanation:** **Idiom:** with a plain `match`, binding `x` for use *after* the match block requires either nesting the rest of the logic inside the `Some` arm (extra indentation) or declaring `x` outside and assigning inside (awkward with non-`Copy`/non-default types). `let else` solves this cleanly: the happy-path binding flows into the normal scope, and the compiler enforces that the `else` block must diverge (never fall through), so `x` is always known to be initialized past that point. A is false — `if let` and `match` can both bind variables too. C is false — `match` arms can absolutely diverge with `return`/`panic!`/etc.; that's exactly what makes the older equivalent pattern work. D is fabricated; no automatic logging happens.

</details>
::

::question-wrapper
### Q19. A team is deciding between representing HTTP methods as `enum Method { Get, Post, Put, Delete }` versus as `String` values like `"GET"`, `"POST"`. What's the best-practice argument for the enum?

- [ ] `String` comparisons are always slower, so the enum is purely a performance choice
- [ ] The enum makes invalid states like `Method::from("PATCH_TYPO")` unrepresentable — the compiler enforces that only the known, valid variants exist, and `match` exhaustiveness ensures every call site handles all of them (or is forced to update when a new one is added)
- [ ] `String` cannot be used in a `match` statement at all
- [ ] Enums are required for anything sent over a network

<details>
<summary>Show Answer</summary>

**Answer:** B — The enum makes invalid states like `Method::from("PATCH_TYPO")` unrepresentable — the compiler enforces that only the known, valid variants exist, and `match` exhaustiveness ensures every call site handles all of them (or is forced to update when a new one is added)

**Explanation:** **Idiom:** this is the "make illegal states unrepresentable" principle that enums are especially good at — a `String` can hold literally any text, pushing validation to runtime (and every call site must remember to validate), while `Method::Get` is guaranteed by the type system to be one of the defined variants, with typos caught at compile time. A is a plausible-sounding but secondary justification; the primary win here is correctness/safety, not raw comparison speed (though it can help too). C is false — `match` does work on `String`/`&str` via literal patterns, it's just less safe since it can't be exhaustive over "all possible strings" in a meaningful way. D is a fabricated blanket rule; plenty of network-serialized data uses strings, though enums are still often preferred internally with a serialization layer at the boundary.

</details>
::

::question-wrapper
### Q20. A struct field is typed `Option<Vec<String>>` to represent "either no tags, or a list of tags." A reviewer suggests changing it to just `Vec<String>` (using an empty vec for "no tags"). What's the tradeoff to consider?

- [ ] There's no difference — both represent "no tags" identically in every context
- [ ] `Option<Vec<String>>` distinguishes "explicitly no tags were provided/loaded" from "tags haven't been fetched/set yet," a distinction `Vec::new()` alone can't express, but at the cost of an extra `Some`/`None` unwrap at every use site
- [ ] `Vec<String>` cannot ever be empty, so this comparison is invalid
- [ ] `Option<Vec<String>>` is always more memory-efficient than `Vec<String>`

<details>
<summary>Show Answer</summary>

**Answer:** B — `Option<Vec<String>>` distinguishes "explicitly no tags were provided/loaded" from "tags haven't been fetched/set yet," a distinction `Vec::new()` alone can't express, but at the cost of an extra `Some`/`None` unwrap at every use site

**Explanation:** **Idiom:** this is a genuine, common judgment call — if "not yet loaded" and "loaded but empty" are meaningfully different states in your domain (e.g., lazily-fetched data, optional JSON fields distinguishing `null` from `[]`), `Option<Vec<T>>` captures that distinction the type system can enforce. If they're not meaningfully different, plain `Vec<T>` (empty vec = no tags) is simpler and avoids the extra layer of unwrapping/matching at every call site — an empty `Vec` is already a perfectly valid, well-supported "nothing here" state, so wrapping it in `Option` for no reason is a common overcomplication. A ignores a real semantic distinction that matters in some domains. C is false — an empty `Vec` is completely valid and common. D is a fabricated blanket memory claim that depends on context (e.g., `Option<Vec<T>>` adds a discriminant unless niche-optimized, and `Vec`'s pointer typically isn't null-representable the same simple way `&T` is, so this generalization doesn't reliably hold).

</details>
::
