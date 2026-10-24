---
name: scan-research
description: >-
  Scans a topic in one pass and returns one sourced Markdown report. It writes a four-line
  question analysis (the decision, the question type, the presuppositions, the useless answer),
  derives three or four sub-questions from the decision, and launches one collection agent per
  sub-question in parallel, each on a budget of eight tool calls with one query spent on what
  would make its answer wrong. Before delivery, a verification agent in a fresh context checks up
  to five Answer sentences against their sources, and the main session rewrites what it flags. In
  the report, every claim has a full URL, a publication date and a source grade, every key figure
  has a quote from its page, and the Answer states the mechanism behind it. Web-first, plus any
  documents the user supplies; academic sources come in only where a sub-question turns on
  effectiveness, causality or a comparison of methods. Use it when the request wants coverage of a
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
- Edit
- Glob
- Grep
- AskUserQuestion
- Agent
---

# scan-research

A scan is the step between one search and a research-team run. Spend its tokens on searching and
on checking the sentences the Answer rests on, and none on re-reading. The collectors search and
read the pages. The main session reads only their evidence files, the verdicts and the agents'
summaries, so its context stays small for every turn that re-reads it. The one exception is the
fallback under Errors. Its safeguards are a fixed collection budget, a grade
on every source, a quote from the page on every key figure, and one verifier that checks up to
five Answer sentences without seeing the report. research-team adds a script that matches every
quote against a cached copy of its page, verification of up to 10 sentences (STANDARD) or 25
(DEEP), and a script check of the report and its translation.

Pause for the user only at the scope question in Step 1 and the hand-over to research-team.
Before each step, say in one line what it does; when a wait ends, say in one line what came back.
While agents run, do the work that does not depend on them. When nothing is left, end the turn
with one line on what is running; their notifications resume the scan.

**Language.** Section headings, table headers and the confidence levels are English. The prose
under them follows the user's standing convention for deliverables.

## Step 1 Frame

Draft the report's header first and keep it as written.

- Purpose: what this scan is for, in one line.
- As-of date `{AS_OF}`: today's date as YYYY-MM-DD. Everything later is judged against it.
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

- Sub-questions: three or four, each ending in a question mark, each derived from the decision
  as a condition that must hold for the answer to come out one way or the other. A sub-question
  whose answer would leave what the reader does unchanged belongs in "Not covered". Count the
  `verify` presuppositions among them. Fewer than three means the request was a lookup: answer it
  with WebSearch in the conversation and write no files. More than four is a hand-over (below).
- Falsifier per sub-question: one line each, stating what would have to be true for the
  expected answer to be wrong. Where you have no expected answer, the falsifier is the strongest
  claim on the other side of the sub-question.
- Materials: the absolute paths of the documents the user supplied, or `none`.

This skill reports what the sources say and why they say it. Its one round of doubt during
collection is the falsifier query. Disconfirmation plans and competing hypotheses belong to
research-team. Where the sources disagree, record the disagreement in Step 4. Where they agree,
report the agreement as it stands.

Ask the user at most one AskUserQuestion round, and only when the scope is ambiguous: which
market, which period, which reader. Otherwise proceed on defaults and state them in the report
header.

The output directory is `research/{YYYYMMDD}-{topic-slug}/` unless the user gives a path. It is
`{RUN_DIR}` below, as an absolute path. The first Write into it creates it.

## Step 2 Collect

Launch one `scan-research:scan-collector` per sub-question, all in one message, each with
`model: sonnet`.
Tell the user in one line which sub-questions are being collected. Each launch prompt puts the
text shared by every launch first, so the launches share a cached prefix:

```text
Collect evidence for one sub-question, following your agent definition.

- Topic: {the topic as a whole}
- Decision: {the Decision line}
- As-of date: {AS_OF}
- Run directory: {RUN_DIR}
- Materials: {absolute paths, or none}

<assignment>
- Sub-question: {this agent's sub-question, verbatim}
- Falsifier: {the falsifier line for this sub-question}
- Evidence path: {RUN_DIR}/evidence/sq{n}.md, n being the sub-question's number
</assignment>
```

Each collector has a budget of eight tool calls and spends one of them on the falsifier. The
collector's definition holds the facets, the grades and the quote rule, so a scan spends about 32
tool calls on collection, plus up to 8 for each relaunch. Claude Code caps WebSearch at 200 calls per session, shared by the
parent and every subagent.

While the collectors run, write the report's header and question analysis into
`{RUN_DIR}/report.md`.

## Step 3 Assemble

Each collector writes its evidence file and returns only the path plus a seven-line summary. Read
the files with Read and take every table from the files. A table copied out of a message body is
where URLs get mangled.

Merge the per-file source tables into one, numbering the sources `[S1]`, `[S2]`, … by origin in
the order they first appear. A publisher with more than one page gets one number and one row per
page. Each of those rows has its own URL, date, grade and quote. A reprint, or a report traceable
to another row's press release, takes that row's number, with the reprint noted. The
corroboration count treats one number as one origin. Carry the Quote column through.

## Step 4 Write

Complete `{RUN_DIR}/report.md` in one pass from the evidence files:

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
Claims, each sentence ending with its `[S#]`. Figures carry unit, denominator and period.
Falsifier: {the query}, {what came back}, {whether it changes the answer}. Where nothing came
back, the line cites nothing.
### {sub-question 2}
…

## Where sources disagree
The points on which the sources differ: both values, and the reason, such as a different
definition, period or population. Omit the section where they agree.

