# Setup checklist: Email MCP for Inbox Zero

Complete these steps **before** using the skill. The agent cannot triage a live inbox without an email MCP connected in Cursor.

## Required capabilities

Your MCP must expose tools that cover at least:

| Capability | Purpose | Example tool name patterns |
|------------|---------|----------------------------|
| List / search | Fetch unread or recent mail | `list`, `search`, `inbox` |
| Read | Full message + thread context | `get`, `read`, `fetch` |
| Label / categorize | Apply labels or categories | `label`, `modify`, `categorize` |
| Archive | Move out of inbox | `archive`, `modify` |
| Draft (optional but recommended) | Create reply drafts | `draft`, `create_draft` |

Send mail is **optional** and off by default in preferences (`send_replies: false`).

## 1. Choose an email MCP

Pick one path:

### Option A — Gmail (or Google Workspace) MCP

- [ ] Install a Gmail-compatible MCP server that Cursor can run (official or community).
- [ ] Complete Google OAuth for the mailbox to triage.
- [ ] Confirm scopes include read + modify (labels/archive). Avoid broad send scope unless you will enable `send_replies`.

### Option B — Make (or similar) email scenarios as MCP tools

- [ ] Connect Gmail / Outlook in Make (or equivalent).
- [ ] Create **on-demand** scenarios whose names include the capability keywords above (e.g. `email_list_unread`, `email_get_message`, `email_apply_label`, `email_archive`, `email_create_draft`).
- [ ] Turn each scenario **ON**.
- [ ] Connect Make MCP in Cursor (see Make’s Cursor docs). Same pattern as calendar MCP: OAuth or scoped token URL.

### Option C — Other provider MCP (Outlook, Fastmail, etc.)

- [ ] Add the provider’s MCP to Cursor.
- [ ] Verify the five capabilities in the table above exist under some tool names.

## 2. Add the server in Cursor

- [ ] Open **Cursor Settings → Tools & MCP**
- [ ] **Add Custom MCP** (or edit `mcp.json`)
- [ ] Add your email MCP server entry (command or URL — follow that MCP’s docs)
- [ ] If the server shows **Needs login** / auth error, complete OAuth or paste the API token
- [ ] Confirm the server status is ready and tools are listed

Do not commit API tokens or OAuth secrets into the skills repo. Keep credentials in user-level MCP config only.

Example shape (replace with your server’s real config):

```json
{
  "mcpServers": {
    "email": {
      "command": "npx",
      "args": ["-y", "your-email-mcp-package"],
      "env": {
        "EMAIL_API_KEY": "set-in-your-local-env-not-in-git"
      }
    }
  }
}
```

Or for a remote MCP:

```json
{
  "mcpServers": {
    "email": {
      "url": "https://your-email-mcp.example/mcp"
    }
  }
}
```

## 3. Smoke test

In Cursor **Agent** mode:

- [ ] Ask: “Using my email MCP, list my unread messages.”
- [ ] Confirm real subjects/senders return (not invented).
- [ ] Ask to apply a test label or archive a throwaway message, then verify in the mailbox UI.
- [ ] Optional: create a draft reply and confirm it appears in Drafts.

## 4. Preferences (recommended)

- [ ] Copy [preferences.example.yaml](preferences.example.yaml) to `inbox-zero.preferences.yaml` (workspace or home).
- [ ] Set `important_contacts`, work hours, and keep `send_replies: false` until you trust drafts.

When smoke tests pass, the [inbox-zero](SKILL.md) skill can run daily briefing and triage against the live inbox.
