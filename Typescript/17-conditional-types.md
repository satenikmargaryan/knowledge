# Conditional types and `infer`

`T extends U ? X : Y` — an `if` at the type level. `extends` here means *"is assignable to"*, not
inheritance.

```ts
type IsString<T> = T extends string ? "yes" : "no";
type A = IsString<"hi">;    // "yes"
type B = IsString<42>;      // "no"
```

Alone that's a toy. It earns its place when the **return type depends on the input type**:

```ts
type Unwrap<T> = T extends Promise<infer U> ? U : T;
type A = Unwrap<Promise<string>>;   // string
type B = Unwrap<number>;            // number
```

## `infer` — capture a type from a position

`infer U` says *"match here, and name whatever sits in this slot `U`."* It's pattern matching on
types, and only legal inside the `extends` clause of a conditional.

```ts
type ElementOf<T>  = T extends (infer U)[] ? U : never;
type ReturnOf<T>   = T extends (...args: any[]) => infer R ? R : never;
type FirstArg<T>   = T extends (a: infer A, ...rest: any[]) => any ? A : never;

type X = ElementOf<string[]>;              // string
type Y = ReturnOf<() => number>;           // number
```

That is exactly how the built-in `ReturnType` and `Parameters` are written — see
[Utility types](18-utility-types.md).

## Distribution over unions

A conditional type applied to a **naked** type parameter runs **once per union member**, then
unions the results. This surprises people more than any other rule here:

```ts
type ToArray<T> = T extends any ? T[] : never;
type R = ToArray<string | number>;    // string[] | number[]   — NOT (string | number)[]
```

That's how filtering works — members that fall to `never` disappear, since `never` in a union is
nothing:

```ts
type Exclude<T, U> = T extends U ? never : T;
type C = Exclude<"a" | "b" | "c", "b">;    // "a" | "c"
```

**Turning distribution off** — wrap both sides in `[]`:

```ts
type IsAny<T> = [T] extends [any] ? true : false;   // T is checked whole
```

> Gotcha: `Exclude<never, "a">` is `never`, not `"a"` — distributing over an empty union gives an
> empty result. The same rule, applied to zero members.

## Recursion

Conditional types can call themselves. Useful for deep transforms:

```ts
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K]
};
```

## When to reach for one

Rarely in app code; often in a library or a shared helper, where one function must serve many
input shapes. If a plain [overload](07-function-types.md) or a union covers it, prefer that —
it's far easier to read six months later.

> Nothing here exists at runtime. Conditional types only decide what the compiler believes; they
> emit no code and cannot inspect a value.

→ next: [Utility types](18-utility-types.md)
