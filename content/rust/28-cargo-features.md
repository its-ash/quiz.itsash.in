# 28 — Cargo Features

::question-wrapper{language="toml"}
### Q1. What is the correct way to declare a feature named `logging` that has no dependencies of its own in `Cargo.toml`?

::code-wrapper{language="toml"}
```toml
[dependencies]
tracing = { version = "0.1", optional = true }
```
::

- [ ] `[features]` section with `logging = []`
- [ ] `[dependencies.logging]` with `optional = true`
- [ ] `logging = true` at the top level of `Cargo.toml`
- [ ] `[profile.logging]` with `enabled = true`

<details>
<summary>Show Answer</summary>

**Answer:** A — `[features]` section with `logging = []`

**Explanation:** Features are declared under a `[features]` table as `name = [list-of-other-features-or-optional-deps-to-enable]`; an empty list is valid for a "marker" feature used only in `#[cfg(feature = "logging")]` checks. `[dependencies.logging]` is wrong because features aren't dependencies unless tied to an optional crate. `logging = true` at the top level isn't valid TOML for Cargo's schema. `[profile.*]` tables configure compilation profiles (opt-level, debug info), not feature flags — a common mix-up for newcomers scanning `Cargo.toml`.

</details>
::

::question-wrapper
### Q2. A crate declares `serde = { version = "1", optional = true }` under `[dependencies]`. What happens by default?

- [ ] Cargo implicitly creates a feature named `serde` that enables the dependency
- [ ] The dependency is compiled in unconditionally, `optional` only affects `cargo doc`
- [ ] The build fails until a matching `[features]` entry named `serde` is added manually
- [ ] The dependency is never compiled unless explicitly named in `[build-dependencies]` too

<details>
<summary>Show Answer</summary>

**Answer:** A — Cargo implicitly creates a feature named `serde` that enables the dependency

**Explanation:** Marking a dependency `optional = true` automatically creates an implicit feature of the same name; enabling that feature pulls in the crate. No manual `[features]` entry is required (though on the 2021 edition you can opt into requiring the explicit `dep:serde` syntax to *suppress* that implicit feature — see the `dep:` question later in this quiz). It doesn't compile unconditionally, and it has nothing to do with `[build-dependencies]`, which is a separate dependency graph for build scripts.

</details>
::

::question-wrapper
### Q3. Which flag builds a crate with its `default` feature set disabled?

- [ ] `cargo build --no-default-features`
- [ ] `cargo build --features=none`
- [ ] `cargo build --default-features=false`
- [ ] `cargo build --minimal`

<details>
<summary>Show Answer</summary>

**Answer:** A — `cargo build --no-default-features`

**Explanation:** `--no-default-features` opts out of whatever `[features] default = [...]` lists; you then opt back in to specific features with `--features`. `--default-features=false` is invalid CLI syntax (that phrasing only exists as a Cargo.toml dependency table key, not a top-level flag), `--features=none` just tries to enable a feature literally named `none`, and `--minimal` isn't a real Cargo flag.

</details>
::

::question-wrapper{language="toml"}
### Q4. In `Cargo.toml`, what does this dependency declaration mean?

::code-wrapper{language="toml"}
```toml
[dependencies.reqwest]
version = "0.11"
default-features = false
features = ["json"]
```
::

- [ ] Depend on `reqwest` with its own defaults disabled, enabling only the `json` feature
- [ ] Depend on `reqwest` with all features enabled, then filter down to `json` at link time
- [ ] `default-features = false` disables features for the whole workspace, not just `reqwest`
- [ ] This is a syntax error; `features` and `default-features` cannot appear together

<details>
<summary>Show Answer</summary>

**Answer:** A — Depend on `reqwest` with its own defaults disabled, enabling only the `json` feature

**Explanation:** `default-features = false` on a *specific dependency entry* opts that dependency out of its own `default` feature set (defined in *its* `Cargo.toml`), scoped only to that dependency — it has no effect on the current crate's own features or the workspace. `features = ["json"]` then explicitly turns on just that one. The two keys commonly appear together and are not mutually exclusive; there's nothing resembling "filtering at link time."

</details>
::

