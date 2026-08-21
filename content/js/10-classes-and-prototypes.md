# 10 — Classes & Prototypes

::question-wrapper
### Q1. Which statement best describes the relationship between JavaScript `class` syntax and prototypes?

- [ ] `class Foo {}` is just syntactic sugar — it creates an ordinary function `Foo` whose `.prototype` object holds the instance methods, the same underlying mechanism as manually writing `function Foo() {}` and assigning methods onto `Foo.prototype`
- [ ] `class` introduces a completely new object model separate from prototypes — classes do not have a `.prototype` property at all
- [ ] `class` methods are copied onto every instance individually when `new` is called, so no prototype chain lookup is involved
- [ ] `class` syntax only works for built-in types like `Array` and `Map`; user-defined types must still use `function` constructors

<details>
<summary>Show Answer</summary>

**Answer:** A — `class Foo {}` is just syntactic sugar — it creates an ordinary function `Foo` whose `.prototype` object holds the instance methods, the same underlying mechanism as manually writing `function Foo() {}` and assigning methods onto `Foo.prototype`

**Explanation:** Under the hood, a `class` declaration still produces a function value (`typeof Foo === "function"`), and every method written in the class body is installed onto `Foo.prototype`, exactly like the pre-ES6 pattern of `function Foo() {}` plus `Foo.prototype.method = ...`. Classes add stricter syntax (mandatory `new`, real inheritance via `extends`, TDZ, always-strict-mode bodies), but the runtime object model — functions with a `.prototype` object shared by instances — is unchanged. B, C, and D all describe a different object model than the one JS actually uses.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does this log?

::code-wrapper{language="javascript"}
```javascript
class Animal {
  speak() {
    return "...";
  }
}

function LegacyAnimal() {}
LegacyAnimal.prototype.speak = function () {
  return "...";
};

console.log(typeof Animal, typeof Animal.prototype.speak);
console.log(typeof LegacyAnimal, typeof LegacyAnimal.prototype.speak);
```
::

- [ ] `"function" "undefined"` then `"function" "function"` — only the legacy pattern actually attaches `speak` onto the prototype
- [ ] `"function" "function"` then `"function" "function"` — a `class` declaration produces an ordinary function value whose methods are installed on `.prototype`, exactly like manually building a constructor function and assigning to `.prototype` by hand
- [ ] `"object" "function"` then `"function" "function"` — classes are plain objects, not functions
- [ ] `"function" "function"` then `"object" "function"`

<details>
<summary>Show Answer</summary>

**Answer:** B — `"function" "function"` then `"function" "function"` — a `class` declaration produces an ordinary function value whose methods are installed on `.prototype`, exactly like manually building a constructor function and assigning to `.prototype` by hand

**Explanation:** **Idiom:** both blocks log identically, which is exactly the point — `Animal` is a function just like `LegacyAnimal`, and `speak` lives on `Animal.prototype` just like it lives on `LegacyAnimal.prototype`. The `class` keyword is a more readable, safer syntax for building this same function-plus-prototype structure; it doesn't create a fundamentally different kind of value. Options C and D invent a distinction (class as "object") that doesn't exist in JS.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What happens when this runs?

::code-wrapper{language="javascript"}
```javascript
const p = new Point3D(1, 2, 3);

class Point3D {
  constructor(x, y, z) {
    this.x = x;
    this.y = y;
    this.z = z;
  }
}
```
::

- [ ] It works fine — classes, like function declarations, are fully hoisted with their implementation
- [ ] It logs `undefined` when `p` is later inspected, because `Point3D` is hoisted but left uninitialized like a `var`
- [ ] `ReferenceError: Cannot access 'Point3D' before initialization` — the class binding is hoisted but stays in the temporal dead zone until the `class` statement itself is evaluated, just like `let`/`const`
- [ ] `TypeError: Point3D is not a constructor`

<details>
<summary>Show Answer</summary>

