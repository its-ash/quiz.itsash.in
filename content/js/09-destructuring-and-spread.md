# 09 — Destructuring & Spread

::question-wrapper{language="javascript"}
### Q1. What does this log?

::code-wrapper{language="javascript"}
```javascript
const arr = [10, 20, 30];
const [x, y] = arr;
console.log(x, y);
```
::

- [ ] 10 20
- [ ] 10 20 30
- [ ] undefined undefined
- [ ] [10, 20]

<details>
<summary>Show Answer</summary>

**Answer:** A — 10 20

**Explanation:** Array destructuring binds by position, not by consuming the whole array. `x` takes `arr[0]`, `y` takes `arr[1]`, and `arr[2]` is simply left unbound — it still exists in `arr`, but no new variable captures it. B is wrong because destructuring only creates as many bindings as the pattern lists. C and D confuse destructuring with cloning or logging the array itself.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does this log?

::code-wrapper{language="javascript"}
```javascript
const user = { id: 7, name: 'Ada' };
const { name, id } = user;
console.log(name, id);
```
::

- [ ] undefined undefined
- [ ] 7 Ada
- [ ] Ada 7
- [ ] TypeError: Cannot destructure

<details>
<summary>Show Answer</summary>

**Answer:** C — Ada 7

**Explanation:** **Idiom.** Unlike array destructuring, object destructuring matches by *key name*, not position — writing `{ name, id }` instead of `{ id, name }` has zero effect on the result because each identifier looks itself up on `user`. B is the tempting wrong answer for anyone assuming destructuring mirrors the object's declared field order. A and D would only apply if the keys didn't exist or `user` were nullish.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What happens when this runs?

::code-wrapper{language="javascript"}
```javascript
const config = { url: 'https://api.example.com' };
const { url: endpoint } = config;
console.log(url);
```
::

- [ ] ReferenceError: url is not defined
- [ ] https://api.example.com
- [ ] undefined
- [ ] endpoint

<details>
<summary>Show Answer</summary>

**Answer:** A — ReferenceError: url is not defined

**Explanation:** **Debug.** Renaming with `{ url: endpoint }` means the value at `config.url` is bound *only* to the new identifier `endpoint` — the name `url` is never declared as a local binding. Logging `url` therefore hits a plain undeclared-variable error, not a "no value" case. B and C wrongly assume the original key name survives as a variable; D confuses the renamed variable's identifier with what was actually requested.

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does this log?

::code-wrapper{language="javascript"}
```javascript
function connect({ port = 8080, host = 'localhost', timeout = 3000 } = {}) {
  return `${host}:${port} (${timeout}ms)`;
}
console.log(connect({ port: null, host: undefined }));
```
::

- [ ] localhost:8080 (3000ms)
- [ ] null:8080 (3000ms)
- [ ] TypeError: Cannot read properties of null
- [ ] localhost:null (3000ms)

<details>
<summary>Show Answer</summary>

**Answer:** D — localhost:null (3000ms)

**Explanation:** **Debug.** Destructuring defaults trigger for exactly one value: `undefined`. `port: null` is an explicit, real value, so the default is skipped and `port` stays `null`. `host: undefined` *is* `undefined`, so its default `'localhost'` kicks in. `timeout` isn't passed at all, which is also `undefined` under the hood, so it defaults to `3000`. A wrongly assumes `null` behaves like a missing value; B mixes up which field falls back.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does this log?

::code-wrapper{language="javascript"}
```javascript
const response = {
  data: [{ id: 1, meta: { active: true } }],
};
const {
  data: [{ meta: { active } }],
} = response;
console.log(active);
```
::

- [ ] undefined
- [ ] true
- [ ] TypeError: Cannot destructure property 'meta' of undefined
- [ ] { active: true }

<details>
<summary>Show Answer</summary>

**Answer:** B — true

**Explanation:** Destructuring patterns can nest arbitrarily deep: `data` is renamed by pattern position into an array pattern, whose single element is object-destructured for `meta`, which is itself object-destructured for `active`. Each layer just walks one level deeper into the structure — since every intermediate value actually exists, no layer fails. C is the trap for anyone who assumes deep nesting is fragile by default; it only throws if an *intermediate* value were `null`/`undefined`, which isn't the case here.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What does this log?

::code-wrapper{language="javascript"}
```javascript
const rgb = [255, 165, 0];
const [, green] = rgb;
console.log(green);
```
::

- [ ] 165
- [ ] 255
- [ ] 0
- [ ] undefined

<details>
<summary>Show Answer</summary>

**Answer:** A — 165

