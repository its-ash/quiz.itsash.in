# 20 — Modules & Crates

::question-wrapper
### Q1. What is the default visibility of an item (function, struct, module) declared inside a Rust module with no visibility modifier?

- [ ] `pub` — visible everywhere, including other crates
- [ ] `pub(crate)` — visible anywhere in the current crate
- [ ] Private — visible only in the defining module and its descendant modules
- [ ] Private — visible only within the exact same file

<details>
<summary>Show Answer</summary>

**Answer:** C — Private — visible only in the defining module and its descendant modules

**Explanation:** Rust's default visibility is private, but "private" means "visible within this module and everything nested inside it," not "visible only in this file." An item with no modifier is invisible to sibling or parent modules, and completely invisible outside the crate, ruling out A (which describes `pub`) and B (which describes `pub(crate)`). D is the common misconception — file boundaries and module boundaries aren't the same thing, and a child module declared via `mod` (even in a separate file) still counts as "inside" its parent for privacy purposes.

</details>
::

::question-wrapper
### Q2. What does `pub(crate)` mean when applied to a function?

- [ ] The function is visible only within the module it's declared in
- [ ] The function is visible anywhere inside the current crate, but not to external crates that depend on this one
- [ ] The function is visible to external crates but not within the current crate
- [ ] The function is exported only when the crate is compiled as a library

<details>
<summary>Show Answer</summary>

**Answer:** B — The function is visible anywhere inside the current crate, but not to external crates that depend on this one

**Explanation:** `pub(crate)` is a restricted-visibility modifier: it opens the item up crate-wide (any module in the same crate can call it) while still hiding it from downstream crates that use this one as a dependency — useful for internal helpers shared across modules without committing them to the public API. A undersells it (that's closer to fully private); C has it backwards; D confuses visibility with crate type (`pub(crate)` behaves the same whether the crate is a binary or library, it just controls in-crate reach).

</details>
::

::question-wrapper
### Q3. In a typical binary crate, which file is the crate root that the compiler starts building from?

- [ ] `lib.rs`
- [ ] `mod.rs`
- [ ] `main.rs`
- [ ] `Cargo.toml`

<details>
<summary>Show Answer</summary>

**Answer:** C — `main.rs`

**Explanation:** For a binary crate, `src/main.rs` is the crate root — the module tree is built starting from the `mod` declarations in that file. `lib.rs` (A) is the equivalent root for a *library* crate (a package can have both, producing both a binary and a library target). `mod.rs` (B) is a filename convention for a submodule's contents, not a crate root. `Cargo.toml` (D) is the package manifest — it configures the build but contains no Rust code or module declarations itself.

</details>
::

::question-wrapper
### Q4. Given `src/main.rs` contains `mod network;`, which file layouts are valid ways to provide that module's contents (Rust 2018+)?

- [ ] Only `src/network.rs`
- [ ] Only `src/network/mod.rs`
- [ ] Either `src/network.rs`, or `src/network/mod.rs` — both are recognized, though mixing conventions within one project is discouraged
- [ ] `src/mod/network.rs`

<details>
<summary>Show Answer</summary>

**Answer:** C — Either `src/network.rs`, or `src/network/mod.rs` — both are recognized, though mixing conventions within one project is discouraged

**Explanation:** The compiler accepts two layouts for a module declared with `mod network;`: the flat `src/network.rs` (the modern, 2018-edition-encouraged style, needed if `network` itself has submodules living in a `src/network/` directory), or the older `src/network/mod.rs` style inherited from Rust 2015. A and B are each only half the truth — both forms genuinely work. D is not a recognized layout at all; `mod.rs` must live *inside* a directory named after the module, not inside a literal `mod/` directory.

</details>
::

::question-wrapper
### Q5. What does `use crate::utils::parse;` do?

- [ ] Declares a new module named `parse`
- [ ] Brings the `parse` item, referenced by its absolute path from the crate root, into scope under a short local name
- [ ] Makes `parse` public to other crates
- [ ] Re-exports `parse` so downstream crates can import it via this path

<details>
<summary>Show Answer</summary>

**Answer:** B — Brings the `parse` item, referenced by its absolute path from the crate root, into scope under a short local name

**Explanation:** `use` is purely a local scoping/aliasing tool: `crate::utils::parse` is an absolute path starting from the crate root, and the `use` statement lets the current file refer to that item as just `parse` instead of writing the full path every time. It does not declare anything new (rules out A — `mod` does that), does not change any item's visibility (rules out C — visibility is set at the item's definition with `pub`/`pub(crate)`/etc.), and a plain `use` (without `pub`) is not visible to code importing *this* module, so it does not re-export anything (rules out D — that requires `pub use`).

