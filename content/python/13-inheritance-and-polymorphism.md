# 13 — Inheritance & Polymorphism

::question-wrapper
### Q1. What is the Method Resolution Order (MRO)?

- [ ] The specific, linear order in which Python searches base classes for a method or attribute when it's not found on the instance or its own class
- [ ] The order in which a class's methods are defined in the source file
- [ ] The order in which methods execute when an instance is created
- [ ] A runtime cache of the most recently called methods, for performance

<details>
<summary>Show Answer</summary>

**Answer:** A — The specific, linear order in which Python searches base classes for a method or attribute when it's not found on the instance or its own class

**Explanation:** Every class has a computed MRO (viewable via `ClassName.__mro__` or `ClassName.mro()`) that linearizes the entire inheritance graph into a single, deterministic sequence. Attribute and method lookups walk this sequence in order and stop at the first match. It has nothing to do with source-code definition order or a runtime cache — it's a static property computed when the class is created.

</details>
::

::question-wrapper
### Q2. In single inheritance, what does `super().__init__(*args)` inside a subclass's `__init__` do?

- [ ] Calls the immediate parent class's `__init__`, passing along the given arguments, using the instance's actual MRO
- [ ] Calls `object.__init__` directly, skipping any intermediate parent classes
- [ ] Re-runs the subclass's own `__init__` recursively
- [ ] Creates a brand-new parent-class instance separate from `self`

<details>
<summary>Show Answer</summary>

**Answer:** A — Calls the immediate parent class's `__init__`, passing along the given arguments, using the instance's actual MRO

**Explanation:** `super()` (zero-argument form, Python 3+) returns a proxy object bound to the current class and instance, and delegates attribute lookup to the *next* class in `self`'s MRO after the current one. In simple single inheritance, that next class is just the immediate parent — but as covered later in this quiz, in multiple inheritance "next in the MRO" is not always the same as "the literal parent class in the `class` statement."

</details>
::

::question-wrapper
### Q3. What is "diamond inheritance"?

- [ ] A class hierarchy where two classes `B` and `C` both inherit from a common base `A`, and a class `D` inherits from both `B` and `C`, forming a diamond shape
- [ ] A class that inherits from four or more base classes
- [ ] A circular inheritance chain where `A` inherits from `B` and `B` inherits from `A`
- [ ] Inheriting from a built-in type like `dict` or `list`

<details>
<summary>Show Answer</summary>

**Answer:** A — A class hierarchy where two classes `B` and `C` both inherit from a common base `A`, and a class `D` inherits from both `B` and `C`, forming a diamond shape

**Explanation:** Diagrammed, `A` sits at the top, `B` and `C` branch off it, and `D` sits at the bottom pointing to both — a diamond. The classic question this raises is: when `D` calls a method defined on `A` (and possibly overridden in `B` and/or `C`), which version runs, and does `A`'s code run once or twice? Python's C3 linearization (covered next) exists specifically to answer this unambiguously.

</details>
::

::question-wrapper
### Q4. What guarantee does Python's C3 linearization algorithm provide when computing a class's MRO?

- [ ] Every class appears exactly once in the MRO, a subclass always precedes its own base classes, and the relative order of base classes as listed in the `class` statement is preserved
- [ ] It guarantees the MRO always matches simple depth-first, left-to-right traversal of the inheritance tree, as in old-style classes
- [ ] It guarantees every base class's `__init__` runs automatically, even without `super()` calls
- [ ] It only applies to classes that inherit from exactly two base classes

<details>
<summary>Show Answer</summary>

**Answer:** A — Every class appears exactly once in the MRO, a subclass always precedes its own base classes, and the relative order of base classes as listed in the `class` statement is preserved

**Explanation:** **Debug** — Python 2's old-style classes used naive depth-first-left-to-right (DFLR) search, which could visit a common ancestor multiple times or in an inconsistent order for diamond hierarchies. C3 linearization (used by all classes in Python 3, since they all implicitly inherit from `object`) fixes this by merging each base's own MRO plus the base list itself, guaranteeing monotonicity and local precedence. If no consistent order can be computed, Python raises `TypeError` at class-creation time rather than silently picking an ambiguous order (covered in Q9).

