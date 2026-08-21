# 04 — Data Types

::question-wrapper{language="rust"}
### Q1. What is the default integer type Rust infers when no suffix or context specifies otherwise?

::code-wrapper{language="rust"}
```rust
let x = 42;
```
::

- [ ] `i64`
- [ ] `i32`
- [ ] `u32`
- [ ] `isize`

<details>
<summary>Show Answer</summary>

**Answer:** B — `i32`

**Explanation:** When an integer literal's type can't be inferred from context (e.g. no explicit annotation, no usage that constrains it), Rust defaults to `i32`, generally the fastest integer type on modern platforms even on 64-bit systems. `isize` (D) is instead the pointer-sized type used for indexing, not the default for bare literals.

</details>
::

::question-wrapper
### Q2. What are `isize` and `usize` primarily used for?

- [ ] They are arbitrary-precision integer types with no fixed size
- [ ] Their size is platform-dependent (matching pointer width — e.g. 8 bytes on 64-bit systems), and they're primarily used for indexing collections and representing memory sizes/offsets
- [ ] They are always exactly 32 bits regardless of platform, used for network protocol fields
- [ ] They are floating-point types optimized for scientific computation

<details>
<summary>Show Answer</summary>

**Answer:** B — Their size is platform-dependent, matching pointer width, and they're primarily used for indexing collections and memory sizes

**Explanation:** `usize`/`isize` are defined to be exactly as wide as a pointer on the target platform (4 bytes on 32-bit, 8 bytes on 64-bit), which is why `Vec::len()` and slice indexing return/accept `usize` — indices can never exceed addressable memory. **Portability:** code that hardcodes assumptions like "`usize` is always 8 bytes" (as option D of a fixed 32-bit width would imply) can break when cross-compiling to 32-bit or embedded targets.

</details>
::

::question-wrapper{language="rust"}
### Q3. What does this program print when run with `cargo run` (debug profile, the default)?

::code-wrapper{language="rust"}
```rust
fn main() {
    let x: u8 = 255;
    let y = x + 1;
    println!("{y}");
}
```
::

- [ ] `0`, because `u8` silently wraps around on overflow
- [ ] `256`, because Rust automatically promotes to a wider type on overflow
- [ ] It panics at runtime with an "attempt to add with overflow" message, because debug builds include overflow checks
- [ ] Compile error, because the compiler statically proves this will overflow

<details>
<summary>Show Answer</summary>

**Answer:** C — It panics at runtime with an "attempt to add with overflow" message, because debug builds include overflow checks

**Explanation:** **Debug:** in debug/dev builds, arithmetic overflow triggers a runtime panic by default (a deliberate safety net during development); this is *not* a compile-time error (ruling out D — the compiler doesn't generally prove overflow for non-constant expressions) and there is no automatic widening (ruling out B — Rust never implicitly changes an integer's type). To get the wraparound behavior some beginners expect, you'd need an explicit method like `x.wrapping_add(1)`.

</details>
::

::question-wrapper
### Q4. What is the correct way to describe integer overflow behavior in `--release` builds by default?

- [ ] Release builds panic on overflow just like debug builds, for consistency
- [ ] Release builds disable overflow checks by default and perform two's-complement wraparound silently (e.g., `255u8 + 1` becomes `0`) — a deliberate trade-off for performance
- [ ] Release builds return `None` from `+` automatically, requiring `.unwrap()`
- [ ] Overflow is undefined behavior in release builds, same as in C

<details>
<summary>Show Answer</summary>

**Answer:** B — Release builds disable overflow checks by default and perform two's-complement wraparound silently — a deliberate trade-off for performance

**Explanation:** **Performance:** the `overflow-checks` profile setting defaults to `true` in `dev` and `false` in `release`, so the exact same overflowing code that panics in debug (Q3) silently wraps to `0` in release — a classic and dangerous debug/release behavior divergence. Unlike C's undefined behavior for signed overflow (D is false for Rust — Rust always defines wraparound as two's-complement, never UB, unlike C/C++), so this is technically safe but can still produce silently wrong application results if unnoticed.

</details>
::

::question-wrapper{language="rust"}
### Q5. What does casting with `as` do in this example?

::code-wrapper{language="rust"}
```rust
fn main() {
    let x: i32 = 300;
    let y = x as u8;
    println!("{y}");
}
```
::

