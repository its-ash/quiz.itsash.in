# 19 — Smart Pointers

::question-wrapper{language="rust"}
### Q1. What problem does `Box<T>` primarily solve for a type like this?

::code-wrapper{language="rust"}
```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}
```
::

- [ ] It makes `List` implement `Copy`
- [ ] It gives `List` a known, fixed size at compile time by putting the recursive part on the heap
- [ ] It makes the enum thread-safe
- [ ] It automatically derives `Clone` for `List`

<details>
<summary>Show Answer</summary>

**Answer:** B — It gives `List` a known, fixed size at compile time by putting the recursive part on the heap

**Explanation:** Without indirection, `List` would need to contain a `List` inline, giving it infinite size — the compiler cannot compute `size_of::<List>()`. `Box<List>` stores a single pointer-sized value (the heap address) inline instead, so the enum's size is fixed regardless of how deep the list grows. **Safety**: this is a compile-time size problem, not a runtime one — code with an unboxed recursive variant is rejected before it ever runs. It has nothing to do with `Copy`, thread safety, or `Clone`, none of which `Box` grants automatically.

</details>
::

::question-wrapper
### Q2. What is `Rc<T>` used for?

- [ ] Enabling shared ownership of heap data within a single thread via reference counting
- [ ] Enabling shared mutable access across threads
- [ ] Automatic garbage collection of cyclic data structures
- [ ] Making a type `Copy` instead of `Clone`

<details>
<summary>Show Answer</summary>

**Answer:** A — Enabling shared ownership of heap data within a single thread via reference counting

**Explanation:** `Rc<T>` ("reference counted") lets multiple owners share the same heap allocation; each `clone()` bumps a counter, and the value is dropped once the count reaches zero. **Safety**: `Rc<T>`'s counter increments are not atomic, so it is neither `Send` nor `Sync` — sharing one across threads is a compile error, which rules out B. It does not collect cycles (rules out C, see the leak question later in this file), and it does not affect `Copy`/`Clone` derivation semantics beyond providing its own cheap `Clone` impl.

</details>
::

::question-wrapper
### Q3. Given `let b = Rc::clone(&a);`, what is idiomatic about writing it this way instead of `let b = a.clone();`?

- [ ] `Rc::clone` is faster because it skips the reference count
- [ ] Nothing — they are different operations with different results
- [ ] `Rc::clone(&a)` makes it visually clear at the call site that this is a cheap pointer/refcount clone, not a deep data clone
- [ ] `a.clone()` would move `a` instead of borrowing it

<details>
<summary>Show Answer</summary>

**Answer:** C — `Rc::clone(&a)` makes it visually clear at the call site that this is a cheap pointer/refcount clone, not a deep data clone

**Explanation:** Both calls do exactly the same thing — increment the strong count and return a new `Rc<T>` pointing at the same allocation. **Idiom**: Rust style favors `Rc::clone(&a)` as documentation-by-convention, distinguishing "cheap, shared-ownership clone" from a `T::clone()` that might be an expensive deep copy. It's purely a readability convention, not a performance difference (rules out A), the operations are identical (rules out B), and `a.clone()` takes `&self` so it borrows, it never moves `a` (rules out D).

</details>
::

::question-wrapper
### Q4. When does the value inside an `Rc<T>` actually get dropped?

- [ ] As soon as any single `Rc` handle to it goes out of scope
- [ ] Only when the strong count drops to zero
- [ ] Immediately when `Rc::clone` is called
- [ ] Never — `Rc` values are leaked by design

<details>
<summary>Show Answer</summary>

**Answer:** B — Only when the strong count drops to zero

**Explanation:** Every `Rc` clone increments a shared strong count; dropping any one handle decrements it, and the inner value's destructor only runs when that count hits zero. A is the tempting-but-wrong beginner assumption — it confuses "one handle drops" with "the value drops," which only coincide when there was exactly one owner left. `Rc::clone` increments, it never triggers a drop (rules out C), and `Rc` values are reclaimed normally (rules out D) — unless a reference cycle prevents the count from ever reaching zero, which is a separate leak scenario covered later.

</details>
::

::question-wrapper
### Q5. What does `RefCell<T>` provide that a plain `T` behind a shared reference does not?

- [ ] Compile-time enforcement of Rust's borrow rules, just faster
- [ ] Interior mutability — the ability to mutate data through a shared (`&`) reference, with borrow rules checked at runtime instead of compile time
- [ ] Thread-safe mutability across multiple threads
- [ ] Automatic synchronization equivalent to a mutex

