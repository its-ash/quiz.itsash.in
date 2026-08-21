# 12 — Classes & Objects

::question-wrapper
### Q1. What is the difference between a class attribute and an instance attribute?

- [ ] A class attribute is defined in the class body and shared by all instances unless overridden; an instance attribute is set per-object, usually via `self` in `__init__`
- [ ] A class attribute can only hold integers or strings; an instance attribute can hold any type
- [ ] There is no functional difference — both are stored identically in every object's `__dict__`
- [ ] A class attribute is only visible inside `__init__`, while an instance attribute is visible everywhere

<details>
<summary>Show Answer</summary>

**Answer:** A — A class attribute is defined in the class body and shared by all instances unless overridden; an instance attribute is set per-object, usually via `self` in `__init__`

**Explanation:** Class attributes live in the class's own `__dict__` and are looked up via the class if an instance doesn't have its own copy. Instance attributes live in each object's own `__dict__` and take precedence over a class attribute of the same name. They are stored differently, not identically (option C) — this distinction is exactly what causes the shared-mutable-default bug covered later in this quiz.

</details>
::

::question-wrapper
### Q2. What is the difference between `__init__` and `__new__`?

- [ ] `__new__` is responsible for *creating* and returning a new instance (called before the object exists); `__init__` *initializes* an already-created instance and returns `None`
- [ ] They are aliases for the same method; Python calls whichever one is defined
- [ ] `__init__` creates the object; `__new__` only runs for subclasses of built-in immutable types
- [ ] `__new__` runs after `__init__` to finalize attribute defaults

<details>
<summary>Show Answer</summary>

**Answer:** A — `__new__` is responsible for *creating* and returning a new instance (called before the object exists); `__init__` *initializes* an already-created instance and returns `None`

**Explanation:** `__new__` is a static method (implicitly) that allocates and returns the object — it's what actually calls `object.__new__(cls)` under the hood. Only after `__new__` returns an instance of `cls` (or a subclass) does Python call `__init__` on it to set up its initial state. Most classes never override `__new__` because `__init__` is sufficient; overriding `__new__` matters for immutable types and patterns like singletons, covered later in this quiz.

</details>
::

::question-wrapper
### Q3. Is `self` a reserved keyword in Python?

- [ ] No — it's just a strong convention; any valid identifier could be used for the first parameter of an instance method
- [ ] Yes — Python's parser specifically recognizes `self` and treats it as the instance reference
- [ ] No, but using anything other than `self` causes a `SyntaxError`
- [ ] Yes, but only inside `__init__`; other methods may use any name

<details>
<summary>Show Answer</summary>

**Answer:** A — No — it's just a strong convention; any valid identifier could be used for the first parameter of an instance method

**Explanation:** When you call `obj.method(args)`, Python translates it to `ClassName.method(obj, args)` — the first positional parameter always receives the instance, regardless of what it's named. `self` is PEP 8 convention (and nearly universal in the ecosystem), but writing `def method(this, ...)` is perfectly legal and behaves identically. It only becomes a real problem when it breaks readability or violates a linter's expectations, not because Python enforces the name.

</details>
::

::question-wrapper
### Q4. In `class Foo: def bar(self, x): ...`, what gets passed as `self` when you call `Foo().bar(5)`?

- [ ] The `Foo` instance created by `Foo()`, automatically supplied by Python's attribute-lookup/binding machinery
- [ ] The `Foo` class itself
- [ ] `None`, unless explicitly passed
- [ ] The integer `5`, with `x` left unbound

<details>
<summary>Show Answer</summary>

**Answer:** A — The `Foo` instance created by `Foo()`, automatically supplied by Python's attribute-lookup/binding machinery

**Explanation:** Accessing `instance.bar` produces a *bound method* — a callable that has already captured the instance as its first argument. `bar(5)` on that bound method is equivalent to `Foo.bar(instance, 5)`, so `self` gets the instance and `x` gets `5`. Confusing `self` with the class itself (option B) is what `cls` in classmethods is for, not `self`.

</details>
::

::question-wrapper
### Q5. What distinguishes a `@classmethod` from a regular instance method?

- [ ] It receives the class (`cls`) as its first argument instead of the instance, and can be called on the class itself without an instance
- [ ] It cannot access any attributes at all
- [ ] It automatically becomes private (name-mangled)
- [ ] It runs at import time instead of when explicitly called

<details>
<summary>Show Answer</summary>

