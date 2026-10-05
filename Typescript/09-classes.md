# Classes

A **class** is a blueprint for objects: the fields they hold plus the methods that work on them.
Unlike an [interface](08-interfaces.md), a class is a **real value** — it survives compilation and
exists at runtime.

```ts
class User {
  name: string;              // field must be declared in TS
  constructor(name: string) {
    this.name = name;        // runs on `new User(...)`
  }
  greet() { return `Hi, ${this.name}` }
}
const u = new User("Ann");
```

> TS requires fields to be declared up front. JS lets you invent `this.whatever` anywhere; TS
> wants the shape known so it can check it.

**Parameter properties** — declare and assign in one go. `public`/`private`/`protected`/`readonly`
on a constructor parameter creates the field:

```ts
class User {
  constructor(public name: string, private id: number) {}   // same as above, no boilerplate
}
```

## Access modifiers

| Modifier | Reachable from |
|---|---|
| `public` (default) | anywhere |
| `protected` | the class and its subclasses |
| `private` | the class only |
| `readonly` | anywhere, but assign only in the constructor |

```ts
class Account {
  constructor(private balance: number) {}
  deposit(n: number) { this.balance += n }     // inside: fine
}
new Account(10).balance;                        // Error: private
```

**`private` vs. `#private`** — `private` is a compile-time rule only; it's gone in the emitted JS,
so `(acc as any).balance` still works. `#balance` is a real JS private field — unreachable at
runtime, full stop.

```ts
class Account { #balance = 0 }   // truly hidden, enforced by the engine
```

Pick `private` for normal code (better tooling, works with older targets), `#` when hiding must
hold at runtime.

## Inheritance

```ts
class Animal {
  constructor(protected name: string) {}
  speak() { return "..." }
}
class Dog extends Animal {
  constructor(name: string, private breed: string) {
    super(name);                 // must call super() before using `this`
  }
  speak() { return "Woof" }      // override
  describe() { return `${this.name} the ${this.breed}` }   // protected: visible here
}
```

A subclass instance **is** usable wherever the parent is expected:

```ts
const pets: Animal[] = [new Dog("Rex", "lab"), new Animal("Generic")];
pets.forEach(p => p.speak());    // each runs its own version — polymorphism
```

> `super.speak()` calls the parent's version — useful when overriding means *adding* to the
> parent's behaviour, not replacing it.

## Getters and setters

Look like properties, run like methods. Good for a computed or validated value:

```ts
class Circle {
  constructor(private r: number) {}
  get area() { return Math.PI * this.r ** 2 }      // read: circle.area — no ()
  set radius(v: number) {
    if (v <= 0) throw new Error("must be positive");   // guard on write
    this.r = v;
  }
}
```

## `static`

Belongs to the class, not to an instance. For factories, constants, counters:

```ts
class User {
  static count = 0;
  static fromJson(s: string) { return new User(JSON.parse(s).name) }   // factory
  constructor(public name: string) { User.count++ }
}
User.count;                 // on the class
new User("Ann").count;      // Error: not on the instance
```

More — why a static counter keeps its value: [Static members](11-static-members.md).

## Where classes earn their place

Use a class when data and the behaviour over it belong together, and when you need `instanceof`,
state, or a hierarchy. Reach for a plain object + [interface](08-interfaces.md) when the thing is
just data — that's cheaper and erased at compile time.

→ next: [Abstract classes and `implements`](10-abstract-classes.md)
