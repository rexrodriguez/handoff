# Charter marker

Path: `.claude/handoff.active` in the project root. Plain markdown. Not committed. Deleted when the handoff closes.

The SessionStart hook reads this file after a compaction or resume and re-injects the handoff contract, so keep it short and current. Update `Status` and `Current step` whenever they change.

```markdown
# handoff active
Task: <the task line, verbatim>
Flags: <--merge | --no-pr | --budget X | --ask-on "..."> or none
Started: <ISO timestamp>
Branch: <feature branch name once created>
Log: docs/superpowers/handoff/<date>-<slug>.md
Status: RUNNING | BLOCKED | PAUSED | COMPLETE
Current step: <brainstorming | writing-plans | executing task N of M | verification | finishing>
Last ruling: <one line>
```
