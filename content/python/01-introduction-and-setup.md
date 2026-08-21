# 01 — Introduction & Setup

::question-wrapper
### Q1. What is CPython?

- [ ] It is a third-party JIT-compiled alternative to Python
- [ ] It is the reference implementation of Python, written primarily in C
- [ ] It is a tool that transpiles Python source into C++ for compilation
- [ ] It is the deprecated Python 2 interpreter, superseded by Python 3

<details>
<summary>Show Answer</summary>

**Answer:** B — It is the reference implementation of Python, written primarily in C

**Explanation:** CPython is the original and most widely used Python interpreter, implemented in C, and it is what most people mean when they say "Python." Option A describes PyPy, a different implementation. Option C misdescribes CPython — it compiles Python source to bytecode executed by a virtual machine, not to C++. Option D is wrong because CPython supports both historical Python 2 releases and current Python 3 releases; it isn't tied to one language version.

</details>
::

::question-wrapper
### Q2. A team runs a long-lived, CPU-bound pure-Python service and wants a drop-in interpreter swap for better throughput. Which alternative implementation is specifically known for using a JIT compiler to speed up such workloads?

- [ ] Jython
- [ ] PyPy
- [ ] IronPython
- [ ] MicroPython

<details>
<summary>Show Answer</summary>

**Answer:** B — PyPy

**Explanation:** PyPy includes a just-in-time compiler that can dramatically speed up long-running pure-Python code by compiling hot code paths to machine code at runtime. Jython (A) targets the JVM and IronPython (C) targets .NET — both let Python interoperate with those platforms but neither is primarily a JIT speed play for pure-Python workloads. MicroPython (D) is a lean reimplementation for microcontrollers, prioritizing footprint over raw throughput.

</details>
::

::question-wrapper{language="python"}
### Q3. In Python 3, how must `print` be used?

::code-wrapper{language="python"}
```python
print("deployment finished")
```
::

- [ ] As a statement, without parentheses: `print "deployment finished"`
- [ ] As a function call, with parentheses: `print("deployment finished")`
- [ ] Only via `sys.stdout.write`, since `print` was removed in Python 3
- [ ] Either form works interchangeably in Python 3

<details>
<summary>Show Answer</summary>

**Answer:** B — As a function call, with parentheses: `print("deployment finished")`

**Explanation:** Python 3 turned `print` into a regular builtin function, so it must be called with parentheses. Option A is the Python 2 statement form and raises a `SyntaxError` under Python 3. Option C is false — `print` still exists as a convenient wrapper; you are not forced onto `sys.stdout.write`. Option D is wrong because the statement form is not valid Python 3 syntax at all.

</details>
::

::question-wrapper{language="python"}
### Q4. What does `/` (true division) return in Python 3 when dividing two integers?

::code-wrapper{language="python"}
```python
result = 7 / 2
```
::

- [ ] `3`, an integer truncated toward zero, matching Python 2's `/`
- [ ] `3.5`, a float — Python 3's `/` always performs true division
- [ ] A `TypeError`, because `/` requires float operands explicitly
- [ ] `3`, an integer floored toward negative infinity

<details>
<summary>Show Answer</summary>

**Answer:** B — `3.5`, a float — Python 3's `/` always performs true division

**Explanation:** In Python 3, `/` always performs true division and returns a `float`, even when both operands are `int`. This is a deliberate change from Python 2, where `/` between two ints performed floor division by default. Option A describes the old Python 2 behavior. Option D describes what `//` (floor division) does, not `/`. Option C is wrong — `/` works fine on ints and simply promotes the result to `float`.

</details>
::

::question-wrapper
### Q5. What does a `.pyc` file contain?

- [ ] Native machine code compiled for the host CPU
- [ ] Compiled Python bytecode, cached to speed up future imports of that module
- [ ] A minified copy of the original `.py` source text
- [ ] An encrypted, obfuscated version of the source code

<details>
<summary>Show Answer</summary>

**Answer:** B — Compiled Python bytecode, cached to speed up future imports of that module

