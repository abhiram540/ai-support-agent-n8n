# Reflection

**What I built.** An n8n agent that triages CodeBasics learner-support queries end to end:
a **Form Trigger** takes an incoming query, an LLM (Google **Gemini 2.5 Flash**) classifies the
intent, the agent looks up a (mock) learner account, retrieves the right FAQ answer, applies
refund-policy logic, and either drafts a learner-facing reply or raises a human ticket — logging
every case to a Google Sheet. I deliberately scoped it to two query types done properly —
**course-access** and **refunds** — plus a general auto-path and an explicit human-escalation
branch, rather than covering every query type thinly.

**What broke and how I fixed it.**
1. *The classifier's output wasn't reliably parseable.* The LLM occasionally wrapped its JSON
   in markdown code fences, which broke `JSON.parse`. I fixed it by slicing the string from the
   first `{` to the last `}` before parsing, and wrapping it in a try/catch that falls back to
   the `other` category — so a malformed model response degrades gracefully instead of crashing
   the run.
2. *Context was lost after the LLM nodes.* The LangChain chain nodes only emit a `text` field,
   so the learner's original details disappeared downstream. I fixed this by re-reading the
   original data from earlier nodes by name (`$('Build Query')`, `$('Lookup Learner Account')`)
   inside the Code nodes, so every step has the full context it needs.
3. *Business decisions were drifting into the model.* My first version asked the LLM whether a
   refund was eligible. That's the wrong place for it — it's non-deterministic and hard to
   audit. I moved refund eligibility into a deterministic `Refund Window Check` Code node
   (compare purchase date to a 7-day window) and let the LLM only *write* the reply. The model
   reasons and communicates; code owns the rules.
4. *The LLM node wouldn't run — `429`, quota `limit: 0`.* On Gemini's free tier, `gemini-2.0-flash`
   returned a quota error where the free allowance was literally zero for that model on my key.
   The misleading part was the "too many requests / retry in 29s" wording — waiting never helped.
   I fixed it by switching the model to **`gemini-2.5-flash`**, which had free-tier quota, and the
   workflow ran immediately. Lesson: `limit: 0` is an entitlement problem, not a rate-limit you
   wait out — change the model (or project), don't retry.
5. *The static input made testing clumsy.* My first version used a Set node with one hardcoded
   query, so every run sent the same message and I had to hand-edit fields between tests. I
   replaced it with a **Form Trigger** plus a `Build Query` resolver: each run opens a form where
   I pick one of three preset test cases (or type a custom query). This made the three test runs
   repeatable and one-click, and turned the trigger into something closer to a real support intake.

**What I'd improve next.**
- Add a **reply-sending node** (Gmail / email) on the auto path so it actually closes the loop
  with the learner, not just drafts the reply.
- Swap the hardcoded FAQ Code nodes for a **vector store / RAG retrieval** over the real help
  docs, so answers stay current without editing the workflow.
- Add a **confidence threshold**: if the classifier's confidence is low, route to a human even
  when the category looks auto-answerable — better to over-escalate than send a wrong answer.
- Add **evaluation**: log every classification and have a human spot-check a sample weekly to
  measure mis-routing, and track auto-resolution rate and escalation rate as the core PM metrics.
- Handle **multi-intent** messages (e.g., "I can't log in *and* I want a refund"), which the
  current single-label classifier would only partly address.