</details>
::

::question-wrapper
### Q6. What is the purpose of `pub use inner::Thing;` written in a parent module?

- [ ] It moves `Thing` out of `inner` into the parent module
- [ ] It re-exports `Thing`, making it accessible via the parent module's path (e.g. `crate::Thing`) in addition to its original path, without duplicating the definition
- [ ] It is a syntax error — `use` cannot be combined with `pub`
- [ ] It makes `inner` itself public

<details>
<summary>Show Answer</summary>

**Answer:** B — It re-exports `Thing`, making it accessible via the parent module's path (e.g. `crate::Thing`) in addition to its original path, without duplicating the definition

**Explanation:** `pub use` is Rust's re-export mechanism: it takes an item visible at this point and makes it visible under a *new* public path too, letting library authors flatten a deep internal module tree into a clean public API (e.g. `pub use inner::Thing;` at the crate root lets consumers write `my_crate::Thing` instead of `my_crate::inner::Thing`). `Thing` still physically lives in `inner`; nothing is moved (rules out A) or duplicated. `pub use` is valid, common syntax (rules out C). Re-exporting `Thing` says nothing about `inner`'s own visibility — `inner` can remain a private module while one specific item from it is re-exported (rules out D).

</details>
::

::question-wrapper
### Q7. Given `pub struct Config { pub name: String, version: u32 }`, what is true about a caller outside the module accessing `config.version`?

- [ ] It compiles — marking the struct `pub` makes all its fields `pub` too
- [ ] It fails to compile — `version` has no visibility modifier, so it defaults to private even though the struct itself is `pub`
- [ ] It compiles only if the caller is in the same crate
- [ ] It compiles, but emits a deprecation warning

<details>
<summary>Show Answer</summary>

**Answer:** B — It fails to compile — `version` has no visibility modifier, so it defaults to private even though the struct itself is `pub`

**Explanation:** Struct visibility and field visibility are independent in Rust: `pub` on the struct only controls whether the *struct type itself* (and its `pub` fields/methods) can be named from outside the module — each field still needs its own `pub` to be externally readable/writable. This is a common gotcha (assumption A) because many other languages tie member visibility to the containing type's visibility. It's not scoped to same-crate access — `pub(crate)` would be needed for that specific behavior, and plain private means "this module and descendants" regardless of crate boundary (rules out C). There's no warning-only leniency here; it's a hard compile error (rules out D).

</details>
::

::question-wrapper
### Q8. What does `super::` mean in a `use` path or item path?

- [ ] It refers to the crate root, equivalent to `crate::`
- [ ] It refers to the parent of the current module
- [ ] It refers to a trait's default (super) implementation
- [ ] It's not valid Rust syntax

<details>
<summary>Show Answer</summary>

**Answer:** B — It refers to the parent of the current module

**Explanation:** `super` is a relative-path keyword meaning "one level up in the module tree" — useful in a submodule (e.g. `tests`) that needs to reach items defined in its parent, via `use super::*;`. `crate::` (confused with A) instead always means "start from the absolute crate root," which is a different starting point unless the current module happens to be a direct child of the root. `super` has nothing to do with trait default methods (rules out C), and it is valid, commonly used syntax (rules out D).

</details>
::

::question-wrapper
### Q9. A private helper function is defined in module `a`. Module `a::b` is a child module (declared via `mod b;` inside `a`'s file). Can code inside `a::b` call that private function?

