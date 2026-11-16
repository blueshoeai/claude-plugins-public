---
name: capture-to-blueshoe
description: Use at the end of any task that touched a Blueshoe matter, and whenever the user says "file this to Blueshoe", "make sure Blueshoe has everything", "update the record", "sync the matter", or "is this in Blueshoe". Reconciles the session's emails, documents, deadlines and decisions against the matter and files whatever is missing.
---

# Capture to Blueshoe

The record is only useful if it is complete. This skill makes sure that everything a session read or produced about a matter is in Blueshoe before the session ends — not summarized, not described, but the thing itself. Run it as the last step of every matter task, and on demand.

## What counts as "in the record"

- A document is in the record when the file itself is under the matter's `files/` (and, for PDFs and Word files, a text extract sits beside it so the layer can read it).
- An email is in the record when `correspondence_list` shows it (same thread, same date, same sender) with its attachments filed.
- A deadline is in the record when `obligation_list` shows it with the right time and a `narrative` naming the source.
- A decision, instruction, or status is in the record when a dated note under `files/notes/` states it.

## Checklist

Work through all five; report each as filed / already there / could not file.

### 1. Inventory what this session touched

List, from the conversation itself:
- every email read, sent, or drafted that involves the client, the other side, a court, a tribunal, or a third party on the matter;
- every file created, edited, sent, or received (filings, letters, exhibits, declarations, agreements, comparisons, spreadsheets);
- every deadline, date, or time limit read in any of them;
- every instruction, decision, or approval a person gave about how the matter is handled.

### 2. Reconcile emails

1. `correspondence_list({matter_id, limit:200})`, paging while `next_cursor` is set. Index by `thread_key`, `subject`, `from_addr`, `occurred_at`.
2. For each email from the inventory, and for each thread in the user's mailbox that matches the matter's parties and counsel (search the mailbox by the party names, firm names, and domains from `matter_get` / `case_party_list`), check whether it is already there.
3. File what is missing:
   - Preferred, when the user has approved it once in the session: forward the thread to the matter's Blueshoe intake address, so capture files it with attachments.
   - Otherwise fetch the raw RFC 5322 message from the mailbox and call `correspondence_file({matter_id, messages:[{raw: <base64>, provider_id}], source})`. Up to 25 per call; a message already on the record comes back `created:false`, so re-sending is safe.
   - A message too large for either route (very large attachments): file the attachments as documents under `files/correspondence/<date> <from> - <subject>/` and a short markdown note with the headers and body.
4. Ask the user once per session for permission to forward; do not forward or send anything without it.

### 3. Reconcile documents

1. `document_list({matter_id})` → the set of paths and sizes.
2. For each file in the inventory, decide the destination folder by what it is:
   - `files/<forum> filing <YYYY-MM-DD>/` for a filing set, with an index file (document, as-filed filename, pages, SHA-256, status);
   - `files/correspondence/<YYYY-MM-DD> from|to <who> - <subject>/` for letters and email attachments;
   - `files/evidence/` for client-supplied evidence;
   - `files/engagement/` for engagement letters and fee agreements;
   - `files/drafts/` for working drafts that matter (mark them DRAFT in the filename);
   - `files/notes/` for notes and memos.
3. Upload what is missing. Binary files: compute SHA-256 and size; `place_file_upload_begin({place_key, path, content_type, sha256, size_bytes})`; `curl -X PUT -H "Content-Type: <type>" --data-binary @file "<url>"`; `place_file_upload_confirm` with the same path, hash, and size. Get the `place_key` from the response to any `document_create` on the matter. Text and markdown: `document_create`. A new version of an existing path: `document_replace`.
4. For every binary filing or letter, also create a `.md` text extract next to it (pdftotext or pandoc), headed with the source filename, page count, and SHA-256, so findings can cite it.
5. Never file a document the user has said is privileged work product to a client-visible area; `document_create` and uploads land in the firm's own area by default. Mark sealed court filings CONFIDENTIAL in the filename.

### 4. Reconcile deadlines

1. `obligation_list({matter_id})`.
2. For each deadline in the inventory — a date in an email ("by 5:00pm PT Monday"), a rule-based period ("20 days after service", "14 calendar days from the commencement letter"), a contractual window ("60 days after termination"), or a commitment the firm made ("copies by FedEx on Monday") — check whether it is recorded with the right time.
3. Record what is missing with `obligation_record`:
   - `due_on` for an all-day date; `due_at` with an explicit UTC offset (e.g. `2026-11-16T17:00:00-08:00`) plus `time_zone` for a clock time — a bare local time is stored as UTC;
   - `kind` from the allowed list (answer, deposition, discovery_cutoff, expert_disclosure, hearing, ime, mediation, records_due, service, sol, trial, wc_reassignment) — use the nearest and say so in the narrative;
   - `narrative` quoting the deadline language and naming the source document or message;
   - `is_critical: true` for anything that forfeits a right if missed.
   A period whose anchor is not yet known (service not yet made) is recorded as a note in the narrative of a related obligation, or in the dated note, until the anchor exists.
4. Read `obligation_list` again and confirm the stored times. If an earlier entry is wrong, record the correct one and name the wrong one in its narrative (the connector has no retract).

### 5. Record decisions and status

Write one dated note, `files/notes/<YYYY-MM-DD> <topic>.md`, with: instructions and decisions people gave this session (who, when, what); the status of anything in progress (what is filed, what is awaiting approval, what is blocked and on whom); known gaps in the record (documents referenced but not on file); and anything the next session must not repeat. Write the people's words, not a summary of them, where the wording matters.

## Report

End with a short table: item, where it went (path or obligation id), or why it could not be filed and what would let it be. Anything that could not be filed is a gap the attorney needs to know about; do not leave it implicit.
