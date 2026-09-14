# handoff

A companion plugin for [superpowers](https://github.com/obra/superpowers). It stands in for you at every point where superpowers would stop and ask, so you can hand a task to the agent and read the result later.

Superpowers is unchanged. Brainstorming, writing-plans, subagent-driven-development, TDD, verification, code review, and finishing all run exactly as written, including superpowers' own model selection. What changes is who answers when a skill says "get approval", "ask your human partner", or "which option?". During a handoff, that is the agent, following a fixed set of rulings, and every ruling is logged.

## Install

Requires superpowers to be installed first.

```bash
claude plugin marketplace add <path-or-github-url-of-this-folder>
```

```bash
claude plugin install handoff@rex-plugins
```

## Use

```
/handoff add a --dry-run flag to the export command
```

Optional flags anywhere in the task line:

| Flag | Effect |
|---|---|
| `--merge` | Allow finishing option 1, local merge to the base branch. Off by default. |
| `--no-pr` | Finish by keeping the pushed branch instead of opening a PR. |
| `--budget small\|medium\|large` | Passed to superpowers' Model Selection as a bias. Small tiers every role down one step. Large tiers design and review up one step. |
| `--ask-on "<decision>"` | Name one decision the agent must not rule on. Reaching it ends the handoff with a report. |

`/handoff off` ends an active handoff early. Work is committed as WIP, never discarded.

Come back and type anything. Proxy mode ends at your first message and normal superpowers gates resume for the rest of the session.

## What you get back

The final message is a handback report with the PR link first, what was built, the exact verification output, the two or three rulings with the highest cost if wrong, product calls made by conservative default, unfixed reviewer findings, blocked tasks, and follow-ups. The full decision log is committed at `docs/superpowers/handoff/<date>-<slug>.md` so it travels with the PR.

## What still stops

Superpowers' four hard stops are never proxied: irreversible or destructive operations, security-sensitive actions, merges to shared branches and publishes, and a plan so broken every path is a guess. Discarding work never happens without you in the room.

## How the rulings work

`skills/handoff/references/rulings.md` lists each superpowers gate and the rule that replaces you. Clarifying questions are answered from the repo first, then the task line, then by running an experiment, then by repo convention, and only last by a conservative product default that gets flagged in the report. Design and spec approval are given by a read-only reviewer agent pinned to Fable that returns APPROVE or REVISE with cited objections. The finishing menu defaults to push and open a PR.

A gate with no listed rule gets the default: the most reversible option that matches existing repo conventions, logged with its cost if wrong. Add the new rule to the file afterward.

## Surviving compaction

A short charter marker at `.claude/handoff.active` records the task, flags, branch, log path, and current step. A SessionStart hook on `compact` and `resume` re-injects it when present and stays silent otherwise. The marker is deleted when the handoff closes and should not be committed.

## Which model makes the rulings

The main session's model. Set `/model` before `/handoff`. Fable for the most judgment, Sonnet for the tightest budget. Implementers and reviewers are still tiered by superpowers' Model Selection regardless. The design and spec proxy reviewer is always Fable.

## Orchestrator mode

By default the handoff runs in the main session, which keeps your transcript readable and lets your next message interrupt it. If you would rather the whole run happen inside one Fable subagent, add two lines to `skills/handoff/SKILL.md` frontmatter:

```yaml
context: fork
agent: handoff-orchestrator
```

`agents/handoff-orchestrator.md` ships ready for this. Trade-offs: the run is not in your main transcript, an interrupting message has to be relayed to the subagent, and superpowers' bootstrap normally tells subagents to ignore its process, which the orchestrator agent overrides for itself.

## Files

```
.claude-plugin/plugin.json          manifest, depends on superpowers
.claude-plugin/marketplace.json     lets this folder act as its own marketplace
skills/handoff/SKILL.md             the /handoff command
skills/handoff/references/rulings.md        one ruling per superpowers gate
skills/handoff/references/charter.md        marker file format
skills/handoff/references/log-template.md   decision log
skills/handoff/references/report-template.md handback report
agents/handoff-proxy-reviewer.md    Fable, read-only, APPROVE or REVISE
agents/handoff-orchestrator.md      optional, see Orchestrator mode
hooks/hooks.json, hooks/session-start, hooks/run-hook.cmd   re-arm after compaction
```

MIT licensed.
