# 11 — Structs

::question-wrapper{language="rust"}
### Q1. Given `struct User { name: String, age: u8, active: bool }`, what does struct update syntax do here?

::code-wrapper{language="rust"}
```rust
let base = User { name: String::from("Alice"), age: 30, active: true };
let updated = User { age: 31, ..base };
```
::

- [ ] It mutates `base` in place, changing only `age`
- [ ] It creates a new `User`, copying/moving every field not explicitly listed from `base`, and `age` overrides `base.age`
- [ ] It merges `base` and the new fields into a `HashMap`
- [ ] It fails to compile because `..` requires all fields to be listed

<details>
<summary>Show Answer</summary>

**Answer:** B — It creates a new `User`, copying/moving every field not explicitly listed from `base`, and `age` overrides `base.age`

**Explanation:** `..base` fills in every remaining field from `base`, field-by-field, as if you'd written `name: base.name, active: base.active` yourself. It does not mutate `base` — it constructs an entirely new value. A is wrong because `base` is left untouched (aside from any fields moved out of it, see later questions). C is a fabricated behavior; struct update syntax has nothing to do with `HashMap`. D is wrong — `..base` is exactly the mechanism that lets you *avoid* listing every field.

</details>
::

::question-wrapper{language="rust"}
### Q2. What kind of struct is this, and how do you construct one?

::code-wrapper{language="rust"}
```rust
struct Point(f64, f64);
```
::

- [ ] A unit struct; construct with `Point;`
- [ ] A tuple struct; construct with `Point(1.0, 2.0)`, and access fields with `.0` and `.1`
- [ ] An enum variant; construct with `Point::new(1.0, 2.0)`
- [ ] Invalid syntax — structs require named fields

<details>
<summary>Show Answer</summary>

**Answer:** B — A tuple struct; construct with `Point(1.0, 2.0)`, and access fields with `.0` and `.1`

**Explanation:** Tuple structs have a name but unnamed, positionally-indexed fields, giving you a distinct type (unlike a plain `(f64, f64)` tuple) while keeping tuple-like ergonomics. A confuses this with a unit struct (`struct Marker;`, zero fields). C is false — this is not enum syntax at all. D is wrong — tuple structs with unnamed positional fields are valid, standard Rust.

</details>
::

::question-wrapper{language="rust"}
### Q3. What is a "unit struct," and what is it typically used for?

::code-wrapper{language="rust"}
```rust
struct Meters;
```
::

- [ ] A struct with exactly one field
- [ ] A zero-sized struct with no fields at all, commonly used as a marker type or to implement a trait with no data
- [ ] A struct that can only hold `u8` values
- [ ] Shorthand for `struct Meters { value: () }`

<details>
<summary>Show Answer</summary>

**Answer:** B — A zero-sized struct with no fields at all, commonly used as a marker type or to implement a trait with no data

**Explanation:** `struct Meters;` declares a type with zero size and zero fields — it exists purely at the type level, useful for marker types, phantom-type tags, or as a target for trait implementations where behavior matters but data doesn't. A confuses it with a newtype/tuple struct holding one value. C is fabricated. D is close conceptually but wrong mechanically — a unit struct has no field at all, not an explicit `()` field; `size_of::<Meters>()` is `0`, same as `size_of::<()>()`, but they are different types.

</details>
::

::question-wrapper{language="rust"}
### Q4. What's the essential difference between these two `impl` block items?

::code-wrapper{language="rust"}
```rust
impl Rectangle {
    fn area(&self) -> u32 { self.width * self.height }
    fn square(size: u32) -> Rectangle { Rectangle { width: size, height: size } }
}
```
::

- [ ] `area` is an associated function; `square` is a method
- [ ] `area` is a method (takes `self`, called as `rect.area()`); `square` is an associated function (no `self`, called as `Rectangle::square(5)`)
- [ ] Both are methods, just with different return types
- [ ] `square` cannot be called without first creating a `Rectangle` instance

<details>
<summary>Show Answer</summary>

**Answer:** B — `area` is a method (takes `self`, called as `rect.area()`); `square` is an associated function (no `self`, called as `Rectangle::square(5)`)

