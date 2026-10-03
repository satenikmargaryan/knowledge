# Type casts (type assertions)

A **type assertion** tells the compiler "trust me, treat this as that type". Keyword: `as`.

```ts
const value: unknown = "hello";
const len = (value as string).length;
```

(The older `<string>value` syntax clashes with JSX — use `as`.)

## A cast changes nothing at runtime

A cast is **not** a conversion. It transforms nothing, checks nothing, emits no JS — it only
silences the compiler:

```ts
const n: unknown = 42;
const s = n as string;   // compiles happily
s.toUpperCase();         // runtime crash — never was a string
```

Real conversions do run:

```ts
String(42);    // "42"
Number("42");  // 42
```

So a cast is a **promise to the compiler**. If the promise is wrong, nothing catches it — we get
exactly the runtime error TS was meant to prevent.

## When a cast is legitimate

- **Knowledge the compiler can't have** — most often the DOM:

  ```ts
  const input = document.getElementById("email") as HTMLInputElement;
  input.value = "a@b.com";   // `value` only exists on HTMLInputElement
  ```
- Narrowing an `unknown`/union **already validated** elsewhere.
- Working around an imprecise third-party type definition.

## `as const` — the safe one

Infers the **narrowest literal type** instead of widening, and makes the value read-only:

```ts
let a = "dark";            // string
let b = "dark" as const;   // "dark"

const roles = ["admin", "user"] as const;   // readonly ["admin","user"]
type Role = typeof roles[number];           // "admin" | "user"
```

## Rules of thumb

- **Prefer narrowing over casting** — a `typeof`/`instanceof` check is verified at runtime; a
  cast is only a claim.
- **A cast is a last resort, not a fix for a red squiggle.** The compiler is usually right;
  casting hides the problem.
- **Avoid the double cast `value as unknown as T`.** TS blocks nonsense casts between unrelated
  types; routing through `unknown` defeats that guard. Rare, and comment it.
- **`as const` is different** — it adds precision rather than removing checks.

→ next: [function types](07-function-types.md)
