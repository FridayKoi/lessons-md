# 📓 LESSONS.md

English · [简体中文](README.zh-CN.md)

**The mistake notebook for AI coding agents. Don't give your agent a bigger memory. Give it a smaller one — one that only remembers the mistakes.**

AI coding agents (Claude Code, ZCode, Codex, Cursor, ...) are amnesiacs: every new session, they repeat the same mistakes in your project — spawning a second instance of a running app and deadlocking it, editing a config file that must never be touched, rebuilding what already exists. You correct them, they apologize, and next session they do it again.

**Mistake Notebook** is a per-project "error book" (错题本): a *tiny* markdown file that records only recurring failure patterns as precise rules, gets loaded into every session, and **escalates rules that keep getting violated**.

- 🎯 **High signal, zero noise** — not a memory dump. One entry = one mistake class = a few dozen tokens.
- 🔁 **Recurrence tracking** — if the same mistake happens again, the entry's counter goes up. It doesn't just remember; it knows *how often you've been burned*.
- 🔴 **Automatic escalation** — a mistake that recurs 3+ times stops being a suggestion and becomes a hard ban.
- 🧰 **Tool-agnostic** — the notebook is plain markdown referenced from `AGENTS.md` / `CLAUDE.md`, so it works with any agent that reads them. Skills are provided for Claude Code / ZCode; everything else works by pasting a prompt.

## Why not just use CLAUDE.md / a memory plugin?

| | CLAUDE.md (manual) | Memory plugins (e.g. claude-mem) | **Mistake Notebook** |
|---|---|---|---|
| What's stored | Whatever you bother to write | Everything the agent did | Only failure patterns |
| Signal/noise | High, but maintenance is on you | Low — retrieval needed, token-heavy | High — small enough to read whole every session |
| Knows a mistake *repeated*? | No | No | Yes — counter + escalation |
| Setup | Manual writing | Service/DB, lifecycle hooks | One file + two skills |
| Works across tools | Partially | Usually per-tool | Any agent reading AGENTS.md/CLAUDE.md |

The insight: you don't need to remember *what happened*. You need to remember *what not to do*. Failures are the highest-value, most compressible part of session history.

## How it works

```
 you correct the agent / it visibly fails
              │
              ▼
   /retro  (mistake-retro skill)
   distills the session into candidate lessons,
   dedupes against existing entries,
   bumps the recurrence counter on matches
              │
              ▼
        LESSONS.md  ◄──── read at session start
   (one entry per mistake class,          (mistake-recall skill
    escalation at 3+ recurrences)          + one line in AGENTS.md)
```

### Entry format

```markdown
## [E-001] Never spawn a second editor process
- Trigger: any task that involves opening/editing files in the editor
- ❌ Wrong: launching a new standalone editor process
- ✅ Right: reuse the running instance (e.g. `code -r`); a second process conflicts
  with the original and deadlocks the UI
- Recurred: 3 times (09-10, 09-12, 09-17)
- Level: 🔴 Ban
- Source: 2026-09-10, session where the app froze waiting on a file lock
- Related: E-002 (optional - same root cause, different mechanism; when one is matched, read the whole family)
```

**Levels:**
- 🟡 **Advice** — new entry. "Prefer the right way."
- 🔴 **Ban** — 3+ recurrences. "Never do this. If you catch yourself about to, stop and follow ✅ instead."

## Install

See [docs/INSTALL.md](docs/INSTALL.md) for step-by-step instructions per tool (Claude Code, ZCode, Codex, Cursor). Short version:

1. Copy `skills/mistake-retro/` and `skills/mistake-recall/` into your agent's skills folder.
2. Create `LESSONS.md` in your project root (copy from [`LESSONS.template.md`](LESSONS.template.md)).
3. Add a short "Mistake Notebook" section to your project's `AGENTS.md` / `CLAUDE.md` (snippet in INSTALL.md).
4. After a session where the agent messed up: run `/retro`.

## Files

```
mistake-notebook/
├── LESSONS.template.md        ← copy into each project as LESSONS.md
├── examples/
│   └── LESSONS.example.md     ← a filled-in notebook (see it before you use it)
├── skills/
│   ├── mistake-retro/         ← /retro : distill session failures into entries
│   └── mistake-recall/        ← makes the agent consult the notebook before acting
└── docs/
    └── INSTALL.md
```

## FAQ

**Doesn't this just duplicate CLAUDE.md?**
CLAUDE.md is where you put architecture and conventions. LESSONS.md is where *the agent's own failures* accumulate, with evidence and recurrence counts. Keeping them separate means the notebook stays small and disposable — you can delete it and start fresh on a new project without losing your rules.

**Can I edit LESSONS.md by hand?**
Yes — it's deliberately plain markdown. Human edits are first-class; the retro skill merges around them.

**My agent doesn't have skills. Can I still use it?**
Yes. The notebook + the AGENTS.md snippet alone give you ~80% of the value. `/retro` is a convenience: without skills, paste the retro SKILL.md body as a prompt at the end of a session.

## Roadmap

- [ ] Claude Code hook: auto-run retro at session end
- [ ] `npx mistake-notebook init` scaffold command
- [ ] Optional per-entry grep-based "guard" checks the agent can run before risky actions

## License

MIT
