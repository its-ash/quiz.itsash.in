# 11 — The `this` Keyword

::question-wrapper{language="javascript"}
### Q1. What does the following log?

::code-wrapper{language="javascript"}
```javascript
const user = {
  name: 'Ava',
  greet() {
    return `Hi, I'm ${this.name}`;
  }
};

console.log(user.greet());
```
::

- [ ] "Hi, I'm Ava"
- [ ] "Hi, I'm undefined"
- [ ] Throws a TypeError
- [ ] "Hi, I'm user"

<details>
<summary>Show Answer</summary>

**Answer:** A — "Hi, I'm Ava"

**Explanation:** This is **implicit binding**: when a function is called as a property of an object (`user.greet()`), `this` is bound to the object on the left of the dot at call time, not to where the function was defined. "Hi, I'm undefined" would only happen if `greet` were invoked without a receiver (e.g. destructured first). There's no reason for a TypeError here since `user.name` exists.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does this print in an ES module (strict mode)?

::code-wrapper{language="javascript"}
```javascript
function whoAmI() {
  return this;
}

console.log(whoAmI());
```
::

- [ ] undefined
- [ ] The global object (`globalThis`)
- [ ] An empty object `{}`
- [ ] The `whoAmI` function itself

<details>
<summary>Show Answer</summary>

**Answer:** A — undefined

**Explanation:** This is **default binding**. When a function is called with no receiver and no explicit binding, `this` falls back to the global object in sloppy mode, but ES modules (and any code under `'use strict'`) run in strict mode by default, where the fallback is `undefined` instead. The "global object" answer is a very common trap left over from sloppy-mode intuition.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What does `call` do here?

::code-wrapper{language="javascript"}
```javascript
function intro() {
  return `${this.role}: ${this.name}`;
}

const emp = { name: 'Sam', role: 'Dev' };
console.log(intro.call(emp));
```
::

- [ ] "Dev: Sam"
- [ ] "undefined: undefined"
- [ ] Throws a TypeError because `intro` isn't a method of `emp`
- [ ] "Sam: Dev"

<details>
<summary>Show Answer</summary>

**Answer:** A — "Dev: Sam"

**Explanation:** `Function.prototype.call(thisArg, ...args)` invokes the function immediately with `this` **explicitly** set to `thisArg`. This is **explicit binding** — it doesn't matter that `intro` was never attached to `emp`; `call` temporarily makes `emp` the receiver. The "Sam: Dev" option swaps the template order, which isn't how the string is built.

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does this log?

::code-wrapper{language="javascript"}
```javascript
function total(a, b) {
  return this.base + a + b;
}

console.log(total.apply({ base: 10 }, [2, 3]));
```
::

- [ ] 15
- [ ] NaN
- [ ] Throws a TypeError because arrays aren't valid arguments
- [ ] 5

<details>
<summary>Show Answer</summary>

**Answer:** A — 15

**Explanation:** `apply(thisArg, argsArray)` behaves exactly like `call`, except positional arguments are supplied as an array, which `apply` spreads into `a` and `b`. So `this.base` is `10`, `a` is `2`, `b` is `3`, giving `10 + 2 + 3 = 15`. `apply` accepting an array-like for arguments is its entire purpose — it doesn't throw.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What is logged after 100ms?

::code-wrapper{language="javascript"}
```javascript
const user = { name: 'Ava', greet() { return `Hi, I'm ${this.name}`; } };
const greetLater = user.greet.bind(user);

setTimeout(() => console.log(greetLater()), 100);
```
::

- [ ] "Hi, I'm Ava"
- [ ] "Hi, I'm undefined"
- [ ] The string is logged immediately, not after 100ms
- [ ] Throws a TypeError because `bind` already executed `greet`

<details>
<summary>Show Answer</summary>

**Answer:** A — "Hi, I'm Ava"

**Explanation:** `bind` does **not** call the function — it returns a new function with `this` permanently locked to the given value, to be invoked later. That's the trap in option C/D: beginners often confuse `bind` (defer + fix `this`) with `call`/`apply` (invoke now). Since `greetLater` carries `user` as its bound receiver no matter how or when it's called, it still logs "Hi, I'm Ava" once the timer fires.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What does this log?

::code-wrapper{language="javascript"}
```javascript
const counter = {
  count: 0,
  increment: () => {
    this.count++;
    return this.count;
  }
};