<details>
<summary>Show Answer</summary>

**Answer:** B — Interior mutability — the ability to mutate data through a shared (`&`) reference, with borrow rules checked at runtime instead of compile time

**Explanation:** `RefCell<T>` moves Rust's "one mutable borrow XOR many shared borrows" rule from compile time to runtime: `borrow()` and `borrow_mut()` track active borrows and panic if violated. **Safety**: this trades a compile error for a runtime panic — it does not weaken the rule, it just enforces it later. It is single-threaded only (no `Sync` for its guards in the way a mutex provides), so it is not a threading primitive (rules out C and D), and it is strictly runtime checking, not a faster compile-time check (rules out A).

</details>
::

::question-wrapper{language="rust"}
### Q6. Why does this code fail to compile?

::code-wrapper{language="rust"}
```rust
use std::rc::Rc;
use std::thread;

fn main() {
    let data = Rc::new(vec![1, 2, 3]);
    let handle = thread::spawn(move || {
        println!("{:?}", data);
    });
    handle.join().unwrap();
}
```
::

- [ ] `Vec<i32>` cannot be sent between threads
- [ ] `Rc<T>` does not implement `Send`, because its non-atomic refcount would cause a data race if incremented from multiple threads
- [ ] `thread::spawn` requires a `'static` closure and this one isn't
- [ ] `println!` cannot be used inside a spawned thread

<details>
<summary>Show Answer</summary>

**Answer:** B — `Rc<T>` does not implement `Send`, because its non-atomic refcount would cause a data race if incremented from multiple threads

**Explanation:** `Rc<T>`'s clone/drop increments and decrements a plain (non-atomic) counter; if two threads did that concurrently it would be a data race, so the standard library simply withholds `Send`/`Sync` for `Rc<T>` and the compiler rejects moving it into another thread. The fix is `Arc<T>`, which uses atomic operations for its counters. `Vec<i32>` itself is perfectly `Send` (rules out A), the closure is `'static` here since `data` is moved in and owns its data (rules out C), and `println!` is fine inside threads (rules out D) — the error is specifically about `Rc`.

</details>
::

::question-wrapper
### Q7. What is `Box<dyn Trait>` primarily used for?

- [ ] Storing a trait object with a size known only at runtime, enabling dynamic dispatch through a heap-allocated pointer
- [ ] Making a trait's methods run faster via static dispatch
- [ ] Automatically implementing the trait for `Box`
- [ ] Sharing ownership of a trait object across threads

<details>
<summary>Show Answer</summary>

**Answer:** A — Storing a trait object with a size known only at runtime, enabling dynamic dispatch through a heap-allocated pointer

**Explanation:** `dyn Trait` is unsized (different implementors have different sizes), so it can't be stored by value; `Box<dyn Trait>` gives it a fixed-size, heap-allocated home and dispatches method calls through a vtable at runtime. That's the opposite of static dispatch — `dyn` implies dynamic dispatch, which is typically slightly slower than monomorphized generics, not faster (rules out B). `Box` doesn't implement the trait itself, it just stores something that does (rules out C), and ownership sharing across threads is `Arc<dyn Trait + Send + Sync>`'s job, not plain `Box`'s (rules out D).

</details>
::

::question-wrapper{language="rust"}
### Q8. What happens when this code runs?

::code-wrapper{language="rust"}
```rust
use std::cell::RefCell;

