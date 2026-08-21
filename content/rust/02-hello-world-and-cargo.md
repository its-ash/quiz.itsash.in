# 02 — Hello World & Cargo

::question-wrapper
### Q1. What does `cargo new my_app` create by default?

- [ ] Only a `Cargo.toml` file with no source directory
- [ ] A `src/main.rs` binary crate, a `Cargo.toml`, and a git repository (if not already inside one)
- [ ] A `src/lib.rs` library crate and a `Cargo.lock` only
- [ ] An empty directory that must be populated manually with `cargo init --full`

<details>
<summary>Show Answer</summary>

**Answer:** B — A `src/main.rs` binary crate, a `Cargo.toml`, and a git repository (if not already inside one)

**Explanation:** `cargo new` scaffolds a binary crate by default (`src/main.rs` with a `Hello, world!` program), generates `Cargo.toml`, and also runs `git init` plus a `.gitignore` unless one already exists in a parent directory or `--vcs none` is passed. Use `cargo new --lib` to get a `src/lib.rs` library crate instead (C describes the wrong default).

</details>
::

::question-wrapper
### Q2. What is the key functional difference between `cargo build` and `cargo check`?

- [ ] `cargo check` only validates `Cargo.toml` syntax; it does not touch source files
- [ ] `cargo check` runs the full borrow checker and type checker like `cargo build`, but skips code generation/linking, making it much faster for catching compile errors
- [ ] `cargo build` is faster because it skips borrow checking, which only `cargo check` performs
- [ ] They are functionally identical; `cargo check` is simply a deprecated alias

<details>
<summary>Show Answer</summary>

**Answer:** B — `cargo check` runs the full borrow checker and type checker like `cargo build`, but skips code generation/linking, making it much faster for catching compile errors

**Explanation:** Both commands run the same front-end analysis (type checking, borrow checking, trait resolution), so `cargo check` catches the same compile errors as `cargo build`. The difference is that `check` stops before LLVM codegen and linking, which are the most time-consuming stages, so it's dramatically faster for a tight edit-check loop. Option C inverts this — borrow checking is not something `build` skips.

</details>
::

::question-wrapper
### Q3. What is the correct relationship between `Cargo.toml` and `Cargo.lock`?

- [ ] `Cargo.toml` is auto-generated from `Cargo.lock` on every build
- [ ] `Cargo.toml` declares dependency requirements (often with version ranges); `Cargo.lock` records the exact resolved versions actually used, ensuring reproducible builds
- [ ] They serve the same purpose and either one alone is sufficient
- [ ] `Cargo.lock` is only created for library crates, never for binaries

<details>
<summary>Show Answer</summary>

**Answer:** B — `Cargo.toml` declares dependency requirements (often with version ranges); `Cargo.lock` records the exact resolved versions actually used, ensuring reproducible builds

**Explanation:** `Cargo.toml` is the human-edited manifest expressing acceptable version ranges (e.g. `serde = "1.0"`); `Cargo.lock` is generated/updated by Cargo and pins the exact versions (and their transitive dependency graph) that were actually resolved, so a second `cargo build` on another machine reproduces the identical dependency tree rather than re-resolving to potentially newer semver-compatible versions. Both binaries and libraries get a `Cargo.lock` when built, though it's typically only *committed to version control* for binaries (option D is wrong on both the "only libraries" claim and the general practice).

</details>
::

::question-wrapper
### Q4. What command compiles and immediately runs a binary crate in one step during development?

- [ ] `cargo build && ./target/debug/app` is the only way; there is no shortcut
- [ ] `cargo run`
- [ ] `cargo exec`
- [ ] `cargo start`

<details>
<summary>Show Answer</summary>

**Answer:** B — `cargo run`

**Explanation:** `cargo run` builds the binary (if needed — it skips rebuilding when nothing changed) and then executes it, forwarding any arguments after `--`. `cargo exec` and `cargo start` (C, D) are not real Cargo subcommands — plausible-sounding distractors modeled after other ecosystems' tooling (e.g. `npm start`).

</details>
::

::question-wrapper
### Q5. A repository has a root `Cargo.toml` containing a `[workspace]` table with `members = ["core", "cli"]`. What does this achieve?

