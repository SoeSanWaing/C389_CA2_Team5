# PRD: Personal Productivity Agent with Notes and Reading Assistant (S5)

| | |
|---|---|
| **Module** | C389 AI Solution Design and Implementation, CA2 (Project 2) |
| **Scenario** | S5 Notes and Reading Assistant. Pattern: **a tool, with retrieval** |
| **Team** | Team 5 (five members, see Section 11) |
| **Repo** | https://github.com/SoeSanWaing/C389_CA2_Team5 |
| **Status** | Draft v0.2, 18 Nov 2026 (changes from v0.1 are listed in Section 15) |
| **Key dates** | Merged on the nominated account **before Lesson 14**. Demo, report (PDF), slides (.pptx) and final workflow JSON due 23:59 on the team's presentation day (Lesson 17 or 18) |

Related: [Member 1 PRD: Hub Agent and Merged Build](PRD-Member1-Hub-Agent.md) goes into more detail on the hub, the system message and the merge.

---

## 1. Summary

We extend the Finance Tracker baseline (built in Lessons 9 to 13) with a second area: the user's own **documents and notes**. The finished agent:

- still logs, analyses, advises on and deletes spending, exactly as the baseline did;
- stores the user's documents (notes, readings, a fee schedule) in a vector store plus a `documents` tab;
- answers questions **only from those documents**, names the document each answer came from, and says plainly when nothing matches;
- can answer a document question **beside** a spending figure in the same turn (for example, a fee schedule next to what was spent on fees);
- stores lesson notes for one module as one document per lesson, can answer **scoped to one lesson**, and gives **cited practice questions** from a bank generated when the notes are ingested.

Everything runs from one chat page that the team serves, through one hub agent with seven tools.

### One-line statements (for report 1.1)
- **Scenario:** a notes and reading assistant that answers only from the user's own stored documents and cites them.
- **Extension:** a seventh hub tool, `notes_assistant`, backed by a retrieval spoke, a vector store and a `documents` tab, sitting beside the finance tools.
- **Unique feature (our own):** a lesson-scoped revision assistant. Lesson notes are ingested with practice Q&A generated and tagged at ingestion time, so a user can ask for practice questions on a lesson and get them with their source lesson, or the plain refusal if the topic is not stored.

---

## 2. Goals and non-goals

### Goals
| # | Goal | Measure |
|---|---|---|
| G1 | The hub picks the right tool between finance and notes | Fixed eval set and our own messages pass, with tools read from the execution |
| G2 | Notes answers are grounded | Every notes answer names a source document; nothing is answered from general knowledge |
| G3 | Clear refusal when nothing matches | The exact refusal wording appears whenever retrieval finds no match |
| G4 | Finance keeps working | Fixed eval set scores the same or better after the extension |
| G5 | Finance and notes work in one turn | The combined message calls both tools and answers both parts |
| G6 | Deployed and live | Page answers from its own origin; five failure cases recorded from the page; cold and warm times recorded |
| G7 | Practice questions are grounded | Every practice question comes from a stored lesson, names its source, and was spot-checked against that lesson; a topic with no stored lesson gets the refusal |

### Non-goals
- Uploading documents through the chat. Documents are added through a separate ingestion workflow.
- Answering general-knowledge questions.
- Multi-user accounts. One user, one sheet, one vector store.
- Any document that holds other people's personal details (brief §7, Data and privacy).
- A separate quiz tool or action. Practice requests go through the existing `ask` action.
- Marking the user's answers, summarising whole decks, or generating study notes on demand.
- Generating practice questions at ask time. The bank is generated at ingestion (see Open question 8).

---

## 3. Users and core scenarios

**User:** a polytechnic student who tracks their own spending and keeps notes, readings and a (made-up) school fee schedule.

