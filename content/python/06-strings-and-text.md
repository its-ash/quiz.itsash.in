# 06 — Strings & Text

::question-wrapper{language="python"}
### Q1. What does the following code print?

::code-wrapper{language="python"}
```python
s = "Hello, World"
s.replace("World", "Python")
print(s)
```
::

- [ ] Hello, World
- [ ] Hello, Python
- [ ] TypeError: 'str' object does not support item assignment
- [ ] None

<details>
<summary>Show Answer</summary>

**Answer:** A — Hello, World

**Explanation:** Strings are immutable in Python. `.replace()` (like `.upper()`, `.strip()`, `.lower()`, and every other "modifying" string method) does not change `s` in place — it computes and returns a brand-new string object. Because the return value here is never assigned back to `s`, the original variable still points at the untouched `"Hello, World"`. Option B is the tempting mistake of assuming methods mutate the receiver, as they do for lists. There's no item assignment happening (C), and `.replace()` returns a string, never `None` (D) — only a handful of in-place methods like `list.sort()` return `None`.

</details>
::

::question-wrapper{language="python"}
### Q2. A teammate reports that empty/whitespace-only usernames are slipping past validation. What does this function actually return for `validate_username("   ")`?

::code-wrapper{language="python"}
```python
def validate_username(name):
    name.strip()
    if len(name) == 0:
        return "empty"
    return name

print(validate_username("   "))
```
::

- [ ] "empty"
- [ ] "   " (the three spaces, unchanged)
- [ ] ""
- [ ] Raises AttributeError: 'str' object has no attribute 'strip'

<details>
<summary>Show Answer</summary>

**Answer:** B — "   " (the three spaces, unchanged)

**Explanation:** `name.strip()` returns a new stripped string, but that result is discarded — the local variable `name` still refers to the original three-space string. `len(name)` is therefore `3`, not `0`, so the `"empty"` branch never runs and the untrimmed spaces are returned as if they were a valid username. This is the single most common beginner bug with immutable strings: the fix is `name = name.strip()`. Nothing here raises an error (D), and it never reaches `""` (C) because that would require reassignment first.

</details>
::

::question-wrapper{language="python"}
### Q3. What gets printed, and in what order?

::code-wrapper{language="python"}
```python
def log(msg):
    print(f"LOG: {msg}")
    return msg.upper()

name = "world"
greeting = f"Hello, {log(name)}!"
print(greeting)
```
::

- [ ] Hello, WORLD! then LOG: world
- [ ] LOG: WORLD then Hello, WORLD!
- [ ] LOG: world then Hello, WORLD!
- [ ] Error — you can't call functions inside an f-string's braces

<details>
<summary>Show Answer</summary>

**Answer:** C — LOG: world then Hello, WORLD!

**Explanation:** An f-string's `{}` can hold any valid Python expression, including function calls, and that expression is evaluated immediately, right where the f-string is written — before the surrounding string is even finished being built. So `log(name)` runs first: it prints `LOG: world` (using the original lowercase `name`) and then returns `"WORLD"`. Only after that does the f-string finish assembling `"Hello, WORLD!"`, which the second `print` then outputs. Option B is wrong because `log` prints `msg` (still lowercase) before it uppercases anything.

</details>
::

::question-wrapper{language="python"}
### Q4. What does this print?

::code-wrapper{language="python"}
```python
count = 42
items = ["a", "b"]
print(f"{count=}, {len(items)=}")
```
::

- [ ] 42, 2
- [ ] count, len(items)
- [ ] SyntaxError: invalid syntax
- [ ] count=42, len(items)=2

<details>
<summary>Show Answer</summary>

**Answer:** D — count=42, len(items)=2

**Explanation:** Python 3.8+ supports the `=` debug specifier: writing `{expr=}` inside an f-string expands to the literal source text of `expr`, an `=`, and the value's repr — all in one go. So `{count=}` becomes `count=42` and `{len(items)=}` becomes `len(items)=2`, without you having to type the variable name twice. It's not a syntax error (C) — this has been valid since 3.8 — and it doesn't silently drop the expression text (B) or the value (A); it shows both.

</details>
::

::question-wrapper{language="python"}
### Q5. What is printed?

::code-wrapper{language="python"}
```python
price = 19.999
print(f"${price:.2f}")
```
::

- [ ] $20.00
- [ ] $19.99
- [ ] $19.999
- [ ] ValueError: cannot format 19.999

