# branch-memory

A Claude Code plugin that scopes `MEMORY.md` per git branch. Fully automatic — open Claude and work; memory captures itself.

## Install

```bash
claude plugin install branch-memory@mythxn
```

## How it works

**You just open Claude and work. The plugin handles everything else.**

On every `SessionStart`, the plugin:

1. **Detects the git branch** in your working directory
2. **Consolidates merged branches** — if any branches have been merged into the current one, their notes are automatically folded into `global.md` and the branch files are removed
3. **Assembles `MEMORY.md`** from `global.md` + `branches/<branch>.md`
4. **Injects a Memory Protocol** into `MEMORY.md` — Claude reads this and autonomously writes notes during the session without being asked

On `SessionStart` of the *next* session, step 3 picks up whatever Claude wrote — your memory grows automatically.

Non-git folders are skipped cleanly.

## Memory layout

```
~/.claude/projects/<project-key>/memory/
├── global.md                  ← Cross-branch notes (auto-written by Claude)
├── branches/
│   ├── main.md                ← Notes for main (auto-written by Claude)
│   ├── feat_my-feature.md     ← Notes for feat/my-feature (/ → _ in filename)
│   └── ...
└── MEMORY.md                  ← Auto-generated on each SessionStart (do not edit)
```

## What Claude captures automatically

The Memory Protocol (injected into every `MEMORY.md`) tells Claude to write proactively:

| Situation | Where |
|-----------|-------|
| Bug fixed, feature shipped, decision made, TODO left | `branches/<branch>.md` |
| Architecture insight, key path, project-wide pattern | `global.md` |

Claude uses its own Edit/Write tools to append notes to those files during the session.

## Merge consolidation

When you switch to a branch that has other branches merged into it, the plugin detects the merged branches, moves their memory files into `global.md`, and removes the stale branch files — automatically.

```
feat/my-feature merged into main
    ↓
SessionStart on main
    ↓
branches/feat_my-feature.md → appended to global.md under "Consolidated from `feat/my-feature`"
branches/feat_my-feature.md → deleted
```

## Behavior matrix

| Scenario | Behavior |
|----------|----------|
| Git repo, any branch | Generates branch-scoped `MEMORY.md` |
| New branch (first time) | Creates empty branch file, generates `MEMORY.md` |
| Branch merged into current | Branch notes consolidated into `global.md` |
| Non-git folder | Exits cleanly — `MEMORY.md` untouched |
| `global.md` missing | Auto-creates with placeholder text |

## Manual notes

You can still write notes manually — Claude will also do it automatically, but you can always:

```bash
# Branch-specific
$EDITOR ~/.claude/projects/$(pwd | tr '/' '-')/memory/branches/$(git rev-parse --abbrev-ref HEAD | tr '/' '_').md

# Global
$EDITOR ~/.claude/projects/$(pwd | tr '/' '-')/memory/global.md
```

## License

MIT
