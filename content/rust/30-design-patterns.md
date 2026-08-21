# 30 — Design Patterns

::question-wrapper{language="rust"}
### Q1. What is the defining structural feature of the builder pattern as commonly written in Rust?

::code-wrapper{language="rust"}
```rust
let req = RequestBuilder::new("https://api.example.com")
    .method("POST")
    .header("Content-Type", "application/json")
    .timeout_secs(30)
    .build()?;
```
::

- [ ] A separate builder struct accumulates configuration through chained methods that each consume and return `self` (or `&mut self`), with a final `build()` that validates and produces the target type
- [ ] A struct that implements `Default` and is mutated directly via public fields
- [ ] A macro that generates a constructor accepting every field as a positional argument
- [ ] A trait that all configurable types must implement, with no separate builder struct involved

<details>
<summary>Show Answer</summary>

**Answer:** A — a separate builder struct with chained methods and a final validating `build()`

**Explanation:** The builder pattern exists in Rust largely because the language has no named/default function arguments — chained `.method(...)` calls on a dedicated builder struct simulate that ergonomics while letting each step be independently optional, and `build()` gives a single place to validate combinations and return `Result` for invalid configurations. Direct public-field mutation skips validation and locks in field names/types as public API forever. A positional-argument macro reintroduces the exact problem builders solve (unreadable call sites, fragile argument order). A trait-only approach without a separate accumulating struct doesn't match how the pattern is idiomatically structured in the ecosystem (e.g. `reqwest::RequestBuilder`, `std::process::Command`).

</details>
::

::question-wrapper{language="rust"}
### Q2. What core problem does the newtype pattern (`struct Meters(f64);`) solve?

::code-wrapper{language="rust"}
```rust
struct Meters(f64);
struct Feet(f64);

fn distance_traveled(m: Meters) -> Meters { m }
```
::

- [ ] It creates a distinct type at compile time so values with the same underlying representation (e.g. two `f64`s meaning different units) cannot be accidentally interchanged
- [ ] It reduces the runtime memory footprint of `f64` by removing unused precision bits
- [ ] It's required by the compiler any time a struct has exactly one field
- [ ] It automatically implements `Add` and `Sub` for the wrapped type

<details>
<summary>Show Answer</summary>

**Answer:** A — creates a distinct compile-time type so same-representation values with different meanings can't be mixed up accidentally

**Explanation:** **Safety:** `Meters(f64)` and `Feet(f64)` share an identical runtime layout (a single `f64`, and with `#[repr(transparent)]` even guaranteed ABI-identical to `f64`) but are different types to the compiler — passing a `Feet` where `Meters` is expected is a compile error, catching unit-confusion bugs (the kind that famously destroyed the Mars Climate Orbiter) at compile time instead of production. It has zero effect on runtime memory representation/precision — the wrapper is typically free at runtime (a compile-time-only distinction). It's a voluntary pattern, not a compiler requirement for single-field structs (plenty of single-field structs aren't newtypes in this sense). And trait impls like `Add`/`Sub` are never automatic — they must be explicitly implemented (or derived, where applicable) for the new type; wrapping a value implements nothing by itself.

</details>
::

::question-wrapper{language="rust"}
### Q3. The newtype pattern is also used to work around Rust's orphan rule. Which scenario is a correct use of newtype for this purpose?

::code-wrapper{language="rust"}
```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}
```
::

- [ ] Wrapping a foreign type (`Vec<String>`, from `std`) in a local tuple struct so you can implement a foreign trait (`Display`, also from `std`) on the local wrapper, since neither the trait nor the raw type is local
- [ ] Wrapping a local struct so you can implement a local trait on it, which would otherwise be impossible
- [ ] Using newtype to bypass the borrow checker's mutable-aliasing rules
- [ ] The orphan rule doesn't apply to `Display`, so no wrapper is needed here at all

<details>
<summary>Show Answer</summary>

**Answer:** A — wrap the foreign type locally so the impl target (the wrapper) is local, satisfying the orphan rule for the foreign trait