fn main() {
    let cell = RefCell::new(5);
    let _b1 = cell.borrow_mut();
    let _b2 = cell.borrow_mut();
    println!("{}", *_b1);
}
```
::

- [ ] It prints `5` — `RefCell` allows nested mutable borrows within the same scope
- [ ] It panics at the second `borrow_mut()` call with "already borrowed: BorrowMutError"
- [ ] It fails to compile because `RefCell` requires `unsafe`
- [ ] It silently returns a stale value for `_b2`

<details>
<summary>Show Answer</summary>

**Answer:** B — It panics at the second `borrow_mut()` call with "already borrowed: BorrowMutError"

**Explanation:** `_b1` is a live `RefMut` guard that isn't dropped before `_b2` is requested, so `cell` already has one active mutable borrow when the second `borrow_mut()` runs; `RefCell` enforces exclusivity at runtime and panics rather than allowing the second borrow. **Debug**: the correct fix is to drop `_b1` (e.g., wrap it in its own block, or call `drop(_b1)`) before taking the next borrow, or to use `try_borrow_mut()` and handle the `Err` case instead of unwrapping blindly. This compiles fine (rules out C) — the whole point of `RefCell` is that these checks happen at runtime, not compile time — and there is no silent stale-data behavior (rules out D); the panic is loud and immediate.

</details>
::

::question-wrapper{language="rust"}
### Q9. This code builds a parent-child tree using `Rc<RefCell<Node>>` where each parent also stores an `Rc` back to its child, and each child stores an `Rc` back to its parent. What is the consequence?

::code-wrapper{language="rust"}
```rust
struct Node {
    parent: RefCell<Option<Rc<Node>>>,
    child: RefCell<Option<Rc<Node>>>,
}
```
::

- [ ] The program fails to compile because Rust detects the cycle
- [ ] The program panics at runtime with a stack overflow when dropped
- [ ] Both nodes leak memory — their strong counts never reach zero, so `Drop` never runs, even though nothing is UB
- [ ] Rust's garbage collector silently reclaims the cycle at the next allocation

<details>
<summary>Show Answer</summary>

**Answer:** C — Both nodes leak memory — their strong counts never reach zero, so `Drop` never runs, even though nothing is UB

**Explanation:** Parent holds a strong `Rc` to child and child holds a strong `Rc` back to parent, so each keeps the other's strong count above zero forever; once external references go out of scope, both nodes become unreachable garbage that will never be freed. **Safety**: this is a memory leak, not undefined behavior — Rust's ownership model guarantees memory safety (no dangling pointers, no double frees) but does not guarantee the absence of leaks, and reference cycles are the classic way to produce one. The compiler has no cycle detector for `Rc` graphs (rules out A), there's no stack overflow or panic involved in the leak itself (rules out B), and Rust has no garbage collector to sweep up cycles (rules out D). The idiomatic fix is to make the parent link a `Weak<Node>` instead of a strong `Rc`.

</details>
::

::question-wrapper
### Q10. `Weak<T>` is obtained via `Rc::downgrade(&rc)`. What does calling `.upgrade()` on a `Weak<T>` return once every strong `Rc` to the value has been dropped?

- [ ] `Rc<T>` pointing to freed memory (undefined behavior)
- [ ] `None`, safely indicating the value no longer exists
- [ ] A panic
- [ ] A default-constructed `T`

<details>
<summary>Show Answer</summary>

**Answer:** B — `None`, safely indicating the value no longer exists

**Explanation:** `Weak<T>::upgrade()` returns `Option<Rc<T>>`: if the strong count is still above zero it bumps it and returns `Some(rc)`, and if the value has already been dropped it returns `None` — `Weak` never lets you touch freed memory. This is exactly why `Weak` is the standard tool for breaking parent/back-references in tree and graph structures without risking a dangling pointer (rules out A, which describes what would be UB in a language like C). There's no panic on upgrade (rules out C) and no requirement that `T: Default` (rules out D) — the `Option` return is the whole safety mechanism.

</details>
::

::question-wrapper
### Q11. On a typical 64-bit target, what is `std::mem::size_of::<Rc<i32>>()`?

- [ ] 8 bytes — `Rc<T>` is just a pointer to a heap block that also holds the strong/weak counts
- [ ] 24 bytes — the strong count, weak count, and value are all stored inline in the `Rc` handle
- [ ] 4 bytes — same as `i32`
- [ ] 16 bytes — one pointer plus one inline count

<details>
<summary>Show Answer</summary>

**Answer:** A — 8 bytes — `Rc<T>` is just a pointer to a heap block that also holds the strong/weak counts

**Explanation:** `Rc::new(v)` allocates a single heap block containing an `RcBox` (strong count, weak count, and the value `v` together); the `Rc<T>` handle you hold is just one pointer to that block, so it's a single pointer-sized value like `Box<T>`. **Performance**: this is a common gotcha — people assume the counts live "in the `Rc`" and inflate its size, but they live on the heap alongside the data, which is also why cloning an `Rc` is cheap (copy one pointer, bump one heap-resident counter) rather than copying counts by value. B describes a design `Rc` doesn't use; C ignores that `Rc` is a pointer, not the value itself; D is a plausible-sounding but incorrect split.

</details>
::

::question-wrapper
### Q12. What's the difference between `let b2 = Box::new(42); let b3 = b2.clone();` and `let r2 = Rc::new(42); let r3 = Rc::clone(&r2);`?

- [ ] No difference — both share the same heap allocation afterward
- [ ] `Box::clone` performs a deep clone into a *new* heap allocation, while `Rc::clone` shares the *same* allocation and just bumps a refcount
- [ ] `Box` cannot be cloned at all
- [ ] `Rc::clone` performs a deep clone while `Box::clone` shares the allocation

<details>
<summary>Show Answer</summary>

**Answer:** B — `Box::clone` performs a deep clone into a *new* heap allocation, while `Rc::clone` shares the *same* allocation and just bumps a refcount

**Explanation:** `Box<T>` implements single ownership, so its `Clone` impl (when `T: Clone`) allocates a fresh block and copies the value into it — `b2` and `b3` end up as two independent `i32`s on the heap. `Rc<T>`'s `Clone` impl is the opposite: it does not touch the underlying `T` at all, it just increments the shared strong count and returns a handle to the *same* block. Assuming both behave like `Rc` (A) is the classic trap; `Box<T>` is `Clone` whenever `T: Clone` (rules out C); D has the two exactly backwards.

</details>
::

::question-wrapper{language="rust"}
### Q13. What causes this to panic?

::code-wrapper{language="rust"}
```rust
use std::cell::RefCell;

