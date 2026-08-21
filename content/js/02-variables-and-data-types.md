# 02 — Variables & Data Types

::question-wrapper{language="javascript"}
### Q1. What is the key scoping difference between `var` and `let`?

::code-wrapper{language="javascript"}
```javascript
if (true) {
  var a = 1;
  let b = 2;
}
console.log(a);
console.log(b);
```
::

- [ ] Both `a` and `b` are accessible outside the block; the code logs `1` then `2`
- [ ] `var` is function/global-scoped so `a` logs `1`; `let` is block-scoped so accessing `b` throws a `ReferenceError`
- [ ] Neither is accessible outside the block; both throw `ReferenceError`
- [ ] `let` is function-scoped and `var` is block-scoped, the reverse of the truth

<details>
<summary>Show Answer</summary>

**Answer:** B — `var` is function/global-scoped so `a` logs `1`; `let` is block-scoped so accessing `b` throws

**Explanation:** `var` ignores block boundaries (`{}`) and attaches to the nearest function or global scope, so `a` leaks out of the `if` block. `let` (and `const`) are genuinely block-scoped, so `b` does not exist outside the `if` block, and referencing it throws `ReferenceError: b is not defined`. Option A wrongly assumes both leak. Option D swaps the actual rule.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What is variable hoisting, as it applies to `var`?

::code-wrapper{language="javascript"}
```javascript
console.log(x);
var x = 5;
```
::

- [ ] This throws a `ReferenceError` because `x` is used before declaration
- [ ] This logs `undefined`, because the declaration `var x` is hoisted to the top of scope but the assignment `= 5` is not
- [ ] This logs `5`, because JavaScript hoists both the declaration and the assignment
- [ ] This is a `SyntaxError` because `var` cannot be declared after use

<details>
<summary>Show Answer</summary>

**Answer:** B — Logs `undefined`; the declaration is hoisted but the assignment stays in place

**Explanation:** During compilation, `var x` is conceptually moved to the top of its enclosing scope and initialized to `undefined`; the `x = 5` assignment still executes at its original position. So by the time `console.log(x)` runs, `x` exists but hasn't been assigned yet. Option A confuses this with `let`/`const`'s TDZ behavior. Option C is the common misconception that hoisting moves the whole statement, initializer included — it only moves the declaration.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What is the Temporal Dead Zone (TDZ)?

::code-wrapper{language="javascript"}
```javascript
console.log(y);
let y = 10;
```
::

- [ ] The period during which a `var` variable holds `undefined` before assignment
- [ ] The span between entering a scope and a `let`/`const` variable's declaration line, during which accessing the variable throws a `ReferenceError` instead of returning `undefined`
- [ ] A deprecated ES5 feature no longer relevant in modern JS
- [ ] The time it takes the garbage collector to free an unused variable

<details>
<summary>Show Answer</summary>

**Answer:** B — The span before a `let`/`const` declaration executes, where accessing it throws instead of returning `undefined`

**Explanation:** `let` and `const` are hoisted too, but unlike `var` they are not initialized to `undefined` — they remain in an uninitialized "temporal dead zone" from the top of the block until their declaration executes. **Debug:** this is why `console.log(y)` here throws `ReferenceError: Cannot access 'y' before initialization` rather than silently logging `undefined` like the `var` case would. Option A describes `var`'s actual (different) behavior. Option C and D are fabricated.

</details>
::

::question-wrapper
### Q4. Which of the following are JavaScript primitive types? (Select the option that lists ONLY primitives)

- [ ] `string`, `number`, `Array`, `boolean`
- [ ] `string`, `number`, `boolean`, `undefined`, `null`, `symbol`, `bigint`
- [ ] `object`, `function`, `Date`, `Map`
- [ ] `string`, `Number`, `Boolean` (the wrapper objects created via `new`)

<details>
<summary>Show Answer</summary>

**Answer:** B — `string`, `number`, `boolean`, `undefined`, `null`, `symbol`, `bigint`

