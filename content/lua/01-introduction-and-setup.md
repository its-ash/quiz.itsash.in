# 01 — Introduction & Setup

::question-wrapper
### Q1. What is Lua fundamentally designed to be?

- [ ] A lightweight, embeddable scripting language meant to be hosted inside larger applications
- [ ] A general-purpose systems programming language for writing operating systems
- [ ] A markup language for describing web page layouts
- [ ] A statically-typed compiled language for high-performance servers

<details>
<summary>Show Answer</summary>

**Answer:** A — A lightweight, embeddable scripting language meant to be hosted inside larger applications

**Explanation:** Lua was created in 1993 at PUC-Rio specifically to be embedded into C host applications as a configuration/extension language; its small footprint, C API, and minimal standard library all reflect that goal. It's not aimed at OS development (option B) or markup (option C); while it can run standalone scripts, that's a side effect of the reference `lua` interpreter, not the language's primary design goal, and option D wrongly implies static typing and ahead-of-time compilation, which Lua doesn't have.

</details>
::

::question-wrapper
### Q2. Which statement accurately describes the relationship between `lua` (the reference/PUC-Lua interpreter) and `luajit` (LuaJIT)?

- [ ] They are the same VM; `luajit` is just a faster build flag for the reference interpreter
- [ ] LuaJIT is a strict superset that always supports every language feature added in the latest PUC-Lua release
- [ ] LuaJIT is a separate implementation with its own VM and JIT compiler, and its language support lags behind — it targets roughly Lua 5.1 semantics with some 5.2 extensions, not 5.4
- [ ] LuaJIT only works with Lua bytecode compiled by `luac`, not with `.lua` source files

<details>
<summary>Show Answer</summary>

**Answer:** C — LuaJIT is a separate implementation with its own VM and JIT compiler, and its language support lags behind — it targets roughly Lua 5.1 semantics with some 5.2 extensions, not 5.4

**Explanation:** LuaJIT (by Mike Pall) is a from-scratch VM and JIT compiler with excellent performance, but its core language-version tracking stalled around 5.1 with a handful of 5.2-ish additions; it does not support Lua 5.3/5.4 features such as the integer subtype, native bitwise-operator syntax, or `<const>`/`<close>` attributes. Option A is wrong because they are distinct codebases, not a build flag; option B inverts reality; option D is wrong because LuaJIT compiles and runs `.lua` source directly, just like PUC-Lua.

</details>
::

::question-wrapper{language="bash"}
### Q3. Given the following two invocations, what's the key difference in behavior?

::code-wrapper{language="bash"}
```bash
lua script.lua
lua
```
::

- [ ] Both start an interactive REPL; the filename is ignored unless preceded by `-i`
- [ ] The first executes `script.lua` as a program and exits; the second (no arguments) starts the interactive read-eval-print loop
- [ ] The first is invalid syntax; you must use `lua -f script.lua`
- [ ] The second only works if a `.luarc` config file is present in the current directory

<details>
<summary>Show Answer</summary>

**Answer:** B — The first executes `script.lua` as a program and exits; the second (no arguments) starts the interactive read-eval-print loop

**Explanation:** Passing a filename runs that chunk to completion and exits (unless the script itself starts a loop); running bare `lua` with no script argument drops you into the interactive prompt (`>`) for evaluating statements and expressions one at a time. Option A confuses `-i` (which runs a script AND then keeps the REPL open) with plain script execution; options C and D describe flags and config files that don't exist in stock Lua.

</details>
::

::question-wrapper
### Q4. What best distinguishes `lua.c` (the standalone interpreter) from "Lua" as a language and runtime?

- [ ] `lua.c` is required to compile any `.lua` file; without it Lua code cannot run at all
- [ ] `lua.c` is the only supported way to call Lua from C++ or other languages
- [ ] `lua.c` replaced the Lua VM starting in version 5.4
- [ ] `lua.c` is a thin, optional command-line front-end built on the same `liblua` C API that any host application uses to embed Lua — the language itself doesn't require a standalone binary

<details>
<summary>Show Answer</summary>

**Answer:** D — `lua.c` is a thin, optional command-line front-end built on the same `liblua` C API that any host application uses to embed Lua — the language itself doesn't require a standalone binary

