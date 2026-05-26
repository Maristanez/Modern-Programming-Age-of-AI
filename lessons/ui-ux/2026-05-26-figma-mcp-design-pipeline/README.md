# Design-to-Code Pipelines (Figma MCP + Claude Design)

**Date:** 2026-05-26
**Source:** Companion docs — *The Design-to-Code Pipeline*, *Refactoring a "Slop" Codebase with the Figma MCP Pipeline*, and *The Claude Design Pipeline*
**Type:** Workflow / pipeline pattern

## TL;DR

Two routes from visual design to production code, plus the refactor discipline that
applies to both. **Figma MCP route:** Dribbble → Figma (Auto-Layout + tokens) → Figma
MCP → Claude Code. **Claude Design route:** claude.ai/design canvas → Handoff to
Claude Code, collapsing the trace-in-Figma and token-extraction steps into a single
conversation. **Brownfield (slop codebase):** regardless of which route you use to
*design*, refactor existing code bottom-up in tiny preservation-locked prompts so
business logic survives the redesign. Core idea: give the AI structural *facts*
(tokens, Auto-Layout, handoff bundle) instead of visual *assumptions* (PNGs), and
manage scope so the AI never holds more than one component in its head at a time.

## Why it matters

The naive move — paste a Dribbble screenshot, ask Claude to "build this" — fails
in predictable ways: hallucinated hex codes, hardcoded spacing, non-responsive
`position: absolute` CSS, and silently dropped props during refactors. Both
pipelines remove ambiguity by inserting structured translation between the vibe
and the code:

- **Figma MCP route** — Dribbble decides the vibe; Figma turns it into math; the
  MCP exposes that math as a queryable API; Claude Code consumes facts, not
  approximations.
- **Claude Design route** — you describe what you want on a canvas backed by a
  design system Claude has already extracted from your repo. The handoff bundle
  carries tokens, component structure, and page intent across to Claude Code in
  one step.

On greenfield work either route gives consistent components on the first try. On
refactors, the refactoring discipline (the third companion doc) is the only
practical way to redesign an entire app without nuking auth checks, data fetches,
and prop chains.

## How it works

The three companion docs in this folder cover the three angles:

- **[`design-to-code-pipeline.md`](./design-to-code-pipeline.md)** — the
  greenfield Figma MCP flow. How to set up the Figma Remote MCP, why Auto-Layout
  is non-negotiable, the "ingest tokens globally before building any component"
  rule, and which other MCPs (shadcn, Chrome DevTools, Playwright) round out the
  stack.
- **[`refactoring-slop-codebase.md`](./refactoring-slop-codebase.md)** — the
  pipeline applied to an existing codebase. Phase 0 audit, Phase 1 token sync,
  Phase 2 atomic component refactors with a strict preservation block, Phase 3 the
  analyze-then-write logic/visual split for messy files, Phase 4 page-level
  assembly. Plus the golden rules, failure modes, and the universal preservation
  block you paste into every refactor prompt.
- **[`claude-design-pipeline.md`](./claude-design-pipeline.md)** — the native
  Anthropic alternative to the Figma MCP path. How claude.ai/design ingests your
  repo to build a design system, the structured-brief prompt template, refining on
  the canvas via chat + inline comments + sliders, the Handoff-to-Claude-Code
  bundle, and (critically) how to combine Claude Design with a slop codebase
  *without* triggering a destructive one-shot redesign.

### Two routes, one discipline

```
Route A (Figma MCP):
  Dribbble (vibe)  →  Figma (math)  →  Figma MCP (facts)  →  Claude Code (code)

Route B (Claude Design):
  Reference + repo  →  claude.ai/design (canvas + design system)  →  Handoff bundle  →  Claude Code (code)

Either route + existing slop codebase:
  → switch to refactoring-slop-codebase.md discipline before touching code
```

### Key non-obvious rules

- **Pick a route based on the team, not the trend.** Figma MCP wins where a
  mature Figma file or traditional-designer workflow already exists. Claude Design
  wins for solo founders / PMs / engineers prototyping from scratch. Many teams
  end up using Claude Design for exploration and Figma for production polish.
- **Never mix raw PNGs with structured design context during the build.** Two
  sources of truth = hallucination. Keep Dribbble in the browser; only feed Figma
  data or the Claude Design handoff to the agent.