**Explanation:** `.pyc` files hold the compiled bytecode for a module so that CPython can skip re-parsing and re-compiling the source text on subsequent imports, speeding up startup. It is not native machine code (A) — CPython's bytecode still runs on the interpreter's virtual machine. It is not a text-based minified copy (C), and it provides no meaningful security or obfuscation (D); bytecode can be decompiled fairly easily.

</details>
::

::question-wrapper
### Q6. Where does CPython 3 store cached bytecode for an imported module named `utils.py`, by default?

- [ ] Inside a `__pycache__/` directory next to the source, e.g. `__pycache__/utils.cpython-311.pyc`
- [ ] As `utils.pyc`, directly replacing `utils.py` in place
- [ ] In the interpreter's installation directory, keyed by project name
- [ ] In a single, project-wide `compiled.pyc` bundle

<details>
<summary>Show Answer</summary>

**Answer:** A — Inside a `__pycache__/` directory next to the source, e.g. `__pycache__/utils.cpython-311.pyc`

**Explanation:** Since Python 3.2 (PEP 3147), cached bytecode lives in a `__pycache__` subdirectory alongside the source, with a filename tagged by interpreter implementation and version (e.g., `cpython-311`). Option B describes the old Python 2 layout, where `.pyc` sat next to `.py` with no subfolder. Options C and D describe caching schemes CPython does not use.

</details>
::

::question-wrapper
### Q7. Per PEP 8, how many spaces should be used for each indentation level?

- [ ] 2 spaces
- [ ] 4 spaces
- [ ] A single hard tab character
- [ ] 8 spaces

<details>
<summary>Show Answer</summary>

**Answer:** B — 4 spaces

**Explanation:** PEP 8 recommends 4 spaces per indentation level and explicitly recommends spaces over tabs. Options A and D are used by some other languages/style guides but are not the PEP 8 recommendation for Python. Option C is discouraged: PEP 8 says tabs and spaces should not be mixed, and spaces are preferred; Python 3 even raises a `TabError` if a file inconsistently mixes tabs and spaces in a way that changes meaning.

</details>
::

::question-wrapper{language="bash"}
### Q8. At the interactive REPL, what does the special name `_` refer to immediately after evaluating an expression?

::code-wrapper{language="bash"}
```bash
>>> 40 + 2
42
>>> _
```
::

- [ ] The value of the last expression evaluated at the prompt (here, `42`)
- [ ] A `SyntaxError`, since `_` has no special meaning in Python
- [ ] The current line number in the session
- [ ] The name of the most recently called function

<details>
<summary>Show Answer</summary>

**Answer:** A — The value of the last expression evaluated at the prompt (here, `42`)

**Explanation:** **Debug** — the interactive interpreter automatically binds `_` to the result of the last statement that produced a displayed value, which is convenient for chaining exploratory work at the prompt. This behavior is unique to the interactive REPL; it does not happen when running a `.py` file as a script, which is the trap — code relying on `_` outside the REPL will raise a `NameError` (unless `_` was assigned some other way, e.g. as the conventional "throwaway" variable name).

</details>
::

::question-wrapper
### Q9. What determines whether CPython recompiles a module instead of reusing its cached `.pyc`, under the default (non hash-based) invalidation mode?

- [ ] CPython compares the source file's modification time and size, embedded in the `.pyc` header, against the current source file
- [ ] CPython recompiles on every run regardless of any cache, since caching is opt-in and off by default
- [ ] CPython caches bytecode permanently and never re-checks the source afterward
- [ ] CPython computes a SHA-256 hash of the full source on every import and compares it, ignoring timestamps entirely

<details>
<summary>Show Answer</summary>

**Answer:** A — CPython compares the source file's modification time and size, embedded in the `.pyc` header, against the current source file

**Explanation:** By default, CPython uses timestamp-based invalidation: the `.pyc` header stores the source's mtime and size, and a mismatch triggers recompilation. Option D describes hash-based `.pyc` invalidation, a real but opt-in mode added in PEP 552 (Python 3.7+) — it exists, but it is not the default, which is the gotcha. Option B is wrong because caching is on by default for imported modules. Option C is wrong because stale caches are actively detected, not blindly trusted forever.

