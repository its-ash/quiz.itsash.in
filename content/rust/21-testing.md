# 21 — Testing

::question-wrapper{language="rust"}
### Q1. What does the `#[test]` attribute do?

::code-wrapper{language="rust"}
```rust
#[test]
fn adds_two() {
    assert_eq!(2 + 2, 4);
}
```
::

- [ ] It marks the function to run automatically before `main`
- [ ] It marks the function as a test case that `cargo test` discovers and runs as its own isolated execution
- [ ] It disables the function in release builds only
- [ ] It generates documentation from the function's body

<details>
<summary>Show Answer</summary>

**Answer:** B — It marks the function as a test case that `cargo test` discovers and runs as its own isolated execution

**Explanation:** `#[test]` registers a function with the built-in test harness; `cargo test` compiles a special test binary that runs every `#[test]`-annotated function, each in its own thread by default, reporting pass/fail per test. It has nothing to do with `main` startup order (rules out A) — test binaries don't even have the crate's normal `main`. It's excluded from *normal* (non-test) builds entirely, not just release builds specifically (rules out C, `#[test]` code is compiled out unless `cfg(test)` is active, which happens for `cargo test` regardless of debug/release). Doc generation is `///` doc comments, unrelated to `#[test]` (rules out D).

</details>
::

::question-wrapper
### Q2. Why is `#[cfg(test)]` commonly placed on a `mod tests { ... }` block containing unit tests?

- [ ] It makes the tests run in parallel instead of sequentially
- [ ] It ensures the test module (and everything inside it, including test-only imports) is compiled only when building for testing, not in normal builds
- [ ] It is required syntax for any function using `assert_eq!`
- [ ] It hides the test results from the terminal output

<details>
<summary>Show Answer</summary>

**Answer:** B — It ensures the test module (and everything inside it, including test-only imports) is compiled only when building for testing, not in normal builds

**Explanation:** `#[cfg(test)]` is conditional compilation: the annotated item only exists in builds where the `test` cfg flag is set, which `cargo test` sets automatically. Wrapping unit tests in a `#[cfg(test)] mod tests { use super::*; ... }` block means test code, test-only dependencies, and test helper functions never bloat or ship in the production binary. It's unrelated to test parallelism, which is a `cargo test` runtime behavior controlled separately (rules out A, see also the test-isolation question later in this file). `assert_eq!` works in any function regardless of `cfg` (rules out C). Output visibility is controlled by `cargo test` flags like `--nocapture`, not by `cfg(test)` (rules out D).

</details>
::

::question-wrapper
### Q3. Where do Rust integration tests live, and how do they differ from unit tests in `src/`?

- [ ] In `src/tests/`, and they have access to private items just like unit tests
- [ ] In a top-level `tests/` directory (sibling to `src/`); each file there is compiled as its own separate crate that can only call the library's `pub` API, not its private internals
- [ ] Integration tests and unit tests are the same thing in Rust — there's no distinction
- [ ] In `Cargo.toml`, as declarative `[[test]]` blocks with no Rust code

<details>
<summary>Show Answer</summary>

**Answer:** B — In a top-level `tests/` directory (sibling to `src/`); each file there is compiled as its own separate crate that can only call the library's `pub` API, not its private internals

**Explanation:** Cargo automatically treats every `.rs` file directly inside `tests/` as an independent test crate that depends on and links against your library crate exactly like an external consumer would — meaning it exercises only the public API, which is valuable for catching "this only works because a test reaches into internals" bugs. A is wrong about location and about access — putting a file in `src/tests/` doesn't grant it the "separate crate, public-API-only" property that defines integration tests; that's purely a `tests/`-directory behavior. C conflates two genuinely different testing layers Rust distinguishes on purpose. D misunderstands the mechanism — no manifest declaration is needed; Cargo's `tests/` convention is filesystem-based and automatic.

</details>
::

::question-wrapper{language="rust"}
### Q4. Do doctests (code examples in `///` documentation comments) actually get executed?

::code-wrapper{language="rust"}
```rust
/// Adds one to the given number.
///
/// ```
/// let x = my_crate::add_one(2);
/// assert_eq!(x, 3);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```
::

