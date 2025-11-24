## Context Management & Compact Strategy

### Important Note on Context Percentage
The `/context` command displays percentage based on a 200K token baseline, but our model supports 1M tokens input.
**Ignore the percentage - focus on the actual token count.**

To calculate real usage: `actual_tokens ÷ 1,000,000 × 100 = real_percentage`

Example: If `/context` shows "140k tokens (90%)", that's actually only 14% of 1M capacity.

### When to Check Context
Run `/context` to monitor context usage under these conditions:
1. After completing a feature or module
2. After fixing a bug
3. After extensive code review sessions
4. Before switching to an unrelated task

### When to Compact
Execute `/compact` when **actual token count exceeds 900,000** (90% of 1M capacity).

Do NOT rely on the displayed percentage - it is misleading for our setup.

### What to Preserve During Compact
When running `/compact`, always include instructions to preserve:
- Project architecture decisions and rationale
- Current task objectives and progress
- Key code patterns and conventions established
- Outstanding TODOs and known issues
- Recent debugging context if still relevant

Example: `/compact preserve architecture decisions, current task progress, and established coding patterns`
