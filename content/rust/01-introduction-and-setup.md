# 01 — Introduction & Setup

::question-wrapper
### Q1. What is `rustup` primarily responsible for?

- [ ] Compiling Rust source files into binaries
- [ ] Managing Rust toolchain installations (stable/beta/nightly) and their components
- [ ] Managing project dependencies declared in `Cargo.toml`
- [ ] Formatting Rust source code according to style guidelines

<details>
<summary>Show Answer</summary>

**Answer:** B — Managing Rust toolchain installations (stable/beta/nightly) and their components

**Explanation:** `rustup` is the toolchain multiplexer/installer — it installs and switches between `rustc`/`cargo` versions and channels, and manages components like `clippy` or `rust-src`. Compiling source is `rustc`'s job (A), managing dependencies is `cargo`'s job (C), and formatting is `rustfmt`'s job (D) — `rustup` sits a layer above all of these tools rather than performing their work itself.

</details>
::

::question-wrapper
### Q2. A developer runs `rustc main.rs` directly instead of using `cargo build`. What is the main practical difference for a project with dependencies?

- [ ] There is no difference — both resolve crates listed in `Cargo.toml`
- [ ] `rustc` compiles a single file/crate root and has no knowledge of `Cargo.toml` dependencies, so external crates won't be found
- [ ] `rustc` is strictly for nightly builds while `cargo build` is stable-only
- [ ] `rustc main.rs` will automatically download and link any crates used with `use`

<details>
<summary>Show Answer</summary>

**Answer:** B — `rustc` compiles a single file/crate root and has no knowledge of `Cargo.toml` dependencies, so external crates won't be found

**Explanation:** `rustc` is the raw compiler; it takes a crate root and emits an artifact, but it has no concept of `Cargo.toml`, dependency resolution, or crates.io. `cargo` wraps `rustc`, resolving dependencies from `Cargo.toml`/`Cargo.lock` and invoking `rustc` with the correct `--extern` flags. **Idiom:** running bare `rustc` is fine for a single throwaway file, but any real project needs `cargo` to manage dependencies.

</details>
::

::question-wrapper
### Q3. As of Rust 2024, what edition does `cargo new my_project` select by default when using a current stable toolchain?

- [ ] 2015, for backward compatibility
- [ ] 2018
- [ ] 2021
- [ ] 2024

<details>
<summary>Show Answer</summary>

**Answer:** D — 2024

**Explanation:** `cargo new` writes the newest stable edition supported by the installed toolchain into `Cargo.toml`'s `edition` field. Since the 2024 edition shipped, current `cargo` defaults new projects to `edition = "2024"`. Older answers (2015/2018/2021) were correct defaults at earlier points in Rust's history but are not current — the edition a project targets is fixed by what's written in `Cargo.toml`, not by which compiler happens to build it later.

</details>
::

::question-wrapper
### Q4. What command installs a specific nightly toolchain and sets it as the default for the current shell/user?

- [ ] `cargo install nightly`
- [ ] `rustup toolchain add nightly`
- [ ] `rustup default nightly` (after nothing else)
- [ ] `rustup install nightly && rustup default nightly`

<details>
<summary>Show Answer</summary>

**Answer:** D — `rustup install nightly && rustup default nightly`

**Explanation:** `rustup install nightly` (alias for `rustup toolchain install nightly`) downloads the nightly toolchain, and `rustup default nightly` makes it the active default. `rustup toolchain add nightly` alone (C-style option) only installs it without switching the default. `cargo install` (A) installs Rust *binaries/crates*, not toolchains — a common beginner mix-up since both start with "install". Running `rustup default nightly` with nothing installed first (C) will still work because `rustup default` implicitly installs if missing, but it's not the most explicit/idiomatic pairing shown here.

</details>
::

::question-wrapper
### Q5. Which statement correctly distinguishes the stable, beta, and nightly release channels?

- [ ] Beta is for experimental features, nightly is the well-tested production channel
- [ ] Nightly gets new unstable features first and allows `#![feature(...)]` flags; beta is a preview of the next stable release; stable is the production-recommended channel
- [ ] Stable and nightly are identical except for release cadence
- [ ] Beta only receives security patches, while stable receives new features

<details>
<summary>Show Answer</summary>

**Answer:** B — Nightly gets new unstable features first and allows `#![feature(...)]` flags; beta is a preview of the next stable release; stable is the production-recommended channel

