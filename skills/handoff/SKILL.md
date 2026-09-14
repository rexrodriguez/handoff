---
name: handoff
description: Run a task end to end with superpowers while the human is away. At every superpowers approval gate (classification, clarifying questions, approach, design and spec approval, plan concerns, finishing options) a Fable proxy agent makes the ruling the human would have made. Keeps a decision log and leaves a handback report. Use only when the user invokes /handoff. Never self-invoke.
disable-model-invocation: true
---

# Handoff

The user has left the room and handed you this task:

> $ARGUMENTS

This invocation is the explicit instruction that superpowers' `using-superpowers` skill requires before any gate may be skipped. It does not change how superpowers works. Every superpowers skill still runs exactly as written. What changes is who answers when a skill stops to ask the human. Until the handoff ends, that seat is filled by the `handoff-proxy` agent. You drive superpowers. The proxy decides. `references/rulings.md` says what each gate asks and how the proxy is expected to rule.

## Non-negotiables

1. **Superpowers still drives.** Brainstorming, writing-plans, subagent-driven-development, test-driven-development, verification-before-completion, requesting-code-review, and finishing-a-development-branch all run as they normally would. Model selection for implementers and reviewers stays with superpowers' own Model Selection section.
2. **Every gate gets a ruling, not a skip.** When a superpowers skill says "get approval", "ask your human partner", "STOP and wait", or "Which option?", do the work that would have produced the question, then dispatch `handoff-proxy` with a gate packet, apply the ruling it returns, and log it. Do not decide a listed gate yourself, and do not silently pass through one.
3. **Hard stops still stop.** Irreversible or destructive operations, security-sensitive actions, a merge to a shared branch, and a plan so broken that every path is a guess end the handoff with a report. These are superpowers' own four stop reasons and neither you nor the proxy overrides them.
4. **The human is back the moment they send a message.** Any new user message ends proxy mode. Acknowledge it, summarize where you are, and return to normal superpowers gates for the rest of the session.
5. **Never widen the charter.** Scope is exactly what the task line above says, as refined by the design. Discovered work goes in the report as a follow-up, not into the branch.

## Who decides

The `handoff-proxy` agent runs on Fable by default, regardless of the session model. You may be on Opus or Sonnet to keep the driving cheap. Each dispatch passes `model` explicitly from the `--decider` flag, default `fable`, so the proxy's model never silently inherits yours.

The proxy is dispatched only at gates. Gathering evidence, dispatching implementers, running tests, writing the log, and everything else stays with you. Batch what you can: all clarifying questions in one packet, all plan concerns in one packet. A typical task needs five to eight proxy calls.

**Same model, no round trip.** If the decider resolves to the model you are already running on, or is `inherit`, answer DECIDE gates yourself, inline, following the same rule text and logging the same ruling line. The proxy would reach the same answer from the same evidence. REVIEW gates always dispatch regardless of model. The value there is a reviewer that has not seen your reasoning, not a different model, which is the same reason superpowers dispatches a separate code reviewer.

A gate packet contains:

```
Gate: <superpowers skill> / <gate name from rulings.md>
Kind: DECIDE | REVIEW
Task line: <verbatim>
Flags: <verbatim>
Rule: <the matching section of rulings.md, pasted>
Evidence: <what you found in the repo, task line, or by running something, with file:line>
Options: <the choices as you see them, or the design/spec path for REVIEW>
```

## Steps

### 1. Open the handoff

1. Write the charter marker at `.claude/handoff.active` in the project root with the fields in `references/charter.md`. It survives compaction and lets the SessionStart hook re-arm this skill. It is not committed.
2. Create the decision log at `docs/superpowers/handoff/YYYY-MM-DD-<slug>.md` from `references/log-template.md`. Commit it with the branch so the human reviews it alongside the code.
3. Parse options from the task line. Recognized flags, all optional:
   - `--merge` allows finishing option 1, local merge to the base branch. Default is off.
   - `--no-pr` finishes with option 3, keep the branch. Default is to push and open a PR.
   - `--budget <small|medium|large>` is passed through to superpowers as guidance for its Model Selection: small biases every role one tier down, large biases design and review one tier up, medium changes nothing.
   - `--decider <fable|opus|sonnet|haiku|inherit>` sets the model for every `handoff-proxy` dispatch. Default `fable`. `inherit` omits the model parameter so the proxy runs on the session model.
   - `--ask-on <text>` names one decision the proxy must not rule on. If you reach it, stop and report instead.
4. Announce in one line that handoff is active, the decider model, and the flags. Then stop narrating to the human. The log carries the record.

### 2. Run superpowers with proxy rulings

Invoke `superpowers:brainstorming` and follow it. At each gate, gather the evidence the rule calls for, build the gate packet, dispatch `handoff-proxy`, apply its ruling, and append the ruling to the log. Gates that `references/rulings.md` marks as no dispatch you answer yourself as written there.

For design and spec approval the packet kind is REVIEW and the proxy returns APPROVE or REVISE with cited objections. Address objections and re-dispatch once. After two REVISE rounds, send one final DECIDE packet with the remaining objections and the proxy settles it. Log whatever stays unresolved.

Continue into `superpowers:writing-plans` and `superpowers:subagent-driven-development` (or `executing-plans` when subagents are unavailable) the same way. During execution, superpowers' own "rulings, not stalls" rule and ledger already apply to conflicts inside the plan, and those stay with you as controller. Mirror each ledger ruling into the handoff log with one line so the human has a single place to read.

If the proxy answers with `[ASK]`, check the four hard stops. If it is one of them, stop and go to step 4 with status BLOCKED. Otherwise log it under open decisions, apply the proxy's conservative default, and continue.

### 3. Finish

1. Run `superpowers:verification-before-completion`. If verification fails and cannot be fixed within the fix-loop cap, mark the task BLOCKED in the log and go to step 4 without opening a PR.
2. Run `superpowers:finishing-a-development-branch`. Send its menu to the proxy as a DECIDE packet with the finishing rule and the current status.
3. Fill the handback report section of the log from `references/report-template.md`. Commit and push it.

### 4. Close the handoff

1. Delete `.claude/handoff.active`.
2. Post the handback report as your final message, verbatim from the log, with the PR link first if one exists.

## When the human interrupts

Reply with three lines: where you are in the plan, the last ruling the proxy made, and what you were about to do. Then wait. The charter marker stays until the task ends or the user runs `/handoff off`.

## Ending early

`/handoff off` as the task line closes the handoff without finishing: write the report with status PAUSED, list the next step, delete the marker, and stop. Uncommitted work is committed to the branch as WIP first, never discarded.