- [ ] Compile error: `300` does not fit in a `u8`
- [ ] Panics at runtime with an overflow error, same as arithmetic overflow
- [ ] `44` — `as` performs a truncating cast that keeps only the low 8 bits of the value's two's-complement representation, silently discarding the rest, in both debug and release
- [ ] `255`, because `as` saturates to the target type's maximum value

<details>
<summary>Show Answer</summary>

**Answer:** C — `44` — `as` performs a truncating cast that keeps only the low 8 bits, silently discarding the rest, in both debug and release

**Explanation:** Unlike arithmetic operators, `as` numeric casts are **not** checked for overflow in either debug or release — `300` in binary is `0b1_0010_1100`; truncating to 8 bits keeps `0b0010_1100 = 44`. This silent truncation (no panic, no error, consistent across profiles) is a well-known gotcha distinct from arithmetic overflow's debug-only panic behavior (Q3/Q4) — if you want saturating or checked conversion instead, use `u8::try_from(x)` (returns `Result`) or the saturating conversion methods, not a bare `as`.

</details>
::

::question-wrapper
### Q6. Which statement correctly describes Rust's `char` type?

- [ ] A `char` is always exactly 1 byte, like in C
- [ ] A `char` represents a single Unicode scalar value and is always 4 bytes in memory, which is not the same as "one byte" or even always "one visual character" (e.g. some emoji/grapheme clusters require multiple `char`s)
- [ ] A `char` is a UTF-8 encoded byte sequence of variable length
- [ ] A `char` can only represent ASCII characters

<details>
<summary>Show Answer</summary>

**Answer:** B — A `char` represents a single Unicode scalar value and is always 4 bytes in memory

**Explanation:** Rust's `char` is a 32-bit value representing any Unicode Scalar Value (a subset of all Unicode code points, excluding surrogate halves), not a raw byte — this is different from indexing into a `String`'s UTF-8 bytes, where individual bytes are `u8`, not `char`. It's also not always one "visual glyph" — a grapheme cluster like a flag emoji or accented character built from combining marks can require multiple `char`s to represent, which trips up naive character-counting code (contradicting the assumption in A that mirrors C's 1-byte `char`).

</details>
::

::question-wrapper{language="rust"}
### Q7. What is the difference between a tuple and an array in Rust?

::code-wrapper{language="rust"}
```rust
let t: (i32, f64, bool) = (1, 2.0, true);
let a: [i32; 3] = [1, 2, 3];
```
::

- [ ] They are identical; `(i32, f64, bool)` and `[i32; 3]` are just two syntaxes for the same thing
- [ ] A tuple can hold elements of different types with a fixed length known at compile time; an array holds elements of a single type with a fixed length known at compile time — arrays are indexed with `[]`, tuples with `.0`, `.1`, etc.
- [ ] Arrays can hold mixed types, but tuples cannot
- [ ] Tuples are heap-allocated and growable, while arrays are always stack-allocated and fixed-size

<details>
<summary>Show Answer</summary>

**Answer:** B — A tuple can hold elements of different types with a fixed compile-time length; an array holds a single element type with a fixed compile-time length; arrays use `[]` indexing, tuples use `.0`/`.1`/etc.

**Explanation:** Both are fixed-size, stack-allocatable (unless boxed) compound types, but they differ in type homogeneity and access syntax — `t.0` accesses a tuple field, while `a[0]` accesses an array element. Option D is backwards regarding growability: neither is growable — that's what `Vec` is for — and both are typically stack-allocated by default, not heap-allocated.

</details>
::

::question-wrapper{language="rust"}
### Q8. What happens when this floating-point comparison runs?

::code-wrapper{language="rust"}
```rust
fn main() {
    let x = 0.1 + 0.2;
    println!("{}", x == 0.3);
}
```
::

- [ ] `true`, since `0.1 + 0.2` is mathematically `0.3`
- [ ] `false` — IEEE 754 binary floating-point cannot represent `0.1`, `0.2`, or `0.3` exactly, so `0.1 + 0.2` yields `0.30000000000000004`, which is not bit-for-bit equal to the literal `0.3`
- [ ] Compile error: floating-point values cannot be compared with `==`
- [ ] It depends on whether the build is debug or release

<details>
<summary>Show Answer</summary>