console.log(counter.increment());
```
::

- [ ] 1
- [ ] NaN
- [ ] undefined
- [ ] Throws a TypeError

<details>
<summary>Show Answer</summary>

**Answer:** B — NaN

**Explanation:** **Idiom:** arrow functions have no `this` of their own — they capture `this` lexically from the enclosing scope at the point they're *defined*, not from the object they're attached to. Here `increment` is defined at the top level of the module, so its `this` is the module's `this` (`undefined` in an ES module, or `module.exports` in CommonJS) — either way, not `counter`. `this.count` is therefore `undefined`, and `undefined++` evaluates to `NaN`. Using an arrow function as an object method is a classic beginner mistake for exactly this reason.

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does this log?

::code-wrapper{language="javascript"}
```javascript
class Timer {
  constructor() {
    this.seconds = 0;
  }
  tick() {
    this.seconds++;
    return this.seconds;
  }
}

const t = new Timer();
console.log(t.tick());
```
::

- [ ] 1
- [ ] 0
- [ ] undefined
- [ ] NaN

<details>
<summary>Show Answer</summary>

**Answer:** A — 1

**Explanation:** `new Timer()` creates a fresh object and binds it as `this` inside the constructor, setting `this.seconds = 0`. Calling `t.tick()` afterward is ordinary implicit binding (`t` is the receiver), so `this.seconds++` increments `0` to `1` and returns it. There's no ambiguity here — the gotchas around `this` and classes usually appear once a method is detached from its instance (see later questions).

</details>
::

::question-wrapper{language="javascript"}
### Q8. What happens here?

::code-wrapper{language="javascript"}
```javascript
const Foo = () => {};
const f = new Foo();
```
::

- [ ] Throws `TypeError: Foo is not a constructor`
- [ ] `f` becomes a new empty object whose prototype is `Foo.prototype`
- [ ] `f` is `undefined`
- [ ] `this` inside `Foo` is bound to the new object, same as a regular function

<details>
<summary>Show Answer</summary>

**Answer:** A — Throws `TypeError: Foo is not a constructor`

**Explanation:** **Safety:** arrow functions intentionally lack the internal `[[Construct]]` method that regular functions have, so they can never be used with `new` — this is part of what makes their lexical `this` reliable (there's no separate "new binding" mode to worry about). This is a deliberate design choice, not an oversight: it prevents arrow functions from ever being accidentally used as constructors.

</details>
::

::question-wrapper{language="javascript"}
### Q9. This module runs under strict mode (it's an ES module). What happens?

::code-wrapper{language="javascript"}
```javascript
const obj = {
  items: [1, 2, 3],
  label: 'nums',
  printAll() {
    this.items.forEach(function (item) {
      console.log(this.label, item);
    });
  }
};

obj.printAll();
```
::

- [ ] Logs "nums 1", "nums 2", "nums 3"
- [ ] Throws `TypeError: Cannot read properties of undefined (reading 'label')`
- [ ] Logs "undefined 1", "undefined 2", "undefined 3"
- [ ] Logs "nums undefined" three times

<details>
<summary>Show Answer</summary>

**Answer:** B — Throws `TypeError: Cannot read properties of undefined (reading 'label')`

**Explanation:** **Debug:** the callback passed to `forEach` is a plain function, not a method call — it gets **default binding**, not the `this` from `printAll`. In strict-mode code (which class bodies and ES modules always are), default binding makes `this` `undefined` rather than falling back to the global object, so `this.label` throws immediately on the first iteration. The fix is to either use an arrow function for the callback (`this.items.forEach(item => console.log(this.label, item))`, which lexically inherits `printAll`'s `this`), or pass `this` as `forEach`'s optional second `thisArg` argument.

</details>
::

::question-wrapper{language="javascript"}
### Q10. What does this log in Node.js?

::code-wrapper{language="javascript"}
```javascript
class Countdown {
  constructor() {
    this.time = 3;
  }
  start() {
    setTimeout(function () {
      console.log(this.time);
    }, 0);
  }
}

new Countdown().start();
```
::

- [ ] 3
- [ ] undefined
- [ ] Throws a TypeError
- [ ] NaN

<details>
<summary>Show Answer</summary>

**Answer:** B — undefined

**Explanation:** The callback given to `setTimeout` is invoked as a plain function call by the timer internals, not as a method of the `Countdown` instance, so `this` inside it is **not** `countdown`. In Node it ends up being the `Timeout` object (or the global object in a browser under sloppy mode); either way, `this.time` doesn't exist on it, so the property access quietly returns `undefined` rather than throwing. The fix is to use an arrow function for the callback so it inherits `this` lexically from `start()`.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What does this log when the button is clicked?

::code-wrapper{language="javascript"}
```javascript
button.addEventListener('click', () => {
  console.log(this === button);
});
```
::

- [ ] false
- [ ] true
- [ ] Throws a ReferenceError because `this` is undeclared
- [ ] It depends on whether the button has an `onclick` attribute

<details>
<summary>Show Answer</summary>

**Answer:** A — false

**Explanation:** DOM listeners normally invoke a plain `function` callback with `this` set to the element the listener is attached to — that's how `this === button` would become `true` with a regular `function` handler. But arrow functions ignore that call-time binding entirely and use the lexical `this` from the surrounding scope where the arrow was defined (module/outer scope), which is not the button. Reaching for an arrow function "for brevity" in an event handler is a common way to accidentally lose access to the element via `this`.

</details>
::

::question-wrapper{language="javascript"}
### Q12. What does this log?

::code-wrapper{language="javascript"}
```javascript
function intro() {
  return `${this.role}: ${this.name}`;
}

