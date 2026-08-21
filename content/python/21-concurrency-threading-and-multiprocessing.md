# 21 — Concurrency: Threading & Multiprocessing

::question-wrapper{language="python"}
### Q1. What does the CPython Global Interpreter Lock (GIL) actually restrict?

::code-wrapper{language="python"}
```python
import threading

def cpu_heavy():
    total = 0
    for i in range(50_000_000):
        total += i

t1 = threading.Thread(target=cpu_heavy)
t2 = threading.Thread(target=cpu_heavy)
t1.start(); t2.start()
t1.join(); t2.join()
```
::

- [ ] It prevents more than one thread from existing in a process at all
- [ ] It allows only one thread to execute Python bytecode at a time, so this two-thread CPU-bound loop runs no faster (often slightly slower) than doing the work in one thread sequentially
- [ ] It only restricts I/O operations, so this CPU-bound example runs fully in parallel
- [ ] It was removed in Python 3.12, so this now runs on two cores

<details>
<summary>Show Answer</summary>

**Answer:** B — Only one thread executes Python bytecode at a time; this CPU-bound loop gets no real speedup from two threads

**Explanation:** **Performance:** The GIL ensures only one OS thread runs Python bytecode at any instant, even on a multi-core machine. Threads still exist and get scheduled (option A is wrong), but for a pure-Python CPU-bound loop like this, the two threads take turns on a single core, so total wall-clock time is roughly the same as — or worse than, due to context-switch overhead — running both loops sequentially. As of the environment's cutoff, the GIL is still the default in mainline CPython 3.12; an optional free-threaded ("no-GIL") build exists as of 3.13 but is not the default, so option D's blanket claim is wrong for standard installs.

</details>
::

::question-wrapper{language="python"}
### Q2. For which of these workloads does `threading` typically give a real, measurable speedup in CPython?

::code-wrapper{language="python"}
```python
import threading, requests

def fetch(url):
    return requests.get(url).text

threads = [threading.Thread(target=fetch, args=(u,)) for u in urls]
```
::

- [ ] Computing the SHA-256 hash of a 10 GB file in pure Python
- [ ] Fetching 50 URLs over the network concurrently, as shown above
- [ ] Multiplying two large matrices with nested Python `for` loops
- [ ] Parsing a large JSON file with the pure-Python `json` module

<details>
<summary>Show Answer</summary>

**Answer:** B — Fetching 50 URLs over the network concurrently

**Explanation:** **Performance:** Threading shines for I/O-bound work because CPython releases the GIL around blocking I/O calls (socket reads, file I/O, `time.sleep`), letting other threads run while one waits on the network. The other three options are CPU-bound pure-Python work, where the GIL prevents any of the threads from running Python bytecode simultaneously — you'd see little to no speedup, and possibly a slowdown from thread-switching overhead. The common beginner mistake is reaching for `threading` for CPU-heavy loops expecting linear speedup with thread count.

</details>
::

::question-wrapper{language="python"}
### Q3. Why does `multiprocessing` achieve real parallel speedup for CPU-bound work where `threading` does not?

::code-wrapper{language="python"}
```python
from multiprocessing import Process

def cpu_heavy():
    total = sum(i * i for i in range(20_000_000))

if __name__ == "__main__":
    procs = [Process(target=cpu_heavy) for _ in range(4)]
    for p in procs: p.start()
    for p in procs: p.join()
```
::

- [ ] Each `Process` is a separate OS process with its own Python interpreter and its own GIL, so four processes can genuinely run on four cores simultaneously
- [ ] `multiprocessing` disables the GIL globally for the whole machine
- [ ] `Process` objects share the same GIL but are scheduled with higher OS priority than threads
- [ ] There's no real difference; `multiprocessing.Process` is just an alias for `threading.Thread` under the hood

<details>
<summary>Show Answer</summary>

**Answer:** A — Each `Process` is a separate OS process with its own interpreter and its own GIL, so multiple processes can run truly in parallel

**Explanation:** **Performance:** `multiprocessing` sidesteps the GIL entirely by spawning independent OS processes, each with its own Python interpreter, memory space, and GIL — there is no single lock shared across processes, so four processes really can use four CPU cores concurrently. This is the fundamental reason `multiprocessing` is the standard recommendation for CPU-bound parallelism in pure Python, unlike `threading`. The trade-off (covered later in this quiz) is the overhead of inter-process communication, since memory is not shared by default.