<details>
<summary>Show Answer</summary>

**Answer:** A — $20.00

**Explanation:** The `.2f` format spec rounds to 2 decimal places using round-half-to-even on the underlying binary value, it does not truncate. `19.999` rounded to two decimals is `20.00`: the third decimal digit (`9`) rounds the second decimal up from `9` to `10`, which carries into the whole-number part. Option B is the truncation mistake a beginner might expect; `.2f` never just chops digits off.

</details>
::

::question-wrapper{language="python"}
### Q6. What does the first line printed by this loop (for `i = 0`) look like?

::code-wrapper{language="python"}
```python
label = "ID"
for i in range(3):
    print(f"{label:>5}: {i:03d}")
```
::

- [ ] `ID:    000` (label left-aligned, no leading spaces)
- [ ] `   ID: 000` (label right-aligned in a width-5 field, index zero-padded to 3 digits)
- [ ] `ID: 0`
- [ ] Raises ValueError: Invalid format specifier

<details>
<summary>Show Answer</summary>

**Answer:** B — `   ID: 000` (label right-aligned in a width-5 field, index zero-padded to 3 digits)

**Explanation:** `{label:>5}` right-aligns `"ID"` (2 characters) inside a minimum field width of 5, padding with 3 leading spaces. `{i:03d}` formats the integer with zero-padding to a minimum of 3 digits, so `0` becomes `000`. Together that produces `"   ID: 000"`. Left-alignment (`<`, option A) would put the padding after `ID`, not before it, and no error is raised — `>` and `03d` are both standard, well-formed format specs.

</details>
::

::question-wrapper{language="python"}
### Q7. What is printed?

::code-wrapper{language="python"}
```python
population = 1234567
print(f"{population:,}")
print(f"{population:_}")
```
::

- [ ] 1234567 then 1234567
- [ ] 1,234,567 twice
- [ ] 1,234,567 then 1_234_567
- [ ] ValueError: unsupported format character

<details>
<summary>Show Answer</summary>

**Answer:** C — 1,234,567 then 1_234_567

**Explanation:** The format spec mini-language supports `,` and `_` as thousands-separator options for numeric types — `,` inserts commas, `_` inserts underscores, both grouping digits in threes. They're independent options, so the two `print` calls produce different separators, not the same one twice (B). This is unrelated to numeric literals like `1_000` in source code; here `_` is purely a display-formatting choice.

</details>
::

::question-wrapper{language="python"}
### Q8. What does this print?

::code-wrapper{language="python"}
```python
def build_template(fmt_type):
    name = "Alice"
    if fmt_type == "fstring":
        return f"Hello, {name}"
    else:
        return "Hello, {name}"

t1 = build_template("fstring")
t2 = build_template("format")

name = "Bob"
print(t1)
print(t2.format(name=name))
```
::

- [ ] Hello, Bob then Hello, Bob
- [ ] Hello, Alice then Hello, Alice
- [ ] NameError: name 'name' is not defined
- [ ] Hello, Alice then Hello, Bob

<details>
<summary>Show Answer</summary>

**Answer:** D — Hello, Alice then Hello, Bob

**Explanation:** An f-string is evaluated eagerly, at the exact point it's written — `f"Hello, {name}"` bakes in whatever `name` is inside the function *right then* (`"Alice"`), producing a fixed, already-finished string. It cannot be "replayed" later with a different value. The plain string `"Hello, {name}"` (no `f` prefix) is just literal text containing `{name}` — nothing is substituted until something later calls `.format(name=...)` on it, using whatever value is passed in at that call site, which by then is the outer, reassigned `"Bob"`. This is the key practical difference between f-strings (eager) and `.format()` templates (lazy, reusable).

</details>
::

::question-wrapper{language="python"}
### Q9. Which statement best describes current best practice for string formatting in modern Python (3.6+)?

::question-wrapper
- [ ] f-strings — they're generally the fastest and most readable option, but %-style and `.format()` still show up because logging APIs rely on lazy %-style formatting and some templates are built before the values exist
- [ ] f-strings — %-formatting and `.format()` are deprecated and will be removed in a future Python version
- [ ] % formatting — because it's the only style guaranteed to work identically on every Python version, including Python 2
- [ ] `.format()` — because f-strings only work inside the interactive REPL, not in saved scripts

<details>
<summary>Show Answer</summary>