**Answer:** B — `false` — IEEE 754 binary floating-point cannot represent `0.1`, `0.2`, or `0.3` exactly, so `0.1 + 0.2` yields `0.30000000000000004`

**Explanation:** This is Rust's version of the classic cross-language floating-point "wat": `f64` uses base-2 (binary) fractional representation, and decimal fractions like 0.1 are repeating binary fractions that get rounded to the nearest representable `f64`, so accumulated rounding error makes `0.1 + 0.2 != 0.3` bit-for-bit. Unlike overflow (Q3/Q4), this behavior is identical in debug and release (ruling out D) — the fix in production code is comparing with an epsilon tolerance (e.g. `(x - 0.3).abs() < f64::EPSILON`) or using a decimal/fixed-point type when exactness matters.

</details>
::

::question-wrapper{language="rust"}
### Q9. What is the result of indexing an array out of bounds with a compile-time-constant index, as shown here?

::code-wrapper{language="rust"}
```rust
fn main() {
    let a = [1, 2, 3];
    let x = a[5];
    println!("{x}");
}
```
::

- [ ] It compiles and panics at runtime with an index-out-of-bounds message
- [ ] Compile error — since both the array's length and the index are known constants at compile time, the compiler statically rejects this rather than deferring to a runtime panic
- [ ] It compiles and returns a default value of `0`
- [ ] It compiles and returns garbage/undefined memory contents, as in C

<details>
<summary>Show Answer</summary>

**Answer:** B — Compile error, because both the array's length and the index are known constants, letting the compiler statically reject it

**Explanation:** **Safety:** when the index and array size are both compile-time constants, `rustc` performs constant evaluation and rejects out-of-bounds access with a hard compile error (`this operation will panic at runtime` / index out of bounds lint promoted to a hard error) rather than waiting for a runtime panic. This is a genuine edge case beginners often get backwards — they expect all indexing errors to be runtime-only (as in the more general case of Q10, where the index is not a compile-time constant), but constant-folded indices are checked earlier. Undefined behavior (D) never occurs in safe Rust indexing, unlike C's raw array access.

</details>
::

::question-wrapper{language="rust"}
### Q10. What happens with a runtime-computed (non-constant) out-of-bounds index?

::code-wrapper{language="rust"}
```rust
fn main() {
    let a = [1, 2, 3];
    let i = get_index();
    println!("{}", a[i]);
}

fn get_index() -> usize {
    5
}
```
::

- [ ] Compile error, identical to the constant-index case
- [ ] It compiles successfully and panics at runtime with an "index out of bounds" message when executed
- [ ] It compiles and silently returns `0`
- [ ] It compiles and wraps around to `a[5 % 3]`, i.e. `a[2]`

<details>
<summary>Show Answer</summary>

**Answer:** B — It compiles successfully and panics at runtime with an "index out of bounds" message when executed

**Explanation:** **Safety:** because `i`'s value isn't known until runtime, the compiler cannot statically reject it as it did in Q9 — instead, array/slice indexing always performs a bounds check at runtime, panicking (`index out of bounds: the len is 3 but the index is 5`) rather than silently wrapping (D, a C-like buffer-overrun assumption) or returning a default (C). The safe alternative to avoid the panic is `a.get(i)`, which returns `Option<&i32>` (`None` here) instead of panicking.

</details>
::

::question-wrapper{language="rust"}
### Q11. What is the result of this signed/unsigned cast?

::code-wrapper{language="rust"}
```rust
fn main() {
    let x: i32 = -1;
    let y = x as u32;
    println!("{y}");
}
```
::

- [ ] Compile error: cannot cast a negative value to an unsigned type
- [ ] Panics at runtime: negative value cannot be represented as unsigned
- [ ] `4294967295` — the cast reinterprets `-1`'s two's-complement bit pattern (`0xFFFFFFFF`) as an unsigned value, i.e. `u32::MAX`
- [ ] `0`, because negative values saturate to the unsigned minimum

<details>
<summary>Show Answer</summary>

**Answer:** C — `4294967295` — the cast reinterprets `-1`'s two's-complement bit pattern as an unsigned value, i.e. `u32::MAX`

