---
name: scan-research
description: >-
  Scans a topic in one pass and returns a single sourced Markdown report. It splits the topic into
  three to five sub-questions, launches up to three collection agents in parallel, and writes a
  report whose every claim carries a full URL, a publication date and a source grade — with a
  six-line checklist in place of a verification pipeline. Web-first; academic sources come in only
  where a sub-question turns on effectiveness or causality. Use it when the request wants coverage
  of a subject with its sources attached but does not warrant a verified evidence ledger:
  「ざっと調べて」「一通り調べて」「概況をまとめて」「軽く調査して」「サッと調べて」
  「全体像を掴みたい」, "give me a scan of", "brief me on", "what's the state of".
  Do not use it when being wrong is expensive — money, an external publication, a management
  decision, a contested subject, or an explicit ask for depth — which belongs to research-team;
  nor for a pure literature review with no web side, which belongs to literature-review; nor for a
  single fact one search settles, which needs no skill at all.
allowed-tools:
- WebSearch
- WebFetch
- Read
- Write
- Glob
- Grep
- AskUserQuestion
- Agent
---

# scan-research

A scan sits between answering from one search and running research-team. It buys coverage and
sourcing, and deliberately does not buy verification: no evidence ledger, no quote matching
against the live page, no verifier agents, no audit gate. What replaces them is a floor on
collection, a grade on every source, and a checklist at the end.

Run the whole thing in the main session. The only delegation is collection.

**Language.** Section headings, table headers and the confidence levels are English. The prose
under them follows the user's standing convention for deliverables.

## Step 1 Frame

Write these three lines and keep them; they become the report's header.

- **Purpose** — what this scan is for, in one line.
- **As-of date** — today's date. Everything later is judged against it.
- **Sub-questions** — three to five, each ending in a question mark, together covering the topic.
  Fewer than three means the request was a lookup; more than five means it belongs to research-team.

This skill reports what the web says about the topic. It does not set out to test a hypothesis, so
there is no disconfirmation plan and no duty to hunt for the opposing case. Where the sources
themselves disagree, record the disagreement (Step 3); where they do not, do not manufacture one.

Ask the user at most one AskUserQuestion round, and only when the scope is genuinely ambiguous
(which market, which period, which reader). Otherwise proceed on defaults and state them in the
report header.

Create the output directory `research/{YYYYMMDD}-{topic-slug}/` unless the user names one.

## Step 2 Collect

Launch one `scan-collector` per sub-question, at most three, in a single message. Where there are
four or five sub-questions, take the remainder yourself under the same floors rather than opening
a second wave — a wave costs more wall-clock than the work it carries.

Pass each launch exactly these variables and nothing else; the procedure lives in the agent
definition:

| Variable | Value |
|---|---|
| `{TOPIC}` | The topic as a whole, for context |
| `{SUB_QUESTION}` | This agent's one sub-question, verbatim |
| `{RUN_DIR}` | The absolute path of the output directory |
| `{OUT_PATH}` | `{RUN_DIR}/evidence/{sub-question-slug}.md` |
| `{AS_OF}` | The as-of date from Step 1 |

**Floors, per sub-question.** 4 or more searches; 2 or more independent sources. Across the whole
scan, all three facets below must appear. A floor is a minimum, not a target — but a shortfall is
written into the gaps list, never passed over in silence.

| Facet | What it asks for | Query shape |
|---|---|---|
| definition | The standard understanding and vocabulary | "what is {topic}" / "{topic} overview" |
| data | Figures, scale, trend | "{topic} statistics" / "{topic} market size" |
| practice | How it plays out in application, and what practitioners report | "{topic} case study lessons" / "{topic} 導入 課題" |

Run each query in whatever language its best sources are written in — a domestic rule or a
national statistic exists only in its own language. Use WebSearch's `allowed_domains` for a
primary-source sweep (`go.jp`, `e-stat.go.jp`, `oecd.org`, the regulator's or the company's own
domain) rather than spending a query per `site:`.