- [ ] No — private items are never visible outside their exact defining module
- [ ] Yes — Rust's privacy rule makes a private item visible in its defining module *and all of that module's descendants*, so child modules can see their ancestors' private items
- [ ] Only if `a::b` adds `use super::*;`
- [ ] Only if the function is also marked `pub(crate)`

<details>
<summary>Show Answer</summary>

**Answer:** B — Yes — Rust's privacy rule makes a private item visible in its defining module *and all of that module's descendants*, so child modules can see their ancestors' private items

**Explanation:** This is one of Rust's most commonly-missed privacy rules: "private" is not "only this module," it's "this module plus everything nested inside it." A child module is considered part of its parent's privacy boundary, so `a::b` can freely call a private (unmarked) item defined in `a` by referring to it via a path like `super::helper()`. A states the opposite of the actual rule. C conflates *importing a name into scope* with *visibility* — a `use super::*;` would bring the name into unqualified scope for convenience, but `super::helper()` would already compile without it, since visibility (not name resolution) is what was in question. D is unnecessary — no extra `pub(crate)` marker is needed for a descendant to see an ancestor's private item.

</details>
::

::question-wrapper
### Q10. What is the difference between `pub(super)` and `pub(crate)` on an item?

- [ ] They are identical
- [ ] `pub(super)` restricts visibility to just the parent module (and, transitively, that parent's descendants), while `pub(crate)` opens it to the entire crate
- [ ] `pub(super)` is for structs only, `pub(crate)` is for functions only
- [ ] `pub(super)` makes the item visible to external crates, `pub(crate)` does not

<details>
<summary>Show Answer</summary>

**Answer:** B — `pub(super)` restricts visibility to just the parent module (and, transitively, that parent's descendants), while `pub(crate)` opens it to the entire crate

**Explanation:** `pub(super)` is a narrower, path-scoped visibility modifier meaning "visible to my parent module" — useful for an item a submodule wants to expose upward without exposing it crate-wide. `pub(crate)` is broader, reaching every module in the crate regardless of position in the tree. They are not interchangeable (rules out A); both modifiers apply to any item kind — functions, structs, enums, modules — not restricted by kind (rules out C); and neither one reaches outside the crate at all — that requires plain `pub` (rules out D, which also has the external-visibility claim backwards).

</details>
::

::question-wrapper
### Q11. What happens if a project has both `src/network.rs` and `src/network/mod.rs` present at the same time, with `mod network;` declared in the crate root?

- [ ] The compiler merges both files' contents into one module
- [ ] The compiler picks `network.rs` and silently ignores `network/mod.rs`
- [ ] It's a compile error — the module `network` would be ambiguously defined by two different files
- [ ] The compiler picks whichever file was modified more recently

<details>
<summary>Show Answer</summary>

**Answer:** C — It's a compile error — the module `network` would be ambiguously defined by two different files

**Explanation:** The two module-file conventions (`name.rs` vs `name/mod.rs`) are alternatives, not layers that combine — the compiler requires exactly one to exist for a given `mod` declaration, and finding both is a "file for module found at two places" ambiguity error. This trips people up when migrating a codebase from the old `mod.rs` convention to the new flat-file convention and forgetting to delete the old file. There's no merging (rules out A), no silent precedence (rules out B), and Rust's module resolution has nothing to do with filesystem timestamps (rules out D) — it's a static, deterministic error regardless of file mtimes.

</details>
::

::question-wrapper
### Q12. A crate has `pub mod net { pub fn connect() {} }` in `lib.rs`, but nowhere does the crate write `use` for `connect`. Can an external crate that depends on this one call `this_crate::net::connect()`?

- [ ] No — without an explicit `use` inside the defining crate, nothing is exported
- [ ] Yes — `pub` on both the module and the function is sufficient; `use` is unrelated to whether external crates can reach an item, it only affects local scoping within the defining crate itself
- [ ] No — only items re-exported with `pub use` at the crate root are ever externally reachable
- [ ] Yes, but only if the external crate also declares `mod net;`

<details>
<summary>Show Answer</summary>

**Answer:** B — Yes — `pub` on both the module and the function is sufficient; `use` is unrelated to whether external crates can reach an item, it only affects local scoping within the defining crate itself

**Explanation:** External reachability is governed entirely by the chain of `pub` visibility from the crate root down to the item (`pub mod net` + `pub fn connect` means the full path `this_crate::net::connect` is public), regardless of whether the defining crate itself ever writes a `use` for it internally. `use` only affects how *that crate's own code* refers to the item by a shorter name — it plays no role in what's exposed externally. A and C both overstate what `use`/`pub use` are required for: `pub use` is only needed if you want to expose the item under a *different, shorter* path than its natural one, not merely to expose it at all. D is not how Rust's module system works — a downstream crate accesses items through the dependency's public path, it doesn't need to mirror the internal module structure with its own `mod` declaration.

</details>
::

::question-wrapper
### Q13. `mod shapes` privately defines `pub fn circle_area(r: f64) -> f64 {...}` and a private (non-`pub`) helper `fn validate(r: f64) -> bool {...}`. Elsewhere, code does `use shapes::*;`. What becomes available at the call site?

- [ ] Both `circle_area` and `validate`, since glob imports bring in everything regardless of visibility
- [ ] Only `circle_area` — glob imports still respect normal privacy rules, so private items are never pulled in even by `*`
- [ ] Neither — glob imports only work for enums, not modules
- [ ] Only `validate`, since glob imports prioritize private items

<details>
<summary>Show Answer</summary>

**Answer:** B — Only `circle_area` — glob imports still respect normal privacy rules, so private items are never pulled in even by `*`

**Explanation:** `use path::*;` is sugar for "bring every *visible-from-here* item at `path` into scope" — it is not a privacy bypass. Since `validate` has no `pub` and the `use` site is outside `shapes` and its descendants, `validate` was never visible there to begin with, glob or not. Assuming `*` reaches into private internals (A) is the tempting mistake; glob imports work for any module's contents, not just enum variants (rules out C, which describes a much narrower legitimate use of glob imports); and there's no such "private items take priority" behavior (rules out D).

</details>
::

::question-wrapper
### Q14. `main.rs` contains, in this order: `mod b; mod a;` where `a`'s code calls a function defined in `b`, and `b`'s code (declared textually *after* `a` in the file) calls a function defined in `a`. Does this compile?

- [ ] No — `b` is declared before `a`, so `a`'s items don't exist yet when `b` is compiled
- [ ] Yes — Rust resolves the whole module graph before checking cross-references, so declaration order of `mod` statements doesn't matter, unlike top-to-bottom execution order in scripting languages
- [ ] No — mutual references between sibling modules are always a compile error
- [ ] Yes, but only if both modules are also marked `pub`

<details>
<summary>Show Answer</summary>

**Answer:** B — Yes — Rust resolves the whole module graph before checking cross-references, so declaration order of `mod` statements doesn't matter, unlike top-to-bottom execution order in scripting languages

**Explanation:** Unlike C's single-pass, order-sensitive translation units, Rust builds a full module tree from all `mod` declarations in the crate before resolving any paths, so two sibling modules can freely call into each other regardless of which `mod` line appears first — there's no "not declared yet" state at the module-graph level. A projects familiar top-down/declaration-order thinking from other languages onto Rust, which is exactly the gotcha. Mutual references between modules are completely ordinary and common (rules out C) — Rust isn't a header-file/forward-declaration language. Visibility markers control whether *other* modules outside this pair can see the items, not whether `a` and `b` can see each other as siblings in the same crate (rules out D, and here both functions only need to be visible to sibling modules, which private-by-default already permits per the ancestor/descendant privacy rule, not `pub` specifically).

</details>
::

::question-wrapper
### Q15. When designing a library's public API, which is the more idiomatic structure?

- [ ] Require consumers to import everything via deep paths like `my_crate::internal::storage::backend::Client`
- [ ] Keep implementation details in whatever module nesting makes sense internally, then use `pub use` at the crate root (or a curated `prelude` module) to re-export the small set of types consumers actually need under short, stable paths
- [ ] Make every module and item `pub` so nothing is ever hidden
- [ ] Avoid modules entirely and put all code in a single file to eliminate path issues

<details>
<summary>Show Answer</summary>

**Answer:** B — Keep implementation details in whatever module nesting makes sense internally, then use `pub use` at the crate root (or a curated `prelude` module) to re-export the small set of types consumers actually need under short, stable paths

**Explanation:** **Idiom**: separating internal organization (which can change freely) from the public API surface (exposed via a small number of `pub use` re-exports) is standard practice in well-designed Rust crates — it lets internals be refactored without breaking downstream code, since only the re-exported paths are a semver commitment. Forcing consumers through deep internal paths (A) leaks implementation structure into your API contract, making future refactors breaking changes. Making everything `pub` (C) is the opposite problem — it maximizes the semver-committed surface area, including things you'll want to change later. Avoiding modules altogether (D) doesn't scale and throws away the organizational and privacy benefits modules provide.

</details>
::

::question-wrapper
### Q16. For a helper function used by multiple internal modules but never meant to be called by downstream crates, which visibility is generally the better default?

- [ ] `pub`, in case a downstream crate wants it someday
- [ ] `pub(crate)` — visible to every module inside this crate, but excluded from the crate's public API and semver contract
- [ ] Leave it fully private and duplicate the function in every module that needs it
- [ ] `pub(super)`, regardless of whether the callers are actually siblings or unrelated modules

<details>
<summary>Show Answer</summary>

**Answer:** B — `pub(crate)` — visible to every module inside this crate, but excluded from the crate's public API and semver contract

**Explanation:** **Idiom**: `pub(crate)` is the standard choice for "shared internally, not part of the public contract" — it gives every module in the crate access while keeping the item free to change or remove later without a semver-breaking release, since it was never externally reachable. Defaulting to `pub` "just in case" (A) is a common anti-pattern that needlessly locks the signature into your public API forever. Duplicating the function per module (C) creates maintenance drift and defeats the purpose of having a module system. `pub(super)` (D) only works when every caller happens to be the direct parent module — reaching for it regardless of the actual caller locations is fragile and will break the moment a caller lives elsewhere in the tree.

</details>
::

::question-wrapper
### Q17. In a large project migrating from the Rust 2015-style `mod.rs` layout to the modern flat-file layout, what is the practical motivation the community usually cites?

- [ ] `mod.rs` files are compiled slower than flat files
- [ ] Having many files all literally named `mod.rs` is hard to distinguish in editor tabs and file pickers, whereas `network.rs` immediately identifies itself
- [ ] `mod.rs` is deprecated and will be a hard compile error in a future edition
- [ ] Only the flat-file style supports `pub use` re-exports

<details>
<summary>Show Answer</summary>

**Answer:** B — Having many files all literally named `mod.rs` is hard to distinguish in editor tabs and file pickers, whereas `network.rs` immediately identifies itself

**Explanation:** **Idiom**: both layouts compile to the identical module structure and have no performance difference, so the widely cited reason for preferring `name.rs` over `name/mod.rs` is purely ergonomic — a dozen open `mod.rs` tabs are indistinguishable at a glance, while `network.rs`, `parser.rs`, etc. are self-identifying. There's no compile-speed difference (rules out A), `mod.rs` remains fully supported and not deprecated in current editions (rules out C), and `pub use` re-exporting works identically regardless of which file-layout convention defines the module (rules out D).

</details>
::

::question-wrapper
### Q18. Per common Rust formatting convention (and `rustfmt`/clippy defaults), how should `use` statements typically be grouped at the top of a file?

- [ ] In reverse-alphabetical order with no grouping
- [ ] Grouped into standard library (`std`/`core`/`alloc`), external crate, and local crate (`crate::`/`self::`/`super::`) blocks, each internally sorted
- [ ] All on a single line separated by semicolons for compactness
- [ ] Inline within each function right before first use, never at the top of the file

<details>
<summary>Show Answer</summary>

**Answer:** B — Grouped into standard library (`std`/`core`/`alloc`), external crate, and local crate (`crate::`/`self::`/`super::`) blocks, each internally sorted

**Explanation:** **Idiom**: separating `use` declarations into std / external-crate / local-crate groups (a convention `rustfmt` can enforce with `group_imports`, and that `cargo fmt`/community style guides converge on) makes it easy to scan a file's dependencies at a glance — what's from the standard library, what's a third-party crate, and what's local to this project. Reverse-alphabetical-only (A) ignores the semantic grouping that makes imports scannable. Cramming everything onto one line (C) is valid syntax but actively fights readability and diffing. Scattering `use` statements inline per-function (D) is unconventional in Rust — the idiom is top-of-file imports, unlike languages that favor fully local imports.

</details>
::

::question-wrapper
### Q19. Why is a blanket `pub use inner_module::*;` at a crate root generally discouraged compared to explicitly naming re-exports?

- [ ] It is a syntax error in current Rust editions
- [ ] It obscures exactly what's part of the public API (making intentional vs. accidental exposure hard to audit) and risks silent re-export naming conflicts as `inner_module` evolves
- [ ] Glob re-exports are always slower at runtime than named re-exports
- [ ] `pub use` with `*` only works for enums

<details>
<summary>Show Answer</summary>

**Answer:** B — It obscures exactly what's part of the public API (making intentional vs. accidental exposure hard to audit) and risks silent re-export naming conflicts as `inner_module` evolves

**Explanation:** **Idiom**: a glob `pub use` re-exports *everything* currently visible in `inner_module`, including items added later that the author may not have intended to commit to the public API — every new item silently becomes part of the semver contract, and two glob-reexported modules that later both add an item of the same name produce a re-export collision far from where the actual conflict originates. Explicit, named `pub use` statements make the public surface auditable at a glance and immune to that drift. It's valid, working syntax, not an error (rules out A); there is no runtime cost difference — `use`/`pub use` are purely a compile-time path/name resolution mechanism with zero runtime representation (rules out C); and glob re-exports work for any module's public contents, not just enums (rules out D).

</details>
::

::question-wrapper
### Q20. A crate is organized as `src/lib.rs`, `src/net/mod.rs` (or `src/net.rs`) declaring `pub(crate) mod client;` and `src/net/client.rs` defining `pub struct Client { ... }` with mostly `pub(crate)` helper functions alongside it. `lib.rs` also contains `pub use net::client::Client;`. What is the effect of this structure for a downstream crate?

- [ ] It cannot see `Client` at all, since `net` itself is only `pub(crate)`
- [ ] It can reach `Client` only via the full path `this_crate::net::client::Client`
- [ ] It can reach `Client` via the short, stable path `this_crate::Client`, while the `net`/`client` module layout and any `pub(crate)` helpers stay fully internal and free to refactor
- [ ] It gets a compile error because `pub use` cannot re-export an item from a `pub(crate)` module

<details>
<summary>Show Answer</summary>

**Answer:** C — It can reach `Client` via the short, stable path `this_crate::Client`, while the `net`/`client` module layout and any `pub(crate)` helpers stay fully internal and free to refactor

**Explanation:** This is the idiomatic re-export pattern in practice: `net` being merely `pub(crate)` would normally block external access to anything nested inside it, but the `pub use net::client::Client;` at the crate root creates an independent, fully `pub` path directly to `Client` that does not depend on `net`'s own visibility — re-export visibility is evaluated at the `pub use` site, not inherited from the source module's visibility. A is the tempting mistake of assuming a `pub use` inherits its source module's restricted visibility rather than establishing its own. B undersells the re-export — the whole point of `pub use` is to avoid forcing consumers through the internal path. D is wrong: re-exporting an item out of a less-visible module into a more-visible path is exactly what `pub use` is for and is a completely standard, compiling pattern, as long as the *item itself* (`Client`) is at least as visible as the path you're re-exporting it to (`Client` is `pub`, so this works).

</details>
::
