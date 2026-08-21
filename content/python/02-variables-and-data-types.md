# 02 — Variables & Data Types

::question-wrapper{language="python"}
### Q1. Which statement correctly describes Python's object model?

::code-wrapper{language="python"}
```python
x = 42
def greet(): pass
```
::

- [ ] Only class instances are objects; primitives like `int` and functions are special language constructs
- [ ] Everything referenced by a name — integers, strings, functions, classes, and modules — is an object with a type and identity
- [ ] Only mutable values (lists, dicts) are objects; immutable values are stored inline, not as objects
- [ ] Objects exist only after you explicitly instantiate a class with `()`

<details>
<summary>Show Answer</summary>

**Answer:** B — Everything referenced by a name — integers, strings, functions, classes, and modules — is an object with a type and identity

**Explanation:** In Python, "everything is an object" is not a slogan but a literal implementation fact: `42`, `greet`, and even the `int` and `function` types themselves are objects with a type, an identity (`id()`), and typically attributes. Option A is the kind of assumption carried over from languages with true primitive types, which Python does not have. Option C is backwards — immutability affects whether an object can change in place, not whether it "is" an object. Option D is wrong because literals like `42` and `"hi"` are objects the moment they're created, with no explicit constructor call required.

</details>
::

::question-wrapper{language="python"}
### Q2. What happens when a name already bound to an `int` is reassigned to a `str`?

::code-wrapper{language="python"}
```python
status = 200
status = "OK"
```
::

- [ ] A `TypeError`, because a variable's type cannot change after first assignment
- [ ] This is legal — Python is dynamically typed, so a name can be rebound to any object regardless of the previous object's type
- [ ] The assignment silently converts `"OK"` to an integer to match the variable's original type
- [ ] It works only if both statements are inside the same `with` block

<details>
<summary>Show Answer</summary>

**Answer:** B — This is legal — Python is dynamically typed, so a name can be rebound to any object regardless of the previous object's type

**Explanation:** Python variables are just names in a namespace bound to objects; the name `status` has no fixed type of its own, so rebinding it to a `str` after an `int` is completely legal at runtime. Option A confuses Python with statically-typed languages that give variables a fixed declared type. Option C invents a coercion Python never performs implicitly between unrelated types. Option D's premise is irrelevant — no such scoping restriction exists.

</details>
::

::question-wrapper{language="python"}
### Q3. What is the core difference between `is` and `==`?

::code-wrapper{language="python"}
```python
a = [1, 2, 3]
b = [1, 2, 3]
```
::

- [ ] `is` checks value equality; `==` checks object identity
- [ ] `is` checks object identity (same object in memory); `==` checks value equality (calls `__eq__`)
- [ ] They are fully interchangeable for all built-in types
- [ ] `is` only works on numbers; `==` only works on strings

<details>
<summary>Show Answer</summary>

**Answer:** B — `is` checks object identity (same object in memory); `==` checks value equality (calls `__eq__`)

**Explanation:** `is` answers "are these the same object?" (equivalent to comparing `id(a) == id(b)`), while `==` answers "do these compare equal?" by invoking `__eq__`. In the code above, `a == b` is `True` (same contents) but `a is b` is `False` (two distinct list objects). Option A states the definitions backwards. Option C is a common trap — for mutable objects and most non-cached immutables, `is` and `==` diverge exactly as shown here. Option D is simply false; both operators work across types.

</details>
::

::question-wrapper
### Q4. What does the built-in `id()` function return?

- [ ] The object's type name as a string
- [ ] An integer that is guaranteed unique among currently alive objects during the program's run — in CPython, typically the object's memory address
- [ ] A cryptographic hash of the object's contents
- [ ] The line number where the object was created

<details>
<summary>Show Answer</summary>

**Answer:** B — An integer that is guaranteed unique among currently alive objects during the program's run — in CPython, typically the object's memory address

**Explanation:** `id()` returns a value guaranteed to be unique and constant for an object for its lifetime; CPython implements this as the object's memory address, but that detail is implementation-specific and should not be relied on beyond identity comparison. Option A describes `type(obj).__name__`. Option C describes `hash()`, which is a different, value-based concept and is not guaranteed unique. Option D describes nothing Python tracks by default.