- [ ] No — they're purely illustrative text, never compiled or run
- [ ] Yes — `cargo test` compiles and runs every fenced code block in a doc comment as its own test, unless the block is explicitly marked otherwise (e.g. ` ```ignore `)
- [ ] They're only checked for correct Markdown syntax, not compiled
- [ ] They run only when `cargo doc` is invoked, never during `cargo test`

<details>
<summary>Show Answer</summary>

**Answer:** B — Yes — `cargo test` compiles and runs every fenced code block in a doc comment as its own test, unless the block is explicitly marked otherwise (e.g. ` ```ignore `)

**Explanation:** Doctests are real, executable tests: `cargo test` extracts each ` ``` ` fenced block from doc comments, wraps it in an implicit `fn main()` if needed, compiles it as its own tiny binary linked against the crate's public API, and runs it — an `assert_eq!` failure or panic inside one fails the test suite just like a `#[test]` function would. This is a genuine gotcha for newcomers who assume doc examples are "just comments" (option A) and are surprised when a stale example breaks CI. Fences can opt out with annotations like ` ```ignore ` or ` ```text ` (rules out the idea that all blocks always run unconditionally, though the default is that they do). It's not merely a syntax/lint check (rules out C), and doctests run under `cargo test`, independent of whether `cargo doc` is ever invoked (rules out D).

</details>
::

::question-wrapper
### Q5. What does `#[should_panic]` do when applied to a `#[test]` function?

- [ ] It suppresses panic output so the test suite doesn't print a stack trace
- [ ] It asserts the test function must panic for the test to be considered passing; if the function returns normally, the test fails
- [ ] It marks the test as expected to fail and skips it
- [ ] It catches the panic and converts it into a `Result::Err`

<details>
<summary>Show Answer</summary>

**Answer:** B — It asserts the test function must panic for the test to be considered passing; if the function returns normally, the test fails

**Explanation:** `#[should_panic]` inverts the usual pass condition: the test harness runs the function expecting a panic, marks the test as passed if one occurs, and — importantly — marks it as *failed* if the function completes without panicking. It's commonly paired with `expected = "substring"` to also verify the panic message matches, catching the case where the function panics for the wrong reason. It doesn't just silence output (rules out A) — panic details still print by default so you can see what happened. It's not the same as `#[ignore]`, which actually skips a test rather than expecting a panic (rules out C). And it doesn't convert anything into `Result` — that's a different, non-panicking test convention (`fn test() -> Result<(), String>`) entirely (rules out D).

</details>
::

::question-wrapper
### Q6. How does `cargo test` run multiple test functions by default?

- [ ] Sequentially, one at a time, in the order they appear in the source file
- [ ] In parallel across multiple threads, so tests should not assume exclusive access to shared external state (files, env vars, ports) unless they coordinate
- [ ] In a random, single-threaded order
- [ ] All tests run in one shared thread but interleaved via async cooperative scheduling

<details>
<summary>Show Answer</summary>

**Answer:** B — In parallel across multiple threads, so tests should not assume exclusive access to shared external state (files, env vars, ports) unless they coordinate

**Explanation:** By default, `cargo test` spins up a thread pool and runs test functions concurrently for speed, which is exactly why tests that mutate shared global state (a file on disk, a process-wide environment variable, a fixed network port, a `static` with interior mutability) can flake or corrupt each other unless they're made independent or explicitly serialized (e.g. `--test-threads=1`, or a crate like `serial_test`). This surprises people coming from test runners that default to sequential execution (option A). It's not randomized-but-single-threaded (rules out C), and it's plain OS-thread parallelism, not async scheduling (rules out D) — `cargo test`'s default harness has no async runtime involved.

</details>
::

::question-wrapper{language="rust"}
### Q7. What is the simplest way to check that a function returns the expected value in a `#[test]`?

::code-wrapper{language="rust"}
```rust
#[test]
fn parses_valid_input() {
    let result = parse("42");
    assert_eq!(result, Ok(42));
}
```
::