- [ ] It merges `core` and `cli` into a single crate at build time
- [ ] It groups multiple crates so they share a single `Cargo.lock` and `target/` directory, and can be built/tested together with one `cargo build` from the workspace root
- [ ] It has no build effect; `members` is purely documentation
- [ ] It forces `core` and `cli` to be published together as one package to crates.io

<details>
<summary>Show Answer</summary>

**Answer:** B — It groups multiple crates so they share a single `Cargo.lock` and `target/` directory, and can be built/tested together with one `cargo build` from the workspace root

**Explanation:** A Cargo workspace lets related crates share dependency resolution (one `Cargo.lock`) and a build output directory, avoiding duplicate compilation of shared dependencies across crates. Each member remains its own independently publishable crate (contradicting D) — the workspace is an organizational/build-sharing construct, not a merge (contradicting A).

</details>
::

::question-wrapper
### Q6. Where does `cargo build` (default profile) place the compiled binary?

- [ ] `./bin/`
- [ ] `./target/debug/`
- [ ] `./target/release/`
- [ ] Directly in the project root alongside `Cargo.toml`

<details>
<summary>Show Answer</summary>

**Answer:** B — `./target/debug/`

**Explanation:** The default `cargo build` uses the `dev` profile, which outputs to `target/debug/`. Only `cargo build --release` outputs to `target/release/` (C is the release path, a common mix-up). Cargo never places build artifacts in the project root (D) — `target/` is the dedicated, gitignored build directory.

</details>
::

::question-wrapper
### Q7. What is the purpose of the `[dependencies]` section in `Cargo.toml`?

- [ ] It lists development-only tools like `rustfmt` and `clippy`
- [ ] It declares external crates the package needs at compile/runtime, with version requirements
- [ ] It lists the Rust toolchain version required to build the project
- [ ] It configures which target platforms the binary can run on

<details>
<summary>Show Answer</summary>

**Answer:** B — It declares external crates the package needs at compile/runtime, with version requirements

**Explanation:** `[dependencies]` is where crates like `serde` or `tokio` are declared with semver-style version requirements that Cargo resolves against crates.io (or another registry/path/git source). Dev-only tooling dependencies belong in `[dev-dependencies]` (A describes a different section), toolchain version pinning belongs in `rust-toolchain.toml` (C), and target platforms are handled via `--target` flags or `[target.'cfg(...)']` sections, not `[dependencies]` (D).

</details>
::

::question-wrapper
### Q8. A developer deletes `Cargo.lock` from a binary project and runs `cargo build`. The project depends on `rand = "0.8"`. What happens?

- [ ] `cargo build` fails immediately because `Cargo.lock` is required to exist
- [ ] Cargo silently reuses cached resolution from `~/.cargo` with no re-resolution
- [ ] Cargo re-resolves dependency versions from scratch (honoring the semver ranges in `Cargo.toml`) and regenerates a new `Cargo.lock`, which may pick up newer compatible versions than before (e.g. `0.8.5` instead of a previously pinned `0.8.3`)
- [ ] The build always fails because `rand`'s exact version can no longer be determined

<details>
<summary>Show Answer</summary>

**Answer:** C — Cargo re-resolves dependency versions from scratch (honoring the semver ranges in `Cargo.toml`) and regenerates a new `Cargo.lock`, which may pick up newer compatible versions than before

**Explanation:** `Cargo.lock` is not required for the build to succeed — Cargo can always regenerate it by resolving `Cargo.toml`'s version requirements against the registry. **Debug:** this is exactly why deleting a committed lock file for a *binary* project is risky in production contexts — a newer semver-compatible dependency version could introduce a regression or subtle behavior change that wasn't present when the lock was last committed, even though no application code changed.

</details>
::

::question-wrapper
### Q9. In a fresh clone of a workspace with `members = ["core", "cli"]`, running `cargo build -p cli` from the workspace root does what?

- [ ] Builds every member of the workspace regardless of the `-p` flag
- [ ] Fails, because `-p` is not a valid flag for workspaces
- [ ] Builds only the `cli` package (and its path/workspace dependencies like `core`, if `cli` depends on it), skipping unrelated workspace members
- [ ] Builds only `core`, since `-p` selects dependencies rather than the named package

<details>
<summary>Show Answer</summary>

