---
name: amy-connectors
description: "Google Calendar, Drive, Docs, Sheets, Slides, Forms, Gmail, Notion, and Canva through Amy Connectors MCP with Nango OAuth."
version: 1.0.0
author: AmyBot
license: MIT
metadata:
  hermes:
    tags: [Amy, AmyBot, Nango, MCP, Google, Calendar, Drive, Docs, Sheets, Slides, Forms, Gmail, Notion, Canva, OAuth]
---

# Amy Connectors

Use this skill for connected SaaS tools exposed through the Amy Connectors MCP server. Amy Connectors stores OAuth tokens in Nango. Do not look for `~/.hermes/google_token.json`, do not run the legacy Google Workspace setup flow, and do not ask the user for a Google desktop OAuth client when the Amy Connectors MCP tools are available.

## Routing Rules

- For Google Calendar, use `mcp_amy_connectors_calendar_*` tools.
- For Google Drive, use `mcp_amy_connectors_drive_*` tools.
- For Google Sheets, use `mcp_amy_connectors_sheets_*` tools.
- For Google Docs, use `mcp_amy_connectors_docs_*` tools.
- For Google Slides, use `mcp_amy_connectors_slides_*` tools.
- For Google Forms, use `mcp_amy_connectors_forms_*` tools.
- For Gmail, use `mcp_amy_connectors_gmail_*` tools only if Gmail is enabled in the portal.
- For Notion, use `mcp_amy_connectors_notion_*` tools.
- For Canva, use `mcp_amy_connectors_canva_*` tools.

## Calendar Examples

When the user asks to check their calendar, call `mcp_amy_connectors_calendar_list_events` with the requested time window. If the user does not provide a time window, use a practical default such as today or the next 7 days and mention the range you checked.

When creating or changing events, use the Calendar MCP mutation tools and summarize exactly what changed.

## Connection Errors

If an Amy Connectors tool says no enabled connection exists, tell the user the connector is not enabled in the Amy Portal Integrations page or the Hermes MCP runtime is reading the wrong connector state path. Do not say the problem is `~/.hermes/google_token.json`; that file belongs to the disabled legacy `google-workspace` skill.

If the MCP server itself is unavailable, tell the user to restart Hermes and verify `mcp_servers.amy_connectors` in `~/.hermes/config.yaml`.