::question-wrapper
### Q5. Crate `A` depends on `net-lib` with `features = ["tls"]`. Crate `B` (a sibling dependency in the same build) depends on `net-lib` with `default-features = false` and no extra features. Both end up in the same final binary. What features does `net-lib` actually get compiled with?

- [ ] Only `tls` — Cargo unifies to the union of all requested features across the graph
- [ ] Neither — `default-features = false` from B overrides A's request
- [ ] `net-lib` is compiled twice, once per requested feature set
- [ ] The build fails with a "conflicting feature requirements" error

<details>
<summary>Show Answer</summary>

**Answer:** A — Only `tls` — Cargo unifies to the union of all requested features across the graph

**Explanation:** **Safety/Idiom:** Cargo compiles each version of a dependency exactly once per build graph, so it must reconcile every requester's feature request into a single set — it takes the *union*. B's `default-features = false` only says "don't turn on `net-lib`'s own defaults *for my request*"; it can't retract a feature that A separately asked for. This is the classic "feature unification" footgun: you cannot rely on disabling defaults to shrink a dependency if any other crate in the graph re-enables them. There's no per-requester recompilation and no conflict error — features are required to be purely additive for exactly this reason.

</details>
::

::question-wrapper{language="toml"}
### Q6. A library crate wants an optional dependency `regex` available for internal use, but does NOT want a public feature named `regex` to appear in its feature list (to avoid it being confused with a user-facing toggle). Which mechanism achieves this?

::code-wrapper{language="toml"}
```toml
[dependencies]
regex = { version = "1", optional = true }

[features]
search = ["dep:regex"]
```
::

- [ ] The `dep:regex` syntax in `[features]`, which suppresses the automatic implicit `regex` feature
- [ ] Renaming the crate import with `package = "regex"` under a different key
- [ ] Marking the dependency `optional = true, hidden = true`
- [ ] There is no way to do this; every optional dependency always gets a public feature

<details>
<summary>Show Answer</summary>

**Answer:** A — The `dep:regex` syntax in `[features]`, which suppresses the automatic implicit `regex` feature

**Explanation:** Since the 2021 edition's feature resolver, referencing an optional dependency as `dep:regex` inside any `[features]` entry suppresses the automatic implicit feature of the same name, letting you expose only `search` publicly while still gating the dependency internally. `hidden = true` isn't a real Cargo.toml key. Renaming via `package =` changes which crate is pulled in under a local name, it doesn't touch feature visibility. Before this mechanism existed, optional deps unavoidably created a same-named public feature — that's exactly the gap `dep:` was added to close.

</details>
::

::question-wrapper
### Q7. What is the idiomatic way to express "this crate is `no_std` unless a feature is enabled"?

- [ ] Add a feature (e.g. `std`) that, when enabled, brings in `std`; keep the crate `#![no_std]` by default and gate `std`-only code behind `#[cfg(feature = "std")]`
- [ ] Add a feature named `no_std` that disables `std` when enabled
- [ ] Use `default-features = false` in the crate's own `[package]` section to disable `std`
- [ ] It cannot be done — `no_std` must be a fixed, unconditional attribute

<details>
<summary>Show Answer</summary>

**Answer:** A — Add a positive `std` feature; stay `#![no_std]` by default and gate on `#[cfg(feature = "std")]`

**Explanation:** **Idiom:** Cargo features must be additive/unification-safe — enabling a feature can only ever add capability, never remove it, because unification takes the union across the whole graph. A `no_std` feature that *removes* `std` support when turned on would break the instant two crates in the graph disagree (one wants `std`, one wants "not std" — unification would nonsensically enable both). The idiomatic pattern flips the polarity: the crate is minimal (`no_std`) by default, and an additive `std` feature unlocks more. `default-features = false` in `[package]` isn't a valid key — that phrasing only applies to dependency entries.

</details>
::

::question-wrapper
### Q8. A `[features]` table defines `backend-a = []` and `backend-b = []`, and the crate's code assumes exactly one is active, panicking or producing nonsensical output if both are compiled in at once. A downstream user's dependency graph ends up unifying both features on. What is the actual build-time outcome?

