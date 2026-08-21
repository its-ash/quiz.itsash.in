# 08 — Arrays & Array Methods

::question-wrapper{language="javascript"}
### Q1. What does the following log?

::code-wrapper{language="javascript"}
```javascript
function addItem(cart, item) {
  cart.push(item);
  return cart;
}

const original = ['apple'];
const updated = addItem(original, 'banana');

console.log(original === updated, original.length);
```
::

- [ ] true, 2
- [ ] false, 1
- [ ] true, 1
- [ ] false, 2

<details>
<summary>Show Answer</summary>

**Answer:** A — true, 2

**Explanation:** Arrays are passed by reference, so `cart` inside `addItem` refers to the very same array as `original`. `push` mutates that array in place and the function returns `cart`, meaning `updated` and `original` point to the identical object — hence `===` is `true` — and the length is now `2`. The distractors assume `push` either returns a fresh array or silently fails to mutate the shared reference, which is the opposite of how mutating array methods work.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does this log?

::code-wrapper{language="javascript"}
```javascript
const arr = [1, 2, 3];
console.log(typeof arr);
console.log(Array.isArray(arr));
```
::

- [ ] "array", true
- [ ] "object", true
- [ ] "object", false
- [ ] "array", false

<details>
<summary>Show Answer</summary>

**Answer:** B — "object", true

**Explanation:** JavaScript has no distinct `typeof` tag for arrays — they are a specialized kind of object, so `typeof arr` always returns `"object"`. `Array.isArray()` is the correct and reliable way to detect an array because it inspects the actual internal class of the value rather than the loose `typeof` result (it even works correctly across iframes/realms, where `instanceof Array` can fail). **Idiom:** always reach for `Array.isArray()`, never `typeof x === 'object'`, when you need to confirm a value is an array.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What does `.sort()` produce here?

::code-wrapper{language="javascript"}
```javascript
const scores = [10, 2, 1, 20];
scores.sort();
console.log(scores);
```
::

- [ ] [1, 2, 10, 20]
- [ ] [10, 2, 1, 20]
- [ ] [1, 10, 2, 20]
- [ ] [20, 10, 2, 1]

<details>
<summary>Show Answer</summary>

**Answer:** C — [1, 10, 2, 20]

**Explanation:** With no comparator, `.sort()` converts every element to a string and compares them by UTF-16 code unit, not numeric value: `"1" < "10" < "2" < "20"` lexicographically. **Debug:** this is the single most common `.sort()` bug — to get real numeric order you must pass a comparator, e.g. `scores.sort((a, b) => a - b)`. The tempting `[1, 2, 10, 20]` is what a numeric sort would give, not the default string sort.

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does this log?

::code-wrapper{language="javascript"}
```javascript
const nums = [3, 1, 2];
const result = nums.sort((a, b) => a - b);
console.log(result === nums, nums);
```
::

- [ ] false, [1, 2, 3]
- [ ] true, [3, 1, 2]
- [ ] false, [3, 1, 2]
- [ ] true, [1, 2, 3]

<details>
<summary>Show Answer</summary>

**Answer:** D — true, [1, 2, 3]

**Explanation:** `.sort()` reorders the array in place and returns a reference to that same array — it does not build a new one. So `result === nums` is `true`, and `nums` itself has been reordered to `[1, 2, 3]`. The `false` distractors wrongly assume `sort` behaves like `map`/`filter` and returns a new array; the `[3, 1, 2]` distractors wrongly assume the original stays untouched.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does this log?

::code-wrapper{language="javascript"}
```javascript
const a = [1, 2, 3];
const b = [1, 2, 3];
console.log(a === b, a == b, JSON.stringify(a) === JSON.stringify(b));
```
::

- [ ] false, false, true
- [ ] true, true, true
- [ ] false, false, false
- [ ] true, false, true

<details>
<summary>Show Answer</summary>

**Answer:** A — false, false, true

