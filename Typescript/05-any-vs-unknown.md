# `unknown` (and `any`)

Some values are genuinely unknown: API data, parsed JSON, user input, a `catch` block. TS has
two types for that, and they behave very differently.

- **`any`** — turns type checking **off**. Anything goes.
- **`unknown`** — "this could be anything, so **prove** what it is before using it."

```ts
let a: any = fetchSomething();
a.toUpperCase();   // compiles — crashes at runtime if it's a number
a.foo.bar.baz();   // also compiles. No protection at all.

let u: unknown = fetchSomething();
u.toUpperCase();   // Error: 'u' is of type 'unknown'
```

`unknown` is the **safe** `any`: both accept any value in, but `unknown` blocks all use until
we've checked.

## Side by side

| | `any` | `unknown` |
|---|---|---|
| **Meaning** | "stop checking this" | "could be anything — check first" |
| Assign any value **to** it | yes | yes |
| Assign it **to** another type | yes, anything | no — only after narrowing |
| Properties / methods / operators | allowed, unchecked | blocked until narrowed |
| Errors surface | runtime crash | compile time, in the editor |
| Spreads to other code | yes — infects what it touches | no — stays contained |
| Use for | escape hatch, migrating old JS | any value of unknown shape |

In one line: **both let anything in, but only `any` lets anything out.**

```ts
let a: any = "hello";
let n1: number = a;    // allowed — n1 is secretly a string
let u: unknown = "hello";
let n2: number = u;    // Error: 'unknown' not assignable to 'number'
if (typeof u === "number") { let n3: number = u; }   // OK — proven
```

The assignability row matters most: `any` is **contagious** — everything derived from it is
unchecked too, so one `any` can quietly disable safety across a whole chain. `unknown` can't
leak; the compiler stops it at first use, forcing the check where the uncertainty actually is.

## Narrowing makes `unknown` usable

Prove the type with an ordinary runtime check; TS narrows inside it:

```ts
function printLength(value: unknown) {
  if (typeof value === "string") console.log(value.length);      // string here
  else if (Array.isArray(value)) console.log(value.length);      // array here
  else console.log("no length");
}
```

Ways to narrow: `typeof`, `Array.isArray()`, `instanceof`, `in`, comparing to `null`/`undefined`.

`catch` is a common real case, since JS can throw *anything*, not just `Error`:

```ts
try { risky() }
catch (err) {                      // unknown in modern TS
  if (err instanceof Error) console.log(err.message);
  else console.log("Unknown error:", err);
}
```

**Takeaway:** prefer `unknown` over `any`. `any` removes the safety we adopted TS for;
`unknown` keeps it and just asks us to check.

→ next: [type casts](06-type-casts.md)