**Explanation:** **Idiom:** The orphan rule blocks `impl Display for Vec<String>` directly because neither `Display` nor `Vec` is defined in your crate — both are foreign. Wrapping `Vec<String>` in a local tuple struct `Wrapper` makes the impl target local, satisfying "at least one of trait or type must be local," even though the trait itself remains foreign. Implementing a local trait on a local type was never blocked by the orphan rule in the first place (that's always allowed), so option B describes a non-problem. Newtype has nothing to do with borrow-checker aliasing rules — those are governed by `&`/`&mut` reference rules, unrelated to type wrapping. And `Display` is subject to the orphan rule exactly like any other foreign trait; there's no special exemption.

</details>
::

::question-wrapper{language="rust"}
### Q4. What does the typestate pattern encode, and how does it typically prevent misuse at compile time?

::code-wrapper{language="rust"}
```rust
struct Locked;
struct Unlocked;

struct Door<State> {
    _state: std::marker::PhantomData<State>,
}

impl Door<Locked> {
    fn unlock(self, _key: &str) -> Door<Unlocked> { Door { _state: std::marker::PhantomData } }
}

impl Door<Unlocked> {
    fn open(self) -> Door<Unlocked> { println!("opened"); self }
}
```
::

- [ ] It encodes an object's runtime state as distinct types (often via a generic parameter), so methods only valid in a given state are only defined on that state's type — calling `open()` on a `Door<Locked>` is a compile error, not a runtime check
- [ ] It stores the current state as an enum field and uses a `match` inside every method to check validity at runtime
- [ ] It's a synonym for the state machine pattern implemented purely with `if`/`else` branches
- [ ] It requires `unsafe` to transition between states since ownership must be forcibly reinterpreted

<details>
<summary>Show Answer</summary>

**Answer:** A — states become distinct types, so only valid-for-that-state methods exist, and invalid transitions fail to compile

**Explanation:** **Safety:** In the typestate pattern, `Door<Locked>` and `Door<Unlocked>` are different types with non-overlapping method sets (`unlock` only exists on `Door<Locked>`, `open` only on `Door<Unlocked>`), so attempting `Door<Locked>::open()` is a "no method named `open` found" compile error — the illegal transition is caught before the program ever runs, unlike the described enum-plus-runtime-`match` approach (option B), which is a valid but strictly weaker pattern (it moves the same check to runtime, where a forgotten branch panics or misbehaves in production instead of failing the build). It's related to but distinct from a general state machine (which typestate implements specifically via the type system, not just any state-tracking technique), and it requires no `unsafe` — transitions are ordinary by-value moves consuming the old-state value and returning a new-state value.

</details>
::

::question-wrapper{language="rust"}
### Q5. What guarantee does RAII (Resource Acquisition Is Initialization) via `Drop` provide in Rust?

::code-wrapper{language="rust"}
```rust
struct FileGuard {
    file: std::fs::File,
}

impl Drop for FileGuard {
    fn drop(&mut self) {
        println!("closing file");
    }
}
```
::

- [ ] Cleanup code in `drop()` runs automatically when the value goes out of scope, whether via normal control flow, an early `return`, or an unwinding panic (barring `std::mem::forget` or a process abort)
- [ ] `drop()` only runs if the program exits normally via `main` returning `Ok(())`
- [ ] `drop()` must be called manually by the programmer; Rust does not call it automatically
- [ ] `drop()` runs at a nondeterministic time chosen by the garbage collector, similar to Java's `finalize()`

<details>
<summary>Show Answer</summary>

**Answer:** A — `drop()` runs automatically at scope exit through normal flow, early return, or unwinding, with only `mem::forget`/abort as exceptions

**Explanation:** **Safety:** Rust has deterministic, scope-based destruction (no garbage collector) — the compiler inserts a call to `drop()` at every point a value's owning scope ends, including early returns and (during unwinding, not `panic = "abort"` builds) stack unwinding from a panic, which is what makes RAII reliable for releasing locks, closing files, and flushing buffers even in error paths. The two real escape hatches are `std::mem::forget` (explicitly suppresses the drop) and process abort/`SIGKILL` (no unwinding happens at all). It is not tied to `main` returning `Ok`, is very much automatic (that's the entire point of RAII — no explicit `.close()` call needed, unlike C's manual `fclose`), and has nothing to do with garbage collection or Java's notoriously non-deterministic, sometimes-never-called `finalize()` — Rust's timing is fully deterministic and scope-derived.

</details>
::

::question-wrapper{language="rust"}
### Q6. In the visitor pattern implemented via a Rust enum (rather than the classic OOP double-dispatch visitor), how is "visiting different node types" typically expressed?

::code-wrapper{language="rust"}
```rust
enum Expr {
    Num(f64),
    Add(Box<Expr>, Box<Expr>),
    Mul(Box<Expr>, Box<Expr>),
}

fn eval(e: &Expr) -> f64 {
    match e {
        Expr::Num(n) => *n,
        Expr::Add(l, r) => eval(l) + eval(r),
        Expr::Mul(l, r) => eval(l) * eval(r),
    }
}
```
::

- [ ] An exhaustive `match` over the enum's variants, where the compiler enforces that every variant is handled — new variants force every existing `match` to be updated or fail to compile
- [ ] A separate `Visitor` trait with one method per variant is mandatory; enums cannot be visited without it
- [ ] Dynamic dispatch through `Box<dyn Any>` and runtime type-checking via `downcast_ref`
- [ ] Enums cannot express the visitor pattern in Rust; only trait objects can

<details>
<summary>Show Answer</summary>

**Answer:** A — an exhaustive `match`, with the compiler enforcing every variant is handled

**Explanation:** **Idiom:** Rust's enums plus exhaustive `match` give you the visitor pattern's core benefit (dispatch based on concrete node kind) without needing the classic double-dispatch machinery OOP languages use to work around lacking sum types — and critically, adding a new `Expr` variant makes every non-wildcard `match` across the codebase a compile error until updated, which is a *stronger* guarantee than the classic visitor pattern's `Visitor` trait (where forgetting to implement a new visit method for a new node type is usually just a silent no-op or an easy-to-miss default). A separate `Visitor` trait is a valid *alternative* implementation style (useful when the set of "visitors"/operations grows faster than the set of node types), not a requirement — enums alone are sufficient and commonly used exactly this way (e.g. rustc's own AST handling). `dyn Any` downcasting throws away compile-time exhaustiveness checking entirely and is not idiomatic here.

