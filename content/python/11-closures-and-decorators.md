# 11 — Closures & Decorators

::question-wrapper
### Q1. What is a closure in Python?

- [ ] A function that has access to variables from its enclosing lexical scope, even after the enclosing function has finished executing
- [ ] Any function defined inside a class body
- [ ] A function that automatically closes open file handles when it returns
- [ ] A function decorated with `@staticmethod`

<details>
<summary>Show Answer</summary>

**Answer:** A — A function that has access to variables from its enclosing lexical scope, even after the enclosing function has finished executing

**Explanation:** A closure is an inner function that "remembers" the variables from its enclosing scope via a cell object, so it can still read (and with `nonlocal`, write) them long after the outer function has returned. It has nothing to do with classes or file handles — those are unrelated meanings of "closing" that beginners sometimes conflate with the term.

</details>
::

::question-wrapper
### Q2. What does the `nonlocal` keyword do?

- [ ] Lets an inner function assign to a variable defined in its nearest enclosing (non-global) function scope
- [ ] Lets an inner function assign to a variable in the module's global scope
- [ ] Declares a variable that is shared across all instances of a class
- [ ] Makes a variable visible to every thread without a lock

<details>
<summary>Show Answer</summary>

**Answer:** A — Lets an inner function assign to a variable defined in its nearest enclosing (non-global) function scope

**Explanation:** Without `nonlocal`, assigning to a name inside a nested function creates a brand-new local variable that shadows the outer one instead of modifying it. `nonlocal` tells Python to bind to the enclosing function's variable instead. `global` (option B) is the different keyword for module-level scope — mixing the two up is a common mistake.

</details>
::

::question-wrapper
### Q3. `@my_decorator` placed above `def func(): ...` is syntactic sugar for which statement?

- [ ] `func = my_decorator(func)`
- [ ] `func = my_decorator()(func)`
- [ ] `my_decorator.func = func`
- [ ] `func = my_decorator`

<details>
<summary>Show Answer</summary>

**Answer:** A — `func = my_decorator(func)`

**Explanation:** A plain decorator is just a function that takes the decorated function as its single argument and returns a (usually different) callable, which is rebound to the original name. Option B is the expansion for a decorator *factory* called with arguments, like `@my_decorator(arg)`, not a plain decorator.

</details>
::

::question-wrapper
### Q4. What does `functools.wraps` do when applied inside a decorator's wrapper function?

- [ ] Copies `__name__`, `__doc__`, and other metadata from the original function onto the wrapper
- [ ] Wraps the return value of the function in a `try`/`except` block automatically
- [ ] Caches the function's return value like `lru_cache`
- [ ] Converts a regular function into a coroutine

<details>
<summary>Show Answer</summary>

**Answer:** A — Copies `__name__`, `__doc__`, and other metadata from the original function onto the wrapper

**Explanation:** Without `@functools.wraps(func)` on the inner `wrapper`, introspection tools, debuggers, and `help()` will report the wrapper's own name (typically `"wrapper"`) and lose the original docstring, which makes stack traces and documentation confusing. It does not add error handling or caching — those are unrelated decorator patterns.

</details>
::

::question-wrapper{language="python"}
### Q5. Given stacked decorators:

::code-wrapper{language="python"}
```python
@first
@second
def handler():
    ...
```
::

Which decorator's wrapping logic executes *first* when `handler()` is called?

- [ ] `second`'s, because it wraps the original function directly and runs on the way in before control reaches `first`
- [ ] `first`'s, because it's listed first in the source
- [ ] Both run simultaneously
- [ ] Neither — only `first` applies; `second` is silently discarded

<details>
<summary>Show Answer</summary>

**Answer:** A — `second`'s, because it wraps the original function directly and runs on the way in before control reaches `first`

**Explanation:** Decorators apply bottom-up but execute outside-in: `handler = first(second(handler))`, so `first`'s wrapper is the outermost call, but its body typically calls the thing it wraps (`second`'s wrapper) before that call returns — so on entry, `first`'s pre-call code runs, then `second`'s pre-call code, then the real function. The naive assumption that "top decorator runs first in every sense" is the beginner trap; it's true for *application order*, not necessarily for every line of runtime behavior.

</details>
::

