# 28 — TypeScript Essentials

::question-wrapper{language="typescript"}
### Q1. A `Coordinate` class is passed to a function expecting a `Point` interface, without ever declaring `implements Point`. What happens?

::code-wrapper{language="typescript"}
```typescript
interface Point {
  x: number;
  y: number;
}

class Coordinate {
  constructor(public x: number, public y: number) {}
}

function distanceFromOrigin(p: Point): number {
  return Math.sqrt(p.x ** 2 + p.y ** 2);
}

const c = new Coordinate(3, 4);
distanceFromOrigin(c);
```
::

- [ ] A compile error, because `Coordinate` never declares `implements Point`
- [ ] It compiles and runs fine — TypeScript's structural typing considers `Coordinate` assignable to `Point` because it has the same shape (an `x` and `y` of type `number`), regardless of any explicit relationship between the two types
- [ ] It compiles but throws a runtime `TypeError`, since `Coordinate` and `Point` are unrelated types
- [ ] It only compiles if `Coordinate` is renamed to `Point`

<details>
<summary>Show Answer</summary>

**Answer:** B — It compiles and runs fine — TypeScript's structural typing considers `Coordinate` assignable to `Point` because it has the same shape (an `x` and `y` of type `number`), regardless of any explicit relationship between the two types

**Explanation:** TypeScript uses structural ("duck") typing — compatibility is determined by comparing the shape (member names and types) of two types, not by their declared names or explicit inheritance relationships. Since `Coordinate` has `x: number` and `y: number`, it structurally satisfies `Point` even though it never says `implements Point`. Contrast this with nominal typing systems like Java/C#, where a class must explicitly declare `implements Point` (or extend it) to be assignable — shape alone isn't enough there. Option A assumes nominal-typing rules that don't apply to TS. Option C is wrong because type checking is entirely a compile-time construct (see Q10 on type erasure) — there's no runtime shape check that could throw; if it type-checks, it runs like plain JS. Option D is a non-sequitur; renaming has nothing to do with structural compatibility.

</details>
::

::question-wrapper{language="typescript"}
### Q2. A codebase defines a `Shape` interface and a class hierarchy (`BaseShape` → `Circle`) implementing it. A teammate then passes a plain object literal — never part of that hierarchy — into a function typed to accept `Shape`. Why does this compile?

::code-wrapper{language="typescript"}
```typescript
interface Shape {
  area(): number;
}

abstract class BaseShape implements Shape {
  abstract area(): number;
}

class Circle extends BaseShape {
  constructor(private radius: number) { super(); }
  area() { return Math.PI * this.radius ** 2; }
}

function totalArea(shapes: Shape[]): number {
  return shapes.reduce((sum, s) => sum + s.area(), 0);
}

const adHoc = { area: () => 42 };

totalArea([new Circle(2), adHoc]);
```
::

- [ ] It compiles cleanly — `Shape` is just an interface describing a required shape (`area(): number`), and any value satisfying that shape is assignable, whether or not it's ever part of the class hierarchy
- [ ] It's a compile error — `adHoc` must extend `BaseShape` to be treated as a `Shape`
- [ ] It compiles, but `adHoc.area()` will be `undefined` at runtime since it isn't a real class instance
- [ ] It compiles only because `BaseShape` is declared `abstract`

<details>
<summary>Show Answer</summary>

**Answer:** A — It compiles cleanly — `Shape` is just an interface describing a required shape (`area(): number`), and any value satisfying that shape is assignable, whether or not it's ever part of the class hierarchy

**Explanation:** This is the same structural-typing rule as Q1, applied to a scenario that specifically tempts a "class hierarchy" mental model — a developer coming from Java/C# might assume that because `Circle` participates in an explicit `implements`/`extends` chain, only members of that hierarchy can be `Shape`s. But `interface Shape` only constrains what members must exist (a callable `area(): number`); a completely unrelated object literal that happens to have that member is just as valid a `Shape` as any class instance. Options B and D invent nominal-typing rules TS doesn't enforce. Option C is wrong because `adHoc.area` is a real function — it executes normally and returns `42`; nothing is "missing" at runtime just because the object skipped the class hierarchy.

</details>
::

::question-wrapper{language="typescript"}
### Q3. Inside the `if (typeof id === "string")` block, what does TypeScript infer the type of `id` to be, and why is this necessary?

::code-wrapper{language="typescript"}
```typescript
function formatId(id: string | number): string {
  if (typeof id === "string") {
    return id.toUpperCase();
  }
  return id.toFixed(2);
}
```
::

- [ ] `id` is still `string | number` inside the block; TypeScript doesn't narrow based on `typeof` checks
- [ ] `id` becomes `any` inside the block because `typeof` returns a string at runtime
- [ ] `id` is narrowed to `string` inside the block (and implicitly to `number` after it, since `toFixed` requires it) — the `typeof` check acts as a type guard, letting TypeScript prove which branch of the union applies before calling a method unique to that branch
- [ ] TypeScript throws a compile error because `id.toUpperCase()` isn't defined on `number`

