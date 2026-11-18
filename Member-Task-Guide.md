# Team 5: Member Task Guide (CA2, Scenario 5)

Based on **PRD v0.2** (18 Nov 2026). Each member's section runs in time order, from the first week to the demo. Section and item codes (F, R, S, E, G, N) refer to the PRD.

Timeline used in every section (from PRD Section 12): **this week → Lesson 13 → before Lesson 14 → Lesson 14 → term break week 1 → term break week 2 → 3 days before the demo → Lesson 17/18.** Check the real dates for Lesson 14 and the demo day and write them here: ______

---

## Rules that apply to everyone

- **Contracts:** Spokes talk to each other only through the contracts in PRD 11.1. Any change is agreed in the group chat **before** it is built.
- **No secrets (N6):** No API key, Google credential or token in the repo, report, slides, screenshots or chat. Refer to named n8n credentials only. Crop screenshots.
- **Data (N7):** Own or made-up data only. No real personal details.
- **Fixed eval prompts are never edited (N8).** Our own messages are added beside them.
- **Test alone first:** Every spoke must work on its own with test input before it is merged.
- **Own words (N10):** Report text, lesson sheets and the journal are written in your own words.
- **Ask early:** If you are blocked or behind, say so in the group chat the same day, so the work can be reassigned openly.

---

# Member 1: Hub and merge

**You own:** the hub agent, the 7-tool wiring, the system message, the merge of all workflows.
**Report:** 1.3, 2.1, 2.3. **Presentation:** intro (0–4 min) and wrap-up (22–24 min).

### This week
1. Run or join the meeting where the team agrees the interface contracts (PRD 11.1) and the nominated n8n account (Open question 2).
2. Make sure the repo skeleton is merged and everyone has access.
3. List the seven tools by their real names. Tools 4 and 5 are baseline tools from Lessons 11–12; put their real names in the PRD tool list and your system message.

### Lesson 13
4. Build the hub skeleton: Tools Agent, low temperature, window memory keyed to `sessionId`, one call-workflow tool per spoke. Use placeholder tools if a teammate's spoke is not ready.
5. Draft the system message: it must name every tool, forbid guessing a date or stating a number no tool returned, and contain rules R1–R10 (PRD Section 8).
6. Write the `notes_assistant` tool description. It must say when finance is meant instead. Include the practice-question and lesson-scope wording from R10.
7. Update the Member 1 PRD with R10 and the lesson-scope wording.

### Before Lesson 14
8. Merge everything on the nominated account. Follow the merge checklist in the Member 1 PRD (sub-workflow links, credentials).
9. Confirm the hub shows the finance tools working with tool 7 **not yet connected**, so M2 can run the "before" eval.

### Lesson 14 (clinic)
10. Connect `notes_assistant` as tool 7 with M4. Confirm the hub shows **exactly 7 tools**.
11. Get S-8 (finance + notes in one turn) working from the page, with M3 and M4.
12. Join the combined-turn session with M2, M3 and M4 for report 2.4.

### Term break, week 1
13. After the "after" eval run, fix routing problems. Test E4 and E5 (the deliberately confusing messages).
14. After **any** prompt change, ask M2 to rerun the whole eval set, not just the failing message.
15. Check that S-10 to S-12 route to `notes_assistant` and not to the Coach.

### Term break, week 2
16. Freeze the system message and tool description. Save an exact copy for report 2.3.
17. Write 1.3 (guardrails), 2.1 (architecture diagram), and 2.3 (**exact, unedited** tool description and system rules).
18. Write your own row in Section 5.

### 3 days before the demo, then Lesson 17/18
19. Prepare the intro (problem, S5, architecture) and the wrap-up (before vs after results, what we cut).
20. Join the full rehearsal. Check the submission list: report PDF, `.pptx`, final workflow JSON, repo by 23:59.

**Done when:** hub shows exactly 7 tools; S-1 to S-12 behave as in PRD Section 3; 1.3, 2.1, 2.3 written; Member 1 PRD updated.

---

# Member 2: Logger, Delete, seed documents, evaluation

**You own:** Logger, Delete Expense, `transactions` and `categories` tabs, seed documents and lesson notes (F20), eval runs (F21).
**Report:** 2.2 (your rows), 2.4, 2.5 (your part), 3.1, 3.2. **Presentation:** logging and deleting (4–8 min).