const emp = { name: 'Sam', role: 'Dev' };
const bound = intro.bind(emp);

console.log(bound.call({ name: 'Other', role: 'X' }));
```
::

- [ ] "Dev: Sam"
- [ ] "X: Other"
- [ ] Throws a TypeError because `call` can't be used on a bound function
- [ ] "undefined: undefined"

<details>
<summary>Show Answer</summary>

**Answer:** A — "Dev: Sam"

**Explanation:** **Idiom:** `bind` creates a "hard-bound" function whose `this` cannot be overridden by any later `call`, `apply`, or even another `bind` — the original receiver (`emp`) always wins. Beginners often assume the *most recent* binding call takes precedence, but explicit-binding precedence rules put hard binding above ordinary `call`/`apply` on the bound wrapper.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What does this log?

::code-wrapper{language="javascript"}
```javascript
function Person(name) {
  this.name = name;
}

const BoundPerson = Person.bind({ name: 'ignored' });
const p = new BoundPerson('Zoe');

console.log(p.name);
```
::

- [ ] 'ignored'
- [ ] 'Zoe'
- [ ] undefined
- [ ] Throws a TypeError because a bound function can't be used with `new`

<details>
<summary>Show Answer</summary>

**Answer:** B — 'Zoe'

**Explanation:** **Safety:** this is the one case where hard binding *does* get overridden — the spec explicitly makes **new binding** take precedence over a bound `this` when a bound function is invoked with `new`. The bound `{ name: 'ignored' }` receiver is discarded, and `p` becomes a freshly constructed object as if `new Person('Zoe')` had been called directly. This is easy to get wrong because it contradicts Q12's rule, but `new` is the one exception to "bind always wins."

</details>
::

::question-wrapper{language="javascript"}
### Q14. What does this log?

::code-wrapper{language="javascript"}
```javascript
'use strict';

function f() {
  return this;
}

console.log(f());
```
::

- [ ] undefined
- [ ] `globalThis`
- [ ] `{}`
- [ ] `null`

<details>
<summary>Show Answer</summary>

**Answer:** A — undefined

**Explanation:** **Portability:** under `'use strict'`, default binding no longer falls back to the global object the way sloppy-mode code does — it leaves `this` as `undefined`. This matters for portability between script contexts: the same unbound function call in a non-strict `<script>` tag would return `window` in a browser, so relying on `this` in a top-level function call behaves differently depending on strict mode.

</details>
::

::question-wrapper{language="javascript"}
### Q15. Why is `this.handleClick = this.handleClick.bind(this)` written inside this constructor?

::code-wrapper{language="javascript"}
```javascript
class Button {
  constructor() {
    this.clicks = 0;
    this.handleClick = this.handleClick.bind(this);
  }
  handleClick() {
    this.clicks++;
  }
}

const b = new Button();
el.addEventListener('click', b.handleClick);
```
::

- [ ] So `handleClick` keeps referring to `b` even though the listener calls it as a plain function, not `b.handleClick()`
- [ ] To improve performance by caching the method lookup
- [ ] Because class methods can't be passed as callbacks otherwise (a syntax restriction)
- [ ] To make `handleClick` a private field

<details>
<summary>Show Answer</summary>

**Answer:** A — So `handleClick` keeps referring to `b` even though the listener calls it as a plain function, not `b.handleClick()`

**Explanation:** **Idiom:** once a method is passed by reference (`b.handleClick` without calling it), it's detached from its instance — the event system will call it as `handleClick(event)`, a plain invocation with default binding, not `b.handleClick(event)`. Pre-binding in the constructor guarantees `this` stays `b` regardless of how the function is later invoked. This is a genuine correctness requirement, not a performance trick, and class syntax has no restriction that would otherwise forbid passing a method as a callback (it would just silently lose its `this`).

</details>
::

::question-wrapper{language="javascript"}
### Q16. What is the trade-off of using an arrow class field instead of a bound prototype method?

::code-wrapper{language="javascript"}
```javascript
class Button {
  clicks = 0;
  handleClick = () => {
    this.clicks++;
  };
}
```
::

- [ ] `handleClick` is guaranteed auto-bound to each instance, but it's created fresh per instance (living on the object, not the shared prototype), using more memory than a single prototype method
- [ ] `handleClick` behaves identically to a prototype method in every respect, including memory usage
- [ ] Arrow class fields cannot access other instance fields like `this.clicks`
- [ ] This syntax is invalid — arrow functions cannot be used as class fields

<details>
<summary>Show Answer</summary>

**Answer:** A — `handleClick` is guaranteed auto-bound to each instance, but it's created fresh per instance (living on the object, not the shared prototype), using more memory than a single prototype method

**Explanation:** **Performance:** an arrow class field is initialized per-instance in the constructor step, capturing that instance's `this` lexically — so it never needs `.bind()` and is safe to hand off as a callback directly. The cost is that every instance gets its own copy of the function instead of sharing one on `Button.prototype`, which matters if you're creating thousands of instances. For a handful of instances this is a non-issue and the auto-binding convenience usually wins.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What does this log?

::code-wrapper{language="javascript"}
```javascript
'use strict';

