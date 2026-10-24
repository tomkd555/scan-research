# scan-research

A Claude Code plugin for light web research. It derives three or four sub-questions from the
decision the answer serves and collects each one in parallel on a fixed budget. It writes one
sourced Markdown report, and before delivery a verifier in a fresh context checks the Answer's
sentences against their sources. Every claim has a full URL, a publication date and a source
grade, and every key figure has a quote from its page. The plugin consists of Markdown files only.

## When to use it

| Your request | Use |
|---|---|
| A fact that one search answers | WebSearch on its own |
| An overview of a topic, with its sources, for a decision you can revisit | **scan-research** |
| High stakes: a budget, an external publication, a management decision, or a subject the sources disagree about | [research-team](https://github.com/tomkd555/research-skill) |
| A literature review with no web component | literature-review, which ships with research-team |

A scan stops and offers research-team when:

- the request needs more than four sub-questions
- the sources conflict on the figure the answer depends on
- a sub-question the answer turns on has fewer than two independent sources
- a falsifier search or the verifier finds evidence against the answer
- the stakes turn out to be money or an external publication

## Install

```
/plugin marketplace add https://github.com/tomkd555/scan-research
/plugin install scan-research@tomkd555
```

Then ask for a scan in either language: 「ざっと調べて」「一通り調べて」「概況をまとめて」,
"give me a scan of", "brief me on". A run writes into `research/{YYYYMMDD}-{topic-slug}/`:

| File | Content |
|---|---|
| `report.md` | question analysis, the answer with its mechanism, findings per sub-question, sources, verification outcomes, what was left out |
| `evidence/sq{n}.md` | each collector's source table with quotes, falsifier, search log and gaps |
| `claims.md`, `verdicts.md` | the Answer sentences sent to the verifier, and its verdicts |

| Agent | Model | Role |
|---|---|---|
| scan-collector | Sonnet, medium effort | one sub-question, 8 tool calls, one falsifier query |
| scan-verifier | Opus | up to 5 Answer sentences against their sources, without the report |

## License

MIT
