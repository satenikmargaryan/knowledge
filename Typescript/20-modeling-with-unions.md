# Modeling with unions

[Discriminated unions](15-discriminated-unions.md) are a shape. This note is the *why* — the
design habit they enable: **make impossible states impossible to write down.**

## The problem: a bag of optionals

```ts
interface State {          // the usual first attempt
  loading: boolean;
  data?: User;
  error?: string;
}
```

Four booleans-worth of combinations, and most are nonsense: loading *and* an error, data *and* an
error, nothing at all. Every reader of this type must handle states that should never exist, and
`data` needs a `?.` forever even in the branch where it certainly arrived.

```ts
type State =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: User }      // data exists only here
  | { status: "error"; error: string };    // error exists only here
```

Now the illegal combinations **cannot be constructed**. The compiler rejects
`{ status: "success" }` with no data, and `state.data` is a plain `User` inside the success
branch — no optional chaining, no defensive checks.

> The rule behind it: **put a field in the member where it's valid, not in the parent.** If a
> property only makes sense in some states, it belongs to those states.

## Reusable exhaustiveness

Writing `const _x: never = s` in every switch gets old. One helper covers the codebase:

```ts
function assertNever(x: never): never {
  throw new Error(`Unhandled: ${JSON.stringify(x)}`);
}

switch (state.status) {
  case "idle": return null;
  case "loading": return <Spinner />;
  case "success": return <List items={state.data} />;
  case "error": return <Err msg={state.error} />;
  default: return assertNever(state);     // compile error if a case is missing
}
```

It gives both halves: a **compile error** when a new member is added, and a **runtime throw** if
bad data sneaks past the types at a boundary.

## Pulling one member out

`Extract` picks a single member by its tag — useful for a function that only accepts one
([utility types](18-utility-types.md)):

```ts
type Success = Extract<State, { status: "success" }>;   // { status: "success"; data: User }
function render(s: Success) { return s.data.name }
```

And the tags themselves, when you need them as a value or a key:

```ts
type Status = State["status"];               // "idle" | "loading" | "success" | "error"
type Labels = Record<Status, string>;        // every status must get a label
```

## Where it fits

| Good fit | Poor fit |
|---|---|
| Async / request state | A shape with no real variants |
| Reducer or event actions | Members that differ by one optional field |
| Parser or validation results | Open sets that grow at runtime |
| Anything today modelled as "a flag plus some optionals" | Deeply nested variants — flatten first |

**Rule of thumb:** when you catch yourself writing a comment like *"`error` is only set when
`ok` is false"*, that comment is a discriminated union asking to be written.

See also: [Discriminated unions](15-discriminated-unions.md) · [Type guards](19-type-guards.md)

→ next: [Decorators](21-decorators.md)