</details>
::

::question-wrapper{language="python"}
### Q4. Given `concurrent.futures`, which executor should you pick for (a) downloading 200 files over HTTP and (b) computing 200 independent CPU-heavy Fibonacci-like calculations in pure Python?

::code-wrapper{language="python"}
```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor
```
::

- [ ] `ProcessPoolExecutor` for both, since it's always faster than `ThreadPoolExecutor`
- [ ] `ThreadPoolExecutor` for the downloads (I/O-bound), `ProcessPoolExecutor` for the CPU-heavy computation (bypasses the GIL for true parallelism)
- [ ] `ThreadPoolExecutor` for both, since threads are lighter-weight and that outweighs GIL contention
- [ ] It doesn't matter — both executors use the same underlying worker model

<details>
<summary>Show Answer</summary>

**Answer:** B — `ThreadPoolExecutor` for I/O-bound downloads, `ProcessPoolExecutor` for CPU-bound computation

**Explanation:** **Idiom:** This is the standard rule of thumb: I/O-bound tasks spend most of their time waiting (network, disk), so lightweight threads are sufficient and avoid process-spawn/IPC overhead. CPU-bound pure-Python tasks are limited by the GIL under threads, so `ProcessPoolExecutor` is needed to actually use multiple cores. Defaulting to `ProcessPoolExecutor` "because it's always faster" (option A) is wrong — for I/O-bound work, process overhead (spawning, pickling task arguments/results) usually makes it slower than threads for the same job.

</details>
::

::question-wrapper
### Q5. Why does a network call like `socket.recv()` or `requests.get()` allow other Python threads to make progress, even though the GIL normally lets only one thread run at a time?

- [ ] It doesn't — network calls block all threads until they return
- [ ] Blocking I/O calls in CPython release the GIL while waiting on the OS/kernel for data, allowing other threads to acquire it and run Python bytecode in the meantime
- [ ] Network calls are implemented entirely in a separate process automatically
- [ ] Only `asyncio`, not `threading`, gets this benefit

<details>
<summary>Show Answer</summary>

**Answer:** B — Blocking I/O calls release the GIL while waiting on the OS, letting other threads run meanwhile

**Explanation:** CPython's C implementations of blocking I/O operations explicitly release the GIL before making the blocking system call and re-acquire it afterward. This is precisely why `threading` is effective for I/O-bound concurrency: while one thread is parked waiting on the kernel for socket data, the GIL is free for another thread to run actual Python code. This release/reacquire dance is unrelated to `asyncio` (option D) — it's a `threading`-level mechanism that predates `asyncio` entirely.

</details>
::

::question-wrapper{language="python"}
### Q6. Despite the GIL serializing bytecode execution, this code can still print a final `counter` value less than `200000`. Why?

::code-wrapper{language="python"}
```python
import threading

counter = 0

def increment():
    global counter
    for _ in range(100_000):
        counter += 1

threads = [threading.Thread(target=increment) for _ in range(2)]
for t in threads: t.start()
for t in threads: t.join()

print(counter)
```
::

- [ ] It's impossible — the GIL guarantees `counter` is always exactly `200000`
- [ ] `counter += 1` is not a single atomic bytecode operation; the GIL can switch threads mid-sequence (between the read, add, and store steps), so increments from the two threads can be lost to a race condition
- [ ] `threading.Thread` silently drops some loop iterations for performance
- [ ] `global counter` desynchronizes the variable between threads

<details>
<summary>Show Answer</summary>

**Answer:** B — `counter += 1` isn't atomic; the GIL can switch threads between the read/add/store steps, losing increments to a race condition

**Explanation:** **Safety:** A very common misconception is that the GIL makes all Python code thread-safe — it does not. `counter += 1` compiles to multiple bytecode instructions (load, add, store), and CPython's scheduler can switch to another thread between any of them. If thread A reads `counter=5`, thread B also reads `5` before A writes back `6`, both compute `6`, and one increment is lost. The correct fix is a `threading.Lock` (or an atomic structure) around the read-modify-write: `with lock: counter += 1`.

</details>
::