**Answer:** C — Builds only the `cli` package (and its path/workspace dependencies like `core`, if `cli` depends on it), skipping unrelated workspace members

**Explanation:** `-p`/`--package` scopes a workspace command to one member (plus whatever that member transitively needs), which is useful in large workspaces to avoid rebuilding unrelated crates. Without `-p`, `cargo build` at the workspace root builds all members by default — that's the "build everything" behavior in option A, but it only applies when `-p` is *omitted*.

</details>
::

::question-wrapper
### Q10. A project's `Cargo.toml` has `edition = "2021"` but the committed `Cargo.lock` was generated by an older Cargo that used lockfile format version 3. A contributor with a very new Cargo runs `cargo build`. What typically happens?

- [ ] The build always fails outright with an incompatible lockfile error
- [ ] Cargo silently deletes and ignores the old lock file every time
- [ ] Newer Cargo versions can read and work with older lockfile format versions (and may upgrade the format in place on next resolution), so the build generally proceeds without manual intervention
- [ ] The project must be re-initialized with `cargo new` to fix the format mismatch

<details>
<summary>Show Answer</summary>

**Answer:** C — Newer Cargo versions can read and work with older lockfile format versions (and may upgrade the format in place on next resolution), so the build generally proceeds without manual intervention

**Explanation:** Cargo maintains backward compatibility for reading older lock file formats; it's forward compatibility (an old Cargo reading a lock file produced by a much newer Cargo using a newer format) that can actually cause problems. Options A and D describe unnecessarily destructive/drastic reactions to what is normally a non-issue.

</details>
::

::question-wrapper
### Q11. What happens when `cargo run` is invoked in a crate whose source has zero changes since the last successful build?

- [ ] It always recompiles from scratch as a safety measure
- [ ] It skips recompilation (relying on cached fingerprint/timestamp checks in `target/`) and directly executes the existing binary
- [ ] It fails, requiring an explicit `cargo build` first
- [ ] It recompiles only `Cargo.toml` changes, ignoring source files entirely

<details>
<summary>Show Answer</summary>

**Answer:** B — It skips recompilation (relying on cached fingerprint/timestamp checks in `target/`) and directly executes the existing binary

**Explanation:** Cargo tracks fingerprints (mtimes, flags, dependency versions) in `target/` to detect whether a rebuild is actually necessary; if nothing relevant changed, `cargo run` just executes the already-built artifact, making repeated `cargo run` calls during development fast. This incremental behavior is why `cargo run` is safe to invoke frequently rather than something to avoid for fear of unnecessary rebuilds (contradicting A).

</details>
::

::question-wrapper
### Q12. An empty `src/main.rs` file (0 bytes) exists in an otherwise valid Cargo project. What happens on `cargo build`?

- [ ] It compiles successfully, producing a binary that does nothing when run
- [ ] Compile error: a binary crate root must contain a `fn main()` entry point
- [ ] It's treated as a library crate automatically since there's no `main` function
- [ ] Cargo auto-generates a default `fn main() {}` to fill the gap

<details>
<summary>Show Answer</summary>

**Answer:** B — Compile error: a binary crate root must contain a `fn main()` entry point

**Explanation:** For a binary target, Rust requires a `main` function as the entry point (`error[E0601]: main function not found`); an empty file has no such function, so it fails to compile rather than silently producing a no-op executable (A) or being reinterpreted as a library (C — target type is determined by `Cargo.toml`/directory layout, e.g. `src/main.rs` vs `src/lib.rs`, not by file contents).

</details>
::

::question-wrapper
### Q13. In a workspace, `core/Cargo.toml` depends on `serde = "1.0.150"` and `cli/Cargo.toml` depends on `serde = "1.0.190"`. What does Cargo do when resolving the workspace?

- [ ] The build fails because two members request different versions
- [ ] Both versions are compiled and linked into the final binary as separate copies unconditionally
- [ ] Cargo's resolver picks a single version of `serde` satisfying both semver ranges (here, `1.0.190` or newer within `1.x`) to share across the workspace where possible, avoiding duplicate compilation
- [ ] Only the workspace root's own direct dependency version is used, and member-level requirements are ignored

<details>
<summary>Show Answer</summary>