- [ ] `assert_eq!`, which panics (failing the test) with a diff of the left and right values if they aren't equal
- [ ] `println!`, since printed output alone fails the test if incorrect
- [ ] `if result != Ok(42) { return; }`, which cargo interprets as a failure
- [ ] `#[test]` functions cannot assert on return values, only on side effects

<details>
<summary>Show Answer</summary>

**Answer:** A — `assert_eq!`, which panics (failing the test) with a diff of the left and right values if they aren't equal

**Explanation:** The test harness considers a test failed if the function panics (or, for the `Result`-returning test convention, if it returns `Err`); `assert_eq!`/`assert_ne!`/`assert!` are macros that panic with a helpful message (showing both compared values for `assert_eq!`) when the condition doesn't hold, which is exactly what drives pass/fail. `println!` output alone is inert — the harness doesn't parse printed text to decide pass/fail, it only reacts to a panic or early `Err` (rules out B). Silently `return`-ing early does not signal failure to the harness at all — the test would report as passed (rules out C), which is itself a subtle pitfall to watch for in hand-rolled test logic. Return-value assertions are the most common thing tests do; nothing prevents it (rules out D).

</details>
::

::question-wrapper{language="rust"}
### Q8. What happens when a `#[test]` function is empty (no assertions, no panics, just an empty body)?

::code-wrapper{language="rust"}
```rust
#[test]
fn todo_write_this_test() {}
```
::

- [ ] `cargo test` fails it, requiring at least one assertion
- [ ] It reports as passing — an empty function neither panics nor returns `Err`, so the harness has no reason to consider it failed, even though it verifies nothing
- [ ] It's a compile error — `#[test]` functions must contain at least one `assert!`
- [ ] `cargo test` marks it `ignored` automatically since it does nothing

<details>
<summary>Show Answer</summary>

**Answer:** B — It reports as passing — an empty function neither panics nor returns `Err`, so the harness has no reason to consider it failed, even though it verifies nothing

**Explanation:** The test harness's pass criterion is purely "did this function panic or return `Err`" — it has no concept of "did this test actually check anything." A stub test left behind as a TODO will happily report green forever, which is a genuine production trap: a passing test suite can hide untested code paths behind empty or assertion-less test functions. Nothing in the language or `cargo test` enforces a minimum number of assertions (rules out A and C — there's no such compile-time requirement). `#[ignore]` is an explicit, separate opt-out attribute a developer must add themselves; it is never inferred from an empty body (rules out D).

</details>
::

::question-wrapper
### Q9. Two tests both write to the same hardcoded file path, `/tmp/output.txt`, then assert on its contents. Running `cargo test` (default parallel execution) shows intermittent, non-deterministic failures. Why?

- [ ] `cargo test` corrupts the filesystem when run in parallel
- [ ] The two tests race on the shared file — running concurrently on different threads, one test's write can interleave with or overwrite the other's before its own assertion reads the file back
- [ ] Rust's file I/O is not thread-safe at the language level, causing undefined behavior
- [ ] `assert_eq!` is not reentrant across threads

<details>
<summary>Show Answer</summary>

**Answer:** B — The two tests race on the shared file — running concurrently on different threads, one test's write can interleave with or overwrite the other's before its own assertion reads the file back

**Explanation:** This is the canonical test-isolation pitfall: tests are expected to be independent, but a hardcoded shared resource (a fixed file path, a fixed port, a shared env var, a `static` counter) breaks that independence, and default parallel test execution turns the resulting race into a flaky, order-dependent failure that's hard to reproduce. **Debug**: fixes include giving each test a unique temp path (e.g. via `tempfile` or a name derived from the test), serializing the offending tests (`--test-threads=1` or `#[serial]` from `serial_test`), or restructuring the test to avoid shared mutable external state entirely. Filesystem operations themselves aren't corrupted by concurrency at the OS level (rules out A); this isn't a language-level data race or UB — `std::fs` calls are ordinary syscalls, safe to call from multiple threads, they just aren't automatically *coordinated* for you (rules out C); and `assert_eq!` is a plain macro with no reentrancy concept — the race is entirely in the shared file, not in the assertion (rules out D).

