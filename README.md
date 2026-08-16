# scan-research

A Claude Code plugin for light web research.

It splits a topic into three to five sub-questions and runs up to three collection agents in
parallel. The result is one Markdown report, and every claim in that report carries a full URL, a
publication date and a source grade.

There is no verification stage: no evidence ledger, no matching of quotes against the live page,
no verifier agents, no audit gate. A six-line checklist replaces them. The plugin ships as
Markdown only, with no Python and no JSON schema.

## When to use it

| Your request | Use |
|---|---|
| A single fact that one search settles | Plain WebSearch. No skill needed |
| An overview of a topic, with its sources, for a decision you can revisit | **scan-research** |
| High stakes: a budget, an external publication, a management decision, or a subject the sources disagree about | [research-team](https://github.com/tomkd555/research-skill) |
| A literature review with no web component | literature-review, which ships with research-team |

scan-research stops and hands the job to research-team when the sources conflict on the figure the
answer depends on, or when the run passes about 60 searches.

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
| `evidence/{sub-question-slug}.md` | Each collection agent | Its source table, search log and gaps |
| `report.md` | The main session | Answer, findings per sub-question, where sources disagree, source table, what was left out |

## Contents

| Path | What it holds |
|---|---|
| `plugins/scan-research/skills/scan-research/SKILL.md` | The procedure, the collection floors, the evidence standards |
| `plugins/scan-research/agents/scan-collector.md` | The collection agent, one per sub-question, running on Sonnet |

## License

MIT