</details>
::

::question-wrapper
### Q10. Does the script you run directly (e.g., `python app.py`) get its own `.pyc` written to `__pycache__`?

- [ ] Yes — every executed file is cached identically, including the entry-point script
- [ ] No — only imported modules are cached; the file run directly as `__main__` is compiled fresh each run and not cached
- [ ] Yes, but only when the `-O` flag is passed
- [ ] No — CPython never caches bytecode for any file, despite common belief

<details>
<summary>Show Answer</summary>

**Answer:** B — No — only imported modules are cached; the file run directly as `__main__` is compiled fresh each run and not cached

**Explanation:** **Performance** — CPython caches bytecode for modules reached via the import system, but the top-level script executed as `__main__` is recompiled every invocation, since there's no stable cache-invalidation target the way there is for an importable module. This surprises developers who expect a `__pycache__/app.cpython-311.pyc` to appear next to `app.py` after running it directly — it won't, though any modules `app.py` imports will get cached.

</details>
::

::question-wrapper
### Q11. Two different Python versions, 3.9 and 3.11, both import the same `utils.py` from a shared `__pycache__` directory. What happens?

- [ ] A version-conflict `ImportError` is raised immediately
- [ ] Each interpreter reads and writes its own version-tagged file (`utils.cpython-39.pyc` and `utils.cpython-311.pyc`), so both coexist safely
- [ ] The second interpreter to run overwrites and corrupts the first interpreter's cache entry
- [ ] Only one Python version may be installed on a machine at a time, so this scenario cannot occur

<details>
<summary>Show Answer</summary>

**Answer:** B — Each interpreter reads and writes its own version-tagged file (`utils.cpython-39.pyc` and `utils.cpython-311.pyc`), so both coexist safely

**Explanation:** **Portability** — the PEP 3147 naming scheme embeds the implementation and version tag in the cache filename specifically so multiple interpreters can safely share one `__pycache__` directory without clobbering each other's bytecode. Options A, C, and D describe conflicts that the tagging scheme was designed to prevent; multiple Python versions coexisting on one machine (e.g., via `pyenv` or system packages) is routine.

</details>
::

::question-wrapper
### Q12. A developer deletes a stray `__pycache__` directory before committing a project to git. Is this safe?

- [ ] No — `.pyc` files sometimes contain unique logic not present in the `.py` source, so deleting them causes data loss
- [ ] Yes — `__pycache__` is a disposable, regeneratable cache; it is generally recommended to `.gitignore` it rather than commit it
- [ ] No — deleting it forces a full reinstall of the Python interpreter
- [ ] Yes, but only because the interpreter falls back to Python 2 mode without a valid cache

<details>
<summary>Show Answer</summary>

**Answer:** B — Yes — `__pycache__` is a disposable, regeneratable cache; it is generally recommended to `.gitignore` it rather than commit it

**Explanation:** `.pyc` files are purely derived artifacts recompiled automatically from `.py` source as needed, so removing them is always safe and they are conventionally excluded from version control. Option A is a myth — bytecode caches never contain logic absent from the source they were compiled from. Options C and D describe consequences that simply do not occur; deleting a cache directory has zero effect on the interpreter installation itself.

</details>
::

::question-wrapper
### Q13. Which statement correctly contrasts `str` between Python 2 and Python 3?

- [ ] In Python 2, `str` held Unicode code points by default; Python 3's `str` became raw bytes
- [ ] In Python 2, `str` was a byte sequence by default (with a separate `unicode` type for text); in Python 3, `str` became a sequence of Unicode code points, and raw bytes moved to the distinct `bytes` type
- [ ] Both versions treat `str` identically as raw, undecoded bytes
- [ ] Python 3 removed the text/bytes distinction entirely, merging both into one type

<details>
<summary>Show Answer</summary>

**Answer:** B — In Python 2, `str` was a byte sequence by default (with a separate `unicode` type for text); in Python 3, `str` became a sequence of Unicode code points, and raw bytes moved to the distinct `bytes` type

