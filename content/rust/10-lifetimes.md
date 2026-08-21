# 10 — Lifetimes

::question-wrapper
### Q1. What does the signature `fn longest<'a>(x: &'a str, y: &'a str) -> &'a str` actually promise to the caller?

- [ ] That `x` and `y` live exactly as long as each other
- [ ] That the returned reference is valid for at least as long as the shorter of the two input borrows overlap
- [ ] That the function allocates a new string with a `'static` lifetime
- [ ] That `x` and `y` must be declared in the same scope

<details>
<summary>Show Answer</summary>

**Answer:** B — That the returned reference is valid for at least as long as the shorter of the two input borrows overlap

**Explanation:** `'a` is not a concrete duration; it's a constraint the compiler solves for at each call site — it becomes the intersection (overlap) of however long `x` and `y` are actually borrowed for. The returned reference is only guaranteed valid within that overlap. A is wrong because `x` and `y` can have different concrete lifetimes; the annotation just forces the compiler to pick the smaller one for `'a`. C is wrong because lifetime parameters describe borrows of existing data, not allocation. D is wrong — lifetimes are about how long borrows are valid, not lexical co-location.

</details>
::

::question-wrapper
### Q2. Given `fn first_word(s: &str) -> &str`, why does this compile without any explicit lifetime annotations?

- [ ] The function doesn't return a reference, so no lifetime is needed
- [ ] Lifetime elision rule: a single input reference's lifetime is automatically assigned to all elided output lifetimes
- [ ] The Rust compiler infers lifetimes only for `&str`, never for `&T`
- [ ] Because `s` is immutable, lifetimes are not checked

<details>
<summary>Show Answer</summary>

**Answer:** B — Lifetime elision rule: a single input reference's lifetime is automatically assigned to all elided output lifetimes

**Explanation:** This is desugared by the compiler to `fn first_word<'a>(s: &'a str) -> &'a str`. The elision rules exist so common patterns don't require boilerplate. A is false — the return type is clearly `&str`, a reference. C is false — elision applies uniformly to any `&T`, not just `&str`. D is false — mutability is irrelevant to lifetime elision; the rule is purely about counting input reference parameters.

</details>
::

::question-wrapper
### Q3. Why does `fn combine(x: &str, y: &str) -> &str` fail to compile as written?

- [ ] It doesn't — this compiles fine using elision
- [ ] There are two input lifetimes and no `&self`, so elision cannot determine which one the output should borrow from
- [ ] `&str` return types always require `'static`
- [ ] Rust forbids functions with two reference parameters

<details>
<summary>Show Answer</summary>

**Answer:** B — There are two input lifetimes and no `&self`, so elision cannot determine which one the output should borrow from