**Explanation:** Both `===` and `==` compare arrays (and objects in general) by reference, not by contents — since neither operand is a primitive, `==` doesn't attempt any coercion and falls straight to a reference check. `a` and `b` are two distinct array instances, so both comparisons are `false` even though the elements match. `JSON.stringify` turns each into the identical string `"[1,2,3]"`, so that comparison is `true`. **Debug:** this is why you never compare arrays with `===`/`==` for value equality — use a deep-equality check, or `JSON.stringify` for simple cases.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What happens when this runs?

::code-wrapper{language="javascript"}
```javascript
function sumAll(nums) {
  return nums.reduce((acc, n) => acc + n);
}

console.log(sumAll([5, 10, 15]));
console.log(sumAll([]));
```
::

- [ ] 30 then 0
- [ ] 30 then a TypeError is thrown
- [ ] 30 then undefined
- [ ] NaN then a TypeError is thrown

<details>
<summary>Show Answer</summary>

**Answer:** B — 30 then a TypeError is thrown

**Explanation:** Without an initial value, `.reduce()` uses the array's first element as the starting accumulator and begins iterating from index 1, so `sumAll([5, 10, 15])` correctly computes `30`. But an empty array has no first element to seed the accumulator and nothing to iterate, so `.reduce()` throws `TypeError: Reduce of empty array with no initial value` instead of silently returning `0` or `undefined`. **Safety:** always pass an explicit initial value — `nums.reduce((acc, n) => acc + n, 0)` — to make `reduce` safe on empty arrays.

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does this log?

::code-wrapper{language="javascript"}
```javascript
const values = [1, NaN, 3];
console.log(values.indexOf(NaN), values.includes(NaN));
```
::

- [ ] 1, true
- [ ] -1, false
- [ ] -1, true
- [ ] 1, false

<details>
<summary>Show Answer</summary>

**Answer:** C — -1, true

**Explanation:** `.indexOf()` compares elements using strict equality (`===`), and `NaN === NaN` is always `false` by IEEE-754 rules, so `.indexOf()` can never locate `NaN` and returns `-1`. `.includes()` uses the SameValueZero algorithm instead, which specifically treats `NaN` as equal to itself, so it correctly returns `true`. **Debug:** if you need to check for `NaN` membership, use `.includes()`, not `.indexOf()`.

</details>
::

::question-wrapper{language="javascript"}
### Q8. What does this log?

::code-wrapper{language="javascript"}
```javascript
const sparse = new Array(3);
let count = 0;
sparse.forEach(() => count++);

let loopCount = 0;
for (let i = 0; i < sparse.length; i++) {
  loopCount++;
}

console.log(count, loopCount);
```
::

- [ ] 3, 3
- [ ] 0, 0
- [ ] 3, 0
- [ ] 0, 3

<details>
<summary>Show Answer</summary>

**Answer:** D — 0, 3

**Explanation:** `new Array(3)` creates an array with `length` 3 but no actual elements — just empty "holes" with no own property at any index. `.forEach()` (like `.map()` and `.filter()`) only invokes its callback for indices that actually exist, so it skips every hole entirely, leaving `count` at `0`. A classic index-based `for` loop only checks `i < length`, with no awareness of holes, so it runs the full 3 iterations regardless. **Debug:** iteration methods and manual index loops disagree on sparse arrays — this is a frequent source of "why didn't my `.forEach()` run" bugs.

</details>
::

::question-wrapper{language="javascript"}
### Q9. What does this log?

::code-wrapper{language="javascript"}
```javascript
const arr = [1, 2, 3];
arr.length = 5;
console.log(arr.length, arr[3], arr[4]);
```
::

- [ ] 5, undefined, undefined
- [ ] 3, undefined, undefined
- [ ] 5, 0, 0
- [ ] 5, null, null

<details>
<summary>Show Answer</summary>

**Answer:** A — 5, undefined, undefined

**Explanation:** `.length` is a writable property; setting it larger than the current size extends the array with trailing empty slots (holes), and `.length` itself updates to reflect the new size (`5`). Reading a hole returns `undefined`, the same value you'd get from reading any nonexistent property — not `0` and not `null`, which are actual stored values, not "nothing is there" markers. **Debug:** growing `.length` manually is a common way to accidentally introduce holes into what looked like a dense array.

