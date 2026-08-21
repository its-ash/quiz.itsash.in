# 03 — Operators & Expressions

::question-wrapper{language="python"}
### Q1. What does the following print?

::code-wrapper{language="python"}
```python
print(7 / 2)
print(7 // 2)
```
::

- [ ] `3.5` then `3`
- [ ] `3` then `3.5`
- [ ] `3.5` then `3.5`
- [ ] `4` then `3`

<details>
<summary>Show Answer</summary>

**Answer:** A — `3.5` then `3`

**Explanation:** `/` is true division and always returns a `float` in Python 3, regardless of operand types. `//` is floor division; with two `int` operands it returns the floored quotient as an `int` (`3`, not `3.5`). B swaps the two operators' behavior. C wrongly assumes `//` also returns a float for int operands. D rounds `7 / 2` incorrectly instead of truncating toward the floor.

</details>
::

::question-wrapper{language="python"}
### Q2. What does the following print?

::code-wrapper{language="python"}
```python
x = 5
print(1 < x < 10)
```
::

- [ ] `False`
- [ ] `True`
- [ ] Raises `SyntaxError`
- [ ] `10`

<details>
<summary>Show Answer</summary>

**Answer:** B — `True`

**Explanation:** Python evaluates the chained comparison `1 < x < 10` as `(1 < x) and (x < 10)`. Both sub-comparisons are `True` here, so the result is `True`. C is wrong because chained comparisons are ordinary, valid Python syntax. D is wrong because comparison operators always yield a `bool`, never one of the compared values.

</details>
::

::question-wrapper{language="python"}
### Q3. What does the following print?

::code-wrapper{language="python"}
```python
print(5 < 3 < 10)
```
::

- [ ] `True`
- [ ] Raises `SyntaxError`
- [ ] `False`
- [ ] `3`

<details>
<summary>Show Answer</summary>

**Answer:** C — `False`

**Explanation:** `5 < 3 < 10` means `(5 < 3) and (3 < 10)` = `False and True` = `False`. A programmer used to C-family languages might expect left-to-right evaluation like `(5 < 3) < 10`, which — because `bool` is a subtype of `int` in Python — would actually compute `False < 10` → `0 < 10` → `True`. That is genuinely *not* how Python parses a comparison chain, which is why the real answer flips to `False`. B is wrong: chaining is valid syntax. D is nonsensical since comparisons never return an operand's value.

</details>
::

::question-wrapper{language="python"}
### Q4. What does the following print?

::code-wrapper{language="python"}
```python
def f():
    print("called")
    return 5

print(1 < f() < 10)
```
::

- [ ] `called` is printed twice, then `True`
- [ ] `called` is printed twice, then `False`
- [ ] `called` is never printed, then `True`
- [ ] `called` is printed once, then `True`

<details>
<summary>Show Answer</summary>

**Answer:** D — `called` is printed once, then `True`

**Explanation:** In a chained comparison, each middle sub-expression is evaluated only once and its value is reused for both surrounding comparisons. `1 < f() < 10` is not sugar for `1 < f() and f() < 10` (which would call `f()` twice, as A/B assume) — it calls `f()` a single time, gets `5`, and checks `1 < 5` and `5 < 10`, both `True`. C is wrong because the middle value must still be computed before any comparison can happen.

</details>
::

::question-wrapper{language="python"}
### Q5. What does the following print?

::code-wrapper{language="python"}
```python
a = 100
b = 100
print(a is b)
```
::

- [ ] `True`
- [ ] `False`
- [ ] Raises `NameError`
- [ ] `None`

<details>
<summary>Show Answer</summary>

**Answer:** A — `True`

**Explanation:** CPython pre-allocates and caches small integers from -5 to 256 inclusive as singleton objects. Every reference to the literal `100` resolves to the same cached object, so identity (`is`) happens to agree with equality (`==`) here. B is what you'd expect for arbitrary objects in general, but it doesn't hold inside this cached range — which is exactly the trap: it teaches beginners that `is` "works" for ints when really it's a caching accident.

</details>
::

::question-wrapper{language="python"}
### Q6. What does the following print?

::code-wrapper{language="python"}
```python
a = 1000
b = 1000
print(a is b)
```
::

