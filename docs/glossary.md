# Glossary

**Ad-hoc tier.** `/start <tenant>`: a roamer-weight drop-in pinned to one tenant with a real write scope. Do the task, `/save`, hand back.

**Agent-first.** Main chats orchestrate; background agents do the work. See [orchestration.md](./orchestration.md).

**Announced routing.** Every `/save` ends with where each row was filed (`filed: 2 → finance · 1 → mesh`) so misfiles get a one-word correction.

**Bucket.** A role's memory partition, e.g. `/memories/finance/`. Ticket roles share a project bucket.

**Claim.** Moving a ticket to In Progress with a comment naming your session key and a UTC timestamp. Released after 36 hours.

**Context cycle.** `/save`, `/compact`, `/start <role>`. Never `/clear`.

**Cross-tenant log.** One append-only file that leaders on both sides read. The only cross-tenant channel.

**Embed-on-save.** `/save` sends the rows it wrote to the semantic index in one call.

**Federated reads.** Any session may read any role (and, with access, either tenant). Roles gate writes, not reads.

**Graph-on-save.** `/save` links the obvious edges of what it wrote (tickets, KB paths, sections).

**Handoff.** A file in another role's inbox. Tombstoned by the recipient after acting; the tombstone is the claim.

**In flight.** The state-file block for things outside the mesh: background jobs, scratch clones, open PRs, half-applied changes.

**KB (knowledge base).** The model-maintained wiki of what's true. See [knowledge-base.md](./knowledge-base.md).

**KB promotion.** `/save` distilling a durable fact into a KB entry and its index line.

**Lane.** A concurrent leader session's declared scope: name, repos held, DO-NOT list for siblings.

**Leader.** The hub role. The **master leader seat** is whichever leader session holds the leader state file; all others are sub-leaders.

**Librarian pass.** Periodic digestion of old memory rows into KB entries.

**MCP server.** The service exposing memory and KB verbs (and optionally search, graph, domain data) to every client.

**Memory row.** One dated file about one thing, under decisions, sessions, lessons, or notes.

**Recall ladder.** Index, semantic search, graph, targeted reads. Never bulk-load. See [recall-ladder.md](./recall-ladder.md).

**Report-back.** The handoff every sub-leader `/save` writes to the leader inbox.

**Role.** A write scope: a persistent topic or a ticket bucket.

**Roamer.** The lightest tier, for phone and blank chats. Two or three reads, then answer.

**Roster (agent).** The set of agent types a session can dispatch. Frozen at session start.

**Route, never drop.** Customer data is saved in full in the tenant that owns it, never discarded out of caution.

**State file.** `/memories/<role>/state/<key>.md`. One per live session, 2KB cap, rewritten each save.

**Sub-leader.** Any session that isn't the master leader: full tools, subordinate authority, mandatory report-back.

**Surface.** Where Claude runs: Code, desktop, web, phone, cloud container, non-Claude client.

**Tenant.** A side with its own stores and server (work, personal).

**Tombstone.** A recoverable delete; the store keeps version history.

**Verifier.** A write-less agent dispatched to refute a claim by re-deriving it independently.

**Worker git rule.** Dispatched workers never run git. They emit patches; the orchestrator applies them.
