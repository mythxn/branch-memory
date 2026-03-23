# branch-memory

A Claude Code plugin that scopes `MEMORY.md` per git branch. Each branch gets its own isolated session context, with a shared space for cross-branch notes.

## Install

```bash
claude plugin install branch-memory@mythxn
```

## What it does

On every session start, the plugin:

1. Detects the current git branch in your working directory
2. Assembles a `MEMORY.md` from two sources:
   - **`global.md`** — notes that apply to the entire project, regardless of branch
   - **`branches/<branch>.md`** — notes specific to the current branch
3. Writes the combined result to `MEMORY.md`, which Claude Code automatically loads as context

Non-git folders are skipped cleanly — the plugin exits without touching anything.

## Memory layout

```
~/.claude/projects/<project-key>/memory/
├── global.md                  ← Cross-branch notes (edit this directly)
├── branches/
│   ├── main.md                ← Notes for the main branch
│   ├── feat_my-feature.md     ← Notes for feat/my-feature (/ → _ in filename)
│   └── ...
└── MEMORY.md                  ← Auto-generated on each SessionStart (do not edit)
```

## How to write notes

Edit the source files directly — never edit `MEMORY.md` (it gets overwritten on each session start).

**Branch-specific notes** (current branch only):
```bash
# Edit from inside your repo
$EDITOR ~/.claude/projects/$(pwd | tr '/' '-')/memory/branches/$(git rev-parse --abbrev-ref HEAD | tr '/' '_').md
```

**Global notes** (all branches):
```bash
$EDITOR ~/.claude/projects/$(pwd | tr '/' '-')/memory/global.md
```

Or ask Claude: *"Remember X for this branch"* / *"Remember X globally"* — Claude will write to the right file.

## Behavior

| Scenario | Behavior |
|----------|----------|
| Git repo, any branch | Generates branch-scoped `MEMORY.md` |
| New branch (first time) | Creates empty branch file, generates `MEMORY.md` |
| Non-git folder | Exits cleanly — `MEMORY.md` untouched |
| Different project | Uses that project's own memory directory |
| `global.md` missing | Auto-creates with placeholder text |

## Generated MEMORY.md format

```markdown
<!-- AUTO-GENERATED: do not edit directly -->
<!-- Current branch: feat/my-feature -->
<!-- Global notes  → memory/global.md -->
<!-- Branch notes  → memory/branches/feat_my-feature.md -->

# Global Notes
(Cross-branch notes go here)

---

# Branch Notes: feat/my-feature
(Branch-specific notes go here)
```

## License

MIT