- **Image Node trap (Figma route).** A Dribbble image dropped raw into Figma is
  opaque to the MCP. Trace over it with native nodes (or reskin a UI kit) before
  the MCP can read anything useful.
- **Design system first. Always.** Whichever route, lock the tokens before
  generating a single screen — Phase 4 Step 1 in the Figma flow, Phase 1 in the
  Claude Design flow. Otherwise components hardcode hex values and your design
  system drifts on day one.
- **The handoff is for new code. The refactor discipline is for existing code.**
  Claude Design's one-click handoff assumes Claude Code is *generating* fresh
  components, not *replacing* tangled ones. Use the handoff for new screens; use
  the bottom-up preservation-locked discipline for existing screens.
- **For refactors, the AI is a contractor, not an architect.** One or two files
  per prompt, always with a preservation block, always with a commit after
  success. `git restore` is your safety net.
- **Analyze-then-write for tangled files.** Before letting Claude rewrite a
  `Dashboard.tsx` full of hooks and fetches, force it to first list every
  `useState`, `useEffect`, fetch call, and handler. Read the analysis. Correct
  omissions. *Then* let it write — and only the JSX block.

## How to apply it

### Greenfield via Figma MCP

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

### Greenfield via Claude Design

1. Open claude.ai/design; new project.
2. Set up the design system (ingest repo + 2–3 reference URLs via web capture).
   Verify the extracted tokens before generating anything.
3. Write a **structured brief** (artifact, audience, sections, key actions,
   constraints, tone) — dense prompts outperform vague ones.
4. Generate → refine on canvas with chat (structural changes), inline comments
   (local tweaks), and sliders (spacing/color).
5. Add motion before handoff so animation intent travels with the bundle.
6. Export → Handoff to Claude Code. Paste the handoff command into Claude Code in
   your terminal.
7. Treat the first generated pass as a scaffold, not a finished product. Run the
   dev server, compare against the canvas, refine incrementally.

### Brownfield (existing "slop" codebase)

1. `git checkout -b refactor/<route>-redesign`. Never refactor on `main`.
2. Use **either** route to produce the *visual target* — a Figma file with
   Auto-Layout + tokens, or a Claude Design canvas you've refined to match your
   real goal. Do NOT one-shot a handoff against the slop repo.
3. Phase 0: have Claude write `docs/refactor-audit.md` (pure vs mixed components,
   reuse counts) and a frame/canvas → existing-component map. Start with safe,
   single-use components.
4. Phase 1: sync tokens to `tailwind.config.ts`. Commit immediately.
5. Phase 2: refactor primitives bottom-up. Use the universal preservation block
   on every prompt. After each: read the diff, run `tsc --noEmit`, spot-check in
   the running app, commit.
6. Phase 3: for tangled files, run the **analyze step** first (read-only listing
   of state, effects, fetches, handlers). Confirm. Then the **write step** —
   JSX only, hooks/imports/data fetching untouched.
7. Phase 4: page-level assembly — by this point most children are already
   correct, so page prompts are mostly composition.

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

- **Code Connect maturity.** The Figma greenfield doc mentions it as the
  long-term drift defense, but doesn't cover the rough edges (per-component
  binding, CI checks for unbound files). Worth a follow-up lesson once it's been
  used on a real project.
- **Where does shadcn MCP fit in a refactor?** The docs cover it for greenfield
  scaffolding but don't explicitly address whether it helps when retro-fitting
  primitives in Phase 2 of a refactor.
- **Context exhaustion thresholds.** The refactor doc names "stop, commit, fresh
  session" as the recovery — but is there a heuristic (file count? minutes?
  prompt count?) that predicts when to bail proactively rather than reactively?
- **Visual verification loop automation.** Chrome DevTools MCP is mentioned as
  the polish loop. Could it be wired into a per-commit screenshot diff against
  the Figma frame or Claude Design canvas, or is that overkill?
- **Claude Design ↔ Figma round-tripping.** No native `.fig` export exists yet.
  Is there a viable workflow for teams that prototype in Claude Design and want
  to hand polish to a Figma-native designer downstream, or is this currently
  one-way?
- **Worth a dedicated workspace?** This is a repeatable workflow, but the README
  here doubles as the runbook. If the same person ends up running it multiple
  times — especially across both routes — promote it to
  `workspaces/design-to-code/` with skills for each phase.
