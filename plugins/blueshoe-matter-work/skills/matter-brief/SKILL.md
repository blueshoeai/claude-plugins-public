---
name: matter-brief
description: Use when the user asks to be briefed, caught up, or brought up to speed on a Blueshoe matter ("brief me on the X matter", "where are we on X", "what's due this week on X", "what's changed since Friday"). Pulls the matter's brief, chronology, findings, gaps, contradictions, deadlines, and recent documents from Blueshoe and summarizes them with cites.
---

# Matter brief

Produce a short, sourced status of one Blueshoe matter from the record, not from memory.

## Steps

1. Resolve the matter: `matter_list`, match on the user's words, take `matter_id`. If more than one matches, ask which.
2. In one `execute` call, fetch:
   - `brief_get({matter_id})`
   - `projection_get({matter_id, slug:'chronology'})` — keep the last 10 events
   - `finding_list({matter_id})` — count by kind; keep the 15 most recent by `start_min`
   - `projection_get({matter_id, slug:'gaps'})` and `projection_get({matter_id, slug:'contradictions.open'})`
   - `obligation_list({matter_id})` — split past-due, next 14 days, later
   - `document_list({matter_id})` — the 10 most recently updated paths
   - `event_list({matter_id, grain:'activity', limit:20})` — what happened since the brief's `generated_at`
3. If the user named a period ("since Friday"), filter events, findings, and documents to it.

## Output

Write it in plain prose with short sections, in this order, and keep it under a page:

- **Standing** — the platform brief, with its `generated_at`; note in one line anything in the activity feed that postdates it.
- **What's due** — past-due first, then the next 14 days, each with its source quote from the narrative.
- **What happened recently** — the chronology tail, one line per event, with finding id.
- **Open gaps and contradictions** — each with the record's own wording and the finding id; flag any the attorney has not seen.
- **Recently filed** — document paths and dates.
- **Questions for the attorney** — anything above that needs a decision.

Cite findings by id and quote verbatim from `cites` when a fact matters. Do not add facts that are not in the record; if something the user mentions is missing from Blueshoe, say so and offer to file it.
