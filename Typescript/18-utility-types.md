# Built-in utility types

TypeScript ships the common transforms, so the examples below use one shape throughout:

```ts
interface User { id: number; name: string; email?: string }
```

## On object shapes

| Type | Does | Example |
|---|---|---|
| `Partial<T>` | every key optional | `Partial<User>` — patch objects |
| `Required<T>` | every key mandatory | `Required<User>` — `email` now required |
| `Readonly<T>` | every key `readonly` | freeze a config |
| `Pick<T, K>` | keep these keys | `Pick<User, "id" \| "name">` |
| `Omit<T, K>` | drop these keys | `Omit<User, "id">` — a create payload |
| `Record<K, V>` | object with keys `K`, values `V` | `Record<string, number>` |

```ts
function update(id: number, changes: Partial<User>) {}   // any subset of fields
update(1, { name: "Ann" });

type NewUser = Omit<User, "id">;        // id is assigned by the server
type ByRole = Record<"admin" | "user", User[]>;   // both keys required
```

> `Pick`/`Omit` differ in what breaks later. `Pick` is an allow-list — a new field on `User` is
> excluded until you add it. `Omit` is a deny-list — a new field flows in automatically. Pick the
> one whose failure mode you want.

## On unions

| Type | Does |
|---|---|
| `Exclude<T, U>` | remove members assignable to `U` |
| `Extract<T, U>` | keep only members assignable to `U` |
| `NonNullable<T>` | strip `null` and `undefined` |

```ts
type Status = "idle" | "loading" | "error";
type Settled = Exclude<Status, "loading">;      // "idle" | "error"
type Clean = NonNullable<string | null>;        // string
```

## On functions and promises

| Type | Does |
|---|---|
| `ReturnType<F>` | what `F` returns |
| `Parameters<F>` | its parameters, as a tuple |
| `Awaited<T>` | unwrap a promise (recursively) |
| `ConstructorParameters<C>` | a constructor's parameters |
| `InstanceType<C>` | what `new C()` gives |

```ts
function createUser(name: string, age: number) { return { id: 1, name, age } }

type User = ReturnType<typeof createUser>;    // { id: number; name: string; age: number }
type Args = Parameters<typeof createUser>;    // [name: string, age: number]
type Data = Awaited<ReturnType<typeof fetchUser>>;   // the resolved value
```

**`typeof` first.** These take a *type*, so a value needs `typeof` in front of it — the single
most common mistake: `ReturnType<createUser>` is an error, `ReturnType<typeof createUser>` is not.

## They're all one-liners

Nothing magic: each is a [mapped](16-type-operators.md) or
[conditional](17-conditional-types.md) type. Worth reading once — after this they stop feeling
like compiler built-ins:

```ts
type Partial<T>     = { [K in keyof T]?: T[K] };
type Pick<T, K extends keyof T> = { [P in K]: T[P] };
type Record<K extends keyof any, V> = { [P in K]: V };
type Exclude<T, U>  = T extends U ? never : T;          // distributes over the union
type Omit<T, K>     = Pick<T, Exclude<keyof T, K>>;     // built from the two above
type ReturnType<F extends (...a: any) => any> = F extends (...a: any) => infer R ? R : never;
```

If a built-in doesn't exist, write your own the same way:

```ts
type DeepPartial<T> = { [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K] };
```

## Gotchas

**`Omit` doesn't check the key exists.** `Pick<User, "emial">` errors; `Omit<User, "emial">`
silently omits nothing — typos pass. Rename a field and every `Omit` referring to the old name
goes quiet instead of failing.

**They're all shallow.** `Partial`, `Readonly` and `Required` touch the top level only; nested
objects keep their original modifiers.

**`Partial` on an update payload hides bugs.** It makes *every* field optional, including ones
the operation genuinely requires. `Pick` the fields that may change instead.

```ts
function renameLoose(u: Partial<User>) {}            // accepts {} — nothing to rename
function rename(u: Pick<User, "id" | "name">) {}     // both required, as intended
```

**`Record<string, T>` is not a finite key set.** TS assumes every lookup succeeds and hands you
`T`, not `T | undefined` — `noUncheckedIndexedAccess` fixes that. `Record<Role, T>` with a union
key behaves properly: every key required, every lookup safe.

## Deriving beats repeating

The real habit worth keeping: one source of truth, everything else computed from it.

```ts
const ROLES = ["admin", "user"] as const;
type Role = typeof ROLES[number];                 // "admin" | "user" — from the array
type RoleCount = Record<Role, number>;            // both keys, enforced
```

Add `"guest"` to `ROLES` and every derived type follows. Write them out by hand and they drift —
quietly, until a bug.

See also: [Type operators](16-type-operators.md) · [Conditional types](17-conditional-types.md)

→ next: [Type guards](19-type-guards.md)
