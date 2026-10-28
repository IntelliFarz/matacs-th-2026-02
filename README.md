# MATACS-TH-2026-02: AI Proposal Triage Design

- **Name:** Farzain Ahmed 
- **Reference:** MATACS-TH-2026-02
- **Time actually spent:** about 2 hours 15 minutes

## Summary
A design for an AI-assisted proposal triage tool. AI extracts evidence from proposals; plain code checks mandatory criteria, scores, ranks and explains; a human makes the final award decision. Criteria and weights come from admin config and are versioned. Sample result: A ranked 1, E ranked 2, B ineligible, C and D flagged for human review.

## Files
- [WRITEUP.md](WRITEUP.md): answers to the five questions and my assumptions
- [diagram/process-flow.png](diagram/process-flow.png): flow from upload to ranked output
- [PROMPTS.md](PROMPTS.md): the extraction prompt, exactly as sent
- [ranking.json](ranking.json): expected ranking of proposals A–E
- No `prototype/` folder: I prioritised the design.

## How I used AI tools
I used Claude to help plan the workflow, draft the files, and check the scoring arithmetic. I made the final design decisions myself and can explain each one.

**Where I disagreed with or corrected the AI:**
[WRITE THIS YOURSELF, ONLY IF IT REALLY HAPPENED. Describe one specific thing you changed or rejected and why, in 2 or 3 sentences. If you changed nothing, say so honestly and delete this placeholder. Do not copy an example.]

Points worth testing before you write the above (do not claim a correction unless you actually make one):
- Missing numeric evidence is flagged instead of scored 0.
- B is "ineligible", not "flagged".
- C is flagged (not ranked) because of the embedded instruction.
- Best-in-pool scoring formulas and the 25-points-per-item team checklist.

## Notes
- The brief and proposal text are not copied here; proposals are referred to by letter.
- No code, so no API keys or `.env.example` are needed.
