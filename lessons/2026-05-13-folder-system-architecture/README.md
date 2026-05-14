# Lesson 1 — Folder System Architecture (the three-layer routing system)

**Date:** 2026-05-13
**Source:** Jake Van Clief — *"Stop Building AI Agents. Use This Folder System Instead"*
**Type:** Organizational system / framework

---

## TL;DR

Instead of building bespoke "agents" with custom orchestration code, you organize your
workspace into a **three-layer folder system** and let the AI navigate it:

1. **Global map** — a single root `CLAUDE.md` that describes the whole system and points
   the AI to everything else.
2. **Per-workspace context** — each working folder has its own `CLAUDE.md` with the task
   guidance specific to that environment.
3. **Skills & tools on demand** — skills (`.claude/skills/`) and MCP servers (`.mcp.json`)
   live *inside* the workspace folder whose workflow needs them, so they're only loaded
   when work is actually happening there.

Plus a naming convention: dated assets use `YYYY-MM-DD-title`, so things sort
chronologically and can be moved or referenced without any backend or database. The
folder structure *is* the routing logic.

## Why this instead of "agents"

- **No orchestration code to maintain.** The "routing" is just the AI reading `CLAUDE.md`
  files and walking the directory tree.
- **Composable & portable.** Folders can be copied, moved, archived, or shared. A lesson
  or workspace is self-contained.
- **Context stays scoped.** Layer 3 (skills/MCP) only enters the picture when you're in
  the relevant folder, so you don't pay context cost for tools you aren't using.
- **Scales by accretion.** Adding capability = adding a folder + linking it from the map.
  Nothing else has to change.

## The three layers in detail

### Layer 1 — Global map (`/CLAUDE.md`)

The entry point. It should answer, for a fresh AI session:

- What is this repo / workspace?
- What are the major areas (lessons, workspaces, projects)?
- Where do I go for task X?
- What conventions must I follow?

Keep it a *map*, not a manual. It links outward; details live in Layer 2.

### Layer 2 — Per-workspace context (`workspaces/<name>/CLAUDE.md`)

Each workspace folder is an "environment" for a particular kind of work (e.g. ingesting a
new lesson, drafting a blog post, doing research). Its `CLAUDE.md` holds:

- The goal of work done in this folder.
- Step-by-step guidance / checklists for the recurring task.
- Pointers to the Layer 3 skills/MCP in this folder and when to use them.
- Output conventions (where results go, naming, etc.).

### Layer 3 — Skills & MCP, invoked on demand (`workspaces/<name>/.claude/skills/`, `workspaces/<name>/.mcp.json`)

The actual capabilities — reusable skills and external tool servers — nested inside the
workspace that needs them. Because they're scoped to the folder:

- They're discovered/loaded when the AI is operating in that folder.
- Different workspaces can have different (even conflicting) tool setups.
- You can optionally *also* register some at the project root if you want them
  auto-discovered everywhere — a deliberate trade-off (convenience vs. scoped context).

## The naming convention

`YYYY-MM-DD-title` (e.g. `2026-05-13-folder-system-architecture`):

- Sorts chronologically in any file browser.
- Makes "what did I learn and when" trivially answerable.
- Lets the AI move/rename/reference assets purely by path — no index, no DB, no backend.

## How this repo dogfoods it

| Layer | In this repo |
|-------|--------------|
| 1 — global map | [`/CLAUDE.md`](../../CLAUDE.md) |
| 2 — per-workspace context | [`workspaces/ingest-new-lesson/CLAUDE.md`](../../workspaces/ingest-new-lesson/CLAUDE.md) |
| 3 — skills / MCP on demand | [`workspaces/ingest-new-lesson/.claude/skills/lesson-intake/SKILL.md`](../../workspaces/ingest-new-lesson/.claude/skills/lesson-intake/SKILL.md), [`workspaces/ingest-new-lesson/.mcp.json`](../../workspaces/ingest-new-lesson/.mcp.json) |
| naming | `lessons/YYYY-MM-DD-title/`, this folder being the first |

## Applying it to your own work

1. Put a `CLAUDE.md` at the root that maps the territory.
2. Make a folder per recurring kind of work; give each its own `CLAUDE.md`.
3. Drop skills/MCP into the folder whose workflow uses them; promote to root only if you
   genuinely want them everywhere.
4. Date anything that's a point-in-time artifact with `YYYY-MM-DD-`.
5. When you learn something new, it's a new dated folder + a link from the map — done.

## Open questions captured from the originating session

- Should some Layer-3 skills/MCP also be registered at the **project root** for
  auto-discovery, or kept strictly workspace-scoped? (Pending the user's decision.)