### This week
1. Agree the `txn_id` format with the team (Open question 3), for example `YYYYMMDD-amount-description-hash`.
2. Check the baseline column names against your Lesson 10 sheet before changing anything.

### Lesson 13
3. **Logger:** input `{ description, amount, category, date }`, output `{ status, txn_id, reason }`.
   - Same `txn_id` already in the sheet → `status: "duplicate"`, no new row.
   - Unreadable or missing amount, or category not in `categories` → `status: "refused"` with a reason.
   - Test alone with a normal entry, a duplicate, a missing amount and an unknown category.
4. **Delete Expense:** `action: "find"` returns matches only; `action: "confirm"` with one `txn_id` deletes that row only. Zero or several matches → nothing deleted. Test all three cases.
5. **Tabs:** keep `transactions` and `categories` columns unchanged. Add the `school fees` row to `categories` if the fee schedule needs it (F17); M3 adds the matching `budgets` row.
6. **Seed documents:** write the four made-up documents (Fee Schedule 2026, C389 Lesson Notes: Agents and Tools, RAG reading summary, Budgeting Tips).
7. **Lesson notes:** write 3–5 lesson files for one module, one markdown file per lesson, using the template (title, short summary, key points, key terms, in your own words). Leave one lesson **deliberately not stored**, so S-11 can show the refusal.
8. For each document, write 2–3 test questions with the correct answer.
9. Hand everything to M4 for review and ingestion. Confirm with the team that slide use is allowed for coursework (Open question 9).

### Before Lesson 14
10. Write `eval/expected.md` **before** sending any message: the expected tool(s) for every fixed-set message, E1–E5, and S-10 to S-12.
11. Run the **"before"** eval: merged finance build, tool 7 not connected, fixed set only.
12. Read the tools from the **execution**, not the reply. Screenshot each execution. Record results in `eval/`.

### Lesson 14, then term break week 1
13. Run the **"after"** eval: tool 7 connected, fixed set again, plus E1–E5 and S-10 to S-12.
14. Label every failure **prompt**, **tool** or **data**, with the cause. Hand the cases to the right owner.
15. After any prompt change, rerun the whole set.
16. Run the combined-turn session with M1, M3 and M4 and take the screenshot of **one execution** that calls both `analyse_spending` and `notes_assistant`.

### Term break, week 2
17. Write 2.4, 3.1 (method), 3.2 (results matrix: ID, prompt, expected tools, actual tools, pass/fail), 2.2 (your rows), 2.5 (your part), and your row in Section 5. Write the failures from your own components for 3.4.

### 3 days before the demo, then Lesson 17/18
18. Rehearse your segment: log an expense, show a duplicate refusal, delete with confirmation.

**Done when:** Logger and Delete work alone and merged; seed documents and lesson notes delivered; `eval/expected.md` written before testing; before and after runs recorded with screenshots; 2.4, 3.1, 3.2 written.

---

# Member 3: Analyst, Coach, Daily Digest, budgets

**You own:** Analyst (code only, no language model), Coach, Daily Digest, `budgets` tab.
**Report:** 2.2 (your rows), 3.3. **Presentation:** Analyst, Coach, digest (8–12 min).

### This week
1. Check baseline column names for `budgets` and the digest tab against your Lesson 10 sheet.
2. Agree the Analyst output shape with M1 and M4 (the Coach and the combined turn depend on it).

### Lesson 13
3. **Analyst:** Code nodes only. Input `{ month, category? }`. Output `{ month, totals[{category, spent, budget, remaining}], skipped_repeats, skipped_unreadable }`. Repeats and unreadable amounts are skipped **and counted**. Test alone with sample rows.
4. **Coach:** receives Analyst output, returns `{ advice }` in **2–3 sentences**, using only figures the Analyst returned. No new numbers.
5. **`budgets` tab:** one row per category, including the `school fees` row (F17).
6. **Daily Digest:** schedule trigger **once a day** (never every minute, N5) → Analyst → one row in the digest tab. Check the schedule interval before activating.
7. Make sure each spoke works alone with test input.

### Before Lesson 14
8. Support M1 in the merge. Check that the Analyst and Coach still work through the hub.
9. Help M2 with the "before" eval for finance messages (S-2, S-3).

