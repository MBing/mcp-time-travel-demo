# mcp-time-travel demo: Playwright MCP

End-to-end walkthrough of [mcp-time-travel](https://github.com/MBing/mcp-time-travel) using [Playwright MCP](https://github.com/microsoft/playwright-mcp) as the target server.

## Prerequisites

- Node.js 18+
- Claude Code installed

## Step 1 -- Open this repo in Claude Code

```bash
cd mcp-time-travel-demo
claude
```

The project-scoped `.mcp.json` is picked up automatically, which defines:

- `playwright` -- the real Playwright MCP server
- `playwright-recorded` -- mcp-time-travel proxies to `playwright` while recording all traffic

Run `/mcp` to verify both servers appear.

## Step 2 -- Record a session

Make sure the `playwright-recorded` server is active. Ask something like:

> Navigate to https://example.com, take a screenshot, then click the "More information" link.

This generates tool calls (`browser_navigate`, `browser_screenshot`, `browser_click`, etc.) that get recorded transparently.

When the conversation ends, the session is saved automatically. You should see stderr output like:

```
[mcp-time-travel] Recording session: 20260316-...
[mcp-time-travel] Session saved: 20260316-...
```

## Step 3 -- List sessions

```bash
npx mcp-time-travel list
```

You should see your session in the table with the server name `playwright` and the number of tool calls. Copy the session ID for the next steps.

## Step 4 -- Inspect the session

```bash
npx mcp-time-travel inspect <session-id>
```

Expected output:

- **Session Summary** -- ID, server name, transport, duration, tool call count, error count
- **Top Tools** -- bar chart showing tool frequency (e.g. `browser_navigate`, `browser_screenshot`)
- **Timeline** -- each call with seq number, tool name, and latency in ms

## Step 5 -- Debug the session

```bash
npx mcp-time-travel debug <session-id>
```

Interactive step-through debugger:

| Key | Action |
|-----|--------|
| `n` | Next tool call |
| `p` | Previous tool call |
| `l` | List all tool calls |
| `g <n>` | Go to step N |
| `q` | Quit |

You can inspect the full input/output JSON for each Playwright tool call.

## Step 6 -- Replay the session

Add a replay entry to `.mcp.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp"]
    },
    "playwright-recorded": {
      "command": "npx",
      "args": ["mcp-time-travel", "record", "--server", "playwright"]
    },
    "playwright-replay": {
      "command": "npx",
      "args": ["mcp-time-travel", "replay", "<session-id>"]
    }
  }
}
```

Replace `<session-id>` with the ID from Step 3. Restart Claude Code and use the `playwright-replay` server. Ask Claude to repeat the same task -- it receives the recorded responses without launching a real browser.

If the agent sends different inputs than what was recorded, you will see warnings on stderr:

```
[mcp-time-travel] Warning: input mismatch for "browser_navigate" at seq 1
```
