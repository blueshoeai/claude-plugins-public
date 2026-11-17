---
name: blueshoe-matter-work
description: Use whenever a task touches a matter the firm keeps in Blueshoe — drafting a filing or letter, answering a question about a case, reviewing an email from the other side, calendaring a deadline, filing a document, preparing for a call. Covers starting from the Blueshoe record, keeping it current, and the firm's standing rules.
---

# Working a Blueshoe matter

Blueshoe is the firm's system of record. Each matter holds documents, an intelligence layer (findings with verbatim cites, a chronology, gaps, contradictions), deadlines (obligations), tasks, correspondence, and a generated brief. Start every matter task from that record, and before the task ends make sure everything the task touched or produced is in it (the `capture-to-blueshoe` skill is the checklist for that).

All operations run through the Blueshoe connector's `execute` tool (a JavaScript body with `await ops.<operation>(args)`). Find an operation's exact arguments with `search` before calling it. `references/operations.md` lists the operations this skill relies on.

## 1. Start from the record

Before drafting, advising, or answering anything about a matter:

1. `matter_list` → the matter id (a UUID; never pass a title). `matter_team_get({matter_id})` → who the attorney of record and the team are; `matter_get` → parties and open obligations.
2. `brief_get({matter_id})` — the platform's standing paragraph. Check `generated_at`; it can lag the file by days.
3. `projection_get({matter_id, slug:'chronology'})` — every dated event with its cites. Build timelines from this, not from rereading the file.
4. `finding_list({matter_id})` — the held findings. For any fact that will go into a document, open `finding_history({matter_id, finding_id})` and use its `cites` (file id, page, verbatim quote) as the source. Treat the layer's cautious glosses ("records the proposed allocation; does not state the shares were issued") as the ceiling on what the record supports.
5. `projection_get` with `gaps` and `contradictions.open` — what the record says is missing and where it disagrees with itself. Put these in front of the attorney; do not resolve them silently.
6. `obligation_list({matter_id})` and `projection_get({matter_id, slug:'deadlines.upcoming'})` — what is due. Hand-entered deadlines may not appear in the projection; read the list.
7. `document_list({matter_id})` and `correspondence_list({matter_id})` — what is actually on file. Read primary documents with `document_get` (text) or `document_revision_get` (a short-lived download URL for PDFs and Word files; fetch it with curl).
8. Read any dated notes under `files/notes/` — that is where earlier sessions left standing rules, decisions, and status for this matter — and the summaries at the top of `files/sessions/`, which say what earlier sessions did and left open.

Findings are for orientation and sourcing, never for authority: every fact in a filing traces to a document that has been read. When a finding is wrong, record it with `finding_feedback_file` in the attorney's own words, while the attorney is in the conversation.

## 2. Write back, the same day

Run the `capture-to-blueshoe` checklist at the end of every matter task. In short:

- **Every final or filed document** goes to the matter as the actual file (`place_file_upload_begin` → PUT → `place_file_upload_confirm`) plus a text extract (`document_create`) so the layer can read it.
- **Every email to or from the other side, the client, a court, or a tribunal** is filed with its attachments (`correspondence_file`, or by cc'ing the matter's intake address).
- **Every deadline read from any document** is recorded with `obligation_record` the moment it is read, with the source quoted in `narrative`, `due_on` for all-day dates, and `due_at` with an explicit UTC offset plus `time_zone` for clock times.
- **Rules, decisions, and status** the next session needs go in a dated note under `files/notes/`.
- **The session itself** — summary and transcript — goes to `files/sessions/` (step 6 of the capture checklist).

If urgency forces a step to be skipped, say so in the reply and finish it before the session ends.

## 3. Firm standing rules

These are generic to any firm using Blueshoe. Firm-specific values — who the attorney of record is, fee terms, approval thresholds — come from the matter's team (`matter_team_get`), the signed engagement letter on file, and the firm's notes; never assume them.

- **Only a licensed attorney on the matter team signs and sends.** Filings, and anything addressed to opposing counsel, a tribunal, or a court, go out under the attorney of record's name and signature. Staff who are not admitted in the relevant jurisdiction (including law graduates awaiting admission) draft and prepare but do not sign, appear, or speak for the client — many jurisdictions treat representation in arbitration as the practice of law regardless of what the forum's own rules allow. Signature blocks carry name, bar number, address, telephone, and email; check the forum's rule for what it requires.
- **The attorney approves every client-facing recommendation.** Draft the email for the attorney to send; leave the recommendation sentence for the attorney to write.
- **Quote adverse parties and their counsel verbatim**, with exhibit cites. Never paraphrase or characterize their words.
- **Hedge unverified facts** with "on information and belief." Nothing in a sworn document rests on a fact the client has not confirmed.
- **Read every cited authority in full against the source before citing it**, and check that it supports the stated proposition. Keep a table of authorities (citation, pin, proposition) for the attorney; the duty of candor to the tribunal sits with the signer.
- **Plain, non-formal tone; nothing superfluous; no AI phrasing.**
- **Before filing, review for self-damaging material** in the client's own statements and remove what hurts without changing the facts. Quotes already in a signed declaration stay.
- **Sealed or confidential filings** are described, not quoted or attached, unless the attorney decides otherwise.
- **Clients do not respond to the adverse party directly**; everything is forwarded to counsel.
- **Attach editable files only when asking for edits**, and say so in the first line. Download every attachment the attorney sends back; comments often live inside the file. Address each comment and report item by item.

## 4. Engagement-letter checks

Read the signed engagement letter on the matter (usually under `files/engagement/`) before spending money or changing posture:

- Costs above the letter's per-item approval threshold need the client's written approval first. Say so before any filing fee is paid.
- Note any fee-tier change the action triggers (many contingent letters change the percentage when a demand or complaint on the merits is filed) and make sure the attorney tells the client first.
- Work outside the engagement's scope — another forum, defense of a claim against the client — needs a written addendum.
- If there is no engagement letter on file, say so; that is a gap to close before the work proceeds.
