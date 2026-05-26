# Modern Programming With AI

A long-lived, extensible knowledge base of lessons, frameworks, and practical systems
for working in the age of AI. New finds (videos, articles, techniques) are ingested as
new **lessons**; the repo's own folder structure makes it easy to reference back to
every system learned.

> Note: this repo currently lives under the GitHub name `Modern-Programming-Age-of-AI`.
> The conceptual project name is "Modern Programming With AI".

## Layout (three-layer folder system)

This repo dogfoods the three-layer routing system from Lesson 1:

1. **Layer 1 — global map:** root [`CLAUDE.md`](./CLAUDE.md). Points the AI at everything.
2. **Layer 2 — per-workspace context:** a `CLAUDE.md` inside each folder under
   [`workspaces/`](./workspaces/), giving task guidance for that environment.
3. **Layer 3 — skills / MCP on demand:** `.claude/skills/` and `.mcp.json` nested
   inside the workspace whose workflow needs them, so they're only invoked when work
   happens in that folder.

```
.
├── CLAUDE.md                       # Layer 1: global map
├── README.md
├── docs/
│   └── plan.md                     # the originating project plan
├── lessons/
│   ├── <category>/                 # optional category subfolder (e.g. ui-ux/)
│   │   └── YYYY-MM-DD-title/       # one folder per lesson, dated
│   │       ├── README.md           # the write-up (entry point)
│   │       └── *.md                # optional companion docs the lesson references
│   └── YYYY-MM-DD-title/           # uncategorized lessons live at the top level
└── workspaces/
    └── <workspace-name>/
        ├── CLAUDE.md               # Layer 2: task guidance for this workspace
        ├── .mcp.json               # Layer 3: MCP servers for this workflow
        └── .claude/skills/<skill>/ # Layer 3: skills for this workflow
```

## Lessons

| Date | Lesson | Source |
|------|--------|--------|
| 2026-05-13 | [Folder System Architecture](./lessons/2026-05-13-folder-system-architecture/README.md) | Jake Van Clief — "Stop Building AI Agents. Use This Folder System Instead" |
| 2026-05-26 | [Design-to-Code Pipelines (Figma MCP + Claude Design)](./lessons/ui-ux/2026-05-26-figma-mcp-design-pipeline/README.md) | Companion docs — *Design-to-Code Pipeline*, *Refactoring a "Slop" Codebase with the Figma MCP*, and *The Claude Design Pipeline* |

## Adding a new lesson

1. Create `lessons/YYYY-MM-DD-short-title/README.md` (or
   `lessons/<category>/YYYY-MM-DD-short-title/README.md` if the lesson fits an
   existing theme like `ui-ux/`) with the write-up. Companion `*.md` docs can sit
   alongside `README.md` in the same folder.
2. Add a row to the table above.
3. If the lesson implies a repeatable workflow, scaffold a folder under `workspaces/`
   with its own `CLAUDE.md` (Layer 2) and any skills / `.mcp.json` it needs (Layer 3).
4. Update root `CLAUDE.md` so the AI can route to it.
