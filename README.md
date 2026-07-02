# Lovision MCP Skills

> Create, edit, inspect, preview, export, and hand off native Lovision designs from external agents.

This package teaches Claude, Cursor, Codex, Gemini CLI, and other MCP-capable agents how to use Lovision MCP correctly. It bundles:

- [`lovision-mcp`](./skills/lovision-mcp/SKILL.md): the agent skill — canonical session-first workflow, tool routing, hard rules, and error handling.
- [`.mcp.json`](./.mcp.json): the remote Lovision MCP server config.
- [`.claude-plugin/`](./.claude-plugin/), [`.cursor-plugin/`](./.cursor-plugin/), and [`gemini-extension.json`](./gemini-extension.json): editor-native plugin manifests.

MCP endpoint:

```text
https://mcp.lovision.ai/
```

Lovision MCP is currently Early Access. If your client does not show a Lovision / OneAuth authorization flow after adding the server, confirm that Lovision has enabled MCP for your workspace and has provided the required `workspaceId` and authorization method.

## Install

### Claude Code plugin

```bash
claude /plugin marketplace add choice-form/lovision-mcp-skills
claude /plugin install lovision
```

The plugin registers the `lovision` MCP server and exposes the skill docs under [`skills/`](./skills/).

### Claude Code — skills only

Use this when Lovision MCP is already configured and you only want the skill instructions:

```bash
npx skills add choice-form/lovision-mcp-skills --skill lovision-mcp -a claude-code
```

Manual MCP setup:

```bash
claude mcp add --transport http lovision https://mcp.lovision.ai/
```

Then run `/mcp` in Claude Code, select `lovision`, and complete the Lovision / OneAuth authorization flow.

### Cursor plugin

```text
/add-plugin choice-form/lovision-mcp-skills
```

Cursor reads [`.cursor-plugin/plugin.json`](./.cursor-plugin/plugin.json), installs the skill, and registers the MCP server.

Skills-only fallback:

```bash
npx skills add choice-form/lovision-mcp-skills --skill lovision-mcp -a cursor
```

Manual Cursor MCP config:

```json
{
  "mcpServers": {
    "lovision": {
      "type": "streamable-http",
      "url": "https://mcp.lovision.ai/"
    }
  }
}
```

### Gemini CLI

```bash
gemini extensions install https://github.com/choice-form/lovision-mcp-skills
gemini /mcp auth lovision
```

### Codex

```bash
codex mcp add lovision --url https://mcp.lovision.ai/
codex mcp login lovision
```

Skills-only:

```bash
npx skills add choice-form/lovision-mcp-skills --skill lovision-mcp -a codex
```

### Other agents

For hosts that do not understand plugin manifests, include the skill directly in the agent prompt:

```markdown
@include skills/lovision-mcp/SKILL.md
```

For raw MCP clients, use [`.mcp.json`](./.mcp.json) as the server config.

## First Test

After connecting, ask the agent:

```text
Use the Lovision MCP server. Initialize a Lovision session with runtime "remote-web" and target.workspaceId "<workspaceId>". If projectId "<projectId>" is available, open that project first; otherwise list accessible projects and ask me which one to open. Then list available capabilities and summarize the project or page context. Do not modify the design yet.
```

For a write smoke test after you confirm the target:

```text
Use Lovision MCP with target.workspaceId "<workspaceId>" and projectId "<projectId>" to create a small frame named "MCP smoke test" on the current page, run a quality check, capture a preview, and summarize what changed.
```

The agent should call `lovision.session.init` first and use `lovision.capabilities.list` plus `lovision.document.getContext` before making edits.

## Maintenance

This repo is the published distribution of `distributions/lovision-mcp-skills/` in the [Lovision monorepo](https://github.com/choice-form/lovision). Changes to the skill or plugin configs are made there and automatically synced here via GitHub Actions on every merge to `main`.

Do not edit files in this repo directly — changes will be overwritten on the next sync.

## Related

- [Lovision MCP overview](https://lovision.ai/docs/mcp/overview)
- [Lovision MCP getting started](https://lovision.ai/docs/mcp/getting-started)
- [Lovision MCP tools](https://lovision.ai/docs/mcp/tools)
