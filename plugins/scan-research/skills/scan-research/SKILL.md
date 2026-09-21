---
name: scan-research
description: >-
  Scans a topic in one pass and returns a single sourced Markdown report. It writes a four-line
  question analysis (the decision, the question type, the presuppositions, the useless answer),
  derives three to five sub-questions from the decision, launches up to three collection agents
  in parallel — each spending one query on what would make its answer wrong — and writes a report
  whose every claim carries a full URL, a publication date and a source grade, whose key figures
  carry a quoted sentence re-fetched before delivery, and whose Answer names the mechanism
  behind it. A nine-line checklist stands in place of a verification pipeline. Web-first, plus
  any documents the user supplies; academic sources come in only where a sub-question turns on
  effectiveness or causality. Use it when the request wants coverage of a subject with its
  sources attached but does not warrant a verified evidence ledger:
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

A scan sits between answering from one search and running research-team. It buys coverage,
sourcing and one round of doubt, and deliberately does not buy verification: no evidence ledger,
no quote matching by script, no verifier agents, no audit gate. What replaces them is a floor on
collection, a grade on every source, a quoted sentence on every key figure with a re-fetch of the
three that matter most, and a checklist at the end.

Run the whole thing in the main session. The only delegation is collection.

**Language.** Section headings, table headers and the confidence levels are English. The prose
under them follows the user's standing convention for deliverables.

## Step 1 Frame

Write the report's header first; it is kept as written.

- **Purpose** — what this scan is for, in one line.
- **As-of date** — today's date. Everything later is judged against it.
- **Question analysis** — four lines:

```markdown
## Question analysis
- Decision: {who does what differently depending on the answer}
- Question type: {descriptive | diagnostic | evaluative | prescriptive | predictive}
- Presuppositions: {what the request assumes} — {verify | accept}
- Useless answer: {the answer that would be sourced and of no use to the decision}
```

  The decision names an actor and an action. A presupposition marked `verify` becomes a
  sub-question; one marked `accept` carries its reason. Write `none identified` when the request
  assumes nothing.

- **Sub-questions** — three to five, each ending in a question mark, each derived from the
  decision: a condition that must hold for the answer to come out one way or the other. A
  sub-question whose answer would leave what the reader does unchanged belongs in "Not covered".
  Fewer than three means the request was a lookup; more than five means it belongs to
  research-team.
- **Falsifier per sub-question** — one line each: what would have to be true for the expected
  answer to be wrong. Where you have no expected answer, the falsifier is the strongest claim on
  the other side of the sub-question. It is the collector's `{FALSIFIER}`.
- **Materials** — the documents the user supplied, as absolute paths, or `none`.

This skill reports what the sources say and why they say it. It runs no disconfirmation plan and
no competing hypotheses; the falsifier query is its one round of doubt. Where the sources
disagree, record the disagreement (Step 3); where they agree, do not manufacture one.

Ask the user at most one AskUserQuestion round, and only when the scope is genuinely ambiguous
(which market, which period, which reader). Otherwise proceed on defaults and state them in the
report header.

Create the output directory `research/{YYYYMMDD}-{topic-slug}/` unless the user names one.

## Step 2 Collect

Launch one `scan-collector` per sub-question, at most three, in a single message. Where there are
four or five sub-questions, take the remainder yourself: run the collector's procedure under the
same floors and write the same evidence file for each, so Step 3 reads every sub-question the same
way. A second wave costs more wall-clock than the work it carries.

Pass each launch exactly these variables and nothing else; the procedure lives in the agent
definition:

| Variable | Value |
|---|---|
| `{TOPIC}` | The topic as a whole, for context |
| `{SUB_QUESTION}` | This agent's one sub-question, verbatim |
| `{FALSIFIER}` | The falsifier line for this sub-question |
| `{MATERIALS}` | The user's documents as absolute paths, or `none` |
| `{RUN_DIR}` | The absolute path of the output directory |
| `{OUT_PATH}` | `{RUN_DIR}/evidence/{sub-question-slug}.md` |
| `{AS_OF}` | The as-of date from Step 1 |

**Floors, per sub-question.** 4 or more searches, one of them the falsifier; 2 or more independent
sources. Across the whole scan, all three facets below must appear. A floor is a minimum; a
shortfall is written into the gaps list, never passed over in silence.

| Facet | What it asks for | Query shape |
|---|---|---|
| definition | The standard understanding and vocabulary | "what is {topic}" / "{topic} overview" |
| data | Figures, scale, trend | "{topic} statistics" / "{topic} market size" |
| practice | How it plays out in application, and what practitioners report | "{topic} case study lessons" / "{topic} 導入 課題" |

Run each query in whatever language its best sources are written in. Use WebSearch's
`allowed_domains` for a primary-source sweep (`go.jp`, `e-stat.go.jp`, `oecd.org`, the
regulator's or the company's own domain); the argument takes a list in one call.

**The academic supplement.** Where a sub-question turns on effectiveness, causality or a comparison
of methods, spend one of its queries on scholarly sources and cite the paper by DOI. A request
whose whole weight is academic belongs to literature-review.

**Budget.** Roughly 25–45 WebSearch calls for the scan (a collector runs its four-search floor
plus what the falsifier and its gaps need, and stops once two queries in a row add nothing),
against Claude Code's 200-per-session cap shared by the parent and every subagent. Passing 60 means
the request outgrew this skill; say so and offer research-team.

## Step 3 Assemble

Each collector writes its evidence file and returns only the path plus a six-line summary. Read
the files with Read. Never transcribe a table out of a message body; that is where URLs get
mangled.