**Explanation:** As with the truncating cast in Q5, `as` between integer types of the same width just reinterprets the bit pattern rather than doing "safe" numeric conversion — `-1i32`'s bits are all ones, which as `u32` is the maximum value `4294967295`. This silent, non-panicking reinterpretation (no compile error, no runtime panic, consistent in debug and release) is a frequent source of subtle bugs when developers assume `as` behaves like a checked conversion; use `u32::try_from(x)` if you want a `Result::Err` on negative input instead.

</details>
::

::question-wrapper{language="rust"}
### Q12. What does this code do with an empty tuple `()`?

::code-wrapper{language="rust"}
```rust
fn log_event(msg: &str) -> () {
    println!("{msg}");
}

fn main() {
    let result = log_event("started");
    println!("{result:?}");
}
```
::

- [ ] Compile error: `()` is not a valid return type
- [ ] It compiles and prints `started` followed by `()`, since `()` (the "unit type") is the zero-sized type conventionally used to mean "no meaningful value," and it implements `Debug`
- [ ] It compiles but `result` cannot be printed because `()` has no fields
- [ ] `-> ()` is redundant syntax error; functions with no return type must omit the arrow entirely

<details>
<summary>Show Answer</summary>

**Answer:** B — It compiles and prints `started` followed by `()`; `()` is the unit type conventionally used for "no meaningful value," and it implements `Debug`

**Explanation:** `()`, the unit type, is what a function implicitly returns when no `-> Type` is written at all (`-> ()` here is just the explicit, equivalent spelling — idiomatically omitted, but not an error, contradicting D). It's a real, zero-sized value that implements `Debug` (printing as literally `()`), so `println!("{result:?}")` works fine — it's not the absence of a value like `void` conceptually implies in some other languages, but an actual (if trivial) value that can be bound, passed around, and matched on.

</details>
::

::question-wrapper
### Q13. What is the maximum safely representable integer using `f64` such that all integers up to that value can be represented exactly?

- [ ] `2^53` (about 9 quadrillion), because `f64` has a 52-bit mantissa (53 bits with the implicit leading bit), beyond which not every integer has an exact representation
- [ ] `f64` can represent all `i64` integers exactly with no limit, since it's a 64-bit type
- [ ] `2^32`, matching `u32::MAX`
- [ ] There is no such limit; floating point can represent any integer exactly

<details>
<summary>Show Answer</summary>

**Answer:** A — `2^53`, because `f64`'s 52-bit mantissa (plus implicit leading bit) limits exact integer representation beyond that point

**Explanation:** An IEEE 754 `f64` has 1 sign bit, 11 exponent bits, and 52 explicit mantissa bits (53 bits of precision including the implicit leading `1`), so integers beyond `2^53` start silently losing precision when represented as `f64` — a tempting-but-wrong assumption (B) is that all 64-bit values fit, since `f64`'s *total* bit width is 64 but its *mantissa* width is what actually limits integer precision, unlike a genuinely 64-bit-precision integer type like `i64`/`u64`.

</details>
::

::question-wrapper{language="rust"}
### Q14. What happens when parsing an out-of-range value into a fixed-width integer type?

::code-wrapper{language="rust"}
```rust
fn main() {
    let result: Result<u8, _> = "300".parse();
    println!("{result:?}");
}
```
::

- [ ] `Ok(44)`, applying the same truncation behavior as an `as` cast
- [ ] `Err(ParseIntError { .. })`, because `.parse()` performs a checked conversion and returns an `Err` when the value doesn't fit the target type, unlike `as` which silently truncates
- [ ] It panics at runtime instead of returning a `Result`
- [ ] Compile error, because `"300"` is a string literal too large for the target type

<details>
<summary>Show Answer</summary>

**Answer:** B — `Err(ParseIntError { .. })`, because `.parse()` performs a checked conversion and returns an `Err` when the value doesn't fit, unlike `as`

**Explanation:** `str::parse::<u8>()` explicitly checks that the parsed numeric value fits within `u8`'s range (`0..=255`) and returns `Err` (specifically a `ParseIntError` with kind `PosOverflow`) rather than truncating — this is a deliberate contrast with `as`'s silent truncation (Q5), and a good example of why `.parse()` is the safer choice for untrusted/external input. The correct way to handle this `Err` case is via `match`, `?`, or `.unwrap_or_default()`/`.unwrap_or(default)` depending on whether the caller can recover, rather than assuming parsing always succeeds.

