# `this`

`this` is not "the object the method was written in." It is decided **when the function is
called**, by *how* it is called. That one sentence explains every surprise below.

```ts
class Dog {
  constructor(public name: string) {}
  speak() { console.log(this.name) }
}
const dog = new Dog("Tim");

dog.speak();                 // "Tim" — called as dog.speak() → this = dog
const fn = dog.speak;
fn();                        // Error: cannot read 'name' of undefined
```

Nothing changed about the function. `fn()` has no object before the dot, so there is no `this`.
Class bodies are always strict mode, so `this` is `undefined` rather than the global object.

**Where this bites:** passing a method somewhere else.

```ts
setTimeout(dog.speak, 100);          // broken — the method travels without the dog
button.addEventListener("click", dog.speak);   // same
```

## The four call forms

| Call | `this` is |
|---|---|
| `obj.method()` | `obj` — whatever is left of the dot |
| `fn()` | `undefined` (strict) |
| `new Fn()` | the brand-new object |
| `fn.call(x)` / `.apply(x)` / `.bind(x)` | `x`, given explicitly |

## Fixes

```ts
setTimeout(() => dog.speak(), 100);        // 1. wrap — the call keeps the dot
setTimeout(dog.speak.bind(dog), 100);      // 2. bind — pin `this` once
```

```ts
class Dog {
  constructor(public name: string) {}
  speak = () => console.log(this.name);    // 3. arrow field — bound at construction
}
setTimeout(new Dog("Tim").speak, 100);     // works
```

An **arrow function has no `this` of its own** — it takes the one from the scope where it was
written, and `bind`/`call` cannot change that. As a class field it is created per instance inside
the constructor, so it captures that instance forever.

> Cost of the arrow field: one function per instance, and it sits on the object instead of the
> prototype — so it can't be overridden by a subclass or spied on via the prototype. Use it for
> callbacks that get handed around; keep normal methods otherwise.

**In a `static` method, `this` is the class itself** — see
[Static members](11-static-members.md):

```ts
class Dog {
  static count = 0;
  static reset() { this.count = 0 }    // this === Dog
}
```

## What TypeScript adds

**`noImplicitThis`** (part of `strict`) — errors when `this` has no known type, which catches the
stray-function case at compile time.

**A `this` parameter** — a fake first parameter that only types `this` and disappears at runtime:

```ts
function handle(this: HTMLButtonElement, e: Event) {
  this.disabled = true;          // typed, and callers are checked
}
btn.addEventListener("click", handle);   // OK
handle(e);                                // Error: `this` of type void
```

**The polymorphic `this` type** — as a return type, `this` means "the current class," so chaining
survives inheritance:

```ts
class Query {
  where(): this { return this }      // not `Query` — the subclass type
}
class UserQuery extends Query {
  byName(): this { return this }
}
new UserQuery().where().byName();    // OK; with `: Query` this would fail
```

## Rule of thumb

Call methods with the dot (`dog.speak()`). The moment a method is passed as a value, it has lost
its `this` — wrap it in an arrow, `bind` it, or make it an arrow field.

See also: [Classes](09-classes.md) · [Static members](11-static-members.md)

→ next: [Generics](13-generics.md)
