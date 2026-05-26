# The Claude Design Pipeline

> **Replaces the Dribbble → Figma → MCP path.**
> A native end-to-end workflow for going from idea → visual design → production code, all inside Anthropic's stack.
> Companion to [`design-to-code-pipeline.md`](./design-to-code-pipeline.md) and [`refactoring-slop-codebase.md`](./refactoring-slop-codebase.md).

---

## Table of Contents

1. [What Claude Design Actually Is](#1-what-claude-design-actually-is)
2. [How It Changes the Pipeline](#2-how-it-changes-the-pipeline)
3. [Architectural Overview](#3-architectural-overview)
4. [Prerequisites & Access](#4-prerequisites--access)
5. [Phase 1 — Set Up Your Design System (Once)](#5-phase-1--set-up-your-design-system-once)
6. [Phase 2 — Generate the Design](#6-phase-2--generate-the-design)
7. [Phase 3 — Refine on the Canvas](#7-phase-3--refine-on-the-canvas)
8. [Phase 4 — Handoff to Claude Code](#8-phase-4--handoff-to-claude-code)
9. [Phase 5 — Build & Ship](#9-phase-5--build--ship)
10. [Using Claude Design on an Existing Slop Codebase](#10-using-claude-design-on-an-existing-slop-codebase)
11. [Claude Design vs Figma MCP — When to Use Which](#11-claude-design-vs-figma-mcp--when-to-use-which)
12. [Limitations & Failure Modes](#12-limitations--failure-modes)
13. [Quick Reference Cheat Sheet](#13-quick-reference-cheat-sheet)

---

## 1. What Claude Design Actually Is

Claude Design is a separate Anthropic Labs product that lives at its own URL (claude.ai/design), with its own canvas, export options, and metered usage. It's powered by Claude Opus 4.7 and is currently in research preview.

It gives you a split interface: a chat panel on the left, a live canvas on the right. You describe what you want, Claude builds it on the canvas, and you refine through a mix of chat, inline comments on specific elements, direct text edits, and auto-generated sliders for spacing, color, and layout.

The core product surface:

- **Canvas + chat** — describe, generate, refine in the same view.
- **Design system ingestion** — reads your codebase + existing design files to build a token library automatically.
- **Web capture tool** — grab elements directly from a live website to use as reference.
- **Multi-format export** — HTML, PDF, PPTX, Canva, .zip.
- **Claude Code handoff** — the one export that actually changes the workflow.

The killer feature: during onboarding, Claude Design reads your codebase and your existing design files, then builds a design system from both. Brand colors, typography, spacing tokens, component patterns. That system is then applied automatically to every new project.

---

## 2. How It Changes the Pipeline

Here's the brutal honest comparison against the Figma MCP workflow:

| Step | Figma MCP Pipeline | Claude Design Pipeline |
|------|-------------------|----------------------|
| Find inspiration | Dribbble browsing | Dribbble browsing (or skip — describe it) |
| Digitize as structured data | Manually trace in Figma with Auto-Layout | Describe in chat, Claude generates on canvas |
| Extract design tokens | Run Figma MCP to extract → update `tailwind.config.ts` | Claude Design reads your repo at onboarding |
| Refine the design | Edit Figma layers manually | Chat + inline comments + adjustment sliders |
| Connect to code | Install MCP, authenticate, point at frame | Click "Handoff to Claude Code" |
| Generate code | Claude Code reads MCP | Claude Code reads handoff bundle |

**What collapses:** Phases 1–3 of the original Figma pipeline merge into one conversation. No manual tracing. No MCP setup. No token extraction prompt.

**What stays the same:** Once code generation begins, every rule from the refactoring strategy doc still applies — atomic commits, preservation blocks, the analyze-then-write pattern. Claude Design changes where the design lives. It does not change how messy codebases get cleaned up.

---

## 3. Architectural Overview

```
┌─────────────┐   ┌─────────────────┐   ┌──────────────┐   ┌──────────────┐
│  Reference  │──▶│  Claude Design  │──▶│   Handoff    │──▶│ Claude Code  │
│   (vibe)    │   │   (canvas +     │   │   Bundle     │   │   (Next.js,  │
│             │   │  design system) │   │  (single cmd)│   │   GitHub)    │
└─────────────┘   └─────────────────┘   └──────────────┘   └──────────────┘
  Dribbble,         describe + refine     design files +     production
  screenshots,      with chat, comments,   tokens + intent    code with
  live sites,       and sliders            packaged together  design intent
  your repo                                                   preserved
```

The pipeline has one less translation layer than the Figma path. The bridge between "this is what it should look like" and "now make it real" is a single clipboard action.

---

## 4. Prerequisites & Access

| Requirement | Detail |
|-------------|--------|
| **Subscription** | Pro, Max, Team, or Enterprise |
| **URL** | claude.ai/design |
| **Enterprise users** | Enterprise admins have to enable it — it is off by default for them |
| **Claude Code** | Required for the handoff step. Install via Anthropic's instructions. |
| **GitHub repo** | Needed if you want the design system ingestion to pull from your existing code. |

---

## 5. Phase 1 — Set Up Your Design System (Once)

This is the step that makes everything else work. Skipping the design system step is one of the top three reasons people get frustrated with Claude Design.

### Option A — Ingest from an Existing Codebase

Connect your GitHub repo at onboarding. Claude Design will:

- Parse your `tailwind.config.ts`, CSS variables, and existing components.
- Extract brand colors, typography scales, spacing tokens, and component patterns.
- Build a reusable design system that applies to every new project in this workspace.

This is the path to take if you already have a codebase — even a slop one. Better tokens go in than you'd get from a blank slate.

### Option B — Describe a New Design System

If you're starting fresh or want a different aesthetic:

```
> Set up a design system for a B2B SaaS product called [Name].
> Brand: [modern, restrained, technical / playful, warm, consumer].
> Primary color: [hex or description].
> Typography: [serif/sans, classical/geometric, etc.].
> Reference aesthetic: [Linear, Vercel, Stripe, etc.].
```

### Option C — Import References

Start from a text prompt, upload images and documents (DOCX, PPTX, XLSX), or point Claude at your codebase. You can also use the web capture tool to grab elements directly from your website so prototypes look like the real product.

**Recommended combination:** ingest your repo (Option A) + capture 2–3 reference URLs (Option C). This grounds the design system in both your existing constraints and your aspirational target.

### Verify the System

Before generating anything, ask:

```
> Show me the design system you've built. List the color tokens,
> typography scale, spacing values, and any component patterns
> you've identified.
```

Read it. Correct it. Lock it in before you generate a single screen.

---

## 6. Phase 2 — Generate the Design

Claude Design performs better when the prompt names the artifact type, the target audience, the content structure, and the main constraints. Early hands-on testing has pointed to the same pattern: dense prompts outperform vague ones.

### The Structured Brief Template

```
ARTIFACT: [Landing page / Dashboard / Mobile onboarding flow / etc.]
AUDIENCE: [Who is this for]
SECTIONS / SCREENS: [List them explicitly]
KEY ACTIONS: [What should the user be able to do]
CONSTRAINTS: [Responsive breakpoints, accessibility, brand rules]
TONE: [Restrained / Playful / Authoritative / etc.]
```

**Example:**

```
ARTIFACT: Dashboard for a peptide tracking app.
AUDIENCE: Power users logging multiple stacks daily.
SCREENS:
  1. Today view (current stack, doses due, quick log)
  2. Stack library (saved protocols)
  3. Analytics (sleep, recovery, mood correlations)
CONSTRAINTS: Mobile-first, dark mode default, keyboard navigable.
TONE: Clinical but warm. Not a medical chart — a personal coach.
```

Hit send. Claude generates the design on the canvas.

---

## 7. Phase 3 — Refine on the Canvas

This is where Claude Design beats Figma for speed. You have three refinement surfaces:

| Surface | When to Use |
|---------|-------------|
| **Chat** | Structural changes, layout direction, big aesthetic shifts ("make this feel more like Linear"). |
| **Inline comments** | Local changes — spacing on one button, copy on one card, swap a component. |
| **Adjustment sliders** | Live tweaks to spacing, color, layout dimensions without re-prompting. |

Anthropic recommends using chat for structure, layout direction, and bigger aesthetic changes, and inline comments for local changes like spacing, button treatment, or component swaps.

### The Iteration Loop

1. Generate → review on canvas.
2. Big issues? Chat: "The hero section feels cramped — increase vertical breathing room and lighten the gradient."
3. Small issues? Click the element, comment: "Make this button radius match the cards above."
4. Polish? Use the adjustment knobs for spacing and color.
5. Once a section is right, ask Claude to apply that pattern across the full design.

### Don't Skip Motion

Claude Design's ability to turn static designs into interactive prototypes has been a step change. Add motion before handoff — Claude Code reads animation intent and translates it into GSAP / Framer Motion. Far better than describing animations after the fact.

```
> Add a fade-in stagger to the stat cards on load. Hover state should
> lift the card 4px with a soft shadow. Page transitions should slide
> from the right with a 200ms ease.
```

---

## 8. Phase 4 — Handoff to Claude Code

This is the moment the workflow stops being "design tool" and becomes "production pipeline."

### Triggering the Handoff

When your design is ready, click Export → Handoff to Claude Code. This creates a bundle that includes your design files, the design system tokens, the component structure, and the intent behind each page.

### What the Bundle Contains

| Component | Why It Matters |
|-----------|---------------|
| Design files | The actual visual artifacts. |
| Design system tokens | Color, type, spacing — already extracted, no MCP needed. |
| Component structure | How the canvas elements map to a React component tree. |
| Page intent | The "why" behind each screen — preserved for the code agent. |

### Executing the Handoff

You export from Claude Design, copy a command, paste it into Claude Code, and it fetches the design file from an API endpoint and starts building.

```bash
# In your terminal, inside your repo:
claude
# Then paste the handoff command from Claude Design
> /design:import [handoff-url-or-token]
```

Claude Code will:

1. Fetch the bundle from the API endpoint.
2. Read the design system tokens and update your `tailwind.config.ts` if needed.
3. Scaffold the component tree based on the canvas structure.
4. Begin implementing screen by screen.

---

## 9. Phase 5 — Build & Ship

For **greenfield projects** the handoff often produces a near-complete first pass. But here's the catch: the first output is often rough. And I mean noticeably rough. Treat it as a strong scaffold, not a finished product.

The build loop:

1. Let Claude Code generate the first pass from the handoff.
2. Run the dev server. Compare side-by-side against the Claude Design canvas.
3. Use Chrome DevTools MCP (or screenshots) to spot mismatches.
4. Refine incrementally — one component at a time, with the universal preservation block from the refactoring doc.
5. Commit between each refinement.

Then:

```bash
git push
# Deploy to Vercel
vercel --prod
```

---

## 10. Using Claude Design on an Existing Slop Codebase

This is your specific situation. Read carefully.

### The Hard Truth

Anthropic is transparent about the product's limitations: the design system import works best with a clean codebase; messy source code produces messy output.

**Claude Design does not solve the refactor problem.** It is a better design tool, not a magic spell. If you point it at a slop repo and click "Handoff to Claude Code → redesign everything," you will get the same destruction patterns as the Figma path: prop dropping, lost business logic, broken state.

### The Right Way to Combine Claude Design with a Slop Codebase

Run them in series, not in parallel:

**Stage 1 — Use Claude Design for the Visual Target**

- Ingest your existing repo at onboarding (this still gives Claude Design useful context, even if messy).
- Generate the *target design* on the canvas — what you want the app to look like after the refactor.
- Refine until the canvas reflects your real goal.
- Do NOT trigger the handoff yet.

**Stage 2 — Export Design System Only**

Export just the design tokens — colors, typography, spacing — and apply them to your `tailwind.config.ts`. Commit.

```bash
git commit -m "refactor(tokens): adopt Claude Design tokens as source of truth"
```

**Stage 3 — Run the Refactoring Strategy**

Now switch to the discipline from [`refactoring-slop-codebase.md`](./refactoring-slop-codebase.md):

- Phase 0 audit (codebase + Claude Design canvas, not Figma).
- Phase 2 atomic component refactoring, referencing the Claude Design canvas as the visual target.
- Phase 3 logic/visual split for messy files.
- Universal preservation block on every prompt.
- Commit after every component.

**Stage 4 — Spot-Check Against the Canvas**

For each refactored component, open Claude Design on one monitor and your dev server on the other. The canvas is your ground truth.

### Why Not Just Hand Off the Whole Thing?

Because the handoff assumes Claude Code is *generating* the components, not *replacing existing ones with tangled business logic*. The handoff bundle has zero awareness of your existing `useEffect` chains, your auth guards, your data fetching patterns. It will overwrite them.

Use the handoff for **new screens you're adding**. Use the discipline for **existing screens you're refactoring**.

---

## 11. Claude Design vs Figma MCP — When to Use Which

| Situation | Recommended Path |
|-----------|------------------|
| Greenfield project, no existing designs | **Claude Design** — fastest path to working prototype. |
| You have a mature Figma file with component library | **Figma MCP** — leverage existing structured assets. |
| Team includes traditional designers who live in Figma | **Figma MCP** — match their workflow. |
| Solo founder / PM / engineer prototyping | **Claude Design** — eliminates the design-handoff translation. |
| Existing slop codebase needing redesign | **Claude Design for visual target + refactor discipline for code** (see Section 10). |
| Need pixel-precise control over a specific element | **Figma MCP** — more granular file-level precision. |
| Need to prototype voice, video, shaders, 3D, AI features | **Claude Design** — frontier prototyping is a native capability. |

Figma still wins where teams need high-precision production design, mature collaboration, and established file workflows. Claude Design looks strongest earlier in the process, where direction, exploration, and branded first drafts matter more than file-level precision.

They are not mutually exclusive. Many teams use Claude Design for exploration and Figma for production polish.

---

## 12. Limitations & Failure Modes

| Limitation | Mitigation |
|------------|-----------|
| First handoff output is rough | Treat as scaffold, not final. Refine with the build loop in Phase 5. |
| Design system import struggles on slop codebases | Use the staged approach in Section 10 — don't expect a one-shot redesign. |
| No native design file export (no `.fig` file) | Export to HTML or rely on the handoff bundle. Live with the lock-in for now. |
| Visual quality ceiling vs. handcrafted design | Claude Design addresses the space before certainty exists — use it for direction and speed, not final-stage craft. |
| Collaboration is basic | Collaboration is basic and not yet fully multiplayer. The editing experience has rough edges. Don't plan team workflows around it yet. |
| Skipping the design system step | Always run Phase 1 first. Generated screens without a locked-in system drift across every prompt. |
| Treating the canvas as the source of truth post-handoff | Once code exists, the code is the source of truth. Use the canvas as a reference, not a live link. |

---

## 13. Quick Reference Cheat Sheet

### Greenfield Workflow

```
1. claude.ai/design → New project
2. Set up design system (ingest repo + reference URLs)
3. Write structured brief (artifact, audience, sections, constraints)
4. Generate → refine with chat + comments + sliders
5. Add motion/interactivity
6. Export → Handoff to Claude Code
7. Paste handoff command into Claude Code in terminal
8. Build, test, commit, deploy
```

### Slop Codebase Workflow

```
1. claude.ai/design → ingest existing repo
2. Generate target design on canvas
3. Refine until canvas matches your real goal
4. Export design tokens → update tailwind.config.ts
5. Commit tokens
6. Switch to refactoring-slop-codebase.md discipline:
   - Phase 0 audit
   - Atomic refactors with preservation block
   - Logic/visual split for complex files
   - Commit after every component
7. Use canvas as ground-truth reference during refactor
```

### The Three Rules

> **1. Design system first. Always. No exceptions.**
> **2. Dense, structured briefs outperform vague ones.**
> **3. The handoff is for new code. The refactor discipline is for existing code.**

---

*Companion to [`design-to-code-pipeline.md`](./design-to-code-pipeline.md) (Figma MCP path) and [`refactoring-slop-codebase.md`](./refactoring-slop-codebase.md) (brownfield discipline). Last updated: 2026.*
