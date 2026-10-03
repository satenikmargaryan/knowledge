# Compile time vs. runtime errors

**TypeScript catches errors at compile time; JavaScript only reveals them at runtime.**

- **Compile time** — before the program runs. TS is compiled (transpiled) to JS, and that step
  checks our type annotations.
- **Runtime** — while executing. Plain JS has no type-checking step, so a mistake stays hidden
  until the interpreter reaches that line.

## Why JS can only find it at runtime

JS is interpreted line by line and **dynamically typed** — a variable's type is known only once
a value is in it. Nothing inspects the code beforehand:

```js
function getLength(value) {
  return value.length;
}
getLength("hello"); // 5
getLength(42);      // runtime error: undefined
```

Nothing complains while writing this. The bug appears the first time a number reaches it — and
if tests never hit that branch, it ships.

## How TypeScript catches it earlier

Types tell the compiler what each value should be; it checks every usage before anything runs:

```ts
function getLength(value: string): number {
  return value.length;
}
getLength(42);  // Compile error: Argument of type 'number' is not
                // assignable to parameter of type 'string'
```

The error shows in the editor and the build, without executing code.

## Why it matters

- **Earlier = cheaper.** Seconds to fix while typing; far more once a user hits it.
- **No need to reach the line.** The compiler checks *all* code paths, including rare branches
  tests never exercise.
- **Editor support.** Known types → accurate autocomplete, inline errors, safe refactoring.
- **Compile-time only.** Types are **erased** when TS compiles to JS. At runtime it's ordinary
  JS — type checking protects *while developing*, it isn't a guard in the shipped program.

→ next: [literal types](03-literal-types.md)
