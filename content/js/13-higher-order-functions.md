# 13 — Higher-Order Functions

::question-wrapper{language="javascript"}
### Q1. What does this log?

::code-wrapper{language="javascript"}
```javascript
function applyDiscount(prices, discountFn) {
  return prices.map(discountFn);
}

const withTax = applyDiscount([100, 200], p => p * 1.1);
console.log(withTax);
```
::

- [ ] [110.00000000000001, 220.00000000000002]
- [ ] [110, 220]
- [ ] Throws a TypeError because `discountFn` isn't a built-in method
- [ ] [100, 200]

<details>
<summary>Show Answer</summary>

**Answer:** A — [110.00000000000001, 220.00000000000002]

**Explanation:** `applyDiscount` is a **higher-order function** — it accepts another function as an argument and delegates the transformation to it via `map`. The math itself hits a classic floating-point trap: `100 * 1.1` and `200 * 1.1` can't be represented exactly in IEEE-754 double precision, so the results carry tiny rounding error instead of the clean `110`/`220` you'd expect. **Debug:** this is why money math is usually done in integer cents or with a decimal library, not floating-point multiplication.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does this log?

::code-wrapper{language="javascript"}
```javascript
function multiplyBy(factor) {
  return function (n) {
    return n * factor;
  };
}

const double = multiplyBy(2);
console.log(double(5));
```
::

- [ ] 10
- [ ] Throws a TypeError because `multiplyBy` doesn't return a callable value
- [ ] NaN
- [ ] 7 (5 + 2)

<details>
<summary>Show Answer</summary>

**Answer:** A — 10

**Explanation:** `multiplyBy` is a factory: it returns a new function that closes over `factor`. `double` is that returned function with `factor` fixed at `2`, so calling `double(5)` computes `5 * 2 = 10`. This "function returning a function" shape is the foundation both of closures and of currying, covered later in this quiz.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What does this log?

::code-wrapper{language="javascript"}
```javascript
const names = ['ana', 'bob'];
console.log(names.map(n => n.toUpperCase()));
```
::

- [ ] ['ANA', 'BOB']
- [ ] ['ana', 'bob']
- [ ] 'ANA,BOB'
- [ ] Throws a TypeError

<details>
<summary>Show Answer</summary>

**Answer:** A — ['ANA', 'BOB']

**Explanation:** `map` creates a **new array** by applying the callback to every element and collecting the return values — it never mutates `names`. Each string's `.toUpperCase()` produces a new uppercase string, and those are collected in order into the result array.

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does this log?

::code-wrapper{language="javascript"}
```javascript
const nums = [1, 2, 3, 4, 5];
console.log(nums.filter(n => n > 3));
```
::

- [ ] [4, 5]
- [ ] [1, 2, 3]
- [ ] true
- [ ] [false, false, false, true, true]

<details>
<summary>Show Answer</summary>

**Answer:** A — [4, 5]