<details>
<summary>Show Answer</summary>

**Answer:** C — `id` is narrowed to `string` inside the block (and implicitly to `number` after it, since `toFixed` requires it) — the `typeof` check acts as a type guard, letting TypeScript prove which branch of the union applies before calling a method unique to that branch

**Explanation:** TypeScript performs control-flow-based type narrowing: a `typeof id === "string"` check inside an `if` is recognized as a type guard, so within that branch the compiler narrows `id`'s static type from `string | number` down to just `string`, permitting `toUpperCase()` (a string-only method) without a type error. After the `if` returns, the compiler also narrows the remaining code to the `number` branch by elimination, permitting `toFixed()`. Without this narrowing, calling either method directly on the union type would fail to compile, since neither method exists on both members of the union. Option D describes what would happen *without* the guard.

</details>
::

::question-wrapper{language="typescript"}
### Q4. Why does `err.field` type-check inside the `if` branch, given that `handleError`'s parameter is typed as the base `Error`?

::code-wrapper{language="typescript"}
```typescript
class ValidationError extends Error {
  constructor(public field: string) {
    super(`Invalid field: ${field}`);
  }
}

function handleError(err: Error) {
  if (err instanceof ValidationError) {
    console.log(err.field);
  } else {
    console.log(err.message);
  }
}
```
::

- [ ] It doesn't type-check — `field` isn't a member of `Error`, so this is a compile error regardless of the `instanceof` check
- [ ] `instanceof` is purely a runtime check with no effect on TypeScript's static types; `err` stays typed as `Error` throughout, so `err.field` compiles only because of an implicit `any` fallback
- [ ] `field` type-checks because every `Error` instance has a `field` property by default in TypeScript's lib definitions
- [ ] `instanceof ValidationError` is a recognized type guard, so inside that branch TypeScript narrows `err`'s static type from `Error` down to `ValidationError`, exposing the `field` property that only exists on the subclass

<details>
<summary>Show Answer</summary>

**Answer:** D — `instanceof ValidationError` is a recognized type guard, so inside that branch TypeScript narrows `err`'s static type from `Error` down to `ValidationError`, exposing the `field` property that only exists on the subclass

**Explanation:** **Debug:** like `typeof`, `instanceof` is one of TypeScript's built-in type guards for control-flow narrowing: checking `err instanceof ValidationError` proves (to the compiler, matching real class-based runtime behavior) that inside that branch `err` is specifically a `ValidationError`, not just any `Error`, so members unique to the subclass (`field`) become accessible. Option B is wrong — TS does perform *static* narrowing from `instanceof`, this isn't dynamic/`any`-typed access. Option C invents a nonexistent base `Error` member. Option A ignores narrowing entirely.

</details>
::

::question-wrapper{language="typescript"}
### Q5. What does `greet("")` log, and what's the underlying reason?

::code-wrapper{language="typescript"}
```typescript
function greet(name?: string) {
  if (name) {
    console.log(name.toUpperCase());
  } else {
    console.log("Hello, stranger");
  }
}

greet("");
```
::

- [ ] `""` (empty string) is truthy in JavaScript, so it logs the uppercased (still empty) string
- [ ] `"Hello, stranger"` — a truthiness check like `if (name)` treats an empty string as falsy just like `undefined`, so passing an empty string (a perfectly valid, non-`undefined` argument) is silently routed into the "no name" branch, which is rarely the intent
- [ ] A compile error, because `name` is optional and TypeScript refuses to call `.toUpperCase()` on a possibly-`undefined` value even inside the truthy branch
- [ ] `undefined`, because `greet` was called with an empty string instead of no arguments

<details>
<summary>Show Answer</summary>

**Answer:** B — `"Hello, stranger"` — a truthiness check like `if (name)` treats an empty string as falsy just like `undefined`, so passing an empty string (a perfectly valid, non-`undefined` argument) is silently routed into the "no name" branch, which is rarely the intent

**Explanation:** **Debug:** `if (name)` narrows away `undefined`, but it narrows based on JavaScript's actual runtime truthiness rules, not just "is it `undefined`" — and `""` is one of JS's falsy values (alongside `0`, `NaN`, `null`, `undefined`). So a caller who deliberately passes an empty string (a legitimate string, just an empty one) gets routed into the `else` branch as if no name were given at all, which is almost never the intent behind `if (name)`. TypeScript's narrowing correctly mirrors this: it narrows `name` to `string` in the truthy branch (option C's premise is wrong — it compiles fine), but that says nothing about which runtime value ends up there. The safer, more explicit guard when the intent is "was an argument supplied" is `if (name !== undefined)`.

</details>
::

::question-wrapper{language="typescript"}
### Q6. What role does the shared `kind` literal property play here, and why does `shape.radius` compile inside the `"circle"` case without a separate `typeof`/`instanceof` check?

::code-wrapper{language="typescript"}
```typescript
type Circle = { kind: "circle"; radius: number };
type Square = { kind: "square"; side: number };
type Shape = Circle | Square;

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
  }
}
```
::

