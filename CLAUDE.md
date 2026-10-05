# Knowledge notes

A personal notes repo. Satenik asks for notes on a topic — mostly software development — and
Claude writes them here. The notes are for **re-reading later to refresh knowledge**, not for
teaching from scratch and not for impressing anyone.

## How to write

- **Simple English.** Short sentences. Plain words over clever ones. No long clauses.
- **Notes, not an essay.** Headings, short paragraphs, bullets, small tables. Something you can
  scan in a minute and still get the point.
- **Short.** Say the thing and stop. If a note goes past ~120 lines, split it by topic.
- **One idea per section**, with a bold lead-in when it helps the eye find it.
- **Code examples are the explanation.** Keep them tiny — 3-10 lines, with a `// comment` on the
  line that matters. Prefer one sharp example over three similar ones.
- **Answer "why", not only "what".** The part worth refreshing is usually the reason, the gotcha,
  or the rule of thumb — not the syntax.
- Use a `> blockquote` for a gotcha or a side note.
- Tables for comparisons (`A` vs. `B`, when to use which).
- No filler: no "in this article", no "let's dive in", no closing summary that repeats the note.

## Where to put it

Claude decides the location — don't ask unless it's genuinely ambiguous.

1. Fits an existing folder → put it there.
2. Doesn't fit → create a new folder. One folder per subject, named in plain Title Case with
   spaces (`AI in Software Development`, `Typescript`, `Angular`).
3. File names: lowercase kebab-case (`literal-types.md`, `ai-memory-notes.md`).
4. A subject that grows into several notes gets numbered files (`01-...`, `02-...`) plus one
   index file linking them in reading order — see `Typescript/typescript.course.md`.
5. Add the source link (video, docs, article) near the top when there is one.
6. Cross-link related notes with relative markdown links, and end a note in a numbered series
   with `→ next: [topic](02-topic.md)`.

## Working style

- Add to an existing note instead of creating a near-duplicate.
- Changing an existing note: keep its tone and structure; don't rewrite what already reads well.
- When a new note joins a numbered series, update that series' index file too.