</details>
::

::question-wrapper
### Q5. What is "duck typing"?

- [ ] Relying on an object's behavior (which methods/attributes it supports) rather than its actual type or class hierarchy to decide if it's usable in a given context
- [ ] A typing style where every variable must have an explicit type hint
- [ ] Using `isinstance()` checks exclusively instead of `type()` comparisons
- [ ] A Python 2-only feature removed in Python 3

<details>
<summary>Show Answer</summary>

**Answer:** A — Relying on an object's behavior (which methods/attributes it supports) rather than its actual type or class hierarchy to decide if it's usable in a given context

**Explanation:** "If it walks like a duck and quacks like a duck, it's a duck" — Python code that does `obj.read()` without checking `isinstance(obj, SomeFileType)` first is duck typing: any object with a compatible `.read()` method works, regardless of its actual class or inheritance chain. This is central to Python's idiomatic style and is contrasted with rigid, explicit `isinstance` gatekeeping later in this quiz.

</details>
::

::question-wrapper{language="python"}
### Q6. What happens if a subclass overrides `__init__` but never calls `super().__init__()`?

::code-wrapper{language="python"}
```python
class Vehicle:
    def __init__(self, wheels):
        self.wheels = wheels

class Car(Vehicle):
    def __init__(self, brand):
        self.brand = brand  # forgot super().__init__(wheels)

c = Car("Toyota")
print(c.wheels)
```
::

- [ ] `AttributeError: 'Car' object has no attribute 'wheels'` — the parent's `__init__` never runs, so `self.wheels` is never set
- [ ] `4` — Python automatically infers a default value for `wheels`
- [ ] `None` — uninitialized attributes default to `None`
- [ ] `TypeError` is raised immediately when `Car("Toyota")` is called

<details>
<summary>Show Answer</summary>

**Answer:** A — `AttributeError: 'Car' object has no attribute 'wheels'` — the parent's `__init__` never runs, so `self.wheels` is never set

**Explanation:** **Debug** — Overriding `__init__` completely replaces the parent's version unless the subclass explicitly calls it via `super().__init__(...)`. There is no automatic chaining — Python does not run every `__init__` up the MRO unless the code says so. This is one of the most common real-world inheritance bugs: a subclass "loses" attributes the parent was responsible for setting up, and the failure only surfaces later when that attribute is accessed, far from the actual mistake.

</details>
::

::question-wrapper
### Q7. How can you inspect a class's actual MRO at runtime?

- [ ] `ClassName.__mro__` (a tuple) or `ClassName.mro()` (a list)
- [ ] `ClassName.__bases__` gives the full MRO, including indirect ancestors
- [ ] `dir(ClassName)` returns the MRO in order
- [ ] MRO cannot be inspected; it's only used internally by the interpreter

<details>
<summary>Show Answer</summary>

**Answer:** A — `ClassName.__mro__` (a tuple) or `ClassName.mro()` (a list)

**Explanation:** `__mro__` is the linearized order computed by C3 at class-creation time. `__bases__` (option B) is a common point of confusion — it only lists the *direct* parents named in the `class` statement, not the full linearized ancestor chain, so it's insufficient for understanding how `super()` will actually resolve calls in a multi-level or multiple-inheritance hierarchy.

</details>
::

::question-wrapper{language="python"}
### Q8. Given cooperative multiple inheritance, what does this print?

::code-wrapper{language="python"}
```python
class A:
    def greet(self):
        print("A")

class B(A):
    def greet(self):
        print("B")
        super().greet()

class C(A):
    def greet(self):
        print("C")
        super().greet()

class D(B, C):
    def greet(self):
        print("D")
        super().greet()

D().greet()
```
::

- [ ] `D`, `B`, `C`, `A` — each class's `greet` runs exactly once, in MRO order
- [ ] `D`, `B`, `A`, `C`, `A` — `A` runs twice, once via each branch of the diamond
- [ ] `D`, `B`, `A` only — `C` is never reached
- [ ] `D`, `C`, `B`, `A` — multiple inheritance always resolves right-to-left

<details>
<summary>Show Answer</summary>