**Answer:** A — f-strings — they're generally the fastest and most readable option, but %-style and `.format()` still show up because logging APIs rely on lazy %-style formatting and some templates are built before the values exist

**Explanation:** f-strings (3.6+) are the recommended default: concise, readable, and typically the fastest of the three since the interpolation is compiled in directly. But %-style and `.format()` remain legitimate in real code — `logging.debug("value: %s", x)` deliberately uses %-style so the string is only built if the log level is actually enabled, and `.format()` is still useful for templates defined once and filled in later. Neither older style is deprecated (B), % formatting isn't uniquely version-portable in any meaningful sense today (C), and f-strings work perfectly fine in `.py` scripts, not just the REPL (D).

</details>
::
::

::question-wrapper{language="python"}
### Q10. What happens when this runs?

::code-wrapper{language="python"}
```python
data = b"status: "
result = data + "ok"
```
::

- [ ] result becomes b"status: ok"
- [ ] TypeError: can't concat str to bytes
- [ ] result becomes "status: ok"
- [ ] result becomes b'status: ok' after "ok" is implicitly encoded as ASCII

<details>
<summary>Show Answer</summary>

**Answer:** B — TypeError: can't concat str to bytes

**Explanation:** In Python 3, `str` (text) and `bytes` (raw binary data) are distinct, incompatible types with no implicit conversion between them — unlike Python 2, where `str` and byte strings mixed freely. `+` between a `bytes` object and a `str` object raises `TypeError` immediately; there is no silent auto-encoding (D). To combine them you must be explicit: `data + "ok".encode()` (or `data.decode() + "ok"`), choosing the encoding yourself rather than relying on a guess.

</details>
::

::question-wrapper{language="python"}
### Q11. What happens when this runs?

::code-wrapper{language="python"}
```python
text = "café ☕"
encoded = text.encode("ascii")
```
::

- [ ] encoded becomes b'caf\xc3\xa9 \xe2\x98\x95'
- [ ] encoded becomes b'cafe '
- [ ] UnicodeEncodeError: 'ascii' codec can't encode character '\xe9' in position 3
- [ ] UnicodeDecodeError: 'ascii' codec can't decode byte 0xe9

<details>
<summary>Show Answer</summary>

**Answer:** C — UnicodeEncodeError: 'ascii' codec can't encode character '\xe9' in position 3

**Explanation:** The ASCII codec can only represent code points 0–127. `é` (U+00E9) is outside that range, so `.encode("ascii")` raises `UnicodeEncodeError` at the first offending character it hits — it does not silently strip accents (B) or fall back to UTF-8 bytes (A). It's specifically an *encode* error, not decode (D), because the operation is going from `str` to `bytes`; decode errors happen in the opposite direction. To actually get bytes out of this string, use an encoding that covers the characters (`.encode("utf-8")`) or pass `errors="ignore"`/`"replace"` if lossy output is acceptable.

</details>
::

::question-wrapper{language="python"}
### Q12. This script works fine on macOS but a teammate on Windows gets `UnicodeDecodeError: 'charmap' codec can't decode byte 0x97 in position 42` running it against the exact same UTF-8 encoded file. What's the root cause?

::code-wrapper{language="python"}
```python
with open("report.txt") as f:
    text = f.read()
```
::

- [ ] The file itself becomes corrupted only when stored on a Windows disk
- [ ] Windows is fundamentally unable to read UTF-8 encoded files
- [ ] The character in question is not a valid character on Windows filesystems
- [ ] open() with no encoding= argument uses the platform's default text encoding (often cp1252 on Windows vs. UTF-8 on macOS/Linux), not the file's actual encoding

<details>
<summary>Show Answer</summary>

**Answer:** D — open() with no encoding= argument uses the platform's default text encoding (often cp1252 on Windows vs. UTF-8 on macOS/Linux), not the file's actual encoding

**Explanation:** When `encoding=` is omitted, Python falls back to `locale.getpreferredencoding()`, which depends on the OS and locale settings — commonly UTF-8 on macOS/Linux but historically cp1252 (or another codepage) on Windows. The file's bytes never change between machines; only the *assumed* encoding used to decode them does, and cp1252 doesn't define every byte value, so certain UTF-8 multi-byte sequences trigger a decode error there that never surfaces on a UTF-8-default system. The fix is to always pass `open(path, encoding="utf-8")` explicitly so behavior is identical everywhere.

</details>
::

