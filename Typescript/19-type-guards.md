# Type guards

A **type guard** is a runtime check the compiler understands. It's the bridge between the two
worlds: a real `if` that runs, and a narrower type that holds inside it.

```ts
function printId(id: string | number) {
  if (typeof id === "string") id.toUpperCase();   // string here
  else id.toFixed(2);                             // number here
}
```

This is the honest alternative to a [cast](06-type-casts.md). `as string` only tells the
compiler; a guard **proves** it, and the proof survives into the shipped JS.

## Built-in guards

| Guard | Use for | Watch out |
|---|---|---|
| `typeof x === "string"` | primitives | `typeof null === "object"` |
| `x instanceof Dog` | class instances | fails across iframes/realms |
| `"bark" in x` | object shapes | only checks the key exists |
| `Array.isArray(x)` | arrays | `typeof []` is `"object"` |
| `x === null`, `x != null` | nullish | `!=` catches both `null` and `undefined` |
| `x.kind === "circle"` | [discriminated unions](15-discriminated-unions.md) | the tag must be a literal type |

```ts
if (typeof x === "object") x.foo;   // x could still be null — strictNullChecks catches it
if (x != null) {}                   // loose != on purpose: null and undefined in one check
```

> **Truthiness is a sloppy guard.** `if (count)` also rejects `0`, and `if (name)` rejects `""`.
> Use it to strip `null`/`undefined` only when the falsy values are genuinely impossible.

## Custom guards — `x is T`

Built-ins can't describe your own shapes, so you write a function whose **return type is a
predicate**:

```ts
type Cat = { meow(): void };

function isCat(pet: unknown): pet is Cat {
  return typeof pet === "object" && pet !== null && "meow" in pet;
}

if (isCat(pet)) pet.meow();     // narrowed by the call, not by the if
```

Without `pet is Cat` the return type is just `boolean` and nothing narrows — the predicate is the
entire point.

> **TS trusts you here.** It does not check that the body really proves the claim, so
> `function isCat(p: unknown): p is Cat { return true }` compiles. A guard is a cast with better
> manners: still your responsibility, but at least the check sits in one reviewable place.

**Guarding an array:**

```ts
function isStringArray(x: unknown): x is string[] {
  return Array.isArray(x) && x.every(i => typeof i === "string");
}
```

Also handy as a filter, where TS otherwise keeps the wider type:

```ts
const maybe: (string | null)[] = ["a", null];
const clean = maybe.filter((x): x is string => x !== null);   // string[], not (string|null)[]
```

## Assertion functions — `asserts x is T`

Throws instead of returning a boolean. Everything **after** the call is narrowed:

```ts
function assertIsString(x: unknown): asserts x is string {
  if (typeof x !== "string") throw new Error("not a string");
}

assertIsString(input);
input.toUpperCase();      // narrowed from here on — no if needed
```

> Gotcha: this only works if the function has an **explicit** type annotation. `const f = (x) =>
> {...}` assigned without one won't narrow — TS requires the declared `asserts` signature.

## Where narrowing gets lost

```ts
let value: string | number = getIt();
if (typeof value === "string") {
  setTimeout(() => value.toUpperCase());   // Error: `value` is `let` — could change by then
}
```

- **`let` captured in a callback** — the compiler can't know the value still holds. `const` is
  narrowed safely.
- **After an `await` or a function call** that could mutate the object — narrowing on a mutable
  property is discarded.
- **Destructuring before checking** — pull out `const { kind, radius } = shape` and the link
  between the two is gone; check `shape.kind` first instead.

## Rule of thumb

At the edges of the program — API responses, `JSON.parse`, `catch (err)`,
[`unknown`](05-any-vs-unknown.md) — validate with a guard, not a cast. Inside the program, rely
on a [discriminated union](15-discriminated-unions.md) so no hand-written guard is needed at all.

See also: [Function types](07-function-types.md) · [Type casts](06-type-casts.md)

→ next: [Modeling with unions](20-modeling-with-unions.md)