**The academic supplement.** Where a sub-question turns on effectiveness, causality or a comparison
of methods, spend one of its queries on scholarly sources and cite the paper by DOI. Nothing else
about the step changes. A request whose whole weight is academic belongs to literature-review.

**Budget.** Roughly 25–40 WebSearch calls for the scan, against Claude Code's 200-per-session cap
shared by the parent and every subagent. Passing 60 means the request outgrew this skill; say so
and offer research-team rather than continuing.

## Step 3 Assemble

Each collector writes its evidence file and returns only the path plus a five-line summary. Read
the files with Read. Never transcribe a table out of a message body — that is where URLs get
mangled, and it is the one discipline this skill keeps from the heavier pipeline.

Merge the per-file source tables into one, numbering the sources `[S1]`, `[S2]`, … in the order
they first appear. Two rows from the same publisher, or two reports traceable to one press
release, are one source: give them one number and note the reprint.

## Step 4 Write

One file, `{RUN_DIR}/report.md`, written in a single pass from the evidence files:

```markdown
# {topic} — scan (as of {AS_OF})

Purpose / scope / assumptions — three lines from Step 1.

## Answer
Three to five lines answering the request, ending with a confidence level and its reason.

## Findings
### {sub-question 1}
Claims, each with its `[S#]`. Figures carry unit, denominator and period.
### {sub-question 2}
…

## Where sources disagree
The points on which the sources collected differ — both values, and why they differ (different
definition, period, or population). Omit the section where they agree.

## Sources
| # | Source | URL | Published | Grade | Corroboration |

## Not covered
What this scan left out, and what a research-team run would add. State here that the report
reflects what these searches surfaced, which is not the same as the balance of informed opinion.
```

**Confidence** is one of three, chosen for the answer as a whole:

| Level | When |
|---|---|
| high | Two or more independent grade-A/B sources agree, and they are current |
| medium | Sources agree but are thin, dated, or partly grade C |
| low | One source only, or the sources conflict and nothing settles which is right |

## Evidence standards

**Grades.** A — governments, international organisations, peer-reviewed papers. B — industry
bodies, corporate IR, major research firms, published firm reports; preprints, marked as such.
C — trade media, news reporting, blogs, forums. A key figure never rests on grade C alone; grade C
corroborates.

**Corroboration**, on every key figure: `corroborated` (two or more independent sources at a
comparable value), `single_source` (usable, but say so in the body), `conflicting` (record both
values and the reason). Never hide a conflict — its structure is worth more than a single number.

**Independence** is counted by origin, not by URL: pages from one publisher are one source, and
reprints or a shared press release are one source once traced back.

**Freshness.** Statistics and market data from the last three years; for fast-moving subjects
(AI, regulation, pricing) the last twelve months. Anything older carries a note on why nothing
newer exists. Every source row carries its publication date, and `unknown` where the page states
none — never a guess.

**Primary over aggregated.** Where a secondary source cites primary data, cite the primary source.
Do not fetch a page the search snippet already shows to be a content farm.

## Step 5 Check

Confirm all six before delivering. A line that fails gets fixed, not waived.

1. Every key figure has a full absolute URL and a publication date.
2. No key figure rests on grade C alone.
3. Every single-source claim is marked as such in the body.
4. Every claim in Findings carries an `[S#]`, and every row in Sources is cited at least once.
5. Every sub-question from Step 1 has a subsection in Findings.
6. The Not-covered section names what was skipped and why.

Then deliver: the answer, the key findings with their `[S#]`, any disagreement between sources, the confidence
level, and the path of `report.md`. Never put a number in the final message that is absent from the
report.

## When a scan outgrows itself

Stop and offer research-team when sources conflict on the figure the answer turns on, when the
stakes turn out to be money or an external publication, or when the budget passes 60 searches.
Say what the scan established, and hand over rather than quietly deepening — a scan that drifts
into a pipeline gets neither the pipeline's rigour nor the scan's speed.