fn total(counts: &RefCell<Vec<i32>>) -> i32 {
    let data = counts.borrow();
    data.iter().sum()
}

fn add(counts: &RefCell<Vec<i32>>, n: i32) {
    let mut data = counts.borrow_mut();
    data.push(n);
    println!("running total: {}", total(counts));
}
```
::

- [ ] It never panics — `total` and `add` are unrelated functions
- [ ] `add` holds a `borrow_mut()` guard (`data`) alive while calling `total`, which tries `borrow()` on the same still-mutably-borrowed `RefCell`, panicking with "already borrowed"
- [ ] `push` inside a `RefCell` is not allowed under any circumstances
- [ ] It fails to compile because `RefCell<Vec<i32>>` doesn't implement `Iterator`

<details>
<summary>Show Answer</summary>

**Answer:** B — `add` holds a `borrow_mut()` guard (`data`) alive while calling `total`, which tries `borrow()` on the same still-mutably-borrowed `RefCell`, panicking with "already borrowed"

**Explanation:** `data` in `add` is a `RefMut` that stays alive across the `total(counts)` call (it's used again afterward implicitly by scope, and even if it weren't, it hasn't been dropped yet), so `total`'s `counts.borrow()` runs while a mutable borrow is still outstanding — a runtime `BorrowError`. **Debug**: the fix is to drop or scope `data` before calling anything that might re-borrow `counts`, e.g. `{ let mut data = counts.borrow_mut(); data.push(n); }` in its own block, then call `total(counts)` afterward. This absolutely can panic despite the functions looking unrelated (rules out A) — that's exactly the gotcha; nothing bans `push` through a `RefCell` in general (rules out C); and the code compiles fine since `total` only calls `.iter().sum()` on the `Vec` obtained via `Deref`, not on the `RefCell` itself (rules out D).

</details>
::

::question-wrapper
### Q14. What is the effect of calling `drop(rc_handle)` on one clone of an `Rc<T>` that has two other clones still alive elsewhere?

- [ ] It immediately frees the underlying value and invalidates the other clones
- [ ] It only decrements the strong count by one; the value stays alive because other `Rc` handles still reference it
- [ ] It's a compile error — `Rc<T>` cannot be manually dropped
- [ ] It converts the remaining handles into `Weak<T>`

<details>
<summary>Show Answer</summary>

**Answer:** B — It only decrements the strong count by one; the value stays alive because other `Rc` handles still reference it

**Explanation:** `drop` on an `Rc` handle runs `Rc`'s `Drop` impl, which decrements the strong count and only deallocates the inner value when that count reaches zero. With two other live clones, the count merely goes from 3 to 2 and the data is untouched. A is the tempting mistake of treating `Rc` like `Box` (single owner); `Rc<T>` is a completely ordinary value that can be dropped like anything else, no special restriction exists (rules out C); and dropping one strong handle has zero effect on the type of the others — they remain `Rc<T>`, not `Weak<T>` (rules out D).

</details>
::

::question-wrapper
### Q15. For a struct field that needs to be owned by exactly one place but whose size isn't known until runtime (e.g., a boxed trait object or a recursive variant), which is the idiomatic choice?

- [ ] `Rc<T>`
- [ ] `Box<T>`
- [ ] `RefCell<T>`
- [ ] `Arc<Mutex<T>>`

<details>
<summary>Show Answer</summary>

**Answer:** B — `Box<T>`

**Explanation:** **Idiom**: `Box<T>` is the minimal-overhead choice for "one owner, heap-allocated, unsized-at-compile-time" — no refcounting overhead, no runtime borrow checks, no atomics. Reaching for `Rc<T>` (A) or `Arc<Mutex<T>>` (D) when you don't actually need shared ownership adds needless refcount/locking overhead and complexity; reaching for `RefCell<T>` (C) solves a mutability problem you don't have here (single ownership already gives you `&mut` access). The rule of thumb: start with `Box`, add `Rc`/`Arc` only when multiple owners are genuinely required, and add `RefCell`/`Mutex` only when mutation through a shared reference is genuinely required.

</details>
::

::question-wrapper
### Q16. In a single-threaded tree structure using `Rc<RefCell<Node>>` for parent/child links, what is the idiomatic way to prevent the reference-cycle leak described earlier?

- [ ] Wrap the parent link in `Weak<RefCell<Node>>` instead of `Rc<RefCell<Node>>`, since a child shouldn't keep its parent alive
- [ ] Call `std::mem::forget` on the child before dropping the parent
- [ ] Manually call `Rc::strong_count` and panic if it's above 1
- [ ] Switch every field to `Cell<T>`

<details>
<summary>Show Answer</summary>

**Answer:** A — Wrap the parent link in `Weak<RefCell<Node>>` instead of `Rc<RefCell<Node>>`, since a child shouldn't keep its parent alive

**Explanation:** **Idiom**: the ownership direction should mirror the logical lifetime relationship — a tree's parent owns its children (strong `Rc`), but a child referencing its parent should not keep the parent alive on its own, so that link should be `Weak`. This breaks the cycle: when external owners drop the parent, its strong count can reach zero and it's freed even though children still hold `Weak` back-pointers (which safely resolve to `None` via `upgrade()` afterward). `mem::forget` (B) leaks deliberately, the opposite of what's wanted; manually auditing strong counts (C) is fragile and not how idiomatic Rust manages this; `Cell<T>` (D) solves a different problem (cheap interior mutability for `Copy` types), not cycles.

</details>
::

::question-wrapper
### Q17. When is reaching for `Rc<RefCell<T>>` considered good practice versus a design smell?

- [ ] It should be used everywhere shared state exists, as the default choice
- [ ] It's appropriate when shared, mutable, single-threaded ownership is a genuine requirement of the domain (e.g., a GUI widget tree); it's a smell when it's used to route around the borrow checker in code that could instead pass ownership or `&mut` explicitly
- [ ] It should never be used because `unsafe` is always faster
- [ ] It's only valid inside `#[test]` modules