::question-wrapper{language="python"}
### Q7. What overhead does `multiprocessing` introduce that `threading` does not, when passing data to and from worker processes?

::code-wrapper{language="python"}
```python
from multiprocessing import Pool

def process_item(item):
    return item.upper()

if __name__ == "__main__":
    with Pool(4) as pool:
        results = pool.map(process_item, large_list_of_strings)
```
::

- [ ] None — memory is automatically shared between processes just like threads
- [ ] Arguments and return values must be pickled (serialized) to cross the process boundary and unpickled on the other side, which costs CPU time and can dominate runtime for large or many small objects
- [ ] `multiprocessing.Pool` uses shared memory exclusively, so this overhead only applies to `Process`, not `Pool`
- [ ] The overhead only applies to the return values, not the input arguments

<details>
<summary>Show Answer</summary>

**Answer:** B — Arguments and return values must be pickled/unpickled to cross the process boundary, costing CPU time

**Explanation:** **Performance:** Unlike threads, which share the same process memory space, separate processes do not share memory by default. `Pool.map` serializes each argument with `pickle`, sends it through an OS pipe to a worker process, and pickles the result back. For workloads with many small, fast tasks, this serialization overhead can exceed the actual computation time, sometimes making `multiprocessing` slower than a single-threaded loop — a frequent surprise for people expecting free parallelism. `Pool` uses the same pickling mechanism as `Process` (option C is wrong); it does not get free shared memory.

</details>
::

::question-wrapper{language="python"}
### Q8. What happens when you try to pass this to a `multiprocessing.Pool`?

::code-wrapper{language="python"}
```python
from multiprocessing import Pool

if __name__ == "__main__":
    with Pool(4) as pool:
        results = pool.map(lambda x: x * 2, [1, 2, 3])
```
::

- [ ] It runs fine — lambdas are pickled by value automatically
- [ ] Raises `PicklingError` (or similar) because lambda functions cannot be pickled — only module-level (importable) functions and objects can cross the process boundary this way
- [ ] It silently falls back to running single-threaded
- [ ] It works, but only on Linux, never on any other OS

<details>
<summary>Show Answer</summary>

**Answer:** B — Raises a pickling error because lambdas can't be pickled; only importable, module-level functions/objects can be sent to worker processes

**Explanation:** **Debug:** `pickle` (the mechanism `multiprocessing` uses to send work to worker processes) serializes callables by reference — it stores the module and qualified name and re-imports them in the worker. A `lambda` has no importable name (`<lambda>`), so pickling fails with something like `AttributeError: Can't pickle local object`. The fix is to define the function at module level with `def` instead of as a `lambda`. This is a frequent gotcha for people used to `threading` or `concurrent.futures.ThreadPoolExecutor`, where lambdas work fine since no pickling ever happens.

</details>
::

::question-wrapper{language="python"}
### Q9. Two worker processes need to update a shared counter. Why won't this work as expected, and what's the fix?

::code-wrapper{language="python"}
```python
from multiprocessing import Process

counter = 0

def increment():
    global counter
    counter += 1

procs = [Process(target=increment) for _ in range(10)]
for p in procs: p.start()
for p in procs: p.join()

print(counter)
```
::

- [ ] It prints `10` correctly because `multiprocessing` synchronizes global variables automatically
- [ ] Each process gets its own copy of the module (and thus `counter`) at fork/spawn time; changes in a child are invisible to the parent and other children, so the parent's `counter` stays `0`. The fix is `multiprocessing.Value`/`Array` or a `Manager` for real shared state
- [ ] It raises `RuntimeError` because global variables are forbidden with `multiprocessing`
- [ ] It prints a random number between 0 and 10 due to race conditions, same as with threads

<details>
<summary>Show Answer</summary>

**Answer:** B — Each process gets its own independent copy of `counter`; the parent's value never changes, so the fix is `Value`/`Array`/`Manager` for real shared state

**Explanation:** **Safety:** Unlike threads, which share one address space, each `multiprocessing.Process` has an entirely separate memory space (a copy of the parent's state at fork time, or a fresh reimport under spawn). Incrementing `counter` inside a child mutates that child's private copy — the parent's `counter` never sees it, so the parent prints `0`, not a race-condition-corrupted partial count (that's the threading failure mode from Q6, not this one). To actually share mutable state across processes you need explicit IPC-aware primitives: `multiprocessing.Value`, `Array`, or a `Manager().dict()`/`list()`, all backed by shared memory or a proxy process.

