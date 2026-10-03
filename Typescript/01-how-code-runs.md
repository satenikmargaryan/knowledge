# How code gets executed

The code we write is **not immediately executable** — it must go through a translation process
first. There are several ways to do this; JavaScript's way is **interpreting**.

A CPU only understands **machine code** (raw binary). JavaScript is human-readable text, so the
**interpreter** inside the JS engine (e.g. V8 in Chrome/Node) translates it:

- **Line by line**, into machine code or **byte code** (a compact intermediate form the engine
  runs itself).
- **While the program runs**, not up front. Execution starts almost immediately, but a line is
  translated only when the program reaches it.

## The per-line cycle

1. **Read** the next line as plain text.
2. **Parse** it — validate syntax, work out what it means (declaration, call, loop…).
3. **Translate** it to byte/machine code.
4. **Execute** it immediately → program state (variables, memory) updates.
5. **Next line**, repeat until the end or an error.

Because of step 4, each line's effect lands before the next is read. That's why order matters,
and why an error on line 10 appears only *after* lines 1–9 have run.

> Modern engines aren't purely interpreted: V8 interprets to byte code first, then a **JIT
> compiler** re-compiles hot (frequently run) code into optimized machine code.

→ next: [compile time vs. runtime](02-compile-time-vs-runtime.md)
