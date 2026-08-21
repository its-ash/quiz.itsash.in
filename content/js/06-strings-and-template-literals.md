# 06 — Strings & Template Literals

::question-wrapper
### Q1. Which statement correctly describes string mutability in JavaScript?

- [ ] String methods like `.toUpperCase()` and `.slice()` mutate the original string in place and also return the modified string for chaining
- [ ] Strings are immutable — methods like `.toUpperCase()` and `.slice()` always return a new string, leaving the original string unchanged
- [ ] Strings are mutable only when declared with `let`, but immutable when declared with `const`
- [ ] Whether a string method mutates depends on the method — `.slice()` mutates the original but `.toUpperCase()` does not

<details>
<summary>Show Answer</summary>

**Answer:** B — Strings are immutable — methods like `.toUpperCase()` and `.slice()` always return a new string, leaving the original string unchanged

**Explanation:** Strings are a primitive type in JavaScript, and primitives are immutable — no operation can change the characters of an existing string value. Every "transforming" string method actually builds and returns a brand-new string, leaving the original untouched unless you reassign the variable. Option C confuses variable *reassignability* (`let` vs. `const`) with value *mutability*, which are unrelated concerns — even a `let`-bound string's underlying value can never be mutated in place. Option D is wrong because no built-in string method mutates, regardless of which one you pick.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does this log?

::code-wrapper{language="javascript"}
```javascript
function shout(word) {
  return word.toUpperCase() + "!";
}

const name = "ava";
const msg = `Hello, ${shout(name)}
Welcome aboard.`;

console.log(msg);
```
::

- [ ] `Hello, AVA!` and `Welcome aboard.` printed on the same line, separated by a space
- [ ] A `SyntaxError`, because template literals cannot span multiple lines
- [ ] `Hello, AVA!` followed by a literal newline, then `Welcome aboard.` on the next line
- [ ] The literal text `Hello, ${shout(name)}`, because a function call inside `${}` is not evaluated

<details>
<summary>Show Answer</summary>

**Answer:** C — `Hello, AVA!` followed by a literal newline, then `Welcome aboard.` on the next line

**Explanation:** Template literals preserve any newline characters typed between the backticks as an actual `\n` in the resulting string, so the output spans two lines. A `${}` placeholder can hold any valid JS expression, including function calls — they're evaluated and the result is coerced to a string before splicing in, so `shout("ava")` contributes `"AVA!"`. Option B is wrong because multi-line strings without escape sequences are one of the main reasons template literals exist. Option D is wrong because expressions inside `${}` are always evaluated — that's the entire point of interpolation, not literal text substitution.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What does this log?

::code-wrapper{language="javascript"}
```javascript
const a = "10" + 5;
const b = "10" - 5;
const c = "10" + 5 - 3;

console.log(a, b, c);
```
::

- [ ] `"105" 5 102`
- [ ] `15 5 12`
- [ ] `"105" 5 "1023"`
- [ ] `15 5 "1023"`

<details>
<summary>Show Answer</summary>

**Answer:** A — `"105" 5 102`

**Explanation:** `+` is overloaded: if either operand is a string, it performs string concatenation, so `"10" + 5` produces `"105"`. `-` has no concatenation meaning, so both operands go through ToNumber, giving `10 - 5 = 5`. For `c`, evaluation runs left-to-right: `"10" + 5` first yields the string `"105"`, and then `"105" - 3` coerces `"105"` back to the number `105` for subtraction, producing `102` — not `"1023"`, since `-` never concatenates, and not `12`, since the first `+` already stringified the `10`. This string-then-number flip mid-expression is a classic coercion trap. **Debug**

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does this log?

::code-wrapper{language="javascript"}
```javascript
const results = [
  "5" == 5,
  "5" === 5,
  "5" == "5.0",
  5 == "5.0",
];

console.log(results);
```
::

- [ ] `[true, true, false, true]`
- [ ] `[false, false, false, true]`
- [ ] `[true, false, true, true]`
- [ ] `[true, false, false, true]`

<details>
<summary>Show Answer</summary>

**Answer:** D — `[true, false, false, true]`