::question-wrapper
### Q6. A decorator that itself accepts arguments, e.g. `@retry(times=3)`, requires how many levels of nested functions?

- [ ] Three — an outer factory taking the decorator's arguments, a middle decorator taking the function, and an inner wrapper taking the function's call arguments
- [ ] One — the decorator function itself takes both the decorator arguments and the function
- [ ] Two — a decorator taking the function, and a wrapper taking call arguments
- [ ] Four — Python requires a separate factory for each keyword argument

<details>
<summary>Show Answer</summary>

**Answer:** A — Three — an outer factory taking the decorator's arguments, a middle decorator taking the function, and an inner wrapper taking the function's call arguments

**Explanation:** `retry(times=3)` must first return an actual decorator (level 2), which is then called with the function (`func`), which must return a `wrapper(*args, **kwargs)` (level 3) that does the real work. Beginners often try to collapse this into two levels (option C), which works for plain decorators but not parameterized ones, since `@retry(times=3)` calls `retry` *before* decoration even begins.

</details>
::

::question-wrapper
### Q7. What does a Python closure actually capture from the enclosing scope?

- [ ] A reference to the variable's cell, so the closure always sees the variable's *current* value, not a snapshot taken at definition time
- [ ] A deep copy of the variable's value at the moment the inner function is defined
- [ ] A shallow copy of the value, copied only if the value is mutable
- [ ] Nothing — closures re-evaluate the enclosing function's source code on each call

<details>
<summary>Show Answer</summary>

**Answer:** A — A reference to the variable's cell, so the closure always sees the variable's *current* value, not a snapshot taken at definition time

**Explanation:** Python closures bind by reference to a shared cell object, not by value. This is exactly why the late-binding loop-variable gotcha exists: every closure created in a loop shares the *same* cell for the loop variable, so they all observe whatever that variable holds when they're eventually called, not when they were created.

</details>
::

::question-wrapper{language="python"}
### Q8. What does this print?

::code-wrapper{language="python"}
```python
callbacks = []
for i in range(3):
    callbacks.append(lambda: i)

print([cb() for cb in callbacks])
```
::

- [ ] `[2, 2, 2]`
- [ ] `[0, 1, 2]`
- [ ] `[0, 0, 0]`
- [ ] `RuntimeError: variable modified during iteration`

<details>
<summary>Show Answer</summary>

**Answer:** A — `[2, 2, 2]`

**Explanation:** **Debug** — All three lambdas close over the *same* variable `i`, not three independent copies. By the time the list comprehension calls them, the `for` loop has already finished and `i` holds its final value, `2`. The tempting `[0, 1, 2]` answer assumes each lambda captures the value of `i` at the point it was created, which is how closures work in some other languages but not in Python's late-binding model.

</details>
::

::question-wrapper
### Q9. Which change correctly fixes the late-binding bug from Q8 so the output is `[0, 1, 2]`?

- [ ] `callbacks.append(lambda i=i: i)` — bind `i`'s current value as a default argument at lambda-creation time
- [ ] `callbacks.append(lambda: int(i))` — wrapping in `int()` forces early evaluation
- [ ] Replace the `for` loop with a `while` loop
- [ ] `callbacks.append(lambda: i.copy())` — copy the integer before storing it

<details>
<summary>Show Answer</summary>

**Answer:** A — `callbacks.append(lambda i=i: i)` — bind `i`'s current value as a default argument at lambda-creation time

**Explanation:** Default argument values *are* evaluated once, at function-definition time, so `i=i` captures the loop variable's value on each iteration into a fresh, per-lambda default. `int(i)` (option B) still reads the shared cell at call time, so it doesn't help; integers have no `.copy()` method (option D), and switching loop constructs (option C) doesn't change how closures bind names.

</details>
::

::question-wrapper{language="python"}
### Q10. What happens when this code runs?

::code-wrapper{language="python"}
```python
def outer():
    def inner():
        nonlocal missing
        missing = 1
    inner()

outer()
```
::

- [ ] `SyntaxError: no binding for nonlocal 'missing' found` — raised at compile time, before `outer()` is ever called
- [ ] `UnboundLocalError` raised when `inner()` executes
- [ ] It runs fine; `missing` becomes a new global variable
- [ ] It runs fine; `nonlocal` silently creates `missing` in `outer`'s scope

<details>
<summary>Show Answer</summary>

