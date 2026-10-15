# State files turn into journals unless you cap them

**2026-08-04**

**What happened.** An audit found one session's state file at version 90 and 8KB: every save had appended a new section, and "superseded" blocks were kept "for safety." Fresh sessions spent their context reading a diary. In the same audit, two mechanical steps (an index append and a stale-state sweep) had silently stopped running for weeks, because neither produced any output.

**What we changed.** A 2KB hard cap; each save rewrites the whole body; anything longer than two lines becomes a row with a pointer. The stale-state sweep became mechanical (36 hours unwritten = dead, no judgment). Every save now prints a one-line housekeeping receipt.

**Lesson.** Version history already keeps the old text, so the state file doesn't have to. And any step with no visible output will eventually be skipped.
