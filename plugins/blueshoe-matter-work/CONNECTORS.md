# Connectors

This plugin bundles no connector configuration. It uses connectors the firm has already enabled.

| Connector | Required | Notes |
|---|---|---|
| Blueshoe | Yes | The firm's Blueshoe MCP connector, enabled org-wide by an admin (Blueshoe supplies the URL, `https://mcp.blueshoe.com/?org=<firm slug>`). The skills call its `execute`, `search` and `context_get` tools. If more than one Blueshoe organization is available, `context_get` lists them; use the one for the firm. |
| Mail | For email capture | Whatever mail connector is enabled (Gmail, Microsoft 365). Used to find and file matter emails. |
| Calendar | Optional | For scheduling calls; not used for deadlines, which are recorded in Blueshoe. |

Nothing in the plugin is firm-specific. Who the attorney of record is, fee terms, and approval thresholds come from the matter's team, the engagement letter on file, and dated notes under `files/notes/`.
