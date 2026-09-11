---
name: playwright-mcp-setup
description: Use when the user wants Playwright MCP set up to drive their own Chrome (not a throwaway Chromium) with Claude Code. Triggers include "set up playwright mcp", "playwright extension server", "let claude drive my browser", "browser tools with my logins/sessions", or a 401/blocked page that needs the user's real profile.
---

# Playwright MCP (extension server) setup

Two Playwright MCP servers exist and behave differently:

- **Bundled-browser server**: the default from the official Playwright plugin. Launches its own throwaway Chromium. The user's logins, profile, and tabs are absent.
- **Extension server** (`--extension`): drives the user's real Chrome through the "Playwright MCP Bridge" extension, so their profile, sessions, and tabs are all there.

This skill sets up the **extension server**. Steps 1 and 2 need the user's own Chrome, so hand those to them. You can do steps 3 and 4.

## 1. Install the Chrome extension (user)

Install **Playwright MCP Bridge**: https://chromewebstore.google.com/detail/playwright-extension/mmlmfjhmonkocbjadbfplnigmagldckm

## 2. Copy the token (user)

Open the extension's status page. Click the extension icon in Chrome, or navigate to:

```
chrome-extension://mmlmfjhmonkocbjadbfplnigmagldckm/status.html
```

Copy the `PLAYWRIGHT_MCP_EXTENSION_TOKEN` value. It stops Chrome from asking to approve the connection each session.

## 3. Register the server

Writes to `~/.claude.json`:

```bash
claude mcp add playwright --scope user \
  --env PLAYWRIGHT_MCP_EXTENSION_TOKEN=<token-from-status-page> \
  -- npx @playwright/mcp@latest --extension
```

Resulting entry:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--extension"],
      "env": { "PLAYWRIGHT_MCP_EXTENSION_TOKEN": "<token-from-status-page>" }
    }
  }
}
```

## 4. Disable the plugin server (only if present)

If the official Playwright plugin is enabled, turn it off so two servers aren't running. In `~/.claude/settings.json`:

```json
{
  "enabledPlugins": { "playwright@claude-plugins-official": false },
  "permissions": { "allow": ["mcp__playwright__*"] }
}
```

Skip this if the plugin was never installed.

## 5. Restart Claude Code

After restarting, the browser tools are named `mcp__playwright__*`, driving the user's own Chrome.

## Notes

- The token is optional. Plain `--extension` works but prompts for approval each session. The matching token silences that.
- The token stays in the user's Chrome and their `~/.claude.json`. It is not a shared secret, so each person uses their own.
