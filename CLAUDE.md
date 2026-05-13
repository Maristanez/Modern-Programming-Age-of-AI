# CLAUDE.md — Global Map (Layer 1)

This is the routing map for the "Modern Programming With AI" knowledge base.
Read this first; it tells you where everything is and how the repo is organized.

## What this repo is

A long-lived, extensible collection of **lessons** (frameworks, systems, techniques
for working with AI) plus **workspaces** (folders containing repeatable workflows,
each with their own context and tools). It is meant to grow indefinitely.

## The three-layer folder system (which this repo dogfoods)

- **Layer 1 — Global map:** this file. Lists lessons and workspaces, points you onward.
- **Layer 2 — Per-workspace context:** `workspaces/<name>/CLAUDE.md`. Open the one for
  the workspace you're working in; it has the task guidance for that environment.
- **Layer 3 — Skills & MCP, on demand:** `workspaces/<name>/.claude/skills/` and
  `workspaces/<name>/.mcp.json`. These are scoped to the workspace so they only come
  into play when work is happening in that folder.

Naming convention: dated assets use `YYYY-MM-DD-title` so they sort chronologically
and can be moved/referenced without a backend.

## Lessons

- `lessons/2026-05-13-folder-system-architecture/` — the three-layer routing system
  from Jake Van Clief's video "Stop Building AI Agents. Use This Folder System Instead".
  Start here to understand why the repo is shaped the way it is.

## Workspaces

- `workspaces/ingest-new-lesson/` — workflow for turning a new find (video, article,
  technique) into a properly-structured lesson folder in this repo.

## Conventions for the AI

- When asked to "add a lesson", follow `workspaces/ingest-new-lesson/CLAUDE.md`.
- Keep this file in sync: any new lesson or workspace must be linked here.
- Prefer editing existing files; keep write-ups in each lesson's `README.md`.
- Don't introduce a backend or build system; the folder structure *is* the system.

## Originating plan

See `docs/plan.md` for the original plan that bootstrapped this repo.