**Explanation:** Elision rule 2 (single input lifetime applied to output) only fires when there is exactly one input reference. Rule 3 (use `&self`'s lifetime) only fires for methods. With two unrelated reference parameters and neither rule applicable, the compiler refuses to guess and demands an explicit `fn combine<'a>(x: &'a str, y: &'a str) -> &'a str` (or distinct lifetimes if only one feeds the output). A is wrong — this genuinely fails with "missing lifetime specifier." C and D are fabricated restrictions that don't exist in Rust.

</details>
::

::question-wrapper
### Q4. In `impl<'a> Parser<'a> { fn peek(&self, other: &str) -> &str { ... } }`, which lifetime does the returned `&str` borrow from, per elision rule 3?

- [ ] `other`'s lifetime
- [ ] `&self`'s lifetime
- [ ] A fresh anonymous lifetime unrelated to either parameter
- [ ] This is ambiguous and fails to compile

<details>
<summary>Show Answer</summary>

**Answer:** B — `&self`'s lifetime

**Explanation:** Elision rule 3 states that when a method has multiple input lifetimes but one of them is `&self` or `&mut self`, the lifetime of `self` is assigned to all elided output lifetimes — this matches the common case where a method returns a borrow of its own fields. A is the tempting-but-wrong guess many make when reading `other` as "the last parameter, so it must apply." C and D ignore that rule 3 exists specifically to resolve this case without an error.

</details>
::

::question-wrapper{language="rust"}
### Q5. Why does the following fail to compile?

::code-wrapper{language="rust"}
```rust
fn make_owner() -> &str {
    let s = String::from("temp");
    &s[..]
}
```
::

- [ ] `String::from` cannot be indexed with a range
- [ ] `s` is dropped at the end of the function, so the returned reference would dangle
- [ ] The function is missing a `mut` keyword
- [ ] `&str` cannot be constructed from a `String`

<details>
<summary>Show Answer</summary>

**Answer:** B — `s` is dropped at the end of the function, so the returned reference would dangle

**Explanation:** `s` is a local `String` owned by the function's stack frame; once the function returns, `s` is deallocated. Returning `&s[..]` would produce a reference to freed memory, so the borrow checker rejects it with "cannot return reference to local variable." **Safety:** this is exactly the class of bug lifetimes exist to prevent at compile time instead of at runtime as a use-after-free. The fix is to return an owned `String` (`fn make_owner() -> String`). A and D are false — both operations are valid Rust; C is irrelevant, mutability doesn't affect ownership/drop timing.

</details>
::

::question-wrapper
### Q6. What does the `'static` lifetime bound on a reference, as in `x: &'static str`, guarantee?

- [ ] The value is heap-allocated
- [ ] The reference's data is valid for the entire remainder of the program's execution
- [ ] The value cannot be a string literal
- [ ] The variable is thread-local

<details>
<summary>Show Answer</summary>

**Answer:** B — The reference's data is valid for the entire remainder of the program's execution

**Explanation:** `'static` means the borrow does not need to be dropped before the program ends — the referenced data outlives everything else. String literals are the canonical example because they're baked into the binary's read-only data section. A is wrong — `'static` says nothing about heap vs. stack vs. static memory, only about duration. C is backwards — literals are the most common source of `'static` references. D is unrelated — `'static` has nothing to do with threading.

</details>
::

::question-wrapper{language="rust"}
### Q7. Why must this struct definition include an explicit lifetime parameter?

::code-wrapper{language="rust"}
```rust
struct Excerpt<'a> {
    part: &'a str,
}
```
::

- [ ] All structs in Rust require at least one generic parameter
- [ ] A struct that holds a reference must declare how long that reference is valid for, tied to the struct's own lifetime
- [ ] `&str` fields are only allowed in enums, not structs
- [ ] It's optional style, not a compiler requirement

<details>
<summary>Show Answer</summary>

**Answer:** B — A struct that holds a reference must declare how long that reference is valid for, tied to the struct's own lifetime

**Explanation:** Whenever a struct stores a borrowed reference instead of owned data, the compiler needs to know the reference can't outlive the data it points to, and that no instance of `Excerpt` can outlive `'a`. Omitting the annotation is a hard compile error ("missing lifetime specifier"), not a style choice — D is wrong. A is a fabricated rule; plenty of structs have zero generic parameters. C is false — the restriction applies to any reference field, not specifically `&str` in enums.

</details>
::

::question-wrapper
### Q8. A junior developer claims: "If I add a `T: 'static` bound to a generic function, every value passed in must be created before `main()` starts, like a string literal." Is this accurate?

- [ ] Yes, `'static` bounds always require compile-time-constant data
- [ ] No — `T: 'static` just means `T` contains no borrowed references shorter than `'static`; an owned, heap-allocated `String` created at runtime satisfies it fine
- [ ] No — `T: 'static` means the value must be `Copy`
- [ ] Yes, but only for `String` and `Vec` types specifically

<details>
<summary>Show Answer</summary>

**Answer:** B — No — `T: 'static` just means `T` contains no borrowed references shorter than `'static`; an owned, heap-allocated `String` created at runtime satisfies it fine

**Explanation:** This is one of the most common Rust misconceptions. `T: 'static` does not mean "lives forever" or "known at compile time" — it means "if `T` contains any references, those references must be `'static`." Owned types like `String`, `Vec<u8>`, or `i32` trivially satisfy `T: 'static` because they own their data outright and contain no borrows at all, regardless of when they're created. **Idiom:** this bound shows up constantly on `thread::spawn` and `Box<dyn Trait>` precisely to rule out dangling borrows across thread/heap boundaries, not to force compile-time construction. A, C, and D invent restrictions that don't exist.

</details>
::

::question-wrapper{language="rust"}
### Q9. Consider this function under Non-Lexical Lifetimes (NLL). Why does it compile?

::code-wrapper{language="rust"}
```rust
fn main() {
    let mut v = vec![1, 2, 3];
    let first = &v[0];
    println!("{first}");
    v.push(4);
}
```
::

- [ ] It doesn't compile — `first` still borrows `v` when `push` is called
- [ ] NLL ends `first`'s borrow at its last use (the `println!`), so the mutable borrow in `push` doesn't conflict
- [ ] `v[0]` copies the value instead of borrowing, since `i32` is `Copy`
- [ ] `push` doesn't require a mutable borrow of `v`

<details>
<summary>Show Answer</summary>

**Answer:** B — NLL ends `first`'s borrow at its last use (the `println!`), so the mutable borrow in `push` doesn't conflict

**Explanation:** Before NLL (pre-2018 borrow checker), a reference's lifetime extended to the end of its lexical scope, so this would have failed. NLL changed the analysis to end a borrow's *lifetime* at its last actual use, which can be well before the closing brace — lifetime and scope are related but not identical. **Idiom:** this is exactly why "lifetime" and "scope" are distinct concepts; scope is lexical, lifetime is usage-based. A is the pre-NLL answer and a common outdated assumption. C is a red herring — indexing does borrow, `i32: Copy` doesn't change that `v[0]` on a `Vec` desugars through `Index`, though the copy happens right after the borrow, which is exactly why the borrow can end immediately. D is false — `Vec::push` requires `&mut self`.

</details>
::

::question-wrapper{language="rust"}
### Q10. What's wrong with this attempt at an early-return function?

::code-wrapper{language="rust"}
```rust
fn shortest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() < y.len() { x } else { y }
}

fn caller() -> &'static str {
    let long_lived = String::from("hello world");
    let result;
    {
        let short_lived = String::from("hi");
        result = shortest(long_lived.as_str(), short_lived.as_str());
    }
    result
}
```
::

- [ ] Nothing — this compiles and runs fine
- [ ] `result` may borrow `short_lived`, which is dropped at the end of the inner block, so using `result` afterward is a dangling-reference error
- [ ] `shortest` needs two separate lifetime parameters, one per argument
- [ ] `String::from` cannot be passed to a function expecting `&str`

<details>
<summary>Show Answer</summary>

**Answer:** B — `result` may borrow `short_lived`, which is dropped at the end of the inner block, so using `result` afterward is a dangling-reference error

**Explanation:** Because `shortest` ties both inputs to the *same* lifetime `'a`, the compiler must conservatively assume the returned reference could point into either argument. Since `short_lived` doesn't outlive the inner block, `'a` is constrained to that shorter scope, and using `result` after the block ends is rejected as "`short_lived` does not live long enough." **Safety:** the compiler can't know at compile time which branch of the `if` will run, so it must assume the worst case. The fix is to shrink the scope of `result`'s use to inside the block, or to have `shortest` return an owned `String`. C would actually make it worse — separate lifetimes would still require a return type tied to one of them, and the compiler still can't know statically which. D is false — `&String` coerces to `&str` via deref coercion.

</details>
::

::question-wrapper{language="rust"}
### Q11. Why does this `match`-based function fail to compile?

::code-wrapper{language="rust"}
```rust
fn pick<'a>(flag: bool, a: &'a str) -> &'a str {
    if flag {
        a
    } else {
        let local = String::from("fallback");
        &local
    }
}
```
::

- [ ] `if`/`else` branches must return the same literal value
- [ ] The `else` branch returns a reference to `local`, which is dropped when the branch scope ends — it cannot satisfy the `'a` bound tied to `a`
- [ ] `flag` must be `&bool`, not `bool`
- [ ] `String` cannot be shadowed with `let`

