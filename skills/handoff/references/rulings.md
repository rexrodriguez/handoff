# Rulings

One entry per place a superpowers skill stops for the human. Each says what the human would normally decide, how you decide it instead, and what to log. If a gate is not listed here, the default ruling applies: take the most reversible option that matches existing repo conventions, log it with the cost if wrong, and continue.

Every logged ruling uses superpowers' own shape:

```
Ruling: <what you decided> — <why> — <what it costs if wrong>
```

## brainstorming

### Gate: classify spike, bounded, or architectural

Human role: override the classification.
Ruling: classify per the skill's definitions. When in doubt take the heavier path, exactly as the skill says. Log the classification and the one fact that decided it.

### Gate: clarifying questions, one at a time

Human role: answer them.
Ruling: write each question down, then answer it yourself in this order.

1. **From the repo.** Read code, tests, docs, recent commits, and any CLAUDE.md. Most questions about conventions, naming, placement, and existing behavior are answerable here. Cite the file.
2. **From the task line.** Re-read the charter. The user often already answered it.
3. **By observing.** If the question is empirical, for example "does X handle Y", "is this fast enough", "what does the API return", run it. A five minute probe beats a guess.
4. **By convention.** For a preference question no evidence settles, choose the option that matches what the repo already does. Where the repo has no precedent, choose the smaller and more reversible option.
5. **Product calls.** A question about what the product should do for users, with real cost either way and no precedent, is the one kind you cannot answer well. Choose the conservative default that changes the least for existing users, log it under `Open decisions` in the report, and continue. If the charter named it in `--ask-on`, stop instead.

Log every question and its answer with which of the five sources decided it.

### Gate: propose 2 to 3 approaches with a recommendation

Human role: pick one.
Ruling: pick your own recommendation. The skill already requires you to recommend, so the ruling is only to act on it. If two approaches are close, prefer the one with the smaller blast radius and fewer new files. Log the approaches considered in one line each and why the pick won.

### Gate: present design and get approval, per section for architectural work

Human role: approve or push back.
Ruling: dispatch `handoff-proxy-reviewer` with the design. It reads the repo and returns APPROVE or REVISE with specific objections. On REVISE, address the objections and re-dispatch once. After two REVISE rounds, decide yourself, log the unresolved objections, and continue. Never approve your own design without the reviewer.

### Gate: user reviews written spec

Human role: read the spec file before implementation.
Ruling: run the skill's spec self-review, then dispatch `handoff-proxy-reviewer` against the spec file with the same APPROVE or REVISE contract. Log the verdict.

### Gate: visual companion offer

Human role: accept or decline the browser tab.
Ruling: decline. There is nobody to look at it. Log nothing.

## writing-plans

No human gate. Runs as written.

## executing-plans and subagent-driven-development

### Gate: raise plan concerns with the human before starting

Human role: resolve concerns.
Ruling: resolve each concern yourself. A concern that changes the plan's tasks gets fixed in the plan file and committed. A concern that changes the design goes back to the design gate above, one round only. Log each concern and its resolution.

### Gate: the four stop reasons

These are not proxied. Stop and write the report when you hit any of them.

1. Irreversible or destructive operation. Deleting data, rewriting history on a shared branch, dropping columns, removing files outside the worktree.
2. Security-sensitive action. Touching credentials, auth flows, permissions, secrets, or anything that widens access.
3. Side effect outside the worktree that norms say you ask about first. A merge to a shared branch is always this. A push of your own feature branch and opening a PR is not, and is the default finish. A publish, release, or deploy is always this.
4. A plan so broken every path is a guess.

`--merge` moves local merge to the base branch out of reason 3 and into the finishing ruling below. Nothing else moves.

### Gate: implementer asks questions

Human role: none. The controller answers the implementer. Runs as written.

### Gate: fix loop reaches round 5 with open findings

Human role: none. The skill adjudicates. Mirror the adjudication into the handoff log.

### Gate: implementer escalates that it is stuck

Human role: none. The skill re-dispatches with more context or a stronger model. If the same task fails after that, mark the task BLOCKED in the log, continue with tasks that do not depend on it, and list it under `Blocked` in the report.

## verification-before-completion

No human gate. Runs as written. A failing verification is never rounded up to passing. If it cannot be made to pass within the fix-loop cap, the handoff finishes with status BLOCKED and no PR.

## requesting-code-review and receiving-code-review

No human gate during a handoff. Final reviewer findings are handled by subagent-driven-development's one fix dispatch and adjudication. Load-bearing findings you could not fix are listed under `Open findings` in the report.

## finishing-a-development-branch

### Gate: the options menu

Human role: choose merge locally, push and open a PR, or keep the branch.
Ruling, in order:

1. Status BLOCKED or PAUSED: option 3, keep the branch. Push it if a remote exists so the work is not only local.
2. `--no-pr` set: option 3, keep the branch, pushed.
3. `--merge` set and status COMPLETE: option 1, merge locally to the base branch. Never push the base branch afterward. The human pushes.
4. Otherwise: option 2, push and create a pull request. The PR body is the handback report. Title in conventional commit form.

On a detached HEAD, the same order applies with the two-option menu.

Log the option chosen and why.

### Gate: discard the work

Never. Discarding happens only when the human asks in person.

## Anything else

A gate not listed here is answered by the default ruling at the top of this file. Add it to this file afterward so the next handoff has a rule, which is the `superpowers:writing-skills` way of encoding a lesson.