</details>
::

::question-wrapper{language="python"}
### Q10. What's wrong with this locking pattern, and what does it cause?

::code-wrapper{language="python"}
```python
import threading

lock = threading.Lock()

def outer():
    with lock:
        inner()

def inner():
    with lock:
        print("done")

outer()
```
::

- [ ] Nothing — `threading.Lock` allows the same thread to reacquire it any number of times
- [ ] The thread deadlocks itself: `threading.Lock` is non-reentrant, so `inner()`'s attempt to acquire an already-held lock blocks forever. Using `threading.RLock` instead would allow the same thread to reacquire it
- [ ] It raises `RuntimeError: lock already held` immediately instead of blocking
- [ ] `with lock:` silently no-ops on the second acquisition

<details>
<summary>Show Answer</summary>

**Answer:** B — The thread deadlocks itself; `threading.Lock` is non-reentrant, so use `threading.RLock` for nested acquisition by the same thread

**Explanation:** **Safety:** A plain `threading.Lock` is not reentrant — once a thread holds it, that same thread blocks (rather than passing through) on a second `acquire()`, because the lock has no concept of "owner." Since `inner()` runs in the same thread that already holds the lock via `outer()`, the program hangs forever with no exception (option C is wrong — no error is ever raised; it just blocks). `threading.RLock` tracks the owning thread and an acquisition count, letting the same thread re-enter safely, which is the standard fix for recursive or nested locking within one thread's call stack.

</details>
::

::question-wrapper{language="python"}
### Q11. A CPU-bound function calls into a C extension (e.g., NumPy's matrix multiply) that explicitly releases the GIL during its computation. What does this enable?

::code-wrapper{language="python"}
```python
import threading
import numpy as np

def matmul_heavy():
    a = np.random.rand(2000, 2000)
    b = np.random.rand(2000, 2000)
    a @ b

threads = [threading.Thread(target=matmul_heavy) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()
```
::

- [ ] Nothing — pure computation always requires the GIL, so this is no different from a pure-Python loop
- [ ] These threads can achieve genuine multi-core parallelism, because NumPy's underlying C code releases the GIL for the duration of the heavy computation, unlike pure-Python bytecode loops
- [ ] NumPy always uses multiprocessing internally regardless of the GIL
- [ ] This only works if `threading` is replaced with `multiprocessing`

<details>
<summary>Show Answer</summary>

**Answer:** B — These threads can achieve genuine multi-core parallelism because NumPy's C code releases the GIL during the heavy computation

**Explanation:** **Performance:** The GIL restriction applies specifically to executing Python bytecode, not to C code. Well-written C extensions (NumPy, many `hashlib` and `zlib` operations, some `re` operations) explicitly release the GIL around long-running C-level work using `Py_BEGIN_ALLOW_THREADS`/`Py_END_ALLOW_THREADS`, letting other Python threads run truly concurrently on other cores during that window. This is an important exception to the general "threading doesn't help CPU-bound work" rule (Q1/Q2) and explains why numeric/scientific Python libraries can benefit meaningfully from `threading`, unlike hand-written pure-Python loops.

</details>
::

::question-wrapper{language="python"}
### Q12. What is the effect of `daemon=True` on a thread, and what happens without it?

::code-wrapper{language="python"}
```python
import threading, time

def background_task():
    while True:
        time.sleep(1)

t = threading.Thread(target=background_task, daemon=True)
t.start()
print("main thread finished")
```
::

- [ ] `daemon=True` has no effect; the program always exits once `main()` finishes regardless of running threads
- [ ] A daemon thread is killed abruptly when the main program exits; without `daemon=True`, this infinite-loop thread would keep the whole process alive forever since Python waits for all non-daemon threads to finish
- [ ] `daemon=True` makes the thread run with elevated OS privileges
- [ ] Daemon threads run on a separate GIL, so they never block other threads

<details>
<summary>Show Answer</summary>

**Answer:** B — A daemon thread is killed when the main program exits; without it, this infinite loop would keep the process alive forever