**Explanation:** JavaScript has exactly seven primitive types: `string`, `number`, `boolean`, `undefined`, `null`, `symbol` (ES2015), and `bigint` (ES2020). Everything else — arrays, plain objects, functions, `Date`, `Map` — is a reference type (`object` under the hood). Option A wrongly includes `Array`, which is a reference type. Option D is a trap: `new String("x")` creates a boxed *object*, not a primitive string — `typeof new String("x")` is `"object"`, not `"string"`.

</details>
::

::question-wrapper{language="javascript"}
### Q5. How are primitives and reference types (objects/arrays) different when assigned to a new variable?

::code-wrapper{language="javascript"}
```javascript
let a = { count: 1 };
let b = a;
b.count = 99;
console.log(a.count);
```
::

- [ ] `1`, because `b` is a fully independent copy of `a`
- [ ] `99`, because `b = a` copies the reference, so `a` and `b` point to the same object in memory
- [ ] `undefined`, because reassigning `b.count` breaks the link to `a`
- [ ] This throws a `TypeError` because objects are immutable by default

<details>
<summary>Show Answer</summary>

**Answer:** B — `99`, because objects are assigned/copied by reference

**Explanation:** Reference types are stored as a pointer to a location in memory; `let b = a` copies that pointer, not the object's contents, so `a` and `b` alias the same object — mutating via one is visible through the other. This is different from primitives, where `let x = 1; let y = x;` gives `y` a fully independent copy of the value. Option A describes primitive copy semantics, mistakenly applied to an object. Option C and D are fabricated — nothing here breaks a link or throws.

</details>
::

::question-wrapper
### Q6. What does `typeof null` evaluate to?

- [ ] `"null"`
- [ ] `"undefined"`
- [ ] `"object"`
- [ ] `"boolean"`

<details>
<summary>Show Answer</summary>

**Answer:** C — `"object"`

**Explanation:** **Debug:** this is one of JavaScript's most famous quirks — `typeof null === "object"` is a bug baked into the language since its first implementation (null was represented internally with the same tag as objects) and has been kept for backward compatibility ever since; it will never be fixed. It does *not* mean `null` is actually an object — `null === undefined` is `false`, and `null` has no properties/methods. Option A is the intuitive-but-wrong guess. Option B confuses `null` with `undefined`, a related but distinct "empty" value.

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does `typeof NaN` evaluate to, and what does `NaN === NaN` evaluate to?

::code-wrapper{language="javascript"}
```javascript
console.log(typeof NaN);
console.log(NaN === NaN);
```
::

- [ ] `"NaN"` and `true`
- [ ] `"number"` and `false`
- [ ] `"undefined"` and `false`
- [ ] `"number"` and `true`

<details>
<summary>Show Answer</summary>

**Answer:** B — `"number"` and `false`

**Explanation:** `NaN` ("Not a Number") is, paradoxically, of type `"number"` — it represents an invalid numeric result but is still part of the number type. **Debug:** `NaN` is the *only* value in JavaScript that is not equal to itself, by IEEE-754 floating-point spec design, so `NaN === NaN` is `false`. To actually test for `NaN`, use `Number.isNaN(x)` (not the legacy, coercing global `isNaN(x)`) or `Object.is(x, NaN)`. Options A, C, and D all misstate one or both facts.

</details>
::

::question-wrapper{language="javascript"}
### Q8. Which of these correctly demonstrates implicit type coercion in a comparison?

::code-wrapper{language="javascript"}
```javascript
console.log("5" == 5);
console.log("5" === 5);
console.log([] == false);
```
::

- [ ] `false`, `false`, `false`
- [ ] `true`, `true`, `true`
- [ ] `true`, `false`, `true`
- [ ] `true`, `false`, `false`

<details>
<summary>Show Answer</summary>

**Answer:** C — `true`, `false`, `true`

**Explanation:** `==` performs type coercion before comparing: `"5" == 5` coerces the string to a number, giving `true`. `===` performs no coercion, so comparing a string to a number is always `false`. `[] == false` is a classic gotcha: `[]` is coerced to `""` (via `ToPrimitive`), then `""` and `false` are both coerced to numbers (`0` and `0`), so it's `true` — despite an array and a boolean seeming completely unrelated. **Idiom:** this exact unpredictability is why `===` is recommended over `==` in production code.

