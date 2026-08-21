# 03 — Operators & Expressions

::question-wrapper{language="javascript"}
### Q1. What is the fundamental difference between `==` and `===`?

::code-wrapper{language="javascript"}
```javascript
console.log(1 == "1");
console.log(1 === "1");
```
::

- [ ] They are functionally identical; `===` is just a stylistic alias for `==`
- [ ] `==` (loose equality) performs type coercion before comparing; `===` (strict equality) compares both value and type with no coercion
- [ ] `===` performs coercion, `==` does not
- [ ] `==` compares references for objects, `===` compares values

<details>
<summary>Show Answer</summary>

**Answer:** B — `==` coerces types before comparing; `===` requires matching type and value with no coercion

**Explanation:** `1 == "1"` coerces the string `"1"` to the number `1` before comparing, yielding `true`. `1 === "1"` compares type first — number vs. string — and since the types differ it immediately returns `false` without any coercion. Option C reverses the actual rule. Option D is unrelated to how these operators work with objects (both `==` and `===` compare object references identically; neither does a deep comparison).

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does `[] == ![]` evaluate to?

::code-wrapper{language="javascript"}
```javascript
console.log([] == ![]);
```
::

- [ ] `false`, because an array is never equal to a boolean
- [ ] `true` — `![]` evaluates to `false` first (since `[]` is truthy, `!` negates it), then `[] == false` coerces `[]` to `""` and `false` to `0`, and `""` coerces to `0`, so `0 == 0` is `true`
- [ ] `NaN`, because arrays can't be converted to numbers
- [ ] `SyntaxError`, since `!` cannot be applied to an array literal

<details>
<summary>Show Answer</summary>

**Answer:** B — `true`, through a chain of coercions

**Explanation:** **Debug:** this is a famous JavaScript "wat" moment. Step by step: `[]` is a truthy object, so `![]` is `false`. Now the comparison is `[] == false`. Per the `==` algorithm, when one side is boolean, it's converted to a number first: `false` → `0`. The other side `[]` is an object compared against a number, so it's converted via `ToPrimitive` → `""` (empty string), then `""` → `0` (number coercion of an empty string). Now it's `0 == 0`, which is `true`. Every step is individually "correct" per spec, which is exactly what makes the combined result so counterintuitive. This is a textbook argument for avoiding `==` (option A is the intuitive-but-wrong guess).

</details>
::

::question-wrapper{language="javascript"}
### Q3. What is the difference between `&&`/`||` short-circuiting and how does it affect side effects?

::code-wrapper{language="javascript"}
```javascript
function logAndReturn(val) {
  console.log("called with", val);
  return val;
}
const result = false && logAndReturn(1);
```
::

- [ ] `logAndReturn(1)` is always called regardless of the left operand
- [ ] `&&` only evaluates the right operand if the left is truthy; since `false` is falsy, `logAndReturn(1)` is never called, and `console.log` never runs — `result` is `false`
- [ ] This throws a `TypeError` because `&&` requires both operands to be booleans
- [ ] `result` is `1`, because `&&` always evaluates to the last operand

<details>
<summary>Show Answer</summary>

**Answer:** B — `&&` short-circuits on a falsy left operand, so the right side is never evaluated

**Explanation:** `&&` only evaluates its right-hand operand if the left is truthy (it "short-circuits" otherwise); `||` is the mirror — it skips the right side once the left is truthy. Since `false` is falsy, `logAndReturn(1)` is never invoked at all — no log line prints, and `result` is `false` (the value of the left operand, not a boolean cast of it). **Debug:** relying on short-circuiting for side effects (e.g., `isLoggedIn && trackEvent()`) is idiomatic but can silently skip important calls if the guard condition is wrong — worth being deliberate about. Option D is a common misconception; `&&`/`||` return one of the actual operand values, not necessarily a boolean.

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does the nullish coalescing operator `??` do differently from `||`?