**Explanation:** **Debug:** By default, CPython will not exit the process until every non-daemon thread finishes, since threads represent real OS-level work that might need to complete. An infinite-loop background thread without `daemon=True` would hang the program indefinitely after `print` runs — the process never terminates. Marking it `daemon=True` tells the interpreter it's safe to abruptly kill that thread on exit, which is standard practice for background workers (polling loops, heartbeat threads) that shouldn't block shutdown.

</details>
::

::question-wrapper{language="python"}
### Q13. What is likely to go wrong if this script (using the default `spawn` start method, e.g. on Windows or macOS) omits the `if __name__ == "__main__":` guard?

::code-wrapper{language="python"}
```python
from multiprocessing import Process

def worker():
    print("working")

p = Process(target=worker)
p.start()
p.join()
```
::

- [ ] Nothing — the guard is only a stylistic convention, never functionally required
- [ ] On `spawn`-based platforms, each child process re-imports the main module to set itself up; without the guard, the child re-executes the top-level `Process(...).start()` call too, spawning another child recursively — leading to a runaway process bomb or `RuntimeError`
- [ ] It causes a `SyntaxError` at parse time
- [ ] It only affects `Pool`, never plain `Process`

<details>
<summary>Show Answer</summary>

**Answer:** B — Without the guard, `spawn`-based child processes re-execute the top-level code, recursively spawning more children — a process bomb or `RuntimeError`

**Explanation:** **Portability:** On platforms using the `spawn` start method (the default on Windows and, since Python 3.8, macOS), a new child process starts a fresh Python interpreter and re-imports `__main__` to reconstruct what it needs to run the target function — it does not fork the parent's already-running memory. If the module-level code that creates and starts the `Process` isn't guarded by `if __name__ == "__main__":`, each child re-runs that same top-level code on import, spawning yet another child, recursively. CPython actually detects this specific pattern and raises `RuntimeError: An attempt has been made to start a new process before the current process has finished its bootstrapping phase...` with guidance to add the guard. This is a genuine functional requirement on `spawn`/Windows, not just a style preference (option A) — though on Linux's default `fork` method the same code often "works" without the guard, which is precisely why the bug is a portability trap.

</details>
::

::question-wrapper
### Q14. Is the GIL a single global lock shared across an entire machine, or something else?

- [ ] It's one lock per machine — even unrelated Python processes contend for the same GIL
- [ ] It's one lock per Python process (per interpreter instance); separate processes each have their own independent GIL and don't contend with each other
- [ ] It's one lock per CPU core
- [ ] It's one lock per thread, meaning it provides no serialization at all

<details>
<summary>Show Answer</summary>

**Answer:** B — One GIL per Python process/interpreter; separate processes have entirely independent GILs

**Explanation:** The name "Global Interpreter Lock" refers to global-within-one-interpreter scope, not global-across-the-machine — each Python process runs its own interpreter with its own GIL, completely independent of any other process's GIL. This is exactly why `multiprocessing` sidesteps the limitation: spawning multiple processes means multiple independent GILs, each free to run bytecode on a different core simultaneously, whereas multiple threads inside one process must all share that one process's single GIL.

</details>
::

::question-wrapper{language="python"}
### Q15. Two threads need to hand off work items safely without manual locking. What's the idiomatic tool?

::code-wrapper{language="python"}
```python
import threading, queue

def producer(q):
    for i in range(10):
        q.put(i)

def consumer(q):
    while True:
        item = q.get()
        print(item)
        q.task_done()

q = queue.Queue()
```
::

- [ ] Manually appending to and popping from a shared `list` with no synchronization, since list operations are "atomic enough"
- [ ] `queue.Queue`, which is internally synchronized with its own lock/condition variables, making `put`/`get` safe to call from multiple threads without any extra locking code
- [ ] `multiprocessing.Queue`, since it's a strict superset of `queue.Queue`
- [ ] A plain Python `dict` keyed by timestamp

<details>
<summary>Show Answer</summary>

**Answer:** B — `queue.Queue` is internally synchronized, making it safe for multi-threaded producer/consumer patterns without manual locking