<details>
<summary>Show Answer</summary>

**Answer:** B — The `else` branch returns a reference to `local`, which is dropped when the branch scope ends — it cannot satisfy the `'a` bound tied to `a`

**Explanation:** The function signature promises a return value valid for `'a` (the lifetime of `a`), but `&local` is only valid for the tiny scope of the `else` block. The compiler flags "`local` does not live long enough" because no lifetime annotation can make a truly local value outlive its own creation. The correct fix is to return an owned `String` from both branches (changing the signature to `-> String`, cloning `a` in the `if` branch) rather than trying to force a shared borrow. A, C, and D describe non-existent restrictions.

</details>
::

::question-wrapper
### Q12. Which scenario genuinely requires a struct with **two independent** lifetime parameters, e.g. `struct Pair<'a, 'b>`, instead of one shared `'a`?

- [ ] Never — using a single shared lifetime is always equivalent and preferred
- [ ] When the two borrowed fields come from sources with different, unrelated lifetimes, and a method needs to return a reference tied to only one of them
- [ ] When one field is `&str` and the other is `&[u8]`
- [ ] When the struct also derives `Clone`

<details>
<summary>Show Answer</summary>

**Answer:** B — When the two borrowed fields come from sources with different, unrelated lifetimes, and a method needs to return a reference tied to only one of them

