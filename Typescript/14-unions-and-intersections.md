# Unions and intersections

Two ways to combine types, and they pull in opposite directions.

| | Meaning | Members you can use |
|---|---|---|
| `A \| B` (union) | **either** one | only what A and B **share** |
| `A & B` (intersection) | **both** at once | everything from A **and** B |

```ts
type Cat = { name: string; meow(): void };
type Dog = { name: string; bark(): void };

let pet: Cat | Dog;
pet.name;     // OK — both have it
pet.meow();   // Error: might be a Dog

let hybrid: Cat & Dog;
hybrid.meow(); hybrid.bark();   // both — it must satisfy each side
```

> The counter-intuitive bit: a **union** of object types gives you **fewer** usable members, not
> more. The set of possible values grew, so the set of guaranteed members shrank.

Intersections of primitives collapse to `never` — nothing is both a `string` and a `number`:

```ts
type Impossible = string & number;   // never
```

Same for object types with a **conflicting** property — the type survives, but that property is
`never`, so nothing can ever satisfy it:

```ts
type A = { id: string } & { id: number };
const a: A = { id: 1 };   // Error: id must be string & number = never
```

## Which way values flow

Assigning **into** a union is easy; getting a specific type **out** needs proof:

```ts
let id: string | number;
id = "abc";                      // in: fine, a string is one of the options
const s: string = id;            // out: Error — it might be a number
if (typeof id === "string") { const ok: string = id }   // out, after narrowing
```

> A union accepts fewer things than it looks like it should. `{ a: 1, b: 2 }` is rejected by
> `{ a: number } | { b: number }`-style unions through **excess property checks** on object
> literals — give the literal a type annotation or assign it via a variable first.

**`&` is the `type` alias version of `extends`:**

```ts
interface Employee extends Person { salary: number }    // interface way
type Employee = Person & { salary: number };            // type way
```

Nearly the same result, one difference worth knowing: `extends` **errors** on a conflicting
property, while `&` quietly produces `never` for it — so the mistake surfaces later, at the
assignment. See [Interfaces](08-interfaces.md).

→ next: [Discriminated unions](15-discriminated-unions.md)