**Answer:** A — `D`, `B`, `C`, `A` — each class's `greet` runs exactly once, in MRO order

**Explanation:** **Debug** — `D`'s MRO is `[D, B, C, A, object]` (computed by C3, preserving `B` before `C` since that's their order in `class D(B, C)`). Each `super().greet()` call doesn't jump straight to that class's own literal parent — it advances to the *next class in the shared MRO*, so `B`'s `super().greet()` calls `C`'s (not `A`'s directly), and only `C`'s `super().greet()` finally reaches `A`. This is exactly why naive assumptions like "each branch of the diamond calls `A` independently" (option B, `A` running twice) are wrong — C3 linearization guarantees `A` is visited exactly once.

</details>
::

::question-wrapper{language="python"}
### Q9. What happens when you try to define this class?

::code-wrapper{language="python"}
```python
class X:
    pass

class Y(X):
    pass

class Z(X, Y):
    pass
```
::

- [ ] `TypeError: Cannot create a consistent method resolution order (MRO) for bases X, Y`
- [ ] It works fine; `Z`'s MRO is `[Z, X, Y, object]`
- [ ] It works fine; `Z`'s MRO is `[Z, Y, X, object]`
- [ ] `Z` silently ignores `X` since `Y` already inherits from it

<details>
<summary>Show Answer</summary>

**Answer:** A — `TypeError: Cannot create a consistent method resolution order (MRO) for bases X, Y`

**Explanation:** **Debug** — `class Z(X, Y)` demands that `X` precede `Y` (as listed), but `Y` already inherits from `X`, which means any valid MRO must place `Y` before `X` (subclasses must precede their own bases — a rule from Q4). These two requirements directly contradict each other, so C3 linearization has no valid solution, and Python raises `TypeError` at class-definition time rather than guessing. The fix is to list bases in an order consistent with the existing hierarchy: `class Z(Y, X)` would work, since `Y` already implies `X` comes after it.

</details>
::

::question-wrapper
### Q10. In Python 3, is there any behavioral difference between `super()` and `super(CurrentClass, self)` when called inside a normal instance method of `CurrentClass`?

- [ ] No — Python 3's zero-argument `super()` is compiler-assisted sugar that resolves to exactly `super(CurrentClass, self)` in that context
- [ ] Yes — the zero-argument form always uses `object` as the starting point, skipping intermediate classes
- [ ] Yes — the zero-argument form only works in `__init__`, not other methods
- [ ] Yes — `super()` without arguments raises `TypeError` in Python 3

<details>
<summary>Show Answer</summary>

**Answer:** A — No — Python 3's zero-argument `super()` is compiler-assisted sugar that resolves to exactly `super(CurrentClass, self)` in that context

**Explanation:** The compiler injects a hidden `__class__` cell reference so that bare `super()` inside a method knows both the class it was defined in and the instance (`self`) it's being called on, letting it reconstruct the explicit two-argument form automatically. The explicit `super(CurrentClass, self)` form (Python 2 style) still works in Python 3 and is occasionally still needed — e.g., inside a `@staticmethod` or a nested function where the implicit `__class__` cell isn't available — but for ordinary methods, the two are equivalent.

</details>
::

::question-wrapper{language="python"}
### Q11. A mixin is meant to add caching to any class in a cooperative multiple-inheritance hierarchy. What's wrong with this `__init__`?

::code-wrapper{language="python"}
```python
class CacheMixin:
    def __init__(self, *args, **kwargs):
        self.cache = {}
        super().__init__()  # note: no args/kwargs forwarded

class Repository(CacheMixin, Base):
    def __init__(self, db_url):
        super().__init__(db_url)
```
::

- [ ] `CacheMixin.__init__` drops `*args, **kwargs` when calling `super().__init__()`, so the next class in the MRO (`Base`) never receives `db_url`, breaking `Base`'s own initialization
- [ ] Mixins can never define `__init__`; only the final concrete class can
- [ ] `super().__init__()` inside a mixin always raises `TypeError` since mixins have no base class
- [ ] Nothing is wrong; `Base.__init__` will still receive `db_url` automatically

<details>
<summary>Show Answer</summary>

**Answer:** A — `CacheMixin.__init__` drops `*args, **kwargs` when calling `super().__init__()`, so the next class in the MRO (`Base`) never receives `db_url`, breaking `Base`'s own initialization

**Explanation:** **Debug** — In cooperative multiple inheritance, every class in the chain must forward whatever arguments it doesn't consume itself to `super().__init__(*args, **kwargs)`, so the call correctly propagates down the entire MRO to whichever class ultimately needs them. Here, `CacheMixin` accepts `*args, **kwargs` but then calls `super().__init__()` with nothing, silently swallowing `db_url` before it ever reaches `Base`. The fix is `super().__init__(*args, **kwargs)` in the mixin, ensuring the cooperative chain stays intact.

</details>
::

::question-wrapper{language="python"}
### Q12. What does this print?

::code-wrapper{language="python"}
```python
class Base:
    def __init__(self):
        self.value = self.compute()

    def compute(self):
        return 1

class Derived(Base):
    def compute(self):
        return 2

print(Derived().value)
```
::

- [ ] `2`
- [ ] `1`
- [ ] `AttributeError: 'Derived' object has no attribute 'compute'`
- [ ] `None`

<details>
<summary>Show Answer</summary>

**Answer:** A — `2`

**Explanation:** **Debug** — Even though `compute()` is *called* from within `Base.__init__`, method lookup is always based on the *actual runtime type* of `self` (`Derived`), not on which class's code is currently executing. This is polymorphism working correctly, but it's a common surprise for developers coming from languages with static dispatch — it also means calling overridable methods from `__init__` is risky if the override depends on subclass attributes that haven't been set up yet, since `__init__` for `Derived` hasn't necessarily finished running its own setup by the time `Base.__init__` calls `compute()`.

</details>
::

::question-wrapper{language="python"}
### Q13. A function accepts any "file-like" object and calls `.read()` on it. Which approach is duck typing, and which is its rigid alternative?

::code-wrapper{language="python"}
```python
# Version A
def load(source):
    return source.read()

# Version B
def load(source):
    if not isinstance(source, io.IOBase):
        raise TypeError("source must be a file")
    return source.read()
```
::

- [ ] Version A is duck typing — it works with any object that implements `.read()`, including `io.StringIO`, sockets wrapped in `makefile()`, or test doubles; Version B rejects perfectly valid file-like objects that don't literally subclass `io.IOBase`
- [ ] Version B is duck typing, since it explicitly checks the file's "shape" via `isinstance`
- [ ] Both versions are equally flexible; `isinstance` checks against `io.IOBase` accept any object with a `.read()` method
- [ ] Version A is unsafe and should never be used in production code

<details>
<summary>Show Answer</summary>

**Answer:** A — Version A is duck typing — it works with any object that implements `.read()`, including `io.StringIO`, sockets wrapped in `makefile()`, or test doubles; Version B rejects perfectly valid file-like objects that don't literally subclass `io.IOBase`

**Explanation:** Duck typing (Version A) trusts that if `source` has a working `.read()`, it's usable — this is precisely what makes Python code composable with third-party and test objects that were never designed to subclass anything in particular. Version B's `isinstance` check is overly rigid: plenty of legitimate file-like objects (certain mocks, custom wrappers, some third-party libraries) don't inherit from `io.IOBase` even though `.read()` works perfectly, so the check produces false-negative `TypeError`s for valid input — a real production bug pattern, not just a style nitpick.

</details>
::

::question-wrapper{language="python"}
### Q14. What does this print, and why?

::code-wrapper{language="python"}
```python
class Base:
    def __init__(self, name):
        self.name = name
        print("Base init")

class Mixin:
    def __init__(self, *a, **kw):
        print("Mixin init")
        super().__init__(*a, **kw)

class Combined(Mixin, Base):
    pass

Combined("x")
```
::

- [ ] `Mixin init` then `Base init` — `Combined`'s MRO is `[Combined, Mixin, Base, object]`, so `Mixin.__init__` runs first and its `super().__init__` correctly forwards to `Base`
- [ ] `Base init` then `Mixin init` — base classes always initialize before mixins
- [ ] Only `Mixin init` prints; `Base.__init__` is never reached
- [ ] `TypeError`, because `Combined` doesn't define its own `__init__`

<details>
<summary>Show Answer</summary>

**Answer:** A — `Mixin init` then `Base init` — `Combined`'s MRO is `[Combined, Mixin, Base, object]`, so `Mixin.__init__` runs first and its `super().__init__` correctly forwards to `Base`

**Explanation:** Since `Combined` defines no `__init__` of its own, calling `Combined("x")` resolves `__init__` via the MRO, finding `Mixin.__init__` first (because `Mixin` is listed before `Base` in `class Combined(Mixin, Base)`). Because `Mixin.__init__` properly forwards `*a, **kw` to `super().__init__(*a, **kw)` (the fix from Q11), the chain continues correctly into `Base.__init__`, which prints and sets `self.name`. This demonstrates why mixins are conventionally listed *before* the "real" base class — covered as a best practice later in this quiz.

</details>
::

::question-wrapper
### Q15. When should you prefer duck typing / `isinstance` checks against an Abstract Base Class (ABC) or `collections.abc` protocol over an explicit concrete-type check?

- [ ] When you only care that the object supports the required behavior (e.g., iteration, `.read()`, comparison), since this maximizes compatibility with any conforming object, including ones from third-party code or tests
- [ ] Always avoid `isinstance` entirely; type checks are never appropriate in idiomatic Python
- [ ] Only when performance is not a concern, since duck typing is always slower
- [ ] Only in Python 2 code; Python 3 favors strict type checks exclusively

<details>
<summary>Show Answer</summary>

**Answer:** A — When you only care that the object supports the required behavior (e.g., iteration, `.read()`, comparison), since this maximizes compatibility with any conforming object, including ones from third-party code or tests

**Explanation:** **Idiom** — `isinstance(x, collections.abc.Iterable)` or simply trying `iter(x)` and catching `TypeError` (EAFP style) accepts any object that behaves correctly, regardless of its concrete class — this is far more Pythonic than requiring a specific concrete base class. `isinstance` isn't inherently un-Pythonic (option B overstates it); checking against a *behavioral* ABC/protocol is fine, while checking against one specific *concrete* implementation class (as in Q13's Version B) is the anti-pattern.