**Answer:** A — It receives the class (`cls`) as its first argument instead of the instance, and can be called on the class itself without an instance

**Explanation:** `@classmethod` binds the first parameter to the class the method was accessed through (which, importantly, is the *actual* subclass when called via a subclass — not necessarily the class where the method was defined). This makes classmethods the idiomatic tool for alternative constructors, covered later in this quiz.

</details>
::

::question-wrapper
### Q6. What distinguishes a `@staticmethod` from both instance and class methods?

- [ ] It receives no implicit first argument at all — it behaves like a plain function that simply happens to live in the class's namespace
- [ ] It receives `self` but not `cls`
- [ ] It can only be called before any instance of the class is created
- [ ] It is automatically cached like `functools.lru_cache`

<details>
<summary>Show Answer</summary>

**Answer:** A — It receives no implicit first argument at all — it behaves like a plain function that simply happens to live in the class's namespace

**Explanation:** `@staticmethod` opts a method out of both instance-binding and class-binding; it gets exactly the arguments you pass it, nothing more. It's used purely for namespacing a helper function under a class for organizational reasons, not because it needs `self` or `cls`.

</details>
::

::question-wrapper
### Q7. In what order does Python call `__new__` and `__init__` when you write `Foo(x, y)`?

- [ ] `Foo.__new__(Foo, x, y)` is called first to create the instance; then `Foo.__init__(instance, x, y)` is called on the result
- [ ] `__init__` runs first to set defaults, then `__new__` finalizes the object
- [ ] Only one of them runs, depending on whether the class defines `__slots__`
- [ ] They run concurrently in separate threads

<details>
<summary>Show Answer</summary>

**Answer:** A — `Foo.__new__(Foo, x, y)` is called first to create the instance; then `Foo.__init__(instance, x, y)` is called on the result

**Explanation:** Object construction is a two-step protocol: `type.__call__` (invoked implicitly by `Foo(...)`) first calls `__new__` to obtain an instance, then — *only if* the returned object is an instance of `Foo` (or a subclass) — calls `__init__` on it with the same arguments. This conditional call is itself a gotcha covered later: if `__new__` returns something of an unrelated type, `__init__` is skipped entirely.

</details>
::

::question-wrapper{language="python"}
### Q8. What does this print?

::code-wrapper{language="python"}
```python
class ShoppingCart:
    items = []

    def add(self, item):
        self.items.append(item)

cart_a = ShoppingCart()
cart_b = ShoppingCart()
cart_a.add("apple")
print(cart_b.items)
```
::

- [ ] `['apple']`
- [ ] `[]`
- [ ] `AttributeError: 'ShoppingCart' object has no attribute 'items'`
- [ ] `['apple', 'apple']`

<details>
<summary>Show Answer</summary>

**Answer:** A — `['apple']`

**Explanation:** **Debug** — `items = []` is a *class* attribute, created once when the class body executes, not a fresh list per instance. `self.items.append(...)` doesn't reassign `self.items` (which would create a new instance attribute); it mutates the single list object in place, and since both `cart_a` and `cart_b` look up `items` through the same class (neither has its own instance attribute), they see the same shared list. This is one of Python's most infamous shared-mutable-state footguns — structurally identical to the mutable-default-argument trap.

</details>
::

::question-wrapper{language="python"}
### Q9. Which fix correctly gives each `ShoppingCart` instance its own independent `items` list?

::code-wrapper{language="python"}
```python
class ShoppingCart:
    def __init__(self):
        self.items = []

    def add(self, item):
        self.items.append(item)
```
::

- [ ] This version — assigning `self.items = []` inside `__init__` creates a brand-new list object as an instance attribute for every `ShoppingCart` created
- [ ] Declaring `items: list = []` as a type-annotated class attribute instead
- [ ] Using `self.items = list(ShoppingCart.items)` inside `add` before every append
- [ ] The original code was already correct; the bug in Q8 was caused by calling `.add()` twice

<details>
<summary>Show Answer</summary>

**Answer:** A — This version — assigning `self.items = []` inside `__init__` creates a brand-new list object as an instance attribute for every `ShoppingCart` created

**Explanation:** Because `__init__` runs once per instance, `self.items = []` executes fresh for every object, producing a distinct list each time and shadowing any class-level attribute of the same name. Adding a type annotation (option B) doesn't change the fundamental problem — it's still one shared list evaluated once at class-definition time. This is the general fix pattern: any mutable default (list, dict, set) should be created inside `__init__`, never as a bare class attribute meant to be per-instance state.