**Answer:** C — `ReferenceError: Cannot access 'Point3D' before initialization` — the class binding is hoisted but stays in the temporal dead zone until the `class` statement itself is evaluated, just like `let`/`const`

**Explanation:** **Debug:** unlike function declarations, `class` declarations are not hoisted with their implementation ready to use. The name `Point3D` is hoisted to the top of its scope but remains in the temporal dead zone (TDZ) until the `class` statement's line actually executes, exactly like `let`/`const`. Referencing it any earlier — even just to call `new` on it — throws a `ReferenceError`. This is a common trap for developers used to function declarations being freely callable before their textual position (see quiz 05).

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does this log?

::code-wrapper{language="javascript"}
```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  distanceFromOrigin() {
    return Math.sqrt(this.x ** 2 + this.y ** 2);
  }
}

const p = new Point(3, 4);
console.log(Object.getPrototypeOf(p) === Point.prototype);
console.log(p.hasOwnProperty("distanceFromOrigin"));
console.log(p.hasOwnProperty("x"));
```
::

- [ ] `true`, `true`, `true`
- [ ] `false`, `false`, `true`
- [ ] `true`, `false`, `false`
- [ ] `true`, `false`, `true` — methods defined in a class body live once on the shared prototype, while properties assigned via `this.x =` inside the constructor become the instance's own properties

<details>
<summary>Show Answer</summary>

**Answer:** D — `true`, `false`, `true` — methods defined in a class body live once on the shared prototype, while properties assigned via `this.x =` inside the constructor become the instance's own properties

**Explanation:** `Object.getPrototypeOf(p) === Point.prototype` confirms that `p`'s internal `[[Prototype]]` link points to `Point.prototype`, which is where `new` wires it up. `distanceFromOrigin` was declared as a class method, so it lives once on `Point.prototype` and is *not* an own property of `p` — `hasOwnProperty` correctly reports `false`. `x` and `y`, by contrast, are assigned directly onto `this` inside the constructor, so they genuinely are own properties of every instance, hence `true`. This is the standard way to verify that instance methods are shared, not duplicated per instance.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does this log?

::code-wrapper{language="javascript"}
```javascript
class LivingThing {
  breathe() {
    return "breathing";
  }
}

class Animal extends LivingThing {}
class Dog extends Animal {}

const rex = new Dog();
console.log(rex.breathe());
console.log(Object.getPrototypeOf(Object.getPrototypeOf(rex)) === Animal.prototype);
```
::

- [ ] `"breathing"` then `true` — `breathe` isn't found on `Dog.prototype` or `Animal.prototype`, so the engine keeps walking up the `[[Prototype]]` chain until it reaches `LivingThing.prototype`, where the method is finally found
- [ ] `TypeError: rex.breathe is not a function`, because `breathe` is defined two inheritance levels above `Dog`
- [ ] `"breathing"` then `false`
- [ ] `undefined` then `true`

<details>
<summary>Show Answer</summary>

**Answer:** A — `"breathing"` then `true` — `breathe` isn't found on `Dog.prototype` or `Animal.prototype`, so the engine keeps walking up the `[[Prototype]]` chain until it reaches `LivingThing.prototype`, where the method is finally found

**Explanation:** Property/method lookup in JS is never limited to an object's own prototype — when `rex.breathe()` is called, the engine checks `rex` itself (no `breathe`), then `Dog.prototype` (no `breathe`), then `Animal.prototype` (no `breathe`), then `LivingThing.prototype` (found), and stops there. This walk continues until a match is found or the chain terminates at `null`. `Object.getPrototypeOf(rex)` is `Dog.prototype`, and `Object.getPrototypeOf(Dog.prototype)` is `Animal.prototype` — exactly what `extends` wires up — so the equality check is `true`.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What happens when this runs?

::code-wrapper{language="javascript"}
```javascript
class Vehicle {
  constructor(make) {
    this.make = make;
  }
}

class Car extends Vehicle {
  constructor(make, model) {
    this.model = model;
    super(make);
  }
}

const c = new Car("Toyota", "Corolla");
```
::

