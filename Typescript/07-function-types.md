# Function types

A function is a value in JS — storable, passable, returnable — so it needs a type too. That type
describes **what goes in** and **what comes out**: `(parameters) => returnType`.

```ts
let format: (l: string) => string;
//           ^ takes one string   ^ returns a string

format = (text) => text.trim();    // OK
format = (text) => text.length;    // Error: number not assignable to string
format = (a, b) => a + b;          // Error: expects 1 argument, got 2
```

Read `(l: string) => string` as *"a function taking one string, giving back a string."*

**Parameter names in a type are documentation only** — only position, type and count matter:

```ts
let greet: (name: string) => string;
greet = (n) => `Hello, ${n}`;   // fine — `n` here, `name` in the type
```

We don't annotate `n` in the implementation: TS already knows it from the declared type. That's
**contextual typing**.

**The `=>` in a type is not an arrow function** — it's syntax for "returns":

```ts
let fn: (x: number) => number   // TYPE: the shape of a function
      = (x) => x * 2;           // VALUE: the actual arrow function
```

## Shapes

```ts
() => void                              // no params, no useful return
(a: number, b: number) => number        // two params
(items: string[]) => number             // takes an array
(l?: string) => string                  // optional param
(...nums: number[]) => number           // rest param
(l: string) => (n: number) => string    // returns a function
```

`void` means the return value isn't meant to be used — typical for side-effect callbacks.

## Where they're useful

**Typing a callback parameter** — by far the most common use:

```ts
function transform(items: string[], fn: (l: string) => string): string[] {
  return items.map(fn);
}
transform(["a","b"], (s) => s.toUpperCase());  // OK
transform(["a","b"], (s) => s.length);         // Error — must return string
```

**Naming a reusable signature** with an alias:

```ts
type Formatter = (l: string) => string;
const upper: Formatter = (l) => l.toUpperCase();
```

## Annotating a declaration vs. the function's type

```ts
function format(l: string): string { return l.trim() }        // declaration — inline
const format: (l: string) => string = (l) => l.trim();        // the whole function as a type
```

Use the first for ordinary named functions; use `(l: string) => string` when the function is a
**value** being passed around — callback param, variable, object field, type alias.

> Omit the return type and TS **infers** it. Still worth writing on exported functions: it
> catches the mistake inside the function rather than at the call site.

## Advanced — small notes

**Generics** — a placeholder filled in at the call site:

```ts
function first<T>(items: T[]): T | undefined { return items[0] }
first(["a"]);  // string | undefined
first([1]);    // number | undefined
```

**Overloads** — several signatures when the return type depends on the input:

```ts
function parse(value: string): string;
function parse(value: number): number;
function parse(value: any): any { return value }   // impl, not callable itself
```

**Type guards** (`x is T`) — a return type that drives narrowing through our own helper:

```ts
function isString(x: unknown): x is string { return typeof x === "string" }
if (isString(v)) v.toUpperCase();
```

**Call signature** — a callable that also has properties:

```ts
type Counter = { (l: string): string; calls: number };
```

**`never`** — never finishes normally (always throws/loops). Unlike `void`, which returns
nothing useful:

```ts
function fail(msg: string): never { throw new Error(msg) }
```

**Extracting parts:**

```ts
type Fn = (l: string) => string;
type Args = Parameters<Fn>;   // [l: string]
type Out  = ReturnType<Fn>;   // string
```

**Compatibility** (looks wrong at first):

- **Fewer** parameters is assignable where more are expected — `() => void` works as
  `(l: string) => void`. That's why `arr.map(() => 0)` is legal: a callback may ignore args.
- The **return type must be assignable** to the expected one; the **parameter** check is
  deliberately lenient.

**`this` parameter** — a fake first parameter, erased at compile time:

```ts
function handler(this: HTMLButtonElement, e: Event) { this.disabled = true }
```

**`async`** always returns a promise:

```ts
const load: (id: string) => Promise<string> = async (id) => "data";
```

→ next: [interfaces](08-interfaces.md)