| # | User says | Expected tool(s) | Expected result |
|---|---|---|---|
| S-1 | "Spent $4.50 on chicken rice at lunch" | `log_expense` | One checked row in `transactions` |
| S-2 | "How much have I spent on food this month?" | `analyse_spending` | Figure equal to the Analyst's output |
| S-3 | "How can I cut down on spending?" | `get_advice` | 2–3 sentences using Analyst figures only |
| S-4 | "Delete the Grab ride from yesterday" | `delete_expense` (find) | Names one entry and asks to confirm; deletes only on "yes" |
| S-5 | "What does my fee schedule say about late payment?" | `notes_assistant` (ask) | Answer plus "Source: *Fee Schedule 2026 (sample)*" |
| S-6 | "What documents do I have stored?" | `notes_assistant` (list) | List of document titles |
| S-7 | "What is the population of Japan?" | `notes_assistant` (ask) → no match | "I couldn't find that in your stored documents." |
| S-8 | "How much have I spent on school fees this month, and when is the next instalment due according to my fee schedule?" | `analyse_spending` + `notes_assistant` | Both parts answered, each with its own source |
| S-9 | "Hi!" / "Thanks" | none | Short reply, no tool |
| S-10 | "Give me 3 practice questions on <a topic from a stored lesson>" | `notes_assistant` (ask) | 3 questions with answers, each naming its source lesson |
| S-11 | "Give me 3 practice questions on <a topic in no stored lesson>" | `notes_assistant` (ask) → no match | "I couldn't find that in your stored documents." |
| S-12 | "Using only Lesson <N>, what does it say about <topic>?" | `notes_assistant` (ask, `doc_title` set) | Answer from that lesson only, with its source |

---

## 4. Requirements

### 4.1 Functional requirements

**Baseline (must keep working)**
| ID | Requirement | Owner |
|---|---|---|
| F1 | **Hub**: one chat entry point, model, memory keyed to `sessionId`, system prompt that names every tool and forbids guessing a date or stating a number no tool returned | M1 |
| F2 | **Logger**: one sentence becomes one checked row in `transactions`, with the team's `txn_id` rule (duplicate check), or a refusal with a reason | M2 |
| F3 | **Analyst**: totals per category against budget; repeats and unreadable amounts skipped **and counted**; **no language model** | M3 |
| F4 | **Coach**: advice in 2–3 sentences, using only figures the Analyst returned | M3 |
| F5 | **delete_expense**: finds one entry, says it back, deletes only after the user confirms that entry | M2 |
| F6 | **Chat page**: hub published with a public chat trigger, widget embedded in a page we serve, allowed origin = that page's address (never `*`) | M5 |
| F7 | **Turn log**: every turn logged on a branch **beside** the agent into `agent_log` | M5 |
| F8 | **Daily digest**: one summary row a day, schedule runs **once a day** | M3 |

**S5 extension**
| ID | Requirement | Owner |
|---|---|---|
| F9 | `documents` tab in the team spreadsheet, headers in row 1 (Section 5.2) | M4 |
| F10 | Vector store holding chunked, embedded documents, each chunk tagged with its document title and id | M4 |
| F11 | Ingestion workflow: adds a document to the vector store and writes one row to `documents` | M4 |
| F12 | Notes spoke, copied from the Coach, with trigger input `action` plus `question` / `doc_title` | M4 |
| F13 | Hub tool 7, `notes_assistant`, whose description says when finance is meant instead | M1 |
| F14 | One system message rule for when to call `notes_assistant` rather than a finance tool; it also covers practice-question requests and lesson-scoped questions (R10) | M1 |
| F15 | Answers name the source document; no match gives the exact refusal wording | M4 (spoke), M1 (hub rule) |
| F16 | The combined finance + notes message (S-8) works end to end from the chat page | M1, M3, M4 |
| F17 | If the fee schedule creates a new spending type, add **one row** to `categories` (M2) **and the matching row to `budgets`** (M3), for example `school fees`. Nothing else in the finance tabs changes | M2, M3 |
| F18 | Ingestion generates practice Q&A: after splitting, a model step writes 2–3 questions with short answers per chunk, **only from that chunk** (nothing is written for a chunk with nothing testable), and stores each as an extra vector tagged `type: practice` with `doc_id`, `title` and `source_chunk` | M4 |
| F19 | Notes spoke: if the question asks for practice questions, retrieve only `type: practice` chunks (filtered by `doc_title` if given) and return 3 picked at random from the top matches, with answers and source titles; nothing passes the threshold → refusal. No new action and no change to the Notes contract | M4 |
| F20 | Seed documents: the four seed documents plus lesson notes for one module (3–5 lessons, one markdown file each), with test questions and expected answers | M2 (writes), M4 (checks chunking) |
| F21 | Eval: `eval/expected.md`, before and after runs, execution screenshots, results matrix | M2 |
| F22 | Spot-check every generated practice question against its source chunk before the "after" eval run; record the result in `eval/` | M4 |