### Lesson 14, then term break week 1
10. Join the combined-turn session: S-8 needs your Analyst to return the `school fees` figure alongside M4's fee schedule answer.
11. After the "after" eval, check finance messages for regressions (G4: finance scores the same or better). Fix any problems in your spokes.

### Term break, week 2
12. Write 3.3 (before vs after table) using M2's results: fixed eval set passed, messages that called a tool when none was needed, extension messages passed.
13. Write 2.2 (your rows), the failures from your own components for 3.4, and your row in Section 5.

### 3 days before the demo, then Lesson 17/18
14. Rehearse: ask for spending, show the Coach advice, show the digest row.

**Done when:** Analyst has no language model; Coach uses only Analyst figures; digest runs once a day and writes one row; `school fees` budget row exists; 3.3 written.

---

# Member 4: Pinecone, ingestion, Notes spoke, practice questions

**You own:** Pinecone index, embeddings, `documents` tab, ingestion with the practice Q&A step, Notes spoke (citation, refusal, practice branch).
**Report:** 1.1, 1.2, 1.4. **Presentation:** cited answers, practice questions, refusal, combined message (12–17 min).

### This week
1. Decide the embeddings model (Open question 10). It sets the Pinecone index dimension.
2. Confirm with the team which module and which 3–5 lessons go in (Open question 9).
3. Create the Pinecone account (use the team's nominated account or a shared team email). Store the API key as a **named n8n credential** only.
4. Check the current free-plan limits on Pinecone's pricing page.

### Lesson 13
5. **Create the index:** dimension matching the embeddings model, cosine similarity, one namespace for the notes.
6. **`documents` tab:** headers in row 1: `doc_id`, `title`, `doc_type`, `source`, `chunk_count`, `practice_count`, `ingested_at`, `summary`, `status`.
7. **Ingestion v1 (notes only):** read text → delete old vectors for the same `doc_id` → split (about 500 tokens, 50 overlap) → embed → insert into Pinecone with metadata `{ doc_id, title, doc_type, type: note, source_chunk }` → append one `documents` row. Chunk ids: `{doc_id}-c{n}`.
8. **Test retrieval alone:** query the index directly and check the right chunks and titles come back before connecting anything to the hub.
9. **Notes spoke:** copy from the Coach. Contract `{ action, question?, doc_title? }` → `{ matched, answer, sources[] }` or `{ documents[] }`.
   - `list`: read `documents` where `status = active`.
   - `ask`: retrieve top-k (about 4), filtered by `doc_title` if given.
   - No chunk passes the similarity threshold → `{ matched: false, answer: "", sources: [] }` with **no model call**.
   - Otherwise call the model with the instruction "answer only from these excerpts; if they do not contain the answer, say so".
10. Review M2's seed documents and lesson notes (titles, clean text, one lesson per file, no personal details) and ingest them.
11. Make sure the spoke works alone with test input.

### Before Lesson 14 and Lesson 14
12. **Practice Q&A step (F18):** after the core is stable, add the ingestion step that writes 2–3 questions with short answers per chunk, **only from that chunk** (nothing for chunks with nothing testable). Insert them as `type: practice` vectors, ids `{doc_id}-p{n}`, with `source_chunk`. Record `practice_count` in `documents`.
13. **Practice branch (F19):** a Code node checks whether the question asks for practice questions. If yes, retrieve only `type: practice` chunks (k about 8, filtered by `doc_title` if given), pick 3 at random, format with answers and source titles. No model call at ask time. No match → the refusal.
14. **Cutoff rule:** if this is not stable by the team's agreed cutoff date, drop it and log it as a scope decision in report 2.5.
15. **Spot-check (F22):** check every generated question against its source chunk before the "after" eval run. Record the result in `eval/`.
16. Connect the spoke as tool 7 with M1. Get S-8 working from the page with M1 and M3.

### Term break, week 1
17. Tune the similarity threshold using the trap questions (S-7, S-11, E2): too strict refuses valid questions, too loose lets answers drift.
18. Look at failures from your spoke after the "after" eval. Fix them, labelling each as prompt, tool or data. Rerun the whole set after any change.

### Term break, week 2
19. Write 1.1 (extension summary with example input and output), 1.2 (`documents` tab columns, data types, example values), 1.4 (why "a tool, with retrieval" over other patterns), the failures from your own component for 3.4, and your row in Section 5.

### 3 days before the demo, then Lesson 17/18
20. Make a demo checklist: if you ever fall back to the in-memory store, re-ingest everything first.
21. Rehearse: a cited answer, a lesson-scoped question, practice questions, a refusal, and the combined message.
22. **At the end of the module:** delete the Pinecone index (N7).

**Done when:** index holds notes and practice Q&A with no duplicates after re-ingestion; `documents` tab has one row per document; S-5, S-6, S-7, S-10, S-11, S-12 behave as in PRD Section 3; spot-check recorded; 1.1, 1.2, 1.4 written.

---

# Member 5: Chat page, deployment, turn log, repo

**You own:** chat page, allowed origin, turn log branch, `agent_log`, repo hygiene.
**Report:** 4.1, 4.2, 4.3. **Presentation:** live deployment demo (17–22 min).

### This week
1. Check the repo skeleton and README folder layout (`site/`, `workflows/final/`, `eval/`).
2. Decide where the page is hosted (for example GitHub Pages) and write down the **exact address** (Open question 6).

### Lesson 13
3. **`agent_log` tab:** headers in row 1 for `{ logged_at, session_id, user_message }`.
4. **Chat page:** `site/index.html` embedding the n8n chat widget pointed at the hub's public chat URL. Serve it from the public address.
5. **Allowed origin:** set it to that exact address (scheme + host, no trailing path), **never `*`** (N4).
6. **Turn log branch:** a branch off the Chat Trigger, **parallel to the agent, never after it** (N3). It appends one row per turn to `agent_log`.

### Before Lesson 14
7. After M1's merge, publish the hub with the public chat trigger.
8. Send S-2 from the page. Check the execution and that one `agent_log` row was written per turn.
9. Search every export for keys or tokens **before every push** (N6). Crop screenshots.

### Lesson 14, then term break week 1
10. Run the five failure cases from the page (PRD 9.4) and record each result with a screenshot:
    1. Normal question (S-2).
    2. Workflow switched off: deactivate the hub, send S-2, record the widget's error, reactivate.
    3. Origin not allowed: open the page from another address (for example `localhost`) and send a message.
    4. Tool the agent doesn't have: "Book me a flight to Bangkok".
    5. Empty or whitespace message.
11. Measure response time (PRD 9.5): **cold** = first message after at least 10 minutes idle; **warm** = the next message straight after. Use a stopwatch from send to first reply, plus the n8n execution duration. Three tries each, report the median.

### Term break, week 2
12. Export the final workflow JSON from the nominated account into `workflows/final/`. Check it has no secrets. Check `site/` files are in the repo.
13. Write 4.1 (page URL, allowed origin, daily schedule configuration, UI screenshot), 4.2 (five failure tests), 4.3 (latency table), the failures from your own components for 3.4, and your row in Section 5.

### 3 days before the demo, then Lesson 17/18
14. Make the **backup recording** of a full run.
15. On demo day: have the page already open and warmed up. Tell the audience up front that the first turn is slow. Show at least one refusal.
16. Check the repo is final by 23:59: workflows, site files, no secrets.

**Done when:** page answers from its own origin; one log row per turn; five failure cases and cold/warm times recorded; final JSON and site files in the repo with no secrets; 4.1, 4.2, 4.3 written.

---

# Proposed changes not yet in PRD v0.2

These are suggestions from our planning chat. The member sections above follow v0.2 as written. Agree them as a team before applying:

- **Finance sample data owner:** M3 prepares sample transactions (food, transport, school fees, a "Grab ride yesterday" for S-4, one duplicate) and the matching `budgets` rows. M2 and M3 agree the fee schedule's instalment amount and due dates so S-8 has a checkable figure.
- **Lesson notes split:** M2 and M3 write the lesson files between them, since M2 also carries the eval work and report 2.4, 3.1, 3.2. M4 reviews.
- **Other seed documents:** M5 prepares the RAG reading summary, budgeting tips and one out-of-scope trap file for refusal tests.
- **Spot-check split:** M2 reviews half of the generated practice Q&A, M4 reviews the other half and makes the final call.
- **Random-pick Code node:** M3 builds the node that picks 3 practice questions and formats them; M4 keeps retrieval and detection.
- **Data dates:** make the sample transactions fall in the month you run the evals, so "this month" and "yesterday" messages work.
- **Data freeze:** stop changing documents once the "before" run starts.