**Explanation:** Rust ships a new nightly every day (unstable, feature-gated APIs allowed), promotes a nightly to beta roughly every six weeks as a release candidate, and promotes beta to stable after another six weeks. Option A inverts the roles, and option D describes something closer to an LTS/security-only model, which Rust's channels do not follow.

</details>
::

::question-wrapper
### Q6. What does `rustup component add clippy` do?

- [ ] Installs the `clippy` crate as a project dependency in `Cargo.toml`
- [ ] Adds the `clippy` lint tool as an available component for the currently active toolchain
- [ ] Switches the active toolchain to a Clippy-specific fork of `rustc`
- [ ] Enables Clippy lints globally for all future `cargo build` invocations automatically

<details>
<summary>Show Answer</summary>

**Answer:** B — Adds the `clippy` lint tool as an available component for the currently active toolchain

**Explanation:** Components (`clippy`, `rustfmt`, `rust-src`, `rust-analyzer`, etc.) are optional pieces bundled per toolchain; `rustup component add` fetches and enables one for the active toolchain, after which `cargo clippy` becomes available. It is not a project dependency (A) and does not run automatically during `cargo build` (D) — you must explicitly invoke `cargo clippy`.

</details>
::

::question-wrapper
### Q7. Running `cargo --version` and `rustc --version` on a machine with `rustup` installed can show two different underlying toolchain versions. Why might this legitimately happen?

- [ ] It can't happen — `cargo` and `rustc` are always bundled from the exact same toolchain release
- [ ] `cargo` is versioned independently of `rustc` and is always one major version behind
- [ ] A per-directory `rustup override` or `rust-toolchain.toml` can pin a different toolchain than the global default, and `cargo`/`rustc` are invoked through `rustup`'s shims which resolve the toolchain per-invocation
- [ ] `rustc --version` always reports the nightly version regardless of the active toolchain

<details>
<summary>Show Answer</summary>

**Answer:** C — A per-directory `rustup override` or `rust-toolchain.toml` can pin a different toolchain than the global default, and `cargo`/`rustc` are invoked through `rustup`'s shims which resolve the toolchain per-invocation

**Explanation:** Both `cargo` and `rustc` on the PATH are actually `rustup` "proxy" shims that decide which real toolchain to invoke based on directory overrides, `rust-toolchain.toml`, environment variables, then the global default — so version drift between two separate shell sessions/directories is expected and normal, not a bug. Within a single resolved toolchain, `cargo` and `rustc` do ship together as a matched pair, contradicting the premise that cargo runs "one major version behind" (B).

</details>
::

::question-wrapper
### Q8. A CI pipeline pins `rust-toolchain.toml` with `channel = "1.72.0"`, but a contributor has only nightly installed locally via `rustup`. What happens when they run `cargo build` in that directory?

- [ ] `cargo` silently falls back to their installed nightly toolchain
- [ ] The build fails immediately with a "no default toolchain configured" panic
- [ ] `rustup` automatically downloads and installs the pinned 1.72.0 toolchain (if `auto-install` is enabled, the default), then uses it for that directory
- [ ] `cargo build` ignores `rust-toolchain.toml` unless `--locked` is passed

<details>
<summary>Show Answer</summary>

**Answer:** C — `rustup` automatically downloads and installs the pinned 1.72.0 toolchain (if `auto-install` is enabled, the default), then uses it for that directory

**Explanation:** `rust-toolchain.toml` overrides toolchain selection for anyone building inside that directory tree; by default `rustup` will fetch the missing pinned toolchain automatically rather than silently using whatever is already installed. **Portability:** this is precisely why teams commit `rust-toolchain.toml` — it makes builds reproducible across machines regardless of what each developer happened to install, avoiding the silent-fallback trap in option A which would produce version-dependent, non-reproducible builds.

</details>
::

::question-wrapper
### Q9. A file uses `#![feature(let_chains)]` at the crate root and is compiled with the default stable toolchain. What happens?

- [ ] It compiles fine — feature flags are just documentation comments to the stable compiler
- [ ] Compile error: `#![feature]` attributes are only permitted on a nightly compiler
- [ ] It compiles but emits a deprecation warning
- [ ] It silently disables the named feature and continues compiling the rest of the file

<details>
<summary>Show Answer</summary>

**Answer:** B — Compile error: `#![feature]` attributes are only permitted on a nightly compiler

**Explanation:** Unstable feature gates are only recognized on the nightly channel; stable and beta compilers reject `#![feature(...)]` with a hard error (`error[E0554]: #![feature] may not be used on the stable release channel`). This is a deliberate stability guarantee — it stops unstable APIs from leaking into code that ships on stable, unlike the tempting but incorrect assumption in A that unknown attributes are just ignored.