</details>
::

::question-wrapper{language="javascript"}
### Q10. What does this log?

::code-wrapper{language="javascript"}
```javascript
const arr = [1, 2, 3, 4, 5];
arr.length = 2;
console.log(arr);
```
::

- [ ] [1, 2, 3, 4, 5]
- [ ] [1, 2]
- [ ] [4, 5]
- [ ] []

<details>
<summary>Show Answer</summary>

**Answer:** B — [1, 2]

**Explanation:** Setting `.length` to a value smaller than the current length truncates the array in place: every element at an index `>=` the new length is permanently deleted, keeping only the elements from the front. So `[1, 2, 3, 4, 5]` becomes `[1, 2]`, not the tail `[4, 5]` and not fully emptied. **Idiom:** this truncation behavior is exactly what powers the `arr.length = 0` clear-in-place pattern covered next.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What does this log?

::code-wrapper{language="javascript"}
```javascript
let list = [1, 2, 3];
const ref = list;

function clearWithReassign(a) { a = []; }
function clearWithLength(a) { a.length = 0; }

clearWithReassign(list);
console.log(list, ref);

clearWithLength(list);
console.log(list, ref);
```
::

- [ ] [] [] then [] []
- [ ] [] [1,2,3] then [] []
- [ ] [1,2,3] [1,2,3] then [] []
- [ ] [1,2,3] [1,2,3] then [1,2,3] [1,2,3]

<details>
<summary>Show Answer</summary>

**Answer:** C — [1,2,3] [1,2,3] then [] []

**Explanation:** `clearWithReassign` only rebinds its local parameter `a` to a brand-new empty array; the object that `list` and `ref` point to is never touched, so both still log `[1, 2, 3]`. `clearWithLength` instead mutates the shared array object via `a.length = 0`, so both `list` and `ref` — which reference that same object — become `[]` together. **Idiom:** `array.length = 0` is the standard way to clear an array in place precisely because other variables or closures holding the same reference need to observe the change; reassigning a local variable to `[]` never does.

</details>
::

::question-wrapper{language="javascript"}
### Q12. What do these two calls log?

::code-wrapper{language="javascript"}
```javascript
const nested = [1, [2, 3], [4, [5, 6]]];
console.log(nested.flat());
console.log(nested.flat(2));
```
::

- [ ] [1,2,3,4,5,6] then [1,2,3,4,5,6]
- [ ] [1,2,3,4,5,6] then [1,2,3,4,[5,6]]
- [ ] [1,[2,3],4,[5,6]] then [1,2,3,4,5,6]
- [ ] [1,2,3,4,[5,6]] then [1,2,3,4,5,6]

<details>
<summary>Show Answer</summary>

**Answer:** D — [1,2,3,4,[5,6]] then [1,2,3,4,5,6]

**Explanation:** `.flat()` called with no argument defaults to a depth of `1`, so it only unwraps one level: the top-level `[2, 3]` and `[4, [5, 6]]` each get flattened once, but the inner `[5, 6]` — nested two levels deep — stays wrapped. Passing an explicit depth of `2` flattens two levels, fully unwrapping `[5, 6]` too. **Debug:** forgetting that `.flat()`'s default depth is only `1`, not "flatten completely," is a common surprise — use `Infinity` as the depth if you want to fully flatten an arbitrarily nested array.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What does this log?

::code-wrapper{language="javascript"}
```javascript
const sentences = ['hello world', 'foo bar baz'];
const words = sentences.flatMap(s => s.split(' '));
console.log(words);
```
::

- [ ] ['hello', 'world', 'foo', 'bar', 'baz']
- [ ] [['hello','world'], ['foo','bar','baz']]
- [ ] ['hello world', 'foo bar baz']
- [ ] [5, 5, 3, 3, 3]

<details>
<summary>Show Answer</summary>

**Answer:** A — ['hello', 'world', 'foo', 'bar', 'baz']

