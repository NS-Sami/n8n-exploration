# 05 — If, Filter, Merge, Loop

Four core flow-control concepts in n8n, each explored as a separate experiment in one workflow file. These are the building blocks for any non-trivial automation — the difference between "do this" and "do this *if*, for *these* items, *combined* from multiple sources, *repeatedly*."

---

## Concept A: If node

**Flow:** `Edit Fields (firstName: "Sam") → If (firstName == "Sam"?) → ifWin (Prize: $100) OR ifLose (Prize: $5)`

**What it does:** The **If** node is a conditional branch — it splits the workflow into two paths based on whether a condition is true or false. Output 1 (true path) → `ifWin`. Output 2 (false path) → `ifLose`. Only one path executes per item.

**Condition used:** `$json['firstName :'] == "Sam"` — checks if the firstName field equals "Sam". Since it does, the $100 prize path fires.

**Key detail:** Field names with trailing spaces (`"firstName :"`, `"lastname : "`) are a real gotcha in n8n — the space is part of the key name. So `$json.firstName` would fail; you need `$json['firstName :']` with the exact whitespace. Worth naming fields cleanly from the start to avoid this class of bug.

**When to use:** Any time you need "do X if condition, do Y otherwise" — approval flows, routing based on input values, conditional notifications, etc.

---

## Concept B: Filter node

**Flow:** `Edit Fields1 (array: ["Sam", "Gaming", "Nadim", "Agent"]) → Filter (array contains "Nabil"?)`

**What it does:** The **Filter** node passes items downstream only if they match a condition. Items that don't match are dropped entirely — no false/else path like If. Here the filter checks whether the array `arrayName` contains "Nabil." Since "Nabil" isn't in `["Sam", "Gaming", "Nadim", "Agent"]`, zero items pass through — the workflow stops there for this branch.

**If vs Filter — the practical difference:**
- **If** → always produces output on one of two paths; nothing is dropped, just routed
- **Filter** → drops non-matching items entirely; only a true path exists; items that don't match simply don't continue

**When to use:** Cleaning/narrowing a list before passing it downstream — e.g. "only process leads from Bangladesh", "only send emails to users who opted in", "skip rows where status is already Complete."

---

## Concept C: Merge node

**Flow:** `Edit Fields2 (personOne: "Nabil", personTwo: "Sami") → [story personOne (Gemini) + story personOne1 (Gemini)] → Merge → Create a draft (Gmail)`

**What it does:** Runs two parallel AI calls simultaneously — one generating a funny story about "Nabil", one about "Sami" (both using Gemini 2.5 Flash, both prompted to stay under 50 words). The **Merge** node waits for both branches to finish, then combines their outputs into a single stream before passing to Gmail.

**Why this matters:** Without Merge, you can't use the output of two parallel branches together in the same downstream node. The Gmail "Create a draft" node needs both stories to exist before it can send — Merge provides that synchronization point.

**Subject line logic used in Gmail node:**
```
={{ $json.content.parts[0].text.includes("Nabil") ? "Nabil" : "Sami" }}
```
A JavaScript ternary expression inside an n8n expression — if the story text contains "Nabil", label the draft "Story about Nabil", otherwise "Story about Sami." This is how you write inline conditional logic directly in a field value without a full Code node.

**When to use:** Any time you split the workflow into parallel branches and need to recombine results — parallel API calls, A/B content generation, fetching from multiple sources before doing something with all of them together.

---

## Concept D: Loop (Split in Batches)

**Flow:** `Edit Fields4 (personOne: "Nabil", personTwo: "Sami") → Loop Over Items → Create a draft1 (Gmail) → Wait → [back to Loop Over Items]`

**What it does:** The **Loop Over Items** node (technically "Split in Batches") processes items one at a time in a loop:
1. Takes the first item → runs `Create a draft1` → runs `Wait` → feeds back into the loop
2. Takes the next item → same
3. When all items are processed → exits the loop via output 1 ("done" path)

The **Wait** node inside the loop adds a deliberate pause between iterations — useful for rate-limiting (e.g. "don't hit Gmail's API 100 times per second") or pacing outreach (e.g. "send one email per minute").

**Loop Over Items vs just letting n8n process multiple items naturally:**
- Normally, n8n processes all items through a node simultaneously (in parallel, effectively).
- Loop Over Items forces **sequential, one-at-a-time** processing with full control over what happens between each iteration.
- The Wait node inside the loop is what makes this actually useful — without it, looping is just slower parallel processing.

**When to use:** Rate-limited APIs, sequential operations where order matters, paced outreach automation, or any scenario where you need "do X, pause, do X again" rather than "do X for everything at once."

---

## Inline JS expressions (quick reference from this workflow)

n8n lets you write JavaScript expressions directly inside field values using `{{ }}` syntax:

| Pattern | What it does |
|---|---|
| `{{ $json.fieldName }}` | Access a field from current item |
| `{{ $json['field name with spaces :'] }}` | Access a field with spaces/special chars in the name |
| `{{ $json.text.includes("value") ? "yes" : "no" }}` | Ternary conditional inline |
| `{{ $json.content.parts[0].text }}` | Nested path access (Gemini response) |

These are evaluated at runtime per item — essentially JavaScript expressions with access to the current item's data.

## Workflow file

See `workflows/N8N Concept (If, Filter, Marge, loop).json` for the importable workflow.