- [ ] Cargo compiles successfully with both features enabled — Cargo has no built-in concept of mutually exclusive features
- [ ] Cargo automatically detects the conflict from the `[features]` table shape and refuses to build
- [ ] Cargo picks whichever feature was requested first in the dependency graph and silently ignores the other
- [ ] Cargo emits a warning but disables both features to avoid ambiguity

<details>
<summary>Show Answer</summary>

**Answer:** A — Cargo compiles successfully; it has no built-in concept of mutually exclusive features

**Explanation:** **Safety:** Cargo's feature system was designed to be purely additive so unification always has a well-defined answer (the union); it has no syntax for declaring two features as mutually exclusive, and it will happily enable both if any two crates in the graph disagree. If the crate author needs a hard failure, the idiom is to add `compile_error!("backend-a and backend-b are mutually exclusive")` inside a `#[cfg(all(feature = "backend-a", feature = "backend-b"))]` block so misuse fails loudly at compile time instead of silently misbehaving at runtime. Cargo does not pick a winner or auto-disable — that would violate the additive guarantee other tooling relies on.

</details>
::

::question-wrapper
### Q9. A binary crate depends on `image` with `default-features = false` in hopes of dropping the (heavy) `png` default codec. Another dependency deeper in the graph pulls in `image` with its regular defaults. After unification, is `png` support present in the final binary?

- [ ] Yes — because the other dependency's request for defaults wins the union, `default-features = false` cannot retract it
- [ ] No — `default-features = false` anywhere in the graph always wins and strips defaults for everyone
- [ ] It depends on declaration order in `Cargo.toml`
- [ ] No — Cargo deduplicates by intersecting feature requests, not unioning them

<details>
<summary>Show Answer</summary>

**Answer:** A — Yes; the other dependency's request for defaults wins the union

**Explanation:** **Performance:** This is a widely-hit real-world footgun: `default-features = false` is a per-requester opt-out, not a graph-wide veto. Because unification takes the union of every requester's features, as soon as *any* crate in the graph asks for `image`'s defaults, they're included in the single shared compilation — your own `default-features = false` only means "I personally don't require them," not "disable them globally." The only reliable fix is to get the other dependency changed (or patched/forked) to also opt out, or to avoid depending on the offending crate. There is no declaration-order tiebreak and no intersection semantics — it's always a union.

</details>
::

::question-wrapper
### Q10. A crate has `[features] full = ["net", "db", "compress"]`. If a user builds with `cargo build --features net`, which of `db` and `compress` get enabled?

- [ ] Neither — `full` is just an alias that itself must be explicitly requested to pull in its members
- [ ] Both — enabling any member of `full` retroactively enables the whole group
- [ ] Only `db`, because it's listed second
- [ ] It's a compile error to request a partial subset of a feature group

<details>
<summary>Show Answer</summary>

**Answer:** A — Neither; `full` is just an alias and must itself be requested

**Explanation:** A feature entry like `full = ["net", "db", "compress"]` defines a one-directional relationship: turning `full` *on* turns those three on. It says nothing about the reverse — enabling `net` alone has no effect on `full`, `db`, or `compress`. This trips people up because they expect feature "groups" to behave symmetrically; in reality `full` is nothing more than syntactic sugar for "enable these three together," requested independently like any other feature.

</details>
::

::question-wrapper
### Q11. A crate lists `criterion = "0.5"` only under `[dev-dependencies]`, not under `[dependencies]`. Can library code (non-test, non-bench) reference `criterion` when the crate is built normally with `cargo build`?

- [ ] No — dev-dependencies are only available to tests, examples, and benchmarks, not the library target during a normal build
- [ ] Yes — dev-dependencies are merged into the normal dependency graph automatically
- [ ] Only if a matching feature flag with the same name is also enabled
- [ ] Only in debug builds; release builds strip dev-dependencies but still link them in debug

<details>
<summary>Show Answer</summary>

**Answer:** A — No; dev-dependencies are only available to tests, examples, and benchmarks