::question-wrapper{language="python"}
### Q13. What are the values of `check1` and `check2`?

::code-wrapper{language="python"}
```python
log_line = "2024-01-15 ERROR: connection timeout"
levels = ["ERROR", "WARN"]

check1 = "ERROR" in log_line
check2 = "ERR" in levels
```
::

- [ ] True, False
- [ ] True, True
- [ ] False, True
- [ ] TypeError: 'in' not supported between instances of 'str' and 'list'

<details>
<summary>Show Answer</summary>

**Answer:** A — True, False

**Explanation:** `in` means something different depending on the container. Against a `str`, it tests substring containment at any position, so `"ERROR" in log_line` is `True` because that exact sequence of characters occurs in the text. Against a `list`, `in` tests exact element membership — it does not search inside each element's characters — so `"ERR" in levels` is `False`, even though `"ERR"` happens to be a substring of the list's `"ERROR"` entry. Conflating these two behaviors is a common source of silent logic bugs.

</details>
::

::question-wrapper{language="python"}
### Q14. What is printed?

::code-wrapper{language="python"}
```python
border = "-" * 10
empty1 = "x" * 0
empty2 = "x" * -5
print(repr(border), repr(empty1), repr(empty2))
```
::

- [ ] '----------' '' Error: negative repeat count
- [ ] '----------' '' ''
- [ ] '----------' 'x' ''
- [ ] TypeError: can't multiply sequence by non-int

<details>
<summary>Show Answer</summary>

**Answer:** B — '----------' '' ''

**Explanation:** `str * int` repeats the string that many times. A multiplier of `0` produces an empty string, and — this is the gotcha — so does any *negative* multiplier: Python silently clamps negative repeat counts down to `0` rather than raising an error. There's no exception for either case (ruling out A and D), and `"x" * 0` is `''`, not `'x'` (C).

</details>
::

::question-wrapper{language="python"}
### Q15. What is printed?

::code-wrapper{language="python"}
```python
s = "python"
a = s[::-1]
b = s[2:100]
c = s[50:60]
print(a, b, c)
```
::

- [ ] nohtyp thon IndexError: string index out of range
- [ ] The call raises IndexError while evaluating `a`
- [ ] nohtyp thon ''
- [ ] nohtyp IndexError: string index out of range ''

<details>
<summary>Show Answer</summary>

**Answer:** C — nohtyp thon ''

**Explanation:** `s[::-1]` with a step of `-1` reverses the whole string, giving `"nohtyp"`. Slicing is far more forgiving than direct indexing: out-of-range start/stop bounds are silently clamped rather than raising. `s[2:100]` clamps the stop to the string's actual length and returns everything from index 2 onward, `"thon"`. `s[50:60]` starts past the end of the string entirely, so it returns an empty string `''` — not an error. Contrast this with `s[50]`, which *would* raise `IndexError`; slicing and indexing have different bounds-checking rules, and forgetting that can hide real off-by-one bugs.

</details>
::

::question-wrapper{language="python"}
### Q16. What is printed?

::code-wrapper{language="python"}
```python
path = "C:\new_folder\test"
print(path)
```
::

- [ ] C:\new_folder\test, printed exactly as written
- [ ] SyntaxError: invalid syntax
- [ ] FileNotFoundError: no such file or directory
- [ ] A garbled, multi-line result — \n becomes a newline and \t becomes a tab, so it prints as `C:` on one line, `ew_folder` on the next, then a tab before `est`

<details>
<summary>Show Answer</summary>

**Answer:** D — A garbled, multi-line result — \n becomes a newline and \t becomes a tab, so it prints as `C:` on one line, `ew_folder` on the next, then a tab before `est`

**Explanation:** In an ordinary string literal, backslashes introduce escape sequences. `\n` inside `"...C:\new_folder..."` is not "backslash then n" — it's interpreted as an actual newline character, and `\t` in `\test` becomes an actual tab character. The path is silently corrupted with no error raised (ruling out B and C), which is far more dangerous than a crash because it can go unnoticed. The fix is a raw string, `r"C:\new_folder\test"`, which tells Python to treat every backslash as a literal character — essential for Windows paths and regex patterns, where backslashes are meaningful on their own.

</details>
::

::question-wrapper{language="python"}
### Q17. What is printed?

::code-wrapper{language="python"}
```python
a = "hello world"
b = "hello world"
c = "".join(["hello", " ", "world"])

print(a == b, a is b)
print(a == c, a is c)
```
::