<details>
<summary>Show Answer</summary>

**Answer:** B — It's appropriate when shared, mutable, single-threaded ownership is a genuine requirement of the domain (e.g., a GUI widget tree); it's a smell when it's used to route around the borrow checker in code that could instead pass ownership or `&mut` explicitly

**Explanation:** **Idiom**: `Rc<RefCell<T>>` is a legitimate, commonly used pattern for graphs/trees with genuinely shared mutable nodes, but it moves Rust's aliasing guarantees to runtime, so overusing it as a generic escape hatch from ownership/borrowing errors trades compile-time safety for panics later and is widely considered a code smell. Treating it as an unconditional default (A) throws away the compiler's static guarantees for no reason in code that doesn't need shared mutation. It isn't about `unsafe` being faster — `unsafe` isn't involved here at all (rules out C) — and it's a general-purpose pattern, not something scoped to tests (rules out D).

</details>
::

::question-wrapper
### Q18. `Cell<T>` and `RefCell<T>` both provide interior mutability. What's the key practical difference?

- [ ] `Cell<T>` requires `T: Copy` and only exposes `get`/`set` (no borrows, so it can never panic from a borrow conflict); `RefCell<T>` hands out borrow guards and can panic if borrow rules are violated
- [ ] `Cell<T>` is thread-safe and `RefCell<T>` is not
- [ ] They are interchangeable in every situation
- [ ] `RefCell<T>` is faster because it avoids runtime checks entirely

<details>
<summary>Show Answer</summary>

**Answer:** A — `Cell<T>` requires `T: Copy` and only exposes `get`/`set` (no borrows, so it can never panic from a borrow conflict); `RefCell<T>` hands out borrow guards and can panic if borrow rules are violated