</details>
::

::question-wrapper
### Q5. Which of these is the correct classification of built-in types by mutability?

- [ ] Mutable: `int`, `str`, `tuple` — Immutable: `list`, `dict`, `set`
- [ ] Mutable: `list`, `dict`, `set`, `bytearray` — Immutable: `int`, `float`, `str`, `tuple`, `bool`, `frozenset`
- [ ] All built-in container types are mutable; only numbers are immutable
- [ ] Mutability is a runtime flag you must set explicitly with `mutable=True`

<details>
<summary>Show Answer</summary>

**Answer:** B — Mutable: `list`, `dict`, `set`, `bytearray` — Immutable: `int`, `float`, `str`, `tuple`, `bool`, `frozenset`

**Explanation:** `list`, `dict`, `set`, and `bytearray` support in-place mutation (`append`, `__setitem__`, `add`, and friends), whereas `int`, `float`, `str`, `tuple`, `bool`, and `frozenset` cannot be changed after creation — any "modification" actually produces a new object. Option A has the classification exactly backwards. Option C is wrong because `tuple` and `frozenset` are containers that are nonetheless immutable. Option D describes a mechanism Python does not have — mutability is a property of the type, not a per-instance flag.

</details>
::

::question-wrapper{language="python"}
### Q6. What is the difference between `type(obj) == SomeClass` and `isinstance(obj, SomeClass)` when `obj` might be an instance of a subclass?

::code-wrapper{language="python"}
```python
class Animal: pass
class Dog(Animal): pass

d = Dog()
```
::

- [ ] They always behave identically for every object
- [ ] `type(d) == Animal` is `False` for a `Dog` instance, while `isinstance(d, Animal)` is `True`, because `isinstance` accounts for inheritance
- [ ] `type(d) == Animal` is `True` because Python treats subclasses as equal types
- [ ] `isinstance` only works with built-in types, not user-defined classes

<details>
<summary>Show Answer</summary>

**Answer:** B — `type(d) == Animal` is `False` for a `Dog` instance, while `isinstance(d, Animal)` is `True`, because `isinstance` accounts for inheritance

**Explanation:** `type(d)` returns the exact class `Dog`, which does not equal `Animal`, whereas `isinstance` walks the MRO and correctly reports that a `Dog` "is-an" `Animal`. This distinction matters constantly in real code that accepts subclasses polymorphically. Option A ignores exactly this divergence. Option C misunderstands `type()`, which never treats a subclass as equal to its parent. Option D is false — `isinstance` works uniformly across built-in and user-defined classes.

</details>
::

::question-wrapper{language="python"}
### Q7. What happens when you execute `b = a` where `a` is a `list`?

::code-wrapper{language="python"}
```python
a = [1, 2, 3]
b = a
```
::

- [ ] A new, independent copy of the list is created and bound to `b`
- [ ] `b` becomes a second name bound to the exact same list object as `a`; no copying occurs
- [ ] `b` is bound to a lazily-copied "view" that only copies on first mutation
- [ ] This raises a `TypeError` because lists cannot be reassigned

<details>
<summary>Show Answer</summary>

**Answer:** B — `b` becomes a second name bound to the exact same list object as `a`; no copying occurs

**Explanation:** Assignment in Python never copies an object; it binds a name to whatever object is on the right-hand side. So `a` and `b` here are two labels for one list, and `a is b` is `True` — mutating through either name affects what the other sees. Option A describes copy semantics found in some other languages, not Python's default `=`. Option C describes copy-on-write, a strategy CPython does not use for `list` assignment. Option D is false; reassignment is always legal.

</details>
::

::question-wrapper{language="python"}
### Q8. Given the following, what is printed, and why?

::code-wrapper{language="python"}
```python
a = 100
b = 100
print(a is b)

x = 1000
y = 1000
print(x is y)
```
::

