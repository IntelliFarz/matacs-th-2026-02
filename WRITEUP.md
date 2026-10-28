# Write-up: AI Proposal Triage Design

**Principle:** AI reads, code decides, a human awards. Proposals are referred to by letter (A–E).

## Assumptions
- **Ineligible** = a mandatory criterion clearly fails. **Flagged** = evidence is missing, ambiguous, conflicting or suspicious, so a person must resolve it before the proposal is ranked.
- Each criterion in the admin config has an `on_missing` rule. Numeric criteria (price, timeline, experience, warranty) use `flag`: a missing value is never guessed or silently scored 0. Checklist items (e.g. safety plan) use `zero`: if the proposal does not mention it, it earns no points.
- Price above the budget ceiling, or a timeline above 16 weeks, makes a proposal ineligible (configurable). No sample exceeds either.
- "Last 5 years" means from the calendar year five years before upload (2021 onward).
- The brief gives no scoring formulas, so I defined them (Q2).
- The samples have no page numbers; the production design records them.

## 1. AI vs. rules
**AI (one step):** extraction. It reads messy PDF text and returns each fact with a page number and a short quote, or `null` if absent. Wording varies too much for simple rules.

**Code/rules:** intake, loading criteria, evidence check, mandatory gate, scoring, ranking, tie-breaks, reason text and audit log.

**Why the line is here:** anything that changes the order of bidders must be repeatable and testable, so it is code. The AI never sees the weights or other proposals, so it cannot be steered by them.

## 2. Consistency and explainability
- **Repeatable:** extraction runs at temperature 0 with a strict JSON schema and is cached by file hash + prompt version + model version. Re-ranking reuses the cache, and scoring is pure code, so the same proposals give the same ranking.
- **Formulas** (all from config): for quantity criteria, score = 100 × value ÷ best value in the pool (price and timeline invert this: lowest ÷ own). Warranty averages panel and workmanship scores. Team and approach has four yes/no items (named lead, crew, safety plan, monitoring plan), 25 points each. The pool is every proposal that passed the mandatory gate, so flagging one does not change others' scores.
- **Tie-break:** higher Experience score, then lower price, else shown as a tie for the reviewer.
- **Explainable:** each result shows the per-criterion score, weighted points, quote, page and rule applied. Reason text comes from templates over stored scores, not from AI, so it cannot invent a justification.
- **Sample result:** A 98.01, E 84.80. B ineligible. C and D flagged (provisional 77.06 and 69.82; see `ranking.json`).

## 3. Human in the loop
1. Reviewer confirms uploads and category.
2. Flagged proposals: reviewer confirms a value, enters one with a note, or excludes the proposal. D needs a schedule. C needs the injected instruction reviewed and the install period checked.
3. Ineligible proposals: evidence is shown. A reviewer may override with a written reason (for B, if the bidder licence is later proven).
4. Final: the reviewer sees the ranked list and makes the award decision. The tool never awards.

Overrides re-run through the same code and display as "overridden by [user]".

**Audit log (append-only):** user, time, file hashes, criteria version, prompt and model versions, raw AI output, evidence-check results, scores, flags, every override with its reason, and the final decision.

## 4. Changing criteria
- Criteria sets are versioned and immutable. An edit creates a new version, and each score records the version used.
- Old results stay viewable, labelled "scored under v1", and the admin sees how many are affected.
- **Weight or formula change:** rescore from cached extraction. No AI cost.
- **New or changed criterion:** re-extract only that field.
- **Mandatory change:** show the reviewer a before/after diff, since eligibility may change.
- Earlier overrides do not carry over automatically; the reviewer re-confirms. A recorded award decision is never rewritten.

## 5. Risks

| Risk | Safeguard |
|---|---|
| Wrong extraction | Every value needs a quote. Code checks the quote appears in the source text. Conflicts become flags. Humans review flags. |
| Made-up facts | Prompt allows `missing`; schema rejects values without evidence; unsupported values are discarded and flagged. |
| Prompt injection (C) | Text is wrapped as untrusted data. The AI has no tools and returns only JSON. Suspicious instructions are recorded and the proposal is flagged. Scores come from code, so the AI cannot award marks. |
| Bias | Only configured criteria are scored. Company name, size and writing style are not used. Reviewers can see the evidence for every score. Poor PDF text quality (e.g. a scanned file) is flagged rather than penalised. |
| Cost | One extraction call per proposal, cached. Re-ranking and criteria changes mostly cost nothing. A page limit and a small model cap spend. |
| Privacy and security | Role-based access, encrypted storage, retention limits, no personal data in logs, API keys in environment variables, vendor set not to train on the data. |
| Over-trust | Output is labelled "triage, not decision". The reviewer must acknowledge flags before the final screen. |
| Formula limits | Best-in-pool scoring is relative, so adding a bid can change other scores. These are rescored and logged. Formulas are shown to admins. |