**Answer:** A — `SyntaxError: no binding for nonlocal 'missing' found` — raised at compile time, before `outer()` is ever called

**Explanation:** Unlike `global`, which will happily create a new module-level name if one doesn't exist, `nonlocal` requires that some enclosing function scope *already* bind that name (e.g., via assignment) — Python checks this at compile time. Since `outer` never assigns `missing` anywhere, the whole module fails to compile with a `SyntaxError`, not a runtime error, which surprises people who expect the failure to happen only when `inner()` is called.

</details>
::

::question-wrapper{language="python"}
### Q11. A shared mutable counter is implemented two ways. Which one actually works in Python 3 without extra tricks?

::code-wrapper{language="python"}
```python
# Version A
def make_counter_a():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment

# Version B
def make_counter_b():
    count = 0
    def increment():
        count += 1
        return count
    return increment
```
::

- [ ] Only Version A works; Version B raises `UnboundLocalError` on the first call to `increment()`
- [ ] Only Version B works; Version A raises a `SyntaxError`
- [ ] Both versions work identically
- [ ] Neither works; both need a `global` declaration

<details>
<summary>Show Answer</summary>

**Answer:** A — Only Version A works; Version B raises `UnboundLocalError` on the first call to `increment()`

**Explanation:** `count += 1` is equivalent to `count = count + 1`, and the presence of that assignment makes Python treat `count` as local to `increment` at compile time — so the read on the right-hand side happens before any local `count` has been assigned, raising `UnboundLocalError`. Version A's `nonlocal count` tells Python to use the enclosing cell instead of creating a new local, which is exactly the fix this pattern needs.

</details>
::

::question-wrapper{language="python"}
### Q12. A decorator wraps an instance method but the wrapper is defined as `def wrapper(*args): ...` (no `**kwargs`). What breaks?

::code-wrapper{language="python"}
```python
def log_call(func):
    def wrapper(*args):
        print(f"Calling {func.__name__}")
        return func(*args)
    return wrapper

class Service:
    @log_call
    def fetch(self, url, timeout=5):
        return f"{url} in {timeout}s"

Service().fetch("/api", timeout=2)
```
::

- [ ] `TypeError: wrapper() got an unexpected keyword argument 'timeout'`
- [ ] It works fine; `*args` silently absorbs keyword arguments too
- [ ] `self` is dropped and `fetch` is called as a static method
- [ ] `AttributeError: 'Service' object has no attribute 'fetch'`

<details>
<summary>Show Answer</summary>

**Answer:** A — `TypeError: wrapper() got an unexpected keyword argument 'timeout'`

**Explanation:** `*args` only collects positional arguments; a caller passing `timeout=2` as a keyword argument has nothing to bind to, so Python raises `TypeError`. A decorator meant to be transparent for arbitrary wrapped callables must define `wrapper(*args, **kwargs)` and forward both. Beginners assume `*args` is a catch-all for "anything," which is the trap here.

</details>
::

::question-wrapper{language="python"}
### Q13. What does this print?

::code-wrapper{language="python"}
```python
def broken_decorator(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        # forgot to return wrapper

@broken_decorator
def add(a, b):
    return a + b

print(add(2, 3))
```
::

- [ ] `TypeError: 'NoneType' object is not callable`
- [ ] `5`
- [ ] `None`
- [ ] `NameError: name 'add' is not defined`

<details>
<summary>Show Answer</summary>

**Answer:** A — `TypeError: 'NoneType' object is not callable`

**Explanation:** **Debug** — `broken_decorator` never returns `wrapper` (or anything), so it implicitly returns `None`. Since `add = broken_decorator(add)`, the name `add` is rebound to `None`, and calling `add(2, 3)` tries to call `None`, which raises `TypeError`. The fix is to always `return wrapper` from the decorator — a missing `return` is one of the most common real-world decorator bugs, and it fails at the call site, far from the actual mistake.

</details>
::

::question-wrapper{language="python"}
### Q14. What is printed by this decorator-with-arguments code?

::code-wrapper{language="python"}
```python
import functools

def repeat(times):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(times):
                results.append(func(*args, **kwargs))
            return results
        return wrapper
    return decorator

@repeat(times=2)
def shout(word):
    return word.upper()

print(shout("hi"))
```
::