**Explanation:** `.flatMap()` first maps each element with the callback — here producing an array of words per sentence via `.split(' ')` — then flattens the result by exactly one level, merging those per-sentence word arrays into a single flat array. Plain `.map()` without the flattening step is what would give the nested array-of-arrays distractor. **Idiom:** `.flatMap()` is more efficient than chaining `.map().flat()` because it avoids building the intermediate nested array.

</details>
::

::question-wrapper{language="javascript"}
### Q14. What happens when this runs?

::code-wrapper{language="javascript"}
```javascript
function sumPositive() {
  return arguments.filter(n => n > 0).length;
}

sumPositive(1, -2, 3);
```
::

- [ ] Returns 2
- [ ] Throws a TypeError because arguments has no filter method
- [ ] Returns 3
- [ ] Returns undefined silently

<details>
<summary>Show Answer</summary>

**Answer:** B — Throws a TypeError because arguments has no filter method

**Explanation:** `arguments` is array-like — it has indexed properties and a `length` — but it does not inherit from `Array.prototype`, so array methods like `.filter()`, `.map()`, and `.reduce()` simply don't exist on it, and calling one throws `TypeError: arguments.filter is not a function`. **Safety:** convert it to a real array first with `Array.from(arguments)` or `[...arguments]` (or just use a rest parameter `...args`, which is already a real array) before using array methods on it.

</details>
::

::question-wrapper{language="javascript"}
### Q15. What happens when this runs in a browser?

::code-wrapper{language="javascript"}
```javascript
const items = document.querySelectorAll('li');
const texts = items.map(el => el.textContent);
```
::

- [ ] Works fine, texts is an array of strings
- [ ] items is automatically a real Array so this always works
- [ ] Throws a TypeError because NodeList has no map method
- [ ] Returns a new NodeList instead of an array

<details>
<summary>Show Answer</summary>

**Answer:** C — Throws a TypeError because NodeList has no map method

**Explanation:** `querySelectorAll` returns a static `NodeList` — an array-like, iterable object that supports `.forEach()` in modern browsers but does not inherit from `Array.prototype`, so `.map()`, `.filter()`, and `.reduce()` are missing and calling them throws a `TypeError`. `Array.isArray(items)` is `false`, confirming it isn't a real array. **Debug:** convert with `Array.from(items)` or `[...items]` first to unlock the full array method set.

</details>
::

::question-wrapper{language="javascript"}
### Q16. What does this log?

::code-wrapper{language="javascript"}
```javascript
const arr = [1, 2, 3, 4, 5];
console.log(arr.slice(-2));
console.log(arr);
```
::

- [ ] [1, 2, 3] then [1, 2, 3, 4, 5]
- [ ] [4, 5] then [1, 2, 3]
- [ ] [3, 4, 5] then [1, 2, 3, 4, 5]
- [ ] [4, 5] then [1, 2, 3, 4, 5]

<details>
<summary>Show Answer</summary>

**Answer:** D — [4, 5] then [1, 2, 3, 4, 5]

**Explanation:** A negative index in `.slice()` counts back from the end, so `slice(-2)` extracts the last two elements, `[4, 5]`. `.slice()` never mutates its source — it always returns a shallow copy of the requested range — so `arr` is unchanged afterward. The `[1, 2, 3]`-then-mutated distractor wrongly assumes `.slice()` behaves like `.splice()`, which does mutate.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What does this log?

::code-wrapper{language="javascript"}
```javascript
const arr = [1, 2, 3, 4, 5];
const removed = arr.splice(-2, 1);
console.log(removed, arr);
```
::

- [ ] [4], [1, 2, 3, 5]
- [ ] [4, 5], [1, 2, 3]
- [ ] [5], [1, 2, 3, 4]
- [ ] [-2], [1, 2, 3, 4, 5]

<details>
<summary>Show Answer</summary>

**Answer:** A — [4], [1, 2, 3, 5]

**Explanation:** Like `.slice()`, a negative start in `.splice()` counts back from the end, so `-2` points at index `3` (the value `4`). `.splice(-2, 1)` deletes just `1` element starting there and returns it as `[4]` — unlike `.slice()`, `.splice()` mutates the original array in place, leaving `arr` as `[1, 2, 3, 5]`. **Debug:** the `[4, 5]`-removed distractor applies `.slice()`'s "grab to the end" instinct to `.splice()`, forgetting that the second argument caps how many elements get deleted.