- [ ] It creates the car normally, with both `make` and `model` set
- [ ] `ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor`
- [ ] `TypeError: Cannot set property 'model' of undefined`
- [ ] It works, but `make` ends up `undefined`

<details>
<summary>Show Answer</summary>

**Answer:** B — `ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived constructor`

**Explanation:** **Safety:** in a derived class, `this` doesn't exist until `super()` has run — the parent constructor is what actually allocates and initializes the object. `Car`'s constructor tries to write `this.model = model` *before* calling `super(make)`, so it touches `this` while it's still uninitialized, and the engine throws immediately. The fix is to call `super(make)` first and only then assign `this.model = model`. This rule has no equivalent in base (non-`extends`) classes, where `this` is available from the start of the constructor.

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does this log?

::code-wrapper{language="javascript"}
```javascript
class AnimalBase {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} makes a sound`;
  }
}

class Cat extends AnimalBase {
  speak() {
    return `${this.name} meows`;
  }
}

const c = new Cat("Whiskers");
console.log(c.name, c.speak());
```
::

- [ ] `undefined "undefined meows"` — without an explicit constructor, `name` never gets assigned
- [ ] `TypeError: Cannot read properties of undefined`
- [ ] `"Whiskers" "Whiskers meows"` — omitting the constructor doesn't skip initialization; JS auto-generates `constructor(...args) { super(...args); }` for any subclass that doesn't define its own, forwarding every argument straight to the parent constructor
- [ ] `"Whiskers" "Whiskers makes a sound"` — subclass method overrides only apply if a constructor is explicitly defined

<details>
<summary>Show Answer</summary>

**Answer:** C — `"Whiskers" "Whiskers meows"` — omitting the constructor doesn't skip initialization; JS auto-generates `constructor(...args) { super(...args); }` for any subclass that doesn't define its own, forwarding every argument straight to the parent constructor

**Explanation:** `Cat` has no explicit `constructor`, so the spec supplies a default one that simply forwards all arguments to `super`. That means `new Cat("Whiskers")` still runs `AnimalBase`'s constructor with `"Whiskers"`, correctly setting `this.name`. Method overriding is completely independent of whether a constructor was written — `speak` resolves to `Cat.prototype.speak` via normal prototype lookup regardless, so the answer combines the parent-initialized `name` with the overridden `meows` behavior.

</details>
::

::question-wrapper{language="javascript"}
### Q8. What does this log?

::code-wrapper{language="javascript"}
```javascript
class Employee {
  describe() {
    return "an employee";
  }
}

class Manager extends Employee {
  describe() {
    return `a manager (${super.describe()})`;
  }
}

console.log(new Manager().describe());
```
::

- [ ] `"an employee"`
- [ ] `"a manager (a manager (an employee))"` — `super.describe()` re-invokes `Manager`'s own `describe`, causing infinite-looking recursion
- [ ] `TypeError: super.describe is not a function`
- [ ] `"a manager (an employee)"` — `describe` is overridden in `Manager`, shadowing `Employee`'s version for normal calls, but `super.describe()` explicitly reaches one level up the prototype chain to invoke `Employee`'s original implementation

<details>
<summary>Show Answer</summary>

**Answer:** D — `"a manager (an employee)"` — `describe` is overridden in `Manager`, shadowing `Employee`'s version for normal calls, but `super.describe()` explicitly reaches one level up the prototype chain to invoke `Employee`'s original implementation

**Explanation:** Overriding a method in a subclass shadows the parent's version for ordinary calls (`instance.describe()` always finds `Manager.prototype.describe` first). `super.describe()` bypasses that shadowing on purpose — it's a special reference that looks up `describe` starting from `Employee.prototype` instead of `Manager.prototype`, invoking the parent's original implementation without re-entering `Manager`'s override. There's no recursion here at all, ruling out option B.

</details>
::

::question-wrapper{language="javascript"}
### Q9. What does this log?

::code-wrapper{language="javascript"}
```javascript
class MathUtils {
  static square(n) {
    return n * n;
  }
}