**Explanation:** Lua's core is a C library (`liblua`) that exposes the Lua C API; `lua.c` is just a small sample client of that API that happens to ship as the `lua` command. Real-world embedding (games, editor configs, server scripting) links `liblua` directly and often never touches `lua.c`. Options A and B overstate the standalone binary's necessity, and option C is fabricated — the VM has evolved across versions, but `lua.c` never "replaced" it.

</details>
::

::question-wrapper{language="lua"}
### Q5. This script uses a `goto`/label pair. What happens when you run it with Lua 5.1 vs Lua 5.4?

::code-wrapper{language="lua"}
```lua
local i = 1
::top::
print(i)
i = i + 1
if i <= 3 then goto top end
```
::

- [ ] It behaves identically on both — `goto` has existed since Lua 5.0
- [ ] It runs fine on 5.4 but fails to parse on 5.1, because `goto`/labels were introduced in Lua 5.2
- [ ] It runs fine on 5.1 but fails on 5.4, because `goto` was removed in favor of `break` in 5.3
- [ ] Neither version supports `goto`; only LuaJIT does

<details>
<summary>Show Answer</summary>

**Answer:** B — It runs fine on 5.4 but fails to parse on 5.1, because `goto`/labels were introduced in Lua 5.2

**Explanation:** `goto` and `::label::` syntax were added in Lua 5.2; feeding this to a 5.1 interpreter (or most LuaJIT builds, which track roughly 5.1 syntax) raises a syntax error near `goto` or `::`. **Portability:** code using `goto` is a quick way to accidentally require 5.2+, so it's worth checking the target interpreter's version before relying on it. Option C and D are invented behaviors; option A ignores the real version boundary.

</details>
::

::question-wrapper{language="lua"}
### Q6. What does this print under Lua 5.4?

::code-wrapper{language="lua"}
```lua
print(7 / 2)
print(7 // 2)
print(7.0 // 2)
```
::

- [ ] `3.5  3  3.0`
- [ ] `3  3  3`
- [ ] `3.5  3.5  3.5`
- [ ] `4  3  3.0`

<details>
<summary>Show Answer</summary>

**Answer:** A — `3.5  3  3.0`

**Explanation:** `/` is always float division in Lua 5.3+, so `7 / 2` yields `3.5`. `//` is floor division, also introduced in 5.3, which preserves the "more float-like" operand's type: two integer operands yield an integer floor (`7 // 2` is `3`), but if either operand is a float the result is a float floor (`7.0 // 2` is `3.0`). Option B wrongly assumes `/` truncates like C integer division; option D wrongly rounds instead of flooring, and also gets the type wrong for the first result.

</details>
::

::question-wrapper{language="lua"}
### Q7. What happens when this runs on Lua 5.4?

::code-wrapper{language="lua"}
```lua
local MAX_RETRIES <const> = 5
MAX_RETRIES = MAX_RETRIES + 1
```
::

- [ ] It runs fine; `<const>` is just a linting hint with no runtime effect
- [ ] It raises a runtime error only if the reassignment line actually executes inside a loop
- [ ] It raises a compile-time error: attempt to assign to a `<const>` variable
- [ ] `<const>` is only valid on function parameters, so this is a syntax error on the first line

<details>
<summary>Show Answer</summary>

**Answer:** C — It raises a compile-time error: attempt to assign to a `<const>` variable