**Explanation:** `[dev-dependencies]` populate a separate graph used only when building test/bench/example targets (`cargo test`, `cargo bench`, `cargo run --example`); they are not linked into the library or binary artifact produced by a plain `cargo build`, and downstream consumers of your published crate never see them at all. This is why crates.io rejects a `Cargo.toml` where library source references a dev-only dependency — it simply won't resolve outside test builds. There's no feature-flag coupling and no debug/release distinction here; the separation is by target kind, not build profile.

</details>
::

::question-wrapper
### Q12. Inside a `build.rs` script, what is the correct way to check whether the feature `json` is enabled for the crate being built?

- [ ] Read the `CARGO_FEATURE_JSON` environment variable
- [ ] Use `#[cfg(feature = "json")]` directly inside `build.rs`
- [ ] Call `cargo::features::is_enabled("json")` from the `cargo` crate
- [ ] Parse `Cargo.toml` manually since features aren't otherwise visible to build scripts

<details>
<summary>Show Answer</summary>

**Answer:** A — Read the `CARGO_FEATURE_JSON` environment variable

**Explanation:** **Debug:** Cargo exposes each enabled feature to `build.rs` as an environment variable named `CARGO_FEATURE_<FEATURE_NAME_UPPERCASE_WITH_UNDERSCORES>` (e.g. `json` → `CARGO_FEATURE_JSON`), set to `1` when enabled and absent otherwise. `#[cfg(feature = ...)]` doesn't apply here because `build.rs` is compiled and run as a *separate* program before the crate itself, under a different cfg context — a mistake newcomers make constantly. There's no such `cargo` crate API for this, and manually parsing `Cargo.toml` is unnecessary and fragile since Cargo already does the resolution and hands you the answer via env vars.

</details>
::

::question-wrapper
### Q13. Running plain `cargo doc` (no extra flags) on a crate with several feature-gated public items — what shows up in the generated documentation for those items?

- [ ] Only items reachable under the crate's default feature set; feature-gated items behind non-default features are omitted
- [ ] All items regardless of feature gating — `cargo doc` always compiles with every feature enabled
- [ ] Nothing — `cargo doc` requires `--all-features` to produce any output at all
- [ ] Feature-gated items appear but are rendered with strikethrough text automatically

<details>
<summary>Show Answer</summary>

**Answer:** A — Only items reachable under the crate's default feature set

**Explanation:** `cargo doc` builds documentation the same way `cargo build` compiles code: using whatever feature set is active for the invocation, which defaults to the `default` feature list. Items behind non-default `#[cfg(feature = "...")]` gates simply don't exist in that compilation and so never reach the doc generator. To document everything, maintainers run `cargo doc --all-features` (or docs.rs's `[package.metadata.docs.rs] all-features = true` config). Plain `cargo doc` does not fail on zero features, and there's no automatic strikethrough rendering for gated items.

</details>
::

::question-wrapper
### Q14. A workspace has two member crates, `app` and `lib`, both depending on `serde`. `app`'s `Cargo.toml` requests `serde` with the `derive` feature; `lib`'s does not. When building the whole workspace with `cargo build --workspace`, does `lib`'s compilation unit get `serde`'s `derive` feature?

- [ ] Yes — feature unification happens across the whole workspace build by default, so `serde/derive` is active everywhere `serde` is used in that build
- [ ] No — each workspace member is compiled with only the features it explicitly lists
- [ ] Only if `lib` is listed before `app` in the workspace `[members]` array
- [ ] It depends on whether `resolver = "1"` or `"2"` is set, but resolver 2 disables unification entirely

<details>
<summary>Show Answer</summary>

**Answer:** A — Yes, feature unification applies across the whole workspace build by default

**Explanation:** When multiple workspace members share a dependency and are built together in one `cargo build --workspace` invocation, Cargo still compiles that shared dependency once, unified across every member being built in that command — exactly like unification across a single crate's transitive graph. This surprises people who expect workspace members to be isolated. The 2021 "feature resolver v2" (`resolver = "2"`) *narrows* unification (notably separating host/target and dev-dependency feature sets) but does not eliminate cross-member unification for normal dependencies built together; it does not disable unification outright. Member order in `[members]` has no bearing on this.

</details>
::

