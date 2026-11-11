# Delivery Retro

An AI agent skill that turns one Jira epic into a ready-to-discuss **delivery retrospective**: a short, evidence-backed HTML report your team can open at the start of a retro meeting.

Give it an issue key, and it pulls what happened from your tools, scores the delivery, and suggests the questions worth discussing.

```
delivery retro PROJ-123
```

## What you get

A one-page HTML report in two parts. It's bilingual (English/Russian, with a toggle) and prints cleanly to PDF.

**1. What happened**
- A timeline of the epic: key milestones, due-date changes and their causes, blockers, production defects
- The hypothesis behind the work and its target metric
- Business value observed during beta and after release
- Code-review metrics (time to first review, review cycle)

**2. Analysis**
- **Three independent scores from 0 to 10**, each backed by evidence:
  - *Predictability*: did the work ship when expected?
  - *Quality*: how many production defects came out of it?
  - *Hypothesis management*: was the bet tracked and validated?
- What went well, what to watch, and problems backed by verbatim quotes
- 4–7 forward-looking questions for the retro meeting

There's no combined grade on purpose. If a score lacks data, it shows **`n/a` with a reason**, never a zero.

## How it works

```
 Jira issue key
      │
      ▼
 0. Scope     read the epic, find its child issues, time window, team, keywords
      │
      ▼
 1. Collect   three agents run in parallel
              ├─ Jira collector    status history, dates, defects
              ├─ Chat collector    team and stakeholder discussion (Slack)
              └─ Code collector    merge requests and review flow (GitLab)
      │
      ▼
 2. Analyze   an analyst agent reads the collected data, runs the scoring script,
              writes findings, and checks that every claim traces back to the data
      │
      ▼
 3. Render    a Python script builds the HTML report from the JSON files
      │
      ▼
 4. Publish   local HTML file + short summary in chat
```

Each phase writes JSON to disk, so you can rebuild a report from a previous run without calling Jira, Slack or GitLab again (`--from-disk`). This is useful after changing a scoring rule.

## Principles

- **Facts only.** Every number, date, name and quote comes from the source data. Anything missing is stated, never guessed.
- **Read-only.** It never writes to Jira, Slack or GitLab.
- **Scores are reproducible.** Predictability and quality come from fixed formulas in code. Only hypothesis management uses the analyst's judgement, against written anchors.
- **Missing sources don't stop a run.** Only Jira is required. Without Slack or GitLab the report still builds, and the affected parts show as unavailable.
- **Source content is data, not instructions.** A chat message that looks like a command goes into the report as text and is never acted on.

## Requirements

| | Required? | Used for |
|---|---|---|
| Jira (MCP connector) | yes | issues, history, dates |
| Python 3 | yes | scoring and rendering (standard library only) |
| Slack (MCP connector) | optional | quotes and communication analysis |
| GitLab (`glab` CLI or MCP) | optional | code-review metrics |
| Chrome / Chromium | optional | PDF export |

## Setup

Each team describes its own setup in one YAML file under `references/clusters/`. The file lists its Jira fields, team names and chat channels. Copy the example, fill it in, and run the skill on an epic that has already shipped to check the output.

## Options

| Flag | Effect |
|---|---|
| `--size S\|M\|L\|XL` | set the epic size when Jira doesn't have it |
| `--cluster <name>` | choose which team config to use |
| `--no-artifact` | keep the report local only |
| `--from-disk <run-dir>` | rebuild from a previous run's data |