- [ ] Always `False`, because 1000 is outside the small-int cache
- [ ] Unspecified — CPython may print `True` or `False` depending on compile-time constant folding and context, so don't rely on it
- [ ] Always `True`, because Python caches every integer used in a program
- [ ] Raises `OverflowError` since 1000 doesn't fit in a cached slot

<details>
<summary>Show Answer</summary>

**Answer:** B — Unspecified — CPython may print `True` or `False` depending on compile-time constant folding and context, so don't rely on it

**Explanation:** Only integers from -5 to 256 are guaranteed cached singletons. Outside that range, whether `a is b` is `True` depends on implementation details like whether both `1000` literals get folded into the same code object's constant pool (common when both lines are compiled together, e.g. inside one script or function) versus compiled as separate top-level statements at an interactive prompt. This is not part of the language spec and can vary by CPython version or context, so A's "always `False`" is too absolute — real CPython often prints `True` in a script. The takeaway either way: never use `is` for integer value comparison, use `==`.

</details>
::

::question-wrapper{language="python"}
### Q7. What does the following print?

::code-wrapper{language="python"}
```python
a = "hello"
b = "hello"
c = "".join(["he", "llo"])
print(a is b, a is c)
```
::

- [ ] `True True`
- [ ] `False False`
- [ ] `True False`
- [ ] `False True`

<details>
<summary>Show Answer</summary>

**Answer:** C — `True False`

**Explanation:** **Idiom** — `a` and `b` are the identical literal `"hello"` compiled into the same code object, so CPython typically interns/dedupes it, making `a is b` `True`. `c` is built at runtime via `str.join`, producing a fresh string object with equal content but a distinct identity, so `a is c` is `False` even though `a == c` is `True`. String interning is a CPython implementation detail, not a language guarantee — never rely on `is` for string equality, always use `==`.

</details>
::

::question-wrapper{language="python"}
### Q8. What does the following print?

::code-wrapper{language="python"}
```python
print(-7 // 2)
```
::

- [ ] `-3`
- [ ] `3`
- [ ] `-3.5`
- [ ] `-4`

<details>
<summary>Show Answer</summary>

**Answer:** D — `-4`

**Explanation:** **Portability** — `//` floors the true quotient toward negative infinity: `-7 / 2 = -3.5`, and `floor(-3.5) = -4`. This differs from C/Java integer division, which truncates toward zero and would give `-3` (option A) — a common bug when porting numeric code between languages.

</details>
::

::question-wrapper{language="python"}
### Q9. What does the following print?

::code-wrapper{language="python"}
```python
print(-7 % 3)
```
::

- [ ] `2`
- [ ] `-1`
- [ ] `-2`
- [ ] `1`

<details>
<summary>Show Answer</summary>

**Answer:** A — `2`

**Explanation:** **Portability** — Python's `%` result always takes the sign of the divisor. `-7 % 3` must satisfy `-7 == 3 * (-7 // 3) + (-7 % 3)`, i.e. `-7 == 3 * (-3) + 2`, so the remainder is `2`. B (`-1`) is what a C/Java-style truncating modulo would give, since it takes the sign of the dividend instead.

</details>
::

::question-wrapper{language="python"}
### Q10. What does the following print?

::code-wrapper{language="python"}
```python
print(divmod(-7, 3))
```
::

- [ ] `(-2, -1)`
- [ ] `(-3, 2)`
- [ ] `(-3, -1)`
- [ ] `(-2, 2)`

<details>
<summary>Show Answer</summary>

**Answer:** B — `(-3, 2)`

**Explanation:** `divmod(a, b)` returns the pair `(a // b, a % b)`. Here `-7 // 3 = -3` (floors toward negative infinity) and `-7 % 3 = 2` (sign of the divisor), so `divmod(-7, 3)` bundles both results from the same floor-division/modulo rules seen in Q8/Q9 into one tuple.

</details>
::

::question-wrapper{language="python"}
### Q11. What does the following print?

::code-wrapper{language="python"}
```python
print(2 ** 3 ** 2)
```
::

- [ ] `64`
- [ ] `1024`
- [ ] `512`
- [ ] `216`

