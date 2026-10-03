# Interfaces

An **interface** describes the **shape** of an object — which properties it has and their types.
A contract the compiler checks, erased at compile time.

```ts
interface User {
  id: number;
  name: string;
  email?: string;              // optional
  readonly createdAt: Date;    // read, not reassign
}
const u: User = { id: 1, name: "Ann", createdAt: new Date() };
u.createdAt = new Date();      // Error: readonly
```

**Methods and function properties:**

```ts
interface Formatter {
  prefix: string;
  format(l: string): string;   // method shorthand
  onDone: () => void;          // property holding a function
}
```

**`extends`** — compose from smaller interfaces (multiple parents allowed):

```ts
interface Person { name: string }
interface Employee extends Person { salary: number }   // needs both
```

**`implements`** — a class promises to satisfy an interface; the compiler verifies:

```ts
class JsonFormatter implements Formatter {
  prefix = ">";
  format(l: string) { return this.prefix + l }
  onDone() {}
}
```

**Index signature** — keys unknown ahead of time:

```ts
interface Dictionary { [key: string]: number }
const scores: Dictionary = { math: 90, art: 85 };
```

**Declaration merging** — re-declaring an interface *adds* to it instead of erroring. Unique to
interfaces; how third-party types get extended:

```ts
interface Window { myApp: string }   // adds to the built-in Window
```

## `interface` vs. `type`

Heavy overlap for object shapes; either is fine.

| | `interface` | `type` |
|---|---|---|
| Object shapes | yes | yes |
| Unions, tuples, primitives | no | yes — `type Id = string \| number` |
| Extending | `extends` | intersection `&` |
| Re-declaring same name | merges | error |
| Mapped / conditional types | no | yes |

**Rule of thumb:** `interface` for object and class shapes — especially public API surfaces
others extend; `type` for everything else (unions, tuples, function types, aliases). Consistency
matters more than the choice.

> An interface holds no values, so nothing of it remains in the emitted JS — unlike an
> [enum](04-enums.md).
