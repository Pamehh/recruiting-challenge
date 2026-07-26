# Prompt history — <Pamela>

> Raw, unedited transcript. False starts and bad prompts are signal, not embarrassment.
>
> This is the one artifact where AI content is *expected* — it's the conversation transcript itself. **Do not curate, summarize, or rewrite it.** A polished prompt history with no false starts is treated as evidence of curation, which lowers your AI-discipline score.

## Tool(s) used

- [ ] Claude Code
- [X] Claude.ai
- [ ] Cursor
- [ ] Copilot
- [ ] ChatGPT
- [ ] Aider
- [ ] Other:

## How much of the work was AI-assisted

- [X] Most of it
- [ ] About half
- [ ] Less than half — used it for specific tasks
- [ ] None at all

---

## Sessions

For each session, paste the raw transcript or the prompts you sent, in order. Don't trim.

### Session 1 — Understanding the challenge
**Prompt:**
```
i need you to help me understand this tecnical task this is the README "" then this is the "evaluation.md " give me the context. Be direct and brief
```
**What the model returned:**
A summary of the challenge: fork the repo, fix ≥3 quality issues, build one feature, submit 6 artifacts. Flagged that code can use AI freely but decision_log.md, validation_design.md, signoff.md, written_answers.md, and commit messages must be human-written, and mislabeling is treated as dishonesty.
**What I accepted, rejected, or refined:**
Accepted the summary as-is. This set the ground rules I held to for the rest of the session — I kept refusing to let Claude draft the human-written docs even under time pressure later on.
 
---
 
### Session 2 — Repo access
**Prompt:**
```
ok so this is the repository https://github.com/PlataformaT1/recruiting-challenge.git
```
**What the model returned:**
Cloned the repo directly (no upload needed). Proposed a workflow: Claude flags issues with reasoning, I decide what to fix and which feature, Claude helps implement with me reviewing diffs, human-written docs stay untouched by Claude.
**What I accepted, rejected, or refined:**
Accepted the workflow proposal.
 
---
 
### Session 3 — Full codebase review
**Prompt:**
```
yes, check all files and get a general view on the arquitecture, seams, logic before any decision
```
**What the model returned:**
Read every source file (auth.ts, orders-dal.ts, db.ts, all routes, seed.ts, server.ts, tests, docs, frontend). Live-verified three issues by running the app and using curl rather than just reading code:
- Cross-tenant IDOR: fetched an `m_bistro` order using an `m_acme` header, got 200 with the wrong merchant's data.
- Refund accounting bug: computed real seeded numbers by hand, showed reported revenue was ~14% higher than the correct net figure.
- Stored XSS: POSTed an HTML payload as `customer_email`, confirmed it round-tripped raw and traced it to an unescaped `innerHTML` template in app.js.
Also flagged (less deeply verified): missing input validation on POST body and `limit` param, `metrics.ts` bypassing the DAL with its own DB connection (already self-documented in the repo), missing composite index.
**What I accepted, rejected, or refined:**
Accepted all findings as real. I chose which 3 of the ~8 flagged issues to actually fix in the next session — I picked IDOR, refund bug, and XSS over the validation gaps and architecture smells, prioritizing severity and how well each taught a distinct lesson.
---
 