**Explanation:** The presence or absence of a `self` parameter is exactly what distinguishes a method from an associated function in Rust terminology. `Rectangle::new`-style constructors are the classic associated-function use case, since you don't yet have an instance to call a method on. A has the terms backwards. C ignores the defining distinction. D is backwards — associated functions are exactly the tool for building the *first* instance, callable without any existing value.

</details>
::

::question-wrapper
### Q5. Which method receiver should `fn describe(&self) -> String` use if it only needs to read fields and return a new owned `String`?

- [ ] `self` (take ownership)
- [ ] `&mut self`
- [ ] `&self`
- [ ] `&&self`

<details>
<summary>Show Answer</summary>

**Answer:** C — `&self`

**Explanation:** Reading fields without needing to mutate or consume the struct calls for an immutable borrow, `&self` — this is by far the most common receiver, and it lets the caller keep using the instance afterward. `self` (A) would consume the instance, making it unusable after the call, which is unnecessary overhead and an ergonomics regression if the caller wanted to keep the value. `&mut self` (B) would needlessly require the caller to have (or make) a mutable binding when no mutation happens. `&&self` (D) is not valid receiver syntax.

</details>
::

::question-wrapper{language="rust"}
### Q6. What does calling a method that takes `self` (by value, not `&self`) do to the original variable?

::code-wrapper{language="rust"}
```rust
struct Wrapper(String);
impl Wrapper {
    fn into_inner(self) -> String { self.0 }
}
let w = Wrapper(String::from("data"));
let s = w.into_inner();
```
::

- [ ] `w` is still usable after the call, since Rust copies structs implicitly
- [ ] `w` is moved into `into_inner`, so using `w` again after this line is a compile error
- [ ] `w` is automatically cloned before the call
- [ ] This only compiles if `Wrapper` derives `Copy`

<details>
<summary>Show Answer</summary>

**Answer:** B — `w` is moved into `into_inner`, so using `w` again after this line is a compile error

**Explanation:** A `self`-by-value receiver takes ownership, exactly like passing `w` to any function that takes `Wrapper` by value — `Wrapper` contains a `String`, which is not `Copy`, so this is a move, not a copy. Any later use of `w` triggers "use of moved value." A is the common mistaken assumption from languages with reference semantics by default. C is false — Rust never inserts implicit clones. D is backwards — if `Wrapper` *did* derive `Copy`, `w` would still be usable afterward because the value gets copied instead of moved, but that's not the case here since it contains a non-`Copy` `String`.

</details>
::

::question-wrapper
### Q7. What visibility do struct fields have by default, inside their own module, if declared without a `pub` keyword?

- [ ] Public everywhere by default
- [ ] Private — accessible only within the module the struct is defined in (and its descendants)
- [ ] Public within the same crate only, regardless of module
- [ ] Fields have no visibility concept; only whole structs can be public or private

<details>
<summary>Show Answer</summary>

**Answer:** B — Private — accessible only within the module the struct is defined in (and its descendants)

**Explanation:** Struct fields are private by default, even if the struct itself is `pub` — this is a frequent surprise for newcomers who mark `pub struct Foo` and then can't access `foo.bar` from another module until they also add `pub` to the field itself. A is wrong and is the exact misconception this trips people on. C invents a crate-wide default that doesn't exist. D is false — Rust does apply per-field visibility, independent of the struct's own visibility.

</details>
::

::question-wrapper{language="rust"}
### Q8. What happens when you use struct update syntax where the base struct contains a non-`Copy` field that you *don't* override?

::code-wrapper{language="rust"}
```rust
struct Config { name: String, retries: u32 }
let base = Config { name: String::from("svc"), retries: 3 };
let updated = Config { retries: 5, ..base };
println!("{}", base.name);
```
::

- [ ] Compiles fine — `base.name` is still accessible because `..base` only borrows
- [ ] Fails to compile — `name` is moved out of `base` into `updated`, so `base` (or at least `base.name`) can no longer be used as a whole
- [ ] `base.name` is silently cloned automatically
- [ ] Only `retries` fails, since `u32` isn't `Copy`

<details>
<summary>Show Answer</summary>

**Answer:** B — Fails to compile — `name` is moved out of `base` into `updated`, so `base` (or at least `base.name`) can no longer be used as a whole