</details>
::

::question-wrapper
### Q16. What is the best-practice rule about calling `super().__init__()` when overriding `__init__` in a subclass?

- [ ] Call it (typically as the first statement) unless you have a deliberate, documented reason to fully replace the parent's initialization behavior
- [ ] Never call it — each class should be responsible for setting up only its own attributes independently
- [ ] Only call it if the parent class defines more than one attribute
- [ ] Call it only in multiple inheritance, never in single inheritance

<details>
<summary>Show Answer</summary>

**Answer:** A — Call it (typically as the first statement) unless you have a deliberate, documented reason to fully replace the parent's initialization behavior

**Explanation:** **Idiom** — This directly prevents the Q6 bug (missing attributes) and the Q11 bug (broken cooperative chains). Skipping `super().__init__()` should be a conscious, rare decision — e.g., when a subclass genuinely needs to bypass the parent's setup entirely — not the default, since forgetting it silently produces incompletely-initialized objects that only fail later when a missing attribute is accessed.

</details>
::

::question-wrapper
### Q17. For mixins designed to be combined with other classes via multiple inheritance, what's the best-practice pattern for their `__init__` methods?

- [ ] Accept `*args, **kwargs`, do the mixin's own setup, then call `super().__init__(*args, **kwargs)` to forward everything else down the MRO chain
- [ ] Never define `__init__` in a mixin at all
- [ ] Call `Base.__init__(self)` directly by name instead of using `super()`
- [ ] Require every consumer of the mixin to manually call the mixin's `__init__` separately from the class's own `__init__`