**Explanation:** `filter` keeps only the elements for which the predicate returns a truthy value, returning a new array of just those elements — not the boolean results themselves (that would be `map`'s job). `4 > 3` and `5 > 3` are the only true cases.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does this log?

::code-wrapper{language="javascript"}
```javascript
const cart = [{ price: 10 }, { price: 20 }, { price: 5 }];
console.log(cart.reduce((sum, item) => sum + item.price, 0));
```
::

- [ ] 35
- [ ] 0
- [ ] NaN
- [ ] [10, 20, 5]

<details>
<summary>Show Answer</summary>

**Answer:** A — 35

**Explanation:** `reduce` folds the array down to a single value by repeatedly applying the callback, carrying an accumulator (`sum`, starting at the initial value `0`) forward. `0 + 10 = 10`, `10 + 20 = 30`, `30 + 5 = 35` — the final accumulator is what's returned.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What does this log?

::code-wrapper{language="javascript"}
```javascript
const orders = [
  { amount: 100, status: 'paid' },
  { amount: 50, status: 'pending' },
  { amount: 200, status: 'paid' }
];

const total = orders
  .filter(o => o.status === 'paid')
  .map(o => o.amount)
  .reduce((a, b) => a + b, 0);

console.log(total);
```
::

- [ ] 300
- [ ] 350
- [ ] 250
- [ ] Throws a TypeError because you can't chain `.filter`, `.map`, and `.reduce` together

<details>
<summary>Show Answer</summary>

**Answer:** A — 300

**Explanation:** This is a standard **map/filter/reduce composition** pipeline: `filter` keeps only the two `'paid'` orders (`100` and `200`), `map` projects each down to just its `amount`, and `reduce` sums those amounts to `300`. Each method returns a new array (or value), which is exactly what makes chaining them together work.

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does this log?

::code-wrapper{language="javascript"}
```javascript
const add = a => b => a + b;
console.log(add(3)(4));
```
::

- [ ] 7
- [ ] Throws a TypeError because `add(3)` isn't callable
- [ ] "34" (string concatenation)
- [ ] undefined

<details>
<summary>Show Answer</summary>

**Answer:** A — 7

**Explanation:** This is basic **currying**: `add(3)` returns a new arrow function `b => 3 + b` (with `a` fixed at `3` via closure), and calling that with `4` computes `3 + 4 = 7`. Both `a` and `b` are numbers, so there's no string coercion involved.

</details>
::

::question-wrapper{language="javascript"}
### Q8. What does this famously log?

::code-wrapper{language="javascript"}
```javascript
console.log(['1', '2', '3'].map(parseInt));
```
::

- [ ] [1, NaN, NaN]
- [ ] [1, 2, 3]
- [ ] [1, 2, NaN]
- [ ] Throws a TypeError

<details>
<summary>Show Answer</summary>

**Answer:** A — [1, NaN, NaN]

**Explanation:** **Debug:** this is one of JavaScript's most famous "wat" moments. `map` invokes its callback with **three** arguments — `(element, index, array)` — and `parseInt(string, radix)` treats that second argument as a radix. So it actually runs `parseInt('1', 0, [...])` (radix `0` means "auto-detect," which defaults to base 10, giving `1`), `parseInt('2', 1, [...])` (radix `1` is invalid, giving `NaN`), and `parseInt('3', 2, [...])` (base 2 can't contain the digit `'3'`, giving `NaN`). The fix is `['1','2','3'].map(s => parseInt(s, 10))` or `.map(Number)`.

</details>
::

::question-wrapper{language="javascript"}
### Q9. What happens here?

::code-wrapper{language="javascript"}
```javascript
console.log([].reduce((a, b) => a + b));
```
::

- [ ] Throws `TypeError: Reduce of empty array with no initial value`
- [ ] Logs `0`
- [ ] Logs `undefined`
- [ ] Logs `NaN`

<details>
<summary>Show Answer</summary>

**Answer:** A — Throws `TypeError: Reduce of empty array with no initial value`

**Explanation:** Without an initial value, `reduce` tries to use the array's first element as the starting accumulator — but an empty array has no first element, and there's nothing meaningful to return, so it throws instead of silently producing `0` or `undefined`. The fix (and the general best practice — see Q10) is to always supply an explicit initial value.

</details>
::

::question-wrapper{language="javascript"}
### Q10. What does this log?

::code-wrapper{language="javascript"}
```javascript
console.log([].reduce((a, b) => a + b, 100));
```
::

- [ ] 100
- [ ] Throws a TypeError, same as with no initial value
- [ ] NaN
- [ ] undefined

<details>
<summary>Show Answer</summary>

**Answer:** A — 100

**Explanation:** With an initial value supplied, `reduce` on an empty array simply returns that initial value **without ever invoking the callback** — there are no elements to combine it with. This is the safe, predictable behavior that Q9's version lacks, and it's why always passing an initial value is the recommended default.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What does this log?