**Explanation:** **Idiom.** A leading comma with nothing before it is a valid "elision" — it holds the position for `rgb[0]` without binding it to any name, so the next identifier, `green`, lands on `rgb[1]`. This is the standard idiom for skipping array elements you don't need. B would be the answer only if the elision were ignored entirely; D would be the answer if the pattern had too few slots for `rgb`.

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does this log?

::code-wrapper{language="javascript"}
```javascript
const scores = [95, 88, 72, 60];
const [top, ...rest] = scores;
console.log(rest.length, Array.isArray(rest));
```
::

- [ ] 4 true
- [ ] 3 false
- [ ] 3 true
- [ ] undefined true

<details>
<summary>Show Answer</summary>

**Answer:** C — 3 true

**Explanation:** **Idiom.** The rest pattern `...rest` collects every remaining element — here, three of the four — into a genuinely new `Array` instance, regardless of how many items it ends up holding (even zero). B is wrong because rest destructuring always produces a real array, never an array-like or plain object. A miscounts by including `top`'s element in `rest`.

</details>
::

::question-wrapper{language="javascript"}
### Q8. What does this log?

::code-wrapper{language="javascript"}
```javascript
const user = { id: 1, name: 'Kim', role: 'admin', active: true };
const { id, ...meta } = user;
console.log(meta);
```
::

- [ ] { id: 1, name: 'Kim', role: 'admin', active: true }
- [ ] {}
- [ ] TypeError: Rest element must be last
- [ ] { name: 'Kim', role: 'admin', active: true }

<details>
<summary>Show Answer</summary>

**Answer:** D — { name: 'Kim', role: 'admin', active: true }

**Explanation:** **Idiom.** Object rest works like array rest: it gathers every enumerable own property *not already claimed* by an earlier name in the pattern into a fresh plain object. `id` was explicitly pulled out, so it's excluded from `meta`. A is the mistake of assuming rest re-includes everything; C confuses the object-rest constraint (rest must be the *last* property in the pattern, which it correctly is here) with an actual error.

</details>
::

::question-wrapper
### Q9. In `const [a, ...b] = arr;` and `const c = [...arr, 99];`, what role does the `...` play in each statement, respectively?

- [ ] Rest syntax, then spread syntax
- [ ] Spread syntax, then rest syntax
- [ ] Rest syntax in both cases
- [ ] Spread syntax in both cases

<details>
<summary>Show Answer</summary>

**Answer:** A — Rest syntax, then spread syntax

**Explanation:** **Idiom.** The token `...` is overloaded: on the left side of a destructuring assignment it's the *rest* pattern, collecting leftover elements into a new binding; inside an array or object literal (or a function call) it's *spread*, expanding an existing iterable's elements into that new context. Same three dots, opposite direction of data flow — memorize it by which side of `=` (or which kind of expression) you're looking at, not by the symbol itself.

</details>
::

::question-wrapper{language="javascript"}
### Q10. What happens when this code runs?

::code-wrapper{language="javascript"}
```javascript
const a = { x: 1 };
const b = null;
const { x, y } = a;
const { z } = b;
```
::

- [ ] Both lines run fine; y is undefined, z is undefined
- [ ] Line 3 runs fine (y is undefined); line 4 throws TypeError because null cannot be destructured
- [ ] Both lines throw TypeError
- [ ] Line 3 throws ReferenceError; line 4 runs fine

<details>
<summary>Show Answer</summary>

**Answer:** B — Line 3 runs fine (y is undefined); line 4 throws TypeError because null cannot be destructured

**Explanation:** **Safety.** These two failure modes are easy to conflate but are opposites: destructuring an object for a key it doesn't have simply yields `undefined` for that binding (no error — `a` is a valid object, `y` just isn't one of its properties). Destructuring `null` (or `undefined`) itself is different — the engine must first coerce the right-hand side to an object to read properties off it, and that coercion step throws for nullish values. A wrongly treats both as the harmless "missing key" case.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What does this log?

::code-wrapper{language="javascript"}
```javascript
let a = 1, b = 2;
[a, b] = [b, a];
console.log(a, b);
```
::

- [ ] 1 2
- [ ] 1 1
- [ ] 2 1
- [ ] ReferenceError: Invalid destructuring assignment target

<details>
<summary>Show Answer</summary>

**Answer:** C — 2 1

**Explanation:** **Idiom.** The right-hand side `[b, a]` is fully evaluated first, building a temporary array `[2, 1]` from the *current* values of `b` and `a`; only then does array destructuring assign its elements back into `a` and `b` in order. This is the standard no-temp-variable swap idiom — no intermediate variable is needed because the RHS snapshot happens before any reassignment occurs. B is the mistake of assuming the assignments happen sequentially and interfere with each other.

</details>
::

::question-wrapper{language="javascript"}
### Q12. What happens when this runs?