</details>
::

::question-wrapper{language="javascript"}
### Q9. What is the result of adding a number and a string with `+`?

::code-wrapper{language="javascript"}
```javascript
console.log(1 + "2");
console.log(1 + 2 + "3");
console.log("1" + 2 + 3);
```
::

- [ ] `"12"`, `"33"`, `"123"`
- [ ] `3`, `"33"`, `"15"`
- [ ] `"12"`, `"33"`, `"33"`
- [ ] `NaN`, `NaN`, `"123"`

<details>
<summary>Show Answer</summary>

**Answer:** A — `"12"`, `"33"`, `"123"`

**Explanation:** `+` evaluates left-to-right. `1 + "2"`: number meets string, so `1` is coerced to `"1"` and they concatenate to `"12"`. `1 + 2 + "3"`: `1 + 2` happens first (both numbers) giving `3`, then `3 + "3"` concatenates to `"33"`. `"1" + 2 + 3`: `"1" + 2` concatenates to `"12"` first (left-to-right), then `"12" + 3` concatenates to `"123"` — note it does **not** add `2 + 3` first, because `+` has no special "do the numbers first" rule; it strictly evaluates left to right. This ordering trap is a frequent source of bugs when mixing types.

</details>
::

::question-wrapper{language="javascript"}
### Q10. What happens when you access a property on `undefined`, versus declaring a variable with no initializer?

::code-wrapper{language="javascript"}
```javascript
let x;
console.log(x);
console.log(x.length);
```
::

- [ ] Logs `undefined`, then logs `0`
- [ ] Logs `undefined`, then throws `TypeError: Cannot read properties of undefined (reading 'length')`
- [ ] Logs `null`, then throws `ReferenceError`
- [ ] Both lines throw, because `x` is never assigned

<details>
<summary>Show Answer</summary>

**Answer:** B — Logs `undefined`, then throws a `TypeError`

**Explanation:** A `let`/`var` declared without an initializer is automatically set to `undefined` (not an error to read it, unlike TDZ). But `undefined` is a primitive with no properties, so attempting to read `.length` off it throws `TypeError`. **Debug:** this exact error is one of the most common in production JS, typically from an object that was expected to exist (e.g., an API response field) but didn't. The fix is optional chaining (`x?.length`) or an existence check before access. Option A wrongly assumes property access on `undefined` is safe.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What is the result of `typeof` applied to a function?

::code-wrapper{language="javascript"}
```javascript
function greet() {}
console.log(typeof greet);
console.log(greet instanceof Object);
```
::

- [ ] `"object"` and `true`
- [ ] `"function"` and `true`
- [ ] `"function"` and `false`
- [ ] `"object"` and `false`

<details>
<summary>Show Answer</summary>

**Answer:** B — `"function"` and `true`

**Explanation:** JavaScript special-cases `typeof` for callable objects, returning `"function"` even though functions are technically a subtype of `object` under the hood — which is why `greet instanceof Object` is also `true`. This dual nature surprises people who expect `typeof` categories and the prototype chain to be mutually exclusive. Option A misses that `typeof` has a dedicated `"function"` result. Options C and D get the `instanceof` half wrong — functions genuinely do inherit from `Object.prototype` via `Function.prototype`.

</details>
::

::question-wrapper{language="javascript"}
### Q12. What does `0.1 + 0.2 === 0.3` evaluate to, and why?

::code-wrapper{language="javascript"}
```javascript
console.log(0.1 + 0.2 === 0.3);
console.log(0.1 + 0.2);
```
::

- [ ] `true`; JavaScript numbers are always exact
- [ ] `false`; `0.1 + 0.2` actually equals `0.30000000000000004` due to IEEE-754 double-precision floating-point representation, which cannot represent most decimal fractions exactly
- [ ] `false`; this is a bug specific to V8 that other engines don't have
- [ ] `true`; JavaScript rounds floating point arithmetic automatically for `===`