const m = new MathUtils();
console.log(MathUtils.square(5));
console.log(m.square(5));
```
::

- [ ] `25` then `TypeError: m.square is not a function` — static methods are installed directly on the class/constructor object itself, never on `.prototype`, so instances have no access to them at all
- [ ] `25` then `25`
- [ ] `TypeError` on the very first line, because a class with only static methods cannot be instantiated
- [ ] `25` then `undefined`

<details>
<summary>Show Answer</summary>

**Answer:** A — `25` then `TypeError: m.square is not a function` — static methods are installed directly on the class/constructor object itself, never on `.prototype`, so instances have no access to them at all

**Explanation:** `static` methods and properties belong to the class object (`MathUtils`) itself, not to `MathUtils.prototype`. Since instance property/method lookup only ever walks the `[[Prototype]]` chain starting from the instance, and `MathUtils.prototype` is never part of that chain that leads to `MathUtils` itself, `m.square` simply doesn't resolve to anything — calling it throws a `TypeError`. `MathUtils.square(5)` works because it's called directly on the class object where the static method actually lives.

</details>
::

::question-wrapper{language="javascript"}
### Q10. What does this log?

::code-wrapper{language="javascript"}
```javascript
class Base {
  static create() {
    return "created via Base.create";
  }
}

class Derived extends Base {}

console.log(Derived.create());
```
::

- [ ] `TypeError: Derived.create is not a function`
- [ ] `"created via Base.create"` — `extends` links not just `Derived.prototype` to `Base.prototype`, but also `Derived` itself to `Base`, so static members are inherited by the subclass too, the same way instance methods are inherited via `.prototype`
- [ ] `undefined`
- [ ] `ReferenceError: create is not defined`

<details>
<summary>Show Answer</summary>

**Answer:** B — `"created via Base.create"` — `extends` links not just `Derived.prototype` to `Base.prototype`, but also `Derived` itself to `Base`, so static members are inherited by the subclass too, the same way instance methods are inherited via `.prototype`

**Explanation:** **Idiom:** `extends` sets up two parallel prototype links: `Derived.prototype`'s `[[Prototype]]` becomes `Base.prototype` (for instance methods), *and* `Derived`'s own `[[Prototype]]` becomes `Base` (for static members). So looking up `Derived.create` walks from `Derived` up to `Base`, finds the static method there, and it runs successfully. This surprises developers who assume `static` members are entirely non-inherited — they're excluded from *instances*, but not from subclasses.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What happens when this runs?

::code-wrapper{language="javascript"}
```javascript
class BankAccount {
  #balance = 0;
  deposit(amount) {
    this.#balance += amount;
  }
  get balance() {
    return this.#balance;
  }
}