::code-wrapper{language="javascript"}
```javascript
function createUser({ id, name }) {
  return `${id}-${name}`;
}
console.log(createUser());
```
::

- [ ] undefined-undefined
- [ ] TypeError: Cannot destructure property 'id' of 'undefined' as it is undefined
- [ ] ReferenceError: id is not defined
- [ ] 0-undefined

<details>
<summary>Show Answer</summary>

**Answer:** B — TypeError: Cannot destructure property 'id' of 'undefined' as it is undefined

**Explanation:** **Safety.** Calling `createUser()` with no argument means the parameter itself is `undefined` — and destructuring `undefined` throws for the same reason destructuring `null` does, before the function body ever runs. A is the tempting guess for anyone assuming missing parameters just cascade into `undefined` fields, but that only holds once you're safely destructuring a *real* object.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What happens when this runs?

::code-wrapper{language="javascript"}
```javascript
function createUser({ id, name } = {}) {
  return `${id}-${name}`;
}
console.log(createUser());
```
::

- [ ] TypeError: Cannot destructure property 'id' of 'undefined' as it is undefined
- [ ] {}
- [ ] 0-0
- [ ] undefined-undefined

<details>
<summary>Show Answer</summary>

**Answer:** D — undefined-undefined

**Explanation:** **Safety.** Adding `= {}` as the *parameter's own* default only fires when the argument itself is `undefined` — exactly the case of calling with no argument at all. That substitutes an empty object before destructuring runs, so `id` and `name` are destructured off `{}` and each comes out `undefined` (not an error) rather than the whole call throwing. This is the standard fix for the crash in the previous question; A describes what happens *without* the `= {}` guard.

</details>
::

::question-wrapper{language="javascript"}
### Q14. What does this log?

::code-wrapper{language="javascript"}
```javascript
function sum(a, b, c) {
  return a + b + c;
}
const nums = [1, 2, 3];
console.log(sum(...nums), [...nums, 4]);
```
::

- [ ] 6 [1, 2, 3, 4]
- [ ] [1, 2, 3] [1, 2, 3, 4]
- [ ] NaN [1, 2, 3, 4]
- [ ] 6 [4, 1, 2, 3]

<details>
<summary>Show Answer</summary>

**Answer:** A — 6 [1, 2, 3, 4]

**Explanation:** **Idiom.** The same `...nums` spread means different things purely by context: inside a call's argument list it unpacks the array into three separate positional arguments (`sum(1, 2, 3)` → `6`), while inside an array literal it unpacks the array's elements in place, so `[...nums, 4]` produces `[1, 2, 3, 4]` with the new element appended after. B assumes the call spread packs into a single array argument instead of unpacking it.

</details>
::

::question-wrapper{language="javascript"}
### Q15. What does this log?

::code-wrapper{language="javascript"}
```javascript
const original = { name: 'A', tags: ['x', 'y'] };
const copy = { ...original };
copy.tags.push('z');
console.log(original.tags);
```
::

- [ ] ['x', 'y']
- [ ] ['x', 'y', 'z']
- [ ] TypeError: Cannot push to a spread property
- [ ] undefined

<details>
<summary>Show Answer</summary>

**Answer:** B — ['x', 'y', 'z']

**Explanation:** **Safety.** Object spread is a *shallow* copy — it copies each top-level property's value, and for a property whose value is itself an object or array, that "value" is a reference. `copy.tags` and `original.tags` point at the exact same array in memory, so mutating one through `.push` is visible through the other. A is the mistake of assuming spread deep-clones; avoiding this bug requires an explicit deep clone (e.g. `structuredClone`) or spreading the nested array too.

</details>
::

::question-wrapper{language="javascript"}
### Q16. What does this log?

::code-wrapper{language="javascript"}
```javascript
const emoji = '👍A';
console.log(emoji.length, [...emoji].length);
```
::

- [ ] 2 2
- [ ] 3 3
- [ ] 3 2
- [ ] 2 3

<details>
<summary>Show Answer</summary>

**Answer:** C — 3 2

**Explanation:** **Portability.** `.length` counts UTF-16 *code units*: `👍` lies outside the Basic Multilingual Plane, so it's stored as a surrogate pair (2 code units), plus `'A'` (1 unit) gives `3`. Spreading a string, however, iterates by *Unicode code point* — the string iterator correctly recognizes the surrogate pair as one character — so `[...emoji]` yields `['👍', 'A']`, length `2`. This mismatch is why naive indexing (`str[i]`) can slice a surrogate pair in half while spread/`for...of` never do.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What does this log?

::code-wrapper{language="javascript"}
```javascript
const map = new Map([['a', 1], ['b', 2]]);
const obj = { a: 1, b: 2 };
console.log([...map].length);
console.log([...obj]);
```
::