**Explanation:** `..base` moves every non-overridden, non-`Copy` field out of `base` field-by-field. Since `name` (a `String`) isn't `Copy`, it's moved into `updated.name`, leaving `base` partially moved — the compiler then rejects `base.name` (and any whole-`base` use) with "value borrowed here after partial move" / "use of moved value." **Idiom:** the fix is to `.clone()` the field explicitly if you need both, e.g. `name: base.name.clone(), ..base`. Note `retries` is `u32` (`Copy`), so overriding it doesn't move anything — but that's irrelevant since it was overridden, not taken from `base`, and D wrongly claims only `retries` is the problem when the real issue is `name`. A and C both falsely assume Rust performs implicit borrowing/cloning here.

</details>
::

::question-wrapper
### Q9. Calling `Rectangle::square(0)` on a tuple-free `square` associated function that builds `Rectangle { width: size, height: size }` — what happens with `size = 0`?

- [ ] Compile error — struct fields cannot be zero
- [ ] Panics at runtime because zero-area rectangles are invalid
- [ ] Compiles and runs fine, producing a `Rectangle` with `width: 0, height: 0`; nothing about the type prevents degenerate values
- [ ] Returns `None` since the rectangle would be empty

<details>
<summary>Show Answer</summary>

**Answer:** C — Compiles and runs fine, producing a `Rectangle` with `width: 0, height: 0`; nothing about the type prevents degenerate values

**Explanation:** Plain struct fields with primitive types like `u32` accept any value in their range, including `0` — the struct itself enforces no domain invariant unless you add validation logic (e.g., a fallible constructor returning `Result`, or a newtype with private fields and a checked constructor). This is an edge case worth internalizing: structs alone don't guarantee "sensible" values. A, B, and D all invent validation that plain struct construction does not perform.

</details>
::

::question-wrapper{language="rust"}
### Q10. What's the pitfall in this destructuring pattern using struct update syntax combined with a method call?

::code-wrapper{language="rust"}
```rust
struct Session { token: String, expires_in: u32 }
impl Session {
    fn refreshed(self, new_token: String) -> Session {
        Session { token: new_token, ..self }
    }
}
```
::

- [ ] This never compiles — `..self` cannot be used inside a method
- [ ] `self` is consumed by value here, so `refreshed` can only be called once per `Session` instance (each call requires ownership, matching the "produce a new session, discard the old" intent)
- [ ] `expires_in` is silently reset to `0`
- [ ] `new_token` must implement `Copy`

<details>
<summary>Show Answer</summary>

**Answer:** B — `self` is consumed by value here, so `refreshed` can only be called once per `Session` instance (each call requires ownership, matching the "produce a new session, discard the old" intent)

**Explanation:** Taking `self` by value is a deliberate, idiomatic choice for "transform and replace" APIs — it deliberately prevents the caller from accidentally reusing the stale `Session` after refreshing, since the old value is moved into the method and never returned. `..self` inside the struct literal simply copies/moves `self`'s remaining fields (`expires_in`) into the new instance, exactly like the earlier update-syntax examples, just now with `self` as the base. A is false — `..self` is valid anywhere a struct literal appears, including method bodies. C is fabricated; `expires_in` carries over unchanged. D is false — `new_token` is being moved in directly as a field value, not through update syntax, and `String` need not be `Copy` for that.

</details>
::

::question-wrapper
### Q11. Two struct types happen to have identical field names and types: `struct Meters(f64)` and `struct Feet(f64)`. What happens if you try `let m: Meters = Feet(3.0);`?

- [ ] Compiles fine — both wrap an `f64`, so they're structurally interchangeable
- [ ] Compile error — Rust's type system is nominal, not structural; `Meters` and `Feet` are distinct types even with identical layout
- [ ] Compiles, but `m.0` will be `0.0`
- [ ] Only fails at runtime with a type-mismatch panic

<details>
<summary>Show Answer</summary>

**Answer:** B — Compile error — Rust's type system is nominal, not structural; `Meters` and `Feet` are distinct types even with identical layout

**Explanation:** This is precisely the value of the newtype pattern: even though `Meters` and `Feet` are both single-field tuple structs wrapping `f64` with identical memory layout, Rust treats them as unrelated types by name/declaration, not by shape. **Safety:** this is what prevents unit-confusion bugs (e.g., mixing up meters and feet) at compile time rather than in production. A describes structural typing, which languages like TypeScript use but Rust does not. C and D invent runtime behaviors that don't apply — this is caught at compile time, full stop, with "mismatched types."