</details>
::

::question-wrapper
### Q10. Inside `~/projects/experimental/`, a developer runs `rustup override set nightly`. What is the scope of this change?

- [ ] It permanently changes the global default toolchain for the whole system
- [ ] It only affects that one `cargo build` invocation and resets afterward
- [ ] It pins the `experimental` directory (and its subdirectories) to nightly, leaving the global default and other directories untouched
- [ ] It has no effect unless combined with `rustup default nightly`

<details>
<summary>Show Answer</summary>

**Answer:** C — It pins the `experimental` directory (and its subdirectories) to nightly, leaving the global default and other directories untouched

**Explanation:** `rustup override set` writes a directory-scoped override (tracked internally by `rustup`, independent of `rust-toolchain.toml`) that takes precedence over the global default only within that directory tree. It does not touch `rustup default` (A) — a beginner might assume overrides are global since `rustup default` sounds similar, but the two are deliberately separate mechanisms for different scopes.

</details>
::

::question-wrapper
### Q11. A team upgrades a crate from the 2018 edition to the 2021 edition by editing `edition = "2021"` in `Cargo.toml` with no other changes. What is the safest correct process, and why?

- [ ] Editing the field alone is always sufficient and risk-free since editions are purely cosmetic
- [ ] Editions require a full rewrite in a new crate, since old syntax is entirely rejected
- [ ] Run `cargo fix --edition` first to migrate idiom-affecting changes (e.g. array `IntoIterator` behavior, disjoint closure captures), then set the edition and verify tests pass
- [ ] Editions cannot be upgraded incrementally; you must vendor the entire dependency tree first

<details>
<summary>Show Answer</summary>

**Answer:** C — Run `cargo fix --edition` first to migrate idiom-affecting changes (e.g. array `IntoIterator` behavior, disjoint closure captures), then set the edition and verify tests pass

**Explanation:** Editions are Rust's mechanism for introducing small, deliberately-scoped breaking changes (e.g. `into_iter()` on arrays yielding values instead of references since 2021) without splitting the ecosystem; `cargo fix --edition` automates most of the mechanical migration. Editions are *not* purely cosmetic (A is a tempting but wrong simplification) — real behavioral differences exist, which is exactly why blind field-editing without testing can silently change runtime behavior.

</details>
::

::question-wrapper
### Q12. A repository's root contains a `rust-toolchain.toml` with `channel = "stable"` and `components = ["clippy", "rustfmt"]`. A contributor runs `cargo build` without ever running `rustup component add`. What happens to `clippy`/`rustfmt` availability?

- [ ] Nothing — `rust-toolchain.toml` only affects the channel, never components
- [ ] `rustup` installs the listed components automatically as part of resolving the pinned toolchain for that directory
- [ ] The build fails because required components are missing
- [ ] Components listed in `rust-toolchain.toml` are ignored unless `cargo` is run with `--all-features`

<details>
<summary>Show Answer</summary>

**Answer:** B — `rustup` installs the listed components automatically as part of resolving the pinned toolchain for that directory

**Explanation:** The `[toolchain]` table in `rust-toolchain.toml` can declare `channel`, `components`, `targets`, and `profile`; `rustup` provisions all of them automatically the first time the toolchain is resolved in that directory, so a fresh clone gets a fully working `clippy`/`rustfmt` setup with zero manual steps. This is a widely underused feature — many assume (incorrectly, option A) that the file only pins the compiler version.

</details>
::

::question-wrapper
### Q13. A crate declares `rust-version = "1.70"` (MSRV) in `Cargo.toml`. A contributor on Rust 1.65 stable runs `cargo build`. What happens?

- [ ] `cargo` silently ignores `rust-version` — it's purely informational metadata shown on crates.io, so the build proceeds normally on 1.65
- [ ] Since Rust 1.65 supports MSRV-aware resolution for the `rust-version` field itself in newer cargo versions, and if the code actually uses syntax/APIs from 1.70, compilation fails with errors about unrecognized syntax or missing items — cargo does not downgrade language features for you
- [ ] `cargo` automatically installs Rust 1.70 via `rustup` to satisfy the requirement
- [ ] The build always fails immediately with an MSRV violation error before any compilation is attempted, regardless of what language features are actually used

<details>
<summary>Show Answer</summary>