**Explanation:** Collapsing both fields to a single `'a` forces the compiler to use the shorter of the two lifetimes everywhere, which needlessly restricts callers when the fields are genuinely independent (e.g., one long-lived config string and one short-lived per-request buffer). Separate parameters let each field's borrow be tracked precisely. A is the tempting simplification that works until a caller hits an over-restrictive borrow error. C is false — the underlying type (`&str` vs `&[u8]`) has nothing to do with how many lifetime parameters are needed. D is unrelated to lifetimes at all.

</details>
::

::question-wrapper
### Q13. What does the bound `fn process<T: Debug + 'a>(item: T)` communicate about `T`?

- [ ] `T` must be a reference type
- [ ] Any references contained within `T` must live at least as long as `'a`
- [ ] `T` must implement the `Lifetime` trait
- [ ] `'a` is ignored because `T` is passed by value

<details>
<summary>Show Answer</summary>

**Answer:** B — Any references contained within `T` must live at least as long as `'a`

**Explanation:** A lifetime bound on a generic type parameter (`T: 'a`) constrains any borrows nested inside `T` — whether `T` itself is a reference or an owned type that happens to hold one (like `struct Wrapper<'a>(&'a i32)`). It does **not** require `T` itself to be a reference type, so A is wrong. C invents a nonexistent trait — lifetimes are not traits. D is wrong — even owned generic types can embed references, so the bound still matters regardless of pass-by-value.

</details>
::

::question-wrapper
### Q14. What is the lifetime of the string literal in `let s: &'static str = "hello";`, and why?

- [ ] It's `'static` because all `&str` variables default to `'static` unless annotated otherwise
- [ ] It's `'static` because the literal's bytes are embedded directly in the compiled binary's read-only memory, which exists for the whole program run
- [ ] It's tied to the scope of `s` only, and `'static` here is just an alias with no special meaning
- [ ] It's `'static` only in `release` builds; in `debug` builds it's dropped at end of scope

<details>
<summary>Show Answer</summary>

**Answer:** B — It's `'static` because the literal's bytes are embedded directly in the compiled binary's read-only memory, which exists for the whole program run