- [ ] `True` then `True` — CPython always caches and reuses every integer object
- [ ] `True` then `False` — CPython pre-caches and interns small integers in the range -5 to 256, but not arbitrary larger integers, which may or may not be separate objects depending on context
- [ ] `False` then `False` — integers are never cached in CPython
- [ ] `False` then `True` — only large integers are cached to save memory

<details>
<summary>Show Answer</summary>

**Answer:** B — `True` then `False` — CPython pre-caches and interns small integers in the range -5 to 256, but not arbitrary larger integers, which may or may not be separate objects depending on context

**Explanation:** **Performance** — as a memory optimization, CPython pre-allocates and reuses a singleton object for every integer from -5 to 256, so `a is b` is reliably `True` for `100`. Numbers outside that range are ordinarily created fresh each time (`x is y` is typically `False`, though this is an implementation detail that can vary — e.g. constant-folding within the same compiled code unit can sometimes make it `True` too). The trap is treating `is` as safe for comparing arbitrary integers just because it "worked" during testing with small numbers; always use `==` for integer value comparison.

</details>
::

::question-wrapper{language="python"}
### Q9. What is the most accurate description of CPython string interning?

::code-wrapper{language="python"}
```python
s1 = "hello"
s2 = "hello"
print(s1 is s2)

s3 = "".join(["h", "e", "l", "l", "o"])
print(s1 is s3)
```
::

- [ ] All strings with equal content are always the same object, no matter how they are constructed
- [ ] Compile-time string literals that look like identifiers are commonly interned and may share identity, but strings built at runtime (e.g. via concatenation or `join`) are typically distinct objects even with identical content
- [ ] No strings are ever interned in CPython; `is` on strings is always `False`
- [ ] Interning applies only to numeric strings like `"123"`

<details>
<summary>Show Answer</summary>

**Answer:** B — Compile-time string literals that look like identifiers are commonly interned and may share identity, but strings built at runtime (e.g. via concatenation or `join`) are typically distinct objects even with identical content

**Explanation:** **Debug** — CPython, as an optimization, often interns short literals that resemble identifiers at compile time, so `s1 is s2` frequently prints `True`. But `s3`, built at runtime via `"".join(...)`, is a freshly allocated string object even though it's equal in content, so `s1 is s3` is `False`. This is a classic footgun: code that appears to work using `is` for string comparison in a quick test can silently break once the string is produced dynamically (e.g., from user input, formatting, or `+` concatenation). Always compare string values with `==`, never `is`.

</details>
::

::question-wrapper{language="python"}
### Q10. What does the following print, and why is it surprising?

::code-wrapper{language="python"}
```python
n = float("nan")
print(n == n)
print(n is n)
```
::

- [ ] `True` then `True` — a value always equals itself
- [ ] `False` then `True` — per IEEE 754, NaN never equals itself under `==`, but `is` compares identity, and it's the same object bound to `n` both times
- [ ] `False` then `False` — NaN breaks both identity and equality checks
- [ ] `True` then `False` — NaN is equal to itself but Python creates a new object on each reference

<details>
<summary>Show Answer</summary>

**Answer:** B — `False` then `True` — per IEEE 754, NaN never equals itself under `==`, but `is` compares identity, and it's the same object bound to `n` both times

**Explanation:** **Debug** — `NaN != NaN` is mandated by the IEEE 754 floating-point standard (any comparison involving NaN except `!=` is `False`), so `n == n` is `False` even though `n` is literally the same object as itself. `n is n` is `True` because identity doesn't care about the value-equality rules at all — it's trivially the same object. This is a real production gotcha: `x == x` is not a safe way to detect NaN; use `math.isnan(x)` instead, and remember `NaN in some_list` can silently fail to find a NaN it should logically match.

</details>
::

::question-wrapper{language="python"}
### Q11. Do two independently created empty tuples share identity?

::code-wrapper{language="python"}
```python
t1 = ()
t2 = ()
print(t1 is t2)
```
::

- [ ] No — every tuple literal, even an empty one, allocates a new object
- [ ] Yes — CPython caches a singleton empty tuple, so both names typically refer to the same object
- [ ] Only inside function bodies, never at module level
- [ ] Only if both are explicitly declared with `tuple()` instead of `()`