const acct = new BankAccount();
acct.deposit(100);
console.log(acct.#balance);
```
::

- [ ] Logs `100`
- [ ] Logs `undefined`
- [ ] Throws `SyntaxError: Private field '#balance' must be declared in an enclosing class` at parse time — nothing in the file executes at all, not even the earlier `acct.deposit(100)` call, because the whole script fails to parse before any code runs
- [ ] Throws `TypeError: acct.#balance is not accessible outside class BankAccount`

<details>
<summary>Show Answer</summary>

**Answer:** C — Throws `SyntaxError: Private field '#balance' must be declared in an enclosing class` at parse time — nothing in the file executes at all, not even the earlier `acct.deposit(100)` call, because the whole script fails to parse before any code runs

**Explanation:** **Safety:** unlike an unenforced `_balance` naming convention, a true private field (`#balance`) is enforced at the language level. The `.#name` syntax is only legal lexically inside a class body that declares that exact private name — writing `acct.#balance` anywhere else isn't just a runtime access violation, it's not valid syntax at all, so the engine rejects the entire script before executing a single line. This is stricter than a runtime `TypeError` (option D): the failure happens at parse time, not when the line would run.

</details>
::

::question-wrapper{language="javascript"}
### Q12. What does this log?

::code-wrapper{language="javascript"}
```javascript
class Wallet {
  _cash = 50;
  #pin = "1234";
}

const w = new Wallet();
console.log(w._cash);
console.log(w["#pin"]);
```
::

- [ ] `50` then `1234` — bracket notation is a valid way to reach private fields from outside a class
- [ ] `undefined` then `undefined`
- [ ] It throws a `SyntaxError` on the second line
- [ ] `50` then `undefined` — `_cash` is only a naming convention, so it's freely readable from outside; `#pin` is a genuinely private field, and `w["#pin"]` doesn't reach it at all — it's just an ordinary bracket lookup for a property literally named the four characters `"#pin"`, which was never set, so it returns `undefined`

<details>
<summary>Show Answer</summary>

**Answer:** D — `50` then `undefined` — `_cash` is only a naming convention, so it's freely readable from outside; `#pin` is a genuinely private field, and `w["#pin"]` doesn't reach it at all — it's just an ordinary bracket lookup for a property literally named the four characters `"#pin"`, which was never set, so it returns `undefined`

**Explanation:** **Debug:** the leading underscore in `_cash` communicates "please don't touch this from outside," but the engine does nothing to enforce it — `w._cash` reads it just like any other property. `#pin` is different in kind, not just convention: private fields are never accessible via bracket/computed notation at all, even from inside the class. `w["#pin"]` doesn't parse as a private-field access — the string `"#pin"` is just an ordinary property key, and since no property with that literal name was ever set, the lookup quietly returns `undefined` instead of exposing the private value or throwing.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What does this log?

::code-wrapper{language="javascript"}
```javascript
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }
  get area() {
    return this.width * this.height;
  }
}

const r = new Rectangle(4, 5);
console.log(r.area);
r.width = 10;
console.log(r.area);
```
::

- [ ] `20` then `50` — a getter runs its body fresh on every access; nothing caches the earlier result, so once `width` changes, the next read of `area` recomputes from the current `width`/`height`
- [ ] `20` then `20` — `area` is computed once at first access and then cached like a regular property
- [ ] `NaN` then `NaN`
- [ ] `20` then `TypeError: Cannot set property 'width'`

<details>
<summary>Show Answer</summary>

**Answer:** A — `20` then `50` — a getter runs its body fresh on every access; nothing caches the earlier result, so once `width` changes, the next read of `area` recomputes from the current `width`/`height`

**Explanation:** A `get` accessor in a class is a function that runs every single time `instance.prop` is read — it's transparent property syntax over a method call, not a stored value. Nothing about `r.area` is memoized, so the first read computes `4 * 5 = 20`, and after `r.width = 10`, the second read recomputes `10 * 5 = 50` from the current field values. Assuming the result gets cached (option B) is a common but incorrect mental model — that's not how getters work unless you explicitly build caching yourself.

</details>
::

::question-wrapper{language="javascript"}
### Q14. What happens when this runs?

::code-wrapper{language="javascript"}
```javascript
class Temperature {
  #celsius = 0;
  get fahrenheit() {
    return (this.#celsius * 9) / 5 + 32;
  }
}

const t = new Temperature();
t.fahrenheit = 100;
console.log(t.fahrenheit);
```
::

- [ ] Logs `32` — the assignment is silently ignored because there's no setter
- [ ] `TypeError: Cannot set property fahrenheit of #<Temperature> which has only a getter` — class bodies (including field initializers and methods) always execute in strict mode, even without a `"use strict"` pragma, so assigning to a getter-only property throws rather than failing silently
- [ ] Logs `100`
- [ ] Logs `NaN`

<details>
<summary>Show Answer</summary>

**Answer:** B — `TypeError: Cannot set property fahrenheit of #<Temperature> which has only a getter` — class bodies (including field initializers and methods) always execute in strict mode, even without a `"use strict"` pragma, so assigning to a getter-only property throws rather than failing silently

**Explanation:** **Safety:** JS class bodies are implicitly strict mode, no `"use strict"` needed. In strict mode, assigning to a property that only has a getter (no setter defined) throws a `TypeError` instead of silently doing nothing. This is a meaningful difference from a plain object literal in a non-strict script, where the same kind of assignment can fail silently — classes fail loudly by default, which is generally the safer behavior for catching bugs early.

</details>
::

::question-wrapper{language="javascript"}
### Q15. What does this log, in order?

::code-wrapper{language="javascript"}
```javascript
class Counter {
  count = this.logStart();

  constructor() {
    console.log("constructor body runs, count =", this.count);
  }

  logStart() {
    console.log("field initializer runs");
    return 0;
  }
}

new Counter();
```
::

- [ ] `"constructor body runs, count = 0"` then `"field initializer runs"`
- [ ] `"field initializer runs"` then `"constructor body runs, count = undefined"`
- [ ] `"field initializer runs"` then `"constructor body runs, count = 0"` — public class fields are initialized per-instance before the constructor body executes, so `count`'s initializer (and the method call inside it) runs first, and by the time the constructor body logs, `this.count` is already set
- [ ] `TypeError: this.logStart is not a function`, because instance methods aren't available yet during field initialization

<details>
<summary>Show Answer</summary>

**Answer:** C — `"field initializer runs"` then `"constructor body runs, count = 0"` — public class fields are initialized per-instance before the constructor body executes, so `count`'s initializer (and the method call inside it) runs first, and by the time the constructor body logs, `this.count` is already set

**Explanation:** Public instance fields declared with `=` at the class body level are set up per-instance as part of construction, and for a base (non-`extends`) class that happens *before* the explicit constructor body runs. So `count`'s initializer executes first — calling `this.logStart()` (which is already available on the prototype at that point, ruling out option D) and logging `"field initializer runs"` — and only afterward does the constructor body run and log the now-initialized `this.count`.

</details>
::

::question-wrapper{language="javascript"}
### Q16. What does this log?

::code-wrapper{language="javascript"}
```javascript
class Shape {
  constructor() {
    console.log(this.describe());
  }
  describe() {
    return "a shape";
  }
}

class Circle extends Shape {
  radius = 5;
  describe() {
    return `a circle with radius ${this.radius}`;
  }
}

new Circle();
```
::

- [ ] `"a shape"`
- [ ] `"a circle with radius 5"`
- [ ] `ReferenceError: Cannot access 'radius' before initialization`
- [ ] `"a circle with radius undefined"` — method dispatch always uses the actual (derived) prototype, so `this.describe()` inside `Shape`'s constructor already resolves to `Circle`'s overridden `describe`; but `Circle`'s own field initializers (including `radius = 5`) only run *after* `super()` returns, so at the moment `describe` runs mid-`super()` call, `this.radius` isn't set yet and simply reads as `undefined` rather than throwing

<details>
<summary>Show Answer</summary>

**Answer:** D — `"a circle with radius undefined"` — method dispatch always uses the actual (derived) prototype, so `this.describe()` inside `Shape`'s constructor already resolves to `Circle`'s overridden `describe`; but `Circle`'s own field initializers (including `radius = 5`) only run *after* `super()` returns, so at the moment `describe` runs mid-`super()` call, `this.radius` isn't set yet and simply reads as `undefined` rather than throwing

**Explanation:** **Debug:** this combines two rules that interact badly. First, method overriding is resolved dynamically through the prototype chain, so even while `Shape`'s constructor is still running (as part of `super()`), `this.describe()` finds `Circle.prototype.describe`, not `Shape.prototype.describe`. Second, in a derived class, instance field initializers (like `radius = 5`) only run once `super()` has returned — they haven't executed yet at the point `describe` is invoked mid-`super()`. Reading a not-yet-set instance property doesn't throw (that's only TDZ behavior for `let`/`const` bindings); it just returns `undefined`, since the property genuinely doesn't exist yet anywhere in the chain. This ordering gotcha is a real source of bugs when a base constructor calls an overridable method.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What does this log?

