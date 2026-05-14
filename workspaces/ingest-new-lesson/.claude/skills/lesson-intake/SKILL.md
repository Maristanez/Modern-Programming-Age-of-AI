---
name: lesson-intake
description: >-
  Turn a new find (video, article, technique, talk, repo) into a properly-structured
  lesson folder in the "Modern Programming With AI" knowledge base. Use when the user
  says something like "add a lesson", "ingest this video/article", or "write this up as a
  lesson".
---

# Skill: lesson-intake

## When to use

The user has a source they want captured as a durable lesson in this repo.

## Procedure

1. **Gather inputs:** the source (URL/title/notes), today's date, and a short
   kebab-case title.
2. **Create the lesson folder:** `lessons/<YYYY-MM-DD>-<short-title>/`.
3. **Write `README.md`** in that folder with these sections:
   - `# <Lesson title>` plus a metadata block: **Date**, **Source**, **Type**.
   - `## TL;DR` — 3–6 lines.
   - `## Why it matters` — the motivation / problem it solves.
   - `## How it works` — the substance, with subsections as needed.
   - `## How to apply it` — concrete steps the reader can follow.
   - `## Open questions / follow-ups` — anything unresolved.
4. **If the lesson implies a repeatable workflow**, scaffold a new workspace under
   `workspaces/<name>/` using `workspaces/ingest-new-lesson/` as the template:
   a `CLAUDE.md` (Layer 2) and, only if needed, `.claude/skills/` and/or `.mcp.json`
   (Layer 3).
5. **Update the global map:** add the lesson to the "Lessons" section of the root
   `CLAUDE.md` and the "Lessons" table in the root `README.md`.
6. **Commit:** `Add lesson: <title>`.

## Notes

- Keep write-ups in `README.md` so they render on GitHub.
- One lesson per dated folder; never combine lessons.
- Don't add tooling a lesson doesn't actually require.
- This skill is workspace-scoped (Layer 3). If the user wants it auto-discovered from
  anywhere in the repo, it can also be registered at the project root — that's a
  deliberate convenience-vs-scoped-context trade-off for the user to decide.
