# context-telemetry

Auto-track your Claude Code usage patterns with skills, agents, and MCP tools.

## What is this?

A Claude Code plugin that automatically captures every skill invocation (`/commit`, `/simplify`, etc.), agent spawn, and MCP tool call you make. View your usage patterns in a sleek web dashboard.

**No configuration required** - just install and it works.

## Features

- 📊 **Automatic tracking** - PostToolUse hooks capture events transparently
- 🎯 **Zero overhead** - Fire-and-forget design never blocks Claude
- 📈 **Usage analytics** - See your most-used skills, agents, and MCP tools
- 📅 **Timeline view** - Daily usage trends over time
- 🌐 **Web dashboard** - Clean, space-themed UI with charts
- 💾 **Local storage** - All data stays on your machine in SQLite

## Installation

```bash
# Clone or download this repository
cd context-telemetry

# Install as a Claude Code plugin
# (Copy to your plugins directory or install via Claude Code plugin manager)
```

The plugin will auto-install dependencies and start the telemetry server on first use.

## Usage

Once installed, the plugin works automatically. Every time you:
- Use a skill like `/commit` or `/review-pr`
- Spawn an agent with the Agent tool
- Call any MCP server tool

...the event is logged to `~/.context-telemetry/events.db`.

### View Dashboard

Open the web dashboard to see your usage patterns:

```bash
# Dashboard runs at http://localhost:3001
open http://localhost:3001
```

Or ask Claude Code to open it for you:
```
Show me my usage stats
```

### Query Stats Programmatically

Use the MCP tools:

```javascript
// Get stats for the last 7 days
get_stats({ period: "7d" })

// Available periods: "24h", "7d", "30d", "all"
```

## How It Works

### Architecture

```
┌─────────────────┐
│  Claude Code    │
│    Session      │
└────────┬────────┘
         │
         │ PostToolUse hook fires
         ↓
┌─────────────────┐
│   src/hook.js   │ ← Extracts skill/agent/MCP events
└────────┬────────┘
         │
         │ HTTP POST (fire-and-forget)
         ↓
┌─────────────────┐
│  Daemon Server  │ ← Runs on localhost:3001
│   (src/server)  │    Stores events in SQLite
└────────┬────────┘
         │
         │ Provides data to
         ↓
┌─────────────────┐
│  MCP Server +   │ ← get_stats tool
│   Dashboard     │    Web UI at /
└─────────────────┘
```

### Session Management

- Each Claude Code session registers a heartbeat file
- A single background daemon serves all sessions
- Daemon auto-terminates when no sessions have been active for 90s
- Next session automatically restarts the daemon

### Data Storage

All data is stored locally at:
- **Database**: `~/.context-telemetry/events.db`
- **PID file**: `~/.context-telemetry/server.pid`
- **Session heartbeats**: `~/.context-telemetry/sessions/<pid>.json`

## Configuration

### Change Port

Set environment variable in `~/.claude/settings.json`:

```json
{
  "env": {
    "CONTEXT_TELEMETRY_PORT": "8080"
  }
}
```

### Custom Hook Endpoint

If you want hooks to POST to a different server:

```json
{
  "env": {
    "AI_TELEMETRY_URL": "http://localhost:8080/api/events"
  }
}
```

## Development

### Install dependencies

```bash
npm install
```

### Run daemon manually

```bash
node src/server.js --daemon
```

### Test MCP server

```bash
node src/server.js
# Communicate via stdio (MCP protocol)
```

### Test hook

```bash
echo '{"tool_name":"Skill","tool_input":{"skill":"test"},"session_id":"test","cwd":"/tmp"}' | node src/hook.js
```

## API Reference

### HTTP Endpoints

**POST /api/events**
```json
{
  "event_type": "skill",
  "name": "commit",
  "detail": "-m 'fix bug'",
  "session_id": "abc123",
  "cwd": "/path/to/repo",
  "timestamp": "2026-04-01T10:30:00Z"
}
```

**GET /api/stats?period={24h|7d|30d|all}**

Returns aggregated statistics with top skills, agents, MCP tools, timeline, and recent events.

**GET /**

Serves the web dashboard.

### MCP Tools

**get_stats**
- Input: `{ period: "7d" }` (optional, default: "7d")
- Returns: Usage statistics as JSON

**get_dashboard_url**
- Input: `{}`
- Returns: Dashboard URL

## Privacy

All telemetry data stays on your local machine. No data is sent to external servers.

The SQLite database contains:
- Skill names and arguments
- Agent types and descriptions
- MCP server and tool names
- Working directory paths
- Timestamps

## License

MIT

## Requirements

- Node.js >= 22.5
- Claude Code (for plugin functionality)
