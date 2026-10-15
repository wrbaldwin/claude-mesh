# Orchestration

How a day of work actually runs once memory and the KB are in place.

## A few long-lived orchestration chats

I run a small fixed set of main chats, each long-lived, each owning a domain: a **Leader** (the operator queue, both tenants), one for the platform itself, and one or two for major domains. Everything else (ticket sessions, quick drop-ins, other clients) is subordinate to one of them.

Chats are the **orchestration** layer; roles are the **write-scope** layer. They are different axes. A chat writes to whichever role the content belongs to.

## Agent-first: the chat is a thin orchestrator

Every main chat holds decisions, dialogue with the operator, git operations, and agent summaries. **The work happens in background subagents.** If the chat is parsing a PDF, running a loader, grepping a corpus, or writing a six-file change, it is doing it wrong.

Why: the chat's context is the scarce resource. A chat that spends it on implementation is the chat that compacts mid-task and loses the thread. Delegation is also cheaper on the expensive model's quota, because worker tokens bill to the worker's model.

Code implementation is **not** an exception. The seductive drift is dispatching agents for things that look like delegation (sweeps, research) and then writing the code inline because "I already have the context." Holding the context is the reason to write a good briefing, not a reason to skip one.

Litmus per turn: *what did I do myself that a briefing could have carried?* If the answer isn't "only decisions and git", the turn was run wrong.

## Model tiers: set `model` on every dispatch

The harness default is usually "inherit", which silently runs every worker on the chat's model.

| Tier | For |
|---|---|
| Orchestrator (your top model) | Decisions, operator dialogue, git, reads the chat must literally see |
| Judgment (e.g. opus) | Research, root cause, design, drafting anything a human reads, verification |
| Mechanical (e.g. sonnet) | Parsing, sweeps, bulk edits, watching CI, mechanical patches |
| Trivial (e.g. haiku) | One-shot lookups, probes, fan-out classification |

Also check that no environment variable overrides per-dispatch model choice.

## Dispatch discipline

- **The prompt is the worker's entire briefing.** Workers inherit `CLAUDE.md` but none of the chat's history. Include: objective, output format with a **report cap of ~1–2k tokens**, sources and tools, and explicit boundaries (data wall, no sends, no ticket transitions, files off-limits, never escalate privileges to get past a deny).
- **Delegation floor.** Each agent costs roughly 20k tokens of startup. Don't dispatch what's cheaper to do directly.
- **Cap fan-out at about 3–4 concurrent workers.** More returns re-flood the context you were protecting.
- **Workers don't touch git.** See [git-discipline.md](./git-discipline.md).

## Two-way messaging

Workers can message the chat mid-run. The chat batches their questions to the operator as one numbered list, and sends answers back to the *same* worker, which resumes with full context. The chat is the operator's only interface; workers never block on the operator directly.

## Reuse, then discard

Prefer messaging a live instance over spawning a new one: it keeps its context and skips the startup cost (one drafting agent held a draft through seven revisions). But discard and respawn when an instance has bloated, typically after re-reading a big document in a loop. Symptoms: slow turns, re-asking answered questions, truncated reports. A fresh instance plus a five-line rehydration beats a fat one.

## Outward actions stay chat-gated

Workers read, draft, summarize and propose. The chat decides before anything leaves the building (sending email, transitioning tickets, touching production). High-stakes worker claims get checked by a **write-less verifier** agent whose job is to refute, not confirm ([templates/agents/verifier.md](../templates/agents/verifier.md)).

## `/compact`, never `/clear`

`/clear` orphans every live agent: instances become unreachable and in-flight work is lost. `/compact` keeps them; they resume with full transcripts afterwards.

Context-cycle ritual when a chat gets long:

```
/save
/compact
/start <role>
```

Type them one at a time; `/compact` is processed by the client and ends the turn. Don't hand yourself a pasted "resume prompt": a save that leaves the state file's `Now`, `In flight` and `Resume hint` accurate has already written it, in the one place a fresh session looks.

Two things to know about the agent roster:

- **Instances survive a compact.** Record them in the state file's `## Agents` block anyway, because a session that *ends* can't be resumed into.
- **The set of agent types is frozen at session start.** A specialist added to `.claude/agents/` mid-session never resolves. If a dispatch fails with "agent type not found", start a new session. Never absorb the work into the chat; that is the failure agent-first exists to prevent, and it looks like normal operation while it burns the chat's context.

## One master seat, many sessions

- **Single master.** Exactly one Leader session holds the leader state file and owns the queue. Others are sub-leaders: full tools, subordinate authority, mandatory report-back to the leader inbox on every `/save`.
- **Concurrent leaders declare lanes.** When you open a second session while the first waits on something slow, it writes a `## Lane` block at once (lane name, repos held, DO-NOT list for siblings). First writer wins a lane.
- **Claims.** Before working a ticket, check its status. Claim by moving it to In Progress with a comment naming your session key and a UTC timestamp. A claim older than 36 hours counts as released, because a dead session can never come back to unclaim.
- **Inbox tombstone is a claim.** If an inbox item vanished before you acted on it, a sibling took it. That's not an error.

## Bringing results to the operator

Keep about three workstreams in flight. Bring finished items back **one at a time**, each with at most one decision, and hold the rest in a short queue ("X is back, after this"). Refill a freed slot from the oldest waiting item.
