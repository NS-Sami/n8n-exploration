# 03 — Web Scraping + AI Data Extraction

## Workflow: Website → HTML extraction → AI analysis → structured JSON

**Flow:** `When clicking 'Execute workflow' → HTTP Request → HTML → Message a model (Gemini) → Code in JavaScript`

**What it does:**
1. **HTTP Request** — sends a GET request to a target URL (e.g. `https://leftclick.ai/`) and pulls down the raw HTML of the page.
2. **HTML node** — n8n's native HTML-parsing node, set to `extractHtmlContent`. Strips the raw HTML down to readable text content, dropping tags/scripts/styles so the AI isn't fed a wall of markup.
3. **Message a model** — sends the extracted text to Google Gemini (`gemini-3-flash-preview`) with a prompt instructing it to act as a "web scraping assistant" and return a fixed JSON shape:
   ```json
   {
     "summary": "",
     "threeUniquePoints": [],
     "probableCustomerDemographic": "",
     "contactInformationIfAny": [{}]
   }
   ```
4. **Code in JavaScript** — parses Gemini's text response into the actual structured object (same pattern as BUGLOG 002/003 — strip code fences, `JSON.parse`).

**Concept learned:** This is a general-purpose **"scrape any page → get structured insights about it" pipeline** — no fixed input source like a spreadsheet or form. Useful pattern for things like lead research, competitor analysis, or auto-summarizing a company from their website alone.

## Note: `thoughtSignature` field in Gemini's response

When using a "thinking" model variant (e.g. `gemini-3-flash-preview`), the API response includes a `thoughtSignature` field alongside `text` inside `parts[0]`. This is an opaque, encrypted token representing the model's internal reasoning trace — not part of the actual answer, and not something the parsing code needs to touch. It exists mainly for maintaining reasoning continuity across multi-turn conversations. Safe to ignore in single-shot extraction use cases like this one; the parsing code only ever reads `parts[0].text`, never `thoughtSignature`.

## Reusable prompt pattern

Worth keeping as a template for future extraction workflows — the structure of "describe the output as JSON with explicit empty fields" works well to keep the model from going off-script:

```
Your task is to take input of a bunch of text from a website and provide
me some data based on the input text in this JSON format.

{"summary": "", "threeUniquePoints": [], "probableCustomerDemographic": "",
"contactInformationIfAny": [{}]}
```

## Open questions / follow-ups

- [ ] Test against a page with thin/sparse content — does the model hallucinate fields when there's not enough real data?
- [ ] Consider adding a Structured Output Parser here instead of manual Code-node parsing, since this is the same use case it's designed for
