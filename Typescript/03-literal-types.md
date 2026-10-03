# Literal types

A normal type says *what kind* of value; a **literal type** says *which exact value*.

```ts
let status: string = "active";   // any string
let mode: "dark" = "dark";       // only "dark"
mode = "light";                  // Error
```

The power comes from combining literals in a **union** (`|`) — a small set of allowed values:

```ts
type Theme = "light" | "dark" | "system";

function setTheme(theme: Theme) {}
setTheme("dark");   // OK
setTheme("blue");   // Error — not one of the three
```

The compiler now knows the complete list, so it catches typos (`"darkk"`), autocompletes the
options, and warns about an unhandled case in a `switch`.

Works with other primitives too:

```ts
type Dice = 1 | 2 | 3 | 4 | 5 | 6;   // number literals
type Flag = true;                     // boolean literal
```

> `const x = "dark"` infers the literal type `"dark"`; `let x = "dark"` widens to `string`,
> because a `let` can be reassigned.

→ next: [enums](04-enums.md)