- [ ] `['HI', 'HI']`
- [ ] `'HIHI'`
- [ ] `TypeError: decorator() missing 1 required positional argument: 'func'`
- [ ] `['hi', 'hi']`

<details>
<summary>Show Answer</summary>

**Answer:** A — `['HI', 'HI']`

**Explanation:** `repeat(times=2)` returns `decorator`, which is then applied to `shout`, producing `wrapper`. Calling `shout("hi")` actually calls `wrapper("hi")`, which invokes the original `shout("hi")` twice and collects the (already-uppercased) results into a list. Option C is the mistake of thinking `@repeat(times=2)` applies `repeat` directly to the function instead of first calling it to get a decorator.

</details>
::

::question-wrapper{language="python"}
### Q15. Which is the idiomatic, most robust way to write a decorator that is meant to work on any function signature?

::code-wrapper{language="python"}
```python
import functools

def timed(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        try:
            return func(*args, **kwargs)
        finally:
            print(f"{func.__name__} took {time.perf_counter() - start:.4f}s")
    return wrapper
```
::

- [ ] This version — `*args, **kwargs` forwards any signature, `functools.wraps` preserves metadata, and `finally` ensures timing prints even if `func` raises
- [ ] Same, but without `functools.wraps`, since it only affects `help()` output and nothing functional
- [ ] Same, but without `try`/`finally`, since decorators shouldn't handle control flow
- [ ] Same, but replacing `*args, **kwargs` with the wrapped function's exact named parameters for clarity

<details>
<summary>Show Answer</summary>

**Answer:** A — This version — `*args, **kwargs` forwards any signature, `functools.wraps` preserves metadata, and `finally` ensures timing prints even if `func` raises

**Explanation:** **Idiom** — A general-purpose decorator must accept and forward an arbitrary signature (`*args, **kwargs`), preserve introspection metadata (`functools.wraps`), and use `finally` so cleanup/logging code runs even when the wrapped function raises. Skipping `wraps` (option B) is a real bug in production code — it breaks tools that rely on `__name__`/`__qualname__`, such as API routers and test discovery. Hardcoding named parameters (option D) makes the decorator unreusable for any other function.

</details>
::

::question-wrapper
### Q16. A decorator needs to work both as `@cache` (no parentheses) and `@cache(maxsize=100)` (with parentheses). What is the idiomatic way to support both forms?

- [ ] Detect whether the single positional argument is callable: if so, treat it as the direct-decoration case and decorate immediately; otherwise treat it as configuration and return a real decorator
- [ ] Always require parentheses and document that `@cache` alone is unsupported
- [ ] Overload the function name with two different `def cache` definitions
- [ ] Use `*args` and assume the first argument is always the function

<details>
<summary>Show Answer</summary>

**Answer:** A — Detect whether the single positional argument is callable: if so, treat it as the direct-decoration case and decorate immediately; otherwise treat it as configuration and return a real decorator

**Explanation:** **Idiom** — Libraries like `functools.lru_cache` (Python 3.8+) support this dual-mode pattern by checking `callable(arg) and not kwargs` to distinguish "I was called directly on a function" from "I was called with configuration and need to return a decorator." Requiring parentheses always (option B) is simpler but breaks compatibility with existing dual-mode decorators users expect to work either way; redefining a function twice (option C) simply overwrites the first definition rather than overloading it.

</details>
::

::question-wrapper
### Q17. For simple memoization of a pure function keyed on its arguments, which is the best-practice choice?

- [ ] `functools.lru_cache`, since it's a battle-tested, thread-safe-for-reads standard-library decorator with eviction support
- [ ] A hand-rolled decorator using a plain `dict` with no size limit, since it gives identical behavior with less import overhead
- [ ] A global mutable list scanned linearly for matching arguments on every call
- [ ] Re-running the function every time, since memoization is rarely worth the complexity

<details>
<summary>Show Answer</summary>

**Answer:** A — `functools.lru_cache`, since it's a battle-tested, thread-safe-for-reads standard-library decorator with eviction support

**Explanation:** **Idiom** — `functools.lru_cache` handles hashable-argument caching, an optional `maxsize` for bounded memory, and `cache_info()` for introspection, all without extra code. A hand-rolled unbounded dict cache (option B) is a memory-leak risk in long-running processes since entries are never evicted; a linear-scan list (option C) is both slower and more error-prone at determining argument equality.

