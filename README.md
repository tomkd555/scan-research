# scan-research

A light research skill for Claude Code. It scans a topic in one pass and returns a single sourced
Markdown report: three to five sub-questions, up to three collection agents in parallel, and a
report whose every claim carries a full URL, a publication date and a source grade.

It buys coverage and sourcing, and deliberately does not buy verification. There is no evidence
ledger, no matching of quotes against the live page, no verifier agents and no audit gate — a
six-line checklist stands in their place. No Python, no JSON schema.

## Where it sits

| Request | Where it goes |
|---|---|
| A single fact one search settles | Plain WebSearch; no skill needed |
| Coverage of a subject, with sources, for a decision you can revisit | **scan-research** |
| Money, an external publication, a management decision, or a contested subject | [research-team](https://github.com/tomkd555/research-skill) |
| A literature review with no web side | literature-review (ships with research-team) |

A scan that runs into conflicting sources on the figure its answer turns on, or past roughly 60
searches, stops and hands over to research-team rather than deepening on its own.

Web-first. Academic sources come in only where a sub-question turns on effectiveness, causality or
a comparison of methods, and are cited by DOI.

## Contents

```
plugins/scan-research/
  agents/scan-collector.md          the collection agent, one per sub-question
  skills/scan-research/SKILL.md     the procedure, the floors, the evidence standards
```

## Install

```
/plugin marketplace add <path to this repository>
/plugin install scan-research@tomkd555
```

## Output

`research/{YYYYMMDD}-{topic-slug}/`

| File | Written by |
|---|---|
| `evidence/{sub-question-slug}.md` | Each collection agent — its source table, search log and gaps |
| `report.md` | The main session — answer, per-sub-question findings, where sources disagree, sources, what was not covered |

## Licence

MIT