**Explanation:** `==` triggers ToNumber coercion when comparing a string and a number, so `"5" == 5` is `true`. `===` requires matching types with no coercion, so `"5" === 5` is `false` since the operand types differ. `"5" == "5.0"` compares two strings with **no** coercion at all — they're different character sequences, so it's `false` even though they'd represent the same numeric value. `5 == "5.0"` coerces `"5.0"` to the number `5`, matching, so it's `true`. The trap is assuming string-to-string `==` behaves like numeric equality — coercion only kicks in when the operand types actually differ. **Debug**

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does this log?

::code-wrapper{language="javascript"}
```javascript
console.log(String(null));
console.log(`${null}`);
console.log(null + "");
console.log(String(undefined) === (undefined + ""));
```
::

- [ ] `"null" "null" "" true`
- [ ] `"null" "null" "null" true`
- [ ] `"" "" "" true`
- [ ] `"null" "null" "null" false`

<details>
<summary>Show Answer</summary>

**Answer:** B — `"null" "null" "null" true`

**Explanation:** `String(x)` and template-literal interpolation both invoke the same ToString operation, and `ToString(null)` is specified to return the literal string `"null"` — not an empty string, despite `null` being falsy. The `+` operator, when one operand is a string, also converts the other operand through ToPrimitive → ToString, so `null + ""` likewise yields `"null"`. The same logic makes `String(undefined)` and `undefined + ""` both produce `"undefined"`, so the strict-equality check is `true`. Option C's empty-string result is the trap for anyone who conflates "falsy" with "stringifies to empty" — falsiness is a boolean-context concept, unrelated to how a value stringifies. **Debug**

</details>
::

::question-wrapper{language="javascript"}
### Q6. What does this log?

::code-wrapper{language="javascript"}
```javascript
let n = 0;
function next() {
  n += 1;
  return n;
}

const log = `${next()}-${next()}-${next()}`;
console.log(log);
```
::

- [ ] `"3-3-3"`
- [ ] `"3-2-1"`
- [ ] `"1-2-3"`
- [ ] `"0-1-2"`

<details>
<summary>Show Answer</summary>

**Answer:** C — `"1-2-3"`

**Explanation:** Each `${}` slot in a template literal is evaluated in strict left-to-right textual order, with no batching or deferred evaluation. The first `next()` call mutates `n` to `1` and returns `1`; the second call sees the already-mutated `n` and returns `2`; the third returns `3`. This matters whenever interpolated expressions have side effects — reordering placeholders in the template changes the actual side-effect order, not just the visual layout. Option A wrongly assumes templates resolve every placeholder against some final state rather than evaluating each in place as encountered. **Debug**

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does this log?

::code-wrapper{language="javascript"}
```javascript
function tag(strings, ...values) {
  return strings.reduce(
    (out, str, i) => out + str + (values[i] !== undefined ? `[${values[i]}]` : ""),
    ""
  );
}

const price = 42;
const item = "mug";
console.log(tag`The ${item} costs $${price}.`);
```
::

- [ ] `"The [mug] costs $[42]."`
- [ ] A `TypeError`, because tag functions can't accept a variable number of interpolated values
- [ ] `"The mug costs $42."`
- [ ] `"The costs $."`, because the tag function drops the interpolated values entirely

<details>
<summary>Show Answer</summary>

**Answer:** A — `"The [mug] costs $[42]."`

**Explanation:** A tag function receives the literal string segments as its first argument (here `["The ", " costs $", "."]`) and each interpolated expression's *evaluated value* as the subsequent arguments, which the rest parameter `...values` collects into `["mug", 42]`. The tag function fully controls how those pieces combine — nothing happens automatically the way an untagged template works. Option C describes what an *untagged* template would produce; tagging replaces that default behavior entirely with whatever the function returns. Option D wrongly assumes rest parameters can't capture the interpolated values — `...values` works exactly like in any other function signature. **Idiom**

</details>
::

::question-wrapper{language="javascript"}
### Q8. What does this log?

::code-wrapper{language="javascript"}
```javascript
function inspect(strings) {
  console.log(strings[0]);
  console.log(strings.raw[0]);
}

inspect`Line1\nLine2`;
```
::