::code-wrapper{language="javascript"}
```javascript
function findFirstEven(arr) {
  let result;
  arr.forEach(n => {
    if (n % 2 === 0) {
      result = n;
      return;
    }
  });
  return result;
}

console.log(findFirstEven([1, 3, 5, 4, 6, 8]));
```
::

- [ ] 8
- [ ] 4
- [ ] undefined
- [ ] [4, 6, 8]

<details>
<summary>Show Answer</summary>

**Answer:** A — 8

**Explanation:** **Debug:** the `return` inside the `forEach` callback only exits that single invocation of the callback — it does **not** break out of the loop, because `forEach` provides no mechanism for early termination at all. So the callback keeps running for every element, and `result` gets overwritten each time an even number is found (`4`, then `6`, then `8`), ending on the *last* even number rather than the first. The correct fix for "stop at the first match" is `arr.find(n => n % 2 === 0)`, or a `for...of` loop with a real `break`.

</details>
::

::question-wrapper{language="javascript"}
### Q12. What does this log?

::code-wrapper{language="javascript"}
```javascript
const scores = [10, 1, 2, 21];
console.log(scores.sort());
```
::

- [ ] [1, 10, 2, 21]
- [ ] [1, 2, 10, 21]
- [ ] [21, 10, 2, 1]
- [ ] [10, 1, 2, 21] (unchanged)

<details>
<summary>Show Answer</summary>

**Answer:** A — [1, 10, 2, 21]

**Explanation:** **Debug:** `Array.prototype.sort()` with no comparator converts every element to a **string** and compares them lexicographically (by UTF-16 code unit), not numerically. `'10'` sorts before `'2'` because `'1' < '2'` as characters, even though `10 > 2` numerically. The fix is always to pass an explicit comparator for numbers: `scores.sort((a, b) => a - b)`.

</details>
::

::question-wrapper{language="javascript"}
### Q13. Why does `.map(Number)` behave correctly while `.map(parseInt)` (Q8) does not?

::code-wrapper{language="javascript"}
```javascript
console.log(['1', '2', '3'].map(Number));
```
::

- [ ] `Number(value)` only ever looks at its first argument, so the extra `index`/`array` arguments `map` passes are harmless
- [ ] `map` special-cases `Number` and only passes it one argument
- [ ] `Number` and `parseInt` behave identically, so this also produces `[1, NaN, NaN]`
- [ ] `Number` mutates the original array, so this is actually a different kind of bug

<details>
<summary>Show Answer</summary>

**Answer:** A — `Number(value)` only ever looks at its first argument, so the extra `index`/`array` arguments `map` passes are harmless

**Explanation:** **Idiom:** `map` always calls its callback as `callback(element, index, array)` regardless of which function you pass — the difference is entirely in how the callback *reacts* to those extra arguments. `Number(v)` ignores anything beyond the first parameter, so `[1,2,3].map(Number)` is safe and gives `[1, 2, 3]`. `parseInt(string, radix)`, uniquely among common conversion functions, *does* use its second parameter, which is what turns `map`'s index argument into an accidental radix in Q8.

</details>
::

::question-wrapper{language="javascript"}
### Q14. What happens here?

::code-wrapper{language="javascript"}
```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn(...args);
    return (...more) => curried(...args, ...more);
  };
}

function greet(greeting, name = 'friend') {
  return `${greeting}, ${name}!`;
}

const curriedGreet = curry(greet);
console.log(curriedGreet('Hi')('Sam'));
```
::

- [ ] Throws a TypeError because `curriedGreet('Hi')` returns a string, and strings aren't callable
- [ ] "Hi, Sam!"
- [ ] "Hi, friend!"
- [ ] "Hi, undefined!"

<details>
<summary>Show Answer</summary>

**Answer:** A — Throws a TypeError because `curriedGreet('Hi')` returns a string, and strings aren't callable