**Explanation:** This is one of the largest Python 2 → 3 breaking changes: Python 2's `str` was really a byte string (with `unicode` as the separate text type), while Python 3 flips this so `str` is text (Unicode) and `bytes` is the explicit binary type, with no implicit coercion between them. Option A states the reverse of the truth. Options C and D understate a distinction that Python 3 in fact made stricter, not looser — mixing `str` and `bytes` in Python 3 raises a `TypeError` rather than silently coercing.

</details>
::

::question-wrapper{language="bash"}
### Q14. What is the effect of invoking the interpreter with the `-O` flag, e.g. `python -O app.py`?

::code-wrapper{language="bash"}
```bash
python -O app.py
```
::

- [ ] It enables basic optimizations — `assert` statements and code guarded by `if __debug__:` are stripped, and cache files are tagged `opt-1.pyc`
- [ ] It automatically reformats the source file to comply with PEP 8
- [ ] It runs the script inside an online, network-sandboxed mode
- [ ] It has no effect under CPython and is only meaningful under Jython

<details>
<summary>Show Answer</summary>

**Answer:** A — It enables basic optimizations — `assert` statements and code guarded by `if __debug__:` are stripped, and cache files are tagged `opt-1.pyc`

**Explanation:** **Debug** — `-O` sets `__debug__` to `False`, causing `assert` statements (and any `if __debug__:` blocks) to be compiled out entirely — a common trap because assertions silently stop firing in `-O` mode, so they must never be relied on for input validation or security checks in production. Options B and C describe behavior `-O` does not have. Option D is wrong — `-O` is a genuine, commonly used CPython flag.

</details>
::

::question-wrapper
### Q15. Per PEP 8, what is the recommended maximum line length for code?

- [ ] 79 characters, with slightly more latitude (up to ~72) suggested for flowing text like docstrings and comments
- [ ] 120 characters, strictly enforced by the interpreter
- [ ] There is no guidance at all; PEP 8 leaves line length entirely to team preference
- [ ] 40 characters

<details>
<summary>Show Answer</summary>

**Answer:** A — 79 characters, with slightly more latitude (up to ~72) suggested for flowing text like docstrings and comments

**Explanation:** **Idiom** — PEP 8 recommends limiting lines to 79 characters (with long, flowing text such as comments/docstrings recommended to wrap around 72), specifically to support side-by-side diffs and multiple open files. Option B is a common team override but not the PEP 8 default. Option C understates PEP 8, which does give a concrete number even though many real-world projects relax it. Option D is far stricter than PEP 8 actually recommends.

</details>
::

::question-wrapper{language="python"}
### Q16. Per PEP 8 naming conventions, how should a module-level constant be named?

::code-wrapper{language="python"}
```python
MAX_RETRIES = 3
```
::

- [ ] camelCase, e.g. `maxRetries`
- [ ] UPPER_SNAKE_CASE, e.g. `MAX_RETRIES`
- [ ] PascalCase, e.g. `MaxRetries`
- [ ] Always underscore-prefixed, e.g. `_maxretries`

<details>
<summary>Show Answer</summary>

**Answer:** B — UPPER_SNAKE_CASE, e.g. `MAX_RETRIES`

**Explanation:** PEP 8 reserves UPPER_SNAKE_CASE for constants, snake_case for functions/variables, and PascalCase (CapWords) for classes, so `MAX_RETRIES` correctly signals "constant" to readers. Option A is the JavaScript convention, not Python's. Option C (PascalCase) is PEP 8's convention for class names, not constants. Option D's leading underscore signals "internal/non-public," an orthogonal concern to constant-ness.

</details>
::

::question-wrapper
### Q17. A team is starting a brand-new production codebase in 2026. Which major version should they target?

- [ ] Python 2, since it has been stable and unchanged for years
- [ ] Python 3 — Python 2 reached official end-of-life in January 2020 and no longer receives security patches
- [ ] Either is fine, since Python 2 and 3 source is fully interchangeable
- [ ] It's irrelevant — CPython auto-detects and transparently runs either syntax

<details>
<summary>Show Answer</summary>

**Answer:** B — Python 3 — Python 2 reached official end-of-life in January 2020 and no longer receives security patches

