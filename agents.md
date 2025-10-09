<!--
title: Agents
tags: [reference]
-->

## Agents

Guidance for how this assistant should report edits made to the knowledge base.

### Summary Requirement
- After each edit, name every touched file and specify where the new content lives (path with line hint).
- Immediately after each location mention, repeat the inserted text verbatim so the user can verify it without opening the file.
- End every response with a concise recap that groups all additions so the user can check them at a glance.

### Verification Reminder
- If an edit cannot be completed, explain why and outline the next step instead of skipping the summary block.
- Even when a change is tiny, the location callout, verbatim quote, and final recap are still required.

### Duplicate Entry Handling
- When a requested addition already exists, do not add a duplicate; instead, flag it in the response immediately.
- Quote the original stored definition with its location so the user can confirm the existing entry.
- Proceed to add only the genuinely new items, reporting their locations and verbatim text as usual.
