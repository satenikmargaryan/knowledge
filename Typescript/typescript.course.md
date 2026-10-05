# TypeScript Course Notes

**Source:** https://www.youtube.com/watch?v=iJkaAJUzeWQ&t=279s

Notes split by topic — read in order, or jump to what you need.

| # | Topic | In one line |
|---|---|---|
| 1 | [How code gets executed](01-how-code-runs.md) | The interpreter translates JS line by line into machine/byte code, while the program runs. |
| 2 | [Compile time vs. runtime](02-compile-time-vs-runtime.md) | TS catches errors before running; JS only when the line is reached. Types are erased after compiling. |
| 3 | [Literal types](03-literal-types.md) | Pin the exact allowed value; unions (`"light" \| "dark"`) define a complete, checkable set. |
| 4 | [Enums](04-enums.md) | Named values in a fixed set. Why bother over strings: one source of truth, readable names, and they exist at runtime. |
| 5 | [`unknown` and `any`](05-any-vs-unknown.md) | Both let anything in; only `any` lets anything out. Narrow before using. |
| 6 | [Type casts](06-type-casts.md) | `as` is a promise, not a conversion — it changes nothing at runtime. |
| 7 | [Function types](07-function-types.md) | `(l: string) => string` — what goes in, what comes out. Plus generics, overloads, guards. |
| 8 | [Interfaces](08-interfaces.md) | The shape of an object; `extends`, `implements`, merging, and `interface` vs. `type`. |
| 9 | [Classes](09-classes.md) | A blueprint that exists at runtime: fields, access modifiers, inheritance, `static`. |
| 10 | [Abstract classes and `implements`](10-abstract-classes.md) | A half-built parent vs. a pure contract — and when to pick which. |
| 11 | [Static members](11-static-members.md) | On the class, not the instance. Why a static counter keeps counting instead of resetting. |
| 12 | [`this`](12-this-keyword.md) | Decided by how a function is called, not where it's written — and how methods lose it. |
| 13 | [Generics](13-generics.md) | A type parameter the caller fills in. Keeps the link between what goes in and what comes out. |
| 14 | [Unions and intersections](14-unions-and-intersections.md) | `\|` leaves only the shared members, `&` combines them. Which way values flow. |
| 15 | [Discriminated unions](15-discriminated-unions.md) | A literal tag per member, so TS knows which one you hold. Why `type: "success"` is a type, and exhaustive `switch`. |
| 16 | [Type operators](16-type-operators.md) | `keyof`, `typeof`, `T[K]`, mapped and template literal types — build a type from another type. |
| 17 | [Conditional types](17-conditional-types.md) | `T extends U ? X : Y`, `infer`, and why conditionals distribute over unions. |
| 18 | [Utility types](18-utility-types.md) | `Partial`, `Pick`, `Omit`, `Record`, `ReturnType` — how each is written, and where they bite. |
| 19 | [Type guards](19-type-guards.md) | A runtime check the compiler understands. Built-ins, `x is T`, `asserts`, and where narrowing is lost. |
| 20 | [Modeling with unions](20-modeling-with-unions.md) | The design habit: make impossible states impossible. `assertNever`, `Extract`, where it fits. |

## The thread running through all of it

TypeScript adds a **checking step before the code runs**. Everything else follows from that:

- Types exist **only at compile time** — they're erased, so they protect while developing, not
  at runtime. (Exceptions: an [enum](04-enums.md) and a [class](09-classes.md) emit real JS.)
- The more precisely we describe a value, the more the compiler can catch — hence
  [literals](03-literal-types.md) over `string`, and [`unknown`](05-any-vs-unknown.md) over `any`.
- Anything that **tells** the compiler instead of **proving** it — `any`,
  [casts](06-type-casts.md) — gives that protection back.