**Answer:** B — If the code actually uses syntax/APIs from 1.70, compilation fails with errors about unrecognized syntax or missing items; cargo does not downgrade language features for you

**Explanation:** `rust-version` is primarily advisory metadata (and, in newer cargo, can influence dependency version resolution to prefer MSRV-compatible versions), but it does not make cargo auto-install a compiler (C) or hard-fail purely on the declared number (D) — the *actual* failure, if any, comes from the older `rustc` genuinely not understanding newer syntax or standard library items used in the code. **Portability:** MSRV mismatches are a common real-world CI trap — the fix is either raising the contributor's toolchain or avoiding the newer language feature in the crate.

</details>
::

::question-wrapper
### Q14. What happens if you run `rustc` on a `.rs` file that has zero `use` statements and a `fn main() {}` body, with no `Cargo.toml` anywhere nearby?

- [ ] It fails because `rustc` always requires a `Cargo.toml` to know the crate name
- [ ] It compiles successfully — `rustc` can compile a standalone file into a binary using only the standard library, entirely independent of Cargo
- [ ] It fails because `main.rs` must live inside a `src/` directory
- [ ] It compiles but produces a `.rlib` instead of an executable by default

<details>
<summary>Show Answer</summary>

**Answer:** B — It compiles successfully; `rustc` can compile a standalone file into a binary using only the standard library, entirely independent of Cargo

**Explanation:** `rustc` predates and does not require Cargo at all — Cargo is a build-system/package-manager layered on top. A bare `rustc file.rs` with only `std` usage compiles to an executable by default (binary crate type), with no `src/` directory or `Cargo.toml` requirement. Option D confuses the default crate-type: `.rlib` output only happens if you pass `--crate-type=lib` explicitly.

</details>
::

::question-wrapper
### Q15. Which is the recommended way to install Rust on a development machine, and why?

- [ ] Install `rustc` and `cargo` separately via the OS package manager (e.g. `apt install rustc cargo`), since it's pre-integrated with system libraries
- [ ] Use `rustup`, since it manages multiple toolchains/channels, supports per-project overrides, and keeps the toolchain easily updatable independent of the OS release cycle
- [ ] Manually download prebuilt `rustc` binaries from GitHub Releases and add them to PATH
- [ ] Compile `rustc` from source on first install for maximum compatibility

<details>
<summary>Show Answer</summary>

**Answer:** B — Use `rustup`, since it manages multiple toolchains/channels, supports per-project overrides, and keeps the toolchain easily updatable independent of the OS release cycle

**Explanation:** `rustup` is the officially recommended installer specifically because OS package managers (A) often ship an outdated, single, system-wide Rust version tied to the distro's release cadence, making per-project toolchain pinning and quick updates painful. **Idiom:** professional Rust workflows almost universally standardize on `rustup` for exactly this flexibility, reserving OS packages mainly for minimal container images where a single fixed version is acceptable.

</details>
::

::question-wrapper
### Q16. For a team that wants every contributor and CI runner to build with the exact same compiler version automatically, what is the best-practice setup?

- [ ] Document the required version in the README and trust contributors to install it manually
- [ ] Commit a `rust-toolchain.toml` file with a pinned `channel` value at the repository root
- [ ] Add a `rust-version` field to `Cargo.toml` only
- [ ] Require everyone to run `rustup update` before every build

<details>
<summary>Show Answer</summary>

**Answer:** B — Commit a `rust-toolchain.toml` file with a pinned `channel` value at the repository root

