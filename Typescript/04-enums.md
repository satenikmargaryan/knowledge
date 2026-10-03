# Enums

An **enum** names each value in a fixed set and groups those names under one identifier.

```ts
enum Status {
  Pending = "PENDING",
  Active  = "ACTIVE",
  Closed  = "CLOSED",
}
let s: Status = Status.Active;   // write the NAME
console.log(s);                  // "ACTIVE" — the value
```

Without assigned values, TS numbers them from 0:

```ts
enum Direction { Up, Down, Left, Right }   // 0, 1, 2, 3
```

## Why use an enum if strings work?

Honest answer: [string literal unions](03-literal-types.md) cover most cases. Enums solve four
specific problems.

**1. Single source of truth.** With raw strings, `"ACTIVE"` is copy-pasted everywhere; a backend
rename means hunting every occurrence. An enum defines it once.

```ts
if (user.status === "ACTIVE") {}        // scattered
if (user.status === Status.Active) {}   // defined once
```

**2. Names clearer than values.** The stored value is often a code we don't choose (DB column,
API contract, legacy number):

```ts
enum HttpStatus { Ok = 200, NotFound = 404, ServerError = 500 }
if (res.code === HttpStatus.NotFound) {}   // better than === 404
```

**3. Grouping / discoverability.** Typing `Status.` lists every valid option.

**4. Enums exist at runtime; types don't.** The biggest practical difference — an enum compiles
to a real JS object, so we can loop over it, validate against it, or build a dropdown:

```ts
Object.values(Status);          // ["PENDING","ACTIVE","CLOSED"] at runtime
type Theme = "light" | "dark";  // erased — cannot be listed or looped
```

## When a plain union is better

- **No runtime cost** — pure type info, zero JS emitted (an enum generates an object).
- **Less ceremony** — `setTheme("dark")` vs. importing `Theme` for `setTheme(Theme.Dark)`.
- **External data** — API JSON arrives as plain strings, matching a union directly; with an
  enum we end up casting or mapping.

**Rule of thumb:** union by default. Enum when the set must exist at runtime (looping,
validating, generating UI), when values are codes needing readable names, or when one shared
definition matters across many files.

> `const enum` inlines values at compile time (no runtime object, faster) but has restrictions
> and doesn't work in every build setup — know it exists; not the default.

→ next: [any vs. unknown](05-any-vs-unknown.md)
