# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Run is a full-stack CLI + web UI for browsing Claude Code conversation history. It reads from `~/.claude/` directory and presents conversations in a real-time web interface.

## Commands

```bash
# Development (runs both web and API servers concurrently)
pnpm dev

# Production build
pnpm build

# Run production build
pnpm start
```

Development servers:
- Web: http://localhost:12000 (Vite dev server)
- API: http://localhost:12001 (Hono server)

## Architecture

**Backend (`/api`):**
- `index.ts` - CLI entry point using Commander for argument parsing
- `server.ts` - Hono HTTP server with API routes and static file serving
- `storage.ts` - Data access layer reading Claude's JSONL history files
- `watcher.ts` - Chokidar file watcher for real-time updates via SSE

**Frontend (`/web`):**
- React 19 + Vite + Tailwind CSS v4
- `app.tsx` - Main layout with sidebar and session view
- `components/session-list.tsx` - Virtualized session list using TanStack Virtual
- `components/session-view.tsx` - Conversation viewer with SSE streaming
- `components/message-block.tsx` - Message renderer with tool call handling
- `components/tool-renderers/` - Specialized renderers for each Claude tool type
- `hooks/use-event-source.ts` - Custom hook for SSE with reconnection logic

**Data Flow:**
1. File changes in `~/.claude/` detected by Chokidar watcher
2. Server emits SSE events to connected clients
3. Frontend updates UI in real-time via EventSource connections

**API Endpoints:**
- `GET /api/sessions` - List all sessions
- `GET /api/projects` - List all projects
- `GET /api/sessions/stream` - SSE stream for session list updates
- `GET /api/conversation/:id` - Get conversation messages
- `GET /api/conversation/:id/stream` - SSE stream for message updates

## Storage Format

History file (`~/.claude/history.jsonl`):
```jsonl
{"display":"title","timestamp":1234567890,"project":"path/to/project","sessionId":"abc123"}
```

Conversation files (`~/.claude/projects/*/sessionId.jsonl`):
```jsonl
{"type":"user","uuid":"...","message":{"role":"user","content":"..."}}
{"type":"assistant","uuid":"...","message":{"role":"assistant","content":[...]}}
{"type":"summary","summary":"..."}
```

## Key Conventions

- TypeScript strict mode throughout
- Path alias `@claude-run/api` maps to `../api/storage.ts` for shared types
- Vite proxies `/api` requests to port 12001 in development
- SSE streams use offset-based pagination for incremental loading
- File watcher debounces changes at 20ms