- [ ] 2, then [{ a: 1 }, { b: 2 }]
- [ ] 2, then TypeError: obj is not iterable
- [ ] 4, then { a: 1, b: 2 }
- [ ] TypeError on both lines

<details>
<summary>Show Answer</summary>

**Answer:** B — 2, then TypeError: obj is not iterable

**Explanation:** **Debug.** `Map` implements the iterable protocol, so spreading it into an array yields its `[key, value]` entries — two entries, length `2`. A plain object does *not* implement `Symbol.iterator` at all; it only has enumerable own properties, which object spread (`{ ...obj }`) knows how to read but *array* spread (`[...obj]`) does not — array spread strictly requires an iterable, so it throws. This is the key distinction: object spread reads enumerable properties, array/call spread requires iterability.

</details>
::

::question-wrapper{language="javascript"}
### Q18. What does this log?

::code-wrapper{language="javascript"}
```javascript
const defaults = { theme: 'light', size: 'md' };
const overrides = { size: 'lg' };
const final = { ...defaults, size: 'sm', ...overrides };
console.log(final);
```
::

- [ ] { theme: 'light', size: 'sm' }
- [ ] { theme: 'light', size: 'md' }
- [ ] { theme: 'light', size: 'lg' }
- [ ] SyntaxError: Duplicate key 'size'

<details>
<summary>Show Answer</summary>

**Answer:** C — { theme: 'light', size: 'lg' }

**Explanation:** **Idiom.** Object literals allow duplicate keys, and later ones simply overwrite earlier ones in strict left-to-right source order — spread just expands into that same sequence of key writes. So `size` is written by `...defaults` ('md'), then overwritten by the literal `'sm'`, then overwritten again by `...overrides` ('lg'), which appears last and therefore wins. D is wrong because unlike duplicate `let` declarations, duplicate object keys are legal JavaScript, not a syntax error.

</details>
::

::question-wrapper{language="javascript"}
### Q19. What does this log?

::code-wrapper{language="javascript"}
```javascript
const key = 'role';
const user = { role: 'admin', id: 42 };
const { [key]: value } = user;
console.log(value);
```
::

- [ ] SyntaxError — computed keys cannot appear in a destructuring pattern
- [ ] undefined, because `key` refers to the variable name, not the object's property
- [ ] 'role' — the destructured value takes the name of the computed key by default
- [ ] 'admin' — computed property names are valid in destructuring, exactly as in object literals, but they require an explicit `: binding` (no shorthand form is allowed)

<details>
<summary>Show Answer</summary>

**Answer:** D — 'admin' — computed property names are valid in destructuring, exactly as in object literals, but they require an explicit `: binding` (no shorthand form is allowed)

**Explanation:** **Debug.** A common misconception is that computed keys only work in object *literals*, not on the left side of a destructuring pattern — but `{ [key]: value } = user` is valid ES6 syntax: it looks up `user[key]` (`user.role`, i.e. `'admin'`) and binds it to `value`. The real, genuine limitation is that destructuring has no shorthand form for computed keys — you cannot write `{ [key] }` and expect a binding named after the key's runtime value, since JavaScript has no way to turn a dynamic string into an identifier name; the `: binding` is mandatory. A states the actual misconception as fact; C confuses the computed key's name with its resolved value.

</details>
::

::question-wrapper{language="javascript"}
### Q20. A component factory function takes six parameters, several of which are optional and are frequently skipped or passed out of order by callers.

::code-wrapper{language="javascript"}
```javascript
function createWidget(id, name, color, size, isVisible, onClick) { /* ... */ }

function createWidget({ id, name, color = 'blue', size = 'md', isVisible = true, onClick } = {}) { /* ... */ }
```
::

Which signature is the better design, and why?

- [ ] The positional version, because positional parameters are always faster at runtime than destructured ones
- [ ] The positional version, because JavaScript requires optional parameters to appear positionally after required ones, which destructuring cannot express
- [ ] Neither — with six parameters you should always use an array and access by index
- [ ] The destructured version, because it lets callers name only the options they care about, skip optional ones freely, and self-documents each argument at the call site

<details>
<summary>Show Answer</summary>

**Answer:** D — The destructured version, because it lets callers name only the options they care about, skip optional ones freely, and self-documents each argument at the call site

**Explanation:** **Idiom.** Once a function has more than two or three parameters, especially with several optional ones, positional calls become error-prone (`createWidget(undefined, undefined, undefined, 'lg')`) and unreadable at the call site. Destructured parameters with defaults let callers pass an options object naming exactly what they need (`createWidget({ size: 'lg', onClick })`) in any order. A is a myth — the performance difference is negligible and irrelevant to API design; B is factually wrong, since destructuring patterns support per-field defaults regardless of order.

</details>
::
