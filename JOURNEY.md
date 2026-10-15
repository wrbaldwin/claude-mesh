# Journey

How the mesh got to its current shape, in the order it happened. Dates are when the change landed in my own setup. Each storage move fixed the previous one's biggest pain and introduced a new one; none of them was wasted.

## Storage: five eras, four moves

| Era | Where memory + knowledge lived | Pluses | Minuses |
|---|---|---|---|
| **OneDrive** (early 2026) | Flat `.md` files in a synced folder; I pasted context between chats by hand | Zero setup. I could read every file. Worked across work and personal machines | I was the router ("paste courier"). No structure, no concurrency story, sync conflicts |
| **Confluence** (May 2026) | Wiki pages for handoffs, session state, config, and design records | Humans could read it. Real page history. Phone-friendly | Slow and wordy for a model. Large page writes tripped a web firewall. Wiki markup is a bad storage format for machine state |
| **GitHub** (mid May 2026) | Repo for skills, an inbox for cross-tenant handoffs, staging for KB pages | Diffs, history, PRs, works offline. Great for anything code-adjacent | Every save is a commit. Parallel sessions sharing a clone collide on HEAD. Phone and web chat can't reach it |
| **Lakehouse** (May 16 – June 16, 2026) | Memory rows, state and handoffs as Iceberg tables queried through Athena, behind an MCP server | Typed, queryable storage. One API for every surface, phone and web included. Domain data fits naturally | Tiny point reads and writes paid analytics-engine latency (hundreds of ms to seconds) plus per-table permission grants. Slow `/start` and `/save` |
| **S3 + key-value store** (June 2026 – present) | Memory in DynamoDB behind an MCP server; KB as markdown objects in S3 | Every surface reaches the same brain. Point reads in milliseconds. Versioned. Two tenants, two isolated stacks | Infrastructure to own. You can't casually open the files, so trust has to come from habits (indexes, announced writes) |

The honest summary for a newcomer: **start where you can see the files**, get the `/start` + `/save` habit, and move to S3 once you're comfortable not looking.

## Timeline

| Date | Version | Change | Why |
|---|---|---|---|
| ~2025 | 1.x | Flat markdown memory, manual routing | It was the simplest thing that worked |
| early 2026 | 2.x | Wiki handoffs, several commands | Copy-paste routing didn't scale past a couple of chats |
| 2026-05-11 | 3.0 | **Two commands only: `/start` and `/save`.** Identity = role, not device | Fewer verbs got used more consistently |
| 2026-05-14 | 3.1–3.4 | Multi-surface leader; sensitivity framework; Git replaces OneDrive for cross-tenant handoffs | Same account runs on many devices; devices are not routing destinations |
| 2026-05-16 | 3.5–3.7 | Memory moves into a lakehouse behind an MCP server; role taxonomy (persistent roles + ticket buckets) | Wanted typed, queryable storage and one API for every surface |
| 2026-06-02/03 | 4.0–4.4 | Session state, config, and handoffs leave the wiki. `/start` + `/save` touch zero wiki pages. Claude may merge its own verified PRs | The wiki was the slowest, flakiest part of every save |
| 2026-06-16 | 5.0 | **Memory moves off the lakehouse onto a key-value store** (DynamoDB). Domain data stays in the lakehouse | Memory is tiny point reads and writes. KV was 84–293× faster for `/start` and `/save` |
| 2026-06-19 | 5.1 | Role-to-leader handoff lane | The hub session was blind to what spoke sessions finished |
| 2026-07-02 | KB v1 | **Knowledge base on S3**, seeded by distilling ~280 wiki pages down to ~50 curated entries. Wiki frozen read-only | Memory was doing two jobs. Split "what happened" from "what's true" |
| 2026-07-03 | 6.0 | Work and personal tenants unified under one operator context: two stores, role-routed writes | One brain, two walls was simpler than two brains |
| 2026-07-05 | 6.1 | Roles become write scopes, reads are federated. Skill registry: only `/start` + `/save` installed, the rest fetched. Semantic search over KB and memory | Sessions refused useful reads because "another role owns it" |
| 2026-07-06/07 | 6.1.x | Data wall restated as a **storage** rule enforced by architecture, not a text scrub | A regex scrub was guarding an in-account operation and dropping useful content |
| 2026-07-09 | — | Graph rung added to recall (links between tickets, KB entries, tables, people) | "How are X and Y related" isn't a similarity question |
| 2026-07-17 | — | N concurrent leader sessions with declared lanes and ticket claims | I open a second session while the first waits on something slow |
| 2026-08-04 | — | State-file contract: 2KB cap, rewrite don't accrete | One state file had grown into a 90-version journal |
| 2026-08-08 | — | `## In flight` block in state files; `/compact` then `/start` as the context-cycle ritual | Background jobs and scratch clones were getting lost across resets |
| 2026-08-11 | — | **Agent-first**: every main chat is a thin orchestrator; work goes to background agents with explicit model tiers | Preserve the orchestrator's context and the expensive model's quota |
| 2026-08-26 | — | Four long-lived orchestration chats; single master leader seat; **dispatched workers never touch git** | Six parallel workers in one clone corrupted each other's commits in four minutes |
| 2026-08-27 | — | `git ls-remote` is the only acceptable push proof | A push exited 0 having landed the wrong commit |
| 2026-08-28 | — | Agent roster confirmed start-frozen; instances survive `/compact` | A specialist added mid-session never resolved, and the chat quietly did the work itself |
| 2026-09-08 | — | Non-Claude clients (ChatGPT, Codex) reach full parity through an OAuth MCP gateway | One brain, several mouths |
| 2026-09-22/30 | — | Model tiering by task shape; verify numbers before they leave; a read-only adversarial verifier | Three agent-produced numbers in two days were wrong |

## What stayed constant

- Two commands. Every other change was underneath them.
- Paths shaped like `/memories/<role>/...` and `/kb/<section>/...`. Moving backends never changed the paths, which is why the skills survived four migrations.
- Index-first reading. The bigger the stores got, the more this mattered.