<details>
<summary>Show Answer</summary>

**Answer:** B — Yes — CPython caches a singleton empty tuple, so both names typically refer to the same object

**Explanation:** **Performance** — because the empty tuple is immutable and has no meaningful internal state to diverge, CPython optimizes by reusing one singleton empty-tuple object everywhere, so `t1 is t2` is `True`. This is a CPython implementation detail (not a language guarantee), unlike, say, empty lists, where `[] is []` is `False` because lists are mutable and must never be silently shared. The lesson generalizes: never write code whose correctness *depends* on such identity caching — use `==` for comparisons regardless.

</details>
::

::question-wrapper{language="python"}
### Q12. A function receives a mutable list and appends to it, but also reassigns the parameter name inside the function. What does the caller observe?

::code-wrapper{language="python"}
```python
def process(items):
    items.append("processed")
    items = ["replaced"]

data = ["order-1"]
process(data)
print(data)
```
::

- [ ] `["replaced"]` — the reassignment inside the function propagates back to the caller
- [ ] `["order-1", "processed"]` — the in-place `append` mutates the shared object the caller sees, but rebinding the local name `items` only affects the local scope, not the caller's `data`
- [ ] `["order-1"]` — function calls never affect the caller's objects
- [ ] A `TypeError`, because you cannot both mutate and reassign a parameter in the same function

<details>
<summary>Show Answer</summary>

**Answer:** B — `["order-1", "processed"]` — the in-place `append` mutates the shared object the caller sees, but rebinding the local name `items` only affects the local scope, not the caller's `data`

**Explanation:** Python passes arguments by binding the parameter name to the same object the caller passed ("pass by object reference"). `items.append(...)` mutates that shared list, so the caller sees it. But `items = ["replaced"]` merely rebinds the local name `items` to point at a brand-new list — it does not, and cannot, reach back and change what `data` points to. Beginners often expect either "everything propagates" (A) or "nothing propagates" (C); the real behavior is a mix, and it's the single most common source of confusion around Python's argument-passing model. Option D describes a restriction that doesn't exist.

</details>
::

::question-wrapper{language="python"}
### Q13. What does the following print, and what does it reveal about `bool`?

::code-wrapper{language="python"}
```python
print(True == 1)
print(isinstance(True, int))
print(True + True)
```
::

- [ ] `False`, `False`, `TypeError` — booleans are unrelated to integers
- [ ] `True`, `True`, `2` — `bool` is a subclass of `int` in Python, so `True`/`False` behave as `1`/`0` in arithmetic and comparisons
- [ ] `True`, `False`, `TypeError` — booleans equal integers by value but are not related by type
- [ ] `False`, `True`, `2` — `bool` inherits from `int` but the values are never equal

<details>
<summary>Show Answer</summary>

**Answer:** B — `True`, `True`, `2` — `bool` is a subclass of `int` in Python, so `True`/`False` behave as `1`/`0` in arithmetic and comparisons

**Explanation:** **Debug** — `bool` is literally a subclass of `int`, with `True` and `False` behaving as `1` and `0` respectively; that's why `True == 1` is `True`, `isinstance(True, int)` is `True`, and `True + True` evaluates to `2` without error. This surprises people who assume booleans are a wholly separate type. It has real consequences: `sum([True, False, True])` yields `2`, and `{1: "a", True: "b"}` collapses to a single key because `1 == True` and `hash(1) == hash(True)`.

</details>
::

::question-wrapper
### Q14. Is it safe to compare `id(obj_a) == id(obj_b)` for two objects that existed at *different, non-overlapping* points in a long-running program to conclude they were "the same object"?

- [ ] Yes, `id()` values are globally unique for all time, so a match always proves it was the same object
- [ ] No — once an object is garbage-collected, CPython may reuse its freed memory address for a completely unrelated new object, so a matching `id()` across non-overlapping lifetimes proves nothing
- [ ] No, because `id()` is randomized on every call and never repeats
- [ ] Yes, but only for immutable types