::code-wrapper{language="javascript"}
```javascript
class Logger {
  prefix = "[LOG]";

  logRegular(msg) {
    console.log(this.prefix, msg);
  }

  logArrow = (msg) => {
    console.log(this.prefix, msg);
  };
}

const a = new Logger();
const b = new Logger();

console.log(a.logRegular === b.logRegular);
console.log(a.logArrow === b.logArrow);
```
::

- [ ] `true` then `false` — a regular method (`logRegular`) is defined once on `Logger.prototype` and shared by every instance, so both instances reference the identical function object; an arrow function class field (`logArrow`) is instead created fresh during each constructor run and assigned as an own property on that specific instance, so two instances never share the same function reference
- [ ] `false` then `false`
- [ ] `true` then `true`
- [ ] `false` then `true`

<details>
<summary>Show Answer</summary>

**Answer:** A — `true` then `false` — a regular method (`logRegular`) is defined once on `Logger.prototype` and shared by every instance, so both instances reference the identical function object; an arrow function class field (`logArrow`) is instead created fresh during each constructor run and assigned as an own property on that specific instance, so two instances never share the same function reference

**Explanation:** **Performance:** `logRegular` is ordinary class-method syntax, so it's installed exactly once on `Logger.prototype` — every instance looks it up through the prototype chain and gets back the same function object, hence `===` is `true`. `logArrow` is a class *field* whose value happens to be an arrow function; like any instance field, its initializer runs separately for every `new Logger()` call, creating a brand-new closure each time and assigning it as an own property of that instance. That's what makes `this` inside `logArrow` permanently bound to its own instance — but it comes at a real memory cost (one extra function object per instance) compared to a shared prototype method, which matters for classes instantiated many times.

