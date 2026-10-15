# Data walls belong in the architecture, not in a regex

**2026-07-06 / 2026-07-07**

**What happened.** Rows were pattern-scrubbed for sensitive data before embedding. The scrub held useful rows for manual review even though the embedding never left the tenant's own account. Separately, a phone session refused to save a root-cause thread from work because the rule text said work content "never enters memory", even though the thread contained no customer data at all.

**What we changed.** Stated the wall as one narrow storage rule: customer data stays in the tenant that owns it, saved there in full. Everything else is the operator's knowledge. Enforcement moved to architecture: each tenant's server can only write its own account's stores and vector index.

**Lesson.** Over-broad rules cause silent memory loss, which is a real cost, not a safe default. Route, never drop. If a wall depends on text matching, it isn't a wall.
