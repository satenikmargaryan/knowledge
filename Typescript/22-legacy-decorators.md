# Legacy decorators

Decorators existed in TypeScript years before the JS standard settled, and the two versions are
**not compatible**. Angular, NestJS and TypeORM code is written against the old one, so this is
the version most real projects still run.

Which one you get is decided by tsconfig:

```jsonc
{ "compilerOptions": {
    "experimentalDecorators": true,    // legacy. Omit it → the standard ones
    "emitDecoratorMetadata": true      // legacy only; needed by DI frameworks
}}
```

## The old shape

Three parameters, no `context` object, and the return value means different things per target:

```ts
function logged(target: any, key: string, desc: PropertyDescriptor) {
  const original = desc.value;
  desc.value = function (...args: any[]) {       // mutate the descriptor in place
    console.log(`calling ${key}`);
    return original.apply(this, args);
  };
  return desc;
}

class Api {
  @logged
  fetch(url: string) { return url }
}
```

| Target | Parameters |
|---|---|
| Class | `(constructor)` |
| Method / accessor | `(target, key, descriptor)` |
| Property | `(target, key)` — no descriptor |
| **Parameter** | `(target, key, index)` |

`target` is the **prototype** for instance members and the **constructor** for static ones — a
frequent source of confusion.

## Parameter decorators and metadata

Two things the legacy version has and the standard one does not — and the reason frameworks have
not moved:

```ts
class UserService {
  constructor(@Inject(HTTP) private http: Http) {}   // parameter decorator
}
```

With `emitDecoratorMetadata`, the compiler writes the **declared types** into the output as
metadata, which a DI container reads at runtime (via `reflect-metadata`):

```ts
Reflect.getMetadata("design:paramtypes", UserService);   // [Http]
```

This is the one case where a type survives compilation — normally types are
[erased](02-compile-time-vs-runtime.md). It only works for class references, not interfaces or
unions, which is why DI tokens exist.

## Side by side

| | Legacy | Standard |
|---|---|---|
| Flag | `experimentalDecorators` | none (TS 5.0+) |
| Signature | `(target, key, descriptor)` | `(value, context)` |
| Method change | mutate the descriptor | return a replacement |
| Parameter decorators | yes | no |
| Type metadata emit | yes | no |
| Private members | no | yes (`#x`, `private`) |
| Future | frozen | matches the JS proposal |

## Practical notes

- **Don't mix them.** One flag per project; a decorator written for one version silently breaks
  under the other, usually as "`desc` is undefined" or a decorator that quietly does nothing.
- **Framework code decides for you.** On Angular or Nest, keep `experimentalDecorators` on and
  write legacy decorators.
- **New, framework-free code** → the [standard ones](21-decorators.md). Better types, and no
  flag to carry.
- Decorators are **not** a type-system feature. They emit real runtime code, unlike almost
  everything else in these notes.

See also: [Decorators](21-decorators.md) · [Compile time vs. runtime](02-compile-time-vs-runtime.md)

→ next: [Modules](23-modules.md)
