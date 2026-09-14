# Handback report

Write this section last and paste it verbatim as the final message. The PR link, when there is one, goes on the first line. Plain sentences, no headers inside the report, no decorative formatting.

```
PR: https://github.com/<owner>/<repo>/pull/<n>        (omit line if no PR)
Status: COMPLETE | BLOCKED | PAUSED
Branch: <name>
Time: <started> to <finished>

What was asked
<the task line, then the classification and the one sentence design summary>

What was built
<three to six sentences. Files touched by area, not a full list. What the user or caller sees differently.>

How it was verified
<the exact commands run and their result lines. Paste the passing output, not a description of it.>

Rulings you should look at first
<the two or three rulings with the highest cost if wrong, each one line with the log line number>

Open decisions
<product or preference calls made by conservative default. One line each: what was chosen, what the alternative was.>

Open findings
<reviewer findings not fixed, each with the adjudication reason. "None" if none.>

Blocked
<tasks marked BLOCKED and the last thing tried. "None" if none.>

Follow-ups
<work discovered but outside the charter. One line each.>

Full log: docs/superpowers/handoff/<date>-<slug>.md
```