- [ ] `kind` is a discriminant (tag) property with a distinct literal type per union member; switching on it lets TypeScript narrow `shape` to the exact member type whose `kind` literal matches the case, exposing that member's unique properties (`radius`, `side`) without any additional guard
- [ ] `kind` has no special meaning to TypeScript — the code compiles by coincidence because `radius` and `side` happen to both be `number`
- [ ] TypeScript merges `Circle` and `Square` into one type with all properties optional, so `shape.radius` is always accessible but may be `undefined`
- [ ] The `switch` must be replaced with `if (shape.kind === "circle")` — `switch` statements don't support narrowing in TypeScript

<details>
<summary>Show Answer</summary>

**Answer:** A — `kind` is a discriminant (tag) property with a distinct literal type per union member; switching on it lets TypeScript narrow `shape` to the exact member type whose `kind` literal matches the case, exposing that member's unique properties (`radius`, `side`) without any additional guard

**Explanation:** **Idiom:** this pattern — a union of object types that all share one property typed as a distinct literal — is called a discriminated (or "tagged") union. TypeScript specifically recognizes narrowing on that shared literal property, including inside `switch` statements: matching `case "circle"` proves `shape` is specifically the `Circle` member of the union in that branch, so `radius` (a property only `Circle` has) is safely accessible, and likewise `side` only inside `case "square"`. This is idiomatic TS for modeling variant data (API responses, Redux actions, AST nodes) far more safely than `typeof`/`instanceof`-based narrowing when the members aren't classes. Option C is wrong; the union stays a true union, not a merged optional-everything type. Option D is false; `switch` narrowing on discriminants is a well-supported, common pattern.

</details>
::

::question-wrapper{language="typescript"}
### Q7. Why does this compile, and what happens if `type User = { name: string }` were declared twice instead of `interface User`?

::code-wrapper{language="typescript"}
```typescript
interface User {
  name: string;
}

interface User {
  age: number;
}

const u: User = { name: "Asha", age: 30 };
```
::

- [ ] Both `interface` and `type` are 100% interchangeable in every situation, including repeated declarations
- [ ] The `interface` version is a compile error too — TypeScript doesn't allow redeclaring any named type
- [ ] TypeScript performs *declaration merging* on repeated `interface` declarations with the same name, combining their members into one type (`{ name: string; age: number }`); a repeated `type User = ...` alias, by contrast, is a compile error — duplicate identifier — since type aliases cannot be reopened
- [ ] `type` aliases support merging but `interface` declarations don't

<details>
<summary>Show Answer</summary>

**Answer:** C — TypeScript performs *declaration merging* on repeated `interface` declarations with the same name, combining their members into one type (`{ name: string; age: number }`); a repeated `type User = ...` alias, by contrast, is a compile error — duplicate identifier — since type aliases cannot be reopened

**Explanation:** Interfaces and type aliases are largely interchangeable for describing object shapes, but they differ in one important way: declaration merging. Multiple `interface` declarations sharing the same name in the same scope are automatically merged into a single interface containing all their members — a mechanism TS itself relies on for things like augmenting global types (e.g., extending `Window`). Type aliases have no equivalent; a second `type User = { age: number }` in the same scope is a hard compile error (`Duplicate identifier 'User'`), because a type alias is a fixed name bound once, not an open, extensible declaration. Option D reverses the actual rule.

</details>
::

::question-wrapper{language="typescript"}
### Q8. What's the key compile-time difference between `processA` and `processB`?

::code-wrapper{language="typescript"}
```typescript
function processA(value: any) {
  value.toUpperCase();
}

function processB(value: unknown) {
  value.toUpperCase();
}
```
::

- [ ] There's no difference — both compile and behave identically
- [ ] `processB` compiles and `processA` doesn't, because `unknown` is more permissive than `any`
- [ ] Both fail to compile, since neither `any` nor `unknown` allow method calls
- [ ] `processA` compiles because `any` opts the value out of type checking entirely, so any member access is allowed (and unverified) — `processB` fails to compile because `unknown` requires the value's type to be narrowed (e.g., via `typeof`) before any property or method can be accessed on it

<details>
<summary>Show Answer</summary>

**Answer:** D — `processA` compiles because `any` opts the value out of type checking entirely, so any member access is allowed (and unverified) — `processB` fails to compile because `unknown` requires the value's type to be narrowed (e.g., via `typeof`) before any property or method can be accessed on it

**Explanation:** **Safety:** `any` disables TypeScript's type checking for that value completely — every property access, method call, and assignment is allowed without verification, which defeats the purpose of using TypeScript at all for that value. `unknown` is the type-safe counterpart: it can hold any value (just like `any`), but the compiler refuses to let you do anything with it — call a method, access a property, use it as an argument — until you've narrowed it to a more specific type via `typeof`, `instanceof`, a custom type guard, or an assertion. `processB` fails to compile (`'value' is of type 'unknown'`) precisely because that safety net is doing its job — that's a feature, not a limitation.

