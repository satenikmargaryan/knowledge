# Decorators

A decorator is a function that **wraps or replaces** a class or one of its members at definition
time. `@` is just sugar for calling it.

```ts
function logged(fn: any, ctx: ClassMethodDecoratorContext) {
  return function (this: any, ...args: any[]) {
    console.log(`calling ${String(ctx.name)}`);
    return fn.call(this, ...args);          // wrap: do extra, then the original
  };
}

class Api {
  @logged
  fetch(url: string) { return url }
}
```

Nothing is monkey-patched at call time — the class is **built differently**, once, when the
`class` statement runs.

> Two incompatible versions exist. This note is the **standard** one (TS 5.0+, no flag). Angular
> and NestJS code uses the older form — see [Legacy decorators](22-legacy-decorators.md).

## The shape

Every decorator takes `(value, context)` and may return a replacement:

```ts
(value, context) => replacement | void
```

`value` is the thing being decorated; `context` describes it:

```ts
{ kind: "method" | "class" | "field" | "getter" | "setter" | "accessor",
  name: string | symbol,
  static: boolean,
  private: boolean,
  addInitializer(fn): void }
```

Return nothing and the original is kept. There is one matching context type per kind —
`ClassMethodDecoratorContext`, `ClassFieldDecoratorContext`, and so on.

## What can be decorated

| Target | `value` is | Return to replace with |
|---|---|---|
| Class | the class | a new class |
| Method / getter / setter | the function | a new function |
| Field | `undefined` | an **initializer** `(initial) => newValue` |
| `accessor` field | `{ get, set }` | `{ get, set, init }` |

A field decorator never sees the value — fields don't exist until an instance is made, so you get
a hook on initialization instead:

```ts
function upper(_: undefined, ctx: ClassFieldDecoratorContext) {
  return (initial: string) => initial.toUpperCase();   // runs per instance
}

class User { @upper name = "ann" }
new User().name;   // "ANN"
```

Decorators work on **classes and their members only** — not on plain functions, not on object
literals.

## Decorator factories

Need an argument? Write a function that *returns* the decorator — `@x()` with parentheses:

```ts
function min(n: number) {
  return function (fn: any, ctx: ClassMethodDecoratorContext) {
    return function (this: any, v: number) {
      if (v < n) throw new Error(`${String(ctx.name)}: below ${n}`);
      return fn.call(this, v);
    };
  };
}

class Cart { @min(1) setQty(v: number) {} }
```

Most real decorators are factories, which is why so much framework code reads `@Input()` rather
than `@Input`.

## Order

```ts
@a @b method() {}
```

- **Expressions** are evaluated top to bottom: `a`, then `b`.
- **Application** is bottom to top: `b` wraps the method, then `a` wraps that.

So the one written closest to the declaration is applied first and sits innermost — same as
nested function calls, `a(b(method))`.

**`addInitializer`** runs code when the instance (or class, for `static`) is constructed. The
standard way to bind `this`:

```ts
function bound(fn: any, ctx: ClassMethodDecoratorContext) {
  ctx.addInitializer(function (this: any) { this[ctx.name] = fn.bind(this) });
}
```

See also: [Classes](09-classes.md) · [`this`](12-this-keyword.md)

→ next: [Legacy decorators](22-legacy-decorators.md)