</details>
::

::question-wrapper
### Q10. What does `#[ignore]` do on a test function, and how do you run ignored tests?

- [ ] It permanently disables the test; there is no way to run it via `cargo test`
- [ ] It excludes the test from the default `cargo test` run, but it can still be run explicitly with `cargo test -- --ignored`
- [ ] It's identical to deleting the `#[test]` attribute
- [ ] It marks the test as expected to fail, similar to `#[should_panic]`

<details>
<summary>Show Answer</summary>

**Answer:** B — It excludes the test from the default `cargo test` run, but it can still be run explicitly with `cargo test -- --ignored`

**Explanation:** `#[ignore]` is meant for tests that are valid but too slow, environment-dependent, or otherwise unsuitable to run on every default `cargo test` invocation (e.g. tests that hit a real network service); they're skipped by default but remain runnable on demand via `cargo test -- --ignored` (or `--include-ignored` to run everything). It's not a permanent disable (rules out A) and not equivalent to removing `#[test]` entirely, since the function is still recognized and reported as "ignored" rather than simply not existing as a test (rules out C). It has nothing to do with expecting a panic — that's `#[should_panic]`'s job, and the two attributes address unrelated concerns (rules out D).

</details>
::

::question-wrapper
### Q11. A test function is written as `fn returns_config() -> Result<(), String> { ... }` without `#[should_panic]`. What determines whether this test passes?

- [ ] It never passes, because `#[test]` functions must return `()`
- [ ] The harness treats `Ok(())` as pass and `Err(_)` as fail, letting the test use `?` to propagate errors instead of `unwrap()`-triggered panics
- [ ] Only the presence of the `#[test]` attribute matters; the return type is ignored
- [ ] It always fails at compile time because `Result` cannot appear in test signatures

<details>
<summary>Show Answer</summary>

**Answer:** B — The harness treats `Ok(())` as pass and `Err(_)` as fail, letting the test use `?` to propagate errors instead of `unwrap()`-triggered panics

**Explanation:** Since Rust 2018, `#[test]` functions may return any type implementing `std::process::Termination`, most commonly `Result<(), E>` where `E: Debug` — this lets test bodies use `?` on fallible operations and get a clean `Err` printout on failure instead of a panic from `.unwrap()`. `()`-returning tests are still the common case, but they aren't the only option (rules out A and D, both of which describe restrictions that don't exist). The return type is very much inspected by the harness to decide pass/fail — it isn't ignored (rules out C).

</details>
::

::question-wrapper
### Q12. What happens to output from `println!` inside a passing test, by default?

- [ ] It always prints immediately to the terminal
- [ ] The test harness captures (suppresses) stdout for passing tests by default; it's only shown for tests that fail, unless `--nocapture` is passed
- [ ] It's written to a log file instead of the terminal
- [ ] `println!` cannot be used inside `#[test]` functions at all

<details>
<summary>Show Answer</summary>

**Answer:** B — The test harness captures (suppresses) stdout for passing tests by default; it's only shown for tests that fail, unless `--nocapture` is passed

**Explanation:** By default `cargo test` captures each test's stdout/stderr and only surfaces it for tests that fail, keeping a clean summary for a large, mostly-passing suite; debugging with stray `println!`s that never appear is a common early confusion (assumption A). Passing `cargo test -- --nocapture` disables this capturing so all output streams live regardless of pass/fail. There's no log-file redirection involved (rules out C), and `println!` works perfectly fine inside tests — it's simply capture behavior, not a restriction on the macro itself (rules out D).

</details>
::

::question-wrapper
### Q13. A unit test module does `use super::*;` to access the parent module's private items. Why does this work even though those items have no `pub` modifier?

- [ ] `#[cfg(test)]` implicitly makes every item in the crate `pub`
- [ ] The `tests` submodule is a *child* of the module it's testing, and Rust's privacy rule makes a module's private items visible to all of its descendants — the test module included
- [ ] `use super::*;` bypasses privacy checks as a special case for testing
- [ ] It only works if every tested item is additionally marked `pub(crate)`