<details>
<summary>Show Answer</summary>

**Answer:** B — `false`; the sum is `0.30000000000000004` due to IEEE-754 binary floating-point representation

**Explanation:** **Portability:** JavaScript's `number` type uses IEEE-754 double-precision floats (as do essentially all mainstream languages using this format — Python, Java, C, etc. exhibit the identical rounding, so this is not V8-specific). Fractions like `0.1` and `0.2` cannot be represented exactly in binary floating point, so tiny representation errors accumulate. The correct way to compare is checking the difference is within an epsilon (e.g., `Math.abs(a - b) < Number.EPSILON`), not `===`. Option A and D are false — there is no automatic rounding. Option C is a plausible-sounding but wrong claim of engine-specific behavior.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What is the type and value of an empty `const` declaration attempt?

::code-wrapper{language="javascript"}
```javascript
const PI;
```
::

- [ ] Valid; `PI` is `undefined` until assigned later
- [ ] `SyntaxError: Missing initializer in const declaration` — `const` requires an initializer at declaration time
- [ ] Valid; `PI` defaults to `0`
- [ ] Valid, but `PI` enters a permanent TDZ

<details>
<summary>Show Answer</summary>

**Answer:** B — `SyntaxError`; `const` requires an initializer

**Explanation:** Unlike `let` and `var`, `const` bindings must be initialized in the same statement they're declared — there is no such thing as "declare now, assign later" for `const`, because a `const` binding can never be reassigned at all. Option A confuses `const` with `let`'s allowed no-initializer form. Options C and D invent behavior that doesn't exist — this is a parse-time error, so the code never runs.

</details>
::

::question-wrapper{language="javascript"}
### Q14. Does `const` make an object's contents immutable?

::code-wrapper{language="javascript"}
```javascript
const user = { name: "Ana" };
user.name = "Beto";
user = { name: "Carla" };
```
::

- [ ] Both lines work fine — `const` prevents any mutation
- [ ] `user.name = "Beto"` works (mutating properties is allowed); `user = {...}` throws `TypeError: Assignment to constant variable` (reassigning the binding is not allowed)
- [ ] Both lines throw `TypeError`
- [ ] `user.name = "Beto"` throws, but reassigning `user` works

<details>
<summary>Show Answer</summary>

**Answer:** B — Mutating a property works; reassigning the binding throws

**Explanation:** `const` only freezes the *binding* (the variable name cannot be pointed at a new value) — it says nothing about the mutability of the value itself. Since `user` still refers to the same object, mutating its properties is completely legal. **Idiom:** to actually prevent mutation of the object's own properties, use `Object.freeze(user)` (shallow) in addition to `const`. Option A is the classic misconception that `const` means "constant/immutable data." Option D reverses the true behavior.

</details>
::

::question-wrapper
### Q15. Which is the idiomatic default choice for variable declarations in modern JavaScript, and why?

- [ ] Always use `var`, since it has the widest historical browser support
- [ ] Default to `const` for bindings that are never reassigned, use `let` only when reassignment is genuinely needed, and avoid `var` — this communicates intent and avoids `var`'s function-scoping/hoisting pitfalls
- [ ] Always use `let` everywhere for consistency, and reserve `const` only for primitive values
- [ ] It doesn't matter; all three are functionally interchangeable in modern engines

<details>
<summary>Show Answer</summary>

**Answer:** B — Default to `const`, use `let` only when reassignment is needed, avoid `var`