<details>
<summary>Show Answer</summary>

**Answer:** B — No — once an object is garbage-collected, CPython may reuse its freed memory address for a completely unrelated new object, so a matching `id()` across non-overlapping lifetimes proves nothing

**Explanation:** **Safety** — `id()` uniqueness is only guaranteed among objects that are alive *at the same time*; CPython's allocator is free to hand a freshly freed address to a brand-new, unrelated object. A long-running service that logs `id()` values to "track" objects across time can be misled into thinking two clearly different objects are the same one. The safe pattern is to keep a live reference (preventing garbage collection) for as long as identity needs to be checked, rather than persisting bare `id()` integers.

</details>
::

::question-wrapper{language="python"}
### Q15. What is the idiomatic way to check whether `obj` is an instance of `MyClass` or one of its subclasses?

::code-wrapper{language="python"}
```python
if isinstance(obj, MyClass):
    ...
```
::

- [ ] `if type(obj) == MyClass:`
- [ ] `if isinstance(obj, MyClass):`
- [ ] `if obj.__class__.__name__ == "MyClass":`
- [ ] `if str(type(obj)) == "MyClass":`

<details>
<summary>Show Answer</summary>

**Answer:** B — `if isinstance(obj, MyClass):`

**Explanation:** **Idiom** — `isinstance` is the idiomatic, subclass-aware check and also gracefully supports checking against a tuple of types (`isinstance(obj, (int, float))`). Option A silently excludes legitimate subclass instances, which routinely breaks polymorphic code paths (e.g., custom exceptions or ORM model subclasses). Options C and D are fragile string-matching hacks that break under refactors, module renames, or subclassing, and are never the recommended approach.

</details>
::

::question-wrapper{language="python"}
### Q16. What is the idiomatic way to check whether a variable is `None`?

::code-wrapper{language="python"}
```python
if value is None:
    ...
```
::

- [ ] `if value == None:`
- [ ] `if value is None:`
- [ ] `if not value:`
- [ ] `if value.__eq__(None):`

<details>
<summary>Show Answer</summary>

**Answer:** B — `if value is None:`

**Explanation:** **Idiom** — `None` is a singleton, so identity comparison is both correct and faster than an equality check, and PEP 8 explicitly recommends `is`/`is not` for `None` comparisons. Option A technically often works too (since `None.__eq__` falls back to identity-like behavior for the default case), but it's non-idiomatic and, critically, an object could override `__eq__` to claim it equals `None`, silently breaking an `==` check in a way `is` never can. Option C is a different, broader check — it also matches falsy values like `0`, `""`, and `[]`, which is a common and dangerous conflation with "is None". Option D is needlessly indirect and bypasses Python's reflected-comparison protocol.

</details>
::

::question-wrapper
### Q17. Should application logic ever rely on CPython's small-integer or string-interning caching (i.e., using `is` where `==` is meant)?

- [ ] Yes — it's a documented, guaranteed language feature safe to depend on
- [ ] No — it is a CPython implementation detail that can change between versions/implementations (e.g., PyPy) and even between contexts (interactive shell vs. module vs. `-O` optimizations); use `==` for value comparisons
- [ ] Yes, but only for integers, never for strings
- [ ] It doesn't matter, since `is` and `==` always agree for built-in immutable types

<details>
<summary>Show Answer</summary>

**Answer:** B — No — it is a CPython implementation detail that can change between versions/implementations (e.g., PyPy) and even between contexts (interactive shell vs. module vs. `-O` optimizations); use `==` for value comparisons

**Explanation:** **Portability** — small-int caching and string interning are documented as CPython optimizations, not language guarantees; other implementations (PyPy, Jython) or even future CPython releases are free to cache differently, or not at all. Code that happens to pass tests because `100 is 100` was `True` can fail unpredictably in a different environment or with slightly larger numbers. Option D is directly contradicted by the `x = 1000; y = 1000` example seen earlier in this quiz, where `is` and `==` diverge.

</details>
::

::question-wrapper{language="python"}
### Q18. Which of the following can be used as a key in a `dict`?

