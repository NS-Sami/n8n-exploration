# 04 — AI Agents, Tools, and Memory

This is a step up from previous workflows: instead of a fixed linear chain (trigger → AI → action), an **AI Agent** node makes its own decisions about *whether* and *which* tool to call, based on the user's message. This introduces a new class of bugs that don't exist in linear workflows — the model can choose wrong, or choose not to act at all.

---

## Workflow: Chat-based Google Calendar assistant

**Flow:** `When chat message received → AI Agent` with three things wired into the Agent's special inputs:
- **Chat Model** → Google Gemini Chat Model
- **Memory** → Simple Memory (keeps conversation context across messages in a session)
- **Tool** → Get many events in Google Calendar

**What it does:** A conversational assistant you can ask things like *"what am I doing on 24 June, 2026"* — the agent decides to call the Google Calendar tool, retrieves real events, and answers in natural language.

## Key structural difference from earlier workflows

In every prior workflow (form → AI → sheet, scrape → AI → parse, etc.), the path through the workflow was **fixed** — node A always runs, then node B, then node C, regardless of input. With an **AI Agent**, the model itself decides at runtime whether to call a tool, which tool, and with what arguments. The "Tool," "Memory," and "Chat Model" connections aren't a sequential pipeline — they're resources the Agent can choose to use.

This means agent behavior has to be **steered through instructions**, not just wiring. Connecting a tool does not guarantee the agent will use it.

---

## Bug: Agent claims it has no calendar access, despite having a working tool connected

**Symptom:** Asked the agent *"what am I doing on 24 June 2026"*. Got back: *"I am sorry, but I cannot tell you what you are doing on June 24, 2026. I do not have access to your personal calendar information."* — even though a correctly configured "Get many events in Google Calendar" tool was connected.

**How it was diagnosed:** Checked the **Logs panel** after execution. The execution chain went straight from Memory/Model to the final answer — the Calendar tool step never appeared as executed at all. This confirmed the tool wasn't being called; the model was refusing before even attempting it.

**Cause:** With no system prompt set on the AI Agent, the model defaults to its general training behavior — which includes a strong learned pattern of saying "as an AI, I don't have access to personal data like your calendar." Simply having a tool wired up doesn't override that default instinct. The model needs to be explicitly told, in context, that this assistant *does* have real calendar access via its tools, and that it should use them rather than giving the generic disclaimer.

**Fix:** Added a system prompt to the AI Agent node explicitly countering the default refusal pattern:

```
You are a helpful calendar assistant with real, working access to the
user's Google Calendar through your tools. You are NOT a generic AI
without calendar access — you have a "Get many events" tool that
actually queries their real calendar data.

When the user asks about their schedule, upcoming events, or what
they're doing on a specific date, you MUST use the "Get many events"
tool to check before answering. Never claim you don't have calendar
access — you do, via your tools. Only say a date is free if the tool
returns no events for it.
```

After adding this, the Logs panel showed "Get many events in Google Calendar" as an actual executed step, and the agent correctly returned real event data ("Capstone project" as an all-day event on June 24, 2026).

**Lesson:** For AI Agents, a missing system prompt isn't just "less helpful guidance" the way it might be for a single-shot prompt — it can cause the agent to silently skip tools it has every technical ability to call, because the model falls back on default refusal language baked in from general training. Always explicitly state in the system prompt that the agent has real tool access and should use it, especially for categories of request (personal data, calendars, files) where AI models have strong learned habits of saying "I can't do that."

---

## Related earlier bug: Agent looping instead of failing cleanly

Before the system prompt was added, an earlier version of this same workflow (with only a "Create event" tool connected — no "Get many" tool at all) produced a different failure: the agent repeatedly cycled Memory → Model → Memory → Model many times before eventually erroring out with "There was a problem executing the workflow." That was a **tool mismatch** (no read-capable tool existed at all) combined with no system prompt to tell the agent to give up gracefully. See this as a sibling failure mode to the one above: missing capability + no guidance can cause either (a) silent wrong refusal, or (b) an unproductive retry loop — depending on whether *any* plausible tool exists for the model to keep attempting.

---

## Concepts learned

- **AI Agent node** — non-linear; model decides whether/which tool to call at runtime
- **Memory (Simple Memory)** — maintains conversation context across turns in the same chat session, referenced by Session ID
- **Tool nodes** — any normal n8n node (e.g. Google Calendar) can be exposed to an Agent as a callable tool; the Agent decides when to invoke it
- **System prompts are not optional for agents** — they're the primary mechanism for steering tool usage, and their absence can cause silent failures that are hard to diagnose without checking the Logs panel
- **Logs panel is the key debugging tool for agents** — since execution path isn't fixed, the Logs panel (not just the final output) is what reveals whether a tool was actually called or skipped

## Open questions / follow-ups

- [ ] Test what happens with multiple similar tools (e.g. both "Create" and "Get Many" connected) — does the agent reliably pick the right one for read vs. write requests?
- [ ] Explore "Tool Description" field (seen set to "Set Automatically" in the Calendar tool config) — does writing a custom description improve tool selection accuracy?
- [ ] Test memory persistence across a longer multi-turn conversation (e.g. follow-up questions referencing earlier answers)