</details>
::

::question-wrapper{language="python"}
### Q10. What happens here?

::code-wrapper{language="python"}
```python
class Singleton:
    _instance = None

    def __new__(cls):
        return cls._instance or object.__new__(cls)

    def __init__(self):
        print("init ran")

s1 = Singleton()
Singleton._instance = s1
s2 = Singleton()
```
::

- [ ] `"init ran"` prints twice — once for `s1`, and again for `s2`, even though `s2 is s1`, because `__init__` is called whenever `__new__` returns an instance of the class, regardless of whether it's a *reused* instance
- [ ] `"init ran"` prints once, because Python detects that `s2 is s1` and skips re-initialization
- [ ] `TypeError` is raised because `__new__` doesn't accept extra arguments
- [ ] `"init ran"` never prints because `_instance` is `None` initially

<details>
<summary>Show Answer</summary>

**Answer:** A — `"init ran"` prints twice — once for `s1`, and again for `s2`, even though `s2 is s1`, because `__init__` is called whenever `__new__` returns an instance of the class, regardless of whether it's a *reused* instance

**Explanation:** **Debug** — Python's rule is purely type-based: if `__new__` returns an instance of `cls` (or a subclass), `__init__` runs on it — Python does not check whether the object is "new" versus recycled. This is exactly why naive singleton implementations that only override `__new__` still re-run `__init__` on every "construction," silently resetting state each time unless `__init__` is guarded (e.g., with an `if self._initialized:` check).

</details>
::

::question-wrapper{language="python"}
### Q11. What does this print?

::code-wrapper{language="python"}
```python
class Config:
    debug = False

c1 = Config()
c2 = Config()
c1.debug = True
print(c1.debug, c2.debug, Config.debug)
```
::

- [ ] `True False False`
- [ ] `True True True`
- [ ] `False False False`
- [ ] `AttributeError`

<details>
<summary>Show Answer</summary>

**Answer:** A — `True False False`

**Explanation:** **Debug** — `c1.debug = True` does *not* mutate the class attribute; because `bool` is immutable, this assignment creates a brand-new *instance* attribute on `c1` that shadows `Config.debug` when accessed through `c1`. `c2` and `Config` itself are unaffected, since they still resolve `debug` via the class attribute. This is the crucial contrast with Q8: reassigning a name creates an instance attribute (isolated), while mutating a shared mutable object in place (like `.append()`) affects everyone who shares that object — the same operation ("set an attribute on self") behaves completely differently depending on whether you assign or mutate.

</details>
::

::question-wrapper{language="python"}
### Q12. Is this class valid, and does it behave normally?

::code-wrapper{language="python"}
```python
class Point:
    def __init__(this, x, y):
        this.x = x
        this.y = y

    def distance_from_origin(this):
        return (this.x ** 2 + this.y ** 2) ** 0.5

p = Point(3, 4)
print(p.distance_from_origin())
```
::

- [ ] Yes — it prints `5.0`; `this` works exactly like `self` would, since the first parameter name is not special to Python
- [ ] `SyntaxError`, since instance methods must name their first parameter `self`
- [ ] It runs but `this.x` and `this.y` are never actually attached to `p`
- [ ] `TypeError: __init__() takes 3 positional arguments but 4 were given`

<details>
<summary>Show Answer</summary>

**Answer:** A — Yes — it prints `5.0`; `this` works exactly like `self` would, since the first parameter name is not special to Python

**Explanation:** As established in Q3, Python's method-binding mechanism only cares about parameter *position*, not name — whichever name occupies the first slot receives the bound instance. The code is fully functional and computes `sqrt(3² + 4²) = 5.0`; it's simply unconventional and would likely fail code review for violating the near-universal `self` naming convention, not because of anything the interpreter enforces.

</details>
::

::question-wrapper{language="python"}
### Q13. Calling a `@staticmethod` through an instance versus through the class — what's the difference in behavior?

::code-wrapper{language="python"}
```python
class MathUtils:
    @staticmethod
    def add(a, b):
        return a + b

m = MathUtils()
print(MathUtils.add(2, 3), m.add(2, 3))
```
::

- [ ] No difference — both print `5`; a static method never receives an implicit instance argument no matter how it's accessed
- [ ] `m.add(2, 3)` raises `TypeError` because it implicitly passes `m` as an extra argument
- [ ] `MathUtils.add(2, 3)` fails because static methods require an instance to be called
- [ ] The class-level call returns an unbound method object instead of `5`