<details>
<summary>Show Answer</summary>

**Answer:** B — The `tests` submodule is a *child* of the module it's testing, and Rust's privacy rule makes a module's private items visible to all of its descendants — the test module included

**Explanation:** This is the same general privacy rule covered for modules generally: a private item is visible in its defining module and every module nested inside it, and `mod tests { ... }` declared inside the module under test is exactly such a descendant — so `use super::*;` can name and call private functions without any special-case testing behavior. This is precisely why unit tests conventionally live in a `#[cfg(test)] mod tests` *inside* the same file as the code, while integration tests in `tests/` (a separate crate) deliberately cannot see private items — different levels of the module tree, different visibility outcomes. `#[cfg(test)]` only controls conditional compilation, not privacy (rules out A). There's no special glob-import privacy bypass (rules out C); `use` never overrides visibility, it can only bring already-visible items into scope. And no extra `pub(crate)` is required — plain private is already sufficient for a descendant module (rules out D).

</details>
::

::question-wrapper
### Q14. In an integration test file under `tests/`, why does `use my_crate::internal_helper;` fail to compile if `internal_helper` is a `pub(crate)` (not `pub`) function in the library?

- [ ] `tests/` files cannot use `use` statements at all
- [ ] Each file in `tests/` is compiled as an entirely separate crate consuming the library through its public API only; `pub(crate)` explicitly excludes visibility outside the defining crate, and the integration test is, from the compiler's perspective, a different crate
- [ ] `pub(crate)` items are visible to integration tests but not unit tests
- [ ] It's a typo-only issue; `pub(crate)` and `pub` are otherwise identical

<details>
<summary>Show Answer</summary>

**Answer:** B — Each file in `tests/` is compiled as an entirely separate crate consuming the library through its public API only; `pub(crate)` explicitly excludes visibility outside the defining crate, and the integration test is, from the compiler's perspective, a different crate

**Explanation:** This directly follows from how `tests/` is set up (Q3): each file there is its own crate linked against the library the way any external consumer would be, so it is bound by the same rules any other downstream crate faces — `pub(crate)` items are, by definition, invisible past the crate boundary. This is actually a *feature*: it forces integration tests to exercise the same API surface real users get, catching "only works via internals" bugs. `use` is completely normal and necessary in `tests/` files (rules out A). Visibility rules aren't different for integration vs. unit tests as a special case — it's a direct consequence of crate boundaries, and it's the reverse of C (integration tests see *less*, not more, than unit tests). `pub(crate)` and `pub` differ precisely in this cross-crate reachability, which is the whole point, not a typo (rules out D).

</details>
::

::question-wrapper
### Q15. What's the idiomatic way to organize tests that need expensive shared setup (e.g., spinning up an in-memory database) without letting one test's mutations leak into another's results?

- [ ] Use a single `static mut` database instance shared by all tests for efficiency
- [ ] Have each test construct its own fresh, independent instance of the resource (or reset state at the start of each test), so tests remain independent even when run in parallel
- [ ] Force all tests to run with `--test-threads=1` permanently as the default workaround
- [ ] Put all setup logic in `#[test] fn setup()` and rely on test execution order to run it first

<details>
<summary>Show Answer</summary>

**Answer:** B — Have each test construct its own fresh, independent instance of the resource (or reset state at the start of each test), so tests remain independent even when run in parallel

**Explanation:** **Idiom**: the healthiest fix for shared-state flakiness is to make each test self-contained (its own in-memory DB instance, its own temp directory, its own fixture) rather than fighting the test runner's default concurrency — this keeps the suite fast and each test's failure meaningful in isolation. `static mut` (A) is both a legacy `unsafe`-only construct and reintroduces exactly the shared-mutable-state race this question is about — it is not a fix. Forcing single-threaded execution everywhere (C) sacrifices the speed benefit of parallel tests project-wide just to paper over a design issue in a few tests. Relying on a test literally named/ordered to run "first" (D) doesn't work — `cargo test`'s execution order isn't guaranteed or controllable that way, and treating a `#[test]` function as a setup hook is not a supported pattern (use a helper function called from each test, or a `OnceLock`/fixture pattern instead).