</details>
::

::question-wrapper{language="rust"}
### Q12. What happens when a struct's field is itself a reference and the struct's owner tries to outlive the borrowed data?

::code-wrapper{language="rust"}
struct Highlight<'a> { text: &'a str }
```
::

- [ ] The struct owns a copy of the text, so no lifetime issue exists
- [ ] Nothing shown here fails by itself, but any attempt to construct a `Highlight` whose `text` outlives the source string is rejected at compile time — the struct can never outlive the data it borrows
- [ ] This struct definition itself fails to compile
- [ ] `'a` defaults to `'static` automatically

<details>
<summary>Show Answer</summary>

**Answer:** B — Nothing shown here fails by itself, but any attempt to construct a `Highlight` whose `text` outlives the source string is rejected at compile time — the struct can never outlive the data it borrows

**Explanation:** The struct definition itself is valid Rust (tying the struct's lifetime parameter to the borrowed field), but any use site that tries to keep a `Highlight` alive longer than the `&str` it points into will be rejected — this is the struct-holds-a-reference case that ties directly back to explicit lifetime annotations. A is wrong — `&'a str` is a borrow, not an owned copy. C is wrong — the definition compiles fine on its own. D is false — lifetimes on struct fields are never silently defaulted to `'static`; they must be satisfied by whatever data is actually borrowed.

</details>
::

::question-wrapper{language="rust"}
### Q13. `#[derive(Debug)]` is added to a struct containing a field of a type that doesn't implement `Debug`. What happens?

::code-wrapper{language="rust"}
```rust
struct RawSocket(*mut u8);
#[derive(Debug)]
struct Connection {
    id: u32,
    socket: RawSocket,
}
```
::

- [ ] It compiles; `derive(Debug)` silently skips fields that don't implement `Debug`
- [ ] It fails to compile — `derive(Debug)` requires every field's type to also implement `Debug`, and `RawSocket` doesn't
- [ ] It compiles but panics the first time `{:?}` is used on a `Connection`
- [ ] Raw pointers always implement `Debug`, so this compiles without issue

<details>
<summary>Show Answer</summary>

**Answer:** D — Raw pointers always implement `Debug`, so this compiles without issue

**Explanation:** This is a genuine gotcha: `*mut u8` (and other raw pointer types) *do* implement `Debug` in the standard library, printing as a hex address — so `RawSocket`, despite looking like it wraps something exotic, actually satisfies the derive requirement here, and the whole thing compiles fine. The broader rule that trips people up is real (B describes the general case correctly for types that truly lack `Debug`, such as function-pointer-heavy or certain FFI-opaque types), but it doesn't apply to this specific example because raw pointers are one of the types that do implement `Debug`. A is false — `derive` never silently skips fields; it fails the whole derive if any field is missing a required impl. C invents a runtime failure where the real failure (when it does happen) is a compile error, not a panic.

</details>
::

::question-wrapper
### Q14. What does `Rectangle { width: 10, ..Default::default() }` require in order to compile?

- [ ] Nothing extra — `Default::default()` always works for any struct
- [ ] `Rectangle` must implement (or derive) the `Default` trait, since `..Default::default()` calls `Rectangle::default()` to supply the other fields
- [ ] `width` must be removed from the struct entirely
- [ ] This syntax is only valid for tuple structs

<details>
<summary>Show Answer</summary>

**Answer:** B — `Rectangle` must implement (or derive) the `Default` trait, since `..Default::default()` calls `Rectangle::default()` to supply the other fields