</details>
::

::question-wrapper{language="typescript"}
### Q9. The `console.log(data.name)` line fails to compile. What's the idiomatic fix, and why is typing `getUser`'s return as `unknown` (rather than `any`) the better design choice here?

::code-wrapper{language="typescript"}
```typescript
async function getUser(id: string): Promise<unknown> {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
}

const data = await getUser("42");
console.log(data.name);
```
::

- [ ] Narrow `data` first — e.g. `if (typeof data === "object" && data !== null && "name" in data)`, or validate it against a schema — before accessing `.name`; typing external/untrusted data (like a parsed JSON API response) as `unknown` forces every consumer to verify its actual shape before use, catching malformed responses at the access site instead of letting bad data silently flow through the app as `any`
- [ ] Change the return type to `any` so `data.name` compiles without any extra work
- [ ] Add a non-null assertion: `data!.name`
- [ ] Cast to `any` at the call site only: `(data as any).name`

<details>
<summary>Show Answer</summary>

**Answer:** A — Narrow `data` first — e.g. `if (typeof data === "object" && data !== null && "name" in data)`, or validate it against a schema — before accessing `.name`; typing external/untrusted data (like a parsed JSON API response) as `unknown` forces every consumer to verify its actual shape before use, catching malformed responses at the access site instead of letting bad data silently flow through the app as `any`

**Explanation:** **Safety:** `data` genuinely could be anything at runtime — the API could return malformed JSON, an error object, or a shape that doesn't match assumptions — so `unknown` is the honest, safe type for it. Options B and D "solve" the compile error by disabling type checking entirely (`any`), which reintroduces exactly the class of bug `unknown` exists to prevent: if the API's shape changes, `.name` silently returns `undefined` or throws deep inside unrelated code instead of failing where the assumption was made. Option C's non-null assertion (see Q16) only suppresses a null/undefined check — it does nothing about `data`'s completely unknown shape. The idiomatic pattern is to narrow (a type guard, an `in` check, or a runtime validation library) right where the `unknown` value first enters typed code, converting it into a trusted type from that point forward.

</details>
::

::question-wrapper{language="typescript"}
### Q10. Why does `typeof value === "ApiResponse"` fail to do what the function name suggests, and what will `typeof value` actually ever evaluate to?

::code-wrapper{language="typescript"}
```typescript
interface ApiResponse {
  status: number;
  body: string;
}

function isApiResponse(value: unknown) {
  return typeof value === "ApiResponse";
}
```
::

- [ ] `typeof value` can only ever be one of JavaScript's runtime type tags (`"string"`, `"number"`, `"object"`, `"function"`, etc.) — it can never equal `"ApiResponse"`, because `interface`/`type` declarations are a compile-time-only construct that TypeScript *erases* completely when generating JavaScript; there is no runtime trace of `ApiResponse` left to check against
- [ ] It works correctly — TypeScript compiles interface names into runtime string tags that `typeof` can check against
- [ ] This is a compile error, since `typeof` cannot be used inside a function body
- [ ] `typeof value === "ApiResponse"` works, but only in strict mode

<details>
<summary>Show Answer</summary>

**Answer:** A — `typeof value` can only ever be one of JavaScript's runtime type tags (`"string"`, `"number"`, `"object"`, `"function"`, etc.) — it can never equal `"ApiResponse"`, because `interface`/`type` declarations are a compile-time-only construct that TypeScript *erases* completely when generating JavaScript; there is no runtime trace of `ApiResponse` left to check against

**Explanation:** **Debug:** TypeScript's type system exists purely to catch errors during compilation — every `interface`, type alias, and generic parameter is erased entirely when TS is transpiled to plain JavaScript, leaving zero runtime footprint or metadata. Because of this, JavaScript's `typeof` operator (which reflects real runtime type tags: `"string"`, `"number"`, `"boolean"`, `"object"`, `"function"`, `"undefined"`, `"symbol"`, `"bigint"`) has no way to know about interface names — `ApiResponse` never existed at runtime to compare against, so this condition is always `false`. To actually check "does this value look like an `ApiResponse` at runtime," you need a real runtime type guard that inspects the object's actual properties, e.g. `typeof value === "object" && value !== null && "status" in value && "body" in value` — reconstructing at runtime what the type system already knew at compile time.

</details>
::

::question-wrapper{language="typescript"}
### Q11. What does this log, and what's the practical difference between `timeout?: number` and `timeout: number | undefined`?

::code-wrapper{language="typescript"}
```typescript
interface Settings {
  timeout?: number;
}

const s1: Settings = {};
const s2: Settings = { timeout: undefined };

console.log("timeout" in s1, "timeout" in s2);
```
::