**Explanation:** String literals are stored in the binary itself, not on the heap or stack, so a `&str` pointing at one is trivially valid for the program's entire lifetime — this is why literals are the textbook example of `'static` data. A is false — general `&str` variables are *not* `'static` by default; only specific sources like literals are. C understates what `'static` means; it's a real, checked guarantee, not cosmetic. D is fabricated — build profile has no bearing on this.

</details>
::

::question-wrapper
### Q15. A public API function currently returns `&'a str` borrowed from an input, forcing every caller to keep the input alive as long as the result. What's the idiomatic best practice when the borrow relationship isn't essential to the API's purpose?

- [ ] Always keep borrowing return types — allocating is un-idiomatic in Rust
- [ ] Return an owned `String` instead, accepting the extra allocation, so callers aren't forced into awkward lifetime juggling
- [ ] Add `'static` to force the caller to leak the input
- [ ] Wrap the return type in `Rc<str>` regardless of whether sharing is needed

<details>
<summary>Show Answer</summary>

**Answer:** B — Return an owned `String` instead, accepting the extra allocation, so callers aren't forced into awkward lifetime juggling

**Explanation:** **Idiom:** Rust does favor borrowing when it's cheap and natural, but API ergonomics matter — if borrowing propagates a lifetime parameter through public structs and function signatures just to save one allocation, it's usually not worth the complexity for callers. A overstates the "avoid allocation at all costs" mentality; even the standard library returns owned `String`/`Vec` constantly. C is nonsensical — `'static` cannot be conjured onto arbitrary borrowed data without genuinely leaking or owning it. D adds unnecessary reference-counting overhead and API surface when sharing isn't actually required.

</details>
::

::question-wrapper
### Q16. When should you reach for an explicit lifetime annotation instead of relying on elision?

- [ ] Whenever a function takes more than zero parameters
- [ ] Only when the elision rules cannot uniquely determine the output lifetime, or when a struct/impl block needs to tie multiple borrows together explicitly
- [ ] Every function that returns any value at all
- [ ] Never — explicit lifetimes are a deprecated Rust 2015 feature

<details>
<summary>Show Answer</summary>

**Answer:** B — Only when the elision rules cannot uniquely determine the output lifetime, or when a struct/impl block needs to tie multiple borrows together explicitly

**Explanation:** **Idiom:** the elision rules exist precisely so idiomatic Rust rarely needs visible lifetime syntax; reaching for it should be a deliberate signal that the compiler genuinely needs disambiguation (two-plus input references feeding one output, or a struct storing borrowed fields). A and C wildly over-apply the rule and would make trivial functions unnecessarily verbose. D is false — explicit lifetimes are a core, actively used part of the language, not deprecated.

</details>
::

::question-wrapper
### Q17. Why is casually adding `dyn Trait + 'static` (or its implicit default) to every trait object in a long-lived cache considered a code smell when the objects actually hold short-lived borrowed data?

- [ ] It isn't a smell — `'static` should always be the default for trait objects
- [ ] It forces every value stored in the cache to be owned (or leaked), which can silently balloon memory or force unnecessary cloning just to satisfy the bound
- [ ] `dyn Trait` objects cannot have lifetime bounds at all
- [ ] `'static` on trait objects only affects `Debug` formatting

<details>
<summary>Show Answer</summary>

**Answer:** B — It forces every value stored in the cache to be owned (or leaked), which can silently balloon memory or force unnecessary cloning just to satisfy the bound

**Explanation:** `Box<dyn Trait>` defaults to `Box<dyn Trait + 'static>` unless you write a shorter bound like `Box<dyn Trait + 'a>`. Reaching for the default without thinking means anything with a borrowed lifetime can no longer be boxed as that trait object without first cloning into an owned form — a common surprise when refactoring borrowing code into a trait-object-based design. **Performance:** unnecessary clones to satisfy an overly broad `'static` bound are a frequent, easy-to-miss cost. A is the naive default that causes exactly this problem. C is false — `dyn Trait + 'a` is valid syntax. D is fabricated.

</details>
::

::question-wrapper
### Q18. A method takes `&self` and an unrelated `&str` parameter, and needs to return a reference borrowed from the parameter, not from `self`. Why can't you rely on elision here?

