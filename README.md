# Lovision MCP Skills

> Create, edit, inspect, preview, export, and hand off native Lovision designs from external agents.

This package is shaped like an external agent skill/plugin repository. It bundles:

- [`lovision-mcp`](./skills/lovision-mcp/SKILL.md): the agent skill that teaches Claude, Cursor, Codex, Gemini CLI, VS Code, and other MCP-capable agents how to use Lovision MCP safely.
- [`.mcp.json`](./.mcp.json): the remote Lovision MCP server config.
- [`.claude-plugin/`](./.claude-plugin/), [`.cursor-plugin/`](./.cursor-plugin/), and [`gemini-extension.json`](./gemini-extension.json): editor-native plugin manifests.

Every install path should surface the same Markdown skill under [`skills/`](./skills/) and the same MCP endpoint:

```text
https://mcp.lovision.ai/
```

Lovision MCP is currently Early Access. If your client does not show a Lovision / OneAuth authorization flow after adding the server, confirm that Lovision has enabled MCP for your account and has provided the required authorization method.

## Install

### Claude Code plugin

```bash
claude /plugin marketplace add choice-form/lovision-mcp-skills
claude /plugin install lovision
```

The plugin registers the `lovision` MCP server from [`.mcp.json`](./.mcp.json) and exposes the skill docs under [`skills/`](./skills/).

### Claude Code skills only

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

The extension manifest registers Lovision MCP and enables OAuth.

### Codex

Configure MCP:

```bash
codex mcp add lovision --url https://mcp.lovision.ai/
codex mcp login lovision
```

Then add the skill with the skills CLI if your Codex environment supports it:

```bash
npx skills add choice-form/lovision-mcp-skills --skill lovision-mcp -a codex
```

### Other agents

For hosts that do not understand plugin manifests, either use the skills CLI for that host or include the skill directly in the agent prompt:

```markdown
@include skills/lovision-mcp/SKILL.md
```

For raw MCP clients, use [`.mcp.json`](./.mcp.json) as the server config.

## First Test

After connecting, ask the agent:

```text
Use the Lovision MCP server. Initialize a Lovision session with runtime "remote-web". If projectId "<projectId>" is available, open that project first; otherwise list accessible projects and ask me which one to open. Then list available capabilities and summarize the project or page context. Do not modify the design yet.
```

For a write smoke test after you confirm the target:

```text
Use Lovision MCP with projectId "<projectId>" to create a small frame named "MCP smoke test" on the current page, run a quality check, capture a preview, and summarize what changed.
```

The agent should call `lovision.session.init` first and should use `lovision.capabilities.list` plus `lovision.document.getContext` before making edits.

## Related Product Docs

- Lovision MCP overview: `packages/docs/content/docs/mcp/overview.mdx`
- Lovision MCP getting started: `packages/docs/content/docs/mcp/getting-started.mdx`
- Lovision MCP tools: `packages/docs/content/docs/mcp/tools.mdx`