</details>
::

::question-wrapper
### Q16. Which is the more idiomatic assertion style for a test that must produce a clear failure message when comparing two computed values?

- [ ] `if a != b { panic!("failed"); }`
- [ ] `assert_eq!(a, b, "computed value did not match expected for input {:?}", input);` — using the built-in comparison macro with an optional custom context message
- [ ] `assert!(format!("{:?}", a) == format!("{:?}", b));`
- [ ] Silently allowing the test to continue if `a != b`, and checking a log file afterward

<details>
<summary>Show Answer</summary>

**Answer:** B — `assert_eq!(a, b, "computed value did not match expected for input {:?}", input);` — using the built-in comparison macro with an optional custom context message

**Explanation:** **Idiom**: `assert_eq!`/`assert_ne!` are preferred over a hand-rolled `if`/`panic!` because they automatically print both compared values in a readable diff on failure, and they accept an optional trailing format string for extra context — combining both gives the clearest failure output with the least code. A hand-rolled `panic!("failed")` (A) throws away the actual values that mismatched, making debugging a failure much harder. Comparing via `format!("{:?}", ...)` string equality (C) is a needless workaround — it obscures the real values behind their debug-formatted strings and loses type-level comparison semantics (e.g. it would consider `-0.0` and `0.0` different if their `Debug` output differs, or `NaN` cases mismatch bizarrely). Silently continuing on a mismatch (D) defeats the entire purpose of a test.

</details>
::

::question-wrapper
### Q17. Why is it generally considered best practice for unit tests to avoid depending on wall-clock time or real network calls?

- [ ] Rust's test harness physically disallows network access during `cargo test`
- [ ] Such dependencies make tests slow, flaky (affected by network conditions, clock skew, timing races), and non-reproducible across environments and CI runs — prefer injecting a fake clock/mock service or testing pure logic separately from I/O
- [ ] `assert_eq!` cannot compare `std::time::Instant` values
- [ ] Only integration tests are allowed to perform I/O; unit tests are compiled without network access

<details>
<summary>Show Answer</summary>

**Answer:** B — Such dependencies make tests slow, flaky (affected by network conditions, clock skew, timing races), and non-reproducible across environments and CI runs — prefer injecting a fake clock/mock service or testing pure logic separately from I/O

**Explanation:** **Idiom**: tests that reach out over the real network or depend on precise timing are a leading cause of CI flakiness — a temporary DNS blip, a slow CI runner, or a shared clock jitter can fail a test that has nothing wrong with the code under test. The standard fix is dependency injection (pass in a trait object or fake clock/service implementation for tests) so the logic under test is deterministic, with any real I/O covered separately by a smaller number of integration tests that explicitly accept that cost. There is no language- or harness-level network block during `cargo test` (rules out A and D — nothing stops a test from making a real HTTP call, which is exactly the problem). `assert_eq!` works on any `PartialEq + Debug` type including `Instant` (rules out C) — the issue isn't comparability, it's non-determinism.

</details>
::

::question-wrapper
### Q18. What is a common pitfall of writing `#[test]` functions that call `.unwrap()` liberally on `Result`/`Option` values from the code under test?

- [ ] `.unwrap()` is banned inside `#[test]` functions by the compiler
- [ ] It works fine for surfacing failures as panics, but the failure message on an `Err`/`None` can be uninformative compared to `expect("context")` or a `Result`-returning test with `?`, making it harder to diagnose *why* a test failed from CI logs alone
- [ ] `.unwrap()` silently converts errors into passing tests
- [ ] It causes tests to run in a different thread than normal

<details>
<summary>Show Answer</summary>

**Answer:** B — It works fine for surfacing failures as panics, but the failure message on an `Err`/`None` can be uninformative compared to `expect("context")` or a `Result`-returning test with `?`, making it harder to diagnose *why* a test failed from CI logs alone

