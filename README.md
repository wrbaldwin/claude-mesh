# Claude Mesh

A pattern for giving Claude a memory and a second brain that survive the end of a chat, work on every surface (Code, desktop, web, phone, cloud), and stay out of each other's way when you run several sessions at once.

**Pattern version: 6.0** (see [VERSION](./VERSION) and [JOURNEY.md](./JOURNEY.md)). Author: Bill Baldwin. This is the public, generic write-up of the setup I run every day. Names, accounts and employer details are deliberately left out.

---

## The whole idea in two things and two commands

When I explain this to people, I say the mesh is **two things**, loaded by `/start` and written by `/save`:

1. **Memory.** Short summaries of what happened: chats, decisions, lessons, open threads, and where each session left off. It is read and written through an MCP server, backed by a key-value store (DynamoDB in my setup). Memory answers *"what happened, and what's current?"*

2. **Knowledge base (KB).** A wiki-style second brain, in the spirit of Andrej Karpathy's "LLM wiki" idea: instead of re-reading raw notes every time, the model builds and maintains a set of interlinked markdown pages that capture what is *true* and *how things work*. It is organized by Claude, for Claude, stored in S3, and only Claude reads and writes it. The KB answers *"what do we know?"*

And **two skills**:

- **`/start`** orients a session: reads config, the KB index, the role's state and inbox, then waits for instructions.
- **`/save`** banks the session: writes memory rows (one per distinct thought), promotes anything durable into the KB, updates the session's state file, and leaves handoffs for other sessions.

Those are the only two skills installed anywhere. Every other skill (review, interview, triage, and so on) lives as a `SKILL.md` in the repo or KB and is **fetched on trigger**: a trigger table in `CLAUDE.md` maps phrases to skill files, and the session reads the skill when one shows up. Install two things, and everything else stays current without re-installing.

### Two sides

I run two **tenants**: work and personal. Each side has its own MCP server and its own stores (memory + KB). One operator context reaches both: reads are federated, writes go to the side that owns the content. Customer data is never stored outside the tenant that owns it (processing it elsewhere is allowed; storing it is not). See [docs/tenancy-and-data-walls.md](./docs/tenancy-and-data-walls.md).

---

## Start small

You do not need any of the AWS pieces to get the value. The storage has moved four times for me (OneDrive, then Confluence, then GitHub, then a lakehouse, then S3 plus a key-value store), and each one had real pluses and minuses ([JOURNEY.md](./JOURNEY.md)). The habits mattered more than the backend.

**Level 0: a folder you can see.** Make a folder (local, OneDrive, or a private GitHub repo) with two subfolders:

```
my-mesh/
  memories/        # what happened: <role>/decisions, sessions, lessons, notes, state, inbox
  kb/              # what's true: INDEX.md + <section>/<slug>.md
```

Copy [templates/skills/start](./templates/skills/start/SKILL.md) and [templates/skills/save](./templates/skills/save/SKILL.md) into your Claude Code skills folder, point them at that folder, and drop [templates/claude-md.md](./templates/claude-md.md) into your working repo as `CLAUDE.md`. Use `/start` and `/save` every session. You can open the files yourself and see exactly what Claude wrote, which is the best way to build trust in it.

**Level 1: Git.** Put the folder in a private repo. You get history, diffs, and access from any machine. Cost: commits on every save, and parallel sessions fighting over one clone ([docs/git-discipline.md](./docs/git-discipline.md)).

**Level 2: an MCP server.** When you want phone and web chat to share the same brain as Code, put memory behind an MCP server (key-value store) and the KB in object storage. Same paths, same verbs. The skills do not change much because they already speak in paths.

**Level 3: recall and orchestration.** Add semantic search and a small graph over the stores ([docs/recall-ladder.md](./docs/recall-ladder.md)), and run your main chats as thin orchestrators that dispatch background agents ([docs/orchestration.md](./docs/orchestration.md)).

Everyone sets up **their own** KB. A shared team KB is a later step, and the "customer-data-free by construction" rule is what makes that possible.

---

## Repo map

| Path | What it is |
|---|---|
| [JOURNEY.md](./JOURNEY.md) | Dated evolution: what changed, why, what it cost |
| [docs/concepts.md](./docs/concepts.md) | Roles, tiers, state files, handoffs, the leader seat |
| [docs/memory-model.md](./docs/memory-model.md) | Memory paths, verbs, rows, the state-file contract, hygiene |
| [docs/knowledge-base.md](./docs/knowledge-base.md) | The LLM-wiki KB: index, entries, promotion, lifecycle |
| [docs/recall-ladder.md](./docs/recall-ladder.md) | Index, then semantic, then graph, then targeted reads |
| [docs/tenancy-and-data-walls.md](./docs/tenancy-and-data-walls.md) | Two tenants, federated reads, routed writes, the data wall |
| [docs/orchestration.md](./docs/orchestration.md) | Agent-first chats, model tiers, compact-never-clear |
| [docs/git-discipline.md](./docs/git-discipline.md) | Worker git rule, base and branch asserts, `ls-remote` proof |
| [docs/clients.md](./docs/clients.md) | Code, chat, phone, cloud, and non-Claude clients |
| [docs/operating-contract.md](./docs/operating-contract.md) | How sessions behave: terse output, autonomy, gates |
| [docs/glossary.md](./docs/glossary.md) | Terms |
| [templates/](./templates/) | `CLAUDE.md`, generic `/start` + `/save`, a verifier agent |
| [lessons/](./lessons/) | Short dated field lessons |

## Credits

- The KB is modeled on **Andrej Karpathy's "LLM wiki"** idea: a persistent, model-maintained, interlinked markdown wiki in place of re-retrieving raw sources on every question. The specifics here (index caps, promotion at save, tenant walls) are mine.
- The memory verbs (`view`, `create`, `str_replace`, `insert`, `delete`, `rename`) mirror the Anthropic API memory tool, so a file-backed or MCP-backed store looks the same to the model.
- One of my fetched-on-trigger skills, an adversarial "council" that pressure-tests an idea, is adapted from **Nate Herk's** public roast skill. It is not reproduced here.

## Status

Private, shared read-only with a few colleagues. Expect it to change. Questions and "this didn't work for me" notes are welcome.