- [ ] `true true` — an optional property always exists on the object, just possibly holding `undefined`
- [ ] `false false` — neither object has a `timeout` key set to anything
- [ ] `false true` — `timeout?: number` means the *property itself* may be entirely absent (which is what happens for `s1`), whereas explicitly assigning `timeout: undefined` (allowed because the property's value type includes `undefined`) actually creates the key with an `undefined` value, which `in` reports as present
- [ ] `true false` — the reverse of the above

<details>
<summary>Show Answer</summary>

**Answer:** C — `false true` — `timeout?: number` means the *property itself* may be entirely absent (which is what happens for `s1`), whereas explicitly assigning `timeout: undefined` (allowed because the property's value type includes `undefined`) actually creates the key with an `undefined` value, which `in` reports as present

**Explanation:** A `?`-marked property means the key is optional — it may or may not exist on the object at all. `s1 = {}` never sets the key, so `"timeout" in s1` is `false`. `s2 = { timeout: undefined }` explicitly creates the key with the value `undefined`, so `"timeout" in s2` is `true`, even though `timeout` reads as `undefined` either way. This distinction is normally invisible (both read as `undefined` via `s.timeout`), but becomes strictly enforced under `strict` mode's `exactOptionalPropertyTypes` flag, which disallows explicitly assigning `undefined` to a `?`-optional property specifically because it conflates "not provided" with "provided as `undefined`" — two states that `Object.keys`, `in`, and `JSON.stringify` all treat differently (a key explicitly set to `undefined` still shows up in `Object.keys`/`for...in`, but is dropped by `JSON.stringify`).

</details>
::

::question-wrapper{language="typescript"}
### Q12. What's the practical type-safety difference between `a` and `b` after these calls?

::code-wrapper{language="typescript"}
```typescript
function identity(x: any): any {
  return x;
}

function identityGeneric<T>(x: T): T {
  return x;
}

const a = identity(42);
const b = identityGeneric(42);
```
::

- [ ] None — both `a` and `b` end up typed `number`, since `42` is a number literal in both calls
- [ ] `b` is `any` too, since `T` is just an alias for `any` under the hood
- [ ] This is a compile error — generic functions cannot be called without explicitly specifying `<T>`
- [ ] `a` is typed `any` (whatever `identity` returns, since it accepts and returns `any`), losing all type information about the input — `b` is typed `number`, because the generic `identityGeneric<T>` captures the actual argument type (`T` is inferred as `number` from the call site) and returns that same specific type, preserving type safety without hardcoding `number` anywhere

<details>
<summary>Show Answer</summary>

**Answer:** D — `a` is typed `any` (whatever `identity` returns, since it accepts and returns `any`), losing all type information about the input — `b` is typed `number`, because the generic `identityGeneric<T>` captures the actual argument type (`T` is inferred as `number` from the call site) and returns that same specific type, preserving type safety without hardcoding `number` anywhere

**Explanation:** The `any`-based `identity` discards all type information — the compiler has no way to know what `identity(42)` returns beyond "could be anything," so `a` is `any`, and subsequent misuse (`a.someMethodThatDoesntExist()`) would compile without complaint. `identityGeneric<T>` instead lets the compiler infer `T` from the actual argument at each call site — `identityGeneric(42)` infers `T = number`, so the return type is genuinely `number`, not `any`, preserving full type safety without writing a separate overload for every possible input type. This is exactly why generics exist: reusable, type-safe code instead of reaching for `any` to make a function "work with anything." Option C is false — TypeScript infers `T` automatically in the vast majority of cases; explicit `<T>` is only needed when inference is ambiguous or impossible.

</details>
::

::question-wrapper{language="typescript"}
### Q13. Why does this fail to compile, and what's the idiomatic fix?

::code-wrapper{language="typescript"}
```typescript
function getLength<T>(item: T): number {
  return item.length;
}
```
::

- [ ] It compiles fine — generics automatically allow any property access
- [ ] `T` is unconstrained, so it could be instantiated with any type at all (a `number`, a `boolean`) that has no `.length` property; TypeScript rejects the access because it can't prove `length` exists on an arbitrary `T` — the fix is to constrain `T`, e.g. `function getLength<T extends { length: number }>(item: T): number`
- [ ] The fix is to change `T` to `any`
- [ ] Generic type parameters can only be used as function parameters, never in the return type or body

<details>
<summary>Show Answer</summary>

**Answer:** B — `T` is unconstrained, so it could be instantiated with any type at all (a `number`, a `boolean`) that has no `.length` property; TypeScript rejects the access because it can't prove `length` exists on an arbitrary `T` — the fix is to constrain `T`, e.g. `function getLength<T extends { length: number }>(item: T): number`

**Explanation:** A bare `<T>` places no restrictions on what `T` can be — callers could instantiate it with `number`, `boolean`, or any other type lacking a `length` property, so the compiler must reject `item.length` as unsafe for the general case, even though most real call sites (strings, arrays) do have one. Adding a constraint (`T extends { length: number }`, or an existing interface like `ArrayLike<unknown>`) tells the compiler "`T` can be any type, but only ones that include this member," which both permits `item.length` inside the function and still preserves `T`'s specific inferred type at each call site — unlike option C's `any`, which would compile but throw away all type safety, defeating the point of using a generic in the first place.

</details>
::

::question-wrapper{language="typescript"}
### Q14. Why does `x` compile with only a `name` property, but `y` requires both `name` and `createdAt`?

::code-wrapper{language="typescript"}
```typescript
interface Timestamped {
  createdAt: Date;
}

interface Named {
  name: string;
}

type A = Timestamped | Named;
type B = Timestamped & Named;

const x: A = { name: "widget" };
const y: B = { name: "widget", createdAt: new Date() };
```
::

- [ ] `A` (`Timestamped | Named`) is a union — a value only needs to satisfy *one* of the member types, so an object with just `name` (satisfying `Named`) is valid; `B` (`Timestamped & Named`) is an intersection — a value must satisfy *all* member types simultaneously, so it needs every property from both `Timestamped` and `Named` at once
- [ ] This is backwards — `|` should require both properties and `&` should require just one
- [ ] Both `A` and `B` require every property from every member type; `|` and `&` are just stylistic aliases for each other
- [ ] `x` fails to compile because `Timestamped`'s `createdAt` is missing

<details>
<summary>Show Answer</summary>

**Answer:** A — `A` (`Timestamped | Named`) is a union — a value only needs to satisfy *one* of the member types, so an object with just `name` (satisfying `Named`) is valid; `B` (`Timestamped & Named`) is an intersection — a value must satisfy *all* member types simultaneously, so it needs every property from both `Timestamped` and `Named` at once

**Explanation:** `|` (union) describes "this value is one of these types" — it only needs to be a valid instance of at least one member, so `{ name: "widget" }` alone satisfies `Timestamped | Named` by matching the `Named` side. `&` (intersection) describes "this value is all of these types combined" — structurally, the resulting type has every member from every constituent, so a value must supply `createdAt` and `name` to satisfy `Timestamped & Named`. This mirrors set-theory intuition once you think of each type as the set of values that satisfy it: union is the set union (bigger, easier to satisfy), intersection is the set intersection (smaller, stricter — a value must qualify for every constituent set at once).

</details>
::

::question-wrapper{language="typescript"}
### Q15. What happens when this code runs?

::code-wrapper{language="typescript"}
```typescript
interface Config {
  retries: number;
  timeout: number;
}

const raw = JSON.parse('{"retries": 3}');
const config = raw as Config;

console.log(config.timeout.toFixed(0));
```
::

- [ ] TypeScript's compiler validates `raw`'s actual shape against `Config` before allowing the assertion, so this fails to compile since `timeout` is missing
- [ ] `config.timeout` is automatically set to `0` as a default value, since `Config` declares it as `number`
- [ ] It compiles cleanly (the `as Config` assertion only tells the compiler "trust me, treat this as a `Config`" without performing any runtime check), but crashes at runtime with a `TypeError: Cannot read properties of undefined (reading 'toFixed')`, because the real parsed object has no `timeout` property
- [ ] It compiles and runs fine — `JSON.parse` always produces objects matching the asserted type

<details>
<summary>Show Answer</summary>

**Answer:** C — It compiles cleanly (the `as Config` assertion only tells the compiler "trust me, treat this as a `Config`" without performing any runtime check), but crashes at runtime with a `TypeError: Cannot read properties of undefined (reading 'toFixed')`, because the real parsed object has no `timeout` property

**Explanation:** **Safety:** a type assertion (`as Type`) is purely a compiler-level instruction — it tells TypeScript "stop inferring, trust that this value is of this type," and performs zero runtime validation or transformation. Since `raw` came from `JSON.parse` (whose return type is genuinely `any`, with no real guarantee about shape), the assertion silently forces the type to `Config` even though the actual parsed object (`{ retries: 3 }`) has no `timeout` key at all. The compiler is satisfied, so it compiles cleanly, but at runtime `config.timeout` is `undefined`, and calling `.toFixed(0)` on `undefined` throws. This is precisely the class of bug real type checking (via a validation library, or narrowing an `unknown` value with actual runtime guards) would have caught before the crash — an assertion is a promise to the compiler, not a guarantee it enforces.

</details>
::

::question-wrapper{language="typescript"}
### Q16. What does the `!` after `findUser("missing-id")` do, and what's the risk?

::code-wrapper{language="typescript"}
```typescript
function findUser(id: string): { name: string } | undefined {
  return db.get(id);
}

const user = findUser("missing-id")!;
console.log(user.name);
```
::

- [ ] It performs a runtime check that throws a clear, descriptive error if the value is `undefined`, before `.name` is accessed
- [ ] It converts `undefined` to `null` automatically so `.name` returns `null` instead of throwing
- [ ] It's equivalent to `?.`, safely short-circuiting to `undefined` if `user` is nullish
- [ ] The non-null assertion operator only suppresses TypeScript's compile-time warning that the value could be `undefined` — it inserts *no* runtime check at all; if `findUser` genuinely returns `undefined` for this id (as its own signature admits it can), `user.name` still crashes at runtime with a `TypeError`, just without the compiler ever having warned about it

<details>
<summary>Show Answer</summary>

**Answer:** D — The non-null assertion operator only suppresses TypeScript's compile-time warning that the value could be `undefined` — it inserts *no* runtime check at all; if `findUser` genuinely returns `undefined` for this id (as its own signature admits it can), `user.name` still crashes at runtime with a `TypeError`, just without the compiler ever having warned about it

**Explanation:** **Debug:** the `!` (non-null assertion) operator is purely a compile-time instruction telling TypeScript "I know this value isn't null/undefined here, stop warning me" — it compiles to literally nothing in the emitted JavaScript, adding zero runtime protection. `findUser`'s own return type (`{ name: string } | undefined`) admits it can return `undefined`, and asserting it away with `!` doesn't change that reality — it just silences the one warning that would have reminded a developer to handle that case. If `"missing-id"` genuinely isn't found, `user` is `undefined` at runtime and `user.name` throws exactly the crash the type system was trying to prevent, except now with no compile-time trail pointing at the unsafe access. Option C confuses `!` with the optional chaining operator `?.`, which does add a real runtime short-circuit check — the opposite behavior.

</details>
::

::question-wrapper{language="typescript"}
### Q17. What's a key practical difference between the `enum` and the string literal union, especially regarding what ends up in the compiled JavaScript output?

::code-wrapper{language="typescript"}
```typescript
enum Status {
  Active,
  Inactive,
}

type StatusLiteral = "active" | "inactive";

console.log(Status);
```
::

- [ ] A numeric `enum` like `Status` compiles into a real runtime object (roughly `{ 0: "Active", 1: "Inactive", Active: 0, Inactive: 1 }`) that exists in the output JavaScript and can be inspected/iterated at runtime, whereas `StatusLiteral` is purely a compile-time type — it's erased entirely, leaving no runtime object, variable, or footprint whatsoever
- [ ] There's no difference — both `enum` and a union of string literals are purely compile-time constructs that vanish entirely from the emitted JavaScript
- [ ] `StatusLiteral` produces a runtime object too, just with string keys instead of numeric ones
- [ ] `enum` members are always strings by default; `Status.Active` evaluates to `"Active"` unless numbers are given explicitly

<details>
<summary>Show Answer</summary>

**Answer:** A — A numeric `enum` like `Status` compiles into a real runtime object (roughly `{ 0: "Active", 1: "Inactive", Active: 0, Inactive: 1 }`) that exists in the output JavaScript and can be inspected/iterated at runtime, whereas `StatusLiteral` is purely a compile-time type — it's erased entirely, leaving no runtime object, variable, or footprint whatsoever

**Explanation:** **Idiom:** unlike almost every other TS-only construct (interfaces, type aliases, generic parameters), `enum` is one of the few features that isn't purely erased — it compiles to an actual runtime JavaScript object with both forward (`Active: 0`) and, for numeric enums, reverse (`0: "Active"`) mappings, meaning it has a real bundle-size and runtime cost, and can be accidentally used as a value in ways type aliases never can. `type StatusLiteral = "active" | "inactive"`, by contrast, follows the normal type-erasure rule (see Q10) — it exists purely for the compiler and leaves nothing behind in the emitted JS. This is a major reason idiomatic modern TypeScript often prefers string literal unions over enums for simple tag/status values: no runtime overhead, no reverse-mapping surprises, and values serialize naturally as plain strings. Option D is also factually wrong — unqualified enum members default to auto-incrementing numbers (`0`, `1`, `2`, ...), not their name as a string; that requires an explicit string enum (`enum Status { Active = "active" }`).

</details>
::

::question-wrapper{language="typescript"}
### Q18. What does this log, given that `x` is declared `readonly`?

::code-wrapper{language="typescript"}
```typescript
interface Point {
  readonly x: number;
  readonly y: number;
}

function mutate(p: Point) {
  (p as any).x = 999;
}

const origin: Point = { x: 0, y: 0 };
mutate(origin);
console.log(origin.x);
```
::

- [ ] `999` — `readonly` is a compile-time-only constraint enforced by the type checker; it has no runtime representation or enforcement whatsoever, so casting to `any` (which disables type checking entirely) removes the only thing stopping the mutation, and the underlying plain JS object remains fully mutable
- [ ] `0` — `readonly` prevents the mutation from ever taking effect, even through an `as any` cast, since TypeScript enforces immutability at runtime too
- [ ] A runtime `TypeError` is thrown, since `readonly` properties are implemented as non-writable using `Object.defineProperty` under the hood
- [ ] `undefined`, because reassigning a `readonly` property deletes it instead of mutating it

<details>
<summary>Show Answer</summary>

**Answer:** A — `999` — `readonly` is a compile-time-only constraint enforced by the type checker; it has no runtime representation or enforcement whatsoever, so casting to `any` (which disables type checking entirely) removes the only thing stopping the mutation, and the underlying plain JS object remains fully mutable

**Explanation:** **Safety:** like most of TypeScript's type-level features, `readonly` exists solely to catch accidental mutation during compilation — `origin.x = 999` directly would indeed fail to compile (`Cannot assign to 'x' because it is a read-only property`). But it compiles to nothing in the emitted JavaScript (no `Object.freeze`, no `defineProperty`, no runtime check) — the underlying JS object is exactly as mutable as it would be without `readonly` at all. Casting through `as any` (or receiving the object from untyped/JS-authored code TypeScript can't check) removes the compiler's ability to object, and the assignment goes through as ordinary property mutation, logging `999`. Genuine runtime immutability requires a separate mechanism like `Object.freeze` (with its own shallow-only caveat), not the `readonly` type modifier.

</details>
::

::question-wrapper{language="typescript"}
### Q19. Under TypeScript's modern `strict` defaults, this fails to compile. Why, and what's the idiomatic fix?

::code-wrapper{language="typescript"}
```typescript
try {
  riskyOperation();
} catch (err) {
  console.log(err.message);
}
```
::

- [ ] It shouldn't fail — `catch` clause variables are always typed `Error` in TypeScript, so `err.message` is always safe
- [ ] The fix is to type the parameter explicitly as `catch (err: Error)`, which TypeScript allows and enforces at runtime
- [ ] Since TS 4.4, `catch` clause variables default to type `unknown` (reflecting the reality that JavaScript allows `throw`ing *any* value, not just `Error` instances) rather than `any`; `err.message` fails because `unknown` requires narrowing first — the fix is to check the actual shape, e.g. `if (err instanceof Error) console.log(err.message);`
- [ ] This is a syntax error; `catch` clauses cannot have their variable's type inferred at all

<details>
<summary>Show Answer</summary>

**Answer:** C — Since TS 4.4, `catch` clause variables default to type `unknown` (reflecting the reality that JavaScript allows `throw`ing *any* value, not just `Error` instances) rather than `any`; `err.message` fails because `unknown` requires narrowing first — the fix is to check the actual shape, e.g. `if (err instanceof Error) console.log(err.message);`

**Explanation:** **Idiom:** JavaScript places no restriction on what can be thrown — `throw "just a string"` or `throw 42` are both completely legal, so a caught value's real runtime type is never guaranteed to be `Error`. Older TypeScript defaulted `catch` variables to `any` (silently unsafe); modern TypeScript (with `useUnknownInCatchVariables`, on by default under `strict` since TS 4.4) types them as `unknown` instead, correctly reflecting that uncertainty and forcing a narrowing check — typically `if (err instanceof Error)` — before accessing anything `Error`-specific like `.message`. Option B is a common but invalid attempt: TypeScript explicitly disallows annotating a `catch` variable with anything other than `any` or `unknown`, precisely because the compiler cannot guarantee what was actually thrown matches a narrower annotation.

</details>
::

::question-wrapper{language="typescript"}
### Q20. `Shape` later gains a third variant, `{ kind: "triangle"; base: number; height: number }`, but the `switch` in `area` isn't updated. What's the benefit of the `default` branch's `const exhaustiveCheck: never = shape;` line?

::code-wrapper{language="typescript"}
```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.side ** 2;
    default:
      const exhaustiveCheck: never = shape;
      return exhaustiveCheck;
  }
}
```
::

- [ ] Nothing changes — `never` is just a stylistic type with no compile-time effect
- [ ] It causes a runtime error only when `area` is actually called with a triangle shape, not at compile time
- [ ] `never` behaves identically to `unknown` here — it accepts any narrowed type without complaint
- [ ] It causes a compile error the moment `triangle` is added: inside the `default` branch, `shape` is now narrowed to the *unhandled* remaining member (`{ kind: "triangle"; ... }`), which is not assignable to `never` (a type with no valid values) — this deliberately fails compilation and flags the missing `case` at the exact spot it was forgotten, rather than silently returning `undefined` at runtime for triangles

<details>
<summary>Show Answer</summary>

**Answer:** D — It causes a compile error the moment `triangle` is added: inside the `default` branch, `shape` is now narrowed to the *unhandled* remaining member (`{ kind: "triangle"; ... }`), which is not assignable to `never` (a type with no valid values) — this deliberately fails compilation and flags the missing `case` at the exact spot it was forgotten, rather than silently returning `undefined` at runtime for triangles

**Explanation:** **Idiom:** `never` is the type with no possible values — nothing (except another `never`) is assignable to it, which makes it a useful compile-time trick. When every case in the `switch` is handled, TypeScript's narrowing eliminates every union member by the time control reaches `default`, so `shape`'s narrowed type there really is `never`, and the assignment type-checks at zero cost. The moment a new variant (`triangle`) is added to `Shape` without a matching `case`, `shape` in the `default` branch narrows to the remaining unhandled `{ kind: "triangle"; ... }` instead of `never` — and assigning that object to a variable typed `never` is a compile error, forcing a developer to notice and handle the new case immediately, at compile time, rather than discovering the gap only when a triangle silently returns `undefined` in production. This "exhaustiveness check" pattern is one of the most valuable idioms discriminated unions unlock over plain object shapes.

</details>
::
