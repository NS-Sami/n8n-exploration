# 02 — Form Triggers, Wait Nodes, and Email Automation

Three standalone experiment workflows, each isolating a different concept.

---

## Experiment A: Form submission → AI response → delayed email

**Flow:** `On form submission → Message a model (AI) → Wait → Send a message (Gmail)`

**What it does:**
- **On form submission** — n8n's native Form trigger. Starts the workflow when someone fills out an n8n-hosted form (different from the Tally webhook approach — this form is built and hosted by n8n itself, no external tool needed).
- **Message a model** — sends the form's input to an AI model and gets a text response back.
- **Wait** — pauses the workflow for a set duration before continuing. Useful for things like "don't send the follow-up immediately, wait an hour" type automations.
- **Send a message** — sends the AI's response out via Gmail.

**Concept learned:** n8n has its own built-in form-hosting trigger, separate from external form tools. No webhook setup or third-party config needed — n8n generates the form and the URL itself.

**Note:** The model node has a yellow warning triangle in the screenshot — flagged for follow-up (likely a missing required field, e.g. prompt/credential not fully configured). Worth re-checking before relying on this workflow.

---

## Experiment B: Standalone AI → Wait → Email (no trigger shown)

**Flow:** `Message a model1 → Wait1 → Send a message2`

**What it does:** Same pattern as Experiment A (AI response → delay → email), built as an isolated test of just that chain, without a form trigger attached. Used to test the AI → delay → send logic on its own before wiring it to a real trigger.

**Concept learned:** Building and testing a chain of nodes in isolation (using manual/test execution) before connecting it to a live trigger is a reasonable way to debug — confirms the middle of a pipeline works before worrying about what starts it.

---

## Experiment C: Full sheet-driven email draft pipeline

**Flow:** `When clicking 'Execute workflow' → Get row(s) in sheet → Message a model3 → Code in JavaScript → Update row in sheet → Create a draft`

**What it does:** This is the lead-gen/outreach pipeline from earlier in the course (see BUGLOG 002 and 003):
1. Manually triggered (for testing).
2. Pulls candidate rows from Google Sheets.
3. Sends each row's data to an AI model, prompted to generate a structured cold-email JSON (Subject, Icebreaker, CompiledEmail, CallToAction, Postscript).
4. **Code in JavaScript** — parses the AI's raw text response into actual usable fields (`JSON.parse`), running in **"Run Once for Each Item"** mode so all rows get processed, not just the first (see BUGLOG 003 for the bug this fixed).
5. Writes the parsed fields back into the sheet.
6. **Create a draft** — instead of sending the email immediately, creates a Gmail **draft** for each generated email — a safer pattern for cold outreach, since it allows human review before anything actually sends.

**Concept learned:** "Create a draft" vs "Send a message" is an important distinction for any outreach automation — drafting lets a human review/edit AI-generated content before it goes out, which matters a lot for cold email since AI output can be wrong, off-tone, or factually shaky about a prospect.

---

## Open questions / follow-ups

- [ ] Investigate the warning triangle on "Message a model" nodes in Experiments A and B
- [ ] Understand exact configuration options for the Wait node (fixed duration vs. wait-until-webhook vs. wait-until-time)
- [ ] Compare "Create a draft" vs "Send a message" node configs side by side