</details>
::

::question-wrapper{language="javascript"}
### Q18. What does this log?

::code-wrapper{language="javascript"}
```javascript
class Shape {}

const s = new Shape();

class NewShape {}
Shape.prototype = NewShape.prototype;

console.log(s instanceof Shape);
```
::

- [ ] `true` — `s` was created as an instance of `Shape`, so it stays an instance forever
- [ ] `false` — `instanceof` re-reads `Shape.prototype` at the moment of the check, not at construction time; since `Shape.prototype` now points to a different object than the one actually in `s`'s `[[Prototype]]` chain, the check fails even though `s` was legitimately built by `new Shape()`
- [ ] `TypeError: Right-hand side of 'instanceof' is not callable`
- [ ] `undefined`

<details>
<summary>Show Answer</summary>

**Answer:** B — `false` — `instanceof` re-reads `Shape.prototype` at the moment of the check, not at construction time; since `Shape.prototype` now points to a different object than the one actually in `s`'s `[[Prototype]]` chain, the check fails even though `s` was legitimately built by `new Shape()`

**Explanation:** **Debug:** `x instanceof C` works by taking `C.prototype`'s *current* value and checking whether it appears anywhere in `x`'s `[[Prototype]]` chain — it does not remember what `C.prototype` was at construction time. `s` was built while `Shape.prototype` pointed to its original prototype object, so that original object is permanently baked into `s`'s chain. Reassigning `Shape.prototype = NewShape.prototype` afterward changes what the *class* currently points to, but does nothing to `s`'s existing chain, so the two no longer match and `instanceof` reports `false`. Reassigning `.prototype` after instances already exist is a real footgun for exactly this reason.

</details>
::

::question-wrapper{language="javascript"}
### Q19. What does this log?