</details>
::

::question-wrapper
### Q15. When should you prefer an explicit-width integer type like `u16` or `i64` over the default `i32`?

- [ ] Never — always use `i32` everywhere for consistency regardless of the value's actual domain
- [ ] When the value's valid domain and required range are known and meaningful (e.g. a `u16` port number capped at 65535, or an `i64`/`u64` for a value that can exceed `i32::MAX` like a file size in bytes or a timestamp) — this documents intent and avoids both wasted space and overflow risk
- [ ] Only when targeting embedded/no_std platforms; on desktop/server code, width never matters
- [ ] Always prefer the smallest type that compiles without error, regardless of whether the value could plausibly grow

<details>
<summary>Show Answer</summary>

**Answer:** B — When the value's valid domain and required range are known and meaningful, choosing an explicit width documents intent and avoids wasted space or overflow risk

**Explanation:** **Idiom:** using `u16` for a network port or `i64`/`u64` for byte counts/timestamps is self-documenting and prevents a class of bugs where a value that can legitimately exceed `i32::MAX` (about 2.1 billion — easily exceeded by file sizes or millisecond timestamps) silently overflows in release builds (recall Q4). Blindly picking the smallest type that merely compiles today (D) is a trap — it says nothing about whether the value could grow beyond that type's range in production, which is exactly the kind of "compiles now, panics/wraps later" bug this rule of thumb prevents.

</details>
::

::question-wrapper
### Q16. What is the idiomatic way to convert between numeric types when the value's validity at the target type must be guaranteed rather than assumed, e.g. converting a user-supplied `i64` amount into a `u32` quantity field?

- [ ] Always use `as`, since it's the shortest syntax and Rust guarantees it never loses data
- [ ] Use `u32::try_from(value)`, which returns a `Result` that must be handled (via `?`, `match`, or explicit `.expect()` with a justified message), making conversion failure an explicit, visible part of the control flow
- [ ] Use `unsafe { std::mem::transmute(value) }` to reinterpret the bits directly
- [ ] Multiply and divide by powers of two manually to simulate the conversion

<details>
<summary>Show Answer</summary>

**Answer:** B — Use `u32::try_from(value)`, which returns a `Result` that must be handled, making conversion failure explicit

**Explanation:** **Safety:** `TryFrom`/`TryInto` conversions surface out-of-range or negative values as an `Err` at the call site instead of silently truncating/reinterpreting bits the way `as` does (Q5, Q11) — critical for values crossing a trust boundary (user input, network data, file parsing). `as` (A) is the tempting-but-dangerous default precisely because it "just works" syntactically while silently corrupting out-of-range values with no warning; `transmute` (C) is unrelated and unsafe low-level bit reinterpretation, wildly inappropriate for a simple numeric conversion.

</details>
::

::question-wrapper{language="rust"}
### Q17. What is the idiomatic and safe way to compare two floating-point values for near-equality, avoiding the pitfall in Q8?

::code-wrapper{language="rust"}
```rust
let a = 0.1_f64 + 0.2;
let b = 0.3_f64;
```
::

- [ ] `a == b`, trusting exact equality since both are the same type
- [ ] `(a - b).abs() < f64::EPSILON` or a domain-appropriate tolerance, since binary floating-point arithmetic accumulates rounding error and exact equality is unreliable for computed values
- [ ] Convert both to `String` and compare the strings
- [ ] `a.round() == b.round()`, since rounding always eliminates floating-point error

<details>
<summary>Show Answer</summary>

**Answer:** B — `(a - b).abs() < f64::EPSILON` or a domain-appropriate tolerance, since binary floating-point arithmetic accumulates rounding error

**Explanation:** **Idiom:** because `f64` cannot exactly represent most decimal fractions (Q8), idiomatic Rust (like most languages using IEEE 754) compares floats with an epsilon-based tolerance rather than `==`, especially after any arithmetic has occurred. Rounding first (D) is a fragile workaround that only coincidentally works for values far from a rounding boundary and breaks down near `x.5` boundaries or when more precision is actually needed; string comparison (C) is needlessly indirect and has its own formatting-precision pitfalls.

</details>
::

::question-wrapper
### Q18. A function needs to represent "a 2D coordinate pair" that will always have exactly two `f64` values and no more. What is more idiomatic — a tuple `(f64, f64)` or a two-element `Vec<f64>`?