**Explanation:** **Idiom**: `.unwrap()` does correctly fail the test (any panic fails it), but its default panic message is generic (`called Result::unwrap() on an Err value: ...`), whereas `.expect("parsing config from valid TOML should not fail")` or restructuring the test as `fn test() -> Result<(), Error> { ...; Ok(()) }` with `?` gives future-you (or a teammate reading a CI log at 2am) immediate context about which operation failed and why, without needing to reproduce locally. `.unwrap()` is not disallowed anywhere in tests (rules out A), it does not swallow errors into a false pass — quite the opposite, it panics loudly (rules out C) — and it has no effect on which thread a test runs on (rules out D).

</details>
::

::question-wrapper
### Q19. Why do many real-world Rust projects prefer to keep expensive, environment-dependent integration tests separate from fast unit tests (e.g., via `#[ignore]`, feature flags, or separate `tests/` binaries), rather than mixing everything into one `cargo test` run?

- [ ] cargo physically cannot compile more than one kind of test in the same crate
- [ ] Keeping the default `cargo test` run fast and hermetic encourages developers to actually run it constantly (tight feedback loop), while slower/environment-dependent tests are opted into explicitly (e.g., in a scheduled CI job) rather than punishing every local test run
- [ ] `#[ignore]` is required by the compiler for any test that takes longer than one second
- [ ] Mixing test types causes silent data corruption in the test binary

<details>
<summary>Show Answer</summary>

**Answer:** B — Keeping the default `cargo test` run fast and hermetic encourages developers to actually run it constantly (tight feedback loop), while slower/environment-dependent tests are opted into explicitly (e.g., in a scheduled CI job) rather than punishing every local test run

**Explanation:** **Idiom**: a test suite that takes 30 seconds because of a handful of slow, flaky, or environment-coupled tests trains developers to stop running it locally, which defeats the entire purpose of fast feedback — separating "always run this" from "run this in CI or on demand" (via `#[ignore]`, a Cargo feature flag, or a distinct `tests/` binary invoked separately) keeps the fast path fast without deleting valuable slower coverage. This is a process/workflow best practice, not a compiler limitation — nothing stops you from mixing test kinds in one crate (rules out A and D, neither of which describes any real constraint), and there's no enforced timing threshold that mandates `#[ignore]` — it's a judgment call developers make deliberately (rules out C).

</details>
::

::question-wrapper
### Q20. A test suite has one slow test that takes 45 seconds due to a large fixture, buried among hundreds of millisecond-scale unit tests, and it isn't marked `#[ignore]`. What's the idiomatic fix, and why not just delete the slow test?

- [ ] Delete it — a slow test is never worth keeping
- [ ] Mark it `#[ignore]` (with a comment explaining why), or move it to a `tests/` integration binary run separately in CI, so the fast default suite stays quick while the valuable coverage isn't lost — deleting it would remove real regression protection just to fix a workflow annoyance
- [ ] Wrap its body in `#[should_panic]` so it fails fast instead of running to completion
- [ ] Rename the function so `cargo test` alphabetically runs it last

<details>
<summary>Show Answer</summary>

**Answer:** B — Mark it `#[ignore]` (with a comment explaining why), or move it to a `tests/` integration binary run separately in CI, so the fast default suite stays quick while the valuable coverage isn't lost — deleting it would remove real regression protection just to fix a workflow annoyance

**Explanation:** **Idiom**: the test itself may well be catching real bugs (large-input behavior, performance regressions, resource-heavy edge cases) — the actual problem is the *default local workflow* getting slower, not that the test lacks value. `#[ignore]` (paired with `cargo test -- --ignored` in a dedicated CI job or nightly run) or relocating it to its own `tests/` binary that CI invokes on a different cadence both preserve the coverage while keeping the everyday `cargo test` loop fast. Deleting it outright (A) throws away a real regression check to solve what's really a scheduling/workflow problem. `#[should_panic]` (C) changes the test's pass condition to "must panic," which is unrelated to speed and would be actively wrong if the test isn't supposed to panic. `cargo test`'s execution order isn't reliably alphabetical or otherwise developer-controlled, and even if it were, running last doesn't make a 45-second test any faster or less disruptive to the overall run time (rules out D).

</details>
::
