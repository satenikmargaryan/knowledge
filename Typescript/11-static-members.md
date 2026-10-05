# Static members

`static` moves a member off the instances and onto the **class itself**. One copy, shared,
reachable as `Dog.something` — never through an object.

```ts
class Dog {
  static instanceCount: number = 0;   // one, on the class
  name: string;                       // one per dog

  constructor(name: string) {
    Dog.instanceCount++;              // touch the shared one
    this.name = name;                 // make this dog's own
  }

  static decreaseCount() { Dog.instanceCount-- }
}

const dog1 = new Dog("Tim");
console.log(Dog.instanceCount);   // 1
const dog2 = new Dog("Joe");
console.log(Dog.instanceCount);   // 2
```

```ts
dog1.instanceCount;    // Error: it's on Dog, not on a Dog instance
Dog.name;              // not the dog's name — `name` is per-instance
```

## Why the count keeps going up instead of restarting at 1

Two different lifetimes are at play:

| | Created when | How many exist |
|---|---|---|
| `name` (instance field) | every `new Dog(...)` | one per dog |
| `instanceCount` (static) | once, when the class is **defined** | one, ever |

`class Dog { ... }` is itself a statement that runs. When it runs, JS builds **one** `Dog` object
and sets `Dog.instanceCount = 0` on it. That happens **once**, before any `new`.

`new Dog("Joe")` does *not* re-run the class body. It only makes a fresh object and runs the
constructor. So `this.name` is new every time, while `Dog.instanceCount++` keeps bumping the same
number that is already sitting on `Dog`.

```
class Dog {...}  →  Dog { instanceCount: 0 }      ← made once
new Dog("Tim")   →  Dog.instanceCount = 1         ← same box
new Dog("Joe")   →  Dog.instanceCount = 2         ← same box
```

Had `instanceCount` been a normal field, each dog would get its own copy set to `0`, then `++`
would make it `1` — every dog reporting `1`. That is the "always starts from 1" case, and
`static` is exactly what avoids it.

> The counter is **per program run**, not per computer. Restart the app, reload the page, or run
> another test file in a fresh process and the class is defined again — `instanceCount` is back
> to `0`. Within one run, importing the same module twice does *not* reset it: modules are
> evaluated once and cached.

## Static methods

Same rule: on the class, not on instances. Inside one, `this` means **the class**, so there is no
instance to read — `this.name` is meaningless there.

```ts
class Dog {
  static instanceCount = 0;
  static decreaseCount() {
    this.instanceCount--;      // `this` === Dog here; Dog.instanceCount-- is the same
  }
}
Dog.decreaseCount();
```

Typical uses: counters, caches, constants, and **factories** — a named alternative to the
constructor:

```ts
class Dog {
  private constructor(public name: string) {}        // block `new` from outside
  static fromJson(s: string) { return new Dog(JSON.parse(s).name) }
}
Dog.fromJson('{"name":"Tim"}');
```

**Static block** — for setup that needs more than one expression, run once at definition time:

```ts
class Config {
  static values: string[];
  static { Config.values = process.env.LIST?.split(",") ?? [] }
}
```

## Gotcha: statics and inheritance

A subclass inherits statics, but **reading** and **writing** differ. Reading walks up to the
parent; writing creates the subclass's own copy, and the two then drift apart:

```ts
class Puppy extends Dog {}
Puppy.instanceCount;      // 2 — read through to Dog
Puppy.instanceCount++;    // now Puppy has its OWN 3; Dog stays 2
```

The constructor above hardcodes `Dog.instanceCount++`, so every `new Puppy()` counts on `Dog` —
usually what you want for a total. Use `(this.constructor as typeof Dog).instanceCount++` if each
subclass should count itself.

See also: [Classes](09-classes.md)

→ next: [`this`](12-this-keyword.md)
