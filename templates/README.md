# Templates

Generic starting points. They were written for this repo, not find-and-replaced from my own setup, so they assume nothing about your tools beyond Claude Code and a folder.

| File | Use |
|---|---|
| [claude-md.md](./claude-md.md) | `CLAUDE.md` for your mesh home repo |
| [skills/start/SKILL.md](./skills/start/SKILL.md) | `/start`: orient index-first, detect siblings, wait |
| [skills/save/SKILL.md](./skills/save/SKILL.md) | `/save`: rows, KB promotion, state file, hygiene, handoffs, code |
| [agents/verifier.md](./agents/verifier.md) | A read-only agent that tries to refute claims before they leave |

## Ten-minute setup (folder backend)

1. Make a folder, e.g. `~/my-mesh`, with `memories/leader/config/` and `kb/`.
2. Write `memories/leader/config/operator.md` (who you are, how you like to work), `sensitive-topics.md` (what never gets written where), and `context.md` (current projects, roles in use, anything every session should know; a few lines is fine). Copy [docs/operating-contract.md](../docs/operating-contract.md) into `operating-contract.md` and trim it.
3. Write `kb/INDEX.md` with one line: `# KB INDEX (0 entries)`. Let `/save` grow it.
4. Install the skills: copy (or symlink) `skills/start` and `skills/save` into `~/.claude/skills/`. Tell them where the folder is, e.g. set `MESH_HOME=~/my-mesh` or edit the Backend section.
5. Put `claude-md.md`'s block in your repo as `CLAUDE.md`. Optionally copy `agents/verifier.md` into `.claude/agents/`.
6. Run `/start`, work, run `/save`. Open the folder and read what it wrote.

When you move to an MCP backend later, keep the same paths. Only the Backend section of the skills changes.