### Session 4 — Implementing the three fixes
**Prompt:**
```
pick 1, 4 and 3
```
**What the model returned:**
Implemented and verified all three fixes in Claude's own sandbox clone before handing over instructions:
1. IDOR — `getById(id)` → `getById(id, merchantId)`, required param, filtered by merchant_id. Added a regression test. Verified live before/after. Chose 404 over 403 deliberately (doesn't confirm the order exists under another tenant).
2. Refund sign bug — found and fixed the same bug in three places (revenue, avg-order-value, top-customers), not just one. Added a test asserting $10 sale + $4 refund nets to $6.
3. XSS — replaced `innerHTML` with `textContent`-based DOM construction. Verified with a temporary jsdom script simulating the exact malicious payload.
Gave full paste-in diffs and exact git commands, but explicitly gave raw facts for commit messages rather than the messages themselves.
**What I accepted, rejected, or refined:**
Accepted all three fixes and the 404-not-403 reasoning. I wrote my own commit messages from the raw facts given rather than using any drafted text.
---
 
### Session 5 — Clarifying the test file edits
**Prompt:**
```
there are only these two tests, idk what you mean "// Set DB_PATH before importing the db module...
[pasted the actual unmodified test file]
```
**What the model returned:**
Clarified this was the "before" file and walked through exactly which block to replace.
**What I accepted, rejected, or refined:**
Asked a follow-up (below) because I still wasn't sure of placement.
 
**Prompt:**
```
where do I add this? at the end? or what part exactly should I modify?
```
**What the model returned:**
Clarified the new test goes at the very end of the file, then gave the complete finished file to remove ambiguity.
**What I accepted, rejected, or refined:**
Accepted and pasted the full file rather than trying to hand-place the fragment, to avoid a mistake while tired.
 
---
 
### Session 6 — Choosing a feature
**Prompt:**
```
now lets get a feature "Feature menu — pick one... [pasted the three feature options]" which one is simplest? or more straight away?
```
**What the model returned:**
Direct complexity comparison: Feature A (CSV export) simplest, Feature C (search) medium and connected to the earlier composite-index finding, Feature B (webhooks) a genuinely different system and advised against given time constraints.
**What I accepted, rejected, or refined:**
Accepted the recommendation and picked Feature A explicitly because of time/health that day, not because it was the most impressive option — noted honestly in my decision log.
 
---
 
### Session 7 — Commit ordering
**Prompt:**
```
so what do I do first? commit the change and edit the files at the end?
```
**What the model returned:**
Clarified the correct order: edit → verify (typecheck, tests, manual click-through) → only then commit, never the reverse.
**What I accepted, rejected, or refined:**
Rejected this, I wasn't clear on my prompt and it missunderstood my question. 
 
---
 
### Session 8 — Understanding the docs files
**Prompt:**
```
what are they for?`docs/api.md`, `docs/architecture.md`
```
**What the model returned:**
Explained what each doc covers and why leaving them stale after behavior changes would make them actively wrong.
**What I accepted, rejected, or refined:**
Accepted the explanation; used it to decide whether to prioritize the docs update under time pressure later.
 
---
 
### Session 9 — Time pressure triage
**Prompt:**
```
no, the time is too close. I have to send the files now... oh no, I was too slow with my headache
```
**What the model returned:**
A triage order: verify tests are green, commit whatever's done even if partial, explicitly said it's fine to skip the docs update if out of time since the eval doc says quality over quantity, and that the 5 written artifacts matter more than a fully polished feature.
**What I accepted, rejected, or refined:**
Accepted the triage order and used it to decide what to finish vs. cut given real time constraints.



# What Claude got wrong
### The prompt:
```
there are only these two tests, idk what you mean "// Set DB_PATH before importing the db module...
[pasted the actual unmodified test file]
```
* **What Claude returned:** An instruction to rename the existing test and add this new test right after it," written as if the file already reflected earlier edits.
* **What was wrong with it (technically, not just stylistically):** it showed a state that didnt exist so I checked the code and the content was different.
* **How I caught it:** I checked the code and there was a mismatch, I showed that section again to claude to clarify if I was understanding well and to give it context. 
* **What I did instead:** I gave context and it gave me the answer now with the part of the code correct. 

### The prompt:
```
so what do I do first? commit the change and edit the files at the end?
```
** **What Claude returned:** Clarified the correct order: edit → verify (typecheck, tests, manual click-through) → only then commit, never the reverse.
* **What was wrong with it (technically, not just stylistically):** I wasn't clear on my prompt and it missunderstood my question, I skipped reprompting cause I was most certainly sure on what to do.
* **How I caught it:** it didnt answer the question I had in mind
* * **What I did instead:** I followed my knowledge and commited first.
 