::code-wrapper{language="javascript"}
```javascript
const count = 0;
console.log(count || 10);
console.log(count ?? 10);
```
::

- [ ] They behave identically in all cases
- [ ] `||` returns the right operand if the left is any falsy value (`0`, `""`, `NaN`, `false`, `null`, `undefined`); `??` only returns the right operand if the left is specifically `null` or `undefined` — so `count || 10` wrongly overrides a legitimate `0`, while `count ?? 10` correctly preserves it
- [ ] `??` treats `0` the same as `null`, so both log `10`
- [ ] `||` only checks for `null`/`undefined`, and `??` checks all falsy values — the reverse of the truth

<details>
<summary>Show Answer</summary>

**Answer:** B — `||` falls back on any falsy value; `??` only falls back on `null`/`undefined`

**Explanation:** **Debug:** this is one of the most common real-world bugs `??` (ES2020) was introduced to fix: `someConfig.retries || 5` silently replaces a deliberately-set `0` retries value with the default `5`, because `0` is falsy. `someConfig.retries ?? 5` correctly keeps `0` since `0` is neither `null` nor `undefined`. So here, `count || 10` logs `10` (wrong, loses the real `0`), while `count ?? 10` logs `0` (correct). Option C and D invert or misstate the actual distinction.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does optional chaining `?.` do when the left-hand side is `null` or `undefined`?

::code-wrapper{language="javascript"}
```javascript
const user = { profile: null };
console.log(user.profile?.bio);
console.log(user.profile.bio);
```
::

- [ ] Both lines throw the same error
- [ ] `user.profile?.bio` short-circuits and returns `undefined` without throwing, since `user.profile` is `null`; `user.profile.bio` (no `?.`) throws `TypeError: Cannot read properties of null (reading 'bio')`
- [ ] `user.profile?.bio` throws, but `user.profile.bio` returns `undefined`
- [ ] Both lines return `undefined` safely, since JavaScript never throws on property access

<details>
<summary>Show Answer</summary>

**Answer:** B — `?.` short-circuits to `undefined` on `null`/`undefined`; plain access throws

**Explanation:** Optional chaining checks the left side before attempting property access — if it's `null` or `undefined`, the entire chain short-circuits and evaluates to `undefined` immediately, skipping the rest of the expression. Without `?.`, accessing `.bio` on `null` throws a `TypeError`, since `null` has no properties. **Safety:** the fix for the second line's error is exactly to add `?.`, or to explicitly check `user.profile !== null` first. Option D falsely claims JS never throws on property access — it does, for both `null` and `undefined`.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What is the correct operator precedence outcome here?

::code-wrapper{language="javascript"}
```javascript
console.log(2 + 3 * 4);
console.log((2 + 3) * 4);
```
::

- [ ] `20` and `20`
- [ ] `14` and `20`
- [ ] `20` and `14`
- [ ] `14` and `14`

<details>
<summary>Show Answer</summary>

**Answer:** B — `14` and `20`

**Explanation:** `*` has higher precedence than `+`, so `2 + 3 * 4` computes `3 * 4 = 12` first, then `2 + 12 = 14`. Parentheses override natural precedence, so `(2 + 3) * 4` forces `5 * 4 = 20`. This is standard math-like precedence, but it's worth internalizing because JavaScript has many more operators with less intuitive precedence than arithmetic (e.g., where does `??` sit relative to `||`, or `typeof` relative to `**`).

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does associativity determine, and how does `**` (exponentiation) differ from `-` (subtraction) in this regard?

::code-wrapper{language="javascript"}
```javascript
console.log(2 ** 3 ** 2);
console.log(10 - 3 - 2);
```
::

- [ ] Both evaluate left-to-right: `64` and `5`
- [ ] `**` is right-associative (`2 ** (3 ** 2)` = `2 ** 9` = `512`); `-` is left-associative (`(10 - 3) - 2` = `5`)
- [ ] `**` is left-associative like `-`, giving `64` and `5`
- [ ] Both are right-associative, giving `512` and `9`

