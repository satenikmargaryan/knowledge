# Abstract classes and `implements`

Two ways to say "this must have that shape." An **abstract class** is a half-built parent; an
**interface** is a pure contract. They solve nearby problems, so the useful part is knowing which
one fits.

## Abstract classes

`abstract` marks a class that **cannot be instantiated** — only extended. It can mix finished
code with holes the subclass must fill.

```ts
abstract class Shape {
  constructor(public name: string) {}
  abstract area(): number;              // no body — subclass must provide it
  describe() {                          // real method — inherited as-is
    return `${this.name}: ${this.area()}`;
  }
}

new Shape("x");                          // Error: cannot create an instance of an abstract class

class Square extends Shape {
  constructor(private side: number) { super("square") }
  area() { return this.side ** 2 }       // required
}
new Square(3).describe();                // "square: 9"
```

**Why bother.** `describe()` is written once and calls `area()` before any subclass exists. The
parent defines the *flow*; subclasses fill in the *details*. Forget `area()` and the compiler
stops you — not the test suite.

> `abstract` is compile-time only. The emitted JS is a normal class, so nothing prevents
> `new` at runtime — the guard is the checker.

Abstract members can be methods, fields, getters or setters:

```ts
abstract class Base {
  abstract readonly kind: string;      // subclass must declare it
  abstract get label(): string;
}
```

## Classes with interfaces (`implements`)

`implements` says "this class satisfies this contract" and asks the compiler to verify it. No code
is inherited — nothing at all comes from the interface.

```ts
interface Serializable { toJson(): string }
interface Printable { print(): void }

class Report implements Serializable, Printable {   // several interfaces: fine
  constructor(private title: string) {}
  toJson() { return JSON.stringify({ title: this.title }) }
  print() { console.log(this.title) }
}
```

Miss a member and the error lands **on the class**, where the mistake is — the reason to write
`implements` even when the class already happens to match.

> `implements` checks only the **public** surface. It never adds fields, defaults or methods, and
> it can't make a member `private`.

**One catch:** `implements` doesn't infer parameter types the way an annotated variable does.

```ts
interface Handler { handle(n: number): void }
class H implements Handler {
  handle(n) {}        // `n` is implicitly any — annotate it
}
```

## Which one

| | `abstract class` | `interface` |
|---|---|---|
| Can hold implementation | yes | no |
| Can hold state / constructor | yes | no |
| How many per class | one (`extends`) | many (`implements`) |
| Exists at runtime | yes (a real class) | no — erased |
| Can describe non-objects | no | yes, via `type`-like shapes |
| `protected` / `private` members | yes | no, public only |

**Rule of thumb:**

- **Interface** — you only need the shape, or a class must satisfy several contracts. Default
  choice; it costs nothing at runtime.
- **Abstract class** — subclasses share real code or state, and you want to force the holes to be
  filled. Common for a template method: the parent owns the algorithm, children own the steps.

They combine happily — an abstract class can itself implement an interface:

```ts
abstract class Shape implements Serializable {
  abstract area(): number;
  toJson() { return JSON.stringify({ area: this.area() }) }   // shared, uses the hole
}
```

See also: [Classes](09-classes.md) · [Interfaces](08-interfaces.md)

→ next: [Static members](11-static-members.md)