**Explanation:** **Idiom:** preferring `const` signals to readers (and tooling/linters) that a binding won't be reassigned, catching accidental reassignment bugs at parse/lint time; `let`'s block scoping avoids the classic `var` closure-in-loop bug (see Control Flow topic); `var`'s hoisting and function-scoping are considered legacy footguns in modern style guides (Airbnb, StandardJS, etc.) with no upside over `let`/`const`. Option A is outdated advice from an era before ES2015 adoption. Option C undersells `const`'s usefulness for objects/arrays too (the binding, not contents, is what's fixed). Option D ignores real, observable scoping differences.

</details>
::

::question-wrapper{language="javascript"}
### Q16. What is the best-practice way to check if a value is `NaN`?

::code-wrapper{language="javascript"}
```javascript
function isInvalidNumber(val) {
  return val !== val;
}
```
::

- [ ] Use the global `isNaN(val)` function, since it's the shortest and most common option
- [ ] Use `Number.isNaN(val)`, because the global `isNaN()` first coerces its argument, causing false positives like `isNaN("hello")` being `true` when the value isn't even numeric; `Number.isNaN` only returns `true` for the actual `NaN` value with no coercion
- [ ] Use `val === NaN`, since `NaN` is a well-defined constant
- [ ] The `val !== val` trick shown above is the only reliable method; all built-ins are broken

<details>
<summary>Show Answer</summary>

**Answer:** B — Use `Number.isNaN(val)` to avoid the global function's coercion-related false positives

**Explanation:** **Idiom:** the legacy global `isNaN()` coerces its argument to a number first, so `isNaN("hello")` is `true` (because `Number("hello")` is `NaN`) even though `"hello"` was never intended as a numeric check target — a classic false positive. `Number.isNaN()` (ES2015+) does no coercion, returning `true` only if the value literally *is* `NaN`. Option C is invalid — as established, `NaN === NaN` is always `false`, so this check can never succeed. Option D is an overstatement — the `val !== val` trick does work (it exploits the same self-inequality), but `Number.isNaN` is the clearer, idiomatic choice; it's not the *only* reliable method.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What is the idiomatic way to handle very large integers that exceed `Number.MAX_SAFE_INTEGER`?

::code-wrapper{language="javascript"}
```javascript
console.log(Number.MAX_SAFE_INTEGER);
console.log(Number.MAX_SAFE_INTEGER + 1 === Number.MAX_SAFE_INTEGER + 2);
```
::

- [ ] Continue using regular numbers; JavaScript numbers have unlimited precision
- [ ] Use the `bigint` primitive type (e.g., `9007199254740993n`), which represents arbitrary-precision integers exactly, instead of the `number` type, which silently loses precision beyond `2^53 - 1`
- [ ] Round the result using `Math.round()` to fix precision loss
- [ ] Store the number as a string and use `parseInt()` whenever arithmetic is needed

<details>
<summary>Show Answer</summary>

**Answer:** B — Use `bigint` for exact arbitrary-precision integers beyond `Number.MAX_SAFE_INTEGER`

**Explanation:** **Safety:** regular `number` values are IEEE-754 doubles that can only represent integers exactly up to `2^53 - 1` (`Number.MAX_SAFE_INTEGER`); beyond that, distinct integers can silently collapse to the same floating-point value (as the `true` result of the comparison above demonstrates) — a silent correctness bug, not an error you'd notice. `bigint` (suffix `n`, ES2020) stores arbitrary-precision integers with no such ceiling, at the cost of not being directly mixable with `number` in arithmetic (`1n + 1` throws `TypeError`). Option A is false — this is precisely the bug being demonstrated. Option C doesn't address the root precision-loss cause. Option D works but is a clunky workaround compared to a proper primitive type.

</details>
::

::question-wrapper{language="javascript"}
### Q18. Which best describes idiomatic use of `Symbol`?

::code-wrapper{language="javascript"}
```javascript
const id = Symbol("userId");
const obj = { [id]: 42, name: "Ana" };
```
::

- [ ] Symbols are mainly used to create guaranteed-unique property keys (e.g., to avoid accidental collisions with string keys or to define "hidden," non-enumerable-by-default-iteration metadata), not as a general-purpose string replacement
- [ ] Symbols are just a stylistic alternative to strings for any object key, interchangeable with strings in all cases
- [ ] `Symbol("userId") === Symbol("userId")` is `true`, so symbols are good for deduplication
- [ ] Symbols can be implicitly converted to strings via `+`, making them convenient for string concatenation

<details>
<summary>Show Answer</summary>

**Answer:** A — Symbols create guaranteed-unique keys, not a general string replacement

**Explanation:** Every `Symbol()` call produces a value unique from every other symbol, even with the same description string — so `Symbol("userId") === Symbol("userId")` is actually `false` (making option C the tempting-but-wrong trap). Symbol-keyed properties don't show up in `for...in`, `Object.keys()`, or `JSON.stringify()` by default, making them useful for "semi-private" metadata that won't collide with user-defined string keys. Option B ignores this uniqueness/interop purpose. Option D is false — `Symbol` values throw `TypeError` when used with implicit string coercion (e.g., `` `${sym}` `` and `sym + ""` both throw); you must call `.toString()` or `.description` explicitly.

</details>
::

::question-wrapper{language="javascript"}
### Q19. Why does this equality check behave unexpectedly, and what's the idiomatic fix?

::code-wrapper{language="javascript"}
```javascript
function isEmpty(value) {
  return value == null;
}
console.log(isEmpty(undefined));
console.log(isEmpty(null));
console.log(isEmpty(0));
```
::

- [ ] `value == null` is a bug; it should always be written as `value === null` for safety
- [ ] `== null` is actually an idiomatic, intentional pattern: due to type coercion rules, `null == undefined` is `true` (and only to each other), so `value == null` cleanly catches both `null` and `undefined` in one check without matching falsy-but-defined values like `0`
- [ ] This throws a `TypeError` because `null` cannot be compared with `==`
- [ ] `isEmpty(0)` returns `true` because `0` is falsy

<details>
<summary>Show Answer</summary>

**Answer:** B — `== null` intentionally catches both `null` and `undefined` via a special-cased coercion rule, without matching other falsy values

**Explanation:** **Idiom:** per the spec, `==` has a special rule that `null == undefined` is `true`, but `null` and `undefined` are `==` to nothing else (not `0`, not `""`, not `false`). This makes `value == null` a widely-used, deliberate idiom to check for "nullish" in one line, predating `??`/`?.`. So `isEmpty(undefined)` → `true`, `isEmpty(null)` → `true`, `isEmpty(0)` → `false` (0 is not loosely equal to null). Option A is overly cautious — this is one of the few places `==` is considered acceptable/idiomatic rather than a footgun. Option D wrongly assumes truthiness is involved; this comparison uses the `==` coercion table, not `Boolean()` coercion.

</details>
::

::question-wrapper{language="javascript"}
### Q20. What is the best-practice explanation for why `typeof` is safe to use on a variable that might not be declared at all, but not on one known to be in the TDZ?

::code-wrapper{language="javascript"}
```javascript
console.log(typeof undeclaredVar);

console.log(typeof laterVar);
let laterVar = 5;
```
::

- [ ] Both lines behave identically, logging `"undefined"` in both cases
- [ ] `typeof undeclaredVar` safely returns `"undefined"` for a name that was never declared anywhere (a historic idiom for feature-detection); but `typeof laterVar` throws `ReferenceError` because `laterVar` is `let`-declared later in scope, so referencing it here falls inside its Temporal Dead Zone even under `typeof`
- [ ] Both lines throw `ReferenceError`
- [ ] `typeof` never throws under any circumstances, by design, for any identifier

<details>
<summary>Show Answer</summary>

**Answer:** B — `typeof` on a truly undeclared name is safe; but `typeof` on a `let`/`const` name still in its TDZ throws

**Explanation:** **Idiom:** `typeof someGlobalThatMightNotExist` is a long-standing, safe idiom (e.g., `typeof window !== "undefined"` to detect a browser environment) specifically because `typeof` was designed not to throw for names that were never declared. However, this safety does *not* extend to TDZ: if the identifier *is* declared later in the same scope via `let`/`const`, the engine already knows about it and considers it "temporally dead" until its declaration line, so even the normally-safe `typeof` throws. Option D is the common but incorrect belief that `typeof` is unconditionally throw-proof.

</details>
::