- [ ] True True / True False
- [ ] True True / True True
- [ ] True False / True False
- [ ] True True / False False

<details>
<summary>Show Answer</summary>

**Answer:** A — True True / True False

**Explanation:** `==` always compares by value, so all four comparisons of content agree — both pairs hold equal text, hence `True` for both `==` checks. `is` compares object identity, which is a different question. `a` and `b` are two identical string literals appearing in the same compiled code; CPython's compiler commonly folds equal literals into one shared object as an implementation detail, so `a is b` often comes out `True` — but this is not guaranteed by the language spec, only an optimization. `c` is built at runtime via `"".join(...)`, producing a genuinely new object even though its contents match, so `a is c` is `False`. The takeaway: never rely on `is` for string equality — always use `==`.

</details>
::

::question-wrapper{language="python"}
### Q18. A CLI tool reads and writes text files on both Windows and Linux CI runners. Which pattern is the most robust and portable?

::question-wrapper
- [ ] open(path, "r") and open(path, "w") with no encoding argument — Python automatically picks the correct encoding for each OS
- [ ] open(path, "r", encoding="utf-8") and open(path, "w", encoding="utf-8") — always pass encoding explicitly so behavior is identical on every machine
- [ ] open(path, "rb") for every read and write, to sidestep encoding entirely
- [ ] Wrap every read in try/except UnicodeDecodeError and silently fall back to latin-1 on failure

<details>
<summary>Show Answer</summary>

**Answer:** B — open(path, "r", encoding="utf-8") and open(path, "w", encoding="utf-8") — always pass encoding explicitly so behavior is identical on every machine

**Explanation:** Leaving `encoding` unset means Python uses the platform's locale-dependent default, which varies by OS and CI environment — exactly the source of the cross-platform `UnicodeDecodeError` bugs discussed earlier. Passing `encoding="utf-8"` explicitly on every `open()` call makes behavior deterministic everywhere. Reading/writing in binary mode (`"rb"`/`"wb"`, option C) avoids the crash but throws away text semantics — you're back to juggling raw bytes and manual `.encode()`/`.decode()` calls yourself. Silently falling back to latin-1 (D) doesn't fix anything; latin-1 can decode almost any byte sequence without error, so it masks the real problem and produces subtly corrupted (mojibake) text instead of a clear failure.

</details>
::

::question-wrapper{language="python"}
### Q19. What happens when this runs?

::code-wrapper{language="python"}
```python
data = (1, 2)
print("%s" % data)
```
::

- [ ] (1, 2)
- [ ] 1 2
- [ ] TypeError: not all arguments converted during string formatting
- [ ] SyntaxError: invalid syntax

<details>
<summary>Show Answer</summary>

**Answer:** C — TypeError: not all arguments converted during string formatting

**Explanation:** With `%`-style formatting, if the right-hand operand is a tuple, its elements are unpacked positionally to fill each `%s`/`%d` placeholder in order — Python does not treat `(1, 2)` as one opaque value here. There's only a single `%s` placeholder but two tuple elements: the first fills it, leaving the second unconsumed, which triggers `"not all arguments converted during string formatting"`. To format a tuple as one literal value, you must nest it in another tuple: `"%s" % ((1, 2),)`. This is one of `%`-formatting's classic sharp edges and part of why f-strings are now preferred for anything beyond simple cases.

</details>
::

::question-wrapper{language="python"}
### Q20. What is printed?

::code-wrapper{language="python"}
```python
value = 3.14159
width = 10
precision = 2
print(f"{value:{width}.{precision}f}")
```
::

- [ ] "3.14      " (left-aligned in a field of width 10)
- [ ] "3.14"
- [ ] ValueError: Invalid format specifier
- [ ] "      3.14" (right-aligned in a field of width 10)

<details>
<summary>Show Answer</summary>

**Answer:** D — "      3.14" (right-aligned in a field of width 10)

**Explanation:** Format specs can themselves contain nested replacement fields — `{width}` and `{precision}` inside the outer spec are evaluated first, producing the equivalent of `{value:10.2f}` at runtime, so width and precision can be computed dynamically instead of hardcoded. `3.14` is 4 characters, so a minimum width of 10 adds 6 leading spaces. Numeric types default to right-alignment when a minimum width is specified — unlike strings, which default to left-alignment — so the padding goes before the digits, not after (ruling out A).

</details>
::