**Explanation:** **Idiom:** `queue.Queue` is specifically designed for thread-safe communication — its `put`/`get` methods handle all locking internally, and it additionally supports blocking with optional timeouts, making it the standard building block for producer/consumer patterns in `threading` code. Relying on raw `list.append`/`pop` (option A) is risky: while individual list methods are atomic due to the GIL, compound patterns (check-then-act, popping under certain conditions) are not, and it lacks the queue's blocking/wake-up semantics. `multiprocessing.Queue` (option C) is a different, heavier implementation meant for cross-process communication with its own pickling overhead — not a strict superset for thread use.

</details>
::

::question-wrapper
### Q16. A team's data pipeline spawns 500 threads to process 500 small CPU-bound number-crunching tasks in pure Python, expecting a 500x speedup on a 16-core machine. What actually happens, and what's the better approach?

- [ ] It gets the expected roughly-500x speedup since threads scale linearly with count
- [ ] It gets little to no speedup — likely worse than sequential due to GIL contention and context-switching overhead among 500 threads; a `ProcessPoolExecutor` sized to the CPU count (e.g., 16) would actually use the available cores
- [ ] It crashes because Python enforces a hard limit of 100 threads
- [ ] It gets exactly a 16x speedup automatically because the GIL load-balances across cores

<details>
<summary>Show Answer</summary>

**Answer:** B — Little to no speedup, likely worse than sequential due to GIL contention/context-switching; a right-sized `ProcessPoolExecutor` would use the cores

**Explanation:** **Performance:** Since only one thread executes Python bytecode at a time regardless of thread count, adding more threads for CPU-bound work doesn't add parallelism — it adds scheduling and context-switch overhead, which can make total throughput worse than a single thread doing the work sequentially. The fix is `ProcessPoolExecutor(max_workers=16)` (roughly matching core count) so the OS can schedule genuinely parallel processes, each with its own GIL. Over-threading for CPU work is a common performance anti-pattern from people assuming "more threads = more parallel," which only holds once I/O is involved.

</details>
::

::question-wrapper{language="python"}
### Q17. A web server handler stores per-request state in a variable at module scope so different helper functions can access it without passing it explicitly. Under `threading`, why is this dangerous, and what's the fix?

::code-wrapper{language="python"}
```python
current_user = None

def handle_request(user, request):
    global current_user
    current_user = user
    process(request)

def process(request):
    print(f"Processing for {current_user}")
```
::

- [ ] It's safe — the GIL ensures each request sees only its own `current_user`
- [ ] Concurrent requests handled by different threads share the same module-level `current_user`, so one thread can overwrite it while another is mid-request, leaking one user's identity into another's processing; the fix is `threading.local()` to give each thread its own isolated copy
- [ ] It's safe as long as `process` is called immediately after setting `current_user`
- [ ] `global` automatically makes the variable thread-safe by serializing access

<details>
<summary>Show Answer</summary>

**Answer:** B — Module-level `current_user` is shared across all threads, so concurrent requests can clobber each other's value; use `threading.local()` for per-thread isolation

**Explanation:** **Safety:** A module-level variable lives in one shared namespace regardless of which thread touches it — there's no per-thread isolation just because the GIL exists. If Thread A sets `current_user = alice` and, before `process()` runs, the GIL switches to Thread B which sets `current_user = bob`, Thread A's `process()` call could print "Processing for bob," a serious correctness (and potentially security) bug. `threading.local()` creates an object where each thread transparently sees its own independent attribute values — the standard fix for this exact per-request-state pattern, used internally by frameworks like Flask for request-context globals.

</details>
::

::question-wrapper{language="python"}
### Q18. When submitting many quick tasks to a `ProcessPoolExecutor`, why might `pool.map()` significantly underperform compared to running them in a single process, even though the tasks are CPU-bound?

::code-wrapper{language="python"}
from concurrent.futures import ProcessPoolExecutor

def square(x):
    return x * x

with ProcessPoolExecutor() as pool:
    results = list(pool.map(square, range(1_000_000)))