<details>
<summary>Show Answer</summary>

**Answer:** A — No difference — both print `5`; a static method never receives an implicit instance argument no matter how it's accessed

**Explanation:** `@staticmethod` explicitly opts out of the descriptor-based instance-binding that normal methods get, so accessing it via `m.add` does *not* produce a bound method the way `m.instance_method` would — it returns the plain underlying function either way. This is exactly why static methods are safe to call from either the class or an instance with identical results.

</details>
::

::question-wrapper{language="python"}
### Q14. Alternative constructors via `@classmethod` in a subclass — what prints?

::code-wrapper{language="python"}
```python
class Animal:
    def __init__(self, name):
        self.name = name

    @classmethod
    def from_upper(cls, name):
        return cls(name.upper())

class Dog(Animal):
    pass

d = Dog.from_upper("rex")
print(type(d).__name__, d.name)
```
::

- [ ] `Dog REX`
- [ ] `Animal REX`
- [ ] `Dog rex`
- [ ] `TypeError: from_upper() is not inherited by subclasses`

<details>
<summary>Show Answer</summary>

**Answer:** A — `Dog REX`

**Explanation:** Inside a classmethod, `cls` is bound to whichever class the method was actually accessed through — here, `Dog` — not the class where the method was defined (`Animal`). So `cls(name.upper())` calls `Dog("REX")`, correctly producing a `Dog` instance. This polymorphic behavior is precisely why `@classmethod` (not `@staticmethod`) is the right tool for alternative constructors meant to be inheritance-friendly.

</details>
::

::question-wrapper
### Q15. For a class that needs a mutable default (like a list or dict) as part of an instance's state, what is the best-practice approach?

- [ ] Initialize it inside `__init__` as `self.attr = []` (or use `dataclasses.field(default_factory=list)` if using `@dataclass`), never as a bare mutable class attribute
- [ ] Declare it as a class attribute for efficiency, since it avoids re-allocating a list per instance
- [ ] Use a mutable default argument in `__init__`, e.g. `def __init__(self, items=[]):`
- [ ] Store it as a class attribute and document that callers must not mutate it

<details>
<summary>Show Answer</summary>

**Answer:** A — Initialize it inside `__init__` as `self.attr = []` (or use `dataclasses.field(default_factory=list)` if using `@dataclass`), never as a bare mutable class attribute

**Explanation:** **Idiom** — This is the direct fix for the Q8/Q9 gotcha, generalized: any per-instance mutable state belongs in `__init__` (or a `default_factory` for dataclasses), so each instance gets its own object. A mutable default *argument* (option C) is a related but distinct footgun — default argument values are evaluated once at function-definition time and shared across all calls that don't override them, causing the exact same kind of cross-instance leakage.

</details>
::

::question-wrapper
### Q16. When designing alternative ways to construct an object (e.g., `Config.from_file(path)`, `Config.from_env()`), what's the idiomatic choice?

- [ ] `@classmethod`, so `cls(...)` is used internally, keeping the constructors correct for subclasses too
- [ ] `@staticmethod`, since these are just utility functions that don't need class state
- [ ] A plain module-level function that returns a hardcoded `Config(...)` call
- [ ] Overloading `__init__` with many optional parameters instead of separate constructors

<details>
<summary>Show Answer</summary>

**Answer:** A — `@classmethod`, so `cls(...)` is used internally, keeping the constructors correct for subclasses too

**Explanation:** **Idiom** — As shown in Q14, a `@classmethod` that calls `cls(...)` automatically does the right thing for subclasses, while a `@staticmethod` (option B) would have to hardcode the concrete class name, silently breaking for any subclass that calls the "constructor." This is the standard pattern behind `dict.fromkeys`, `datetime.fromtimestamp`, and similar APIs in the standard library.

</details>
::

::question-wrapper
### Q17. When is it actually necessary to override `__new__` instead of just using `__init__`?

- [ ] When you need to control instance *creation* itself — e.g., returning a cached/singleton instance, or subclassing an immutable built-in type like `str` or `tuple` where state must be set during creation
- [ ] Whenever a class defines more than one `__init__` parameter
- [ ] Every time a class is used with inheritance
- [ ] Never — `__new__` is a legacy hook with no valid modern use case

<details>
<summary>Show Answer</summary>

