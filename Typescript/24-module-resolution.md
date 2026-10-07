# Module resolution and declaration files

How `import "./math"` turns into a file on disk, and how types arrive for code that has none.

## The two settings that matter

```jsonc
{ "compilerOptions": {
    "module": "esnext",              // what the OUTPUT uses: import/export or require
    "moduleResolution": "bundler"    // how the compiler FINDS a file
}}
```

| `moduleResolution` | Expects | Use when |
|---|---|---|
| `bundler` | extensionless imports, `package.json` `exports` | Vite, webpack, esbuild (TS 5.0+) |
| `node16` / `nodenext` | explicit `.js` extensions in ESM | running on Node directly |
| `node10` (old `node`) | legacy CommonJS lookup | old projects |

> **The `.js` extension surprise.** Under `node16`/`nodenext` you write
> `import { add } from "./math.js"` **in a `.ts` file** — pointing at the *output* name, since
> TS never rewrites import paths. It looks wrong and is correct.

## ESM vs. CommonJS

```ts
import { add } from "./math";       // ESM — static, hoisted, async-friendly
const { add } = require("./math");  // CJS — runtime call, Node's original system
```

Which one a `.ts` file becomes depends on `module` plus the nearest `package.json`
`"type": "module"`. `.mts`/`.cts` force one per file.

**`esModuleInterop: true`** — lets you write `import express from "express"` against a CJS module
that has no real default export. Leave it on; it's the default in modern setups.

For the rare CJS-only file:

```ts
import fs = require("fs");          // TS-specific CJS import
export = someFunction;              // TS-specific CJS export
```

## Path aliases

```jsonc
{ "compilerOptions": {
    "baseUrl": ".",
    "paths": { "@app/*": ["src/app/*"] }
}}
```

```ts
import { User } from "@app/models/user";    // instead of ../../../models/user
```

> **`tsc` does not rewrite these.** `paths` only teaches the *type checker* where to look; the
> emitted JS still says `@app/...`. The bundler (or `tsconfig-paths`, `tsc-alias`) must be told
> the same mapping, or it breaks at runtime.

## Declaration files

A `.d.ts` file holds **types only** — no implementation, nothing emitted. It's how JS libraries
describe themselves to TS.

```ts
// api.d.ts
export declare function fetchUser(id: number): Promise<User>;
```

Where they come from:

- **Bundled** — the package ships its own (`"types"` in its `package.json`). Nothing to do.
- **DefinitelyTyped** — `npm i -D @types/lodash`, found automatically.
- **Neither** — write your own:

```ts
// types/untyped-lib.d.ts
declare module "untyped-lib" {
  export function doThing(x: string): void;
}
declare module "*.svg" {            // also how non-code imports get typed
  const src: string;
  export default src;
}
```

**`declare global`** reaches the global scope from inside a module:

```ts
declare global {
  interface Window { myApp: { version: string } }
}
export {};                           // keeps this file a module
```

## Namespaces — don't

`namespace Foo { }` was TypeScript's own module system, from before JS had one. It survives only
in old code and `.d.ts` files. **Use modules.** The one place namespaces still make sense is
grouping types inside a declaration file.

See also: [Modules](23-modules.md) · [Compile time vs. runtime](02-compile-time-vs-runtime.md)