<details>
<summary>Show Answer</summary>

**Answer:** B — `**` is right-associative (`512`); `-` is left-associative (`5`)

**Explanation:** Associativity determines grouping order when the same-precedence operator repeats. Most operators (`-`, `+`, `*`, `/`) are left-associative, evaluating left to right: `10 - 3 - 2` groups as `(10 - 3) - 2 = 5`. Exponentiation `**` is a deliberate exception — it's right-associative, matching mathematical convention where `2^3^2` means `2^(3^2)`, not `(2^3)^2`. **Debug:** so `2 ** 3 ** 2` groups as `2 ** (3 ** 2) = 2 ** 9 = 512`, not `(2 ** 3) ** 2 = 64` — a frequent trip-up for people assuming all arithmetic operators associate the same way.

</details>
::

::question-wrapper{language="javascript"}
### Q8. What happens when you compare with `<`/`>` across mixed types, e.g., strings and numbers?

::code-wrapper{language="javascript"}
```javascript
console.log("10" < "9");
console.log(10 < 9);
console.log("10" < 9);
```
::

- [ ] `false`, `false`, `false`
- [ ] `true`, `false`, `false` — the first compares two strings lexicographically ("1" < "9" character-wise), the second and third involve numeric comparison after coercion
- [ ] All three are `true`
- [ ] `SyntaxError`, since strings cannot be compared with `<`

<details>
<summary>Show Answer</summary>

**Answer:** B — `true`, `false`, `false`

**Explanation:** `"10" < "9"` compares two strings *lexicographically* (character by character, like dictionary order), not numerically: the first character `"1"` has a lower char code than `"9"`, so it's `true` — this is the trap, since numerically 10 is greater than 9. `10 < 9` is a normal numeric comparison: `false`. `"10" < 9` mixes a string and a number, so the relational operator coerces the string to a number (`10`), giving `10 < 9` → `false`. **Debug:** relational operators only compare lexicographically when *both* sides are strings; introduce any non-string operand and numeric coercion kicks in instead — an inconsistency that trips up sorting and comparison logic on stringified data (e.g., data from form inputs or JSON).

</details>
::

::question-wrapper{language="javascript"}
### Q9. What is the result of comparing with `NaN` using relational operators?

::code-wrapper{language="javascript"}
```javascript
console.log(NaN < 1);
console.log(NaN > 1);
console.log(NaN <= NaN);
console.log(NaN == NaN);
```
::

- [ ] `true`, `false`, `true`, `false`
- [ ] `false`, `false`, `false`, `false` — every comparison involving `NaN` (including `NaN` against itself) is `false`
- [ ] `false`, `false`, `true`, `true`
- [ ] Throws `TypeError` on every line

<details>
<summary>Show Answer</summary>

**Answer:** B — `false`, `false`, `false`, `false`

**Explanation:** **Debug:** `NaN` compares as `false` against literally everything, including itself, for every relational and equality operator (`<`, `>`, `<=`, `>=`, `==`, `===`). This is by IEEE-754 design — `NaN` is defined as "unordered" relative to all values. It never throws (option D is wrong); it just always evaluates to `false`. This is exactly why `Number.isNaN()` (not a comparison operator) is the only reliable way to detect `NaN`.

</details>
::

::question-wrapper{language="javascript"}
### Q10. What is the value of an empty expression involving the comma operator?

::code-wrapper{language="javascript"}
```javascript
const result = (1 + 2, 3 + 4, 5 + 6);
console.log(result);
```
::

- [ ] `SyntaxError` — commas are not valid inside parentheses like this
- [ ] `3`, the result of the first expression
- [ ] `11`, the result of the second expression
- [ ] `11`, since the comma operator evaluates each operand left to right and yields the value of the *last* one

<details>
<summary>Show Answer</summary>