- [ ] `Vec<f64>`, since vectors are always more efficient than tuples
- [ ] A tuple `(f64, f64)`, or better yet a small named `struct Point { x: f64, y: f64 }` — since the size (2) is fixed and known, a `Vec` would incorrectly imply a runtime-variable length and adds unnecessary heap allocation and indirection
- [ ] Neither; only arrays `[f64; 2]` are valid for fixed-size numeric data
- [ ] It makes no practical difference; `Vec`, tuple, and array are fully interchangeable in all Rust APIs

<details>
<summary>Show Answer</summary>

**Answer:** B — A tuple `(f64, f64)`, or better a named `struct Point { x: f64, y: f64 }` — since size is fixed, `Vec` wrongly implies variable length and adds needless heap allocation

**Explanation:** **Idiom:** `Vec<T>` is heap-allocated and growable/shrinkable at runtime — reaching for it when the length is a fixed, compile-time-known constant (like a 2D point) is both a performance cost (unnecessary allocation/indirection) and a readability/API-design smell, since it implies the length could vary when it never does. A named struct is generally preferred over a bare tuple once fields have clear semantic roles (`x`/`y` versus `.0`/`.1`), improving self-documentation, though the tuple is not wrong, just less expressive.

</details>
::

::question-wrapper
### Q19. What is the best-practice way to handle a computation that might overflow in a context where overflow represents a genuine, expected possibility (e.g. summing user-supplied quantities that could exceed `u32::MAX`), rather than a programmer bug?

- [ ] Rely on debug-mode panics to catch it during testing and ship release builds as-is, trusting silent wraparound is fine for production
- [ ] Use `checked_add`, which returns `Option<T>` (`None` on overflow), or `saturating_add`/`wrapping_add` depending on whether the correct domain behavior is to clamp, wrap, or explicitly reject the overflowing case — chosen deliberately rather than relying on the default panic/wrap split between debug and release
- [ ] Always cast to `i128`/`u128` everywhere to make overflow effectively impossible in every context
- [ ] Catch the debug-mode panic with `std::panic::catch_unwind` in production release builds

<details>
<summary>Show Answer</summary>

**Answer:** B — Use `checked_add` (returns `Option`), or `saturating_add`/`wrapping_add` depending on the correct domain behavior, chosen deliberately

**Explanation:** **Safety:** relying on the implicit debug-panic/release-wrap split (A) means the exact same overflow bug behaves completely differently between environments — code that "worked" in every debug test can silently corrupt data in production release builds. The explicit `checked_*`/`saturating_*`/`wrapping_*` family of methods makes the overflow-handling policy a deliberate, visible choice in the code rather than an accident of build profile; `catch_unwind` (D) is a heavyweight, inappropriate tool for ordinary expected-value validation, not a substitute for correct arithmetic handling.

</details>
::

::question-wrapper
### Q20. When choosing between an array `[T; N]` and a `Vec<T>` for function parameters/return types, what is the idiomatic guidance?

- [ ] Always use `Vec<T>` even for fixed, compile-time-known sizes, since it's more "flexible" by default
- [ ] Use `[T; N]` (or a slice `&[T]`) when the length is fixed and known at compile time, giving stack allocation and compile-time length guarantees; reach for `Vec<T>` when the length is only known at runtime or needs to grow/shrink
- [ ] Arrays and `Vec` are chosen purely based on personal preference with no technical trade-offs
- [ ] `[T; N]` should be avoided entirely in idiomatic Rust in favor of `Vec<T>` for all collections

<details>
<summary>Show Answer</summary>

**Answer:** B — Use `[T; N]` (or `&[T]`) when the length is fixed and compile-time known; reach for `Vec<T>` when the length is runtime-determined or needs to grow/shrink

**Explanation:** **Idiom:** fixed-size arrays avoid heap allocation entirely and encode the exact length in the type system (catching size-mismatch bugs at compile time, as seen with Q9's constant-index rejection), which is strictly better than `Vec` whenever the size genuinely never changes — e.g. `[u8; 32]` for a fixed-size hash digest. Defaulting to `Vec` everywhere (A) throws away those compile-time guarantees and adds unnecessary heap allocation/indirection for data whose size was never actually variable.

</details>
::
