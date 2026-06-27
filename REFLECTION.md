# Reflection

**What I built.** An AI assistant for CodeBasics learner support. When a learner sends a
question, the assistant reads it, works out what it is about (for example a login problem or a
refund request), finds the right answer from our help information, and writes a friendly reply.
For genuinely tricky cases — such as billing disputes or "my payment failed but money was
deducted" — it does not try to answer; it raises a ticket and hands the case to a human. Every
query and its outcome is recorded in a Google Sheet so the team has a clear log. I focused on
doing two common query types properly — **course access** and **refunds** — plus a clear path
for sending edge cases to a person.

**What broke and how I fixed it.**

1. *The AI model would not run because of usage limits.* My first choice (an OpenAI model) and
   even a couple of the Google Gemini models returned an error saying the free usage quota was
   already used up, so the assistant could not reply. I worked through the options and found a
   free model — **Google Gemini 2.5 Flash** — that had available capacity, switched the assistant
   to use it, and everything ran smoothly. This also kept the whole project free to run.

2. *Testing was slow because I had to type the details every time.* To check different
   situations, I originally had to enter the learner's name, email and question by hand for each
   test, which was fiddly and error-prone. To make this easier, I added a simple **intake form**
   with a drop-down of ready-made sample questions (course access, refund, escalation). Now I can
   just pick a scenario from the list and run it in one click, which made testing much faster and
   more reliable.

**What I would improve next.**
- Connect a real email inbox so the assistant can actually receive questions and send its
  replies, instead of being run manually for testing.
- Keep the help answers automatically up to date so responses stay accurate as our courses and
  policies change.
- Track simple success measures — how many queries the assistant resolves on its own versus how
  many it passes to a human — so we can show the impact and spot where it needs improvement.
- When the assistant is unsure about a query, send it to a human rather than guessing, so
  learners always get a reliable answer.