<details>
<summary>Show Answer</summary>

**Answer:** C — `512`

**Explanation:** Unlike almost every other binary operator in Python, `**` is right-associative, so this is `2 ** (3 ** 2)` = `2 ** 9` = `512`. Left-associative evaluation, `(2 ** 3) ** 2` = `8 ** 2` = `64` (option A), is what you'd get if `**` behaved like `+`, `-`, `*`, or `/` — it's the one operator where associativity genuinely trips people up.

</details>
::

::question-wrapper{language="python"}
### Q12. What does the following print?

::code-wrapper{language="python"}
```python
class Money:
    def __init__(self, amount):
        self.amount = amount

    def __radd__(self, other):
        return Money(self.amount + other)

result = 5 + Money(10)
print(result.amount)
```
::

- [ ] Raises `TypeError`
- [ ] `5`
- [ ] `None`
- [ ] `15`

<details>
<summary>Show Answer</summary>

**Answer:** D — `15`

**Explanation:** Python evaluates `5 + Money(10)` by first trying `int.__add__(5, Money(10))`. `int` has no idea how to add a `Money`, so that returns `NotImplemented`. Python then falls back to the reflected method on the right operand, `Money(10).__radd__(5)`, which returns `Money(15)`. A would only be correct if `Money` defined no `__radd__` at all — then both dunder attempts fail and Python genuinely raises `TypeError`.

</details>
::

::question-wrapper{language="python"}
### Q13. What does the following print?

::code-wrapper{language="python"}
```python
a = [1, 2]
b = a
a += [3]

x = 10
y = x
x += 1

print(b, y)
```
::

- [ ] `[1, 2, 3] 10`
- [ ] `[1, 2] 10`
- [ ] `[1, 2, 3] 11`
- [ ] `[1, 2] 11`

<details>
<summary>Show Answer</summary>

**Answer:** A — `[1, 2, 3] 10`

**Explanation:** Lists are mutable, so `a += [3]` invokes `list.__iadd__`, which mutates the list in place and keeps `a`'s identity unchanged — since `b` is an alias to that same object, `b` also shows `[1, 2, 3]`. Ints are immutable, so `x += 1` can't mutate `10` in place; it rebinds `x` to a brand-new object `11`, leaving `y` still pointing at the original `10`. Option C's `11` for `y` is the mistake of assuming `+=` always ripples to every alias the way it does for lists.

</details>
::

::question-wrapper{language="python"}
### Q14. What does the following print?

::code-wrapper{language="python"}
```python
print(0.1 + 0.2 == 0.3)
```
::

- [ ] `True`
- [ ] `False`
- [ ] Raises `TypeError`
- [ ] `0.3`

<details>
<summary>Show Answer</summary>

**Answer:** B — `False`

**Explanation:** **Debug** — `0.1`, `0.2`, and `0.3` have no exact representation in IEEE-754 binary floating point. `0.1 + 0.2` actually evaluates to `0.30000000000000004`, which is not bit-for-bit equal to the literal `0.3`. This is a floating-point representation issue, not a Python-specific bug, and it also underlies mixed-type arithmetic promotion (`int + float` always promotes to `float`, inheriting these same representation limits). The fix is comparing with a tolerance, e.g. `math.isclose(0.1 + 0.2, 0.3)`, instead of `==`.

</details>
::

::question-wrapper{language="python"}
### Q15. What does the following print?

::code-wrapper{language="python"}
```python
print(0 or "default")
```
::

- [ ] `True`
- [ ] `0`
- [ ] `default`
- [ ] `False`

<details>
<summary>Show Answer</summary>

**Answer:** C — `default`

**Explanation:** **Idiom** — `and`/`or` don't coerce their result to `bool`; they evaluate operands left to right and return the first operand that decides the outcome, as-is. `0` is falsy, so `or` moves on and returns the actual value `"default"`. B is what a strict boolean-`or` language would give; A/D wrongly assume Python's logical operators always return `bool`.

</details>
::

::question-wrapper{language="python"}
### Q16. What does the following print?

::code-wrapper{language="python"}
```python
def choose(flag):
    return flag and 0 or 5

print(choose(3))
```
::

