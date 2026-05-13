# CLAUDE.md — Workspace: Ingest a New Lesson (Layer 2)

This folder is the workspace for turning a new find (a video, article, technique, repo,
talk, etc.) into a properly-structured **lesson** in this knowledge base.

## Goal

Given some source material, produce:
1. A new folder `lessons/YYYY-MM-DD-short-title/` at the repo root.
2. A `README.md` inside it: a clear, comprehensive write-up (what the system/idea is, why
   it matters, how it works, how to apply it).
3. If the lesson implies a repeatable workflow, a new folder under `workspaces/` with its
   own `CLAUDE.md` (Layer 2) and any skills / `.mcp.json` it needs (Layer 3).
4. Updates to the root `CLAUDE.md` and `README.md` so the new lesson is linked from the
   global map.

## Steps

1. Identify the date (`YYYY-MM-DD`) and a short kebab-case title.
2. `mkdir lessons/<date>-<title>/`.
3. Write `lessons/<date>-<title>/README.md`. Suggested sections:
   - Title line with **Date** and **Source**
   - **TL;DR**
   - **Why it matters**
   - **How it works** (the substance)
   - **How to apply it**
   - **Open questions / follow-ups**
4. If applicable, scaffold a workspace (see this folder as the template).
5. Update root `CLAUDE.md` ("Lessons" section) and `README.md` ("Lessons" table).
6. Commit with a message like `Add lesson: <title>`.

## Layer 3 in this workspace

- `.claude/skills/lesson-intake/SKILL.md` — the skill describing the intake procedure in
  a form the AI can invoke directly.
- `.mcp.json` — placeholder for MCP servers useful when ingesting lessons (e.g. a web
  fetch / transcript tool). Fill in real servers as needed.

## Conventions

- One lesson = one dated folder. Don't mix lessons.
- Keep write-ups in `README.md` so they render on GitHub.
- Don't build tooling beyond what a lesson actually needs.
