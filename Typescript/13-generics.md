# Generics

A generic is a **type parameter**: a placeholder the caller fills in, the same way a normal
parameter holds a value the caller passes.

The problem they solve — without them you choose between losing the type or writing the function
twice:

```ts
function first(items: any[]): any { return items[0] }    // works, but the result is `any`
const n = first([1, 2]);   // n: any — no checking from here on
```

```ts
function first<T>(items: T[]): T | undefined { return items[0] }
const n = first([1, 2]);       // n: number — inferred from the argument
const s = first(["a", "b"]);   // s: string
```

`T` is not a type; it's a **slot**. Each call fills it in. Read the signature as *"for whatever
type the array holds, give back that same type."* The link between input and output is the whole
point — `any` throws that link away.

## Inference, and overriding it

TS fills `T` from the arguments. Pass it explicitly only when there's nothing to infer from:

```ts
first([1, 2]);              // T = number, inferred
const empty = first<string>([]);    // nothing to infer — say it
```

**Several parameters**, named for what they hold (`T` for a lone one, `K`/`V`, `TItem`, `TKey`
when it helps readability):

```ts
function pair<K, V>(key: K, value: V): [K, V] { return [key, value] }
pair("age", 30);     // [string, number]
```

## Constraints — `extends`

A bare `T` could be anything, so you can't touch it. `extends` narrows what may be passed:

```ts
function len<T>(x: T) { return x.length }                    // Error: T has no .length
function len<T extends { length: number }>(x: T) { return x.length }   // OK
len("abc");    // 3
len([1, 2]);   // 2
len(42);       // Error: number doesn't have length
```

Note `len` still returns the **exact** type passed, unlike `(x: { length: number })` which would
widen it.

**`keyof` constraint** — the classic safe property getter:

```ts
function get<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
const user = { name: "Ann", age: 30 };
get(user, "name");   // string
get(user, "email");  // Error: not a key of user
```

**Defaults** — a fallback when nothing is inferred:

```ts
interface Box<T = string> { value: T }
const b: Box = { value: "hi" };      // T defaults to string
```

## On interfaces, types and classes

```ts
interface ApiResponse<T> {
  data: T;
  error: string | null;
}
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

class Stack<T> {
  private items: T[] = [];
  push(item: T) { this.items.push(item) }
  pop(): T | undefined { return this.items.pop() }
}
const s = new Stack<number>();    // T fixed at construction
s.push("a");                      // Error
```

Here `T` is fixed once per *instance*, not per call — a `Stack<number>` stays numbers for life.

## When not to use one

**Rule of thumb: if a type parameter appears only once in the signature, it isn't doing
anything.** Generics express a *relationship* — between two parameters, or between a parameter
and the return type.

```ts
function log<T>(x: T): void {}          // pointless — write (x: unknown)
function wrap<T>(x: T): T[] { return [x] }   // real: input type shapes the output
```

> Generics are erased at compile time, like every other type. `T` cannot be read at runtime —
> no `new T()`, no `typeof T`. If you need the actual class, pass it as a value:
> `function make<T>(ctor: new () => T): T { return new ctor() }`

See also: [Function types](07-function-types.md) · [Interfaces](08-interfaces.md)