</details>
::

::question-wrapper
### Q18. In modern Python 3 code, which is the preferred way to maintain a mutable counter shared between an outer function and its inner closures?

- [ ] `nonlocal`, since it clearly expresses intent and avoids indirection through a container object
- [ ] A single-element list, e.g. `count = [0]`, mutated via `count[0] += 1`, because closures can't rebind outer names at all
- [ ] A global variable, since it's simplest
- [ ] A mutable default argument on the inner function

<details>
<summary>Show Answer</summary>

**Answer:** A — `nonlocal`, since it clearly expresses intent and avoids indirection through a container object

**Explanation:** **Idiom** — `nonlocal` (Python 3+) directly and readably rebinds an enclosing-scope variable. The list-mutation trick (option B) was a common workaround in Python 2, which lacked `nonlocal`, but it's now considered a code smell — it mutates a container instead of rebinding a name, which is harder to read and easy to mix up with genuinely shared mutable state. A global (option C) unnecessarily widens scope beyond the closure's own use case.

</details>
::

::question-wrapper{language="python"}
### Q19. A class-based decorator is implemented using `__call__`. Which statement about it is correct?

::code-wrapper{language="python"}
```python
import functools

class CountCalls:
    def __init__(self, func):
        functools.update_wrapper(self, func)
        self.func = func
        self.calls = 0

    def __call__(self, *args, **kwargs):
        self.calls += 1
        return self.func(*args, **kwargs)

@CountCalls
def greet():
    return "hi"
```
::

- [ ] `functools.update_wrapper` is needed here (the class-based equivalent of `functools.wraps`) to copy `__name__`/`__doc__` from `func` onto the `CountCalls` instance, and `greet.calls` is now readable state that a plain closure-based decorator can't expose as cleanly
- [ ] Class-based decorators can never preserve the original function's metadata, unlike function-based ones
- [ ] `@CountCalls` is invalid syntax because decorators must be functions, not classes
- [ ] `self.calls` resets to `0` on every call to `greet()`

<details>
<summary>Show Answer</summary>

**Answer:** A — `functools.update_wrapper` is needed here (the class-based equivalent of `functools.wraps`) to copy `__name__`/`__doc__` from `func` onto the `CountCalls` instance, and `greet.calls` is now readable state that a plain closure-based decorator can't expose as cleanly

**Explanation:** **Idiom** — Any callable object can be a decorator; a class implementing `__call__` is a common pattern when the decorator needs to hold persistent, easily-inspectable state (like a call counter) rather than hiding it in a closure cell. `functools.update_wrapper` is the underlying function that `functools.wraps` calls internally, applicable to any object, not just nested functions. `self.calls` persists across calls precisely because `__init__` runs once, at decoration time, not on every invocation.

</details>
::

::question-wrapper{language="python"}
### Q20. A web framework uses `@app.route("/users")` above `@login_required` above the view function. What does the ordering imply, and why does it matter?

::code-wrapper{language="python"}
```python
@app.route("/users")
@login_required
def list_users():
    ...
```
::

- [ ] `login_required` wraps `list_users` first (closest decorator), so the auth check runs on every request before the route even reaches the view logic; reversing the order would let `app.route` register the *unwrapped* view, bypassing the auth check entirely
- [ ] The order is purely stylistic; Python decorators are commutative regardless of which one is listed first
- [ ] `app.route` always executes before `login_required` no matter the order written, because routing decorators have special priority
- [ ] Both decorators wrap independently and neither can see the other's effect

<details>
<summary>Show Answer</summary>

**Answer:** A — `login_required` wraps `list_users` first (closest decorator), so the auth check runs on every request before the route even reaches the view logic; reversing the order would let `app.route` register the *unwrapped* view, bypassing the auth check entirely

**Explanation:** **Idiom** — Since decoration is `list_users = app.route("/users")(login_required(list_users))`, `login_required` produces the auth-checking wrapper first, and *that* wrapper is what gets registered as the route handler. If the order were flipped, `app.route` would register the raw `list_users`, and `login_required` would wrap a function that's no longer connected to any route — a real, security-relevant bug that decorator-order mistakes can cause in production frameworks like Flask.

</details>
::
