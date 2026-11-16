# Blueshoe operations this skill relies on

All calls go through the connector's `execute` tool: the body of an async function with `ops` in scope; return a JSON value. Use `search` with the operation's exact name to get its full declaration before the first call.

## Orientation (read)

| Operation | Use |
|---|---|
| `matter_list` | Find the matter; take `matter_id` from the row. |
| `matter_get({matter_id})` | Parties, assignments, open obligations, next due. |
| `brief_get({matter_id})` | Generated standing paragraph; check `generated_at`. |
| `projection_get({matter_id, slug})` | `chronology`, `gaps`, `contradictions.open`, `deadlines.upcoming`, `record.activity`, `citation.detail` (params: file_id, page, quote). |
| `finding_list({matter_id, kind?, status?})` | Held findings; up to 500. Kinds seen: instrument, event, case_event, state, records_gap, demand_made, norm, position_taken, delay_observed. |
| `finding_history({matter_id, finding_id})` | One finding's cites (file, page, verbatim), edges, closing. |
| `document_list({matter_id})` | Paths, sizes, extraction status, summaries. |
| `document_get({matter_id, path})` | Text content, or a download URL for binaries. |
| `document_revision_list` / `document_revision_get` | A specific past version. |
| `obligation_list({matter_id})`, `obligation_upcoming({matter_id})` | Deadlines of record. |
| `case_date_list`, `matter_milestone_list` | Court dates and milestones (separate objects from obligations). |
| `event_list({matter_id, grain})` | Activity feed; `detail` grain for everything. |
| `task_list`, `task_get` | Open tasks. |

## Write-back

| Operation | Use |
|---|---|
| `document_create({matter_id, filename, content, content_type})` | Text/markdown only. Folder paths in `filename` work (`files/notes/...`). The response includes `place_key`. |
| `place_file_upload_begin({place_key, path, content_type, sha256, size_bytes})` → PUT bytes to `url` → `place_file_upload_confirm(same args)` | Binary files (PDF, Word, images). URL is valid 15 minutes. |
| `document_replace` | New revision of an existing path. |
| `correspondence_file({matter_id, messages:[{raw (base64 RFC 5322), provider_id}]})` | File email with attachments; ≤25 per call, ≤~7 MB each. |
| `obligation_record({items:[...]})` | Deadlines. `due_on` (date) or `due_at` with explicit offset + `time_zone`. Kinds: answer, deposition, discovery_cutoff, expert_disclosure, hearing, ime, mediation, records_due, service, sol, trial, wc_reassignment. |
| `obligation_mark_met` | When satisfied. |
| `case_date_record`, `matter_milestone_record` | Court dates; milestones. |
| `matter_task_create({items:[...]})` | Tasks with `basis` pointing at findings, files, or events. |
| `finding_feedback_file({items:[{matter_id, finding_id, stance, message}]})` | Dispute or confirm a finding, in the attorney's words; attended sessions only. |
| `file_note_create` | Record what a person said (call notes); attended sessions only. |
| `entity_add` | Add people/organizations (opposing counsel, Delaware counsel) to the directory. |

## Not available from the connector (as of Nov 2026)

- `memory_page_write` / playbook pages — run-bound; use a dated note under `files/notes/` instead.
- Retract or supersede an obligation — record the correct entry with a narrative naming the wrong one.
- `mail_search` / `mail_file` read the Blueshoe agent's own mailbox, not the user's Gmail.

## Known defects to work around

- `obligation_record` with a bare local `due_at` stores it as UTC. Always pass an offset.
- `deadlines.upcoming` shows only anchored/computed obligations; read `obligation_list` for hand-entered ones.