<details>
<summary>Show Answer</summary>

**Answer:** A — Accept `*args, **kwargs`, do the mixin's own setup, then call `super().__init__(*args, **kwargs)` to forward everything else down the MRO chain

**Explanation:** **Idiom** — This is the cooperative multiple inheritance pattern demonstrated correctly in Q14: every participant in the chain must both do its own work *and* pass along whatever it doesn't personally need, using `super()` rather than a hardcoded class name (option C), since a hardcoded name breaks the MRO-based dispatch entirely and can call the wrong class or the same class twice in complex hierarchies.

</details>
::

::question-wrapper
### Q18. A design needs a `Car` to have `Engine`-like behavior and `GPS`-like behavior, but these are unrelated capabilities with no natural "is-a" relationship to `Car`. What's the best-practice design choice?

- [ ] Favor composition — give `Car` an `self.engine = Engine()` and `self.gps = GPS()` attribute — over multiple inheritance, since `Car` "has-a" engine and GPS, it isn't "an" `Engine` or "a" `GPS`
- [ ] Always use multiple inheritance (`class Car(Engine, GPS)`) since it's more concise
- [ ] Duplicate the engine and GPS logic directly inside `Car` to avoid any inheritance complexity
- [ ] Use multiple inheritance but avoid calling `super()` anywhere to keep things simple