**Answer:** D — `11`, the value of the last comma-separated expression

**Explanation:** The comma operator evaluates each expression in sequence (for their side effects) and yields only the final one's value. Here `1 + 2` (=3) and `3 + 4` (=7) are computed and discarded, and `5 + 6` (=11) becomes the expression's value, assigned to `result`. This operator is rarely used explicitly outside of terse `for` loop headers (`for (let i = 0, j = 10; i < j; i++, j--)`) — using it for arbitrary sequencing like this example is unusual and hurts readability, but is valid, well-defined syntax, not an error (ruling out option A).

</details>
::

::question-wrapper{language="javascript"}
### Q11. What does the unary `+` operator do to a string operand?

::code-wrapper{language="javascript"}
```javascript
console.log(+"42");
console.log(+"");
console.log(+"  ");
console.log(+"abc");
console.log(+null);
console.log(+undefined);
```
::

- [ ] `42`, `NaN`, `NaN`, `NaN`, `0`, `NaN`
- [ ] `42`, `0`, `0`, `NaN`, `0`, `NaN` — unary `+` coerces to a number; an empty or whitespace-only string becomes `0`, a non-numeric string becomes `NaN`, `null` becomes `0`, and `undefined` becomes `NaN`
- [ ] `"42"`, `""`, `""`, `"abc"`, `null`, `undefined` (no coercion happens)
- [ ] All six results are `NaN`

<details>
<summary>Show Answer</summary>

**Answer:** B — `42`, `0`, `0`, `NaN`, `0`, `NaN`

**Explanation:** Unary `+` is a common idiom for explicit numeric coercion. Strings are converted via `Number()` rules: numeric-looking strings parse normally, and — the gotcha — an empty string or a string of only whitespace converts to `0`, not `NaN` (since `Number("")` trims to nothing and treats that as zero). A truly non-numeric string like `"abc"` produces `NaN`. Separately, `Number(null)` is `0` (a well-known inconsistency), while `Number(undefined)` is `NaN`. **Debug:** the empty-string-becomes-zero behavior is a frequent source of validation bugs when checking "is this field a number" using truthiness or unary `+` alone.

</details>
::

::question-wrapper{language="javascript"}
### Q12. In `a ?? b || c`, what happens?

::code-wrapper{language="javascript"}
```javascript
const a = null;
console.log(a ?? b || c);
```
::

- [ ] It evaluates fine, left to right, treating `??` and `||` as equal precedence
- [ ] It throws a `SyntaxError` — mixing `??` directly with `||` (or `&&`) without explicit parentheses is a syntax error, precisely because their relative precedence/associativity is ambiguous and disallowed by the spec
- [ ] `??` always takes precedence and short-circuits, ignoring `||`
- [ ] `||` always takes precedence over `??`

<details>
<summary>Show Answer</summary>

**Answer:** B — Mixing `??` with `||`/`&&` without parentheses is a `SyntaxError`

**Explanation:** **Safety:** unlike most operators, which have a well-defined relative precedence, the spec explicitly forbids writing `??` directly adjacent to `||` or `&&` without parentheses (`(a ?? b) || c` or `a ?? (b || c)`), because their intended grouping is genuinely ambiguous and error-prone to guess. This is a deliberate design choice to force developers to be explicit rather than rely on a precedence table few people would remember correctly. Options C and D each assume an implicit ordering that the language intentionally refuses to define without parentheses.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What is the output of this logical assignment operator usage?

::code-wrapper{language="javascript"}
```javascript
let config = { timeout: 0 };
config.timeout ??= 3000;
config.retries ||= 5;
console.log(config);
```
::

- [ ] `{ timeout: 3000, retries: 5 }` — both operators overwrite because `0` is falsy
- [ ] `{ timeout: 0, retries: 5 }` — `??=` only assigns if the left is `null`/`undefined` (so `timeout` stays `0`); `||=` assigns if the left is falsy or missing (so `retries`, being `undefined`, becomes `5`)
- [ ] `{ timeout: 0 }` — `retries` is left untouched since it was never declared
- [ ] `TypeError`, since you can't use logical assignment on object properties