- [ ] `0`
- [ ] `3`
- [ ] `None`
- [ ] `5`

<details>
<summary>Show Answer</summary>

**Answer:** D — `5`

**Explanation:** **Idiom** — `3 and 0` evaluates to `0`, because the left operand `3` is truthy so `and` evaluates and returns the right operand. Then `0 or 5` evaluates to `5`, because `0` is falsy so `or` moves on to its right operand. `choose` was clearly meant to return `0` when `flag` is truthy, but the classic `flag and X or Y` idiom silently breaks whenever `X` itself is falsy — which is exactly why Python's conditional expression `0 if flag else 5` should be preferred over this pattern. A is the "intended" answer a reader would wrongly expect.

</details>
::

::question-wrapper{language="python"}
### Q17. What does the following print?

::code-wrapper{language="python"}
```python
a = 5
b = 3
print(a & b, a and b)
```
::

- [ ] `1 3`
- [ ] `1 1`
- [ ] `7 3`
- [ ] `7 5`

<details>
<summary>Show Answer</summary>

**Answer:** A — `1 3`

**Explanation:** `&` is the bitwise AND operator: `5` (`101`) `&` `3` (`011`) = `001` = `1`. `and` is the logical operator and, as in Q15, returns an actual operand rather than a recombined value: since `a` (`5`) is truthy, `and` evaluates and returns `b` (`3`). This distinction matters a lot with array-like objects (e.g. NumPy arrays or pandas Series), where `&`/`|` are required for element-wise boolean logic because `and`/`or` can't be overloaded to short-circuit per-element and instead try to coerce the whole array to a single `bool`, raising an error.

</details>
::

::question-wrapper{language="python"}
### Q18. What does the following print?

::code-wrapper{language="python"}
```python
a = 1
b = 2
print(not a == b)
```
::

- [ ] `False`
- [ ] `True`
- [ ] Raises `SyntaxError`
- [ ] `1`

<details>
<summary>Show Answer</summary>

**Answer:** B — `True`

**Explanation:** `not` binds more loosely than comparison operators, so this parses as `not (a == b)` = `not False` = `True`. A is the trap answer from misreading it as `(not a) == b`, i.e. `False == 2` → `False` — but that grouping is not how Python's operator precedence actually works. This same looser-than-comparison binding is why `is not` and `not in` exist as their own compound operators rather than requiring `not (x is y)` / `not (x in y)`.

</details>
::

::question-wrapper{language="python"}
### Q19. What does the following print?

::code-wrapper{language="python"}
```python
print(1 < "a")
```
::

- [ ] `True`
- [ ] `False`
- [ ] Raises `TypeError`
- [ ] `1`

<details>
<summary>Show Answer</summary>

**Answer:** C — Raises `TypeError`

**Explanation:** **Portability** — Python 3 refuses to order-compare incompatible types: `int` has no defined ordering against `str`, both types' `__lt__`/`__gt__` return `NotImplemented` for each other, so Python raises `TypeError: '<' not supported between instances of 'int' and 'str'`. This is a deliberate break from Python 2, which allowed cross-type comparisons using an arbitrary-but-consistent rule — code ported from Python 2 that relied on sorting mixed-type lists can crash outright in Python 3.

</details>
::

::question-wrapper{language="python"}
### Q20. What does the following print?

::code-wrapper{language="python"}
```python
nan = float('nan')
print(nan == nan, nan is nan)
```
::

- [ ] `True True`
- [ ] `True False`
- [ ] `False False`
- [ ] `False True`

<details>
<summary>Show Answer</summary>

**Answer:** D — `False True`

**Explanation:** **Debug** — IEEE-754 defines NaN as unordered and unequal to everything, including itself, so `nan == nan` is `False` even though both sides are literally the same object — `float.__eq__` follows the IEEE rule rather than special-casing identical operands. `nan is nan` is `True` simply because both names refer to the exact same object; identity is trivially reflexive and unaffected by IEEE semantics. Practical implication: a NaN value can silently fail `==`-based lookups or filters, but CPython's dict/set implementation checks identity before falling back to equality, so the *same* NaN object can still be found again as a dict key — a different, merely equal-valued NaN object cannot.

</details>
::
