# 20 — Type Hints & Typing

::question-wrapper{language="python"}
### Q1. What happens when the following code is run with plain CPython (no external tool)?

::code-wrapper{language="python"}
```python
def add(a: int, b: int) -> int:
    return a + b

print(add("3", "4"))
```
::

- [ ] Raises a `TypeError` because `"3"` is not an `int`
- [ ] Raises a `TypeError` because the return value doesn't match `int`
- [ ] Prints `"34"` with no error at all
- [ ] Raises a `SyntaxError` at import time because the annotations are violated

<details>
<summary>Show Answer</summary>

**Answer:** C — Prints `"34"` with no error at all

**Explanation:** **Safety:** Python's type hints are not enforced at runtime by the interpreter. `int` on `a` and `b` is purely documentation/metadata (stored in `__annotations__`) unless a separate tool checks it. Since `str + str` is valid Python, `add("3", "4")` just concatenates and returns `"34"`. The tempting answers assume CPython behaves like a statically typed language and raises on mismatched types — it does not; only static checkers like mypy or pyright, or runtime validators like `pydantic`, would flag this.

</details>
::

::question-wrapper{language="python"}
### Q2. Which two type hints are exactly equivalent?

::code-wrapper{language="python"}
```python
from typing import Optional, Union

def f(x: Optional[str]) -> None: ...
def g(x: Union[str, None]) -> None: ...
```
::

- [ ] They are equivalent — `Optional[str]` is defined as `Union[str, None]`
- [ ] They are different — `Optional[str]` also allows omitting the argument entirely
- [ ] They are different — `Union[str, None]` allows `int` too, by widening
- [ ] They are equivalent only under Python 3.11+

<details>
<summary>Show Answer</summary>

**Answer:** A — They are equivalent — `Optional[str]` is defined as `Union[str, None]`

**Explanation:** `Optional[X]` is literally shorthand in the `typing` module for `Union[X, None]` — nothing more. A very common misconception (option B) is that `Optional` makes an argument optional in the sense of having a default value; it does not — you still must pass something explicitly unless you separately give the parameter a default like `= None`. This equivalence has held since `Optional` was introduced and is not a 3.11 change.

</details>
::

::question-wrapper{language="python"}
### Q3. What is the key practical difference between annotating a parameter as `Any` versus `object`?

::code-wrapper{language="python"}
```python
from typing import Any

def handle_any(x: Any) -> None:
    x.whatever_method()

def handle_object(x: object) -> None:
    x.whatever_method()
```
::

- [ ] There is no difference — both disable all static type checking on `x`
- [ ] `object` disables checking, `Any` is checked strictly
- [ ] `Any` tells the checker to skip checking on `x` entirely; `object` is a real type, so calling an arbitrary method on it is a static type error
- [ ] `Any` and `object` both restrict `x` to have no methods at all

<details>
<summary>Show Answer</summary>

**Answer:** C — `Any` tells the checker to skip checking on `x` entirely; `object` is a real type, so calling an arbitrary method on it is a static type error

**Explanation:** **Idiom:** `object` is the actual root of Python's type hierarchy — every value is an `object`, but a static checker only knows about the methods `object` itself defines (`__eq__`, `__repr__`, etc.), so `handle_object` would fail a mypy check on `x.whatever_method()`. `Any` is special-cased by type checkers to be compatible with everything in both directions, effectively opting `x` out of static checking. Beginners often assume `object` is the "accept anything, no checks" type since it's the base of everything — that's actually what `Any` is for.

</details>
::

::question-wrapper{language="python"}
### Q4. What does this generic function signature guarantee to a static type checker?

::code-wrapper{language="python"}
```python
from typing import TypeVar

T = TypeVar("T")

def first(items: list[T]) -> T:
    return items[0]

result = first([1, 2, 3])
```
::

- [ ] `result` is inferred as `Any` because `TypeVar` can't track concrete types
- [ ] `result` is inferred as `int`, because the checker binds `T` to `int` from the call site
- [ ] `result` is inferred as `list[int]`
- [ ] `TypeVar` forces a runtime check that all list items share the same type

<details>
<summary>Show Answer</summary>

**Answer:** B — `result` is inferred as `int`, because the checker binds `T` to `int` from the call site