<details>
<summary>Show Answer</summary>

**Answer:** B — `{ timeout: 0, retries: 5 }`

**Explanation:** `??=` (ES2021) is shorthand for `x ?? (x = y)` — it only assigns when the current value is `null`/`undefined`. Since `timeout` is `0` (not nullish), `??=` leaves it untouched. `retries` doesn't exist yet, so accessing it is `undefined`, which is both nullish and falsy — `||=` assigns `5`. **Idiom:** `??=` is the safer choice specifically for numeric defaults like `timeout` where `0` is a meaningful, valid value that shouldn't be clobbered — mirroring the `??` vs `||` distinction from Q4.

</details>
::

::question-wrapper{language="javascript"}
### Q14. What does `typeof` return for these edge-case expressions?

::code-wrapper{language="javascript"}
```javascript
console.log(typeof (1 < 2));
console.log(typeof typeof 1);
console.log(typeof (1, "a"));
```
::

- [ ] `"boolean"`, `"number"`, `"string"`
- [ ] `"boolean"`, `"string"`, `"string"` — `typeof` on a comparison gives `"boolean"`; `typeof typeof x` is always `"string"` because `typeof` itself always returns a string, and the comma operator yields its last operand (`"a"`) before `typeof` runs on it
- [ ] `"boolean"`, `"number"`, `"number"`
- [ ] `SyntaxError` on the last line, since `typeof` cannot be applied to a comma expression

<details>
<summary>Show Answer</summary>

**Answer:** B — `"boolean"`, `"string"`, `"string"`