**Explanation:** `<const>` attributes (Lua 5.4+) are enforced at compile time — the compiler rejects any later assignment to that name with an error like `attempt to assign to const variable 'MAX_RETRIES'` before the script even runs. Option A underestimates it (it's not merely advisory); option B wrongly treats it as a runtime-only, control-flow-dependent check; option D is false — `<const>` attaches to `local` declarations, not to parameters.

</details>
::

::question-wrapper{language="lua"}
### Q8. What is the purpose of the `<close>` attribute in this Lua 5.4 snippet?

::code-wrapper{language="lua"}
```lua
local function open_log()
  local f <close> = io.open("app.log", "w")
  f:write("started\n")
end
```
::

- [ ] It makes the variable read-only for the rest of the block, identical to `<const>`
- [ ] It closes over the enclosing function's upvalues to prevent memory leaks
- [ ] It is purely cosmetic documentation with no effect until Lua 5.5
- [ ] It marks the value as to-be-closed: when the variable goes out of scope (normally or via error), Lua automatically calls its `__close` metamethod, similar to RAII/`defer`

<details>
<summary>Show Answer</summary>

**Answer:** D — It marks the value as to-be-closed: when the variable goes out of scope (normally or via error), Lua automatically calls its `__close` metamethod, similar to RAII/`defer`

**Explanation:** `<close>` (5.4+) guarantees the value's `__close` metamethod runs when the variable leaves scope — including during error unwinding — giving deterministic cleanup (file handles, locks) without a manual `pcall`-and-cleanup dance. Option A confuses it with the unrelated `<const>` attribute; option B misapplies the term "closure" to something it doesn't mean here; option C is false — it is fully functional in 5.4, not a no-op reserved for a future version.

</details>
::

::question-wrapper{language="lua"}
### Q9. You run this snippet with Lua 5.1. What happens?

::code-wrapper{language="lua"}
```lua
local flags = 6
local mask = 2
print(flags & mask)
```
::

- [ ] Prints `2`, since `&` is bitwise AND and has worked the same way since Lua 5.0
- [ ] Raises a syntax error, because native bitwise operators (`&`, `|`, `~`, `<<`, `>>`) were only added in Lua 5.3
- [ ] Prints `4`, because `&` is silently reinterpreted as string concatenation
- [ ] Works, but only if you `require("bit32")` first

<details>
<summary>Show Answer</summary>

**Answer:** B — Raises a syntax error, because native bitwise operators (`&`, `|`, `~`, `<<`, `>>`) were only added in Lua 5.3

**Explanation:** Bitwise operator syntax is new to Lua 5.3; on 5.1 (and LuaJIT's default dialect) the parser doesn't recognize `&` as an operator at all, so it throws a syntax error. Lua 5.1 has no built-in bitwise support; Lua 5.2 added the `bit32` *library* (function calls like `bit32.band`), not the operator syntax, so option D conflates a different version's library with this version's operator — and the operator still wouldn't parse regardless of any `require`. Options A and C describe fabricated behaviors.

</details>
::

::question-wrapper{language="lua"}
### Q10. Given the table below, what does `print(t[0], t[1], t[3])` output?

::code-wrapper{language="lua"}
```lua
local t = {10, 20, 30}
print(t[0], t[1], t[3])
```
::

- [ ] `nil  10  30` — Lua arrays/tables are conventionally 1-indexed, so `t[1]` is the first element and `t[0]` is simply an unset key
- [ ] `10  20  30` — indexing starts at 0 like most C-family languages
- [ ] An "index out of range" runtime error on `t[0]`
- [ ] `nil  nil  20` — because `{10, 20, 30}` builds the table in reverse

<details>
<summary>Show Answer</summary>

**Answer:** A — `nil  10  30` — Lua arrays/tables are conventionally 1-indexed, so `t[1]` is the first element and `t[0]` is simply an unset key

**Explanation:** The table constructor `{10, 20, 30}` assigns sequential integer keys starting at 1 (`t[1]=10, t[2]=20, t[3]=30`); `t[0]` was never set, so indexing it just returns `nil` — Lua tables don't bounds-check or error on missing keys, they simply return `nil`. This 1-based convention is a deliberate, pervasive design choice (matching `string.sub`, `ipairs`, `table.insert`, and friends), not an off-by-one bug. Option B wrongly assumes C-style 0-indexing; option C invents an error Lua doesn't raise for plain table access; option D is not how table constructors work.

</details>
::

::question-wrapper{language="bash"}
### Q11. You compile a script with Lua 5.4's `luac` and then try to run the output on a machine that only has Lua 5.1 installed:

::code-wrapper{language="bash"}
```bash
luac5.4 -o app.luac app.lua
lua5.1 app.luac
```
::

- [ ] It runs correctly; Lua bytecode has been stable and cross-version compatible since 5.0
- [ ] It automatically falls back to interpreting `app.luac` as source text
- [ ] It fails — precompiled Lua bytecode is tied to the exact Lua version (and often platform/word-size) it was compiled for, so 5.4 bytecode is not portable to a 5.1 VM
- [ ] It works, but only for scripts that don't use functions

<details>
<summary>Show Answer</summary>

**Answer:** C — It fails — precompiled Lua bytecode is tied to the exact Lua version (and often platform/word-size) it was compiled for, so 5.4 bytecode is not portable to a 5.1 VM

**Explanation:** **Portability:** the Lua bytecode format changes between major versions (and can vary by platform, e.g. integer size), so `luac`-compiled chunks are only guaranteed to load on a matching Lua version/build — running 5.4 bytecode on a 5.1 `lua` typically fails with a "bad header" or version-mismatch load error. Option A is a common but false assumption carried over from source-level compatibility; options B and D describe behavior Lua doesn't have — a `.luac` file is binary, not valid Lua source, so it can't be reinterpreted as text.

</details>
::

::question-wrapper{language="lua"}
### Q12. You need to comment out this block, which itself contains a `]]` sequence inside a string. Which statement about the comment form shown is correct?

::code-wrapper{language="lua"}
```lua
--[[
local msg = "use [[double brackets]] for long strings"
print(msg)
]]
```
::

- [ ] The form shown above works fine; Lua ignores nested `]]` inside a `--[[ ... ]]` comment
- [ ] You must escape the inner brackets as `\]\]` for this to work
- [ ] Block comments in Lua don't support multi-line content at all; only `--` is allowed
- [ ] It's a syntax error / the comment closes early — the first `]]` inside the string prematurely ends the comment, so use a longer bracket level like `--[==[ ... ]==]` instead

<details>
<summary>Show Answer</summary>

**Answer:** D — It's a syntax error / the comment closes early — the first `]]` inside the string prematurely ends the comment, so use a longer bracket level like `--[==[ ... ]==]` instead

**Explanation:** `--[[ ... ]]` closes at the *first* `]]` it encounters, regardless of context, so this comment actually ends right after `[[double brackets` and the remaining `for long strings"`, `print(msg)`, and the trailing `]]` are left as broken code. The fix is a higher long-bracket level — `--[==[ ... ]==]` — whose closing delimiter must match the exact number of `=` signs, so an inner `]]` no longer terminates it. **Debug:** this is a classic Lua gotcha when commenting out code that itself contains long-bracket strings. Option A is the wrong-but-tempting assumption; option B invents an escape mechanism long brackets don't use; option C is false, since block comments explicitly exist for multi-line content.

</details>
::

::question-wrapper{language="lua"}
### Q13. What happens when you try to run this?

::code-wrapper{language="lua"}
```lua
local end = 10
print(end)
```
::

- [ ] It runs fine and prints `10`; `end` is only special inside `if`/`for`/`function` blocks
- [ ] It raises a syntax error, because `end` is a reserved keyword and cannot be used as an identifier anywhere
- [ ] It's allowed, but only in Lua 5.1 for backward compatibility with 4.x
- [ ] It runs, but `end` silently becomes a global instead of a local

<details>
<summary>Show Answer</summary>

**Answer:** B — It raises a syntax error, because `end` is a reserved keyword and cannot be used as an identifier anywhere

**Explanation:** Lua has a fixed, case-sensitive set of reserved keywords (`and, break, do, else, elseif, end, false, for, function, goto, if, in, local, nil, not, or, repeat, return, then, true, until, while`) that can never be used as identifiers, in any scope, in any version — the parser rejects `local end = 10` immediately. Note that `End` or `END` *would* be legal identifiers, since Lua is case-sensitive and only the exact lowercase keyword is reserved. Option A wrongly assumes context-sensitivity Lua doesn't have; options C and D invent exceptions that don't exist.

</details>
::

::question-wrapper{language="lua"}
### Q14. What does this print?

::code-wrapper{language="lua"}
```lua
print(nil, true, 10, "hi")
```
::

- [ ] `nil true 10 hi` with each argument converted via `tostring()` and separated by a tab character
- [ ] An error, because `print` can't accept mixed types in one call
- [ ] Only `hi` is printed; `print` in Lua only shows its last argument
- [ ] `nil, true, 10, "hi"` including the commas and quotes, since Lua prints the literal source

<details>
<summary>Show Answer</summary>

**Answer:** A — `nil true 10 hi` with each argument converted via `tostring()` and separated by a tab character

**Explanation:** `print` calls `tostring()` on every argument it receives and writes them to stdout separated by tab characters, ending with a newline — so `nil` becomes the string `"nil"`, `true` becomes `"true"`, and no quotes are added around string arguments. Options B and C invent restrictions `print` doesn't have, since it's variadic and prints every argument it's given; option D confuses `print`'s runtime output with Lua source-code literal syntax.

</details>
::

::question-wrapper
### Q15. Compared to ecosystems like Node.js (`npm`) or Python (`pip`), what should you expect out-of-the-box from a fresh Lua installation regarding package management?

- [ ] Lua ships with a bundled package manager called `luaget` since 5.3
- [ ] `require()` automatically downloads missing modules from a central registry the first time they're used
- [ ] There is no bundled package manager — `require()` only loads modules already present on disk/`package.path`; LuaRocks is the de-facto community package manager but must be installed separately
- [ ] Package management is handled entirely by the operating system's package manager (apt/brew), and Lua has no module system of its own

<details>
<summary>Show Answer</summary>

**Answer:** C — There is no bundled package manager — `require()` only loads modules already present on disk/`package.path`; LuaRocks is the de-facto community package manager but must be installed separately

**Explanation:** Historically Lua ships lean and dependency-free; `require()` is purely a local module loader that searches `package.path`/`package.cpath`, with zero networking involved. LuaRocks fills the npm/pip role but is a separate install, which trips up newcomers who expect `npm install`-style tooling to exist by default. Option A invents a nonexistent tool; option B describes behavior `require` has never had; option D understates that Lua does have its own `require`/module mechanism independent of the OS.

</details>
::

::question-wrapper{language="lua"}
### Q16. Which statement about the following code is correct?

::code-wrapper{language="lua"}
```lua
local a = 1
local b = 2;
local c = 3;;
print(a + b + c)
```
::

- [ ] Only the first two lines are valid; `local c = 3;;` is a syntax error from the double semicolon
- [ ] The double semicolon on line 3 causes `c` to be declared twice, triggering a "variable already defined" error
- [ ] Semicolons are required after every statement in Lua 5.4, unlike earlier versions, so lines 1 and 3 are actually errors
- [ ] All lines are valid — semicolons are optional statement separators in Lua and can even appear as standalone empty statements, so `;;` is harmless

<details>
<summary>Show Answer</summary>

**Answer:** D — All lines are valid — semicolons are optional statement separators in Lua and can even appear as standalone empty statements, so `;;` is harmless

**Explanation:** Lua treats `;` as an optional statement separator; an empty statement (just `;`) is legal and does nothing, so `local c = 3;;` is simply `local c = 3` followed by a no-op empty statement. **Idiom:** most style guides recommend omitting semicolons except where needed to disambiguate (e.g. before a line starting with `(`), but including them is never wrong. Options A and B invent errors that don't occur; option C is false in every Lua version, since semicolons have never been mandatory.

</details>
::

::question-wrapper{language="lua"}
### Q17. This function has a subtle bug. What is it?

::code-wrapper{language="lua"}
```lua
function computeTotal(items)
  total = 0
  for i = 1, #items do
    total = total + items[i]
  end
  return total
end
```
::

- [ ] There is no bug; `total` is correctly scoped to the function since it's assigned inside it
- [ ] `total` is missing a `local` declaration, so it's created as a global variable — it can silently collide with other code using the same name
- [ ] `#items` is invalid syntax; length must be computed with `items.length`
- [ ] The `for` loop should use `ipairs(items)` instead, or it won't iterate at all

<details>
<summary>Show Answer</summary>

**Answer:** B — `total` is missing a `local` declaration, so it's created as a global variable — it can silently collide with other code using the same name

**Explanation:** Lua's default assignment behavior is global-by-default — `total = 0` inside the function, without `local`, creates (or overwrites) a global variable, not a function-local one. It "happens to work" here because it's reset to 0 on every call, but it pollutes the global namespace and could clash with an unrelated global named `total` elsewhere in a large embedded application; the fix is `local total = 0`. Option A misses the footgun precisely because the code superficially works; option C is false, since `#` is Lua's built-in length operator; option D is a style preference, not a bug, since the numeric `for` loop shown works fine for a dense array-like table.

</details>
::

::question-wrapper{language="lua"}
### Q18. What does this print on a standard Lua 5.4 build?

::code-wrapper{language="lua"}
```lua
print(math.type(1), math.type(1.0), 1 == 1.0)
```
::

- [ ] `integer  float  true` — 5.4 distinguishes integer and float subtypes internally, but `==` still compares them by mathematical value
- [ ] `number  number  true` — Lua only has one numeric type, so `math.type` always returns `"number"`
- [ ] `integer  float  false` — different subtypes are never considered equal
- [ ] This errors, because `math.type` doesn't exist until Lua 5.5

<details>
<summary>Show Answer</summary>

**Answer:** A — `integer  float  true` — 5.4 distinguishes integer and float subtypes internally, but `==` still compares them by mathematical value

**Explanation:** Since Lua 5.3, numbers have two subtypes — integer and float — distinguishable via `math.type()`, but arithmetic and comparison operators still treat them as one unified "number" for equality purposes, so `1 == 1.0` is `true` even though their subtypes differ (`"integer"` vs `"float"`). **Portability:** this subtype distinction doesn't exist in Lua 5.1/5.2 or in LuaJIT's default number model, where all numbers are doubles and `math.type` isn't available, so code relying on it silently breaks on those runtimes. Option B describes pre-5.3 behavior; option C wrongly assumes subtype mismatch breaks numeric equality; option D is fabricated, since `math.type` has existed since 5.3.

</details>
::

::question-wrapper{language="lua"}
### Q19. In a modern (5.3+) interactive `lua` session, what's the difference between typing these two entries at the prompt?

::code-wrapper{language="lua"}
```lua
> 2 + 2
> print(2 + 2)
```
::

- [ ] They behave differently: the first is a syntax error because bare expressions aren't valid statements, so you must always call `print`
- [ ] The first only works in LuaJIT's REPL, not in standard Lua
- [ ] They behave the same way: the standalone expression `2 + 2` is auto-detected by the REPL, implicitly wrapped as `return 2 + 2`, and its result is printed, same visible output as the explicit `print` call
- [ ] The first silently discards the result and prints nothing, while only the second shows `4`

<details>
<summary>Show Answer</summary>

**Answer:** C — They behave the same way: the standalone expression `2 + 2` is auto-detected by the REPL, implicitly wrapped as `return 2 + 2`, and its result is printed, same visible output as the explicit `print` call

**Explanation:** Since Lua 5.2's `lua.c`, when a line typed at the interactive prompt doesn't parse as a valid statement, the REPL retries it prefixed with `return`, then prints any returned values — so typing a bare expression is a convenient shortcut that ends up displaying the same `4` as an explicit `print(2 + 2)`. Older 5.1-style REPLs instead required an explicit `=2+2` shortcut for this. Option A wrongly treats the shortcut as invalid; option B is false, since this is standard `lua.c` behavior, not LuaJIT-specific; option D invents silent-discard behavior that doesn't happen.

</details>
::

::question-wrapper{language="lua"}
### Q20. You're embedding a scripting layer into a performance-sensitive C application, and the scripts must use this pattern unmodified:

::code-wrapper{language="lua"}
```lua
local flags <const> = 0xF0 & 0x3C
local step = 10 // 3
```
::

- [ ] LuaJIT, because it's always faster and a strict superset of PUC-Lua
- [ ] Lua 5.1, since bitwise and floor-division support was available from the very first release
- [ ] Any Lua version works identically, since these are all part of core Lua syntax since 5.0
- [ ] Lua 5.3 or newer (e.g. 5.4), because `<const>`, native bitwise operators, and `//` all require 5.3+ language support that LuaJIT's default dialect doesn't provide

<details>
<summary>Show Answer</summary>

**Answer:** D — Lua 5.3 or newer (e.g. 5.4), because `<const>`, native bitwise operators, and `//` all require 5.3+ language support that LuaJIT's default dialect doesn't provide

**Explanation:** `<const>` is 5.4-only, while `&` and `//` require 5.3+; LuaJIT's mainline branch tracks roughly 5.1 syntax plus a few extras and does not parse this snippet, so despite LuaJIT's raw speed advantage, it's the wrong choice when the requirement is running this specific 5.3+/5.4 syntax unmodified — a real-world case where "faster" and "compatible" pull in different directions. Option A is the tempting-but-wrong performance-first answer; options B and C are simply factually wrong about when these features were introduced.

</details>
::