## Sources
| # | Source | URL | Published | Grade | Quote | Corroboration |

## Verification
| # | Sentence | Verdict | Action |
(the Answer sentences sent to the verifier in Step 5, with its verdicts and what was done)

## Not covered
What this scan left out, what could not be confirmed, and what a research-team run would add.
State here that the report reflects what these searches surfaced, which differs from the balance
of informed opinion.
```

Write every claim in the Answer and Findings from the evidence files, including points you
believe you already know. A point that only your own knowledge supports goes to "Not covered".
Every sentence that states a fact ends with its `[S#]`, and every sentence with a number carries
one. Findings state each claim in your own words. A page's own wording appears only in the Quote
column, or in quotation marks beside its `[S#]`, so the reader can tell the source's words from
the summary:

<example>
Findings line: Public chargers in the region grew by about a quarter in 2024 [S3], and the
agency calls the network "dense along main roads, thin inland" [S3].
Why it is correct: the claim is in the writer's own words with its source number, and the one
phrase copied from the page is in quotation marks beside it.
</example>

The collector's definition, `../../agents/scan-collector.md` relative to this skill's base
directory, holds the evidence standards. The report applies four of them:

- A key figure in the Answer needs a grade A or B source. Grade C sources corroborate.
- A figure marked `single_source` says so in the body. A figure marked `conflicting` goes to
  "Where sources disagree" with both values; a conflict is never hidden.
- A user-supplied document counts as an interested source unless its publisher is independent of
  the decision.
- A statistic older than three years, or older than twelve months on a fast-moving subject, comes
  with a note on why nothing newer exists.

Sources lists only the rows the Answer or Findings cite, each with its full absolute URL. Size the
report to its findings: the template's sections and nothing more, no summary that repeats the
Answer, and a literal phrase wherever a metaphor would stand.

**Confidence** is one of three, chosen for the answer as a whole:

| Level | When |
|---|---|
| high | Two or more independent grade-A/B sources agree, they are current, no falsifier search moved the answer, and every Answer sentence sent to the verifier came back `supported` |
| medium | Sources agree but are thin, dated, or partly grade C |
| low | One source only, the sources conflict and none of them shows which is right, or a falsifier search found evidence against the answer |

## Step 5 Verify

Take the sentences in the Answer section that state a fact, up to five, those with a figure
first. The assumption sentence of the mechanism paragraph stays out. Write them to
`{RUN_DIR}/claims.md`, each with the source rows that state its claim, at most two:

```markdown
## C1
Sentence: {the sentence as written, without its [S#]}
- {full URL or file:// path} | {grade} | {the row's quote, or no quote}
```

Launch one `scan-research:scan-verifier` with `model: opus` and pass the two paths only. The report, the
question analysis and your reasoning stay out of the prompt, because a verifier shown the intent
reads the intent into the source.

```text
Judge the claims in the file below, following your agent definition.

- Claims: {RUN_DIR}/claims.md
- Write verdicts to: {RUN_DIR}/verdicts.md
```

When its verdicts arrive, read `verdicts.md` and act on each sentence:

| Verdict | Action |
|---|---|
| `supported` | Keep the sentence |
| `partial` | Rewrite it to what the passage in the verdict supports |
| `unsupported` | Remove it from the Answer; the point goes to "Not covered" |
| `unreadable` | Keep it, and say in the sentence that the re-read could not confirm it |
| `contradicted` | Hand over to research-team (below) |

Edit only the flagged sentences, one Edit per sentence plus one Edit to "Not covered" for each
`unsupported` one. Apply the same change to the Findings sentence that states the same claim, and
leave every other sentence as it is. Fill the Verification table and choose the confidence level
again from the edited Answer.

Then deliver, in this order:

- the answer with its mechanism paragraph
- the key findings with their `[S#]`
- any disagreement between sources
- the verification outcomes, in one line
- the confidence level
- the path of `report.md`

Every number in the final message comes from the report.

## Errors

| Situation | Action |
|---|---|
| The Agent tool is unavailable | Run the collector's procedure for each sub-question in this session. In Step 5, re-read each claim's page yourself as `../../agents/scan-verifier.md` describes, and state in "Not covered" that the author ran the check |
| A collector returns an error or no file | Relaunch it once. After a second failure, record the sub-question in "Not covered" |
| WebSearch returns a budget-exceeded string | Record the shortfall in "Not covered" |
| The verifier returns an error or no `verdicts.md` | Relaunch it once. After a second failure, mark every sentence `unverified` and state it in "Not covered" |
| The verifier leaves a claim without a verdict | Mark the sentence `unverified` in the Verification table |

## Hand-over to research-team

Stop and offer research-team when any of these happens:

- Step 1 yields more than four sub-questions.
- Sources conflict on the figure the answer turns on.
- The stakes turn out to be money or an external publication.
- A sub-question the Answer turns on ends with fewer than two independent sources.
- A falsifier search returns evidence against the direction of the answer.
- A presupposition marked `verify` turns out to be false, and the sources in hand leave the
  correction open. A presupposition the sources correct in one line is a finding, written into
  the Answer.
- The verifier marks an Answer sentence `contradicted`.

Each of these is a question the scan's checks cannot resolve. Before offering, finish
`report.md` up to the step reached: remove any `contradicted` sentence from the Answer, fill the
Verification table if Step 5 ran, and name the trigger in "Not covered". Say what the scan
established and what it could not, then hand over. A scan extended toward a pipeline loses its speed and still
lacks the pipeline's checks.
