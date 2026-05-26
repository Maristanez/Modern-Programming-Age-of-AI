# The Design-to-Code Pipeline

> **Dribbble → Figma → MCP → Claude Code**
> A repeatable workflow for translating visual inspiration into pixel-accurate, production-ready Next.js components — without translation loss.

---

## Table of Contents

1. [Why This Pipeline Exists](#1-why-this-pipeline-exists)
2. [Architectural Overview](#2-architectural-overview)
3. [Phase 1 — Source Inspiration (Dribbble)](#3-phase-1--source-inspiration-dribbble)
4. [Phase 2 — Digitize the Math (Figma)](#4-phase-2--digitize-the-math-figma)
5. [Phase 3 — Connect the Pipeline (Figma Remote MCP)](#5-phase-3--connect-the-pipeline-figma-remote-mcp)
6. [Phase 4 — Execute the Build (Claude Code)](#6-phase-4--execute-the-build-claude-code)
7. [Alternative & Complementary MCPs](#7-alternative--complementary-mcps)
8. [Strict Rules & Edge Cases](#8-strict-rules--edge-cases)
9. [Quick Reference Cheat Sheet](#9-quick-reference-cheat-sheet)

---

## 1. Why This Pipeline Exists

Feeding raw Dribbble images directly to an AI agent forces it to **guess** — guess hex codes, guess spacing, guess flex direction. The result is inconsistent styling across components and wasted tokens spent re-prompting.

This pipeline solves that by inserting a structured translation layer:

- **Dribbble** provides the *vibe*.
- **Figma** converts that vibe into *math* (real hex codes, real auto-layout dimensions).
- **MCP** exposes that math as a *data API*.
- **Claude Code** consumes facts, not approximations.

The core principle: **Give the AI structural facts, not visual assumptions.**

---

## 2. Architectural Overview

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌──────────────┐
│   Dribbble  │───▶│    Figma    │───▶│  Figma MCP  │───▶│ Claude Code  │
│   (Vibe)    │    │   (Math)    │    │ (Data API)  │    │  (Next.js)   │
└─────────────┘    └─────────────┘    └─────────────┘    └──────────────┘
   inspiration       auto-layout       structural          pixel-perfect
   reference         + hex tokens      properties          components
```

Each arrow represents a translation step that removes ambiguity before the AI ever touches code.

---

## 3. Phase 1 — Source Inspiration (Dribbble)

- Browse Dribbble (or Mobbin, Behance, etc.) for designs that match your target aesthetic.
- Prioritize shots that clearly expose **grid structure, spacing rhythm, hover states, and component hierarchy** — not just pretty hero images.
- Save references locally. Do **not** pass these PNGs to Claude Code as the source of truth.

> Dribbble is where you decide *what* you want. It is never where the AI gets its instructions from.

---

## 4. Phase 2 — Digitize the Math (Figma)

This is the most critical phase. Skipping it collapses the entire pipeline.

### The Image Node Trap

If you paste a Dribbble PNG into Figma and connect the MCP, the MCP only sees a single **Image Node**. It cannot read the buttons, text, spacing, or colors inside that pixel grid. You will get garbage output.

### What to Do Instead

1. Drop the Dribbble image into a Figma file as a visual reference.
2. **Trace over it** using native Figma primitives — Frames, Text layers, Shapes.
3. **Use Auto-Layout for every container.** The MCP translates Auto-Layout directly into flexbox / grid Tailwind classes. Loose elements floating on a canvas will not generate responsive code.
4. Define exact tokens on these native nodes:
   - Hex colors
   - Border radii
   - Typography scales (family, size, weight, line-height)
   - Spacing values (padding, gap, margin)

### Speed Hack: Reskin a UI Kit

Tracing from scratch is tedious. Instead:

1. Grab a free Tailwind / shadcn UI kit from the [Figma Community](https://www.figma.com/community).
2. Paste it into your file.
3. Reskin its colors, fonts, and radii to match your Dribbble inspiration.

You now have structurally perfect Figma nodes wrapped in your custom styling — ready for the MCP.

---

## 5. Phase 3 — Connect the Pipeline (Figma Remote MCP)

Figma offers two MCP options. **Always prefer the Remote server** — it connects to Figma's hosted endpoints and does not require the desktop app running in the background.

### Installation

Add the Figma Remote MCP at the user scope so it persists across all your projects:

```bash
claude mcp add --scope user --transport http figma https://mcp.figma.com/mcp
```

### Authentication

1. Inside your Claude Code session, run:
   ```
   /mcp
   ```
2. Select the `figma` server and hit Enter.
3. A browser window opens — click **Allow Access**.
4. Return to your terminal. The server status should now show as **connected**.

### Verify It Works

```bash
> List the available tools from the figma MCP server.
```

You should see tools for reading nodes, extracting variables, and inspecting frames.

---

## 6. Phase 4 — Execute the Build (Claude Code)

Run this in two distinct steps. Establish the design system *first*, then build components against it.

### Step 1 — Ingest Tokens Globally

```
> Use the Figma MCP to inspect [Figma file URL or frame ID].
> Extract the complete color palette, typography scales, border radii,
> and spacing variables. Update tailwind.config.ts so these become the
> single source of truth for the entire app.
```

This locks down your design tokens before any component is written. Every subsequent component will reference these variables instead of hardcoded values.

### Step 2 — Build Components Against the Tokens

```
> Using the Figma MCP, inspect Frame [ID]. Build this as a responsive
> Next.js Server Component in app/components/hero-section.tsx.
> Use only the tokens defined in tailwind.config.ts. Translate the
> Auto-Layout properties into matching flex/grid Tailwind classes.
```

### Optional Step 3 — Lock the System with Code Connect

As your design system matures, use Figma's **Code Connect** feature through the MCP to bind your React components directly to their Figma counterparts. This prevents styling drift over time — when the Figma component updates, the AI knows which code file owns it.

---

## 7. Alternative & Complementary MCPs

Figma is the gold standard for design accuracy, but other MCPs cover adjacent use cases.

### Playwright / Puppeteer MCP — Live Site Inspection

When your inspiration lives on a real website rather than a Figma file, a browser-automation MCP gives Claude Code a headless browser. It can inspect the DOM and read computed CSS directly from the live page.

**Best for:** Copying layout mechanics, animations, and spacing scales from production sites.

### shadcn/ui MCP — Component Scaffolding

Gives Claude direct access to the shadcn registry. Combined with your Figma tokens, this lets the AI scaffold battle-tested accessible components instantly, then restyle them to match your design system.

**Best for:** Eliminating boilerplate on standard primitives (buttons, dialogs, forms).

### Chrome DevTools MCP — Visual Debugging

Lets Claude open your local dev server, screenshot it, inspect rendered styles, and iterate visually. Pairs perfectly with the Figma MCP — Figma tells the AI what to build, Chrome DevTools tells it whether the build matches.

**Best for:** The final polish loop where you compare rendered output against the Figma reference.

### The Stack

For maximum accuracy, run these together:

- **Figma MCP** — source of truth for design tokens and structure.
- **shadcn MCP** — accessible primitives to build on top of.
- **Chrome DevTools MCP** — visual verification loop.

---

## 8. Strict Rules & Edge Cases

| Rule | Why |
|------|-----|
| **Never mix Dribbble images with Figma context during the build phase.** | Causes context conflict. The AI tries to reconcile two sources of truth and hallucinates. Keep Dribbble in your browser; only feed Figma data to the agent. |
| **Keep ingested Figma frames focused.** | A "Hero Section" frame ingests cleanly. A full marketing page with 400 nested nodes will overflow the context window and produce garbage. Build component by component. |
| **Always run Phase 4 Step 1 before Step 2.** | If tokens aren't in `tailwind.config.ts` first, every component will hardcode hex values and your design system will drift immediately. |
| **Use Auto-Layout everywhere in Figma.** | Without it, the MCP returns absolute positioning data. Claude will generate non-responsive `position: absolute` CSS. |
| **Re-verify after major Figma edits.** | If you redesign a frame, re-run token extraction. Stale tokens silently break consistency. |

---

## 9. Quick Reference Cheat Sheet

### Setup (One-Time)

```bash
claude mcp add --scope user --transport http figma https://mcp.figma.com/mcp
# Then inside Claude Code:
/mcp   # authenticate via browser
```

### Per-Project Workflow

```
1. Find inspiration on Dribbble                    [Browser]
2. Trace it in Figma with Auto-Layout + tokens     [Figma]
3. Connect MCP, authenticate                       [Terminal]
4. Extract tokens → tailwind.config.ts             [Claude Code]
5. Build components, frame by frame                [Claude Code]
6. Verify with Chrome DevTools MCP (optional)      [Claude Code]
```

### The One Rule to Remember

> **Dribbble decides what. Figma defines how. MCP delivers facts. Claude Code writes code.**
> Never let the AI skip a step.

---

*Last updated: 2026*