**Explanation:** Type inference figures out that `Default::default()` here must produce a `Rectangle` (because it's used as the base in a `Rectangle { .. }` literal), then calls `Rectangle::default()`. If `Rectangle` doesn't implement `Default` (via `#[derive(Default)]` or a manual `impl`), this fails to compile with a trait-bound error, not silently falling back to zeroed memory. A is wrong for exactly that reason. C misunderstands the syntax — `width` stays as an explicit override. D is false — struct update syntax works with named-field structs like this example, not just tuple structs.

</details>
::

::question-wrapper
### Q15. When designing a public API, why is it generally best practice to keep struct fields **private** and expose associated-function constructors plus accessor methods, rather than making all fields `pub`?

- [ ] Private fields are faster at runtime than public ones
- [ ] Private fields let you enforce invariants at construction/mutation time and change the internal representation later without breaking downstream code
- [ ] Rust requires private fields for `#[derive(Debug)]` to work
- [ ] Public fields are not allowed on structs that implement any trait

<details>
<summary>Show Answer</summary>

**Answer:** B — Private fields let you enforce invariants at construction/mutation time and change the internal representation later without breaking downstream code

**Explanation:** **Idiom:** this is the standard "encapsulation" argument applied to Rust — a `pub` field can be set to any value satisfying its type from anywhere, bypassing any validation logic, and it locks the struct's internal layout into your public API forever (a semver-breaking change to alter later). Exposing a constructor (`Config::new(...)` returning `Result` if validation can fail) and getters/setters keeps the door open for future changes. A is false — there's no runtime cost difference between public and private field access. C and D are fabricated constraints.

</details>
::

::question-wrapper{language="rust"}
### Q16. A method needs to mutate one field of a struct but the struct is large and cloning it would be wasteful. Which receiver is idiomatic?

::code-wrapper{language="rust"}
```rust
struct Cache { entries: Vec<String>, hits: u64 }
```
::

- [ ] `fn record_hit(self)` — take by value, mutate, then the caller must reassign the return value
- [ ] `fn record_hit(&mut self)` — mutate the field directly through a mutable borrow
- [ ] `fn record_hit(&self)` — using interior mutability is always the right default even without needing it
- [ ] Clone the struct, mutate the clone, and return it, leaving the original untouched

<details>
<summary>Show Answer</summary>

**Answer:** B — `fn record_hit(&mut self)` — mutate the field directly through a mutable borrow

**Explanation:** **Idiom:** `&mut self` is exactly the tool for "mutate in place without giving up ownership or reallocating" — it's the standard, zero-cost choice for methods like `push`, `insert`, or counters like `hits += 1`. A works but forces every caller into an awkward `cache = cache.record_hit();` pattern for no benefit when in-place mutation is possible. C is a common overcorrection — reaching for `RefCell`/interior mutability by default adds runtime borrow-checking overhead and complexity that plain `&mut self` avoids when you already have exclusive access. D wastes an entire clone of `entries: Vec<String>` just to increment a counter — needless allocation and copying.

</details>
::

::question-wrapper
### Q17. Reviewing a PR, you see a builder struct where every setter method consumes and returns `self` by value: `fn with_retries(mut self, n: u32) -> Self { self.retries = n; self }`. Why is this the idiomatic pattern for builders, rather than using `&mut self` returning `&mut Self`?

- [ ] It isn't idiomatic — `&mut self` chaining is strictly better and should be preferred
- [ ] Taking `self` by value enables fluent, chainable one-liner construction (`Builder::new().with_retries(3).with_timeout(5).build()`) that can be built as a single expression, including from a function's tail-return position, without needing an intermediate `let mut` binding
- [ ] `self`-by-value is required because `Self` cannot appear as a return type otherwise
- [ ] There's no difference between the two approaches in any circumstance

<details>
<summary>Show Answer</summary>

**Answer:** B — Taking `self` by value enables fluent, chainable one-liner construction (`Builder::new().with_retries(3).with_timeout(5).build()`) that can be built as a single expression, including from a function's tail-return position, without needing an intermediate `let mut` binding

**Explanation:** **Idiom:** the by-value builder pattern is idiomatic specifically because it composes into a single expression — useful in contexts like a function's implicit return (`fn make() -> Config { Config::builder().with_retries(3).build() }`) where you can't easily have a `let mut b = ...;` statement. The `&mut self` -> `&mut Self` chaining style also works and avoids repeated moves, but requires an owned `let mut` binding up front and doesn't work as cleanly in expression position, which is why by-value chaining is the more commonly seen idiom for builders. A overstates it as strictly better in all cases — both are valid, with different tradeoffs. C is a fabricated restriction; `Self` return types work fine with any receiver. D dismisses a real, meaningful ergonomic difference.

</details>
::

::question-wrapper
### Q18. What's the best-practice reason to prefer an associated function like `Point::origin() -> Point` over requiring callers to write `Point { x: 0.0, y: 0.0 }` directly everywhere?

- [ ] `Point { x: 0.0, y: 0.0 }` is not valid syntax
- [ ] It centralizes the "what does a default/special-case value look like" decision in one place, and keeps working even if you later make the fields private or add new fields
- [ ] Associated functions run faster than struct literals
- [ ] It's required for the struct to implement `Copy`

<details>
<summary>Show Answer</summary>

**Answer:** B — It centralizes the "what does a default/special-case value look like" decision in one place, and keeps working even if you later make the fields private or add new fields

**Explanation:** **Idiom:** this is the same encapsulation argument as constructors generally — if `Point` gains a third field `z: f64` later, every call site that manually wrote `Point { x: 0.0, y: 0.0 }` breaks, but a single `Point::origin()` definition only needs updating once. It also keeps working if fields become private. A is false — direct struct-literal syntax works fine when fields are public. C is a fabricated performance claim; both compile to equivalent code. D is unrelated — `Copy` is an independent derive, unaffected by whether you use a literal or a constructor function.

</details>
::

::question-wrapper
### Q19. In review, someone suggests changing `fn total(&self) -> f64` (which sums two fields) to instead take `self` by value, arguing "it's simpler." Why would that be a worse choice for a getter-style method used inside a loop like `for item in &items { sum += item.total(); }`?

- [ ] It would fail to compile in a `for` loop
- [ ] Taking `self` by value would move `item` out of the collection on the first iteration (or, since `item` here is `&Item`, force an extra deref/clone), which is unnecessary overhead and friction for a method that never needs to consume the value
- [ ] `f64` cannot be returned from a by-value method
- [ ] There is no difference — both compile to identical machine code

<details>
<summary>Show Answer</summary>

**Answer:** B — Taking `self` by value would move `item` out of the collection on the first iteration (or, since `item` here is `&Item`, force an extra deref/clone), which is unnecessary overhead and friction for a method that never needs to consume the value

**Explanation:** **Idiom:** the general principle is: only take `self` by value when the method genuinely needs ownership (e.g., transforming into a different type, or a builder's terminal `build()`). A pure read like summing fields for a total should use `&self`, so it can be called repeatedly on borrowed items (as in `for item in &items`) without consuming anything. Switching to by-value `self` would, at minimum, require cloning `Item` at each call site to satisfy ownership, adding needless allocation/copy overhead in a hot loop. A is false — it can be made to compile (with clones), it's just wasteful. C is a fabricated restriction on return types. D dismisses a real, meaningful runtime cost difference.

</details>
::

::question-wrapper
### Q20. A struct has grown to 8 fields, and a refactor wants to add a 9th while preserving all existing call sites that construct it with a full field list (no `..` update syntax). What's the best-practice tradeoff to flag in review?

- [ ] Nothing to flag — adding a field to a struct is always backward compatible
- [ ] Adding a field is a breaking change for any code using exhaustive struct-literal construction (every call site must add the new field, or the struct should switch to a builder/constructor pattern, or mark itself `#[non_exhaustive]` for external consumers)
- [ ] Fields can only be added if the struct implements `Default`
- [ ] This requires bumping the struct to a tuple struct instead

<details>
<summary>Show Answer</summary>

**Answer:** B — Adding a field is a breaking change for any code using exhaustive struct-literal construction (every call site must add the new field, or the struct should switch to a builder/constructor pattern, or mark itself `#[non_exhaustive]` for external consumers)

**Explanation:** **Idiom:** unlike adding a method (which is always additive and non-breaking), adding a field to a struct that callers construct with `StructName { a, b, c, ... }` literal syntax breaks every one of those call sites unless they use `..Default::default()` or `..base` update syntax to fill in the gap. This is exactly why widely-consumed public structs either expose a constructor/builder instead of public fields, or are annotated `#[non_exhaustive]` so external crates are forced to use a constructor and can't exhaustively list fields at all. A is the naive assumption that bites maintainers of public crates. C invents an unrelated requirement — `Default` doesn't automatically shield existing literals. D is a non-sequitur; switching to a tuple struct doesn't address the exhaustive-construction problem and loses named-field readability.

</details>
::
