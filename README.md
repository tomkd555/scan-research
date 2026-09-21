# scan-research

A Claude Code plugin for light web research.

It writes a four-line question analysis (the decision the answer serves, the question type, the
presuppositions, the useless answer), derives three to five sub-questions from the decision, and
runs up to three collection agents in parallel, each spending one query on what would make its
answer wrong. The result is one Markdown report: every claim carries a full URL, a publication
date and a source grade, every key figure carries a quoted sentence, the three figures the
answer depends on are re-fetched before delivery, and the Answer names the mechanism behind it.
Documents the user supplies are read alongside the web.

There is no verification pipeline: no evidence ledger, no matching of quotes by script, no
verifier agents, no audit gate. A nine-line checklist replaces them. The plugin ships as Markdown
only, with no Python and no JSON schema.

## When to use it

| Your request | Use |
|---|---|
| A single fact that one search settles | Plain WebSearch. No skill needed |
| An overview of a topic, with its sources, for a decision you can revisit | **scan-research** |
| High stakes: a budget, an external publication, a management decision, or a subject the sources disagree about | [research-team](https://github.com/tomkd555/research-skill) |
| A literature review with no web component | literature-review, which ships with research-team |

scan-research stops and hands the job to research-team when the sources conflict on the figure the
answer depends on, when a falsifier search or a re-fetched page contradicts the answer, when a
presupposition the request rested on turns out to be false, or when the run passes about 60
searches.

The agents search the general web. They reach for academic sources only where a sub-question is
about effectiveness, causality or a comparison of methods, and they cite those by DOI.

## Install

```
/plugin marketplace add https://github.com/tomkd555/scan-research
/plugin install scan-research@tomkd555
```

Then ask for a scan in either language: 「ざっと調べて」「一通り調べて」「概況をまとめて」,
"give me a scan of", "brief me on".

## Output

Everything lands in `research/{YYYYMMDD}-{topic-slug}/`.

| File | Written by | Contents |
|---|---|---|
| `evidence/{sub-question-slug}.md` | Each collection agent | Its source table with quotes, falsifier table, search log and gaps |
| `report.md` | The main session | Question analysis, answer with its mechanism, findings per sub-question with their falsifier, where sources disagree, source table, what was left out |

## Contents

| Path | What it holds |
|---|---|
| `plugins/scan-research/skills/scan-research/SKILL.md` | The procedure, the collection floors, the evidence standards |
| `plugins/scan-research/agents/scan-collector.md` | The collection agent, one per sub-question, running on Sonnet; one falsifier query and a quote on every key figure |

## License

MIT