### 4.2 Non-functional requirements and constraints
| ID | Constraint |
|---|---|
| N1 | **At most seven tools** on the hub |
| N2 | Analyst has **no** language model |
| N3 | Log branch sits beside the agent, never after it |
| N4 | Allowed origin is our own page's address, never `*` |
| N5 | Scheduled work runs once a day at most (shared class model quota) |
| N6 | No API key, Google credential or token in the repo, report, slides, screenshots or chat. Name credentials, never paste them |
| N7 | Own or made-up data only (lesson notes are our own notes based on the slides; confirm slide use is allowed for coursework). Delete the data, including the Pinecone index, at the end of the module |
| N8 | Fixed eval prompts are never edited. Our own messages are added beside them |
| N9 | Report: CA2 template, at most 10 pages excluding appendix, repo link on page 1 |
| N10 | Lesson sheets, journal and report prose in our own words; AI for reference and exploration only |

---

## 5. Data model

One Google spreadsheet on the nominated account. **Baseline tabs keep their names and columns.** Check the baseline column lists below against your Lesson 10 sheet before building; they are a guide, not a change.

### 5.1 Baseline tabs (unchanged)
| Tab | Purpose | Owner |
|---|---|---|
| `transactions` | One row per expense, including `txn_id` | M2 |
| `categories` | Allowed categories (one new row only if S5 needs it, see F17) | M2 |
| `budgets` | Monthly budget per category | M3 |
| digest tab (baseline name) | One summary row per day | M3 |
| `agent_log` | One row per turn | M5 |

### 5.2 New tab: `documents` (for report 1.2)
| Column | What it holds | Example |
|---|---|---|
| `doc_id` | Unique id, set at ingestion | `DOC-003` |
| `title` | Title shown in citations | `Fee Schedule 2026 (sample)` |
| `doc_type` | `note`, `reading` or `fee_schedule` | `fee_schedule` |
| `source` | Where the text came from (file name or "pasted") | `fee_schedule_2026.txt` |
| `chunk_count` | Note chunks written to the vector store | `6` |
| `practice_count` | Practice Q&A chunks generated at ingestion (0 for documents that get none) | `12` |
| `ingested_at` | Date and time of ingestion (from the workflow, not guessed) | `2026-11-20 14:05` |
| `summary` | One-line description for `list` | `Semester fees, instalment dates, late fee` |
| `status` | `active` or `removed` | `active` |