**Explanation:** Committing `rust-toolchain.toml` makes toolchain pinning automatic and enforced by tooling rather than relying on documentation discipline (A, easy to go stale) or `rust-version` (C, which is advisory/MSRV-oriented, not a hard pin — it doesn't force a specific compiler, just declares a floor). `rustup update` (D) does the opposite of pinning — it moves everyone to the latest, which can introduce drift between machines.

</details>
::

::question-wrapper
### Q17. A library needs an unstable standard library API only available via `#![feature(...)]` on nightly. What is the idiomatic recommendation for a crate intended for wide production use?

- [ ] Ship it requiring nightly permanently, since nightly is functionally a superset of stable and equally safe for production
- [ ] Avoid the unstable feature and find a stable-compatible alternative (or gate the nightly-only code path behind an optional feature flag), since requiring nightly forces every downstream consumer onto an unstable, less API-stable compiler
- [ ] Vendor a copy of the nightly compiler inside the crate's repository
- [ ] Silently use the feature without any `#![feature(...)]` gate since stable compilers permit unstable APIs on crates.io

<details>
<summary>Show Answer</summary>

**Answer:** B — Avoid the unstable feature and find a stable-compatible alternative (or gate the nightly-only code path behind an optional feature flag), since requiring nightly forces every downstream consumer onto an unstable, less API-stable compiler

**Explanation:** Widely-used production crates generally avoid hard nightly requirements because nightly's unstable APIs can change or disappear between releases with no deprecation guarantee, unlike stable's strict backward-compatibility promise. Option A is the tempting trap: nightly *does* include everything stable has, but "superset" doesn't mean "equally safe" — the unstable portion is explicitly unguaranteed. Option D is simply false: `#![feature]` gates are mandatory and stable rejects them outright (see Q9).

</details>
::

::question-wrapper
### Q18. When Rust ships a new edition (e.g. 2024), what is the recommended migration workflow for an existing crate?

- [ ] Immediately bump `edition` in `Cargo.toml` with no other action, then fix whatever compile errors appear
- [ ] Never adopt new editions — staying on the original edition forever is the safest long-term choice
- [ ] Run `cargo fix --edition` on the current edition first to apply automated migrations, review the diff, then update `edition` in `Cargo.toml`, and finally run the full test suite
- [ ] Rewrite the crate from scratch targeting the new edition

<details>
<summary>Show Answer</summary>

**Answer:** C — Run `cargo fix --edition` on the current edition first to apply automated migrations, review the diff, then update `edition` in `Cargo.toml`, and finally run the full test suite

**Explanation:** `cargo fix --edition` is purpose-built to mechanically apply the safe, known migrations for the *next* edition before you actually switch, minimizing manual breakage — jumping straight to editing the field (A) skips this safety net and can leave silent behavioral differences uncaught by the compiler (e.g., changes to closure capture or iterator semantics that compile fine both ways but behave differently). Staying on an old edition forever (B) is a legitimate but conservative choice, not the generally recommended one, since crates interop across editions within a dependency graph anyway.

</details>
::

::question-wrapper
### Q19. What is the idiomatic way to check code style and common mistakes beyond what `rustc` itself reports, before opening a pull request?

- [ ] Rely solely on `cargo build` warnings, since `rustc` already reports every stylistic and logic issue
- [ ] Run `cargo fmt` to format and `cargo clippy` to lint, both installed via `rustup component add rustfmt clippy`
- [ ] Manually re-read the diff for style issues; automated linting is unnecessary in Rust
- [ ] Use `rustc --lint-all` as a single command that replaces both formatting and linting

<details>
<summary>Show Answer</summary>

**Answer:** B — Run `cargo fmt` to format and `cargo clippy` to lint, both installed via `rustup component add rustfmt clippy`

**Explanation:** `rustc`'s built-in warnings focus on correctness/soundness, not style or idiom; `clippy` catches a much broader class of stylistic, performance, and common-mistake lints (e.g. needless clones, `.unwrap()` on `Result` in non-test code), while `rustfmt` enforces consistent formatting. `rustc --lint-all` (D) is not a real flag — a plausible-sounding distractor for anyone assuming the compiler alone covers everything.

</details>
::

::question-wrapper
### Q20. A developer notices `cargo build` (debug profile) produces a binary that runs noticeably slower than `cargo build --release`. What best-practice conclusion should they draw?

- [ ] This indicates a compiler bug and should be reported immediately
- [ ] Debug builds intentionally skip most optimizations and include extra runtime checks (like overflow checks) to prioritize fast compile times and better debugging; use `--release` for performance testing and production artifacts, and the default debug profile for day-to-day development iteration
- [ ] `--release` only changes the output file name, not actual compiled behavior
- [ ] Debug and release builds are functionally identical except for binary size

<details>
<summary>Show Answer</summary>

**Answer:** B — Debug builds intentionally skip most optimizations and include extra runtime checks (like overflow checks) to prioritize fast compile times and better debugging; use `--release` for performance testing and production artifacts, and the default debug profile for day-to-day development iteration

**Explanation:** The `dev` (default) profile favors compile speed and debuggability (`opt-level = 0`, overflow checks on, debug symbols included), while `release` enables optimizations (`opt-level = 3` by default) and disables overflow checks, trading compile time for runtime speed. **Performance:** always benchmark and ship with `--release` — never draw performance conclusions, and never ship to production, using unoptimized debug builds, since the "slowness" observed here is expected behavior, not a defect (ruling out option A).

</details>
::