- [ ] Elision rule 3 always binds the output to `&self`'s lifetime when `self` is present, regardless of what you actually want to return — so you must write explicit lifetimes to override it
- [ ] Methods never support lifetime elision
- [ ] `&str` parameters are exempt from elision
- [ ] This works fine with elision as long as the method has a `&mut self` receiver instead

<details>
<summary>Show Answer</summary>

**Answer:** A — Elision rule 3 always binds the output to `&self`'s lifetime when `self` is present, regardless of what you actually want to return — so you must write explicit lifetimes to override it

**Explanation:** Rule 3 is a fixed heuristic, not a smart inference — it fires whenever a method has a `self`/`&self`/`&mut self` receiver plus other reference parameters, and unconditionally assigns `self`'s lifetime to the elided output. If you actually intend to return something borrowed from the *other* parameter, you must write `fn f<'a, 'b>(&'a self, s: &'b str) -> &'b str` explicitly, or the code either fails to compile (if `self`'s and the intended lifetime genuinely diverge) or silently over-constrains callers to keep `self` alive longer than necessary. B, C, and D are fabricated — elision works on methods too, applies to any `&T`, and `&mut self` follows the identical rule 3 as `&self`.

</details>
::

::question-wrapper
### Q19. In a builder-pattern struct, why is it usually better to store owned `String` fields rather than `&'a str` fields tied to the builder's lifetime?

- [ ] `&'a str` fields are always a compile error inside structs
- [ ] Owned fields let the built value (and the builder itself) be returned from functions, stored in collections, or moved across threads without dragging a lifetime parameter through every type that uses the builder
- [ ] `String` is faster to compare than `&str`
- [ ] Borrowed fields make `derive(Clone)` impossible

<details>
<summary>Show Answer</summary>

**Answer:** B — Owned fields let the built value (and the builder itself) be returned from functions, stored in collections, or moved across threads without dragging a lifetime parameter through every type that uses the builder

**Explanation:** **Idiom:** this is a widely recommended Rust API-design tradeoff — a `Config<'a>` builder that borrows its strings infects every function signature and struct that holds a `Config` with the same `'a`, which is rarely worth the saved allocations for something constructed once and used broadly. A is false — borrowed fields compile fine, they're just viral. C is an unrelated, generally false claim (comparison cost is similar). D is false — `Clone` works on borrowed fields too, it just clones the reference (a pointer copy), not the data.

</details>
::

::question-wrapper
### Q20. Code review flags this function: `fn cache_key<'a>(&'a self, id: u32) -> &'a str`, used to look up and return a formatted key. The reviewer says "this design is a lifetime trap waiting to bite the next person who calls it in a loop." What's the most likely underlying issue?

- [ ] Returning `&'a str` tied to `&self` means the borrow of `self` must stay alive as long as the returned key is used, which can prevent later mutable access to `self` (e.g., inserting into the same cache) within that same scope
- [ ] `u32` parameters cannot be used inside lifetime-annotated functions
- [ ] The function should return `Result<&str, Error>` instead
- [ ] There is no real issue — the reviewer is being overly cautious

<details>
<summary>Show Answer</summary>

**Answer:** A — Returning `&'a str` tied to `&self` means the borrow of `self` must stay alive as long as the returned key is used, which can prevent later mutable access to `self` (e.g., inserting into the same cache) within that same scope

**Explanation:** This is the classic "read borrow blocks a later write borrow" trap: if `cache_key` returns a reference borrowed from `self`, and the caller then tries `self.insert(...)` (which needs `&mut self`) while still holding that returned key, the borrow checker rejects it — even though the two operations don't actually alias the same memory at runtime. **Idiom:** the fix is usually to return an owned `String` (or clone before mutating), trading a small allocation for an API that doesn't force awkward borrow-scope gymnastics on every caller. B is a fabricated restriction — primitive parameters don't interact with lifetime annotations. C addresses error handling, not the lifetime coupling being described. D dismisses a real, common production footgun.

</details>
::
