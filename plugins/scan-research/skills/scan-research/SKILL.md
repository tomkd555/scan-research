---
name: scan-research
description: >-
  Scans a topic in one pass and returns one sourced Markdown report. It writes a four-line
  question analysis (the decision, the question type, the presuppositions, the useless answer),
  derives three to five sub-questions from the decision, and launches up to three collection
  agents in parallel, each spending one query on what would make its answer wrong. In the report,
  every claim has a full URL, a publication date and a source grade, every key figure has a
  quote from its page re-fetched before delivery, and the Answer states the mechanism behind it.
  Web-first, plus any documents the user supplies; academic sources come in only where a
  sub-question turns on effectiveness or causality. Use it when the request wants coverage of a
  subject with its sources attached but does not warrant a verified evidence ledger:
  「ざっと調べて」「一通り調べて」「概況をまとめて」「軽く調査して」「サッと調べて」
  「全体像を掴みたい」, "give me a scan of", "brief me on", "what's the state of".
  Requests where being wrong is expensive go to research-team: money, an external publication, a
  management decision, a contested subject, or an explicit ask for depth. A pure literature
  review with no web side goes to literature-review, and a fact that one search answers needs
  only WebSearch.
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

A scan is the step between one search and a research-team run. It provides coverage, sourcing
and one round of doubt. Its safeguards are a floor on collection, a grade on every source, a
quote from the page on every key figure, and a re-fetch of the three key figures that matter most. The
evidence ledger, script-matched quotes, verifier agents and audit gate belong to research-team.

Run the whole thing in the main session. The only delegation is collection. Pause for the user
only at the two points this skill names: the scope question in Step 1 and the hand-over to
research-team. Between them, carry every step through to delivery.

**Language.** Section headings, table headers and the confidence levels are English. The prose
under them follows the user's standing convention for deliverables.

## Step 1 Frame

Write the report's header first and keep it as written.

- Purpose: what this scan is for, in one line.
- As-of date: today's date. Everything later is judged against it.
- Question analysis, in four lines:

```markdown
## Question analysis
- Decision: {who does what differently depending on the answer}
- Question type: {descriptive | diagnostic | evaluative | prescriptive | predictive}
- Presuppositions: {what the request assumes}, marked {verify | accept}
- Useless answer: {the answer that would be sourced and of no use to the decision}
```

  The Decision line states an actor and an action. A presupposition marked `verify` becomes a
  sub-question; one marked `accept` gives its reason. Write `none identified` when the request
  assumes nothing.

- Sub-questions: three to five, each ending in a question mark, each derived from the
  decision as a condition that must hold for the answer to come out one way or the other. A
  sub-question whose answer would leave what the reader does unchanged belongs in "Not covered".
  Fewer than three means the request was a lookup; more than five means it belongs to
  research-team.
- Falsifier per sub-question: one line each, stating what would have to be true for the
  expected answer to be wrong. Where you have no expected answer, the falsifier is the strongest
  claim on the other side of the sub-question. It is the collector's `{FALSIFIER}`.
- Materials: the absolute paths of the documents the user supplied, or `none`.

This skill reports what the sources say and why they say it. Its one round of doubt is the
falsifier query. Disconfirmation plans and competing hypotheses belong to research-team. Where the
sources disagree, record the disagreement in Step 3. Where they agree, report the agreement as it
stands.

Ask the user at most one AskUserQuestion round, and only when the scope is ambiguous: which
market, which period, which reader. Otherwise proceed on defaults and state them in the report
header.

Create the output directory `research/{YYYYMMDD}-{topic-slug}/` unless the user gives a path.

## Step 2 Collect

Launch one `scan-collector` per sub-question, at most three, in one message. Where there are four
or five sub-questions, take the remainder yourself: run the collector's procedure under the same
floors and write the same evidence file for each, so Step 3 reads every sub-question the same way.
A second wave of collectors would add more wall-clock time than it saves. Tell the user in one line
which sub-questions are being collected. The collectors run in the background; collect the
remainder while they run.

Pass each launch exactly these variables. The agent definition contains the procedure and sets the
collector's model and effort, so leave both unset at launch:

| Variable | Value |
|---|---|
| `{TOPIC}` | The topic as a whole, for context |
| `{DECISION}` | The Decision line from the question analysis; the collector uses it to tell which figures are key |
| `{SUB_QUESTION}` | This agent's one sub-question, verbatim |
| `{FALSIFIER}` | The falsifier line for this sub-question |
| `{MATERIALS}` | The user's documents as absolute paths, or `none` |
| `{RUN_DIR}` | The absolute path of the output directory |
| `{OUT_PATH}` | `{RUN_DIR}/evidence/{sub-question-slug}.md` |
| `{AS_OF}` | The as-of date from Step 1 |

**Floors, per sub-question.** 4 or more searches, one of them the falsifier; 2 or more independent
sources. Across the whole scan, all three facets below must appear. A floor is a minimum. Write
any shortfall into the gaps list.

| Facet | What it asks for | Query shape |
|---|---|---|
| definition | The standard understanding and vocabulary | "what is {topic}" / "{topic} overview" |
| data | Figures, scale, trend | "{topic} statistics" / "{topic} market size" |
| practice | How it plays out in application, and what practitioners report | "{topic} case study lessons" / "{topic} 導入 課題" |

Run each query in whatever language its best sources are written in. Use WebSearch's
`allowed_domains` for a primary-source search (`go.jp`, `e-stat.go.jp`, `oecd.org`, the
regulator's or the company's own domain). The argument takes a list of domains in one call.

**The academic supplement.** Where a sub-question turns on effectiveness, causality or a comparison
of methods, spend one of its queries on scholarly sources and cite the paper by DOI. A request
whose whole weight is academic belongs to literature-review.

**Budget.** Roughly 25 to 45 WebSearch calls for the scan. A collector runs its four-search floor
plus what the falsifier and its gaps need, and stops once two queries in a row return only
sources it already has. The cap is Claude Code's 200 searches per session, shared by the parent
and every subagent. Passing 60 means the request outgrew this skill. Say so and offer
research-team.

## Step 3 Assemble

Each collector writes its evidence file and returns only the path plus a six-line summary. Read
the files with Read and take every table from the files. A table copied out of a message body is
where URLs get mangled.

Merge the per-file source tables into one, numbering the sources `[S1]`, `[S2]`, … by origin in
the order they first appear. A publisher with more than one page gets one number and one row per
page. Each of those rows has its own URL, date, grade and quote. A reprint, or a report traceable
to another row's press release, takes that row's number, with the reprint noted. The
corroboration count treats one number as one origin. Carry the Quote column through.

## Step 4 Write

One file, `{RUN_DIR}/report.md`, written in one pass from the evidence files:

```markdown
# {topic}: scan as of {AS_OF}

Purpose, scope and assumptions from Step 1.

## Question analysis
(the four lines from Step 1)

## Answer
Three to five lines answering the request, ending with a confidence level and its reason.

One further paragraph on the mechanism behind the answer: why the sources come out this way, or,
for an evaluative or prescriptive question, the trade-off that decides it. State the one
assumption the answer rests on. Where the sources report a pattern that none of them explains,
say that the mechanism is not established.

## Findings
### {sub-question 1}
Claims, each with its `[S#]`. Figures carry unit, denominator and period.
Falsifier: {the query}, {what came back}, {whether it changes the answer}.
### {sub-question 2}
…

## Where sources disagree
The points on which the sources differ: both values, and the reason, such as a different
definition, period or population. Omit the section where they agree.

## Sources
| # | Source | URL | Published | Grade | Quote | Corroboration |

## Re-fetch
| Figure | URL | Words returned | Outcome |
(the three key figures the Answer depends on; outcome is confirmed, dropped, contradicted or
unreadable)

## Not covered
What this scan left out, what could not be confirmed, and what a research-team run would add.
State here that the report reflects what these searches surfaced, which differs from the balance
of informed opinion.
```

Write every claim in the Answer and Findings from the evidence files, including points you
believe you already know. A point that only your own knowledge supports goes to "Not covered". Findings state each
claim in your own words with its `[S#]`. A page's own wording appears only in the Quote column,
or in quotation marks beside its `[S#]`, so the reader can tell the source's words from the
summary:

<example>
Findings line: Public chargers in the region grew by about a quarter in 2024 [S3], and the
agency calls the network "dense along main roads, thin inland" [S3].
Why it is correct: the claim is in the writer's own words with its source number, and the one
phrase copied from the page is in quotation marks beside it.
</example>

Sources lists only the rows the Answer or Findings cite, each with its full
absolute URL. Size the report to its findings: the template's sections and nothing more, no
summary that repeats the Answer, and a literal phrase wherever a metaphor would stand.

**Confidence** is one of three, chosen for the answer as a whole:

| Level | When |
|---|---|
| high | Two or more independent grade-A/B sources agree, they are current, and no falsifier search moved the answer |
| medium | Sources agree but are thin, dated, or partly grade C |
| low | One source only, the sources conflict and none of them shows which is right, or a falsifier search found evidence against the answer |

## Evidence standards

**Grades.**

- A: governments, international organisations, peer-reviewed papers.
- B: industry bodies, corporate IR, major research firms, published firm reports, and a vendor's
  own product, pricing and support pages. Preprints are B, marked `preprint`.
- C: trade media, news reporting, blogs, forums.

A vendor's page is the authority on its own price and a party to every other claim. A key figure
needs a grade A or B source. Grade C sources corroborate. A document the user supplied is graded
by its origin: a ministry report is A, an internal memo B. It counts as an interested source
unless its publisher is independent of the decision.

**Quotes.** Treat as a key figure any number the Answer or the decision could turn on. Give every
key figure the words from the page that state it, 125 characters at most, copied as the page has
them. WebFetch copies text word for word only up to that length. `no quote` forces
`single_source`.

**Corroboration**, on every key figure: `corroborated` (two or more independent sources at a
comparable value), `single_source` (usable, but say so in the body), `conflicting` (record both
values and the reason). Never hide a conflict.

**Independence** is counted by origin: pages from one publisher are one source, and reprints or a
shared press release are one source once traced back. One user-supplied file is one source.

**Freshness.** Statistics and market data from the last three years; for fast-moving subjects
(AI, regulation, pricing) the last twelve months. Anything older needs a note on why nothing newer
exists. Give every source row the publication date the page states, or `unknown` where it states
none.

**Primary sources.** Where a secondary source cites primary data, cite the primary source.
Do not fetch a page the search snippet already shows to be a content farm.

## Step 5 Re-fetch

The quotes came from the collectors' reads; this step tests the quotes of the three key figures
the Answer most depends on against the page itself. Fetch the three pages again with WebFetch in one message, asking each to
return only the words around the figure, 125 characters at most. WebFetch serves its own copy
of a URL fetched within the last fifteen minutes, so append a harmless query string (`?recheck=1`)
to get a fresh read. For a figure from a user-supplied file, Grep the quoted words in the file.

- `confirmed`: the words returned contain the figure.
- `dropped`: the words lack the figure, or the page is unreachable. The figure leaves the Answer
  and goes to "Not covered".
- `contradicted`: the words state a different value. This is a hand-over (below).
- `unreadable`: the page loads, but WebFetch returns garbled text, as it does for some Shift_JIS
  pages. The figure stays, and the body says the re-fetch could not confirm it.

Edit the report in place: fill the Re-fetch table with the three outcomes and move any dropped
figure out of the Answer.

Then deliver, in this order:

- the answer with its mechanism paragraph
- the key findings with their `[S#]`
- any disagreement between sources
- the re-fetch outcomes, in one line
- the confidence level
- the path of `report.md`

Every number in the final message comes from the report.

## Hand-over to research-team

Stop and offer research-team when any of these happens:

- Sources conflict on the figure the answer turns on.
- The stakes turn out to be money or an external publication.
- The budget passes 60 searches.
- A falsifier search returns evidence against the direction of the answer.
- A presupposition marked `verify` turns out to be false, and the sources in hand leave the
  correction open. A presupposition the sources correct in one line is a finding, written into
  the Answer.
- A re-fetched page contradicts a figure the Answer depends on.

Each of these is a question the scan's checks cannot resolve. Say what the scan established and
what it could not, then hand over. A scan extended toward a pipeline loses its speed and still
lacks the pipeline's checks.
