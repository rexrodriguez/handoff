---
name: handoff
description: Run a task end to end with superpowers while the human is away. Stands in for the human at every superpowers approval gate (classification, clarifying questions, approach, design and spec approval, plan concerns, finishing options) using fixed rulings, keeps a decision log, and leaves a handback report. Use only when the user invokes /handoff. Never self-invoke.
disable-model-invocation: true
---

# Handoff

The user has left the room and handed you this task:

> $ARGUMENTS

This invocation is the explicit instruction that superpowers' `using-superpowers` skill requires before any gate may be skipped. It does not change how superpowers works. Every superpowers skill still runs exactly as written. What changes is who answers when a skill stops to ask the human. Until the handoff ends, that is you, following `references/rulings.md`.

## Non-negotiables

1. **Superpowers still drives.** Brainstorming, writing-plans, subagent-driven-development, test-driven-development, verification-before-completion, requesting-code-review, and finishing-a-development-branch all run as they normally would. Model selection stays with superpowers' own Model Selection section.
2. **Every gate gets a ruling, not a skip.** When a superpowers skill says "get approval", "ask your human partner", "STOP and wait", or "Which option?", do the work that would have produced the question, then answer it yourself per `references/rulings.md`, and log it. Do not silently pass through a gate.
3. **Hard stops still stop.** Irreversible or destructive operations, security-sensitive actions, a merge to a shared branch, and a plan so broken that every path is a guess end the handoff with a report. These are superpowers' own four stop reasons and this skill never overrides them.
4. **The human is back the moment they send a message.** Any new user message ends proxy mode. Acknowledge it, summarize where you are, and return to normal superpowers gates for the rest of the session.
5. **Never widen the charter.** Scope is exactly what the task line above says, as refined by the design. Discovered work goes in the report as a follow-up, not into the branch.

## Steps

### 1. Open the handoff

1. Write the charter marker at `.claude/handoff.active` in the project root with the fields in `references/charter.md`. It survives compaction and lets the SessionStart hook re-arm this skill. It is not committed.
2. Create the decision log at `docs/superpowers/handoff/YYYY-MM-DD-<slug>.md` from `references/log-template.md`. Commit it with the branch so the human reviews it alongside the code.
3. Parse options from the task line. Recognized flags, all optional:
   - `--merge` allows finishing option 1, local merge to the base branch. Default is off.
   - `--no-pr` finishes with option 3, keep the branch. Default is to push and open a PR.
   - `--budget <small|medium|large>` is passed through to superpowers as guidance for its Model Selection: small biases every role one tier down, large biases design and review one tier up, medium changes nothing.
   - `--ask-on <text>` names one decision you must not rule on. If you reach it, stop and report instead.
4. Announce in one line that handoff is active and what the flags are. Then stop narrating to the human. The log carries the record.

### 2. Run superpowers with proxy rulings

Invoke `superpowers:brainstorming` and follow it. At each gate apply the matching ruling from `references/rulings.md` and append a `Ruling:` line to the log. Continue into `superpowers:writing-plans` and `superpowers:subagent-driven-development` (or `executing-plans` when subagents are unavailable) the same way.

For design and spec approval, dispatch the `handoff-proxy-reviewer` agent as the human stand-in. It returns APPROVE or REVISE with reasons. Two REVISE rounds maximum, then rule and log.

During execution, superpowers' own "rulings, not stalls" rule and ledger already apply. Mirror each ledger ruling into the handoff log with one line so the human has a single place to read.

### 3. Finish

1. Run `superpowers:verification-before-completion`. If verification fails and cannot be fixed within the fix-loop cap, mark the task BLOCKED in the log and go to step 4 without opening a PR.
2. Run `superpowers:finishing-a-development-branch`. Answer its menu per the finishing ruling.
3. Fill the handback report section of the log from `references/report-template.md`. Commit and push it.

### 4. Close the handoff

1. Delete `.claude/handoff.active`.
2. Post the handback report as your final message, verbatim from the log, with the PR link first if one exists.

## When the human interrupts

Reply with three lines: where you are in the plan, the last ruling you made, and what you were about to do. Then wait. The charter marker stays until the task ends or the user runs `/handoff off`.

## Ending early

`/handoff off` as the task line closes the handoff without finishing: write the report with status PAUSED, list the next step, delete the marker, and stop. Uncommitted work is committed to the branch as WIP first, never discarded.
