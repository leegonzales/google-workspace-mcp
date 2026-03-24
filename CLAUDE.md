# google-workspace-mcp

> Google Workspace MCP server -- Gmail, Docs, Sheets, Slides, Drive, Calendar, and Chat from the command line.

## Ecosystem Context
See `~/.claude/ecosystem-map.md` for cross-repo relationships.

## Quick Reference
- **Stack:** TypeScript / Node.js (MCP SDK)
- **Upstream:** `gemini-cli-extensions/workspace`
- **Status:** active (fork with multi-account support)
- **Org:** leegonzales

## Fork Changes (vs upstream)

### Multi-Account Profile Support

Upstream hardcodes a single keychain service name, so only one Google account can be authenticated at a time. Our fork adds `WORKSPACE_PROFILE` env var support.

**How it works:**
- No env var → keychain service: `gemini-cli-workspace-oauth` (default, backward-compatible)
- `WORKSPACE_PROFILE=personal` → keychain service: `gemini-cli-workspace-oauth-personal`
- `WORKSPACE_PROFILE=anything` → keychain service: `gemini-cli-workspace-oauth-anything`

Each profile gets its own credential slot in macOS Keychain, so multiple Google accounts coexist.

**Files changed:**
- `workspace-server/src/utils/config.ts` — added `profile` to `WorkspaceConfig`, reads `WORKSPACE_PROFILE`
- `workspace-server/src/auth/token-storage/oauth-credential-storage.ts` — keychain service name derived from profile

### Claude Code MCP Configuration

In `~/.claude/settings.json`, configure one server entry per account:

```json
{
  "mcpServers": {
    "google-workspace-personal": {
      "command": "node",
      "args": ["<path>/workspace-server/dist/index.js"],
      "env": { "WORKSPACE_PROFILE": "personal" }
    }
  }
}
```

The default (no profile) instance is used for Catalyst (`lee@catalystai.services`). The `personal` profile is for `lee.gonzales@gmail.com`.

**First-time auth:** On first tool call, the server opens a browser for OAuth consent. Sign in with the appropriate Google account.

### Syncing with Upstream

```bash
git fetch upstream
git merge upstream/main
npm run build
```

Profile support is isolated to two files, so merge conflicts should be rare.
