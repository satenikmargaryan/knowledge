# Type operators: `keyof`, `typeof`, indexed access, mapped types

Tools for building a type **out of another type** instead of writing it by hand. The payoff is
that the derived type updates itself when the source changes.

## `keyof` — the keys as a union

```ts
interface User { id: number; name: string }
type UserKey = keyof User;        // "id" | "name"
```

## `typeof` — a value's type

The type-level `typeof`, not the JS operator. Use it to stop writing a type that already exists
as a value:

```ts
const config = { host: "localhost", port: 8080 };
type Config = typeof config;      // { host: string; port: number }

function greet(n: string) { return n.length }
type Greet = typeof greet;        // (n: string) => number
```

## Indexed access — `T[K]`

Look up the type of a property, same syntax as reading one:

```ts
type Id = User["id"];             // number
type Any = User[keyof User];      // number | string
type Item = string[][number];     // string — the element type of an array
```

Combined, these three give the safe getter from [Generics](13-generics.md):

```ts
function get<T, K extends keyof T>(obj: T, key: K): T[K] { return obj[key] }
```

## Mapped types

`{ [K in keyof T]: ... }` walks the keys of `T` and builds a new object type. A `for` loop over
keys, at the type level:

```ts
type Flags = { [K in keyof User]: boolean };   // { id: boolean; name: boolean }
```

**Modifiers** — add or remove `?` and `readonly` with `+` / `-`:

```ts
type Optional<T>  = { [K in keyof T]?: T[K] };          // every key optional
type Required<T>  = { [K in keyof T]-?: T[K] };         // strip the ?
type Mutable<T>   = { -readonly [K in keyof T]: T[K] }; // strip readonly
```

**Remap the key** with `as` — rename or filter keys:

```ts
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};
// { getId: () => number; getName: () => string }
```

Mapping to `never` in the `as` clause **drops** the key — that's how `Omit` works:

```ts
type Without<T, Drop> = { [K in keyof T as K extends Drop ? never : K]: T[K] };
```

## Template literal types

String literals built from other types, with the same `${}` syntax:

```ts
type Lang = "en" | "fr";
type Key = `title_${Lang}`;        // "title_en" | "title_fr"
```

Unions multiply out — every combination is generated:

```ts
type Side = "top" | "left";
type Margin = `margin-${Side}`;    // "margin-top" | "margin-left"
```

Built-in helpers: `Uppercase`, `Lowercase`, `Capitalize`, `Uncapitalize`.

> Keep these derived. `type UserKey = "id" | "name"` works today and silently rots when `User`
> gains a field; `keyof User` cannot.

→ next: [Conditional types](17-conditional-types.md)