**Explanation:** A `TypeVar` lets a static checker propagate a concrete type through a generic function: given `list[int]`, `T` is bound to `int` for that call, so the checker infers `first(...)` returns `int`. Nothing about this is enforced by CPython at runtime (option D is wrong — no runtime check ever happens); `TypeVar` is a pure static-analysis construct. Option A is the common mistake of assuming generics degrade to `Any`, when in fact the whole point of `TypeVar` is to preserve the specific type through the call.

</details>
::

::question-wrapper
### Q5. Which statement correctly describes how `mypy` relates to running your program?

- [ ] `mypy` is a runtime import that Python executes automatically before `main()`
- [ ] `mypy` is a separate static-analysis tool you run against your source files; it reports type errors but does not change how the code executes
- [ ] `mypy` patches the CPython interpreter to raise `TypeError` on hint violations
- [ ] `mypy` and type hints are required for Python code to run at all

<details>
<summary>Show Answer</summary>

**Answer:** B — `mypy` is a separate static-analysis tool you run against your source files; it reports type errors but does not change how the code executes

**Explanation:** `mypy` (like `pyright`) reads your source, checks annotations for consistency, and reports diagnostics — completely separately from `python script.py`. It never modifies runtime behavior or the interpreter. Options A, C, and D describe a form of enforcement Python simply does not have out of the box; type hints are always optional documentation to CPython itself.

</details>
::

::question-wrapper
### Q6. In modern Python (3.9+), which is the recommended way to annotate "a list of integers" without importing anything extra?

- [ ] `List[int]` from `typing`, always
- [ ] `list[int]`, using the built-in `list` directly as a generic
- [ ] `list(int)`
- [ ] `list<int>`

<details>
<summary>Show Answer</summary>

**Answer:** B — `list[int]`, using the built-in `list` directly as a generic

**Explanation:** **Idiom:** PEP 585 (Python 3.9+) made the built-in collection types (`list`, `dict`, `set`, `tuple`, etc.) directly subscriptable for annotations, so `list[int]` works without importing `typing.List`. `typing.List[int]` (option A) still works for backward compatibility but is now considered legacy style. `list(int)` (option C) is actually a runtime call to the `list` constructor with `int` as an argument — not an annotation at all, and it would raise `TypeError` if actually executed since `int` isn't iterable.

</details>
::

::question-wrapper
### Q7. What does `Callable[[int, str], bool]` describe?

- [ ] A callable that takes any two positional arguments and returns either an `int`, `str`, or `bool`
- [ ] A callable taking exactly one `int` argument and one `str` argument (positionally) and returning a `bool`
- [ ] A tuple of three types: `int`, `str`, and `bool`
- [ ] A callable that takes an `int` or `str` and returns `bool`

<details>
<summary>Show Answer</summary>

**Answer:** B — A callable taking exactly one `int` argument and one `str` argument (positionally) and returning a `bool`

**Explanation:** `Callable[[ArgTypes...], ReturnType]` uses a list of the parameter types (positional-only, in order) as the first element and the return type as the second. So `Callable[[int, str], bool]` means "call it with `(some_int, some_str)` and get a `bool` back." Option A misreads the union-like syntax that doesn't apply here; `Callable`'s argument list is positional and ordered, not a set of alternatives.

</details>
::

::question-wrapper{language="python"}
### Q8. A class needs to reference its own type in a method signature before the class body finishes executing. Which approach avoids a `NameError` at class-definition time on Python versions before 3.10 without `from __future__ import annotations`?

::code-wrapper{language="python"}
```python
class Node:
    def __init__(self, value: int) -> None:
        self.value = value
        self.next: "Node" | None = None
```
::

- [ ] It will always raise `NameError` regardless of quoting
- [ ] Quoting the type as a string forward reference (`"Node"`) defers evaluation so it works even though `Node` isn't fully defined yet
- [ ] You must define `Node` twice — once as a stub, once for real
- [ ] Forward references only work inside `@dataclass` classes

<details>
<summary>Show Answer</summary>

**Answer:** B — Quoting the type as a string forward reference (`"Node"`) defers evaluation so it works even though `Node` isn't fully defined yet

