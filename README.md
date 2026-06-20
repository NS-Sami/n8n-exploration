# n8n Learning Log

Personal learning repo for n8n workflow automation, built while following [Nick Saraev's N8N Full Course (6 Hours)](https://youtu.be/2GZ2SNXWK-c) on YouTube, plus independent exploration beyond the course.

## Why this repo exists

I'm a CSE student building toward an AI/ML engineering career, with a focus on LLM and RAG systems. n8n is part of that path — it's a fast way to prototype and orchestrate AI pipelines (webhooks, API chaining, structured LLM output parsing) without writing full backend infrastructure for every experiment. This repo tracks what I'm learning, the workflows I build, and the bugs I run into along the way — including the actual debugging process, not just the fixes.

## Structure

| Folder | Contents |
|---|---|
| `notes/` | Concept notes, organized by topic as I progress through the course (see `notes/01-fundamentals.md`, `notes/02-forms-wait-email.md`, `notes/03-web-scraping-ai-extraction.md`, `notes/04-ai-agents-tools-memory.md`, etc.) |
| `workflows/` | Exported JSON files for each workflow I build, so they can be re-imported into any n8n instance |
| `BUGLOG.md` | Real issues I hit, what caused them, and how I fixed them |

## Setup

These workflows were built on **n8n Cloud**. Some workflows that rely on community nodes (e.g. Tally Forms) have been adapted to use n8n's native Webhook node instead, since community nodes aren't supported on Cloud — only on self-hosted instances. Notes on this tradeoff are in `BUGLOG.md`.

To import a workflow:
1. Open n8n → **Workflows** → **Add Workflow** → **Import from File**
2. Select the relevant `.json` file from `workflows/`
3. Reconnect your own credentials (API keys, OAuth) — these are never included in exported files

## Progress

- [x] Environment setup (n8n Cloud)
- [x] First trigger + action workflow
- [x] Webhook-based triggers (Tally form integration)
- [x] n8n native form triggers
- [x] LLM node integration (Google Gemini)
- [x] Structured output parsing (Code node + Structured Output Parser)
- [x] Wait nodes / delayed actions
- [x] Email automation (send vs. draft)
- [x] Web scraping (HTTP Request + HTML extraction) → AI structured analysis
- [x] AI Agents
- [ ] Multi-step automation chains
- [ ] Error handling / retries
- [ ] Deployment considerations

## Tech / Tools Covered

- n8n Cloud
- Google Sheets node
- Google Gemini (LLM node)
- Webhooks
- n8n native form triggers
- Wait node
- Gmail node (send + draft)
- HTTP Request node
- HTML extraction node
- Structured Output Parser
- AI Agent node (tools, memory)
- Google Calendar node (as AI tool)
- JavaScript (Code nodes)

## License

Personal learning project. Course content credit: [Nick Saraev](https://youtu.be/2GZ2SNXWK-c). Workflow JSONs in this repo are my own implementations/adaptations, not redistributions of the course's original files.