</details>
::

::question-wrapper{language="javascript"}
### Q18. What does this log?

::code-wrapper{language="javascript"}
```javascript
const nums = [1, 2, 4, 3, 5];
for (let i = 0; i < nums.length; i++) {
  if (nums[i] % 2 === 0) {
    nums.splice(i, 1);
  }
}
console.log(nums);
```
::

- [ ] [1, 3, 5]
- [ ] [1, 4, 3, 5]
- [ ] [1, 2, 4, 3, 5]
- [ ] []

<details>
<summary>Show Answer</summary>

**Answer:** B — [1, 4, 3, 5]

**Explanation:** `.splice()` mutates the array in place, shifting every later element one index to the left. When the loop removes `2` at index `1`, the next value `4` slides into index `1` — but the `for` loop's counter has no idea a shift happened and moves straight on to index `2`, so the element now sitting at index `1` (`4`) is never re-examined and survives untouched. **Debug:** mutating an array with `.splice()` while walking it forward with a plain index loop silently skips the element that shifts into the just-vacated slot. The safe patterns are to loop backwards, iterate over a copy while mutating the original, or just use a non-mutating `.filter()` instead.

</details>
::

::question-wrapper{language="javascript"}
### Q19. What does this log?

::code-wrapper{language="javascript"}
```javascript
const nums = [1, 2, 3, 4, 5];
let visits = 0;
nums.forEach(n => {
  visits++;
  if (n === 3) return;
});

let loopVisits = 0;
for (const n of nums) {
  loopVisits++;
  if (n === 3) break;
}

console.log(visits, loopVisits);
```
::

- [ ] 5, 5
- [ ] 3, 3
- [ ] 5, 3
- [ ] 3, 5

<details>
<summary>Show Answer</summary>

**Answer:** C — 5, 3

**Explanation:** Inside a `.forEach()` callback, `return` only exits that single invocation — it behaves like `continue`, never like a loop-level `break` — so `.forEach()` always calls the callback once per element no matter what, giving `visits = 5`. A `for...of` loop's `break` is a genuine loop-control statement that exits the loop entirely, so `loopVisits` stops incrementing right after `n === 3`, giving `loopVisits = 3`. **Performance:** there is no way to stop `.forEach()` early — reaching for it when you actually need an early exit means either silently doing extra work or, worse, assuming it stopped when it didn't; use a `for...of` or plain `for` loop whenever short-circuiting matters.

</details>
::

::question-wrapper{language="javascript"}
### Q20. What does this log?

::code-wrapper{language="javascript"}
```javascript
function renderLeaderboard(players) {
  return players.sort((a, b) => b.score - a.score);
}

const players = [{ name: 'A', score: 10 }, { name: 'B', score: 30 }];
const original = players;

renderLeaderboard(players);
console.log(original[0].name);
```
::

- [ ] "A" — original stays in its initial order since sort only affects the returned copy
- [ ] undefined — sort empties the array before reordering it
- [ ] "A" — renderLeaderboard receives a separate copy of the array by value
- [ ] "B" — sort mutated the shared array in place, so original reflects the new order too

<details>
<summary>Show Answer</summary>

**Answer:** D — "B" — sort mutated the shared array in place, so original reflects the new order too

**Explanation:** Arrays are passed by reference, and `.sort()` mutates in place rather than returning a new array; since `players` and `original` are the same array object, sorting inside `renderLeaderboard` silently reorders `original` too, putting the higher-scoring `"B"` first. This is exactly the bug that ES2023's `.toSorted()` (along with `.toReversed()`, `.toSpliced()`, and `.with()`) exists to prevent — these non-mutating counterparts return a new array and leave the original untouched. **Safety:** prefer `.toSorted()` over `.sort()` (or explicitly copy first with `[...players].sort(...)`) whenever a function receives an array reference that other code also holds onto.

</details>
::
