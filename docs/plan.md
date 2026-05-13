# Plan: "Modern Programming With AI" repo — Lesson 1: Folder System Architecture

## Context

The user wants a new GitHub repo, **Modern Programming With AI**, as a long-lived,
extensible knowledge base of lessons, frameworks, and practical systems for working in
the age of AI — not a one-off. New finds (videos, articles, techniques) should be easy to
ingest as new lessons, and the repo's own folder structure should make it easy to
reference back to every system learned.

The first lesson is the folder-based architecture from Jake Van Clief's video
**"Stop Building AI Agents. Use This Folder System Instead"** — a three-layer routing
system (global `CLAUDE.md` map → per-workspace context files → skills/tools invoked on
demand) with structured `YYYY-MM-DD-title` naming so the AI can navigate and move assets
without a backend. The user wants two things from this lesson: (1) a comprehensive
write-up of how the system works, and (2) the repo itself to adopt/integrate that
folder-organization approach as its own structure (dogfooding).

**Constraint discovered:** GitHub MCP access in the originating session was locked to
other repos, so the repo had to be created/pushed by the user (or a future session with
access). This plan scaffolds the repo so it can be pushed as-is.

## Deliverable

A repo that dogfoods the exact three-layer folder system from the video, built to scale
to many future lessons. The three layers map to:

1. Root `CLAUDE.md` = global map.
2. A `CLAUDE.md` inside each workspace folder = task guidance for that environment.
3. `.claude/skills/` + `.mcp.json` nested inside the workspace folder whose workflow
   needs them, so they're only invoked when work happens in that folder.

## Layout

```
.
├── CLAUDE.md                       # Layer 1: global map
├── README.md
├── docs/
│   └── plan.md                     # this file
├── lessons/
│   └── 2026-05-13-folder-system-architecture/
│       └── README.md               # the Lesson 1 write-up
└── workspaces/
    └── ingest-new-lesson/
        ├── CLAUDE.md               # Layer 2: how to ingest a new lesson
        ├── .mcp.json               # Layer 3: MCP servers for this workflow (placeholder)
        └── .claude/skills/lesson-intake/SKILL.md   # Layer 3: skill for this workflow
```

## Status / next steps for the at-home session

- [x] Scaffold the three-layer structure
- [x] Write Lesson 1 ("Folder System Architecture")
- [x] Scaffold the `ingest-new-lesson` workspace (Layer 2 + Layer 3 placeholders)
- [ ] Flesh out Layer 3: register real skills / MCP servers at the appropriate scope
      (the user wanted to decide whether some are also registered at the project root
      for auto-discovery)
- [ ] Add subsequent lessons as new dated folders under `lessons/`
