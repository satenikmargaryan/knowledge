# AI Memory in Claude: Study Notes

> Summary of the official Claude documentation, written in simple words.

---

## 0. The Big Idea

A Claude model does not remember anything by itself between conversations. Every new session starts with an empty context window.

When Claude seems to "remember" something, it is because a system **outside the model** saved the information and put it back into the context window. All the memory features below are different ways of doing this.

There are two main groups:

| Group | What it remembers | Who benefits |
|---|---|---|
| **Memory in the Claude app** and **the memory tool (API)** | Facts about the people who use the app | The users |
| **Memory in Claude Code** | Facts and rules about your project and your way of working | You, the developer |

---

## 1. Context Window: Working Memory

**What it is**
The context window is all the text Claude can use while writing an answer: your messages, its earlier replies, shared files, and the answer it is writing right now.

It is not the same as training data. Training is what the model learned before release. The context window is only what is in front of it right now. That is why it is called **working memory**.

**Size and tokens**
Text is measured in **tokens** (small pieces of text, roughly a short word or part of a word). A bigger context window lets Claude work with longer documents and longer conversations.

**Context rot**
More text is not automatically better. As the number of tokens grows, accuracy and recall get worse. Claude may miss or mix up details. This is called **context rot**.

**Main lesson**
What you put in the context matters as much as how much fits. Give Claude useful, relevant information, not everything you have.

---

## 2. Memory in the Claude App

**How it works**
Claude can create memory from your chat history. It summarizes your conversations and builds a summary of key points. This summary is updated every 24 hours and is added to every new conversation as context.

**Projects**
Each Project has its own separate memory and summary. Information from one Project does not mix with other Projects or with normal chats. This is useful for keeping different clients or topics apart.

**Incognito chats**
Incognito chats are not saved to memory or chat history. Use them for one-time questions you don't want Claude to remember.

**Control**
- Memory is optional and can be turned on or off in Settings.
- You can view, edit, and delete what Claude remembers.
- Deleting a chat does **not** delete memory created from it. You delete memories separately.
- Memory is included in data exports, and memory can be imported from other AI tools.

---

## 3. Memory in Claude Code (for Development)

Every Claude Code session starts with a fresh context window. Two systems carry knowledge from one session to the next.

| | CLAUDE.md | Auto memory |
|---|---|---|
| **Who writes it** | You | Claude |
| **What it holds** | Instructions and rules | Things Claude learned from working with you |
| **Scope** | Personal, project, or company | One project (per git repository) |
| **Loaded** | Every session | Every session (the index only) |

### 3.1 CLAUDE.md: Your Instructions

**What it is**
A markdown file with instructions for Claude. You write it in plain text, and Claude reads it at the start of every session.

**Where it can live**

| Location | Purpose | Who sees it |
|---|---|---|
| `~/.claude/CLAUDE.md` | Your personal preferences for all projects | Only you |
| `./CLAUDE.md` or `./.claude/CLAUDE.md` | Project rules: commands, standards, structure | The whole team (through git) |
| `./CLAUDE.local.md` | Your private notes for one project (add to `.gitignore`) | Only you |
| Company location (set by IT) | Company-wide rules | Everyone; cannot be turned off |

All files that apply are combined, not replaced. More specific files (closer to your working folder) are read last.

**When to add something**
- Claude makes the same mistake a second time.
- A code review finds something Claude should have known.
- You type the same correction you typed last session.
- A new teammate would need the same information.

**What belongs in it**
Short facts needed in every session: build and test commands, coding rules, project structure, "always do X" rules.

**What does not belong in it**
- Things Claude can learn by reading the code.
- Long step-by-step procedures → put them in a **skill** (skills load only when needed).
- Rules for only one part of the code → put them in a **path-specific rule** (see 3.3).

**Writing good instructions**
- **Size:** keep each file under about 200 lines. Long files use more context and Claude follows them less reliably.
- **Structure:** use headings and bullet points.
- **Be specific:** "Use 2-space indentation" instead of "Format code properly." "Run `npm test` before committing" instead of "Test your changes."
- **Be consistent:** if two rules disagree, Claude may follow either one. Review files from time to time and remove old or conflicting rules.

**Imports**
A CLAUDE.md can include other files with `@path/to/file`. Imported files are loaded at the start too, so they help with organization but do not save context space.

**Hidden notes for humans**
HTML comments (`<!-- note -->`) are removed before Claude sees the file. Use them for notes to your teammates that don't need to use tokens.

### 3.2 Auto Memory: Claude's Own Notes

**What it is**
Notes Claude writes for itself while working with you, without you writing anything. It is on by default.

**Four kinds of notes**
- **user:** your role, experience, and working preferences.
- **feedback:** corrections you give and approaches you approve.
- **project:** ongoing work, deadlines, and decisions that are not visible in the code or git history.
- **reference:** where to find outside information, such as an issue tracker or dashboard.

**What it skips**
Anything Claude can find in the code (architecture, file paths, bug fixes) and anything CLAUDE.md already says. Claude does not save something every session; it saves only what would help in a future conversation.

**Where it lives**
`~/.claude/projects/<project>/memory/`
- `MEMORY.md` is the index, with one line per memory.
- Each memory is a separate topic file.
- All folders and worktrees in the same git repository share one memory folder.

**Important facts**
- It is stored **only on your computer**. It is not shared through git, with teammates, or across machines. If the team needs something, put it in CLAUDE.md.
- Memory files are **not** deleted by the automatic cleanup of old sessions. They stay until you or Claude edits or deletes them.
- Claude adds a `modified` date to memory files, so you and Claude can see how fresh a fact is.
- If you say "remember this," Claude saves it to auto memory. If you want it in CLAUDE.md, say "add this to CLAUDE.md."

