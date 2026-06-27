# Test Evidence — 3 runs through the agent

The workflow starts with a **Form Trigger** (`Learner Query Form`). For each run:

1. Click **Test workflow** (or **Execute workflow**) — n8n opens the form in a new tab.
2. From the **Sample query** dropdown, pick the option for the test (below).
3. Click **Submit** — the agent runs.
4. Screenshot the **final node's** output for that path.

> To test your own wording instead, pick **"Custom (type your own below)"**, type into
> **Custom query**, and (optionally) put a seeded email like `abhiram540@gmail.com` in
> **Custom learner email** so it maps to a known account.

---

## Test 1 — Course-access issue (auto-resolved) ✅

**Form selection:** Sample query → **"Course access — Abhiram (login issue)"**
(resolves to Abhiram, abhiram540@gmail.com, Python course; *"…can't log into my account / invalid credentials…"*)

**Path:** Form → Build Query → Classify → `course_access` → Lookup → **Course Access KB** → Draft Reply → Format Output

**Screenshot:** the **Format Output** node's output.
**Expected:**
- `category`: `course_access`, `status`: `auto_resolved`, `handledBy`: `AI Agent`
- `courseName`: `Python for Data Professionals` (account found via seeded mock data)
- `agentReply`: a warm reply addressed to Abhiram with the password-reset / re-sync steps,
  signed *"Team CodeBasics"*.

---

## Test 2 — Refund request, INSIDE the window (auto-approved) ✅

**Form selection:** Sample query → **"Refund within window — Ravi (Power BI)"**
(resolves to Ravi Kumar, order CB-10241, purchased 3 days ago; *"…I'd like a refund please."*)

**Path:** Form → Build Query → Classify → `refund` → Lookup (3 days ago → within 7-day window) → **Refund Window Check** (`APPROVE_AUTO`) → Draft Reply → Format Output

**Screenshot:** the **Format Output** node's output.
**Expected:**
- `category`: `refund`, `status`: `auto_resolved`
- `agentReply`: confirms the purchase is within the 7-day window and a full refund will be
  processed to the original payment method in 5–7 business days.

> Bonus variant (outside-window): pick **Custom**, set **Custom learner email** to
> `meena.nair@example.com` (purchased 40 days ago) and type a refund request. The agent
> politely declines the cash refund and offers a course swap / learning credit instead.

---

## Test 3 — Edge case → ESCALATED to a human ✅ (escalation branch)

**Form selection:** Sample query → **"Escalation — Arjun (payment deducted)"**
(resolves to Arjun Shah, order CB-11233; *"My payment failed but the money was deducted… this is a billing problem."*)

**Path:** Form → Build Query → Classify → `escalation_human` → **Create Escalation Ticket** → Notify Human (Tier-2)

**Screenshot:** the **Create Escalation Ticket** node's output (and/or **Notify Human**).
**Expected:**
- `status`: `escalated`, `handledBy`: `Human (Tier-2 Human Support)`, `priority`: `high`
- `ticketId`: e.g. `TCK-482913`, `slaHours`: 24
- `agentReply`: tells Arjun a specialist will respond within 24 hours.
- `internalNote`: full context for the human (reason, learner, order, payment status).

This run demonstrates the escalation requirement: a billing dispute / "money deducted but
payment failed" never gets an auto-answer — it goes straight to a human ticket.
(The `Format Output` node stays unrun on this path — that's expected.)

---

### Quick reference — which query hits which branch

| If the query is about… | Category | Ends at |
|---|---|---|
| login / can't access course | `course_access` | Auto reply |
| refund / money back | `refund` | Auto reply (approve or decline by window) |
| certificate / discount / guidance | `other` | Auto reply (General Help KB) |
| money deducted, double charge, real bug, partnership | `escalation_human` | **Human ticket** |

---

### Tip for the Google Sheet evidence

After running all 3, your `Log` sheet will have **3 rows** — two `auto_resolved` (course
access, refund) and one `escalated` (the billing case), because both the auto and escalation
paths feed the same `Log to Google Sheet` node. A screenshot of that filled sheet is strong
"tool chaining + output" evidence for the rubric.