const user = { name: 'Ava', greet() { return `Hi, I'm ${this.name}`; } };
const { greet } = user;

console.log(greet());
```
::

- [ ] Throws `TypeError: Cannot read properties of undefined (reading 'name')`
- [ ] "Hi, I'm Ava"
- [ ] "Hi, I'm undefined"
- [ ] Throws a ReferenceError because `greet` isn't defined

<details>
<summary>Show Answer</summary>

**Answer:** A — Throws `TypeError: Cannot read properties of undefined (reading 'name')`

**Explanation:** Destructuring `greet` off `user` copies the *function*, not its association with `user` — calling the standalone `greet()` afterward is a plain invocation with default binding, and in strict mode that means `this` is `undefined`, so `this.name` throws. The best-practice fix is to either call it as `user.greet()`, bind it explicitly (`const greet = user.greet.bind(user)`), or avoid destructuring methods that depend on `this` at all — prefer plain functions with explicit parameters when a value needs to travel independently of its object.

</details>
::

::question-wrapper{language="javascript"}
### Q18. Which style is generally the more testable, reusable choice for a value-transformation function?

::code-wrapper{language="javascript"}
```javascript
// Option A
function applyDiscount() {
  return this.price * (1 - this.rate);
}

// Option B
function applyDiscount(price, rate) {
  return price * (1 - rate);
}
```
::

- [ ] Option B — explicit parameters, no dependency on how the function is called
- [ ] Option A — relying on `this` is always more idiomatic in JavaScript
- [ ] They are equivalent in every practical sense
- [ ] Option A, but only if it's converted to an arrow function

<details>
<summary>Show Answer</summary>

**Answer:** A — Option B — explicit parameters, no dependency on how the function is called

**Explanation:** **Idiom:** functions that read `this` are coupled to their call-site — they only work correctly when invoked as a method (or explicitly bound), which makes them fragile when passed around, destructured, or unit-tested in isolation (you'd have to fake a receiver via `call`/`apply` just to test them). Preferring explicit parameters over implicit `this` is a widely recommended practice for utility/pure functions; `this` binding is best reserved for genuine object methods that need to read multiple pieces of instance state.

</details>
::

::question-wrapper{language="javascript"}
### Q19. What does `this` refer to at the top level of an ES module file?

::code-wrapper{language="javascript"}
```javascript
// inside a .mjs file or a <script type="module">
console.log(this);
```
::

- [ ] undefined
- [ ] The module's own `exports` object, like in CommonJS
- [ ] `globalThis`
- [ ] `null`

<details>
<summary>Show Answer</summary>

**Answer:** A — undefined

**Explanation:** **Portability:** ES modules are always strict and have no top-level `this` binding to any object — it's `undefined` by spec, unlike CommonJS modules, where top-level `this` is `module.exports`. Code migrating from CommonJS to ESM that relied on top-level `this.foo = ...` for exports will silently break (`this` is `undefined`, so property access on it throws) rather than continuing to "work" some other way.

</details>
::

::question-wrapper{language="javascript"}
### Q20. What does this log?

::code-wrapper{language="javascript"}
```javascript
const nums = [1, 2, 3];
const scaled = nums.map(function (x) {
  return this.multiplier * x;
}, { multiplier: 10 });

console.log(scaled);
```
::

- [ ] [10, 20, 30]
- [ ] Throws a TypeError because `map`'s callback can't be passed a receiver
- [ ] [NaN, NaN, NaN]
- [ ] [1, 2, 3]

<details>
<summary>Show Answer</summary>

**Answer:** A — [10, 20, 30]

**Explanation:** **Idiom:** many array iteration methods (`map`, `filter`, `forEach`, `some`, `every`, `find`) accept an optional second argument that becomes `this` inside the callback — it's a lesser-known alternative to `.bind()` or an arrow function for supplying a receiver. Here `this.multiplier` is `10` for every call, so each element is scaled: `[1*10, 2*10, 3*10]`. It's underused compared to arrow functions/closures, but recognizing it avoids "wait, how does `this` work here" confusion when it appears in existing code.

</details>
::