### 3.3 Path-Specific Rules

For bigger projects, you can split instructions into files in the `.claude/rules/` folder, one topic per file (for example `testing.md`, `security.md`).

- Rules **without** a `paths` setting load at the start of every session.
- Rules **with** a `paths` setting (for example `src/api/**/*.ts`) load only when Claude works with matching files.

This keeps the context smaller and more focused.

Personal rules in `~/.claude/rules/` apply to all your projects.

### 3.4 How and When Memory Loads

- **CLAUDE.md** is read from disk once at the start of a session and stays in the context window, so Claude sees it with every message.
- **CLAUDE.md files in subfolders** load later, only when Claude reads files in those folders.
- **MEMORY.md**: only the first 200 lines or 25KB (whichever comes first) load at the start. Anything after that is not loaded. Topic files are not loaded at the start; Claude reads them when needed.
- **After `/compact`** (when a long chat is summarized), the main CLAUDE.md is read again from disk and added back. Instructions given **only in chat** can be lost. Put important instructions in CLAUDE.md.
- **Subagents** do not get the main conversation's auto memory, but they can have their own.

### 3.5 Memory Is Guidance, Not a Guarantee

Claude treats CLAUDE.md and auto memory as context, not as strict settings. It tries to follow them, but it is not guaranteed, especially with vague or conflicting rules.

If something **must** happen every time (for example, before every commit or after every file edit), use a **hook**. Hooks are commands that run automatically at fixed moments, no matter what Claude decides.

For blocking actions completely (certain commands or files), use **settings permissions**, not CLAUDE.md.

### 3.6 AGENTS.md

Some projects use `AGENTS.md`, a file other AI coding tools read. Claude Code can read it too:
- If there is no CLAUDE.md, Claude reads AGENTS.md.
- If there is a CLAUDE.md, Claude reads only CLAUDE.md, unless you change the setting or import AGENTS.md with `@AGENTS.md`.

### 3.7 Useful Commands

| Command | What it does |
|---|---|
| `/init` | Reads your project and creates a first CLAUDE.md (or suggests improvements) |
| `/memory` | Lists memory files, opens them for editing, turns auto memory on or off |
| `/context` | Shows which memory files are loaded right now; use it when Claude ignores a rule |

---

## 4. Memory Tool: Adding Memory to Your Own App

**What it is**
A tool in the Claude API that lets Claude save and read files that stay between conversations. It helps Claude build knowledge over time without keeping everything in the context window.

**How it works**
The tool runs on **your side**:
1. Claude asks for a file action (for example, "view /memories").
2. Your app does the action on your own storage.
3. Your app sends the result back to Claude.

`/memories` is only a name. Your app decides what it really points to: a folder for each user, rows in a database, cloud storage, and so on. You control where and how the data is stored.

When the tool is on, Claude checks its memory folder before starting a task, saves what it learns while working, and reads it back in later conversations.

**Why it is useful**
It supports **just-in-time context**: instead of loading everything at the start, Claude writes down what it learns and reads it back only when needed. This keeps the context focused and helps with long tasks.

**Commands your app must handle**
- `view`: show a folder's contents or a file's contents
- `create`: create a new file
- `str_replace`: replace text in a file
- `insert`: add text at a certain line
- `delete`: delete a file or folder
- `rename`: rename or move a file or folder

**Security (your responsibility)**
- **Path safety:** a harmful path like `/memories/../../secrets.env` can reach files outside the memory folder. Check every path in every command, and make sure it stays inside the memory folder.
- **Sensitive data:** Claude usually refuses to save sensitive information, but you can add your own check that removes it before saving.
- **Size limits:** limit how big memory files can grow.
- **Expiration:** delete memory files that have not been used for a long time.

**Guiding what gets saved**
You can tell Claude what to save in your prompt, for example: "Only save information about <topic>." You can also remind it to keep memory organized, update old files, and not create new files unless needed.

### 4.1 Memory for Long Agent Tasks

Claude is told that its context window could be reset at any moment, so any progress not saved to memory could be lost.

**Pattern for projects that take many sessions**
1. **First session:** set up memory files before real work begins: a progress log (what is done and what is next), a feature list, and a note about any startup script.
2. **Every next session:** start by reading those files. This restores the project state without exploring the code again.
3. **End of each session:** update the progress log with what was finished and what is left.

**Key rule:** work on one feature at a time, and mark it done only after testing confirms it works, not just when the code is written.

**Memory + compaction**
Compaction summarizes old parts of a long conversation to keep the context small. Memory keeps the information that must survive that summary. For long-running agents, use both.

---

## 5. Quick Review

- The model does not remember by itself; memory is saved outside and added back into the context.
- The context window is working memory. More text can cause **context rot**, so keep it relevant.
- **Claude app:** automatic memory summary, separate memory per Project, incognito chats, full user control.
- **Claude Code:** **CLAUDE.md** = your rules (you write), **auto memory** = Claude's notes (Claude writes).
- Keep CLAUDE.md short and specific; use rules for specific files and skills for long procedures.
- Memory is guidance; use **hooks** for things that must always happen.
- Auto memory stays only on your computer and lives until deleted.
- **Memory tool:** runs in your app, you control the storage, and security (especially path checks) is your job.
- For long agent work: save progress, read it at the start, update it at the end.
