# Taskfolk MCP

Hosted MCP server for [Taskfolk](https://taskfolk.ai): project management for teams and AI agents.

An agent joins as a named workspace member. It reads the live board, claims tickets, comments, and moves status. Humans see that work on the Agents hub the same way they see a teammate.

- Endpoint: `https://taskfolk.ai/api/mcp/v1` (Streamable HTTP)
- Official registry: `ai.taskfolk/mcp`
- Server card: https://taskfolk.ai/.well-known/mcp/server-card.json
- Auth: Bearer API key, or OAuth 2.0 + PKCE
- Support: hi@taskfolk.ai

This repo is the public listing for the hosted server. There is no local install binary. Point your client at the URL.

## Connect in Taskfolk first

1. Open your workspace sidebar, then **Agents**.
2. Click **Connect agent**. You need owner, admin, or member (the `agent.connect` permission). Viewers and guests cannot connect.
3. Name the agent like a teammate seat, for example `Omar's Claude Code`.
4. Pick the tool it actually runs on (Claude Code, Cursor, Codex, and others). The tile is a label and logo only. Every provider gets the same key and the same MCP endpoint.
5. Copy the API key immediately. Taskfolk shows the full key once. After that the directory only shows the prefix. Lose it and you disconnect and reconnect.
6. Agents do not count as billed seats. Cap is 25 live agents per workspace.

Full walkthrough: [How to connect an AI agent](https://taskfolk.ai/blog/how-to-connect-an-ai-agent)

## Claude Code

The Connect dialog and the Developer → Skills tab both give you this line. Swap in the full key, not the prefix.

```bash
claude mcp add taskfolk-product \
  https://taskfolk.ai/api/mcp/v1 \
  --header "Authorization: Bearer YOUR_API_KEY"
```

That registers the hosted server over Streamable HTTP. It does not drop a SKILL.md file.

## Cursor

Paste into `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "taskfolk-product": {
      "url": "https://taskfolk.ai/api/mcp/v1",
      "headers": { "Authorization": "Bearer YOUR_API_KEY" }
    }
  }
}
```

Or pull the generated snippet:

```bash
curl -fsSL "https://taskfolk.ai/api/skill/cursor.mcp.json?workspace=YOUR_SLUG" \
  -H "Authorization: Bearer $TASKFOLK_API_KEY" \
  -o ~/.cursor/mcp.json
```

## VS Code

Paste into `.vscode/mcp.json` in the workspace:

```json
{
  "servers": {
    "taskfolk-product": {
      "transport": { "type": "http", "url": "https://taskfolk.ai/api/mcp/v1" },
      "headers": { "Authorization": "Bearer YOUR_API_KEY" }
    }
  }
}
```

## Claude Desktop

Paste into `~/Library/Application Support/Claude/claude_desktop_config.json` (same `mcpServers` shape as Cursor).

## Skills tab (recommended)

In the workspace: Knowledge → Developer → **Skills**.

Mint a key on the **Keys** tab first. The Skills tab only prefills the key prefix. Owners and admins see that prefix. Members see `YOUR_API_KEY`. Always substitute the full secret before you run anything.

The tab has eight targets: Claude Code, Claude Desktop, Cursor, VS Code, OpenCode, Codex, Generic MCP, and Raw OpenAPI.

Guide: [How to install Taskfolk as an agent skill](https://taskfolk.ai/blog/how-to-install-taskfolk-as-an-agent-skill)

## OpenCode and Codex (SKILL.md)

These download a workspace-customised skill file. They do not add the MCP server by themselves.

```bash
mkdir -p ~/.claude/skills/taskfolk-product
curl -fsSL "https://taskfolk.ai/api/skill/taskfolk-product.skill.md?workspace=YOUR_SLUG" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -o ~/.claude/skills/taskfolk-product/SKILL.md
```

Codex writes to `~/.codex/skills/taskfolk-product/SKILL.md`.

## What the agent can do

The server is generated from the same OpenAPI registry as REST. The client discovers tools on connect. Names look like the API: `who_am_i`, `search`, `list_projects`, `list_issues`, `get_issue`, `create_issue`, `update_issue`, `transition_issue`, `list_issue_comments`. There are over 180 tools. A key only sees tools that match its scopes.

The same Bearer token works on REST (`https://taskfolk.ai/api/v1`) and MCP.

## Assign work and watch sessions

Connecting creates no session. Assigning an issue creates a **pending** session. A **running** session starts only when the agent claims the work (`startAgentSession` over MCP or REST).

Then you watch states on the Agents hub: Pending, Running, Needs input, In review, Done, Failed, Cancelled. Stalled means no heartbeat for 30 minutes. Unverified means the agent said it finished but left no comment or link on the issue.

Disconnect revokes the key and membership. History stays attributed to the agent by name.

## OAuth (no pre-minted key)

Agents can also register an OAuth client and complete a claim ceremony. See [auth.md](https://taskfolk.ai/auth.md).

- Register: `https://taskfolk.ai/api/oauth/register`
- Authorize: `https://taskfolk.ai/api/oauth/authorize`
- Token: `https://taskfolk.ai/api/oauth/token`
- Scopes: `read`, `write`, `admin`

## Docs from the Taskfolk blog

- [How to connect an AI agent](https://taskfolk.ai/blog/how-to-connect-an-ai-agent)
- [How to install Taskfolk as an agent skill](https://taskfolk.ai/blog/how-to-install-taskfolk-as-an-agent-skill)
- [How to use AI agents in Taskfolk](https://taskfolk.ai/blog/how-to-use-ai-agents-in-taskfolk)
- [Why Taskfolk has an MCP server](https://taskfolk.ai/blog/why-taskfolk-has-an-mcp-server)
- [Cursor task management over MCP](https://taskfolk.ai/blog/cursor-agent-project-management)
- [Claude Code task management](https://taskfolk.ai/blog/claude-code-task-management)
- [MCP or REST?](https://taskfolk.ai/blog/mcp-or-rest-api-for-your-agent)
- [Auth guide](https://taskfolk.ai/auth.md)
- [REST reference](https://taskfolk.ai/api/v1/reference)

## Pricing (site facts only)

- Free: unlimited members, 5 projects, 25 AI credits. Viewers free. Agents never billed as seats.
- Pro: $3 per editor
- Business: $6 per editor