**Answer:** C — Cargo's resolver picks a single version of `serde` satisfying both semver ranges to share across the workspace where possible, avoiding duplicate compilation

**Explanation:** Because both `1.0.150` and `1.0.190` are compatible under Cargo's caret (`^`) semver default, the resolver unifies them to one shared version (the highest that satisfies all constraints) recorded once in `Cargo.lock`, rather than building duplicate copies (which only happens when version *ranges* are genuinely incompatible, e.g. `1.x` vs `2.x`, producing two separate dependency instances rather than a hard failure).

</details>
::

::question-wrapper
### Q14. A `.gitignore` generated by `cargo new` includes `/target`. A teammate accidentally force-adds and commits the `target/` directory anyway. What is the main practical problem?

- [ ] None — committing build artifacts has no downsides since they're just cached output
- [ ] It bloats the repository with large, machine/platform-specific compiled artifacts that don't belong in version control and can go stale relative to source
- [ ] It will cause `cargo build` to fail on other machines due to a directory name conflict
- [ ] It automatically overrides everyone's local `Cargo.lock`

<details>
<summary>Show Answer</summary>

**Answer:** B — It bloats the repository with large, machine/platform-specific compiled artifacts that don't belong in version control and can go stale relative to source

**Explanation:** `target/` contains derived build output (often gigabytes for large projects) that is regenerable, platform-specific, and irrelevant to other contributors — committing it bloats repo size/clone time and can cause confusing stale-artifact issues, which is exactly why `cargo new` gitignores it by default. It doesn't break other machines' builds outright (C) since `cargo build` will simply overwrite/regenerate what it needs, but it is still bad practice.

</details>
::

::question-wrapper
### Q15. What is the idiomatic reason to commit `Cargo.lock` for a binary application but often *not* commit it for a pure library crate published to crates.io?

- [ ] `Cargo.lock` is required for compilation for binaries but forbidden by crates.io for libraries
- [ ] Applications benefit from pinned, reproducible dependency versions across environments (dev/CI/prod), while libraries should generally let downstream consumers resolve dependency versions themselves against their own constraints, so committing a library's lock file provides little benefit and can be misleading
- [ ] There is no difference; the convention is arbitrary tooling trivia with no technical rationale
- [ ] Libraries never have a `Cargo.lock` generated in the first place

<details>
<summary>Show Answer</summary>

**Answer:** B — Applications benefit from pinned, reproducible dependency versions across environments, while libraries should let downstream consumers resolve dependency versions themselves

**Explanation:** A binary's `Cargo.lock` is the actual deployed artifact's dependency snapshot, so pinning it avoids "works on my machine" drift. A library's `Cargo.lock` is not used when the library is pulled in as a dependency elsewhere (the consuming binary's own lock file governs resolution), so committing it mainly matters for the library's own CI/tests, not for downstream consumers — Cargo does still generate one locally either way (ruling out D).

</details>
::

::question-wrapper
### Q16. During active development, a programmer wants the fastest feedback loop for catching type errors without waiting for full binary generation on every edit. What is the best-practice command to run repeatedly?

- [ ] `cargo build --release`, since release mode catches more errors
- [ ] `cargo check`
- [ ] `cargo run`, since it always recompiles fully every time
- [ ] `rustc src/main.rs` directly, bypassing Cargo for speed

<details>
<summary>Show Answer</summary>

**Answer:** B — `cargo check`

**Explanation:** As established in Q2, `cargo check` performs the same error-catching analysis as `cargo build` while skipping the slow codegen/link stages, making it the standard fast-loop command (many editors/IDEs run it automatically on save via `rust-analyzer`). `--release` (A) is slower, not faster, due to heavier optimization passes — the opposite of what's wanted here, and it doesn't catch a superset of errors that debug builds miss.

</details>
::

::question-wrapper
### Q17. What is the recommended way to add a new dependency to a project, versus manually typing a version guess into `Cargo.toml`?

- [ ] Manually edit `Cargo.toml` with a version pulled from memory, since `cargo add` doesn't verify anything
- [ ] Use `cargo add <crate>`, which fetches the current appropriate version from the registry and writes a correct `Cargo.toml` entry automatically, then run a build to update `Cargo.lock`
- [ ] Directly edit `Cargo.lock` by hand since `Cargo.toml` is regenerated from it
- [ ] Copy an entry from an unrelated project's `Cargo.toml` verbatim, regardless of version compatibility

