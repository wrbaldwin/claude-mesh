# Memory wants a key-value store, not a lakehouse

**2026-06-16**

**What happened.** For about a month, memory rows, session state and handoffs lived in Iceberg tables queried through Athena. It worked, and it was slow and fiddly: every `/start` and `/save` was a set of tiny single-item reads and writes, each paying analytics-engine latency plus per-table permission grants.

**What we changed.** Moved the memory tier to one DynamoDB table (partition = `/memories/<role>`, sort key = relative path). Kept domain data in the lakehouse. The six memory verbs didn't change, so no skill changed.

**Result.** `/start` and `/save` got 84–293× faster.

**Lesson.** Match the store to the access pattern. Coordination state is point lookups; analytics is scans. And keep the interface (paths + verbs) stable so the backend can move under it.
