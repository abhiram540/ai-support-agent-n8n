# AI Learner Support Agent (n8n)

A multi-step n8n agent that triages learner-support queries, answers the easy ones
automatically, and escalates genuine edge cases to a human.

---

## What it does (the 4 rubric stages)

| Stage | Node(s) | What happens |
|-------|---------|--------------|
| **1. Trigger** | `Learner Query Form` → `Build Query` | A Form Trigger opens an intake form each run: pick one of 3 preset test queries or choose **Custom** and type your own. `Build Query` resolves the selection into the learner name / email / order / query the rest of the flow expects. |
| **2. Reason** | `Classify Query (LLM)` + `Google Gemini Chat Model` → `Parse Classification` | LLM classifies the query into `course_access` / `refund` / `escalation_human` / `other`, plus intent, sentiment, confidence, and an escalation reason. JSON is parsed and validated. |
| **3. Act (tool chaining)** | `Lookup Learner Account` → `Route by Intent` → `Course Access KB` / `Refund Window Check` / `General Help KB` → `Draft Learner Reply (LLM)` | Mock account/order lookup → FAQ/knowledge-base retrieval → refund-window policy logic → LLM drafts the learner-facing reply. **4 chained actions.** |
| **4. Output** | `Format Output` → `Log to Google Sheet` **OR** `Create Escalation Ticket` → `Notify Human (Tier-2)` | Auto path returns a learner reply and logs it; escalation path raises a ticket and routes to a human. |

---

## How to import & run

1. **Import the workflow**
   - In n8n: top-right **⋯ → Import from File** → select `CodeBasics_AI_Support_Agent.json`.
2. **Add an LLM credential** (the only credential needed to run) — uses **Google Gemini (free tier)**
   - Get a free API key at <https://aistudio.google.com/app/apikey> (Google account, no card needed).
   - Open **Google Gemini Chat Model** → **Credential to connect with → Create New** →
     credential type **Google Gemini(PaLM) Api** → paste the key → Save.
   - If the **Model** field shows a warning, re-pick a free model from the dropdown.
     **Use `models/gemini-2.5-flash`** — `gemini-2.0-flash` may return a `429 / quota limit: 0`
     error on some free-tier keys.
3. **Run it**
   - Click **Test workflow** (or **Execute workflow**). n8n opens the intake form in a new tab.
   - From the **Sample query** dropdown, pick a preset (or choose **Custom** and type your own),
     then **Submit**. The agent runs end to end.
4. **View the result** — open the final node of the path that ran (`Format Output` for auto
   replies, `Create Escalation Ticket` for the human path) to see the output. See `TEST_QUERIES.md`.

> The **Log to Google Sheet** node ships **disabled** so the workflow runs end-to-end with
> *only* the LLM credential. To enable real logging: open the node, un-disable it, pick a
> Google Sheets credential, paste your Sheet ID, and add a tab named `Log`. It uses
> auto-map, so the columns are created from the output fields automatically.

---

## Mock data (in `Lookup Learner Account`)

| Email | Course | Purchased | Refund window | Payment |
|-------|--------|-----------|---------------|---------|
| abhiram540@gmail.com | Python | 5 days ago | **inside** (≤7d) | success |
| ravi.kumar@example.com | Power BI | 3 days ago | **inside** (≤7d) | success |
| meena.nair@example.com | Python | 40 days ago | **outside** | success |
| arjun.shah@example.com | SQL | 2 days ago | inside | **failed_money_deducted** |

Any unknown email falls back to a `no_account_found` guest record.

---

## Design notes

- **One LLM, two jobs.** A single `Google Gemini Chat Model` feeds both the classifier and the
  reply-drafter, so there's one credential to manage.
- **Determinism where it matters.** Routing and refund-eligibility are decided by code
  (Switch + `Refund Window Check`), not by the LLM — the model reasons and writes, but
  business rules stay deterministic and auditable.
- **Escalation is explicit and testable.** Anything the classifier marks `escalation_human`
  (billing dispute, "money deducted but payment failed", real bug, partnership) skips the
  auto-reply entirely and raises a ticket for a human.