</details>
::

::question-wrapper
### Q7. Why might a codebase choose the classic trait-based visitor pattern (a `Visitor` trait with a `visit_*` method per node type) over a plain `match`-based enum approach for an AST?

- [ ] When new *operations* over the AST are added frequently but the set of node *types* is stable — the trait-based visitor makes adding an operation a matter of writing one new impl, rather than editing every existing `match`
- [ ] The trait-based visitor is strictly faster at runtime in all cases due to avoiding `match` branch prediction misses
- [ ] `match`-based enums cannot support recursive tree structures at all
- [ ] There's no reason to ever prefer it — `match`-based enums are strictly superior in every scenario

<details>
<summary>Show Answer</summary>

**Answer:** A — when operations grow faster than node types, trait-based visitors avoid editing every existing `match`

**Explanation:** **Idiom:** This is the classic "expression problem" trade-off: enum + `match` makes adding a new *node type* force-update every match site (good when types are the axis of change, as in Q6), while trait-based visitor makes adding a new *operation* (a new `impl Visitor`) require zero changes to existing code, at the cost of adding a new *node type* now requiring updates across every existing `Visitor` implementor. Neither is "strictly superior" — the right choice depends on which axis (types vs. operations) changes more often in that codebase. There's no inherent, universal runtime performance advantage to virtual dispatch over a `match` (a `match` on a fieldless discriminant is typically a fast jump table, often faster than a vtable call), and enums recurse over trees perfectly well via `Box`/`Rc` indirection, as shown in Q6's `Expr` — recursion is not a limitation of the enum approach.

</details>
::

::question-wrapper{language="rust"}
### Q8. A `Config` struct has 8 optional fields, most with sensible defaults. What happens if a builder's `build()` method is called before any required fields (say, `api_key`) are set?

::code-wrapper{language="rust"}
```rust
struct ConfigBuilder {
    api_key: Option<String>,
    timeout: u64,
}

impl ConfigBuilder {
    fn build(self) -> Result<Config, BuildError> {
        let api_key = self.api_key.ok_or(BuildError::MissingApiKey)?;
        Ok(Config { api_key, timeout: self.timeout })
    }
}
```
::

- [ ] `build()` returns `Err(BuildError::MissingApiKey)`, since the required field was validated and found absent — this is the idiomatic way to surface missing-required-field errors without a panic
- [ ] `build()` panics immediately with an "unwrap on None" message
- [ ] `build()` silently substitutes an empty string for `api_key` and returns `Ok`
- [ ] This code fails to compile because `Option<String>` can't be used inside a builder struct

<details>
<summary>Show Answer</summary>

**Answer:** A — `build()` returns `Err(BuildError::MissingApiKey)`, the idiomatic way to surface a missing required field