- [ ] Both lines print identically as `Line1\nLine2`, since raw and cooked are always the same for tagged templates
- [ ] A `SyntaxError`, because `\n` cannot appear in a tagged template literal
- [ ] The first log prints the literal text `Line1\nLine2`; the second log prints across two lines
- [ ] The first log prints across two lines (`Line1` then `Line2`); the second log prints the literal text `Line1\nLine2` on one line with a visible backslash

<details>
<summary>Show Answer</summary>

**Answer:** D — The first log prints across two lines (`Line1` then `Line2`); the second log prints the literal text `Line1\nLine2` on one line with a visible backslash

**Explanation:** Every tagged template gives the tag function two views of each segment: the "cooked" value (`strings[i]`), where escapes like `\n` are interpreted per normal string rules, and the "raw" value (`strings.raw[i]`), which preserves the exact source characters with no escape processing. So `strings[0]` contains an actual newline, printing across two lines, while `strings.raw[0]` still contains the literal backslash-`n` characters, printing on one line. This raw/cooked split is what libraries like styled-components and SQL template tags rely on to recover the original source text. **Idiom**

</details>
::

::question-wrapper{language="javascript"}
### Q9. What does this log?

::code-wrapper{language="javascript"}
```javascript
const emoji = "😀";
console.log(emoji.length);
console.log([...emoji].length);
```
::

- [ ] `1` then `1`
- [ ] `2` then `1`
- [ ] `2` then `2`
- [ ] `1` then `2`

<details>
<summary>Show Answer</summary>

**Answer:** B — `2` then `1`

**Explanation:** JavaScript strings are UTF-16 internally, and `.length` counts 16-bit code units, not visible characters. 😀 lies outside the Basic Multilingual Plane, so it's encoded as a *surrogate pair* — two UTF-16 code units representing one code point — making `.length` report `2` for what looks like a single character. Spreading a string (`[...emoji]`) uses the string's default iterator, which is defined to yield whole Unicode code points, correctly treating the pair as one element, so `[...emoji].length` is `1`. This gap between `.length` and perceived-character count is why naive truncation of user text containing emoji is a common source of bugs. **Safety**

</details>
::

::question-wrapper{language="javascript"}
### Q10. What does this log?

::code-wrapper{language="javascript"}
```javascript
const emoji = "😀";
console.log(emoji[0]);
console.log(emoji.charCodeAt(0));
console.log(emoji.codePointAt(0));
```
::

- [ ] All three return the same value, since JS treats emoji as single units for every string API
- [ ] `charCodeAt(0)` returns the full code point; `codePointAt(0)` returns just the high surrogate half
- [ ] `emoji[0]` prints a lone/garbled surrogate half; `charCodeAt(0)` returns the high surrogate's code unit value; `codePointAt(0)` returns the full emoji's actual code point
- [ ] `emoji[0]` throws a `RangeError`, because indexing into a surrogate pair is invalid

<details>
<summary>Show Answer</summary>

**Answer:** C — `emoji[0]` prints a lone/garbled surrogate half; `charCodeAt(0)` returns the high surrogate's code unit value; `codePointAt(0)` returns the full emoji's actual code point

**Explanation:** Bracket indexing and `.charCodeAt()` both operate on raw UTF-16 code units with no concept of surrogate pairs, so index `0` returns just the first unit — a lone high surrogate that doesn't form a valid character on its own. `.codePointAt()`, added specifically to fix this gap, checks whether the code unit at the given index starts a surrogate pair and, if so, combines it with the following low surrogate to return the true code point (`128512`). Nothing throws — string indexing never validates surrogate boundaries, it silently returns whatever unit sits at that position, which is exactly the trap. **Safety**

</details>
::

::question-wrapper{language="javascript"}
### Q11. What does this log?

::code-wrapper{language="javascript"}
```javascript
console.log("Z" < "a");
console.log("apple" < "Apple");
console.log("2" < "10");
```
::

- [ ] `true, true, true`
- [ ] `false, true, false`
- [ ] `false, false, true`
- [ ] `true, false, false`

<details>
<summary>Show Answer</summary>

**Answer:** D — `true, false, false`

