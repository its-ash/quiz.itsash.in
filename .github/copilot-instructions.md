# Content Authoring Instructions

## Code Blocks

When creating or editing Markdown content files, always wrap fenced code blocks with the `::code-wrapper` MDC component:

```md
::code-wrapper{language="bash"}
```bash
rustc src/main.rs && ./main      # produces ./main (or main.exe)
```
::
```

### Rules

- Every `` ``` `` fenced code block MUST be wrapped with `::code-wrapper{language="<lang>"} ... ::`
- The `language` prop MUST match the fenced code block's language
- Optionally add a `filename` prop: `::code-wrapper{language="rust" filename="main.rs"}`
- Supported languages: `rust`, `python`, `javascript`, `typescript`, `bash`, `json`, `yaml`, `markdown`, `sql`, `toml`, `ini`, `xml`, `css`, `shell`, `dockerfile`
- Never use bare fenced code blocks without the `::code-wrapper` wrapper

## Question Format

Content files are quizzes, not quizing docs. Each file is a numbered list of multiple-choice questions.

Every question MUST be wrapped in the `::question-wrapper` MDC component — it renders the whole question (heading, options, and collapsed answer) as one bordered card, mirroring how `::code-wrapper` boxes a fenced code block.

```md
::question-wrapper{language="rust"}
### Q1. <question text>

- [ ] Option A
- [ ] Option B
- [ ] Option C
- [ ] Option D

<details>
<summary>Show Answer</summary>

**Answer:** C

**Explanation:** <why the correct option is right, and briefly why the other options are wrong>

</details>
::
```

If the question includes an embedded code snippet (still wrapped in its own nested `::code-wrapper` per the Code Blocks rules above), set `question-wrapper`'s `language` prop to that snippet's language so the card shows a language badge. Omit the `language` prop entirely for questions with no code.

### Rules

- Number questions sequentially per file: `Q1.`, `Q2.`, ...
- Wrap the entire question (heading through the closing `</details>`) in `::question-wrapper{language="<lang>"} ... ::`; set `language` only when the question contains an embedded code snippet, otherwise omit it
- Each question MUST have 4 options unless the concept is inherently binary (then 2, e.g. True/False)
- Options MUST use `- [ ]` checkbox syntax so the reader can mark a selection
- The answer + explanation MUST be wrapped in `<details><summary>Show Answer</summary> ... </details>` placed immediately after the options, so it stays collapsed until expanded
- The explanation MUST state why the correct option is right and briefly address why the main distractor(s) are wrong
- Prefer non-obvious, "gotcha" questions over straightforward ones — favor scenarios a quizer would plausibly get wrong on the first try
- Distractor options should be plausible mistakes (common misconceptions, off-by-one errors, type-coercion traps), not obviously-wrong filler
- Any code shown in a question MUST additionally be wrapped with `::code-wrapper` per the Code Blocks rules above, nested inside the `::question-wrapper`

## Index Files

When creating or editing content in a topic directory (e.g., `content/js/`, `content/rust/`):

- Always include an `index.md` file at the root of each topic directory (e.g., `content/js/index.md`, `content/rust/index.md`)
- The `index.md` acts as the landing page and table of contents for the topic, listing all quiz files in the directory with links and a brief description of what each quiz covers
- When a new quiz file is added to a directory, update the corresponding `index.md` to include it in the listing
- The `index.md` should group entries logically (e.g., by difficulty or subtopic) and include a short intro describing the overall quiz track

## Zero-to-Hero Quiz Curriculum

When generating a "zero to hero" quiz track for a language (e.g., `content/js/`, `content/rust/`), create a comprehensive set of quizzes covering **all features the language offers**, from absolute basics to advanced/expert topics:

- **Structure**: Numbered files (`01-`, `02-`, ...) ordered by progression — foundations → intermediate → advanced → expert
- **Comprehensive coverage**: Include every major language feature (syntax, types, control flow, functions, collections, OOP/structs, error handling, concurrency/async, modules, macros/meta-programming, testing, tooling, FFI, unsafe, etc.) as its own quiz file
- **Self-contained**: Each quiz's questions and explanations should fully cover its concept so a reader can quiz from the explanations alone
- **Real-world focus**: Base questions on production-grade scenarios, not toy snippets
- **Progressive difficulty**: Order questions within a file from basic recall to edge cases to best-practice judgment calls
- **Include an `index.md`**: Landing page with a grouped table (Parts/Phases), "How to Use This Quiz Track" section, prerequisites, and a curriculum table with `# | Topic | Why It Matters` columns linking each quiz file
- **Final files**: Include quizzes on testing, performance/optimization, tooling/build systems, security, and a capstone mixed-review quiz

## Content Quality

When writing quiz questions:

- Always include questions on **edge cases** — behavior at boundaries, empty inputs, maximum values, error states, and unusual but valid inputs
- Always include questions on **best practices** — idiomatic patterns, recommended approaches, and common pitfalls, framed as "which option is correct/idiomatic"
- Base questions on real-world scenarios, not toy snippets
- When a question covers an error case, the explanation MUST also state the correct way to handle it
- Order questions within a file logically — basic case first, then edge cases, then best-practice judgment calls

## Tricky Examples & Gotchas

Favor questions built around surprising behavior over straightforward recall:

- Prefer non-obvious, counterintuitive, "gotcha" questions — highlight surprising behavior, platform-specific quirks, silent failures, off-by-one traps, type coercion surprises, integer overflow, zero-copy pitfalls, or anything that can bite in production
- The explanation must state *why* the tricky case behaves the way it does, not just *that* it does — include the underlying mechanism (evaluation order, borrow rules, floating-point representation, type coercion table, etc.)
- When a language has a famous "wat" moment or common interview gotcha related to the topic, turn it into a question (e.g., JavaScript `[] + {}`, Rust `String` vs `&str` moves, Python mutable default args, floating-point `0.1 + 0.2`)
- Use a wrong-but-tempting option as one of the distractors (what a beginner would guess), and explain in the answer why it's wrong — this contrast makes the gotcha memorable
- Tag tricky questions with a short bold label in the explanation (e.g., **Performance**, **Safety**, **Idiom**, **Debug**, **Portability**) so readers can scan for what matters to them