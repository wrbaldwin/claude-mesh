# CLAUDE.md template

Copy the block below into the root of your mesh home repo as `CLAUDE.md` (Claude Code reads it at session start; some other coding agents read `AGENTS.md`, so a symlink is handy). Replace every `<placeholder>`. Delete sections you don't use yet; the start-small setup needs only Identity, On /start, On /save, and Rules.

---

```markdown
# Claude Mesh: <your-name>'s home

You are a Claude Code session in my mesh. Start every session in this repo.

## Stores

| Tenant | Memory + KB | How to reach it |
|---|---|---|
| <tenant-a> (work)     | /memories/, /kb/ | <folder path or MCP server name> |
| <tenant-b> (personal) | /memories/, /kb/ | <folder path or MCP server name> |

Role routing: <tenant-a> roles = <work-role>, <work-ticket-prefix>-N.
Every other role -> <tenant-b>. `leader` reads both; home config lives in <tenant-b>.

## Identity

/start argument:
- persistent role (<leader>, <role-1>, <role-2>, ...) -> that role
- ticket key (<PREFIX>-N) -> role = key lowercased (writes to the project bucket)
- tenant name -> ad-hoc tier for that tenant
- nothing -> leader (Code) / roamer (chat)
Identity is the role only. No device or surface prefix.

## Skills

Installed: /start and /save only (symlinked from skills/ in this repo).
Everything else is fetched on trigger: read skills/<name>/SKILL.md and follow it.

| Trigger | Skill |
|---|---|
| "<phrase>", /<name> | skills/<name>/SKILL.md |

## On /start
Follow skills/start/SKILL.md. Config first, index-first, then wait.

## On /save
Follow skills/save/SKILL.md. Rows, KB promotion, state file, hygiene, handoffs, code.

## Data wall
Customer data stays in the tenant that owns it. Storage rule, not processing rule.
Route it in full to the owning tenant; never drop it; never put it in a KB.
Details: /memories/leader/config/sensitive-topics.md (read every session).

## Orchestration
- Main chats orchestrate; work goes to background agents. Set `model` on every dispatch
  (judgment = opus, mechanical = sonnet, trivial = haiku).
- Briefings are self-contained; reports capped at ~1-2k tokens.
- /compact, never /clear. Agent types are frozen at session start.

## Git (parallel sessions share one HEAD per clone)
1. Dispatched workers never run git; they emit patches. Writers that must branch get a worktree.
2. Explicit pathspecs only; never `add -A` while workers are live.
3. Assert the base before branching: fetch, checkout main, pull, verify HEAD == origin/main.
4. Assert the branch before pushing. Push with `git push origin HEAD:refs/heads/<branch>`.
5. Verify with `git ls-remote origin <branch>`. Nothing else counts.
6. `gh pr create --head <branch>`. Merge your own verified, non-gated PRs.
   Gated: data loss, production on <tenant-a>, permission widening, draft/hold.

## Rules
- Operating contract: /memories/leader/config/operating-contract.md (re-read at /save after 24h).
- Memory access goes through the verbs/MCP only; never query backing tables directly.
- Never bulk-load a bucket: index -> semantic -> graph -> targeted reads.
- Code work is done when committed, pushed, verified on the remote, and merged.
```