**Explanation:** String comparison operators compare lexicographically by UTF-16 code unit value, not by alphabetical or locale-aware rules. Uppercase ASCII letters (65–90) all have lower code points than lowercase letters (97–122), so `"Z"` (90) sorts before `"a"` (97) — `true`. For the same reason `"apple" < "Apple"` is `false`: comparing first characters, lowercase `'a'` (97) is greater than uppercase `'A'` (65). And `"2" < "10"` is `false` because string comparison never parses numeric value — it compares character-by-character, and `'2'` (50) is greater than `'1'` (49), regardless of 2 being numerically less than 10. **Debug**

</details>
::

::question-wrapper{language="javascript"}
### Q12. What does this log?

::code-wrapper{language="javascript"}
```javascript
const input = "2024-06-15";
const parts = input.split(/(-)/);
console.log(parts);
```
::

- [ ] `["2024", "-", "06", "-", "15"]`
- [ ] `["2024", "06", "15"]`
- [ ] `["2024-06-15"]`
- [ ] `["2024", "06", "15", "-", "-"]`

<details>
<summary>Show Answer</summary>

**Answer:** A — `["2024", "-", "06", "-", "15"]`

**Explanation:** When the separator passed to `.split()` is a regex with a *capturing group*, the spec splices the captured groups into the result array between the surrounding pieces — a deliberate feature for retaining delimiters when needed. A plain non-capturing `/-/ ` would produce `["2024", "06", "15"]` with the dashes discarded, which is what most people expect and what trips them up when they add parentheses for grouping without realizing it changes the output shape. Option D's ordering is wrong because captures are interleaved at the position they occurred, not appended at the end. **Idiom**

</details>
::

::question-wrapper{language="javascript"}
### Q13. What does this log?

::code-wrapper{language="javascript"}
```javascript
const text = "Hi😀!";
console.log(text.split("").length);
console.log(text.split("").join("|"));
```
::

- [ ] `4` and the emoji renders intact in the joined string
- [ ] `5` and the emoji is split apart, appearing as two broken glyphs separated by a `|` in the joined string
- [ ] `5` and the emoji renders intact between two pipes, e.g. `H|i|😀|!`
- [ ] A `RangeError` is thrown, because `split("")` cannot split a surrogate pair

<details>
<summary>Show Answer</summary>

**Answer:** B — `5` and the emoji is split apart, appearing as two broken glyphs separated by a `|` in the joined string

**Explanation:** `.split("")` breaks a string at every UTF-16 code unit boundary, exactly like indexing — it has no awareness of surrogate pairs. Since 😀 is stored as two code units, splitting produces five array elements (`H`, `i`, high surrogate, low surrogate, `!`), not four, and `.join("|")` inserts a pipe *between* the two surrogate halves, visibly corrupting the emoji into two broken glyphs. Nothing throws — string operations never validate surrogate boundaries. The safe way to split into user-perceived characters is `[...text]` or `Array.from(text)`, which iterate by code point. **Safety**

</details>
::

::question-wrapper{language="javascript"}
### Q14. What does this log?

::code-wrapper{language="javascript"}
```javascript
const log = "error: retrying... error: retrying... error: done";
const withString = log.replace("error", "OK");
const withRegex = log.replace(/error/g, "OK");
console.log(withString);
console.log(withRegex);
```
::

- [ ] Both replace every occurrence of `"error"` with `"OK"`
- [ ] Both replace only the first occurrence, since `.replace()` never replaces more than one match regardless of pattern type
- [ ] `withString` replaces only the first `"error"`; `withRegex` replaces every occurrence because of the `g` flag
- [ ] `withString` replaces every occurrence; `withRegex` replaces only the first, because regex patterns default to single-match unless quoted

<details>
<summary>Show Answer</summary>

**Answer:** C — `withString` replaces only the first `"error"`; `withRegex` replaces every occurrence because of the `g` flag

**Explanation:** When the search pattern passed to `.replace()` is a plain string, it's treated as a literal, single-match search — only the first occurrence is replaced no matter how many times it appears. To replace every match you need a regex with the global (`g`) flag; without `g`, even a regex would only replace the first match. This trips people up because `.replaceAll()` (ES2021) *does* replace every occurrence with a string pattern, so confusing `.replace()`/`.replaceAll()` semantics, or forgetting `/g`, is a common "why did only one get replaced" bug. **Debug**

</details>
::

::question-wrapper{language="javascript"}
### Q15. What does this log?

