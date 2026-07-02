---
name: lovision-mcp
description: READ THIS BEFORE calling any Lovision MCP tool. Use whenever the Lovision MCP server is connected or the user mentions Lovision, lovision.ai, mcp.lovision.ai, Lovision Desktop Local MCP, a Lovision project/document/canvas, Blueprint design generation, project workspace operations, quality checks, preview/export, or asks an external agent to create, inspect, modify, or hand off native Lovision design work. Always initialize with lovision.session.init first, resolve the target before writing, read document context and capabilities before edits, prefer Blueprint for structured design work, run quality and preview after meaningful writes, and finish with lovision.session.finishWork.
---

# lovision-mcp

Lovision MCP lets external agents create, understand, edit, preview, export, and hand off real Lovision native designs. Treat it as a governed design workflow interface, not raw canvas CRUD: sessions, target resolution, permissions, capability profiles, Blueprint, quality checks, receipts, and recovery actions are all part of the contract.

The remote MCP endpoint is:

```text
https://mcp.lovision.ai/
```

Lovision MCP is Early Access. If OAuth does not start after connecting the server, the user may need Lovision to enable MCP for their workspace and provide the current authorization method plus `workspaceId`.

## First Call: `lovision.session.init`

On any new conversation or after `SESSION_NOT_FOUND`, call `lovision.session.init` before using other Lovision tools.

Pass:

- `runtime`: usually `remote-web`; use `desktop-local` for Lovision Desktop Local MCP; use `internal` only for trusted internal environments.
- `agentCredential`: when the host exposes an Agent Connection credential or bearer token.
- `target`: any known `workspaceId`, `projectId`, `documentId`, `pageId`, or `mode`.

Do not guess targets. If the user did not provide a clear workspace/project/document/page, use workspace/project tools to resolve the target or ask a short clarification before writing.

## Canonical Workflow

For design creation or meaningful edits:

```text
lovision.session.init
  -> target resolution / lovision.projects.open / lovision.document.create
  -> lovision.document.getContext + lovision.capabilities.list
  -> lovision.knowledge.* / fonts / tokens / components / media as needed
  -> lovision.blueprint.validate
  -> lovision.blueprint.apply or capability-gated advanced tools
  -> lovision.quality.check + lovision.preview.capture
  -> targeted repair if needed
  -> lovision.export or lovision.workflow.submit if requested
  -> lovision.session.finishWork
```

For read-only requests, stop after context, lookup, preview, quality, export, or project listing as appropriate, then call `finishWork` unless the user explicitly wants the session left open.

## Tool Routing

Session and lifecycle:

- `lovision.session.init`
- `lovision.session.status`
- `lovision.capabilities.list`
- `lovision.production.status`
- `lovision.session.finishWork`
- `lovision.session.abortWork`

Workspace and project operations:

- Projects: `lovision.projects.list`, `get`, `create`, `update`, `duplicate`, `move`, `delete`, `restore`, `export`, `open`
- Folders: `lovision.folders.list`, `create`, `update`, `delete`
- User metadata: `lovision.projectFavorites.set`, `lovision.projectRecents.list`
- Sharing: `lovision.projectSharing.get`, `updateLinkAccess`, `invite`, `revoke`, `updateRole`

Document, pages, viewport, and lookup:

- `lovision.document.create`
- `lovision.document.getContext`
- `lovision.lookup`
- Pages: `lovision.pages.list`, `switch`, `create`, `rename`, `duplicate`, `delete`, `getBackground`, `setBackground`
- Selection and viewport: `lovision.selection.set`, `lovision.viewport.fit`, `lovision.viewport.focus`
- Layout guides: `lovision.layoutGuides.getFrame`, `setFrame`, `addFrameGuide`

Design creation and editing:

- Blueprint: `lovision.blueprint.validate`, `lovision.blueprint.apply`
- Streaming design: `lovision.design.stream.start`, `applyChunk`, `finalize`, `abort`
- Advanced node editing: `lovision.nodes.apply`
- Text ranges: `lovision.textStyles.applyRange`
- Components: `lovision.components.list`, `lovision.components.get`, `lovision.componentSets.create`
- Schema lookup: `lovision.schema.get`

Design system data:

- Fonts: `lovision.fonts.summary`, `list`, `resolve`, `warm`
- Tokens: `lovision.tokens.summary`, `resolve`, `propose`, `commitProposal`
- Knowledge: `lovision.knowledge.list`, `lovision.knowledge.get`

Assets, media, and local files:

- Media acquisition: `lovision.media.acquire`
- Binary resources: `lovision.binary.create`, `read`, `dispose`
- Image insertion: `lovision.assets.insertImage`
- Workflow input resolution: `lovision.assets.resolveWorkflowInput`
- Local files: `lovision.localFiles.requestAccess`, `listGrants`, `read`, `revoke`

Quality and delivery:

- `lovision.quality.check`
- `lovision.preview.capture`
- `lovision.export`
- `lovision.workflow.submit`

## Hard Rules

1. **Use Host Facade backed writes.** Never invent payloads that depend on engine stores, Liveblocks internals, React state, renderer internals, raw patches, or `extraFields.data.config`.
2. **Prefer Blueprint for complex design work.** Use `lovision.blueprint.validate` and `lovision.blueprint.apply` for structured creation and patching.
3. **Use canonical public payload fields.** Text should use `text.content`, `text.layout.mode`, `text.baseStyle`, and `text.styleRanges`. Shapes/vectors/images should use fields such as `shape`, `shapeParams`, `cornerRadius`, `vectorPath`, and `imageRef`.
4. **Respect capability profile.** Discovery is not authorization. Trust `session.init`, `document.getContext`, and `capabilities.list` over guesses. Invocation-time enforcement is expected.
5. **Write safely.** Include `expectedVersion` and `idempotencyKey` when a write tool accepts them.
6. **Verify user-visible changes.** After meaningful writes, run `quality.check` and `preview.capture`; patch actionable findings when practical.
7. **Do not roll back committed mutations because delivery failed.** Preview, export, workflow, or asset-staging failures should produce a safe stopping point and resume action.
8. **Preserve media provenance.** Use Lovision-generated or workspace media first, licensed fallback second, explicit placeholder last. Keep unresolved media findings visible.
9. **Resolve fonts and tokens before relying on them.** Use `lovision.fonts.*` and `lovision.tokens.*`.
10. **Do not read arbitrary local paths.** Desktop Local MCP file access must go through `lovision.localFiles.*` grants. Remote MCP cannot read local files.
11. **Finish non-trivial sessions.** Use `lovision.session.finishWork` with changed nodes, artifacts, unresolved findings, and resume action. Use `abortWork` only for cancellation or unrecoverable setup failure.

## Capability And Error Handling

Maturity tiers:

- `unknown`
- `claimed`
- `verified`
- `trusted-internal`

Runtime values:

- `remote-web`
- `desktop-local`
- `internal`

Common structured errors:

- `PERMISSION_DENIED`: explain the missing permission or ask the user to re-authorize.
- `CAPABILITY_NOT_AVAILABLE`: pick a supported fallback or report the limitation.
- `REQUIRES_AGENT_MATURITY`: do not retry blindly; explain that this agent/client is not trusted enough for that tool.
- `APPROVAL_REQUIRED`: surface the approval request and wait for the user/client.
- `RATE_LIMITED` or `QUOTA_EXCEEDED`: stop at a safe point and include a resume action.
- `SESSION_NOT_FOUND`: call `lovision.session.init` again.

## Media Ladder

When the design needs media:

1. Use Lovision generation or an existing Lovision workspace/product asset.
2. Use a licensed external fallback only if generation/workspace asset is unavailable or unsuitable.
3. Use an explicit placeholder only as a last resort, and include a quality finding or handoff note.

Supported binary image MIME types currently include `image/png`, `image/jpeg`, `image/webp`, and `image/svg+xml`. `lovision.binary.create` is capped at 10 MB.

## Setup Notes

Remote MCP config:

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

Claude Code manual setup:

```bash
claude mcp add --transport http lovision https://mcp.lovision.ai/
```

Codex manual setup:

```bash
codex mcp add lovision --url https://mcp.lovision.ai/
codex mcp login lovision
```

Lovision Desktop Local MCP uses the loopback URL and bearer secret shown in Lovision Desktop `Settings > MCP`. Do not hand-write a Desktop config without the copied `Authorization` header.