<details>
<summary>Show Answer</summary>

**Answer:** B — Use `cargo add <crate>`, which fetches the current appropriate version from the registry and writes a correct `Cargo.toml` entry automatically, then run a build to update `Cargo.lock`

**Explanation:** `cargo add` (built into modern Cargo) queries the registry for the latest suitable version and writes a properly formatted dependency entry, reducing typos and stale-version guesses versus hand-editing (A). `Cargo.lock` is derived *from* `Cargo.toml`, never the reverse (C inverts the real relationship established in Q3).

</details>
::

::question-wrapper
### Q18. A team wants CI to fail if `Cargo.lock` would change (i.e., to catch cases where `Cargo.toml` allows a new version that hasn't been vetted/committed). What is the idiomatic flag to use?

- [ ] `cargo build --frozen` or `cargo build --locked`, which error out instead of silently updating the lock file if it's out of date
- [ ] `cargo build --no-lock`, which disables lock file usage entirely
- [ ] There is no such mechanism; this must be checked manually by diffing `Cargo.lock` after every CI run
- [ ] `cargo build --offline`, which is unrelated to lock file freshness but is commonly confused with it

<details>
<summary>Show Answer</summary>

**Answer:** A — `cargo build --frozen` or `cargo build --locked`, which error out instead of silently updating the lock file if it's out of date

**Explanation:** `--locked` requires `Cargo.lock` to be up to date and errors otherwise; `--frozen` additionally forbids any network access, combining `--locked` and `--offline`. `--offline` alone (D) only controls network access and does not by itself enforce lock file freshness — a common point of confusion since the two flags are often used together but solve different problems.

</details>
::

::question-wrapper
### Q19. What is the idiomatic way to organize a project that has both a reusable library and a thin CLI binary that uses it?

- [ ] Put all logic in `src/main.rs` as one large file; splitting is unnecessary in Rust
- [ ] Use `src/lib.rs` for the core reusable logic and a thin `src/main.rs` (or `src/bin/*.rs`) that depends on the library crate, enabling the logic to be unit-tested and reused independent of the CLI entry point
- [ ] Duplicate the logic in both `main.rs` and a separate crate to avoid coupling
- [ ] Always split into a full Cargo workspace, even for small single-purpose projects, since workspaces are required for any crate with more than one file

<details>
<summary>Show Answer</summary>

**Answer:** B — Use `src/lib.rs` for the core reusable logic and a thin `src/main.rs` that depends on the library crate, enabling the logic to be unit-tested and reused independent of the CLI entry point

**Explanation:** The "thin binary, fat library" pattern is a widely recommended Rust idiom: a single package can contain both a library target (`src/lib.rs`) and one or more binary targets that consume it, giving you testable, reusable logic without needing a full multi-crate workspace (D overstates when workspaces are actually necessary — they matter for multiple *separately versioned/published* crates, not merely for having more than one file).

</details>
::

::question-wrapper
### Q20. Before committing, a developer wants to be sure their code both compiles cleanly and follows the project's formatting/lint conventions with minimal manual effort. What sequence of commands best reflects idiomatic practice?

- [ ] `cargo run` only — if it runs without crashing, the code is ready to commit
- [ ] `cargo fmt && cargo clippy -- -D warnings && cargo test`
- [ ] Skip all tooling and rely on code review to catch formatting and lint issues
- [ ] `cargo build --release` alone, since release mode implies all checks passed

<details>
<summary>Show Answer</summary>

**Answer:** B — `cargo fmt && cargo clippy -- -D warnings && cargo test`

**Explanation:** This sequence formats code consistently, runs Clippy with warnings promoted to errors (`-D warnings`) to enforce a clean lint baseline, and runs the test suite — a standard pre-commit/CI gate in professional Rust projects. Running successfully (A) says nothing about style, lint hygiene, or correctness beyond the one code path exercised; `--release` (D) says nothing about lints or tests at all, and optimized builds can even mask certain debug-only assertions (like overflow checks), making it a poor substitute for an actual verification pipeline.

</details>
::
