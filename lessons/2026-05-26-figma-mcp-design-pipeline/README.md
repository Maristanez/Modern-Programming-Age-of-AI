# Figma MCP Design Pipeline

**Date:** 2026-05-26
**Source:** Companion docs — *The Design-to-Code Pipeline* and *Refactoring a "Slop" Codebase with the Figma MCP Pipeline*
**Type:** Workflow / pipeline pattern

## TL;DR

A two-part workflow for translating visual design into production code without the
AI guessing. **Greenfield:** Dribbble → Figma (with Auto-Layout + tokens) → Figma MCP
→ Claude Code. **Refactor:** the same pipeline, but applied bottom-up to an existing
"slop" codebase in tiny, preservation-locked prompts so business logic survives the
redesign. Core idea: give the AI structural *facts* (tokens, Auto-Layout) instead of
visual *assumptions* (PNGs), and manage scope so the AI never holds more than one
component in its head at a time.

## Why it matters

The naive move — paste a Dribbble screenshot, ask Claude to "build this" — fails
in predictable ways: hallucinated hex codes, hardcoded spacing, non-responsive
`position: absolute` CSS, and silently dropped props during refactors. This pipeline
removes ambiguity at every step:

- **Dribbble** decides the vibe (humans only — never feed it to the AI).
- **Figma** turns the vibe into math (real tokens, real Auto-Layout).
- **The Figma MCP** exposes that math as a queryable data API.
- **Claude Code** consumes facts, not approximations.

On greenfield work this means consistent components on the first try. On refactors,
it's the only practical way to redesign an entire app without nuking auth checks,
data fetches, and prop chains.

## How it works

The two companion docs in this folder cover the two directions:

- **[`design-to-code-pipeline.md`](./design-to-code-pipeline.md)** — the greenfield
  flow. How to set up the Figma Remote MCP, why Auto-Layout is non-negotiable, the
  "ingest tokens globally before building any component" rule, and which other MCPs
  (shadcn, Chrome DevTools, Playwright) round out the stack.
- **[`refactoring-slop-codebase.md`](./refactoring-slop-codebase.md)** — the same
  pipeline applied to an existing codebase. Phase 0 audit, Phase 1 token sync,
  Phase 2 atomic component refactors with a strict preservation block, Phase 3 the
  analyze-then-write logic/visual split for messy files, Phase 4 page-level
  assembly. Plus the golden rules, failure modes, and the universal preservation
  block you paste into every refactor prompt.

### The high-level shape

```
Dribbble (vibe)  →  Figma (math)  →  Figma MCP (facts)  →  Claude Code (code)
                                                ↑
                                                │
                              shadcn MCP (primitives) ──┤
                              Chrome DevTools MCP (verify) ┘
```

### Key non-obvious rules

- **Never mix the Dribbble PNG with the Figma context during the build.** Two
  sources of truth = hallucination.
- **Image Node trap:** a Dribbble image dropped raw into Figma is opaque to the
  MCP. You must trace over it with native nodes (or reskin a UI kit) before the
  MCP can read anything useful.
- **Tokens first, components second.** Run a dedicated prompt that writes the full
  palette/typography/spacing into `tailwind.config.ts` *before* a single component
  is built. Otherwise components hardcode hex values and your design system drifts
  on day one.
- **For refactors, the AI is a contractor, not an architect.** One or two files per
  prompt, always with a preservation block, always with a commit after success.
  `git restore` is your safety net.
- **Analyze-then-write for tangled files.** Before letting Claude rewrite a
  `Dashboard.tsx` full of hooks and fetches, force it to first list every
  `useState`, `useEffect`, fetch call, and handler. Read the analysis. Correct
  omissions. *Then* let it write — and only the JSX block.

## How to apply it

### Greenfield (new project or new page)

1. Browse Dribbble; save references locally.
2. Open Figma; either trace the inspiration with Frames + Auto-Layout + named
   tokens, or paste in a free shadcn/Tailwind UI kit and reskin it.
3. Add the Figma Remote MCP once, at user scope:
   ```bash
   claude mcp add --scope user --transport http figma https://mcp.figma.com/mcp
   ```
   Run `/mcp` inside Claude Code and authenticate.
4. **Token prompt first** — extract the full token set into `tailwind.config.ts`
   and `globals.css`. Commit.
5. **Component prompts** — one frame at a time, into focused files. Reference
   only the tokens already in config.
6. Optionally bind Figma components to React files via Code Connect to prevent
   long-term drift.

### Refactor (existing "slop" codebase)

1. `git checkout -b refactor/figma-redesign`. Never refactor on `main`.
2. Phase 0: have Claude write `docs/refactor-audit.md` (pure vs mixed components,
   reuse counts) and `docs/figma-inventory.md` (frame → existing component map).
   Start with safe, single-use components.
3. Phase 1: sync tokens to `tailwind.config.ts`. Commit immediately.
4. Phase 2: refactor primitives bottom-up. Use the universal preservation block
   on every prompt. After each: read the diff, run `tsc --noEmit`, spot-check in
   the running app, commit.
5. Phase 3: for tangled files, run the **analyze step** first (read-only listing
   of state, effects, fetches, handlers). Confirm. Then the **write step** —
   JSX only, hooks/imports/data fetching untouched.
6. Phase 4: page-level assembly — by this point most children are already correct,
   so page prompts are mostly composition.

### The universal preservation block

Paste into every refactor prompt:

```
Strict preservation requirements:
- Do not change exported names or default exports.
- Do not change the TypeScript props interface.
- Do not remove, rename, or change the type of any prop.
- Preserve all event handlers and callbacks.
- Preserve all hook calls and their dependency arrays.
- Preserve all data fetching logic.
- Preserve all ref forwarding.
- Preserve all aria-* and accessibility attributes.
- Do not modify any file other than the one specified.
```

## Open questions / follow-ups

- **Code Connect maturity.** The greenfield doc mentions it as the long-term
  drift defense, but doesn't cover the rough edges (per-component binding,
  CI checks for unbound files). Worth a follow-up lesson once it's been used
  on a real project.
- **Where does shadcn MCP fit in a refactor?** The docs cover it for greenfield
  scaffolding but don't explicitly address whether it helps when retro-fitting
  primitives in Phase 2 of a refactor.
- **Context exhaustion thresholds.** The refactor doc names "stop, commit, fresh
  session" as the recovery — but is there a heuristic (file count? minutes?
  prompt count?) that predicts when to bail proactively rather than reactively?
- **Visual verification loop automation.** Chrome DevTools MCP is mentioned as
  the polish loop. Could it be wired into a per-commit screenshot diff against
  the Figma frame, or is that overkill?
- **Worth a dedicated workspace?** This is a repeatable workflow, but the README
  here doubles as the runbook. If the same person ends up running it multiple
  times, promote it to `workspaces/figma-redesign/` with skills for each phase.