```
::

- [ ] `ProcessPoolExecutor` never actually parallelizes anything; it's identical to a loop
- [ ] For very cheap per-item work, the fixed cost of pickling each argument/result and IPC round-trips per task can dwarf the actual computation time, making the overhead outweigh the parallelism gained; batching items (e.g., `chunksize`) or doing the work in one process is often faster
- [ ] `pool.map()` silently caps at processing 1,000 items maximum
- [ ] `ProcessPoolExecutor` requires manual chunking or it raises `MemoryError`

<details>
<summary>Show Answer</summary>

**Answer:** B — For cheap per-item work, pickling/IPC overhead per task can dwarf actual computation, so overhead can outweigh parallelism gains; batching (`chunksize`) or single-process execution is often faster

**Explanation:** **Performance:** Each call to `square(x)` here does trivial work, but by default `Pool.map`/`ProcessPoolExecutor.map` may send items to workers with a small chunk size, meaning the per-task pickling/unpickling and inter-process messaging cost can exceed the actual `x * x` computation many times over. The fix is to increase `chunksize` (grouping many items per IPC round-trip) or to recognize that sub-millisecond per-item tasks generally aren't good candidates for process-based parallelism at all — the overhead needs to be amortized over meaningfully larger units of work.

</details>
::

::question-wrapper
### Q19. For a long-running batch of independent CPU-bound tasks where results should be consumed as soon as each one finishes (not necessarily in submission order), which is more appropriate: `Pool.map()` or `Pool.imap_unordered()`?

- [ ] `Pool.map()`, because it always finishes faster regardless of use case
- [ ] `Pool.imap_unordered()`, because it yields each result as soon as any worker finishes it, rather than `map()`'s behavior of collecting all results and returning them (in original order) only once the entire batch completes
- [ ] They are functionally identical; the name difference is purely cosmetic
- [ ] `imap_unordered()` runs tasks sequentially in one process, unlike `map()`

<details>
<summary>Show Answer</summary>

**Answer:** B — `imap_unordered()` yields each result as soon as it's ready, rather than waiting for the whole batch like `map()`

**Explanation:** **Idiom:** `Pool.map()` blocks until every task in the iterable is complete and then returns a list in the original submission order — fine when you need all results together, but wasteful if you want to start processing/streaming results as they arrive. `imap_unordered()` returns an iterator that yields each finished result the moment its worker completes, in whatever order they finish, which better suits streaming pipelines or progress reporting, especially when task durations vary. Choosing `map()` by default even when order doesn't matter needlessly delays consuming the earliest-finished results.

</details>
::

::question-wrapper{language="python"}
### Q20. A codebase mixes `threading.Lock` for protecting a shared cache with `multiprocessing.Process` workers that also read/write that same Python-level cache object. Why is this fundamentally broken, and what's the fix?

::code-wrapper{language="python"}
```python
import threading
from multiprocessing import Process

cache = {}
lock = threading.Lock()

def worker(key, value):
    with lock:
        cache[key] = value

procs = [Process(target=worker, args=(i, i * i)) for i in range(5)]
```
::

- [ ] It works correctly — `threading.Lock` synchronizes across processes just like within a single process
- [ ] `threading.Lock` only synchronizes threads within the same process's memory; each `Process` gets its own independent copy of `lock` and `cache`, so the lock provides no cross-process protection at all, and updates in workers never appear in the parent's `cache`. The fix is `multiprocessing.Manager().dict()` with a `multiprocessing.Lock()`, or another IPC-aware shared structure
- [ ] The code raises `TypeError` at process-start time because locks can't be used in worker functions
- [ ] It's correct, but only if `lock` is created inside `worker()` instead of at module scope

<details>
<summary>Show Answer</summary>

**Answer:** B — `threading.Lock` only works within one process's shared memory; each `Process` gets an independent copy of `lock` and `cache`, so it gives zero cross-process protection, and the parent's `cache` never updates

**Explanation:** **Safety:** `threading.Lock` is implemented against in-process memory primitives — when a `Process` is spawned/forked, it receives its own copy of the `lock` object (a distinct OS-level mutex, not a shared one) and its own copy of `cache`, exactly like the `counter` example in Q9. Acquiring "the same" lock in different processes acquires two entirely different locks, providing no real mutual exclusion across processes, and any writes to `cache` inside a worker only affect that worker's private copy — the parent's `cache` stays `{}`. The correct fix uses cross-process-aware primitives: `multiprocessing.Manager().dict()` for a proxy-backed shared dict, combined with `multiprocessing.Lock()` (not `threading.Lock()`) for true cross-process mutual exclusion.

</details>
::
