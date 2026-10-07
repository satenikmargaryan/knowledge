# Modules

A **module** is a file with its own scope. Names inside it are private until exported — nothing
leaks to the global scope.

> **The rule that bites first:** a file with **no** top-level `import` or `export` is not a
> module — it's a *script*, and everything in it is global. Two such files declaring `const x`
> collide. Add an `export {}` to make a file a module when it has nothing else to export.

```ts
// math.ts
export const PI = 3.14;                   // named export
export function add(a: number, b: number) { return a + b }
export default class Calculator {}        // default export (one per file)
```

```ts
// app.ts
import Calculator, { PI, add } from "./math";
import { add as plus } from "./math";     // rename on import
import * as math from "./math";           // everything as a namespace object
import "./setup";                         // side effects only, imports nothing
```

## Named vs. default

| | Named | Default |
|---|---|---|
| Per file | many | one |
| Import name | must match (or `as`) | anything the importer wants |
| Rename safety | the compiler finds every use | a typo just creates a new name |
| Auto-import / refactor | reliable | weaker |

**Rule of thumb: prefer named exports.** The import name being fixed is a feature — it keeps one
thing called one name across the codebase. Defaults are fine where a file *is* one thing (a React
component, a config object) and the ecosystem expects it.

## Type-only imports

Types vanish at compile time, so an import used only for a type should say so:

```ts
import type { User } from "./types";       // erased entirely
import { type User, getUser } from "./api";   // mixed: type dropped, value kept
export type { User };
```

**Why it matters:** without `type`, the import statement may survive into the output and pull in
a module at runtime for nothing — slower startup, or a crash if the file had side effects. It
also prevents accidental circular imports between modules that only need each other's types.

> `verbatimModuleSyntax` (TS 5.0+) makes this explicit: imports *not* marked `type` are emitted
> as written, those marked `type` are dropped. Turn it on and the rule becomes mechanical. Tools
> that compile file-by-file — esbuild, swc, Babel — need `isolatedModules` for the same reason:
> they can't see whether a name is a type.

## Re-exports and barrels

```ts
export { add, PI } from "./math";          // forward without importing
export * from "./math";                    // everything
export * as math from "./math";            // under a namespace
export { default as Calculator } from "./math";
```

A **barrel** is an `index.ts` that re-exports a folder. Convenient, with a real cost: importing
one symbol loads the whole barrel, which slows builds, weakens tree-shaking and invites circular
imports. Fine for a small public API; avoid for large internal folders.

## Dynamic import

`import()` returns a promise — loaded on demand, which is how code splitting works:

```ts
const { heavy } = await import("./heavy");          // runtime, typed
if (dev) await import("./devtools");                // conditional
```

Unlike the static form, it's an expression: it can live inside an `if`, and the module is only
fetched when the line runs.

→ next: [Module resolution](24-module-resolution.md)
