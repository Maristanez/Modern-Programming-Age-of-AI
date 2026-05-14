# Interpretable Context Methodology (ICM)
### Folder Structure as Agentic Architecture
*By Jake Van Clief — also called the Model Workspace Protocol (MWP)*

---

## TL;DR

ICM replaces multi-agent frameworks and orchestration code with **filesystem structure**. Numbered folders encode the pipeline; plain markdown files carry the prompts, rules, and artifacts. **One agent** reads the right files at the right moment instead of many agents coordinating through a framework.

> If the prompts and context for each stage already exist as files in a well-organized folder hierarchy, you don't need multiple agents or a coordination framework — you need one agent that reads the right files at the right moment.

---

## Core Principles

1. **One stage, one job** — each folder handles a single workflow step (Unix philosophy).
2. **Plain-text interface** — markdown and JSON only; no binary state, no hidden memory.
3. **Layered context loading** — each stage loads only what it needs.
4. **Every output is editable** — humans can inspect and modify artifacts between stages.
5. **Configure the factory, not the product** — set rules once in `_config/`, reuse across runs.

---

## The Five-Layer Context Model

Agents read context top-down through five layers. Each layer answers a different question.

| Layer | File / Location | Question Answered | Typical Tokens |
|---|---|---|---|
| **0 — Identity** | `CLAUDE.md` (root) | *Where am I?* | ~800 |
| **1 — Routing** | `CONTEXT.md` (root) | *Where do I go?* | ~300 |
| **2 — Stage Contract** | `NN_stage/CONTEXT.md` | *What do I do?* | 200–500 |
| **3 — Reference** | `_config/`, `stage/references/` | *What rules apply?* | 500–2k |
| **4 — Artifacts** | `stage/output/`, prior-stage outputs | *What am I working with?* | varies |

**Total per stage:** ~2,000–8,000 tokens. Layer 3 is *internalized as constraints*; Layer 4 is *processed as input*.

---

## Canonical Folder Structure

```
workspace/
├── CLAUDE.md                    # Layer 0 — identity & workspace map
├── CONTEXT.md                   # Layer 1 — stage routing, shared resources
│
├── _config/                     # Layer 3 — stable, workspace-wide rules
│   ├── voice.md
│   ├── design-system.md
│   └── conventions.md
│
├── shared/                      # Layer 3 — cross-stage shared resources
│   └── setup/
│       └── questionnaire.md
│
├── 01_research/                 # Stage 1
│   ├── CONTEXT.md               # Layer 2 — this stage's contract
│   ├── references/              # Layer 3 — stage-scoped references
│   └── output/                  # Layer 4 — artifacts produced here
│
├── 02_script/                   # Stage 2
│   ├── CONTEXT.md
│   ├── references/
│   └── output/
│
└── 03_production/               # Stage 3
    ├── CONTEXT.md
    ├── references/
    └── output/
```

### Naming Conventions

- **Stage folders** are numbered (`01_`, `02_`, `03_`) — the digits encode execution order so no orchestration code is needed.
- **`references/`** inside a stage holds Layer 3 material scoped to that stage.
- **`output/`** inside a stage is the handoff point — its files become the next stage's Layer 4 inputs.
- **`_config/`** (leading underscore) holds workspace-wide Layer 3 reference material.
- **`CONTEXT.md`** is the universal name for routing/contract files at each level.

---

## Stage Contract (Layer 2) Template

Every `NN_stage/CONTEXT.md` has three sections:

```markdown
## Inputs
- Layer 4 (working):   ../01_research/output/findings.md
- Layer 3 (reference): ../_config/voice.md
- Layer 3 (reference): references/structure.md

## Process
Transform the research findings into a script that matches the tone
defined in voice.md and the structural pattern in structure.md.

## Outputs
- script_draft.md -> output/
```

The **Inputs** list makes context selection *explicit, editable, and auditable* — you can see exactly what an agent will load before it runs.

---

## How Agents Run It

- **Single Claude Code session** orchestrates the whole pipeline.
- **Orchestrator** (e.g. Opus 4.6) walks the numbered stages and reads each `CONTEXT.md`.
- **Sub-tasks within a stage** are delegated to faster models (e.g. Sonnet 4.6); the stage's `CONTEXT.md` *is* the sub-agent spec — no separate orchestration code.
- **Human review gates** exist naturally at stage boundaries: inspect or edit `output/` before the next stage runs.

---

## Why This Works

- **Observable by default** — every intermediate state is a plain file. No logging layer, no dashboard, no special tooling.
- **Portable** — copy the folder to any machine; it works immediately.
- **Version-controllable** — Git diffs every prompt, rule, and artifact.
- **Non-technical-editable** — users have modified stage behavior (tone, constraints, ordering) by editing markdown alone, no code required.

---

## Bootstrapping a New Workspace

1. Clone the ICM repo.
2. Navigate to a workspace (e.g. `script-to-animation`, `course-deck-production`, or `workspace-builder` to scaffold a new one).
3. Open Claude Code in that directory.
4. Type `setup` and answer the `shared/setup/questionnaire.md` prompts.
5. Walk through the numbered stages sequentially.

---

## When *Not* to Use ICM

- Real-time multi-agent collaboration.
- High-concurrency systems.
- Complex automated branching logic (ICM is linear-by-design).

---

## References

- Paper: [arxiv.org/abs/2603.16021](https://arxiv.org/abs/2603.16021) — *Interpretable Context Methodology: Folder Structure as Agentic Architecture*
- Repo: [github.com/RinDig/Interpreted-Context-Methdology](https://github.com/RinDig/Interpreted-Context-Methdology)
- Video: [Stop Building AI Agents. Use This Folder System Instead.](https://www.youtube.com/watch?v=MkN-ss2Nl10)
- Community: [Clief Notes — Skool](https://www.skool.com/quantum-quill-lyceum-1116)
