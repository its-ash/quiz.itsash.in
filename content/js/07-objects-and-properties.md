# 07 — Objects & Properties

::question-wrapper{language="javascript"}
### Q1. What does `Object.keys(product)` log?

::code-wrapper{language="javascript"}
```javascript
const price = 42;
const inStock = true;

const product = {
  price,
  inStock,
  describe() {
    return `$${this.price}`;
  },
};

console.log(Object.keys(product));
```
::

- [ ] ["price", "inStock"]
- [ ] ["price", "inStock", "describe"]
- [ ] ["describe", "price", "inStock"]
- [ ] TypeError: describe is not a valid shorthand method

<details>
<summary>Show Answer</summary>

**Answer:** B — ["price", "inStock", "describe"]

**Explanation:** Method shorthand (`describe() {}`) creates a regular own, enumerable, writable, configurable property whose value happens to be a function — it's collected by `Object.keys` exactly like `price` and `inStock`, listed without parentheses. Keys stay in insertion order since none of them are integer-like. Option A wrongly assumes methods aren't "real" properties; C wrongly reorders them (string keys keep insertion order, they aren't sorted); D is wrong because shorthand methods are valid ES6 syntax.

</details>
::

::question-wrapper{language="javascript"}
### Q2. What does this log?

::code-wrapper{language="javascript"}
```javascript
const key = "role" + "Id";
const suffix = 2;

const user = {
  [key]: 101,
  [`level${suffix}`]: "admin",
};

console.log(user);
```
::

- [ ] { key: 101, "level2": "admin" }
- [ ] { roleId: 101, level: "admin" }
- [ ] { roleId: 101, level2: "admin" }
- [ ] SyntaxError: template literals cannot be used as computed keys

<details>
<summary>Show Answer</summary>

**Answer:** C — { roleId: 101, level2: "admin" }

**Explanation:** A computed property name `[expr]` evaluates `expr` at the moment the object literal is constructed and uses the resulting string as the key — so `key` (the variable) evaluates to `"roleId"`, and the template literal evaluates to `"level2"`. Option A confuses the identifier `key` with its computed string value. Option B forgets `suffix` gets interpolated. Option D is wrong: any valid expression, including a template literal, is allowed inside `[]`.

</details>
::

::question-wrapper{language="javascript"}
### Q3. What does this log?

::code-wrapper{language="javascript"}
```javascript
const cache = {};
cache[1] = "first";
cache["1"] = "overwritten";

console.log(cache[1], Object.keys(cache).length);
```
::

- [ ] "overwritten" 1
- [ ] "first" 2
- [ ] "overwritten" 2
- [ ] undefined 1

<details>
<summary>Show Answer</summary>

**Answer:** A — "overwritten" 1

**Explanation:** **Debug:** every non-Symbol property key on a plain object is coerced to a string before it's stored, so the numeric key `1` and the string key `"1"` address the exact same property. The second assignment overwrites the first rather than creating a second entry, leaving one key and the latest value. Options B and C wrongly assume numeric and string keys are distinct slots.

</details>
::

::question-wrapper{language="javascript"}
### Q4. What does this log?

::code-wrapper{language="javascript"}
```javascript
const config = Object.freeze({
  name: "app",
  limits: { maxUsers: 10 },
});

config.name = "changed";
config.limits.maxUsers = 999;

console.log(config.name, config.limits.maxUsers);
```
::

- [ ] "changed" 999
- [ ] "app" 10
- [ ] "changed" 10
- [ ] "app" 999

<details>
<summary>Show Answer</summary>

**Answer:** D — "app" 999

**Explanation:** **Safety:** `Object.freeze` only locks the object's own top-level property slots — it makes `name` non-writable, so the reassignment is silently ignored (in non-strict mode) and `config.name` stays `"app"`. But `limits` itself is just a frozen *reference* to another, unfrozen object; freezing never cascades into values it points to, so mutating `config.limits.maxUsers` succeeds normally. Deep-freezing requires recursively freezing every nested object yourself.

</details>
::

::question-wrapper{language="javascript"}
### Q5. What does this log?

::code-wrapper{language="javascript"}
```javascript
const original = { id: 1, meta: { tags: ["a"] } };
const copy = { ...original };

copy.id = 2;
copy.meta.tags.push("b");

console.log(original.id, original.meta.tags);
```
::

- [ ] 2 ["a"]
- [ ] 1 ["a", "b"]
- [ ] 2 ["a", "b"]
- [ ] 1 ["a"]

<details>
<summary>Show Answer</summary>

**Answer:** B — 1 ["a", "b"]

**Explanation:** **Debug:** the spread operator `{ ...original }` performs a *shallow* copy — each top-level value is copied into a new slot, so reassigning `copy.id` never touches `original.id` (still `1`). But `meta` holds an object reference, and the shallow copy duplicates the reference itself, not the object it points to — `copy.meta` and `original.meta` are the same array's owner, so pushing onto `copy.meta.tags` is visible through `original.meta.tags` too. This is the classic shallow-copy trap with nested reference types.

</details>
::

::question-wrapper{language="javascript"}
### Q6. What does this log?

::code-wrapper{language="javascript"}
```javascript
const obj = { visible: 1 };
Object.defineProperty(obj, "hidden", {
  value: 2,
  enumerable: false,
  writable: true,
  configurable: true,
});

let keys = [];
for (const k in obj) keys.push(k);

console.log(keys, Object.keys(obj), obj.hidden);
```
::

- [ ] ["visible", "hidden"] ["visible", "hidden"] 2
- [ ] ["visible"] ["visible", "hidden"] 2
- [ ] ["visible"] ["visible"] 2
- [ ] ["visible", "hidden"] ["visible"] undefined

<details>
<summary>Show Answer</summary>

**Answer:** C — ["visible"] ["visible"] 2

**Explanation:** `enumerable: false` hides `hidden` from anything that iterates *enumerable* properties — `for...in`, `Object.keys`, `Object.values`/`entries`, and `JSON.stringify` all skip it identically. The property still fully exists though: it's directly readable and (since `writable: true`) writable via `obj.hidden`, which still returns `2`. Option D wrongly assumes non-enumerable also means inaccessible; A and the mixed variants wrongly assume `for...in` and `Object.keys` diverge here — for an own property, both honor the same enumerable flag the same way.

</details>
::

::question-wrapper{language="javascript"}
### Q7. What does this log?

::code-wrapper{language="javascript"}
```javascript
function Base() {}
Base.prototype.role = "guest";

const user = Object.create(Base.prototype);
user.name = "Ana";

console.log(
  "role" in user,
  user.hasOwnProperty("role"),
  Object.hasOwn(user, "role")
);
```
::

- [ ] true false false
- [ ] true true true
- [ ] false false false
- [ ] true false true

<details>
<summary>Show Answer</summary>

**Answer:** A — true false false

**Explanation:** The `in` operator walks the *entire* prototype chain, so `"role" in user` is `true` because `role` is inherited from `Base.prototype`, even though `user` doesn't own it. `hasOwnProperty` and `Object.hasOwn` both check only the object's own properties, so both correctly return `false` for an inherited property. `Object.hasOwn(obj, key)` is the modern (ES2022) replacement for `.hasOwnProperty()` — it works even on objects with no prototype, where `.hasOwnProperty` wouldn't exist to call.

</details>
::

::question-wrapper{language="javascript"}
### Q8. What does this log?

::code-wrapper{language="javascript"}
```javascript
const defaults = { theme: "dark" };
const settings = Object.create(defaults);
settings.fontSize = 14;

const seen = [];
for (const key in settings) seen.push(key);

console.log(seen, Object.keys(settings));
```
::

- [ ] ["fontSize"] ["fontSize"]
- [ ] ["theme", "fontSize"] ["theme", "fontSize"]
- [ ] ["fontSize"] ["theme", "fontSize"]
- [ ] ["theme", "fontSize"] ["fontSize"]

<details>
<summary>Show Answer</summary>

**Answer:** D — ["theme", "fontSize"] ["fontSize"]

**Explanation:** **Debug:** `for...in` walks the prototype chain and includes any *inherited* enumerable property, so it picks up `theme` from `defaults` in addition to `settings`' own `fontSize` — this is the classic footgun that makes `for...in` risky on objects with a non-trivial prototype. `Object.keys` (and `Object.entries`/`for...of Object.entries(...)`) only ever return an object's *own* enumerable properties, giving the safer, predictable `["fontSize"]`. Prefer `Object.keys`/`entries` over `for...in` for plain data iteration.

</details>
::

::question-wrapper{language="javascript"}
### Q9. Given this code, what actually happens when it runs?

::code-wrapper{language="javascript"}
```javascript
const response = { data: null };

const a = response.data?.user?.name;
console.log(a);

const b = response.data.user?.name;
console.log(b);
```
::

- [ ] Both lines log fine; `a` and `b` are both `undefined`
- [ ] `a` logs `undefined`; the line defining `b` throws a `TypeError` before it can log
- [ ] Both lines throw a `TypeError` because `response.data` is `null`
- [ ] `a` throws first, because optional chaining requires every link in a chain to use `?.`

<details>
<summary>Show Answer</summary>

**Answer:** B — `a` logs `undefined`; the line defining `b` throws a `TypeError` before it can log

**Explanation:** **Safety:** `?.` short-circuits the moment it hits a `null`/`undefined` reference and returns `undefined` immediately, without evaluating the rest of the chain — so `response.data?.user?.name` safely yields `undefined` and `a` logs fine. `response.data.user` has no `?.` after `data`, so it accesses `.user` directly on `null`, which throws `TypeError: Cannot read properties of null (reading 'user')` before `b` is ever assigned. Optional chaining protects only the specific link it's placed on, not the whole expression.

</details>
::

::question-wrapper{language="javascript"}
### Q10. What does this log?

::code-wrapper{language="javascript"}
```javascript
const settings = { retries: 0, label: "", timeout: null };

const retries = settings.retries || 3;
const retries2 = settings.retries ?? 3;
const label = settings.label || "default";
const label2 = settings.label ?? "default";

console.log(retries, retries2, label, label2);
```
::

- [ ] 0 0 "" ""
- [ ] 3 3 "default" "default"
- [ ] 3 0 "default" ""
- [ ] 0 3 "" "default"

<details>
<summary>Show Answer</summary>

**Answer:** C — 3 0 "default" ""

**Explanation:** **Debug:** `||` falls back to its right-hand side on *any* falsy value — `0`, `""`, `false`, `NaN`, `null`, `undefined` — so both the legitimate `0` retry count and the legitimate empty-string label get incorrectly overridden. `??` only falls back on `null` or `undefined`, so `retries2` correctly stays `0` and `label2` correctly stays `""`. This is why `??` is the safer default operator for fields (counts, flags, strings) that can hold a meaningful falsy value.

</details>
::

::question-wrapper{language="javascript"}
### Q11. What is logged, and what happens when `delete total;` runs afterward in non-strict mode?

::code-wrapper{language="javascript"}
```javascript
const state = { count: 5 };
const result = delete state.count;
console.log(result, state);

let total = 10;
delete total;
console.log(total);
```
::

- [ ] `true { }` then `10` — `delete` removes object properties but is a silent no-op on variable bindings
- [ ] `true { }` then `undefined` — `delete` also removes the `total` variable itself
- [ ] `false { count: 5 }` then `10` — `delete` can't remove a property declared with a value
- [ ] `true { }` — the second `delete` throws a `SyntaxError` because `total` was declared with `let`

<details>
<summary>Show Answer</summary>

**Answer:** A — `true { }` then `10` — `delete` removes object properties but is a silent no-op on variable bindings

**Explanation:** **Debug:** `delete` operates on object properties, not bindings — it removes the configurable own property and returns `true` on success (state becomes `{}`). Note `const` only prevents *reassigning* the `state` binding, it doesn't stop mutating or deleting its properties. `delete total` targets a variable, not a property; `delete` has no power over `var`/`let`/`const` bindings, so in non-strict mode it's simply a no-op that returns `false`, leaving `total` unchanged at `10` (in strict mode, deleting a bare identifier is a `SyntaxError` at parse time, but that's a different scenario than this sloppy-mode script).

</details>
::

::question-wrapper{language="javascript"}
### Q12. What does this log?

::code-wrapper{language="javascript"}
```javascript
const a = { id: 1 };
const b = { id: 1 };
const c = a;

console.log(a === b, a === c, JSON.stringify(a) === JSON.stringify(b));
```
::

- [ ] true true true
- [ ] true false true
- [ ] false false true
- [ ] false true true

<details>
<summary>Show Answer</summary>

**Answer:** D — false true true

**Explanation:** **Idiom:** `===` compares object operands by *reference*, never by structural content — `a` and `b` are two distinct objects that merely look alike, so `a === b` is `false` even though every property matches. `c` was assigned the exact same reference as `a` (no new object was created), so `a === c` is `true`. `JSON.stringify` converts both objects to the identical string `'{"id":1}'`, and *that* string comparison is `true` — but that's comparing two strings, not testing object identity, and it breaks down for objects with functions, `undefined`, or differently-ordered keys with different serializers.

</details>
::

::question-wrapper{language="javascript"}
### Q13. What does `Object.keys(obj)` log?

::code-wrapper{language="javascript"}
```javascript
const obj = {};
obj.b = 1;
obj[2] = "two";
obj.a = 3;
obj[1] = "one";

console.log(Object.keys(obj));
```
::

- [ ] ["b", "2", "a", "1"]
- [ ] ["1", "2", "b", "a"]
- [ ] ["1", "2", "a", "b"]
- [ ] ["2", "1", "b", "a"]

<details>
<summary>Show Answer</summary>

**Answer:** B — ["1", "2", "b", "a"]

**Explanation:** **Debug:** own-property key ordering follows a fixed spec rule, not insertion order alone: integer-index-like string keys come first, sorted in *ascending numeric* order regardless of when they were added, followed by all other string keys in insertion order, then Symbol keys last. Here `"1"` and `"2"` are integer-like, so they're sorted numerically first (`1` before `2`, even though `2` was inserted first), and only then come `"b"` and `"a"` in the order they were actually assigned.

</details>
::

::question-wrapper{language="javascript"}
### Q14. What does `JSON.stringify(account)` log?

::code-wrapper{language="javascript"}
```javascript
const account = {
  _balance: 100,
  get balance() {
    return `$${this._balance}`;
  },
  set balance(value) {
    this._balance = value;
  },
};

account.balance = 250;
console.log(JSON.stringify(account));
```
::

- [ ] {"_balance":100,"balance":"$100"}
- [ ] {"balance":"$250"}
- [ ] {"_balance":250,"balance":"$250"}
- [ ] {"_balance":250}

<details>
<summary>Show Answer</summary>

**Answer:** C — {"_balance":250,"balance":"$250"}

**Explanation:** **Idiom:** assigning `account.balance = 250` invokes the setter, which stores the raw number into `_balance`. `JSON.stringify` serializes every own enumerable property — including accessor (getter) properties — and for a getter it *calls* it and embeds the returned value, so both the updated `_balance` (`250`) and the computed `balance` getter output (`"$250"`) end up in the JSON. It neither skips getters (ruling out B and D) nor uses a stale pre-assignment snapshot (ruling out A).

</details>
::

::question-wrapper{language="javascript"}
### Q15. Which statement correctly describes what happens?

::code-wrapper{language="javascript"}
```javascript
const original = {
  createdAt: new Date("2024-01-01"),
  greet: function () {},
  count: undefined,
  tag: Symbol("x"),
};

const viaJson = JSON.parse(JSON.stringify(original));
const viaClone = structuredClone(original);
```
::

- [ ] `viaJson.createdAt` becomes a string, and `greet`, `count`, and `tag` are dropped entirely; `structuredClone` throws because it can't clone a function
- [ ] `viaJson.createdAt` stays a `Date` but `greet` is dropped; `structuredClone` also preserves `createdAt` as a `Date` and drops `greet`
- [ ] Both `viaJson` and `viaClone` preserve `createdAt` as a `Date`, `greet` as a function, and `tag` as a Symbol
- [ ] `JSON.stringify` throws a `TypeError` because `Date` isn't valid JSON; `structuredClone` succeeds and preserves everything, including `greet`

<details>
<summary>Show Answer</summary>

**Answer:** A — `viaJson.createdAt` becomes a string, and `greet`, `count`, and `tag` are dropped entirely; `structuredClone` throws because it can't clone a function

**Explanation:** **Safety:** `JSON.stringify` calls `Date`'s `toJSON`, converting it to an ISO string, so the round-trip through `JSON.parse` leaves `createdAt` as a plain string, not a `Date`. It also *silently* omits any property whose value is `undefined`, a function, or a Symbol — no error, they simply vanish from the output. `structuredClone` uses the structured clone algorithm, which correctly deep-clones `Date` (and `Map`/`Set`/typed arrays/circular references) as real `Date` objects — but functions (and Symbols) are explicitly unsupported and cause it to throw a `DataCloneError`, so it can't be used as a drop-in replacement on objects that hold methods.

</details>
::

::question-wrapper{language="javascript"}
### Q16. What does this log?

::code-wrapper{language="javascript"}
```javascript
const dict = Object.create(null);
dict.apple = 1;

console.log(dict.toString);
console.log(dict.hasOwnProperty);
console.log(Object.keys(dict));
```
::

- [ ] "[object Object]" [Function: hasOwnProperty] ["apple"]
- [ ] TypeError TypeError ["apple"]
- [ ] undefined [Function: hasOwnProperty] ["apple"]
- [ ] undefined undefined ["apple"]

<details>
<summary>Show Answer</summary>

**Answer:** D — undefined undefined ["apple"]

**Explanation:** **Safety:** `Object.create(null)` builds an object with *no* prototype at all — not even `Object.prototype` — so it inherits none of the usual built-ins: `toString`, `hasOwnProperty`, `valueOf`, etc. all simply don't exist on it, and accessing them returns `undefined` rather than throwing. This makes it a good "pure dictionary" that can safely use any string key — even `"toString"` or `"__proto__"` — without colliding with inherited methods or enabling prototype pollution, but callers must use `Object.hasOwn(dict, key)` instead of `dict.hasOwnProperty(key)`, since the latter would throw `TypeError: dict.hasOwnProperty is not a function`.

</details>
::

::question-wrapper{language="javascript"}
### Q17. What does this log?

::code-wrapper{language="javascript"}
```javascript
const original = { user: { name: "Kim" }, count: 1 };
const { user, ...rest } = original;

user.name = "Lee";
rest.count = 99;

console.log(original.user.name, original.count);
```
::

- [ ] "Kim" 1
- [ ] "Lee" 1
- [ ] "Lee" 99
- [ ] "Kim" 99

<details>
<summary>Show Answer</summary>

**Answer:** B — "Lee" 1

**Explanation:** **Debug:** destructuring `user` out of `original` binds `user` to the *same* object `original.user` already points to — no copy is made for nested objects — so mutating `user.name` mutates the shared object, and `original.user.name` reads back as `"Lee"`. The rest pattern `...rest`, however, builds a brand-new object whose `count` is a freshly copied primitive value, so writing `rest.count = 99` only touches the new object and leaves `original.count` at `1`. The trap: primitives are safely copied by destructuring/spread, but nested object references are shared, not cloned.

</details>
::

::question-wrapper{language="javascript"}
### Q18. What does this log?

::code-wrapper{language="javascript"}
```javascript
const id = Symbol("id");
const user = {
  name: "Ana",
  [id]: 42,
};

console.log(Object.keys(user));
console.log(JSON.stringify(user));
console.log(user[id]);
```
::

- [ ] ["name", "id"] {"name":"Ana","id":42} 42
- [ ] ["name"] {"name":"Ana","id":42} undefined
- [ ] ["name"] {"name":"Ana"} 42
- [ ] TypeError: Symbol keys are not allowed in object literals

<details>
<summary>Show Answer</summary>

**Answer:** C — ["name"] {"name":"Ana"} 42

**Explanation:** **Idiom:** Symbol-keyed properties are unconditionally excluded from `Object.keys`/`values`/`entries`, `for...in`, and `JSON.stringify` — not because of an enumerable flag, but because Symbols are categorically skipped by all of those mechanisms. This makes Symbols useful for attaching metadata to an object that won't collide with string keys or accidentally leak into serialization or plain iteration. The property is still fully present and directly retrievable via `user[id]`, since the caller holds a reference to the exact same Symbol.

</details>
::

::question-wrapper
### Q19. In modern JS (ES2022+), which is the most robust way to check whether `obj` has an *own* property named `key`, correct even when `obj` was created with `Object.create(null)`?

- [ ] `Object.hasOwn(obj, key)`
- [ ] `obj.hasOwnProperty(key)`
- [ ] `key in obj`
- [ ] `obj[key] !== undefined`

<details>
<summary>Show Answer</summary>

**Answer:** A — `Object.hasOwn(obj, key)`

**Explanation:** **Idiom:** `Object.hasOwn` is a static method — it doesn't rely on `obj` inheriting anything, so it works correctly even on prototype-less objects, objects that shadow their own `hasOwnProperty`, or objects retrieved from `Object.create(null)`. `obj.hasOwnProperty(key)` breaks in exactly those cases, throwing `TypeError: obj.hasOwnProperty is not a function` when no prototype provides it. `key in obj` checks the *entire* prototype chain, so it produces false positives for inherited properties. `obj[key] !== undefined` is unreliable both ways: a property can exist with the value `undefined` (false negative) and a missing key also reads as `undefined` (indistinguishable).

</details>
::

::question-wrapper{language="javascript"}
### Q20. In strict mode, what is logged?

::code-wrapper{language="javascript"}
```javascript
"use strict";

const user = Object.freeze({ name: "Ana", roles: ["admin"] });

try {
  user.name = "Zoe";
  console.log("assigned");
} catch (err) {
  console.log(err.constructor.name);
}

user.roles.push("editor");
console.log(user.roles);
```
::

- [ ] "assigned" then ["admin"]
- [ ] "TypeError" then ["admin"]
- [ ] "assigned" then ["admin", "editor"]
- [ ] "TypeError" then ["admin", "editor"]

<details>
<summary>Show Answer</summary>

**Answer:** D — "TypeError" then ["admin", "editor"]

**Explanation:** **Safety:** in strict mode, assigning to a non-writable property of a frozen object throws a `TypeError` instead of failing silently (as it would in sloppy mode), so the `catch` block runs and logs `"TypeError"`. `Object.freeze`, however, is shallow — it only locks `user`'s own top-level slots (`name`, and the *reference* stored in `roles`), never the contents of objects those slots merely point to. `user.roles` is still a fully mutable array, so `push("editor")` succeeds in place, producing `["admin", "editor"]`. Freezing an object never cascades into anything it references.

</details>
::