**Answer:** A — When you need to control instance *creation* itself — e.g., returning a cached/singleton instance, or subclassing an immutable built-in type like `str` or `tuple` where state must be set during creation

**Explanation:** **Idiom** — `__init__` can only *mutate* an already-created instance, which doesn't work for immutable types (you can't "mutate" a `str` after creation) or for patterns where you might want to return an existing object instead of a new one (singletons, object pools, memoized value types). For ordinary mutable classes, overriding `__new__` is unnecessary complexity — `__init__` alone is the idiomatic choice, which is why it's rare to see `__new__` overrides in typical application code.

</details>
::

::question-wrapper
### Q18. A team debates whether `MAX_RETRIES = 3` should be a class attribute or set in `__init__`. Given it's an immutable, shared constant that instances never need to override individually, what's the best practice?

- [ ] Keep it as a class attribute — it's immutable, so there's no shared-mutation risk, and instances can still override it individually if ever needed via `self.MAX_RETRIES = ...`
- [ ] Always move every attribute into `__init__`, regardless of mutability, to avoid any confusion
- [ ] Store it in a global module-level variable instead of attaching it to the class at all
- [ ] Make it a `@property` that returns a hardcoded value

<details>
<summary>Show Answer</summary>

**Answer:** A — Keep it as a class attribute — it's immutable, so there's no shared-mutation risk, and instances can still override it individually if ever needed via `self.MAX_RETRIES = ...`

**Explanation:** **Idiom** — The Q8 gotcha specifically concerns *mutable* class attributes (lists, dicts, sets) being mutated in place and shared unexpectedly. An immutable constant like an `int` has no such risk: any attempt to "change" it on an instance creates a new instance attribute (as in Q11) rather than corrupting shared state, so class-level placement is both idiomatic and efficient (one shared object instead of a per-instance copy). The rule isn't "never use class attributes" — it's "don't use *mutable* class attributes for per-instance state."

</details>
::

::question-wrapper
### Q19. Which situation makes a plain module-level function more appropriate than a `@staticmethod`?

- [ ] When the function has no meaningful connection to the class's purpose or namespace, and grouping it under the class only adds indirection without organizational benefit
- [ ] Whenever the function doesn't use `self`
- [ ] Whenever the function is longer than a few lines
- [ ] Static methods are always better than module functions because they're faster to call

<details>
<summary>Show Answer</summary>

**Answer:** A — When the function has no meaningful connection to the class's purpose or namespace, and grouping it under the class only adds indirection without organizational benefit

**Explanation:** **Idiom** — `@staticmethod` exists purely for *namespacing* — grouping a helper under a class because it's conceptually related (e.g., `Vector.from_polar`-style helpers, or validation helpers tightly coupled to the class's data). If a function is genuinely general-purpose and unrelated to the class, forcing it into the class as a static method just adds an extra `ClassName.` prefix everywhere it's used, with no benefit — a plain module function is more discoverable and idiomatic in that case. "Doesn't use `self`" (option B) is necessary but not sufficient justification on its own.

</details>
::

::question-wrapper
### Q20. Reviewing a pull request, you see `type(obj) == Foo` used to check an object's type instead of `isinstance(obj, Foo)`. Why is this flagged as a best-practice issue?

- [ ] `isinstance` respects inheritance (so subclass instances are correctly recognized) and works with virtual base classes/ABCs, while `type(obj) == Foo` fails for any subclass instance even though it's substitutable for `Foo`
- [ ] `type(obj) == Foo` is slower in every case and should never be used for performance reasons
- [ ] They are functionally identical; the difference is purely stylistic
- [ ] `isinstance` cannot check against built-in types like `int` or `str`

<details>
<summary>Show Answer</summary>

**Answer:** A — `isinstance` respects inheritance (so subclass instances are correctly recognized) and works with virtual base classes/ABCs, while `type(obj) == Foo` fails for any subclass instance even though it's substitutable for `Foo`

**Explanation:** **Idiom** — Given `class Dog(Animal): ...`, `isinstance(Dog(), Animal)` is `True` (correctly honoring polymorphism), but `type(Dog()) == Animal` is `False`, since `type()` returns the *exact* class, not considering the MRO. Code that relies on exact-type checks silently breaks the moment someone introduces a reasonable subclass, which is precisely the kind of duck-typing/polymorphism violation `isinstance` is designed to avoid. This ties directly into duck typing and explicit type checks covered in the next quiz on inheritance.

</details>
::
