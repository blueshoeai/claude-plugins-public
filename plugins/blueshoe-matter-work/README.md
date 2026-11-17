# Blueshoe Matter Work

Teaches Claude to work a legal matter from the Blueshoe record and — above all — to make sure everything a session reads or produces ends up in Blueshoe. Firm-agnostic: who the attorney of record is, fee terms, and approval thresholds are read from the matter itself, never assumed.

## Skills

- **blueshoe-matter-work** — loads on any matter task. The start-of-matter sequence (team → brief → chronology → findings with cites → gaps → contradictions → deadlines → documents → notes), the write-back rule, standing rules any firm would apply (licensed attorney signs and sends; verbatim quotation of adverse parties; every authority read before it is cited; no AI phrasing; sealed filings described not quoted), and engagement-letter checks before money is spent or posture changes. `references/operations.md` lists every Blueshoe operation used and the connector's current limits.
- **capture-to-blueshoe** — the end-of-task checklist and the "is this in Blueshoe?" action. Inventories the session's emails, documents, deadlines and decisions; reconciles each against `correspondence_list`, `document_list`, `obligation_list`; files what is missing (documents as the real file plus a text extract; emails with attachments; deadlines with explicit time zones; a dated note for decisions and status); reports what could not be filed.
- **matter-brief** — "brief me on X", "what's due this week on X": one-page sourced status from the record with finding ids.

## Session export

The capture skill ends by writing the session itself (summary and, where the client allows, the full transcript) to `files/sessions/` on the matter. That is the copy a person reads. For a complete, mechanical copy of every chat and session, a firm on Claude Enterprise should pull Anthropic's Compliance API into Blueshoe; the skill does not replace that.

## Connector

The plugin bundles no connector configuration. It uses the Blueshoe connector the firm's admin has enabled org-wide, plus whatever mail connector is enabled (see CONNECTORS.md). No environment variables, no placeholders to fill in.

## Installing

- **Firm-wide (claude.ai chat, Cowork, synced Claude Code):** an Owner adds the marketplace repository under Organization settings > Plugins & skills > Add > Sync from GitHub, then sets the plugin's availability (Available, Installed by default, or Required). Members get updates on their next session after each sync.
- **Claude Code directly:** `/plugin marketplace add blueshoeai/claude-plugins-public` then `/plugin install blueshoe-matter-work@blueshoe`.
- **Single user, no catalog:** zip this folder as `blueshoe-matter-work.plugin` and add it under Customize > Plugins.

## Updating

Edit the skills, bump `version` in `.claude-plugin/plugin.json`, merge to the default branch. Org-synced marketplaces pick it up on sync; Claude Code marketplaces with auto-update pick it up on next start.

## Session export

The capture skill ends by writing the session itself (summary and, where the client allows, the full transcript) to `files/sessions/` on the matter. That is the copy a person reads. For a complete, mechanical copy of every chat and session, a firm on Claude Enterprise should pull Anthropic's Compliance API into Blueshoe; the skill does not replace that.

## Connector limits the skills work around (Nov 2026)

- Memory and playbook pages are run-bound (not reachable from the connector); standing rules live here and in dated notes under `files/notes/` on each matter.
- `obligation_record` stores a bare local `due_at` as UTC; the skills always pass an explicit offset.
- No retract/supersede for obligations; a wrong entry is corrected by recording the right one and naming the wrong one in its narrative.
- `mail_search`/`mail_file` read the Blueshoe agent's own mailbox, not the user's; the capture skill uses the user's mail connector and `correspondence_file`.