**Explanation:** **Debug:** `Function.prototype.length` only counts parameters **before** the first one with a default value (or a rest parameter) — so `greet.length` is `1`, not `2`, even though `greet` accepts two arguments. `curry`'s arity check (`args.length >= fn.length`) is satisfied the moment `curriedGreet('Hi')` is called with just one argument, so it immediately invokes `greet('Hi')`, producing the string `"Hi, friend!"` — not another curried function. Calling that string with `('Sam')` then throws, because a string isn't a function. This is why generic `fn.length`-based curry helpers are fragile against default and rest parameters.

</details>
::

::question-wrapper{language="javascript"}
### Q15. Which style is generally preferred, and why?

::code-wrapper{language="javascript"}
```javascript
// Style A
let result = [];
items.forEach(item => {
  if (item.active) result.push(transform(item));
});

// Style B
const result2 = items.filter(item => item.active).map(transform);
```
::

- [ ] Style B — it's declarative, avoids relying on an external mutable accumulator, and makes the transformation's intent explicit
- [ ] Style A — `forEach` is always meaningfully faster, so it should be preferred
- [ ] They're equally idiomatic; there's no practical difference in real codebases
- [ ] Style B is invalid — you can't chain `.filter` directly into `.map`

<details>
<summary>Show Answer</summary>

**Answer:** A — Style B — it's declarative, avoids relying on an external mutable accumulator, and makes the transformation's intent explicit

**Explanation:** **Idiom:** Style A depends on an externally-declared `let result = []` that the callback mutates as a side effect — easy to accidentally push twice, forget to initialize, or shadow in a refactor. Style B expresses the same operation as a pipeline of pure transformations with no external mutable state to get wrong. (For very large arrays where the double pass — one for `filter`, one for `map` — actually matters, a single `reduce` can combine both into one pass, but for typical sizes, readability wins and the filter/map chain is the standard idiom.)

</details>
::

::question-wrapper{language="javascript"}
### Q16. What's the problem with this code?

::code-wrapper{language="javascript"}
```javascript
const total = orders
  .map(o => {
    o.processed = true;
    return o.amount;
  })
  .reduce((a, b) => a + b, 0);
```
::

- [ ] The `map` callback has a hidden side effect — mutating the original `orders` objects — when `map` is expected to just transform and return values
- [ ] This throws a TypeError because `map` callbacks can't have a function body with multiple statements
- [ ] `map` automatically deep-clones each element, so `o.processed = true` has no effect on the original array
- [ ] There's no problem; mutating inside `map` is the standard way to flag processed items

<details>
<summary>Show Answer</summary>

**Answer:** A — The `map` callback has a hidden side effect — mutating the original `orders` objects — when `map` is expected to just transform and return values

**Explanation:** **Idiom:** `map`'s contract is "produce a new value per element," not "do something to the element." Mutating `o.processed` here is easy to miss when reading the code (you have to read past the `return` to notice it), and it silently changes the original `orders` array/objects, which can cause bugs if that same array is read elsewhere, memoized, or the pipeline is accidentally run twice. Best practice is to keep `map`/`filter` callbacks pure and do side-effecting work in a separate, clearly-named step (e.g. a `forEach` or explicit loop).

</details>
::

::question-wrapper{language="javascript"}
### Q17. Which is the better choice for finding a single matching user in a large array, and why?

::code-wrapper{language="javascript"}
```javascript
// Option A
const admin = users.filter(u => u.role === 'admin')[0];

// Option B
const admin2 = users.find(u => u.role === 'admin');
```
::

- [ ] Option B — `find` stops as soon as it finds a match, while `filter` always scans the whole array and allocates an array just to discard everything but index `0`
- [ ] Option A — `filter` is guaranteed to be optimized by the engine for single-element access
- [ ] They perform identically since both are O(n) in the worst case
- [ ] Option B, but only because `filter(...)​[0]` is a syntax error

<details>
<summary>Show Answer</summary>

**Answer:** A — Option B — `find` stops as soon as it finds a match, while `filter` always scans the whole array and allocates an array just to discard everything but index `0`

**Explanation:** **Performance:** `find` short-circuits and returns the first matching element immediately, while `filter` always iterates the entire array (even after finding a match) and builds a brand-new array — wasted work and a wasted allocation when all you wanted was one item. Both are technically O(n) worst case (no match found), but `find`'s best/average case is much better, and it never allocates. Prefer `find` whenever you only need one result.