::question-wrapper{language="toml"}
### Q15. A published library crate wants to offer an async API built on `tokio` but also be usable by consumers who don't want `tokio` pulled in at all. What is the idiomatic Cargo.toml structure?

::code-wrapper{language="toml"}
```toml
[dependencies]
tokio = { version = "1", optional = true, features = ["rt"] }

[features]
async = ["dep:tokio"]
```
::

- [ ] Make `tokio` an optional dependency gated behind an additive `async` feature that is NOT in `default`
- [ ] Depend on `tokio` unconditionally; consumers who don't want it can use `--no-default-features` on their own crate
- [ ] Vendor a minimal subset of `tokio`'s API directly into the crate to avoid the dependency entirely
- [ ] Use two entirely separate published crates with identical code, one with `tokio` and one without

<details>
<summary>Show Answer</summary>

**Answer:** A — Make `tokio` optional, gated behind a non-default `async` feature

**Explanation:** **Idiom:** This is the standard pattern seen across the ecosystem (e.g. `reqwest`, `sqlx`): mark the runtime dependency `optional = true` and expose it through an additive feature that consumers opt into, leaving it off of `default` so sync-only consumers pay zero cost. Depending on `tokio` unconditionally forces every consumer to compile and link it even if they never touch the async API — `--no-default-features` on the *consumer's own crate* does nothing to strip a dependency the library itself declared unconditionally. Vendoring or forking into duplicate crates massively increases maintenance burden for no benefit when Cargo already solves this cleanly.

</details>
::

::question-wrapper
### Q16. Which of these is the best practice for a feature that changes the *behavior* of existing default functionality (e.g. switching a hash function to a faster non-cryptographic one), rather than adding new opt-in API surface?

- [ ] Avoid it if at all possible — behavior-changing features violate the "features are additive" convention and can silently produce different results depending on unrelated crates in the dependency graph
- [ ] It's fine as long as the feature is well documented in the crate's README
- [ ] Always make such features part of `default` so behavior is consistent
- [ ] Use a build-time environment variable instead of a Cargo feature, since env vars are more explicit

<details>
<summary>Show Answer</summary>

**Answer:** A — Avoid it; behavior-changing features break the additive convention

**Explanation:** **Safety:** Because feature unification means "if anything in the dependency graph enables this feature, it's on for everyone," a feature that changes semantics (not just adds capability) can silently alter your crate's behavior based on an unrelated dependency three levels away enabling it for its own reasons — a notoriously hard bug to trace. The Cargo team's own guidance is that features should be strictly additive; behavior toggles are better expressed as separate crates, runtime configuration, or distinct type-level APIs. Documentation doesn't fix the unification hazard, making it `default` doesn't solve the underlying issue, and environment variables introduce their own non-reproducibility problems for library code (they're invisible to `Cargo.lock` and to downstream consumers reasoning about a crate's compiled behavior).

</details>
::

::question-wrapper{language="toml"}
### Q17. A crate wants to enable a specific feature of an optional dependency (`serde/derive`) only when its own dependent turns that dependency on elsewhere in the graph, without itself forcing the optional dependency to be pulled in. Which syntax expresses this "weak" feature dependency?

::code-wrapper{language="toml"}
```toml
[features]
derive-support = ["serde?/derive"]
```
::

- [ ] The `dep_name?/feature_name` syntax (e.g. `serde?/derive`)
- [ ] The `dep_name/feature_name` syntax without a `?`
- [ ] `optional-features = ["serde/derive"]` under `[package]`
- [ ] There is no such mechanism; enabling a sub-feature always forces the base dependency on

<details>
<summary>Show Answer</summary>

**Answer:** A — The `dep_name?/feature_name` weak-dependency-feature syntax

**Explanation:** **Idiom:** `serde?/derive` (the `?` marks it "weak") means "if `serde` ends up enabled by *someone* in the graph, also turn on its `derive` feature — but don't turn `serde` on by yourself." This is exactly for the case in the question: contributing a feature to a dependency without forcing that dependency to exist. The non-`?` form `serde/derive` is the older, "strong" syntax that *does* implicitly enable `serde` as a side effect, which is the opposite of what's wanted here. `[package]` has no `optional-features` key.

