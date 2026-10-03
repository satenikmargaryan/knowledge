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

## The thread running through all of it

TypeScript adds a **checking step before the code runs**. Everything else follows from that:

- Types exist **only at compile time** — they're erased, so they protect while developing, not
  at runtime. (Exception: an [enum](04-enums.md) emits a real JS object.)
- The more precisely we describe a value, the more the compiler can catch — hence
  [literals](03-literal-types.md) over `string`, and [`unknown`](05-any-vs-unknown.md) over `any`.
- Anything that **tells** the compiler instead of **proving** it — `any`,
  [casts](06-type-casts.md) — gives that protection back.