::code-wrapper{language="javascript"}
```javascript
const Serializable = Base => class extends Base {
  toJSON() {
    return JSON.stringify(this);
  }
};

const Comparable = Base => class extends Base {
  equals(other) {
    return this.toJSON() === other.toJSON();
  }
};

class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class ComparablePoint extends Comparable(Serializable(Point)) {}

const a = new ComparablePoint(1, 2);
const b = new ComparablePoint(1, 2);

console.log(a.equals(b));
```
::

- [ ] `SyntaxError: A class can only extend one expression`
- [ ] `false`, because `toJSON` from `Serializable` is shadowed by `Comparable`'s own method
- [ ] `true` — since JS classes support only single inheritance via `extends`, each mixin is written as a function that takes a base class and returns a new subclass extending it; chaining these calls (`Comparable(Serializable(Point))`) layers multiple independent behaviors onto one class, which is how JS works around not having true multiple inheritance
- [ ] `TypeError: other.toJSON is not a function`

<details>
<summary>Show Answer</summary>

**Answer:** C — `true` — since JS classes support only single inheritance via `extends`, each mixin is written as a function that takes a base class and returns a new subclass extending it; chaining these calls (`Comparable(Serializable(Point))`) layers multiple independent behaviors onto one class, which is how JS works around not having true multiple inheritance

**Explanation:** **Idiom:** `extends` only ever accepts a single expression, but that expression can itself be a function call that returns a class — which is exactly what a mixin function is. `Serializable(Point)` returns an anonymous class extending `Point` that adds `toJSON`; `Comparable(...)` wraps that again, adding `equals`. `ComparablePoint` ends up with both `toJSON` and `equals` available through its prototype chain, with no conflict between them, so `a.equals(b)` compares their JSON forms and correctly returns `true`. This composition pattern is JS's standard workaround for the lack of real multiple inheritance.

</details>
::

::question-wrapper
### Q20. A team building a UI has `Modal`, `Tooltip`, and `Dropdown` components. All three need identical positioning logic, but "positioning" isn't conceptually a kind of `Modal`, `Tooltip`, or `Dropdown` — it's an unrelated, cross-cutting concern. What's the best-practice way to share this behavior?

- [ ] Create one deep base class `PositionableUIElement` that all three extend, even though they aren't otherwise conceptually related, so the shared logic lives in exactly one place
- [ ] Copy-paste the positioning code into each of the three components to avoid any coupling between them
- [ ] Use multiple inheritance by having each component `extends` both a base UI class and a separate positioning class at the same time
- [ ] Extract the positioning logic into a standalone function/object (a composed helper or mixin, as in Q19) that each component uses, since `extends` should model a genuine "is-a" relationship — reaching for inheritance purely to share unrelated behavior produces awkward, fragile hierarchies as more cross-cutting concerns pile up

<details>
<summary>Show Answer</summary>

**Answer:** D — Extract the positioning logic into a standalone function/object (a composed helper or mixin, as in Q19) that each component uses, since `extends` should model a genuine "is-a" relationship — reaching for inheritance purely to share unrelated behavior produces awkward, fragile hierarchies as more cross-cutting concerns pile up

**Explanation:** **Idiom:** `extends` should express "this really is a specialized version of that," not just "I want to reuse some code." Forcing `Modal`, `Tooltip`, and `Dropdown` under a shared `PositionableUIElement` base (option A) works today but becomes fragile the moment a fourth unrelated concern (e.g., "draggable") needs sharing too — deep, unrelated hierarchies are hard to reason about and hard to change safely. Copy-pasting (option B) duplicates bugs across three places. Option C isn't even possible — JS's `extends` clause accepts exactly one class expression, though as Q19 showed, that one expression can itself be composed from multiple mixins. Preferring composition (small, focused, combinable units of behavior) over deep inheritance chains is the widely recommended default for sharing behavior that isn't a true "is-a" relationship.

</details>
::