</details>
::

::question-wrapper
### Q18. Why do many crates.io libraries deliberately keep `[features] default = []` (empty) rather than defaulting to their most common configuration?

- [ ] To let downstream consumers opt into exactly what they need without unwanted transitive dependencies or unification surprises leaking through unless explicitly requested
- [ ] Because Cargo requires `default` to be empty for crates published to crates.io
- [ ] Because non-empty defaults are silently ignored by `cargo install`
- [ ] It has no practical effect; `default = []` and omitting `[features]` entirely behave identically in every situation

<details>
<summary>Show Answer</summary>

**Answer:** A — To let consumers opt in explicitly and avoid unwanted transitive weight or unification surprises

**Explanation:** **Idiom:** Keeping `default` minimal (or empty) is a deliberate ecosystem convention for foundational crates so that consumers pay only for what they ask for, and so that unification doesn't quietly drag in heavy optional dependencies just because *some* crate in the graph forgot to pass `default-features = false`. This is purely a design choice, not a crates.io requirement — Cargo places no restriction on `default` content for publishing. `cargo install` respects defaults normally. And the last option is wrong in a subtle way: omitting `[features]` entirely means the crate has zero features at all (nothing to enable), whereas an explicit `default = []` still allows *other*, non-default features to exist and be turned on individually — they're not identical when the crate defines additional optional features.

</details>
::

::question-wrapper
### Q19. What is the most reliable way to verify that a crate compiles correctly under every individual feature combination, not just "all features on" or "default only," before publishing?

- [ ] Use a tool like `cargo hack --feature-powerset --no-dev-deps check` in CI to compile every combination of feature flags
- [ ] Run `cargo check` once with `--all-features`; if that passes, every subset is guaranteed to also compile
- [ ] Run `cargo test` once; test failures would reveal any feature-combination compile errors
- [ ] It's unnecessary — Cargo's unification guarantees every subset compiles if the full set does

<details>
<summary>Show Answer</summary>

**Answer:** A — `cargo hack --feature-powerset` (or similar) to check every combination

**Explanation:** **Idiom/Debug:** `--all-features` on is a *different* compilation than any individual subset — a crate can easily compile fine with everything on (where conflicting `#[cfg]` branches never both trigger issues) yet fail to compile with only feature `x` on and not `y`, especially with mutually-referential `#[cfg(feature = ...)]` code. `cargo-hack`'s `--feature-powerset` mode compiles every combination in turn and is the standard CI safeguard used by mature crates for exactly this reason. `cargo test` only runs whatever single feature set was active for that invocation, so it can't surface combination-specific breakage on its own, and there is no guarantee that "all features compiles" implies "every subset compiles" — the additive-features convention is a social contract enforced by testing, not something the compiler verifies for you.

</details>
::

::question-wrapper
### Q20. A crate has a feature `unstable` that exposes experimental, semver-unstable APIs. What is the recommended convention for how this feature interacts with semantic versioning guarantees?

- [ ] Document explicitly that APIs behind `unstable` are exempt from semver guarantees and may break even in patch releases
- [ ] Treat `unstable` like any other feature — once published, its APIs are covered by normal semver like everything else
- [ ] Cargo automatically excludes `#[cfg(feature = "unstable")]` items from semver compatibility checks with no documentation needed
- [ ] Publish `unstable`-gated code only to a separate `-unstable` suffixed crate name, never mixed into the main crate

<details>
<summary>Show Answer</summary>

**Answer:** A — Explicitly document that `unstable`-gated APIs are exempt from normal semver guarantees

**Explanation:** **Idiom:** Since Cargo has no built-in notion of "this feature's APIs don't count toward semver" (there is no automatic exemption — that part of option C is fabricated), crates that ship experimental surface behind a feature like `unstable` rely entirely on clear documentation (README, feature doc comment, changelog policy) stating that breaking changes there can land in any release, including patches. This is a widely used convention (e.g. in `tokio`, `rand`) precisely because publishing genuinely separate crates for every experimental API would fragment the ecosystem and versioning unnecessarily. Treating it like normal semver-bound code would freeze experimentation prematurely.

</details>
::