**Explanation:** **Safety** — Python 2's "stability" in option A is really abandonment: it stopped receiving even security fixes after January 1, 2020, making it a liability for new production work. Option C is a myth; Python 2 and 3 have real syntactic and semantic differences (print, integer division, string types, etc.) that require porting, not blind compatibility. Option D is false — a `python2`-only script (e.g., using the `print` statement) is a `SyntaxError` under a Python 3 interpreter.

</details>
::

::question-wrapper
### Q18. When is it appropriate to consider PyPy instead of CPython in production?

- [ ] For a long-running, CPU-bound, pure-Python workload where JIT warm-up time is acceptable and speed matters — PyPy can substantially outperform CPython there
- [ ] Always — PyPy is a strict superset of CPython with zero compatibility trade-offs, including every C-extension module
- [ ] Never — PyPy cannot execute real-world Python code
- [ ] Only for short-lived CLI scripts, since PyPy has effectively no startup cost

<details>
<summary>Show Answer</summary>

**Answer:** A — For a long-running, CPU-bound, pure-Python workload where JIT warm-up time is acceptable and speed matters — PyPy can substantially outperform CPython there

**Explanation:** **Performance** — PyPy's JIT needs time to warm up and profile hot paths before it pays off, so it shines on long-lived processes, not brief scripts (making option D backwards). Option B is the tempting-but-wrong choice: historically PyPy has had incomplete or slower support for CPython's C-extension API, so libraries with heavy C extensions (some numeric/scientific packages) are not automatically a free win — compatibility must be checked. Option C is simply false; PyPy runs the vast majority of pure-Python code correctly.

</details>
::

::question-wrapper{language="bash"}
### Q19. What is the recommended way to confirm exactly which Python interpreter and version will run a project, especially with multiple installs or virtual environments present?

::code-wrapper{language="bash"}
```bash
python --version
which python
```
::

- [ ] Run `python --version` and `which python` (or `where python` on Windows) to check both the version and the exact executable path in use
- [ ] Assume the system `python` command always resolves to Python 3, since Python 2 was removed from every OS by 2021
- [ ] Open a `.pyc` file in a text editor and read the version header
- [ ] Check the script's file extension — `.py` always means Python 3, `.py2` means Python 2

<details>
<summary>Show Answer</summary>

**Answer:** A — Run `python --version` and `which python` (or `where python` on Windows) to check both the version and the exact executable path in use

**Explanation:** **Idiom** — on many systems `python` may still resolve to Python 2, to a `pyenv` shim, or to a virtual environment's interpreter, so checking both the reported version and the resolved binary path avoids running against the wrong interpreter. Option B is a real-world trap — some Linux distributions and older macOS installs kept `python` pointing at Python 2 well past its EOL. Options C and D describe checks that don't actually work; `.py2` is not a recognized extension and `.pyc` is binary, not readable version metadata.

</details>
::

::question-wrapper{language="bash"}
### Q20. What is the idiomatic way to explore a script's own functions/classes interactively without retyping all of its definitions at the prompt?

::code-wrapper{language="bash"}
```bash
python -i script.py
```
::

- [ ] Manually retype every class and function definition at the `>>>` prompt each session
- [ ] Use `python -i script.py` (or an enhanced shell like IPython) to run the script and then drop into an interactive session with its namespace already loaded
- [ ] There is no way to combine a script with the REPL — they are mutually exclusive execution modes
- [ ] Rename the script to `.pyc` and import that file directly into the REPL

<details>
<summary>Show Answer</summary>

**Answer:** B — Use `python -i script.py` (or an enhanced shell like IPython) to run the script and then drop into an interactive session with its namespace already loaded

**Explanation:** **Idiom** — the `-i` flag executes the given script and then hands control to an interactive prompt with all of that script's top-level names already bound, which is the standard way to poke at real project state without retyping code. Option A works but wastes effort and invites transcription errors. Option C is simply false. Option D doesn't work as described — you don't "import" a `.pyc` by renaming a `.py`; the cache format is an implementation detail, not a user-facing import mechanism.

</details>
::