::code-wrapper{language="javascript"}
```javascript
const raw = "price: 42 dollars";
const withCapture = raw.replace(/(\d+)/, "[$1]");
const withMatch = raw.replace(/\d+/, "<$&>");
console.log(withCapture);
console.log(withMatch);
```
::

- [ ] `"price: [$1] dollars"` and `"price: <$&> dollars"`
- [ ] A `SyntaxError`, because `$1` and `$&` are not valid inside replacement strings
- [ ] `"price: [42] dollars"` and `"price: <dollars> dollars"`
- [ ] `"price: [42] dollars"` and `"price: <42> dollars"`

<details>
<summary>Show Answer</summary>

**Answer:** D — `"price: [42] dollars"` and `"price: <42> dollars"`

**Explanation:** In a `.replace()` replacement string, `$1` (through `$9`) refers to the text captured by the corresponding parenthesized group, and `$&` refers to the entire matched substring — these are special tokens the engine substitutes, not literal characters. Both examples correctly substitute the matched number, `42`. Someone unfamiliar with these tokens might assume they render as literal text (option A) or that they require a callback function to work — but this replacement-string mini-syntax handles it without one. **Idiom**

</details>
::

::question-wrapper{language="javascript"}
### Q16. What does this log?

::code-wrapper{language="javascript"}
```javascript
console.log("5".padStart(3));
console.log("5".padStart(3, "0"));
console.log("5".padEnd(3, "0"));
```
::

- [ ] `"  5"`, `"005"`, `"500"`
- [ ] `"005"`, `"005"`, `"500"`
- [ ] `"5  "`, `"005"`, `"500"`
- [ ] `"  5"`, `"500"`, `"005"`

<details>
<summary>Show Answer</summary>

**Answer:** A — `"  5"`, `"005"`, `"500"`

**Explanation:** `.padStart()`/`.padEnd()` take an optional second argument for the pad string, and when it's omitted the default pad character is a plain space `" "` — not `"0"` as many assume from seeing it used for zero-padding numbers. So `"5".padStart(3)` produces `"  5"` (two leading spaces), not `"005"`; you must explicitly pass `"0"` for numeric zero-padding. `padStart` pads at the beginning (right-aligning, e.g. clock digits), while `padEnd` pads at the end (left-aligning fixed-width output); mixing them up is a common off-by-direction bug. **Idiom**

</details>
::

::question-wrapper{language="javascript"}
### Q17. What does this log?

::code-wrapper{language="javascript"}
```javascript
const raw = "   pending review   ";
console.log(`[${raw.trim()}]`);
console.log(`[${raw.trimStart()}]`);
console.log(`[${raw.trimEnd()}]`);
```
::

- [ ] `[   pending review   ]` for all three, since trim variants only affect console output, not the actual string
- [ ] `[pending review]`, `[pending review   ]`, `[   pending review]`
- [ ] `[pending review]`, `[pending review]`, `[pending review]`
- [ ] `[pending review]`, `[   pending review]`, `[pending review   ]`

<details>
<summary>Show Answer</summary>

**Answer:** B — `[pending review]`, `[pending review   ]`, `[   pending review]`

**Explanation:** `.trim()` strips whitespace from both ends. `.trimStart()` removes only *leading* whitespace, leaving trailing spaces intact, while `.trimEnd()` removes only *trailing* whitespace, leaving leading spaces intact. All three return a new string — none mutate `raw` — so each call in the example operates on the original, still-padded string, not on a progressively trimmed one. This distinction matters when formatting user input where indentation on one side is intentional. **Idiom**

</details>
::

::question-wrapper{language="javascript"}
### Q18. What does this log?

::code-wrapper{language="javascript"}
```javascript
const words = ["café", "cafe", "cafz"];
console.log([...words].sort());
console.log([...words].sort((a, b) => a.localeCompare(b)));
```
::

- [ ] Both produce the same alphabetically correct order, `["cafe", "café", "cafz"]`, since `.sort()` is locale-aware by default in modern engines
- [ ] Default `.sort()` throws a `TypeError`, because it cannot compare strings containing accented characters
- [ ] Default `.sort()` gives `["cafe", "cafz", "café"]` (comparing raw UTF-16 code points, where `'é'` sorts after `'z'`); `.localeCompare()` gives `["cafe", "café", "cafz"]` (locale-aware, alphabetically correct)
- [ ] `.localeCompare()` gives `["cafe", "cafz", "café"]`; default `.sort()` gives the alphabetically correct `["cafe", "café", "cafz"]` — the two are swapped from what most people expect