**Explanation:** **Debug:** Annotations are normally evaluated at function-definition time by default in older Python, so referencing the enclosing class by name before its `class` statement completes would fail. Wrapping the reference in quotes makes it a forward reference — a string that a type checker parses lazily instead of the interpreter evaluating it immediately. Note the snippet mixes syntaxes for illustration (the `"Node" | None` union-with-string form needs care), but the core mechanism being tested is that string annotations sidestep the ordering problem, unlike option A's assumption that it always fails.

</details>
::

::question-wrapper{language="python"}
### Q9. Why would a codebase use `if TYPE_CHECKING:` around an import?

::code-wrapper{language="python"}
```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from mymodule import HeavyClass

def process(item: "HeavyClass") -> None: ...
```
::

- [ ] To make the import faster by caching it
- [ ] `TYPE_CHECKING` is always `True`, so this is identical to a normal import
- [ ] `TYPE_CHECKING` is `False` at runtime, so the import (which might cause a circular import or unnecessary runtime cost) is skipped when the program actually executes, while static checkers still see it and validate the annotation
- [ ] It silences all mypy errors in the module

<details>
<summary>Show Answer</summary>

**Answer:** C — `TYPE_CHECKING` is `False` at runtime, so the import is skipped when the program executes, while static checkers still see it and validate the annotation

**Explanation:** `typing.TYPE_CHECKING` is `False` during normal interpretation and treated as `True` by static analyzers like mypy. This lets you import a class purely for annotation purposes — avoiding a circular import or a heavy/optional dependency at runtime — while the checker still resolves the forward-referenced type string. Assuming it's always `True` (option B) would mean the import always executes, defeating the entire purpose of the pattern.

</details>
::

::question-wrapper{language="python"}
### Q10. What does `UserId = NewType("UserId", int)` actually produce at runtime?

::code-wrapper{language="python"}
```python
from typing import NewType

UserId = NewType("UserId", int)
uid = UserId(42)
print(type(uid), uid + 1)
```
::

- [ ] A new subclass of `int` with `UserId` as its runtime type
- [ ] A callable that at runtime just returns its argument unchanged — `type(uid)` is `int`, and `uid + 1` works as plain int arithmetic; the distinctness only exists for static checkers
- [ ] A `TypeError`, because `NewType` requires a class, not a builtin
- [ ] A dataclass wrapping the int with a `.value` attribute

<details>
<summary>Show Answer</summary>

**Answer:** B — A callable that at runtime just returns its argument unchanged; the distinctness only exists for static checkers

