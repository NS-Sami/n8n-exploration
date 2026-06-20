# 01 — Fundamentals

## n8n's data model

- Every node passes data to the next node as **JSON**.
- Data flows between nodes as a list of "items" — even a single piece of data is an item inside an array.
- Standard item shape: `{ json: { ...actual data... } }`

## JSON refresher (the part that actually matters for n8n)

- `{}` = object → key-value pairs. Access with dot notation: `obj.key`
- `[]` = array → ordered list. Access by index: `arr[0]`
- Nesting is just objects/arrays containing more objects/arrays. Example from a real bug I hit:
  ```
  content.parts[0].text
  ```
  Read as: object → key "content" → key "parts" (an array) → index 0 (an object) → key "text"

## "The model responded in JSON" ≠ "n8n has structured data"

LLMs return **text**. That text can follow JSON syntax, but it's still just a string until something explicitly parses it (`JSON.parse()`, or n8n's Structured Output Parser node). This tripped me up — see BUGLOG 002.

## Code node: Mode matters

Two modes, very different behavior:

| Mode | Executes | Use `$json` or `$input.first()`? | Return shape |
|---|---|---|---|
| Run Once for All Items | Once, total | `$input.all()` to get everything, `$input.first()` only grabs item 0 | Array: `return [{json:...}, {json:...}]` |
| Run Once for Each Item | Once per item | `$json` = current item | Single object: `return {json:...}` |

Mismatching these is a silent bug — no error thrown, just wrong/missing data downstream. See BUGLOG 003.

## Community nodes vs native nodes

- **Native/core nodes**: ship with n8n, work everywhere (Cloud + self-hosted).
- **Community nodes**: npm packages, install separately, **self-hosted only** — not available on n8n Cloud at all.
- Course videos built on self-hosted setups may use community nodes that won't exist on Cloud. Native alternative (often a Webhook node) usually covers the same functionality.

## Webhooks (basic mental model)

- A Webhook node in n8n generates a unique URL.
- An external service (e.g. Tally, Stripe, etc.) is configured to send a POST request to that URL whenever an event happens (form submitted, payment received, etc.).
- n8n receives that POST request as the **trigger** that starts the workflow, with the event's data as the input.
- This is the underlying mechanism behind most "trigger" integrations, even ones with a dedicated friendly node — dedicated nodes are often just a webhook with nicer field-mapping UI.

## Structured Output Parser (n8n AI nodes)

- Connects to a dedicated **Output Parser** input on AI/LLM nodes (only appears once "Require Specific Output Format" is enabled on the model node).
- Define expected shape via a JSON example or full JSON Schema.
- Handles parsing + validation + retry-on-malformed-output automatically — more robust than manual `JSON.parse()` in a Code node.
- Worth learning early since structured output is common in any AI-driven automation (lead gen, content generation, data extraction).