**Explanation:** Using `.ok_or(...)?` converts the `Option<String>` into a `Result`, propagating a descriptive error rather than panicking — this is exactly why `build()` idiomatically returns `Result<Config, BuildError>` instead of `Config` directly, letting callers handle missing-configuration errors gracefully (e.g. surfacing a helpful message) instead of crashing. It does not panic (there's no `.unwrap()` anywhere in this path), does not silently default a security-sensitive field like an API key to empty string (that would be a dangerous silent failure mode, not idiomatic error handling), and `Option<String>` inside a plain struct is completely ordinary, valid Rust with no compile issue.

</details>
::

::question-wrapper{language="rust"}
### Q9. What happens when a typestate-encoded value is used after a state-transitioning method has consumed it?

::code-wrapper{language="rust"}
```rust
let door = Door::<Locked> { _state: std::marker::PhantomData };
let unlocked = door.unlock("1234");
door.unlock("1234");
```
::

- [ ] Compile error — `unlock` takes `self` by value, so `door` is moved into the first call; the second `door.unlock(...)` is a use-after-move error
- [ ] Both calls succeed; `door` is implicitly cloned since it's a zero-sized type
- [ ] The second call panics at runtime with "use of moved value"
- [ ] `door` is automatically re-locked and re-usable since `Door` implements `Copy` by default

<details>
<summary>Show Answer</summary>

**Answer:** A — compile error; `unlock(self)` moves `door`, so the second call is a use-after-move

**Explanation:** **Safety:** This is precisely why typestate methods take `self` by value rather than `&self`/`&mut self` — consuming `self` guarantees the *old*-state value cannot be reused after transitioning, which is what makes a state transition genuinely one-way and irreversible at compile time (you can't accidentally "unlock an already-consumed door twice"). The borrow checker rejects the second `door.unlock(...)` at compile time with a "use of moved value" error — this is a compile-time diagnostic, never a runtime panic, since move-checking has no runtime component. Structs are never implicitly `Copy` by default in Rust (it must be explicitly derived and is only valid when every field is `Copy`), and even a zero-sized `PhantomData`-only struct doesn't get free `Copy` without `#[derive(Copy, Clone)]` explicitly opting in.

</details>
::

::question-wrapper{language="rust"}
### Q10. In a `Drop` implementation, what happens if `drop()` itself panics while the program is already unwinding from a prior panic (i.e. a "double panic")?

::code-wrapper{language="rust"}
```rust
struct Noisy;
impl Drop for Noisy {
    fn drop(&mut self) {
        panic!("dropped during unwind");
    }
}
```
::

- [ ] The process aborts immediately — Rust cannot unwind through two simultaneous panics, so this becomes a hard abort instead of a normal panic-and-recover
- [ ] The second panic is silently swallowed and unwinding continues normally
- [ ] The second panic simply replaces the first, and the program continues as if only one panic occurred
- [ ] `Drop::drop` cannot panic; the compiler rejects any `drop()` body containing `panic!`

<details>
<summary>Show Answer</summary>

**Answer:** A — the process aborts immediately; Rust cannot unwind through two simultaneous panics

**Explanation:** **Safety/Debug:** If a `drop()` runs as part of unwinding from an earlier panic and itself panics, Rust has no defined way to unwind two panics at once through the same stack, so the runtime immediately calls `abort()` — no further cleanup, no graceful shutdown, process terminates hard. This is a production-relevant gotcha: `Drop` implementations should avoid any code that can panic (avoid `.unwrap()`, indexing, arithmetic that can overflow in debug mode, etc.), preferring to log-and-continue or use fallible alternatives, precisely because a panicking destructor during unwinding is one of the few ways to lose the ability to gracefully report an error at all. The compiler does not reject `panic!` inside `drop()` — it's syntactically and semantically legal, just dangerous in this specific double-unwind scenario; the double-panic-aborts behavior is a hard runtime rule, not something silently absorbed or overwritten.

</details>
::

::question-wrapper{language="rust"}
### Q11. A struct wraps a `Vec<u8>` as a newtype to represent a validated, non-empty buffer. What is the most common mistake that defeats the purpose of this newtype?

::code-wrapper{language="rust"}
```rust
pub struct NonEmptyBuffer(Vec<u8>);

impl NonEmptyBuffer {
    pub fn new(data: Vec<u8>) -> Option<Self> {
        if data.is_empty() { None } else { Some(Self(data)) }
    }
}
```
::

- [ ] Leaving the tuple field `Vec<u8>` public (`pub Vec<u8>` instead of a private field), which lets any caller construct `NonEmptyBuffer(vec![])` directly, bypassing the validating `new()` and violating the non-empty invariant
- [ ] Using `Option<Self>` instead of `Result<Self, Error>` in the constructor, which is always wrong
- [ ] Deriving `Clone` on the struct, which would allow duplicating the buffer
- [ ] Naming the constructor `new()` instead of `from_vec()`

<details>
<summary>Show Answer</summary>

**Answer:** A — leaving the inner field public lets callers construct the newtype directly, bypassing validation

**Explanation:** **Safety:** The entire value of a "validated newtype" (non-empty buffer, positive integer, normalized email, etc.) depends on there being no way to construct one except through the checked constructor — if the tuple field is `pub`, anyone can write `NonEmptyBuffer(vec![])` directly from outside the module, completely bypassing `new()`'s emptiness check and silently reintroducing the exact bug the type was created to prevent. Keeping the field private (as shown, no `pub` on `Vec<u8>`) is what makes `new()` the *only* construction path. `Option` vs `Result` in the constructor is a legitimate, context-dependent API choice (not "always wrong" — `Option` is fine when there's only one failure reason and no extra context to convey). Deriving `Clone` is harmless here — cloning a valid `NonEmptyBuffer` produces another valid one, since the invariant (non-empty) is preserved by copying the same data. Constructor naming is a style preference with no soundness implication.

</details>
::

::question-wrapper{language="rust"}
### Q12. A builder's `build()` method is called twice on the same builder instance. Given standard consuming-builder design (`fn build(self) -> T`), what happens on the second call?

::code-wrapper{language="rust"}
```rust
let builder = RequestBuilder::new("url");
let req1 = builder.build();
let req2 = builder.build();
```
::

- [ ] Compile error — `build(self)` consumes the builder by value on the first call, making `builder` unavailable for the second call (use-after-move)
- [ ] Both calls succeed and produce two independent, identically-configured requests
- [ ] The second call returns a default-constructed, empty request instead of erroring
- [ ] It compiles, but the second call panics at runtime with "builder already consumed"

<details>
<summary>Show Answer</summary>

**Answer:** A — compile error; `build(self)` moves the builder, so calling it twice is a use-after-move caught at compile time

**Explanation:** **Idiom:** Taking `self` by value in `build()` is a deliberate design choice (not an accident) — it statically enforces "a builder can only be finalized once," turning a whole class of "did I already call `.build()`?" bugs into compile errors rather than needing runtime tracking. This is why many builder APIs use `self` (consuming) rather than `&mut self` for chained methods that lead into `build()`: it composes with move semantics to make double-finalization structurally impossible, exactly analogous to the typestate consumption in Q9. There's no runtime panic path here — the compiler rejects the second call before the program can ever run, and there's no default-empty fallback or implicit duplication; if a caller genuinely needs to build multiple times, the builder type would need to implement `Clone` explicitly, or expose `build(&self)` instead (a valid alternative design with its own trade-offs, but not what's shown here).

</details>
::

::question-wrapper{language="rust"}
### Q13. What happens if a `Drop` impl is written for a type that also derives `Copy`?

::code-wrapper{language="rust"}
```rust
#[derive(Clone, Copy)]
struct Point { x: i32, y: i32 }

impl Drop for Point {
    fn drop(&mut self) {}
}
```
::

- [ ] This is a compile error — `Copy` and `Drop` are mutually exclusive on the same type, because `Copy` implies bitwise duplication with no notion of "the original is now gone," which conflicts with `Drop`'s single-owner cleanup guarantee
- [ ] This compiles fine; `drop()` simply runs once per each bitwise copy independently
- [ ] This compiles, but `Copy` is silently ignored and the type behaves as `Clone`-only
- [ ] This compiles only if `Drop::drop` is an empty function body, as shown

<details>
<summary>Show Answer</summary>

**Answer:** A — compile error; `Copy` and `Drop` are mutually exclusive by design

**Explanation:** **Safety:** `Copy` means assignment/passing duplicates the bits with no move semantics — both the "original" and the "copy" remain simultaneously valid and usable. `Drop` is built entirely around Rust's ownership guarantee that a value has exactly one owner responsible for cleanup at scope end. If a type were both, you'd get either a double-free (both bitwise copies' destructors trying to free/release the same resource) or an ill-defined "which copy actually owns the resource" question — so the compiler flatly rejects `#[derive(Copy)]` (or a manual `impl Copy`) on any type that also `impl Drop`, regardless of whether the `drop()` body is empty (option D is a red herring: the restriction is structural/type-level, not based on the body's contents). This isn't silently downgraded to `Clone`-only either — it's a hard compile error at the `derive`/`impl` site.

</details>
::

::question-wrapper{language="rust"}
### Q14. A typestate API models a network connection with `Disconnected -> Connecting -> Connected` states. A method needs to handle the case where connecting can fail and should return to `Disconnected`. How is this typically expressed in the return type?

::code-wrapper{language="rust"}
```rust
impl Connection<Connecting> {
    fn finish(self) -> Result<Connection<Connected>, Connection<Disconnected>> {
        // ...
    }
}
```
::

- [ ] Return a `Result<Connection<Connected>, Connection<Disconnected>>` (or a similar `Result`/enum of the two possible resulting typestates), so callers must handle both outcomes and the compiler enforces that the returned value's state matches what actually happened
- [ ] Panic on connection failure, since typestate patterns cannot express fallible transitions
- [ ] Return `Connection<Connecting>` unchanged and let the caller retry indefinitely
- [ ] Use `unsafe { std::mem::transmute }` to force the state type back to `Disconnected`

<details>
<summary>Show Answer</summary>

**Answer:** A — return a `Result` (or equivalent) whose `Ok`/`Err` carry the two possible resulting typestates

**Explanation:** **Idiom:** Fallible transitions compose naturally with typestate by making the *return type itself* encode "one of these two states is what you'll get," typically via `Result<Connection<Connected>, Connection<Disconnected>>` or a small enum — the caller is then compiler-forced (via `match`/`?`) to handle both branches, and whichever branch executes hands back a correctly-typed value for that actual state, preserving the whole pattern's compile-time guarantees through the fallible step. Typestate absolutely can express fallible transitions this way — panicking would be a strictly worse design that throws away the ability to recover from an ordinary, expected failure (a network connect timing out is not exceptional). Returning `Connection<Connecting>` unchanged doesn't reflect reality (the state actually changed to failed/disconnected) and invites infinite-retry bugs. Using `transmute` to force a type back is exactly the kind of unsound hack the entire pattern exists to make unnecessary — legitimate typestate code should never need `unsafe` to move between states.

</details>
::

::question-wrapper{language="rust"}
### Q15. When is it idiomatic to use the builder pattern versus simply using a struct literal with `..Default::default()`?

::code-wrapper{language="rust"}
```rust
let cfg = Config { timeout: 30, ..Default::default() };
```
::

- [ ] Prefer `..Default::default()` struct-update syntax for simple, all-public-field configs with no cross-field validation needed; reach for a builder when construction needs validation, computed/derived defaults, required fields enforced at compile time, or a fluent multi-step API
- [ ] The builder pattern should always be used, even for a two-field public struct, since it's considered more professional
- [ ] `..Default::default()` cannot be combined with named field initialization in the same literal
- [ ] Builders are strictly for compatibility with older Rust editions; `..Default::default()` fully replaces them in modern code

<details>
<summary>Show Answer</summary>

**Answer:** A — use `..Default::default()` for simple public-field configs with no validation; use a builder when validation, derived defaults, or required-field enforcement is needed

**Explanation:** **Idiom:** `..Default::default()` is lighter-weight and perfectly idiomatic when a struct's fields are all meant to be public and independently valid in any combination — it avoids the ceremony of a whole separate builder type for something trivial. A builder earns its complexity when construction has invariants to check (cross-field validation), needs a required field enforced (impossible to skip `api_key`, unlike a `Default`-based struct literal where every field is implicitly optional-with-a-default), or benefits from a readable fluent chain for many optional settings. "Always use a builder" ignores this real trade-off and adds needless boilerplate for simple cases. The syntax shown (`Config { timeout: 30, ..Default::default() }`) is exactly the valid, common combination of named fields plus a base — the claim that they can't be combined is false. And builders remain a first-class, actively-used pattern in modern Rust (e.g. `std::process::Command`, `reqwest::ClientBuilder`) — they were never edition-specific or made obsolete by `Default`.

</details>
::

::question-wrapper{language="rust"}
### Q16. What is the idiomatic way to make a builder's chained setter methods ergonomic for method chaining while still being usable in a non-chained, imperative style?

::code-wrapper{language="rust"}
```rust
impl RequestBuilder {
    pub fn header(mut self, key: &str, val: &str) -> Self {
        self.headers.push((key.into(), val.into()));
        self
    }
}
```
::

- [ ] Take `mut self` by value and return `Self`, mutating the owned copy and handing it back — this supports both `builder.header(..).header(..)` chaining and `let b = b.header(..);` reassignment style
- [ ] Take `&mut self` and return `()`, which is the only way to support method chaining in Rust
- [ ] Take `&self` (immutable) and return a brand-new heap-allocated builder each call, which is the most performant option
- [ ] It's impossible to support both chained and non-chained calling styles with the same method signature

<details>
<summary>Show Answer</summary>

**Answer:** A — take `mut self` by value, return `Self`; supports both chaining and imperative reassignment

**Explanation:** **Idiom:** Taking `self` by value (not `&mut self`) and returning `Self` is the standard consuming-builder signature: it supports fluent chaining (`RequestBuilder::new(url).header(...).header(...).build()`) because each call produces a new owned value to call the next method on, *and* it equally supports the imperative style `let b = b.header(...);` since it's just an ordinary function taking and returning a value — no special calling convention required. `&mut self -> ()` cannot be chained at all (there's nothing returned to call the next method on — you'd need separate statements), so it's actually the *opposite* of what enables chaining, making that option's claim backwards. `&self` returning a freshly allocated builder is unnecessary allocation churn for no benefit over consuming-and-returning the existing owned value, and it isn't "more performant" — it's strictly more allocation for the same result. Supporting both styles with one signature is exactly what the consuming pattern already does, contrary to the last option.

</details>
::

::question-wrapper{language="rust"}
### Q17. Best practice: should a `Drop` implementation perform fallible I/O (e.g. flushing a buffered writer to disk) directly, given that `drop()` cannot return a `Result`?

::code-wrapper{language="rust"}
struct BufferedLog { writer: std::io::BufWriter<std::fs::File> }
```
::

- [ ] No — since `drop()` can't propagate errors, best practice is to expose an explicit fallible `close()`/`flush()` method that callers invoke to handle errors properly, with `Drop` only as a best-effort fallback (often logging-and-swallowing) for callers who forgot
- [ ] Yes — `Drop` is the only place cleanup should ever happen; explicit `close()` methods are redundant and should be avoided
- [ ] It doesn't matter; any error inside `drop()` automatically becomes the return value of the enclosing function
- [ ] Fallible I/O should never appear in RAII types at all; buffered writers should not implement `Drop`

<details>
<summary>Show Answer</summary>

**Answer:** A — expose an explicit fallible `close()`/`flush()` for real error handling; `Drop` is only a best-effort fallback

**Explanation:** **Idiom/Debug:** Because `Drop::drop(&mut self)` has a fixed `()` return type, there is no way to propagate a flush failure to the caller from inside it — if flushing fails during `drop()`, the best you can typically do is log the error (or, in truly critical cases, panic — accepting the abort-on-double-panic risk from Q10) and move on; the error is otherwise silently lost. This is exactly why types like `BufWriter` document that callers should call an explicit `.flush()` (which *does* return `io::Result<()>`) before the value is dropped, rather than relying on the implicit drop to handle a failure path correctly — the implicit drop exists as a safety net for panics/early-returns, not as the primary error-handling path. Errors inside `drop()` are never auto-propagated to any enclosing function — the calling code has no visibility into `drop()`'s internals at all. And RAII types absolutely can and do wrap fallible resources (files, sockets, locks) — that's a huge fraction of `Drop`'s real-world use; the pattern isn't disqualified by fallibility, it just requires this explicit-close convention to handle errors properly.

</details>
::

::question-wrapper{language="rust"}
### Q18. In a visitor implemented as a trait (`trait Visitor { fn visit_num(&mut self, n: f64); fn visit_add(&mut self, l: &Expr, r: &Expr); }`), what is the best-practice way to allow visitors to skip most node types without writing every method?

::code-wrapper{language="rust"}
```rust
trait Visitor {
    fn visit_num(&mut self, n: f64) {}
    fn visit_add(&mut self, l: &Expr, r: &Expr) {}
    fn visit_mul(&mut self, l: &Expr, r: &Expr) {}
}
```
::

- [ ] Give each method a default (often empty or auto-recursing) implementation in the trait definition, so implementors only override the specific `visit_*` methods they actually care about
- [ ] Make the trait's methods all `unsafe` so unimplemented ones default to undefined behavior instead of a compile error
- [ ] There is no way to make methods optional; every implementor must provide every method or fail to compile
- [ ] Use `#[derive(Visitor)]` from the standard library to auto-generate no-op defaults

<details>
<summary>Show Answer</summary>

**Answer:** A — give trait methods default implementations so implementors override only what they need

**Explanation:** **Idiom:** Rust traits support default method bodies directly in the trait definition; any implementor can simply omit a method to inherit the default (commonly a no-op, or one that recurses into child nodes to keep traversal working), which is precisely how ecosystem visitor traits (e.g. `syn::visit::Visit`) let users override only the handful of node types they care about instead of exhaustively implementing dozens of `visit_*` methods. Making methods `unsafe` has nothing to do with optionality and would just require callers to wrap every call in an `unsafe` block for no benefit — it doesn't create "default to UB," which isn't a real Rust mechanism. There is no `#[derive(Visitor)]` in the standard library; default method bodies in the trait itself are the actual, standard mechanism, and without them every method genuinely would be mandatory, making the "no way" option only true in the absence of defaults, which is exactly what this pattern adds.

</details>
::

::question-wrapper{language="rust"}
### Q19. A codebase has many newtypes like `UserId(u64)`, `OrderId(u64)`, `ProductId(u64)`. What best practice most directly prevents accidentally passing an `OrderId` where a `UserId` is expected, beyond just wrapping in a struct?

::code-wrapper{language="rust"}
```rust
struct UserId(u64);
struct OrderId(u64);

fn get_user(id: UserId) -> User { /* ... */ }
```
::

- [ ] Nothing extra is needed beyond the newtype wrapper itself and using it consistently in function signatures — `get_user(order_id)` where `order_id: OrderId` is already a compile error because `OrderId` and `UserId` are distinct, non-interchangeable types
- [ ] Add a runtime assertion inside `get_user` that checks a "type tag" field to catch mismatches
- [ ] Use `u64` directly everywhere and rely on descriptive parameter names like `user_id: u64` for safety
- [ ] Implement `From<OrderId> for UserId` so the two remain freely convertible for flexibility

<details>
<summary>Show Answer</summary>

**Answer:** A — nothing extra needed; distinct newtypes are already non-interchangeable at compile time when used consistently in signatures

**Explanation:** **Safety:** This is the newtype pattern's core payoff restated concretely: once `UserId` and `OrderId` are separate structs, `fn get_user(id: UserId)` simply cannot accept an `OrderId` argument — the compiler rejects `get_user(order_id)` as a type mismatch with zero runtime cost or extra ceremony, as long as the signatures consistently use the specific ID types rather than degrading back to raw `u64` anywhere in the call chain. A runtime "type tag" check is redundant *and* strictly worse (it defers a catchable compile-time bug to a runtime check that could be skipped or forgotten) given the compiler already enforces this for free. Using bare `u64` everywhere and relying on naming conventions is exactly the unsafe baseline this pattern replaces — parameter names provide no compiler enforcement at all, and a caller passing `user_id` where `order_id_as_u64` was expected compiles silently. And implementing `From<OrderId> for UserId` would be actively counterproductive — it reintroduces easy, silent interchangeability (via `.into()`) between two IDs that should never be conflated, undermining the entire point of separating them.

</details>
::

::question-wrapper{language="rust"}
### Q20. Which of these is a legitimate best-practice reason to prefer the typestate pattern over runtime state validation (an enum field checked with `if`/`match` inside each method) for a resource with a strict lifecycle (e.g. a database transaction: `Begun -> Committed`/`RolledBack`)?

::code-wrapper{language="rust"}
```rust
fn commit(&mut self) -> Result<(), TxError> {
    if self.state != TxState::Begun {
        return Err(TxError::InvalidState);
    }
    // ...
}
```
::

- [ ] Typestate turns "call `commit()` on an already-committed transaction" from a runtime error path (that must be tested and can be forgotten) into something the compiler refuses to build in the first place, eliminating an entire category of production incident
- [ ] Typestate is always faster at runtime because it eliminates one `if` statement, which dominates performance in most programs
- [ ] Runtime state validation is impossible to implement correctly in Rust, so typestate is the only valid option
- [ ] Typestate and runtime validation are functionally identical; the only difference is which one the linter prefers

<details>
<summary>Show Answer</summary>

**Answer:** A — typestate eliminates the invalid-call category at compile time instead of relying on a tested-and-hopefully-not-forgotten runtime check

**Explanation:** **Safety/Idiom:** The runtime-`if` version shown is a perfectly valid, common Rust pattern, but it has a fundamental weakness the question highlights: its correctness depends on every method remembering to check state and on every caller correctly handling the `Err(TxError::InvalidState)` path — a forgotten check, or a caller that unwraps and ignores the error, becomes a real production bug (e.g. double-committing a transaction). Typestate instead makes `Committed<Tx>::commit()` not exist as a callable method at all, so the mistake can't compile, full stop — no test coverage, code review vigilance, or runtime error handling is needed to catch it, because there is no code path where it's possible. This isn't primarily a performance optimization (the eliminated `if` is typically negligible against actual I/O costs like a database round-trip, making that option's performance framing misleading) and it isn't that runtime validation is "impossible to implement correctly" (the example compiles and works) — it's a strictly weaker safety guarantee, not a broken one. The two approaches are absolutely not functionally identical: one defers the invalid-state class of bug to runtime, the other removes it from the possibility space entirely.

</details>
::