**Explanation:** `Cell<T>` sidesteps the borrow-checking problem entirely by never handing out references to its contents — you copy values in and out via `get`/`set`, which is why it's restricted to (effectively) `Copy` types and can never panic at runtime. `RefCell<T>` instead hands out `Ref`/`RefMut` guards so it works for non-`Copy` types too, but that flexibility is exactly what introduces the possibility of a runtime borrow-rule panic. Neither is thread-safe — both are `!Sync` (rules out B); they solve overlapping but distinct problems so they aren't interchangeable (rules out C); and `RefCell` does more runtime bookkeeping than `Cell`, not less (rules out D).

</details>
::

::question-wrapper{language="rust"}
### Q19. What is the idiomatic way to safely attempt a borrow that might conflict, without risking a panic?

::code-wrapper{language="rust"}
```rust
use std::cell::RefCell;

fn maybe_read(cell: &RefCell<i32>) {
    match cell.try_borrow() {
        Ok(guard) => println!("value: {}", *guard),
        Err(_) => println!("currently borrowed elsewhere, skipping"),
    }
}
```
::

- [ ] This is an anti-pattern — always use `borrow()` and let it panic
- [ ] This is idiomatic — `try_borrow`/`try_borrow_mut` return `Result` so conflicting borrows can be handled gracefully instead of panicking
- [ ] `try_borrow` doesn't exist on `RefCell`
- [ ] `try_borrow` still panics, it just delays the panic

<details>
<summary>Show Answer</summary>

**Answer:** B — This is idiomatic — `try_borrow`/`try_borrow_mut` return `Result` so conflicting borrows can be handled gracefully instead of panicking

**Explanation:** **Idiom**: when a borrow conflict is an expected, recoverable possibility (rather than a programming bug you want to fail loudly on), `try_borrow`/`try_borrow_mut` let you branch on `Ok`/`Err` instead of letting `borrow`/`borrow_mut` panic. Reserving the panicking `borrow()`/`borrow_mut()` for cases where a conflict would indicate a genuine logic error (and the fallible `try_*` variants for cases where it's a normal runtime condition) is the recommended split — always panicking (A) throws away that flexibility. The method exists and is stable (rules out C), and it returns a `Result` rather than panicking at all on the `Err` path (rules out D).

</details>
::

::question-wrapper{language="rust"}
### Q20. In this recursive tree-printing function using `Rc<RefCell<Node>>`, why is the explicit block around `borrow()` necessary to avoid a panic?

::code-wrapper{language="rust"}
```rust
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<RefCell<Node>>>>,
}

fn print_tree(node: &Rc<RefCell<Node>>, depth: usize) {
    let children_snapshot = {
        let n = node.borrow();
        println!("{}{}", "  ".repeat(depth), n.value);
        n.children.borrow().clone()
    };
    for child in children_snapshot.iter() {
        print_tree(child, depth + 1);
    }
}
```
::

- [ ] The block is unnecessary style preference; `node.borrow()` could stay alive across the recursive calls with no issue
- [ ] The block ensures the `Ref` guard from `node.borrow()` is dropped before `print_tree` recurses, so the recursive call's own `node.borrow()` on a child doesn't conflict with a still-held parent borrow
- [ ] `RefCell` borrows are automatically released at the end of every statement, so the block does nothing
- [ ] The block is required only because `Vec::clone` needs exclusive access

<details>
<summary>Show Answer</summary>

**Answer:** B — The block ensures the `Ref` guard from `node.borrow()` is dropped before `print_tree` recurses, so the recursive call's own `node.borrow()` on a child doesn't conflict with a still-held parent borrow

**Explanation:** `n` is a `Ref` guard borrowed from `node`; ending the block drops `n` (and the implicit borrow taken by `n.children.borrow()`) before `children_snapshot` is used in the `for` loop, so by the time `print_tree(child, ...)` runs, `node`'s borrow is fully released. Since each recursive call borrows a *different* `Rc<RefCell<Node>>` (a child, not the same node), this particular example wouldn't panic even without the block — but the pattern of scoping borrows tightly is the general discipline that prevents panics in less trivial call graphs (e.g., a function that re-borrows the *same* node it was passed, or mutates while an outer borrow is still live). **Debug**: assuming a `RefCell` borrow can safely span an arbitrary amount of downstream code (A) is exactly the assumption that causes production panics once the call graph changes; borrows are not scope-magic released at every statement boundary, only when the guard value is actually dropped (rules out C); and `Vec::clone` needs a shared reference to source data, not exclusive access, and has nothing to do with why the block exists (rules out D).

</details>
::
