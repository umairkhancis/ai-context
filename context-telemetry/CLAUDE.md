# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

context-telemetry is a Claude Code plugin that automatically tracks skill, agent, and MCP tool usage through a dual-mode architecture:
- **MCP server** that exposes telemetry data to Claude Code sessions
- **Background daemon** that persists events to SQLite and serves a web dashboard
- **PostToolUse hooks** that capture events without blocking Claude

Data is stored at `~/.context-telemetry/events.db`

## Architecture

### Dual-Mode Server (`src/server.js`)

The server runs in two distinct modes:

**MCP Mode** (default):
- Ensures background daemon is running
- Registers session heartbeat at `~/.context-telemetry/sessions/<pid>.json`
- Runs MCP stdio server that proxies requests to daemon via HTTP
- Refreshes heartbeat every 30s

**Daemon Mode** (`--daemon` flag):
- Binds HTTP server to `localhost:3001` (or `CONTEXT_TELEMETRY_PORT`)
- Stores events in SQLite
- Monitors session heartbeats every 30s
- Auto-terminates when no sessions have updated heartbeats in 90s
- Port binding acts as the lock - only one daemon runs at a time

### Session Management Pattern

Critical race condition handling:
1. Daemon binds port FIRST, then writes PID file - prevents race where multiple spawned daemons fight over the PID file
2. Each MCP session writes heartbeat file on startup, updates every 30s
3. Daemon has 60s grace period before first shutdown check (allows first session to write heartbeat)
4. Sessions cleanup their heartbeat files on exit

### Event Flow

```
PostToolUse hook triggers
    ↓
src/hook.js extracts event (Skill/Agent/mcp__)
    ↓
HTTP POST to daemon (3s timeout, fire-and-forget)
    ↓
Daemon stores in SQLite events table
    ↓
MCP tools/dashboard query via HTTP
```

## Development

### Install dependencies
```bash
npm install
```

### Test MCP server locally
```bash
node src/server.js
# In another terminal, send MCP requests via stdio
```

### Run daemon manually
```bash
node src/server.js --daemon
# Access dashboard at http://localhost:3001
```

### Test hook processing
```bash
echo '{"tool_name":"Skill","tool_input":{"skill":"test","args":"foo"},"session_id":"test","cwd":"/tmp"}' | node src/hook.js
```

## Plugin Integration Files

- `.claude-plugin/plugin.json`: MCP server registration, runs `src/init.sh`
- `hooks/hooks.json`: PostToolUse hook config with matcher `Skill|Agent|mcp__.*`
- `src/init.sh`: Entry point that runs `npm install` if needed, then starts server

## Event Types & Schema

Events table:
```sql
CREATE TABLE events (
  id         INTEGER PRIMARY KEY AUTOINCREMENT,
  event_type TEXT NOT NULL,  -- 'skill', 'agent', or 'mcp'
  name       TEXT NOT NULL,  -- skill name / agent type / 'server__tool'
  detail     TEXT,           -- args / description / server name
  session_id TEXT,
  cwd        TEXT,
  timestamp  TEXT
)
```

Event extraction logic in `src/hook.js`:
- **Skill**: `tool_name === "Skill"` → name from `input.skill`, detail from `input.args`
- **Agent**: `tool_name === "Agent"` → name from `input.subagent_type`, detail from `input.description` (truncated to 120 chars)
- **MCP**: `tool_name.startsWith("mcp__")` → name is `server__tool`, detail is server name

Legacy migration: Auto-migrates old `skill_events` table to unified `events` table on daemon startup.

## API Endpoints

**POST /api/events** - Accept telemetry event (hook target)

**GET /api/stats?period={24h|7d|30d|all}** - Returns:
- Top 15 skills/agents/MCP tools with counts and percentages
- Daily timeline of events by type
- 30 most recent events

**GET /** - Serves dashboard HTML

## Key Design Patterns

### Fire-and-forget telemetry
`src/hook.js` never blocks Claude Code:
- 3s timeout on HTTP requests
- All errors swallowed silently  
- Exits immediately whether request succeeds or fails

### Detached daemon spawning
MCP mode spawns daemon as detached process:
```javascript
spawn(process.execPath, [__filename, '--daemon'], {
  detached: true,
  stdio: 'ignore',
  env: process.env
});
child.unref();
```

Polls for PID file up to 3s to confirm daemon started.

### Shared daemon, isolated MCP servers
Every Claude session runs its own MCP server instance, but they all communicate with a single shared daemon via HTTP. This allows centralized data storage while maintaining per-session isolation.

## Configuration

Environment variables:
- `CONTEXT_TELEMETRY_PORT`: Daemon HTTP port (default: 3001)
- `AI_TELEMETRY_URL`: Hook POST target (default: `http://localhost:8765/api/events`)

Note: Default `AI_TELEMETRY_URL` port 8765 is legacy - actual daemon runs on 3001. Update if hook target needs to change.