**Explanation:** **Idiom:** `NewType` creates a lightweight identity function at runtime — calling `UserId(42)` just returns `42`, and `type(uid)` prints `<class 'int'>`, not some special `UserId` type. Its entire purpose is to let a static checker treat `UserId` and plain `int` as distinct (so you can't accidentally pass a raw `int` where a `UserId` is expected) without any runtime wrapping cost. Believing it creates a real subclass (option A) is the most common misunderstanding — that's what subclassing `int` directly would do, not `NewType`.

</details>
::

::question-wrapper{language="python"}
### Q11. What runtime guarantee does a `TypedDict` provide?

::code-wrapper{language="python"}
```python
from typing import TypedDict

class Movie(TypedDict):
    title: str
    year: int

m: Movie = {"title": "Arrival", "year": "2016"}
print(m)
```
::

- [ ] None — at runtime `m` is a plain `dict`; the `"year": "2016"` string instead of `int` will not raise anything, and this only shows up as an error under a static checker
- [ ] A `TypeError` is raised immediately because `year` should be an `int`
- [ ] `TypedDict` automatically coerces `"2016"` to `2016`
- [ ] Missing required keys raise a `KeyError` at construction time

<details>
<summary>Show Answer</summary>

**Answer:** A — None — at runtime `m` is a plain `dict`; the string year does not raise, and only a static checker flags it

**Explanation:** `TypedDict` exists purely to give static checkers shape information about a `dict`'s expected keys and value types; at runtime, `Movie` instances are ordinary `dict` objects with zero validation, coercion, or enforcement. Both the type mismatch (`"2016"` vs `int`) and even a missing required key would run without error — only tools like mypy catch them. For actual runtime validation, you'd reach for something like `pydantic` or manual checks, not `TypedDict`.

</details>
::

::question-wrapper{language="python"}
### Q12. What happens if you call a function decorated with multiple `@overload` signatures using arguments that don't match any of the declared overloads, when running the actual script (not a type checker)?

::code-wrapper{language="python"}
```python
from typing import overload

@overload
def process(x: int) -> int: ...
@overload
def process(x: str) -> str: ...
def process(x):
    return x

process(3.14)
```
::

- [ ] Raises `TypeError` immediately because `float` matches no overload
- [ ] Runs fine at runtime — only the final, un-decorated implementation actually executes, and `@overload` stubs are ignored by the interpreter; only a static checker would flag the `float` argument as invalid
- [ ] Silently returns `None` because no overload matched
- [ ] The `@overload` decorator dispatches to the closest matching stub automatically

<details>
<summary>Show Answer</summary>

**Answer:** B — Runs fine at runtime; `@overload` stubs are ignored by the interpreter and only the real implementation executes

**Explanation:** **Debug:** `@overload`-decorated stub bodies (the `...` ones) are never actually called — Python discards them, and only the final non-decorated `process(x): return x` implementation runs. `@overload` exists solely so a static checker can offer precise per-signature type checking and autocompletion; there is no runtime dispatch mechanism at all (option D is a common false assumption — Python has no built-in multiple dispatch). Since the concrete implementation accepts anything, `process(3.14)` just returns `3.14` with no error.

</details>
::

::question-wrapper
### Q13. A function is annotated `def get_ids() -> list[int]:` but its body is `return []`. Is this a problem at runtime?

- [ ] Yes — an empty list can never satisfy `list[int]`, so this raises `TypeError` on return
- [ ] No — an empty list contains no elements to contradict `int`, and since hints aren't enforced at runtime anyway, this runs without any error regardless
- [ ] Yes, but only in Python 3.12+
- [ ] No, but it silently converts to `[0]`

<details>
<summary>Show Answer</summary>

**Answer:** B — No; an empty list vacuously satisfies the hint, and runtime never checks it anyway

**Explanation:** Two separate reasons both point to "no error": logically, `list[int]` is satisfied vacuously by an empty list since there's no element violating the constraint, and practically, CPython never inspects the return annotation at runtime regardless. Even returning `["not", "an", "int"]` from this function would run without any exception — the mismatch would only surface as a static type error under mypy.

</details>
::

::question-wrapper{language="python"}
### Q14. What is the correct, type-hint-safe way to give a `dataclass` field a mutable default value?

::code-wrapper{language="python"}
```python
from dataclasses import dataclass, field

@dataclass
class Config:
    tags: list[str] = field(default_factory=list)
```
::

- [ ] `tags: list[str] = []` — dataclasses handle mutable defaults safely, unlike plain functions
- [ ] The shown `field(default_factory=list)` form — a bare mutable literal default (`= []`) raises `ValueError` at class-definition time, and `default_factory` calls the factory fresh for each new instance
- [ ] `tags: list[str] = None` and check for `None` in `__post_init__`
- [ ] `tags: List[str] = list` without calling it

<details>
<summary>Show Answer</summary>

**Answer:** B — `field(default_factory=list)`; a bare `= []` default raises `ValueError` at class-definition time

**Explanation:** **Safety:** `@dataclass` explicitly detects a mutable literal default (`list`, `dict`, `set`) for a field and raises `ValueError: mutable default <class 'list'> for field tags is not allowed` at class-definition time — dataclasses deliberately guard against the classic Python mutable-default-argument footgun rather than silently sharing one list across instances. `field(default_factory=list)` tells it to call `list()` fresh per instance instead. Option A is the tempting mistake because dataclasses look like they might "fix" the mutable-default problem automatically — they instead refuse to let you make it.

</details>
::

::question-wrapper{language="python"}
### Q15. What is the idiomatic reason to prefer `Optional[str] = None` over just writing `str = None` for a parameter default?

::code-wrapper{language="python"}
```python
def greet(name: str = None) -> None: ...
def greet2(name: Optional[str] = None) -> None: ...
```
::

- [ ] `str = None` is a `SyntaxError`
- [ ] They behave identically at runtime, but `str = None` is a lie to static checkers — `None` isn't a `str`, so mypy flags it, while `Optional[str] = None` accurately documents that `None` is a valid value
- [ ] `Optional[str]` makes the parameter itself optional to omit, while `str = None` does not
- [ ] `Optional[str] = None` is slower because it wraps the value in a union object at call time

<details>
<summary>Show Answer</summary>

**Answer:** B — They behave identically at runtime; `str = None` is inaccurate to static checkers, while `Optional[str]` documents that `None` is valid

**Explanation:** **Idiom:** Both run identically since CPython ignores hints, but a strict type checker treats `name: str = None` as an error because `None` is not a `str` and the annotation says nothing about it being optional. `Optional[str]` correctly communicates "this can be a `str` or `None`" to both the checker and human readers. Runtime "optionality" (whether you can omit the argument) comes purely from the `= None` default, not from `Optional` — that's a separate, commonly conflated concept (option C).

</details>
::

::question-wrapper{language="python"}
### Q16. When should you prefer `typing.Protocol` over an abstract base class (`abc.ABC`) for defining an interface?

::code-wrapper{language="python"}
```python
from typing import Protocol

class SupportsClose(Protocol):
    def close(self) -> None: ...

def cleanup(resource: SupportsClose) -> None:
    resource.close()
```
::

- [ ] Never — `Protocol` is strictly worse and slower than `ABC`
- [ ] When you want structural ("duck") typing — any object with a matching `close()` method satisfies the type statically without explicitly inheriting from `SupportsClose`, unlike `ABC` which requires nominal subclassing
- [ ] `Protocol` requires runtime registration via `register()`, same as `ABC`
- [ ] `Protocol` can only be used for dataclasses

<details>
<summary>Show Answer</summary>

**Answer:** B — Use `Protocol` for structural typing; any object with a matching method satisfies it without explicit inheritance, unlike `ABC`'s nominal typing

**Explanation:** **Idiom:** `Protocol` implements structural subtyping (PEP 544) — a static checker considers any class that happens to implement the right methods/attributes as compatible, with zero inheritance relationship required. `ABC` requires explicit `class Foo(SupportsClose):` nominal subclassing (or `register()`) to be recognized. This makes `Protocol` a good fit for third-party classes you can't modify to add a base class, which is a common real-world constraint `ABC` can't satisfy as cleanly.

</details>
::

::question-wrapper{language="python"}
### Q17. What does marking a variable with `Final` actually do?

::code-wrapper{language="python"}
```python
from typing import Final

MAX_RETRIES: Final = 3
MAX_RETRIES = 5
```
::

- [ ] It raises `TypeError` immediately on the reassignment line
- [ ] It is purely a static-analysis hint — mypy will flag the reassignment as an error, but CPython executes the second line without complaint and `MAX_RETRIES` becomes `5`
- [ ] It makes the name read-only via the C-level immutability flag
- [ ] `Final` only works inside classes, not at module level

<details>
<summary>Show Answer</summary>

**Answer:** B — It's purely a static-analysis hint; CPython executes the reassignment fine and `MAX_RETRIES` becomes `5`

**Explanation:** **Safety:** Like all typing constructs, `Final` carries zero runtime enforcement — it tells a checker "treat this as a constant; flag any rebinding," but the interpreter has no concept of a `Final`-protected name and simply rebinds it. Developers coming from languages with real `const`/`final` keywords (option C) often expect actual immutability enforcement, which Python's typing system deliberately does not provide — runtime constant-protection would need a different mechanism entirely (e.g., a custom descriptor or just convention).

</details>
::

::question-wrapper{language="python"}
### Q18. What is the purpose of `ClassVar` in this dataclass?

::code-wrapper{language="python"}
```python
from typing import ClassVar
from dataclasses import dataclass

@dataclass
class Counter:
    count: int = 0
    total_instances: ClassVar[int] = 0
```
::

- [ ] It marks `total_instances` as a per-instance field with default `0`, identical to `count`
- [ ] It tells both static checkers and `@dataclass` itself to treat `total_instances` as a class-level attribute, excluding it from the generated `__init__` and instance fields — unlike `count`, which becomes a normal `__init__` parameter
- [ ] It makes `total_instances` thread-safe automatically
- [ ] `ClassVar` has no effect on `@dataclass`'s generated code; it's purely cosmetic

<details>
<summary>Show Answer</summary>

**Answer:** B — It marks `total_instances` as class-level, excluded from the generated `__init__` and instance fields

**Explanation:** **Idiom:** `@dataclass` specifically inspects annotations for `ClassVar` and, unlike ordinary fields, excludes those attributes from the auto-generated `__init__`, `__repr__`, and `__eq__` — they remain shared class attributes, exactly like a normal class-body assignment without a dataclass. This is one of the few typing constructs that a real library (`dataclasses`) actually inspects at runtime rather than ignoring, which trips people up (option D) since most other hints genuinely are inert to runtime code.

</details>
::

::question-wrapper{language="python"}
### Q19. What is `cast()` for, and what does it do at runtime?

::code-wrapper{language="python"}
```python
from typing import cast

def get_config() -> dict:
    ...

raw = get_config()
port = cast(int, raw["port"])
```
::

- [ ] `cast(int, x)` converts `x` to an `int`, like `int(x)`, and raises if conversion fails
- [ ] `cast()` is a pure no-op at runtime — it returns `x` completely unchanged; it exists only to tell the static checker "trust me, treat this expression as `int`" when the checker can't infer it itself
- [ ] `cast()` validates that `x` is already an `int` and raises `TypeError` otherwise
- [ ] `cast()` deep-copies `x` before returning it

<details>
<summary>Show Answer</summary>

**Answer:** B — `cast()` is a pure no-op at runtime; it only tells the static checker to treat the expression as the given type

**Explanation:** **Debug:** `typing.cast(TypeHint, value)` is implemented essentially as `return value` — it performs no conversion, no validation, nothing. It exists purely to override a static checker's inferred type when you know something the checker can't (e.g., after runtime validation it doesn't understand). Confusing it with `int(x)` (option A) is a common and dangerous mistake: if `raw["port"]` is actually the string `"8080"`, `cast(int, raw["port"])` happily returns the string unchanged, and a later arithmetic operation on `port` will fail with a `TypeError` far from where the real bug is — the correct fix is to actually convert/validate the value, e.g. `port = int(raw["port"])`, not just cast it.

</details>
::

::question-wrapper{language="python"}
### Q20. What does declaring a `TypeVar` as covariant (`TypeVar("T_co", covariant=True)`) enable that an invariant `TypeVar` does not?

::code-wrapper{language="python"}
```python
from typing import TypeVar, Generic

T_co = TypeVar("T_co", covariant=True)

class ReadOnlyBox(Generic[T_co]):
    def __init__(self, item: T_co) -> None:
        self._item = item
    def get(self) -> T_co:
        return self._item

def print_box(box: "ReadOnlyBox[object]") -> None:
    print(box.get())

int_box: ReadOnlyBox[int] = ReadOnlyBox(42)
print_box(int_box)
```
::

- [ ] Covariance has a runtime performance cost proportional to the number of subtypes
- [ ] With a covariant `T_co`, a static checker allows `ReadOnlyBox[int]` to be used wherever `ReadOnlyBox[object]` is expected (since `int` is a subtype of `object`), matching the intuitive subtyping of read-only containers; an invariant `TypeVar` would reject this assignment as a type error even though it works fine at runtime
- [ ] Covariance lets you mutate `_item` to any type at runtime without restriction
- [ ] Covariant TypeVars are required for any class that uses `__init__`

<details>
<summary>Show Answer</summary>

**Answer:** B — Covariance lets a static checker treat `ReadOnlyBox[int]` as compatible with `ReadOnlyBox[object]`, matching real subtyping for read-only containers; invariant would reject it

**Explanation:** **Idiom:** By default, `TypeVar`s are invariant — a checker treats `ReadOnlyBox[int]` and `ReadOnlyBox[object]` as unrelated types even though `int` is an `object`. Marking `T_co` covariant tells the checker it's safe to treat the relationship like the underlying type relationship *because* the box is read-only (`get()` only, never accepts a new `T_co` from outside after construction) — this mirrors why `Sequence[int]` is a `Sequence[object]` in typeshed but `list[int]` is intentionally not treated as a `list[object]` (since `list` is mutable and allows appending, which invariance protects against). None of this variance machinery has any runtime effect at all — it exists solely to make static checking of generic containers match real-world expectations.

</details>
::