Merge the per-file source tables into one, numbering the sources `[S1]`, `[S2]`, … by origin in
the order they first appear. A publisher with several pages keeps one number and one row per
page, the number repeated, so each row keeps its own URL, date, grade and quote; a reprint or a
report traceable to another row's press release takes that row's number with the reprint noted.
The corroboration count treats one number as one origin. Carry the Quote column through.

## Step 4 Write

One file, `{RUN_DIR}/report.md`, written in a single pass from the evidence files:

```markdown
# {topic} — scan (as of {AS_OF})

Purpose / scope / assumptions — from Step 1.

## Question analysis
(the four lines from Step 1)

## Answer
Three to five lines answering the request, ending with a confidence level and its reason.

One further paragraph: the mechanism or the decision logic behind the answer — why the sources
come out this way — and the one assumption the answer rests on. Where the sources report a
pattern that none of them explains, say that the mechanism is not established.

## Findings
### {sub-question 1}
Claims, each with its `[S#]`. Figures carry unit, denominator and period.
Falsifier: {the query}, {what came back}, {whether it changes the answer}.
### {sub-question 2}
…

## Where sources disagree
The points on which the sources differ — both values, and why (different definition, period, or
population). Omit the section where they agree.

## Sources
| # | Source | URL | Published | Grade | Quote | Corroboration |

## Re-fetch
| Figure | URL | Sentence returned | Outcome |
(the three key figures the Answer depends on; outcome is confirmed, dropped or contradicted)

## Not covered
What this scan left out, what could not be confirmed, and what a research-team run would add.
State here that the report reflects what these searches surfaced, which differs from the balance
of informed opinion.
```

**Confidence** is one of three, chosen for the answer as a whole:

| Level | When |
|---|---|
| high | Two or more independent grade-A/B sources agree, they are current, and no falsifier search moved the answer |
| medium | Sources agree but are thin, dated, or partly grade C |
| low | One source only, the sources conflict and nothing settles which is right, or a falsifier search found evidence against the answer |

## Evidence standards

**Grades.** A — governments, international organisations, peer-reviewed papers. B — industry
bodies, corporate IR, major research firms, published firm reports, a vendor's own product,
pricing and support pages; preprints, marked as such. C — trade media, news reporting, blogs,
forums. A vendor's page is the authority on its own price and a party to every other claim. A key figure never rests on grade C alone; grade C
corroborates. A document the user supplied is graded by its origin (a ministry report is A, an
internal memo B) and is treated as an interested source unless its publisher is independent of
the decision.

**Quotes.** Every key figure carries the sentence from the page that states it, 25 words at most,
copied as the page has it. `no quote` forces `single_source`.

**Corroboration**, on every key figure: `corroborated` (two or more independent sources at a
comparable value), `single_source` (usable, but say so in the body), `conflicting` (record both
values and the reason). Never hide a conflict.

**Independence** is counted by origin: pages from one publisher are one source, and reprints or a
shared press release are one source once traced back. One user-supplied file is one source.

**Freshness.** Statistics and market data from the last three years; for fast-moving subjects
(AI, regulation, pricing) the last twelve months. Anything older carries a note on why nothing
newer exists. Every source row carries its publication date, and `unknown` where the page states
none — never a guess.

**Primary over aggregated.** Where a secondary source cites primary data, cite the primary source.
Do not fetch a page the search snippet already shows to be a content farm.

## Step 5 Check

Confirm all nine before delivering. A line that fails gets fixed.

1. Every key figure has a full absolute URL and a publication date, or `unknown` where the page
   states none (a price page and a certification list rarely carry one).
2. No key figure rests on grade C alone.
3. Every single-source claim is marked as such in the body.
4. Every claim in Findings carries an `[S#]`, and every row in Sources is cited at least once.
5. Every sub-question from Step 1 has a subsection in Findings, and its falsifier line states
   what was searched and what came back.
6. The Not-covered section names what was skipped, what could not be confirmed, and why.
7. The Question analysis block is in the header, and every presupposition marked `verify` has an
   answer in Findings.
8. Every key figure carries a quote of 25 words at most.
9. **The re-fetch.** For the three key figures the Answer most depends on, fetch the page again
   with WebFetch and ask for the sentence containing the figure and nothing else. WebFetch serves
   its own copy of a URL fetched within the last fifteen minutes, so append a harmless query
   string (`?recheck=1`) to get a fresh read. For a figure from a user-supplied file, Grep the
   quoted sentence in the file. A sentence that carries the figure confirms it. A sentence without
   the figure, or an unreachable page, drops the figure from the Answer and goes to "Not covered".
   A sentence that contradicts the figure is a hand-over (below). Record the three outcomes in the
   report's Re-fetch table.

Then deliver: the answer with its mechanism paragraph, the key findings with their `[S#]`, any
disagreement between sources, the re-fetch outcomes in one line, the confidence level, and the
path of `report.md`. Never put a
number in the final message that is absent from the report.

## When a scan outgrows itself

Stop and offer research-team when: sources conflict on the figure the answer turns on; the stakes
turn out to be money or an external publication; the budget passes 60 searches; a falsifier search
returns evidence that contradicts the direction of the answer; a presupposition marked `verify`
turns out to be false and the sources in hand leave the correction unsettled (a presupposition
the sources correct in one line is a finding, written into the Answer); or a re-fetched page
contradicts a figure the Answer depends on. Each of
these means the scan has found the thing it is unequipped to settle. Say what the scan
established, name what it could not, and hand over — a scan that drifts into a pipeline gets
neither the pipeline's rigour nor the scan's speed.