<details>
<summary>Show Answer</summary>

**Answer:** C — Default `.sort()` gives `["cafe", "cafz", "café"]` (comparing raw UTF-16 code points, where `'é'` sorts after `'z'`); `.localeCompare()` gives `["cafe", "café", "cafz"]` (locale-aware, alphabetically correct)

**Explanation:** The default `.sort()` comparator converts elements to strings and compares them by UTF-16 code unit value. Accented `é` (U+00E9 = 233) has a higher code point than both `'e'` (101) and `'z'` (122), so `"café"` sorts after `"cafz"` even though a human expects it right next to `"cafe"`. `.localeCompare()` uses the environment's locale-aware collation (ICU), which treats diacritics as minor variations of a base letter for sorting, producing the intuitive order. This is why user-facing lists of names/words should be sorted with `.localeCompare()` or `Intl.Collator`, not the default comparator. **Idiom**

</details>
::

::question-wrapper
### Q19. Which statement best captures why template literals are generally preferred over `+` concatenation for building strings with embedded values?

- [ ] String concatenation with `+` should always be preferred because it performs better than template literals in every JS engine
- [ ] Template literals are purely syntactic sugar with no practical benefit over `+` concatenation, so the choice is only about personal style
- [ ] `+` concatenation should be preferred whenever numbers are involved, since template literals silently convert numbers to strings and lose precision
- [ ] Template literals read closer to the final output, avoid `+`'s string/number coercion ambiguity when concatenating operands, and support multi-line text without escape characters

<details>
<summary>Show Answer</summary>

**Answer:** D — Template literals read closer to the final output, avoid `+`'s string/number coercion ambiguity when concatenating operands, and support multi-line text without escape characters

**Explanation:** Template literals let you see the shape of the final string directly in the source (`` `Hello, ${name}!` `` vs. `"Hello, " + name + "!"`), reducing bugs from misplaced quotes or operators in longer chains. They also sidestep a specific `+` trap: because `+` is overloaded for both numeric addition and string concatenation, expressions like `"Total: " + a + b` can silently produce wrong results if `a`/`b` were meant to be summed first (`"Total: " + 1 + 2` → `"Total: 12"`, not `"Total: 3"`), whereas `` `Total: ${a + b}` `` makes the intended arithmetic explicit. Multi-line strings are also native, with no `\n` escapes needed. Neither performance (A) nor precision loss (C) are legitimate concerns — both forms convert values via the same ToString semantics, and modern engines optimize both similarly. **Idiom**

</details>
::

::question-wrapper{language="javascript"}
### Q20. What does this log?

::code-wrapper{language="javascript"}
```javascript
function reverse(str) {
  return str.split("").reverse().join("");
}

console.log(reverse("abc"));
console.log(reverse("Hi😀"));
```
::

- [ ] `"cba"` and a string beginning with two broken/garbled surrogate halves in the wrong order, followed by `"iH"` — not a valid emoji
- [ ] `"cba"` and `"😀iH"` — both reverse correctly since JavaScript strings always reverse by visual character
- [ ] `"cba"` and `"iH😀"` — the emoji is silently moved but stays visually intact because `.split("")` is code-point aware
- [ ] `"cba"` and a `RangeError`, because reversing a surrogate pair is not a defined operation

<details>
<summary>Show Answer</summary>

**Answer:** A — `"cba"` and a string beginning with two broken/garbled surrogate halves in the wrong order, followed by `"iH"` — not a valid emoji

**Explanation:** `.split("")` operates on UTF-16 code units, so it tears the emoji's surrogate pair into two separate array entries; `.reverse()` then treats those two halves as independent elements and flips their order along with everything else, reassembling them as *low surrogate followed by high surrogate* — the opposite of a valid pair — producing a broken result instead of the emoji. This is a real bug pattern in naive "reverse a string" utilities. The fix is to reverse by code point, e.g. `[...str].reverse().join("")`, since the spread iterator groups surrogate pairs into single elements before reversing, keeping each pair intact and correctly ordered. **Safety**

</details>
::