::code-wrapper{language="python"}
```python
config = {
    ("region", "us-east-1"): "primary",
}
```
::

- [ ] A `list`, since dict keys can be any object
- [ ] A `tuple` of hashable elements, like `("region", "us-east-1")`, since it is immutable and hashable
- [ ] A `dict`, since nesting dictionaries is common
- [ ] A `set`, since sets are also collections

<details>
<summary>Show Answer</summary>

**Answer:** B — A `tuple` of hashable elements, like `("region", "us-east-1")`, since it is immutable and hashable

**Explanation:** Dict keys must be hashable, which in practice means immutable (or at least implementing a stable `__hash__`); a `tuple` of hashable elements qualifies and is a common composite-key pattern. `list`, `dict`, and `set` are all mutable and unhashable by default, so using any of them as a key raises `TypeError: unhashable type`. This ties directly back to mutability: a hash must stay constant for an object's lifetime, and a mutable object's contents — and thus its "natural" hash — could change after insertion, which would corrupt the hash table.

</details>
::

::question-wrapper{language="python"}
### Q19. What is the correct, defensive way to convert untrusted user input like `"abc"` to an `int`?

::code-wrapper{language="python"}
```python
raw = input("Enter age: ")
try:
    age = int(raw)
except ValueError:
    age = None
```
::

- [ ] Call `int(raw)` directly and trust it will always succeed
- [ ] Wrap the conversion in `try/except ValueError` (as shown), since `int()` raises `ValueError` on non-numeric strings rather than returning a sentinel
- [ ] Use `float(raw)` instead, since it never raises exceptions
- [ ] Check `raw.isdigit()` only, since that alone guarantees `int()` will succeed for any valid integer, including negatives

<details>
<summary>Show Answer</summary>

**Answer:** B — Wrap the conversion in `try/except ValueError` (as shown), since `int()` raises `ValueError` on non-numeric strings rather than returning a sentinel

**Explanation:** `int()` (and `float()`) raise `ValueError` for unparseable strings instead of returning `None` or `0`, so unguarded conversion of external input is a production crash waiting to happen — catching `ValueError` is the correct handling. Option A ignores that risk entirely. Option C is wrong on its face — `float()` raises `ValueError` too (e.g., `float("abc")`). Option D is a subtle trap: `str.isdigit()` returns `False` for a leading `-` sign (`"-5".isdigit()` is `False`), so relying on it alone would reject legitimate negative integers even though `int("-5")` succeeds fine.

</details>
::

::question-wrapper{language="python"}
### Q20. Two engineers are debugging why mutating `settings_copy` also changed `settings` elsewhere in a large codebase. What is the most likely root cause?

::code-wrapper{language="python"}
```python
settings = {"debug": False}
settings_copy = settings
settings_copy["debug"] = True
print(settings["debug"])
```
::

- [ ] `settings_copy = settings` silently created a deep copy, and the bug lies elsewhere
- [ ] `settings_copy = settings` only creates a new alias to the same dict object — no copy was ever made, so mutating one mutates both; a real copy requires `dict(settings)`, `settings.copy()`, or `copy.deepcopy(settings)` for nested structures
- [ ] `dict` objects are immutable, so this code should have raised an exception
- [ ] This is a garbage collector bug and cannot be fixed in user code

<details>
<summary>Show Answer</summary>

**Answer:** B — `settings_copy = settings` only creates a new alias to the same dict object — no copy was ever made, so mutating one mutates both; a real copy requires `dict(settings)`, `settings.copy()`, or `copy.deepcopy(settings)` for nested structures

**Explanation:** **Idiom** — this is the single most common real-world "why did my data change on its own" bug: plain `=` never copies a mutable object, it only adds another name pointing at the same one. The fix is to explicitly request a copy — a shallow copy (`dict(settings)` / `settings.copy()`) is enough for a flat dict of immutable values, while nested mutable structures need `copy.deepcopy`. Option A describes behavior `=` simply does not have. Option C is false — `dict` is mutable by design. Option D misattributes an application-level aliasing bug to the garbage collector, which is unrelated.

</details>
::