<details>
<summary>Show Answer</summary>

**Answer:** A — Favor composition — give `Car` an `self.engine = Engine()` and `self.gps = GPS()` attribute — over multiple inheritance, since `Car` "has-a" engine and GPS, it isn't "an" `Engine` or "a" `GPS`

**Explanation:** **Idiom** — "Favor composition over inheritance" is a core OOP design principle: multiple inheritance is powerful but adds real MRO complexity (as this whole quiz demonstrates) for relationships that aren't genuinely "is-a." Reserving multiple inheritance for true mixins (small, focused, cooperative classes explicitly designed to be combined) and using composition for unrelated capabilities keeps hierarchies shallow, predictable, and far easier to reason about and test.

</details>
::

::question-wrapper
### Q19. When multiple concrete types should be accepted by one `isinstance` check, what's the idiomatic way to write it?

- [ ] `isinstance(x, (int, float))` — pass a tuple of types as the second argument
- [ ] `isinstance(x, int) or isinstance(x, float)` — always spelled out with `or`, since tuples aren't supported
- [ ] `type(x) in (int, float)` — using `type()` is preferred since it's more explicit
- [ ] `isinstance(x, int, float)` — pass each type as a separate positional argument

<details>
<summary>Show Answer</summary>

**Answer:** A — `isinstance(x, (int, float))` — pass a tuple of types as the second argument

**Explanation:** **Idiom** — `isinstance` natively accepts a tuple of types (or classes) as its second argument and returns `True` if the object matches any of them, which is both more concise and, unlike `type(x) in (int, float)`, still correctly honors subclasses (e.g., a `bool`, which subclasses `int`, matches `isinstance(x, int)` but `type(x) in (int,)` would reject it since `type(True) is bool`, not `int`). `isinstance(x, int, float)` (option D) raises `TypeError` — that's not valid syntax for multiple types.

</details>
::

::question-wrapper
### Q20. When writing `class Combined(Mixin, Base):`, why do style guides recommend listing mixins *before* the primary base class?

- [ ] Because Python's left-to-right MRO ordering means earlier-listed classes take precedence for overridden methods, and mixins are meant to "layer on top of" the base's behavior, intercepting calls via `super()` before they reach the base
- [ ] Because Python requires mixins to be listed first or it raises a `TypeError`
- [ ] The order of base classes has no effect on behavior, only readability
- [ ] Because mixins must always be defined before the base class in the source file, and `class` statement order must match definition order

<details>
<summary>Show Answer</summary>

**Answer:** A — Because Python's left-to-right MRO ordering means earlier-listed classes take precedence for overridden methods, and mixins are meant to "layer on top of" the base's behavior, intercepting calls via `super()` before they reach the base

**Explanation:** **Idiom** — As demonstrated in Q14, `class Combined(Mixin, Base)` produces the MRO `[Combined, Mixin, Base, object]`, so `Mixin`'s methods (including `__init__`) run *before* `Base`'s and can add behavior around calls that eventually reach `Base` via `super()`. Reversing the order (`Base, Mixin`) would mean `Base`'s methods take precedence instead, likely defeating the mixin's entire purpose of augmenting or intercepting behavior. This ordering convention is why nearly every mixin-based library (e.g., Django class-based views) documents "mixins go first, left of the base class."

</details>
::