**Explanation:** `1 < 2` evaluates to the boolean `true`, so `typeof` gives `"boolean"`. `typeof 1` evaluates to the string `"number"`; applying `typeof` to *that* result (a string) gives `"string"` — a fun consequence of `typeof` always producing a string value, so nesting it twice always ends in `"string"` regardless of the original operand. The comma expression `(1, "a")` evaluates to `"a"` (per Q10's rule), and `typeof "a"` is `"string"`. Option D wrongly assumes `typeof` can't handle a parenthesized comma expression — it's just a normal expression to `typeof`.

</details>
::

::question-wrapper{language="javascript"}
### Q15. What is the idiomatic reason to prefer `===`/`!==` over `==`/`!=` in production code?

::code-wrapper{language="javascript"}
```javascript
function isAdmin(role) {
  return role == "admin";
}
```
::

- [ ] `==` is deprecated and will be removed from future ECMAScript versions
- [ ] `===` avoids the unpredictable, hard-to-audit coercion rules of `==` (as seen with `[] == false` and similar cases), making comparisons behave exactly as written; `==` should be reserved for the few well-understood idioms like `x == null`
- [ ] `==` is slower at runtime than `===` in all engines, so `===` is purely a performance optimization
- [ ] There's no real-world difference; the choice is purely stylistic with zero behavioral impact

<details>
<summary>Show Answer</summary>

**Answer:** B — `===` avoids `==`'s unpredictable coercion; `==` is reserved for well-understood idioms like `x == null`

**Explanation:** **Idiom:** the core argument for `===` is correctness and reviewability, not performance — coercion rules like the array/boolean example from Q2 are genuinely hard to reason about at a glance, and bugs from accidental type mismatches (e.g., comparing a form input string against a numeric constant) are common. Style guides carve out `== null` as an accepted exception (per Q19 of the previous file) precisely because that one coercion behavior is well-known and intentional. Option A is false — `==` is not deprecated or going anywhere. Option C overstates and mischaracterizes the actual (in practice negligible) performance difference as the primary motivation.

</details>
::

::question-wrapper{language="javascript"}
### Q16. What is the best-practice way to write a conditional default value assignment when `0`, `""`, or `false` might be legitimate values?

::code-wrapper{language="javascript"}
```javascript
function createUser({ retries = 3, isActive = true } = {}) {}
```
::

- [ ] Always use `||` for defaults: `const r = retries || 3;`
- [ ] Prefer default parameters (as shown) or `??` over `||` for fallback values, since both correctly distinguish "not provided" (`undefined`) from an intentionally falsy value like `0` or `false`
- [ ] Use a ternary checking `typeof x === "boolean"` for every possible falsy type individually
- [ ] It never matters which operator is used, since defaults are rarely falsy in practice

<details>
<summary>Show Answer</summary>

**Answer:** B — Prefer default parameters or `??` since they correctly distinguish "not provided" from a legitimate falsy value

**Explanation:** **Idiom:** default parameters (as in the `createUser` signature) only kick in when an argument is literally `undefined` — passing `retries: 0` explicitly is respected, not overridden — which mirrors `??`'s "nullish, not falsy" semantics discussed in Q4. Using `||` for defaults, by contrast, is a common but subtly wrong pattern that silently discards deliberate falsy inputs like `0`, `""`, or `false`. Option C is technically workable but needlessly verbose compared to the built-in mechanisms designed for exactly this. Option D is wrong in practice — falsy-but-valid defaults (counts, flags, empty strings) are extremely common in real APIs.

</details>
::

::question-wrapper{language="javascript"}
### Q17. Which is the idiomatic pattern for guarding against `null`/`undefined` when calling a method that might not exist?

::code-wrapper{language="javascript"}
```javascript
const config = getConfig();
```
::

- [ ] `config && config.onReady && config.onReady()`
- [ ] `config?.onReady?.()` — optional chaining combined with the optional call syntax, which short-circuits to `undefined` at the first `null`/`undefined` link without throwing, and is more concise/less error-prone than manual `&&` chains
- [ ] `if (config.onReady !== undefined) config.onReady();` (no chaining needed)
- [ ] `try { config.onReady(); } catch (e) {}` — swallow any resulting error

<details>
<summary>Show Answer</summary>

**Answer:** B — `config?.onReady?.()` using optional chaining plus optional call syntax

**Explanation:** **Idiom:** `?.()` (optional call, part of the same ES2020 optional chaining feature) skips the call entirely if `config` or `config.onReady` is nullish, matching the intent of the older `&&`-chain pattern (option A) but far more concisely and without repeating each property name. Option C is unsafe — if `config` itself is `null`/`undefined`, accessing `.onReady` on it throws before the check even runs. Option D is a code smell: swallowing all errors silently (including genuine bugs inside `onReady`) makes debugging much harder — the correct approach is to only guard against the specific "doesn't exist" case, not blanket-catch everything.

</details>
::

::question-wrapper{language="javascript"}
### Q18. Why can chained optional chaining still throw in some cases, and how should that be handled?

::code-wrapper{language="javascript"}
```javascript
const user = { getProfile: null };
console.log(user.getProfile());
console.log(user.getProfile?.());
```
::

- [ ] Both lines behave identically since `getProfile` is falsy either way
- [ ] `user.getProfile()` throws `TypeError: user.getProfile is not a function`, because `null` is not callable, and plain (non-optional) call syntax always attempts the call; `user.getProfile?.()` correctly short-circuits to `undefined` instead, since `?.()` checks for nullish before attempting to invoke
- [ ] `user.getProfile?.()` also throws, because `?.` doesn't work on function calls, only property access
- [ ] Neither line throws, since JS treats calling `null` as a no-op

<details>
<summary>Show Answer</summary>

**Answer:** B — Plain calling `null` throws `TypeError`; `?.()` correctly short-circuits instead

**Explanation:** **Safety:** `getProfile` here is `null`, not a function — calling it directly attempts `null()`, which is a `TypeError` since `null` isn't callable. `?.()` is specifically designed to guard exactly this: if the reference just before it is nullish, the call is skipped and the whole expression evaluates to `undefined`. The idiomatic fix for code that might call a possibly-missing/possibly-null method is exactly this optional call syntax rather than a manual `typeof x === "function"` check everywhere. Option C is wrong — `?.()` is valid, dedicated syntax for guarding function calls, not just property reads.

</details>
::

::question-wrapper{language="javascript"}
### Q19. In a chain of mixed logical/comparison operators used for validation, what's the risk of relying on operator precedence instead of explicit parentheses?

::code-wrapper{language="javascript"}
```javascript
function canCheckout(cart, user) {
  return cart.length > 0 && user.verified || user.isAdmin;
}
```
::

- [ ] There's no risk; `&&` and `||` always evaluate strictly left to right regardless of precedence
- [ ] `&&` binds tighter than `||`, so this actually means `(cart.length > 0 && user.verified) || user.isAdmin` — which may be the intended logic, but relying on readers to recall that `&&` outranks `||` (rather than writing the parens explicitly) risks misreadings and subtle bugs when the expression grows more complex
- [ ] `||` binds tighter than `&&`, so this means `cart.length > 0 && (user.verified || user.isAdmin)`
- [ ] This is a `SyntaxError` because `&&` and `||` cannot be mixed without parentheses (like `??`/`||`)

<details>
<summary>Show Answer</summary>

**Answer:** B — `&&` binds tighter, meaning `(... && ...) || ...`; explicit parens are still best practice to avoid misreadings

**Explanation:** **Idiom:** `&&` does have higher precedence than `||` (unlike the `??`/`||` combination, which the spec explicitly disallows without parens — see Q12 — `&&` and `||` together are legal and well-defined). Here that means admins bypass the cart check entirely, which may or may not be the intended business logic. Because getting this precedence order wrong is an easy, silent mistake (and because `&&`/`||`/`??` chains multiply in complexity fast), production code should make grouping explicit with parentheses even when not strictly required — self-documenting code beats a mentally-recalled precedence table. Option D wrongly extends the `??`/`||` restriction to `&&`/`||`, which don't share that restriction.

</details>
::

::question-wrapper{language="javascript"}
### Q20. What is the best-practice way to compare two numbers for "close enough" equality given floating-point imprecision?

::code-wrapper{language="javascript"}
```javascript
function isCloseEnough(a, b) {
  return a === b;
}
console.log(isCloseEnough(0.1 + 0.2, 0.3));
```
::

- [ ] Keep using `===`; floating-point issues are rare enough to ignore in production
- [ ] Compare the absolute difference against a small epsilon: `Math.abs(a - b) < Number.EPSILON` (or a domain-appropriate tolerance), rather than exact equality, since binary floating-point arithmetic accumulates tiny representation errors
- [ ] Convert both numbers to strings and compare those instead
- [ ] Use `==` instead of `===` to allow "reasonably close" values to match

<details>
<summary>Show Answer</summary>

**Answer:** B — Compare via an epsilon-tolerant difference check instead of exact equality

**Explanation:** **Idiom:** as established in the Variables topic, `0.1 + 0.2 !== 0.3` due to IEEE-754 rounding, so `isCloseEnough` above returns `false` for what a user would consider "the same" value. The standard fix is tolerance-based comparison — `Math.abs(a - b) < Number.EPSILON` for values near the precision limit, or a larger domain-specific epsilon (e.g., `0.0001`) for currency/measurement contexts. Option A dismisses a genuinely common bug source (any repeated decimal arithmetic — totals, percentages). Option C is a fragile workaround that breaks on formatting differences (`"0.30000000000000004"` vs `"0.3"`). Option D is wrong — `==` performs *type* coercion, not numeric fuzzy-matching; `0.1 + 0.2 == 0.3` is still `false`.

</details>
::