### 5.3 Vector store
- **Decision made (M4): Pinecone**, free Starter plan. Reasons: data persists between restarts, native n8n node, and our data is tiny (the seed documents plus 3–5 lessons). Fallback if credentials fail on demo day: n8n in-memory store plus a "re-ingest" step on the demo checklist.
- Starter limits as found in Nov 2026 (verify on Pinecone's pricing page before building): 2 GB storage, 2M write units and 1M read units a month, 5 indexes, one project, AWS us-east-1 only, community support. Going over the quota is blocked with an error, not billed, so no test loops.
- Index set-up (M4, before ingestion): the dimension must match the embeddings model, cosine similarity, one namespace for the notes. Changing the embeddings model later means recreating the index.
- Each chunk stores the text plus metadata `{ doc_id, title, doc_type, type, source_chunk }`, so retrieval can always name its source. `type` is `note` for lesson text and `practice` for generated Q&A.
- Chunk ids are fixed: `{doc_id}-c{n}` for notes, `{doc_id}-p{n}` for practice Q&A. Re-ingesting a document first deletes its old vectors by `doc_id`, so no duplicates build up.
- Chunking: about 500 tokens with about 50 tokens overlap (tune during testing). Lesson files are short, so smaller chunks may work better.
- The Pinecone key lives only in a named n8n credential on the nominated account (N6). The index is deleted at the end of the module (N7).

### 5.4 Seed documents (made-up)
| Title | Type | Used for |
|---|---|---|
| Fee Schedule 2026 (sample) | fee_schedule | Combined finance + notes message (S-8) |
| C389 Lesson Notes: Agents and Tools | note | Plain notes questions |
| Reading: Retrieval-Augmented Generation (summary) | reading | Citation across documents |
| Budgeting Tips (personal notes) | note | Tests that notes advice is not confused with the Coach |
| Lesson notes for one module (3–5 files, one document per lesson, titled like "<Module> Lesson 3: <topic>") | note | Lesson-scoped questions (S-12) and the practice-question bank (S-10) |

**Lesson file template (M2 writes, M4 checks):** one markdown file per lesson with a title, a short summary, key points and key terms, in our own words. No practice questions are written by hand; they are generated at ingestion (F18). Leave one lesson of the module **deliberately not stored**, so S-11 can show the refusal.

Also write, for each document, 2–3 test questions with the correct answer (for `eval/expected.md`).

---

## 6. Architecture (for report 2.1)

```mermaid
flowchart LR
  P["Chat page (M5)<br/>site/index.html, allowed origin"] --> CT["Chat Trigger (public)"]
  CT --> HUB["Hub AI Agent (M1)<br/>model + memory by sessionId"]
  CT --> LOG["Turn log branch (M5)<br/>beside the agent"]
  LOG --> T_LOG[("agent_log")]

  HUB --> L["1 log_expense → Logger (M2)"]
  HUB --> A["2 analyse_spending → Analyst, code only (M3)"]
  HUB --> C["3 get_advice → Coach (M3)"]
  HUB --> B4["4 baseline tool"]
  HUB --> B5["5 baseline tool"]
  HUB --> D["6 delete_expense (M2)"]
  HUB ==> N["7 notes_assistant → Notes spoke (M4) ★<br/>copied from the Coach"]

  L --> T_TX[("transactions")]
  L --> T_CAT[("categories")]
  D --> T_TX
  A --> T_TX
  A --> T_BUD[("budgets")]
  C --> A
  N ==> VS[("Vector store: Pinecone ★")]
  N ==> T_DOC[("documents ★")]

  ING["Ingestion workflow (M4) ★<br/>manual trigger + practice Q&A step"] ==> VS
  ING ==> T_DOC
  SCH["Daily Digest (M3)<br/>schedule, once a day"] --> A
  SCH --> T_DIG[("digest tab")]
```
★ = S5 parts.

> Tools 4 and 5 are the remaining baseline tools from Lessons 11–12. Put their real names in before finalising the system message.

### 6.1 Workflow inventory
| Workflow | Type | Owner | Starts from |
|---|---|---|---|
| Hub | Chat trigger + AI agent | M1 | Baseline hub |
| Logger | Sub-workflow | M2 | Baseline |
| Delete Expense | Sub-workflow | M2 | Lesson 13 |
| Analyst | Sub-workflow, code only | M3 | Baseline |
| Coach | Sub-workflow | M3 | Baseline |
| Daily Digest | Schedule, once a day | M3 | Baseline |
| Turn log | Branch on the hub | M5 | Baseline |
| **Notes spoke** ★ | Sub-workflow, with a practice-request branch | M4 | **Copied from the Coach** |
| **Ingestion** ★ | Manual trigger, includes the practice Q&A step | M4 | New |

---

## 7. Component specs

### 7.1 Hub (M1)
Full spec in [PRD-Member1-Hub-Agent.md](PRD-Member1-Hub-Agent.md). In short: Tools Agent, low temperature, window memory keyed to `sessionId`, seven call-workflow tools, system message with tool list and rules (Section 8).

### 7.2 Logger and Delete Expense (M2)
- Input `{ description, amount, category, date }`. `date` comes from the date tool via the hub.
- **`txn_id` rule** (team to fix the format, for example `YYYYMMDD-amount-description-hash`). The same `txn_id` already in the sheet → `status: "duplicate"`, no new row.
- Unreadable or missing amount, or a category not in `categories` → `status: "refused"` with a reason.
- Delete: `action: "find"` returns matches only; `action: "confirm"` with one `txn_id` deletes that row only. Zero or several matches → nothing deleted.

### 7.3 Analyst, Coach, Daily Digest (M3)
- Analyst: Code node(s) only. Output `{ month, totals[{category, spent, budget, remaining}], skipped_repeats, skipped_unreadable }`.
- Coach: receives Analyst output, returns `{ advice }` in 2–3 sentences, no new numbers.
- Digest: schedule trigger once a day → Analyst → one row in the digest tab.

### 7.4 Notes spoke and ingestion (M4)
**Notes spoke** (copied from the Coach):
1. Trigger input `{ action, question?, doc_title? }`.
2. `list` → read `documents` where `status = active` → return titles and summaries.
3. `ask` → a Code node checks whether the question asks for practice questions (words such as "practice questions", "quiz me", "test me"). If **yes**, retrieve only `type: practice` chunks (k ≈ 8), filtered by `doc_title` if given. If **no**, retrieve top-k `type: note` chunks (k ≈ 4), filtered by `doc_title` if given.
4. If no chunk passes the similarity threshold → return `{ matched: false, answer: "", sources: [] }`. **No model call**, so nothing can be made up.
5. Otherwise call the model with: the chunks, their titles, and the instruction "answer only from these excerpts; if they do not contain the answer, say so". Return `{ matched: true, answer, sources: [titles] }`.
6. **Practice path:** no model call at ask time. A Code node picks 3 of the retrieved Q&A at random and formats them with their answers and source titles, so the questions stay grounded and a repeat request can return different ones.

**Ingestion** (manual trigger): read text (Drive file or pasted) → delete any old vectors for the same `doc_id` → split → embed → insert into Pinecone with metadata → **practice Q&A step:** for each chunk, call the model with "write 2–3 questions with short answers using only this text; if it has nothing testable, return nothing" → embed and insert the results as `type: practice` vectors → append one row to `documents` (including `chunk_count` and `practice_count`). A human spot-checks the generated Q&A (F22) before the "after" eval run.

### 7.5 Chat page, turn log, repo (M5)
- `site/index.html` embeds the n8n chat widget pointing at the hub's public chat URL. Served from a public address (for example GitHub Pages).
- Chat trigger allowed origin = that exact address (scheme + host, no trailing path).
- Turn log: a branch off the Chat Trigger, parallel to the agent, appending `{ logged_at, session_id, user_message }` to `agent_log`.
- Repo hygiene: folder layout per README, no secrets, final JSON exported from the nominated account.

---

## 8. Agent rules (for report 1.3 and 2.3)

| Rule | Exact condition |
|---|---|
| R1 Route | Call `notes_assistant` when the user asks what a document, note, reading or fee schedule **says**. Call a finance tool when they ask what they **spent, logged or budgeted**, or want to log or delete an expense |
| R2 Both | If a message needs both, call both and answer each part with its own source |
| R3 Numbers | Never state a number no tool returned |
| R4 Dates | Never guess a date; use the date tool |
| R5 Cite or refuse | Notes answers name the source document. No match → *"I couldn't find that in your stored documents."* No general knowledge |
| R6 Confirm delete | Find first, name the one entry, wait for an explicit yes next turn. Several matches → ask which, delete nothing |
| R7 Unreadable amount | Don't log; ask for the amount |
| R8 No tool needed | Greetings, thanks and questions about this chat need no tool |
| R9 Out of scope | Say plainly it can't be done and list what the assistant can do |
| R10 Practice questions | Requests for practice questions on the notes, or questions limited to one lesson, go to `notes_assistant` (`ask`, with `doc_title` for a lesson). Never write practice questions from general knowledge. No stored lesson on the topic → the refusal in R5 |

The draft system message and the `notes_assistant` tool description are in the Member 1 PRD, Section 5. The report pastes the **final** text from the build, word for word. Member 1 adds R10 and the lesson-scope wording to the tool description in that file.

---

## 9. Evaluation plan (for report section 3)

### 9.1 Method
1. Write the expected tool(s) for every message **before** sending it, in `eval/expected.md`.
2. **Before run:** merged finance build, tool 7 not connected. Run the fixed eval set from Lesson 13.
3. **After run:** tool 7 connected. Run the fixed eval set again, plus our own S5 messages.
4. Read the tools from the **execution**, not the reply. Screenshot each execution.
5. Each failure is labelled **prompt**, **tool** or **data**, with its cause.
6. After any prompt change, rerun the whole set, not just the failing message.

### 9.2 Our own messages (3–5, added beside the fixed set)
| ID | Message | Expected tools |
|---|---|---|
| E1 | "Give me 3 practice questions on <a topic from a stored lesson>" | `notes_assistant` (practice branch), each question citing its lesson |
| E2 | "Who won the 2022 World Cup?" | `notes_assistant` → no match, refusal |
| E3 | S-8 combined message | `analyse_spending` + `notes_assistant` |
| E4 | "What did my budgeting notes say about food spending?" | `notes_assistant` only (not `get_advice`) |
| E5 | "Log $350 for school fees" | `log_expense` only (not `notes_assistant`) |

E4 and E5 are deliberately confusing, to test the routing rule.

The v0.1 version of E1 (fee schedule late fee) is already covered by S-5, so it was swapped for a practice-question message to stay within 3–5 own messages. S-10 to S-12 are also run and recorded as extra rows in the results matrix.

### 9.3 Before vs after measures (report 3.3)
| Measure | Before | After | Out of |
|---|---|---|---|
| Fixed eval set passed | | | |
| Messages that called a tool when none was needed | | | |
| Extension messages passed | — | | 5 |

### 9.4 Five failure cases, run from the page (report 4.2)
| Case | How we run it |
|---|---|
| Normal question | Send S-2 from the page |
| Workflow switched off | Deactivate the hub, send S-2, record the widget's error, reactivate |
| Origin not allowed | Open the same page from another address (for example `localhost`), send a message |
| Tool the agent doesn't have | "Book me a flight to Bangkok" |
| Empty message | Send an empty or whitespace message |

### 9.5 Response times (report 4.3)
- **Cold:** first message after at least 10 minutes idle.
- **Warm:** the next message straight after.
- Measure: stopwatch from send to first reply, plus the execution duration in n8n. Three tries each, report the median.

---

## 10. Acceptance criteria (definition of done)

The build is done when, on the nominated account and from the chat page:
- [ ] All seven tools connected; hub shows exactly 7 tools
- [ ] S-1 to S-9 behave as in Section 3, read from the executions
- [ ] Combined message S-8 calls both tools in one execution
- [ ] `documents` tab has headers in row 1 and one row per ingested document
- [ ] Pinecone index holds the seed documents, the lesson notes and the generated practice Q&A; re-ingest on demo day only if the fallback in-memory store is used
- [ ] Re-ingesting a document leaves no duplicate vectors
- [ ] S-10, S-11 and S-12 behave as in Section 3
- [ ] Generated practice Q&A spot-checked against the lessons; result recorded in `eval/`
- [ ] Turn log writes one row per turn; digest writes one row a day
- [ ] Allowed origin = page address; five failure cases recorded
- [ ] Before and after eval runs recorded in `eval/`
- [ ] Final workflow JSON exported from the nominated account in `workflows/final/`, no secrets
- [ ] Site files in `site/`

---

## 11. Team, ownership and deliverables

| Member | Build | Report sections | Presentation |
|---|---|---|---|
| 1 | Hub: session memory, 7-tool wiring, system message routing, merge of all workflows | 1.3, 2.1, 2.3 | Intro, architecture, wrap-up |
| 2 | Logger (`txn_id` duplicate check), Delete Expense; `transactions`, `categories`; seed documents and lesson notes (F20); eval runs (F21) | 2.2 (their rows), 2.4, 2.5 (their part), 3.1, 3.2 | Demo: logging and deleting |
| 3 | Analyst (code only), Coach, Daily Digest; `budgets` (including the `school fees` row, F17) | 2.2 (their rows), 3.3 | Demo: Analyst, Coach, digest |
| 4 | Pinecone index, embeddings, `documents` tab, ingestion with the practice Q&A step, Notes spoke (citation, refusal, practice branch) | 1.1, 1.2, 1.4 | Demo: cited answers, practice questions and refusals |
| 5 | Chat page, CORS origin, turn log, `agent_log`, repo hygiene | 4.1, 4.2, 4.3 | Live deployment demo |

Remaining sections: **2.4** (combined message evidence) → M2, with M1, M3 and M4 present for the combined-message session; **3.1, 3.2** (eval method, results matrix) → M2; **3.4** (failure analysis) → each member writes the failures from their own component, labelled prompt, tool or data; **3.5** (limits) → whole team review; **5** (contributions) → each member writes their own row.

### 11.1 Interface contracts
| Spoke | Input | Output |
|---|---|---|
| Logger | `{ description, amount, category, date }` | `{ status, txn_id, reason }` |
| Analyst | `{ month, category? }` | `{ month, totals[], skipped_repeats, skipped_unreadable }` |
| Coach | `{ question }` | `{ advice }` |
| Delete Expense | `{ action, query?, txn_id? }` | `{ matches[] }` or `{ deleted, txn_id }` |
| Notes | `{ action, question?, doc_title? }` | `{ matched, answer, sources[] }` or `{ documents[] }` |

Any change to a contract is agreed in the group chat **before** it is built. The Notes contract is **unchanged** by the practice-question feature: practice requests use `ask`, and the spoke decides internally.

### 11.2 Presentation plan (30 minutes including questions)
| Min | Segment | Who |
|---|---|---|
| 0–4 | Intro: problem, S5, architecture | M1 |
| 4–8 | Logging, duplicate refusal, delete with confirm | M2 |
| 8–12 | Analyst, Coach, digest row | M3 |
| 12–17 | Cited answers, practice questions, refusal, combined message | M4 |
| 17–22 | Live page, five failure cases, cold vs warm | M5 |
| 22–24 | Eval before vs after, what we cut, wrap-up | M1 |
| 24–30 | Questions (anyone may be asked about any part) | All |

Demo rules: page already open and warmed up, say up front that the first turn is slow, show at least one refusal, backup recording ready.

---

## 12. Timeline

| When | Milestone | Who |
|---|---|---|
| This week | Contracts agreed; nominated account chosen; repo skeleton merged | All |
| Lesson 13 | Each spoke works alone with test input; `documents` tab and seed docs ready; delete-with-confirm done | All |
| **Before Lesson 14** | **Everything merged on the nominated account**; "before" eval run recorded | M1 + all |
| Lesson 14 (clinic) | Notes spoke connected as tool 7; S-8 working from the page | M1, M4 |
| Before the "after" eval run | Practice Q&A step and branch working; generated Q&A spot-checked; extension frozen. If it is not stable by the cutoff date the team agrees, drop it and log it as a scope decision in report 2.5 | M4, M2 |
| Term break, week 1 | "After" eval run; failure analysis; prompt tuning; five failure cases and latency recorded | M1, M3, M5 |
| Term break, week 2 | Build frozen; final JSON exported to repo; report draft complete | All |
| 3 days before demo | Slides done; full rehearsal with the page open; backup recording made | All |
| Lesson 17/18 | Present; submit report PDF + .pptx on SA3.0; repo final by 23:59 | All |

---

## 13. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Hub confuses notes and finance questions | Eval failures | Sharp routing rule, tool description that names when finance is meant, confusing cases in our eval set |
| Notes answer drifts into general knowledge | Breaks S5's main rule | Similarity threshold in the spoke; skip the model call when nothing matches; exact refusal wording |
| In-memory vector store lost on restart | Empty answers at the demo | Use a hosted store, or re-ingest on the demo-day checklist |
| Merge breaks sub-workflow links or credentials | Hub tools fail | Merge before Lesson 14 and follow the checklist in the Member 1 PRD |
| Shared model quota runs out | Slow or failed demo | Digest once a day only; no test loops; backup recording |
| Nominated account owner absent | No demo | Choose the most reliable attendee; a second member can edit the sheet |
| Secrets leak into JSON or screenshots | Rule breach | Search exports for keys before every push; crop screenshots |
| Finance breaks after extension | Lose baseline marks | Before/after runs of the fixed eval set; fix before adding more |
| Generated practice questions contain facts not in the lessons | Breaks grounding (G2, G7) | Generate from one chunk at a time, nothing for chunks with nothing testable, keep `source_chunk` on every question, human spot-check before the "after" run |
| Pinecone quota hit or key fails on demo day | Empty answers | Few documents, no test loops, fallback in-memory store plus re-ingest on the demo checklist, key only as a named credential |
| Duplicate vectors from re-ingestion | Skewed retrieval, repeated questions | Fixed chunk ids; delete by `doc_id` before re-ingesting |
| Practice request routed to the Coach or answered from general knowledge | Eval failure | R10, E1 and the refusal case S-11 in our eval set |

---

## 14. Open questions

1. Real names of baseline tools 4 and 5.
2. Nominated account: which member?
3. `txn_id` format (M2).
4. ~~Vector store: hosted or in-memory?~~ Decided: Pinecone (M4). Fallback: in-memory plus re-ingest.
5. Does the fee schedule add a spending category such as `school fees` (F17)?
6. Where the page is hosted (GitHub Pages or other) and its exact address, for the allowed origin (M5).
7. ~~Owners for the unassigned report sections~~ Assigned in Section 11 (3.5 and 5 as suggested there).
8. Practice questions are generated once at ingestion, so they form a fixed bank (3 picked at random per request). Do we also want fresh questions generated at ask time from retrieved lesson text (a model call in the spoke, same "only from these excerpts" rule)? Decide after the core is stable.
9. Which module and which 3–5 lessons? Is slide use allowed for coursework, or do we use our own notes?
10. Which embeddings model (it sets the Pinecone index dimension)?
11. Cutoff date for dropping the practice-question feature if it is not stable.

---

## 15. Changes from v0.1 (18 Nov 2026)

- **Extension added:** lesson-scoped revision assistant with practice questions generated at ingestion (G7, S-10 to S-12, F18 to F22, R10). No new tool or action; the Notes contract is unchanged.
- **Vector store decided:** Pinecone, with index rules and a fallback (Section 5.3).
- **Workload rebalanced:** M2 takes seed documents and lesson notes, eval runs, and report 2.4, 3.1, 3.2. M3 gets the `school fees` row in `budgets`. M1 and M4 keep the hard builds. Report 3.4 is written per component.
- **Eval:** E1 swapped for a practice-question message to stay within 3–5 own messages.
- **Other documents to update:** the Member 1 PRD needs R10 and the lesson-scope wording in the system message and `notes_assistant` tool description.