</details>
::

::question-wrapper{language="javascript"}
### Q18. What does this log?

::code-wrapper{language="javascript"}
```javascript
const pipe = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);

const process = pipe(
  s => s.trim(),
  s => s.toLowerCase(),
  s => s.replace(/\s+/g, '-')
);

console.log(process('  Hello World  '));
```
::

- [ ] 'hello-world'
- [ ] 'Hello-World'
- [ ] '-hello-world-'
- [ ] Throws a TypeError because `pipe` can't accept a variable number of functions

<details>
<summary>Show Answer</summary>

**Answer:** A — 'hello-world'

**Explanation:** **Idiom:** `pipe` is a standard function-composition utility built with `reduce`: it threads the initial value `x` through each function in `fns`, left to right, using each result as the input to the next. `'  Hello World  '` gets trimmed to `'Hello World'`, lowercased to `'hello world'`, then has its whitespace replaced with a dash, giving `'hello-world'`. This `reduce`-over-functions pattern is a common, idiomatic way to build reusable pipelines instead of nesting calls manually (`f(g(h(x)))`).

</details>
::

::question-wrapper{language="javascript"}
### Q19. What's the best-practice benefit of writing `createLogger` this way?

::code-wrapper{language="javascript"}
```javascript
const createLogger = level => message => console.log(`[${level}] ${message}`);

const logError = createLogger('ERROR');
const logInfo = createLogger('INFO');

logError('Disk full');
logInfo('Server started');
```
::

- [ ] It lets you "lock in" the `level` argument once via partial application, producing specialized, reusable functions instead of repeating the level at every call site
- [ ] It's purely stylistic — passing `level` as a normal second argument to a single `log(level, message)` function would be identical in every way
- [ ] Currying here makes the logger run faster than a two-argument function would
- [ ] It's required — `console.log` cannot be called with a template literal directly

<details>
<summary>Show Answer</summary>

**Answer:** A — It lets you "lock in" the `level` argument once via partial application, producing specialized, reusable functions instead of repeating the level at every call site

**Explanation:** **Idiom:** this is a textbook, practical use of currying: `level` is a configuration-like argument that's known up front and reused many times, while `message` varies per call. Currying `createLogger` lets you produce `logError`/`logInfo` once and then call them like ordinary single-argument functions everywhere else, instead of writing `log('ERROR', ...)` at every call site and risking a copy-pasted wrong level string.

</details>
::

::question-wrapper{language="javascript"}
### Q20. Is deep currying like this a good practice when all five arguments are always supplied together at one call site?

::code-wrapper{language="javascript"}
```javascript
const add5 = curry(a => b => c => d => e => a + b + c + d + e);
console.log(add5(1)(2)(3)(4)(5));
```
::

- [ ] No — currying pays off when arguments arrive separately over time (partial application); when they always arrive together, a plain `(a, b, c, d, e) => ...` function is clearer and easier to debug
- [ ] Yes — currying should always be preferred for functional purity, regardless of call pattern
- [ ] No — curried functions are fundamentally incapable of accepting more than one argument per call
- [ ] Yes — curried chains are always faster than an equivalent multi-parameter function

<details>
<summary>Show Answer</summary>

**Answer:** A — No — currying pays off when arguments arrive separately over time (partial application); when they always arrive together, a plain `(a, b, c, d, e) => ...` function is clearer and easier to debug

**Explanation:** **Idiom:** currying's real value (as in Q19) is enabling partial application — supplying some arguments now and the rest later, at a different place in the code. When every argument is always known and passed at the same call site, chaining five single-argument calls adds indirection, produces longer/harder-to-read stack traces, and gives no practical benefit over a single multi-parameter function. Good judgment means reaching for currying where it solves a real problem (config-then-use, building specialized functions), not applying it uniformly everywhere as a stylistic default.

</details>
::
