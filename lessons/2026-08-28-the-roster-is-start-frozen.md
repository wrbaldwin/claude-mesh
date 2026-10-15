# The agent roster is frozen at session start

**2026-08-28**

**What happened.** A new specialist agent was added to `.claude/agents/` mid-session. Dispatching it failed with "agent type not found". The dangerous part wasn't the error, it was the recovery: a chat that can't dispatch quietly does the work itself, burning its own context while looking like normal operation.

**What we tested.** Live agent instances survive `/compact` and resume with full transcripts (probes returned exact nonces with zero tool calls). What doesn't change after start is the set of agent *types*.

**What we changed.** `/start` compares the session's roster with the agents directory and names any drift. "Agent type not found" means start a new session, never absorb the work. `/clear` stays banned because it orphans live instances.

**Lesson.** When a delegation path fails, the fallback that "just works" is often the failure you designed against.
