# Discriminated unions

The pattern that makes a [union](14-unions-and-intersections.md) pleasant to work with: when
every member carries a different literal tag, the compiler can tell them apart.

Give every member a **literal** field with a different value, and TS narrows on it.

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number };

function area(s: Shape) {
  switch (s.kind) {
    case "circle": return Math.PI * s.radius ** 2;   // s is the circle member here
    case "square": return s.side ** 2;
  }
}
```

`kind` is the **discriminant** — a [literal type](03-literal-types.md), not `string`. Checking it
tells the compiler exactly which member you hold, so `s.radius` becomes legal in that branch only.

## Why a string can sit where a type goes

```ts
interface Success {
  type: "success";     // looks like a value — it's a type
  message: string;
}
```

Two things look odd here, and neither is special syntax.

**1. `"success"` is a [literal type](03-literal-types.md).** Every value is also a type: the type
whose only member is that one value. `string` allows millions of strings; `"success"` allows
exactly one. It sits in a type position because that's what it is.

```ts
let a: string = "ok";        // any string
let b: "success" = "ok";     // Error: only "success" is allowed
const s: Success = { type: "success", message: "" };   // the only legal value for `type`
```

So the field is not holding a default or an example — it is **constrained** to that one string.
That's precisely what makes it a usable tag: checking `res.type === "success"` rules out every
other member of the union.

**2. `type` as a property name is just a name.** `type` is a *contextual* keyword in TypeScript —
reserved only where a type alias could start, never as an object key. `interface`, `string`,
`any` and `number` work as property names too.

```ts
const x = { type: "a", interface: 1, string: 2 };   // all fine
```

Common tag names in the wild: `type`, `kind`, `tag`, `status`, `_tag`. The name carries no
meaning to the compiler — only the fact that each member's value is a different literal does.

> With `type: string` instead, narrowing dies: `string` doesn't tell the members apart. The same
> trap bites when a tag is read from a wider value — a plain `const t = "success"` widens to
> `string`, so use `as const` or annotate. See [Type casts](06-type-casts.md).

This is how to model a value that comes in a few distinct shapes — API results, form state,
reducer actions:

```ts
type Result<T> =
  | { ok: true; value: T }
  | { ok: false; error: string };

if (res.ok) res.value;   // only here
else res.error;          // only here
```

## Exhaustiveness with `never`

Add a case to `Shape` later and you want the compiler to find every `switch` that forgot it.
Assigning to `never` does that:

```ts
function area(s: Shape): number {
  switch (s.kind) {
    case "circle": return Math.PI * s.radius ** 2;
    case "square": return s.side ** 2;
    default:
      const _exhaustive: never = s;   // Error if a new member is unhandled
      return _exhaustive;
  }
}
```

After both cases, `s` has nothing left — type `never`. Add `{ kind: "triangle" }` and `s` is that
member in the default branch, which won't assign to `never`. A compile error instead of a silent
`undefined` at runtime.

## Narrowing, in one place

TS narrows a union from ordinary runtime checks — `typeof`, `instanceof`, `in`, `Array.isArray`,
a `null` comparison, or a discriminant like `s.kind === "circle"`. The full list, plus custom
`x is T` guards and assertion functions, is in [Type guards](19-type-guards.md).

The design habit this unlocks: [Modeling with unions](20-modeling-with-unions.md).

→ next: [Type operators](16-type-operators.md)
