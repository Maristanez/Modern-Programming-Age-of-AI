# Refactoring a "Slop" Codebase with the Figma MCP Pipeline

> **Companion doc to [`design-to-code-pipeline.md`](./design-to-code-pipeline.md)**
> How to redesign an existing app's UI without nuking your business logic.

---

## Table of Contents

1. [The Honest Answer Up Front](#1-the-honest-answer-up-front)
2. [Why "Redesign Everything" Fails](#2-why-redesign-everything-fails)
3. [Core Principle: Manage the Blast Radius](#3-core-principle-manage-the-blast-radius)
4. [Phase 0 — Assess Before You Touch](#4-phase-0--assess-before-you-touch)
5. [Phase 1 — Establish Global Truth](#5-phase-1--establish-global-truth)
6. [Phase 2 — Atomic Component Refactoring](#6-phase-2--atomic-component-refactoring)
7. [Phase 3 — The Logic vs. Visual Split](#7-phase-3--the-logic-vs-visual-split)
8. [Phase 4 — Page-Level Assembly](#8-phase-4--page-level-assembly)
9. [Golden Rules](#9-golden-rules)
10. [Failure Modes & Recovery](#10-failure-modes--recovery)
11. [Quick Reference Cheat Sheet](#11-quick-reference-cheat-sheet)

---

## 1. The Honest Answer Up Front

**Can you point Claude Code at a slop codebase and say "redesign everything to match this Figma file"?**

No. Not in one shot. Not even close.

If you do, you will get:
- Hallucinated component APIs that don't exist in your codebase.
- Silently deleted business logic (auth checks, data fetches, side effects).
- Broken prop chains where the new UI forgets that `isLoading`, `disabled`, or `onSubmit` ever existed.
- A blown context window halfway through the job, leaving your repo in a half-refactored, non-compiling state.

**But you can absolutely redesign the whole app.** You just have to act as the project manager and force the work to happen incrementally. The AI is the contractor; you control the scope of every work order.

---

## 2. Why "Redesign Everything" Fails

Claude Code is bounded by two hard limits:

| Limit | What It Means for You |
|-------|----------------------|
| **Context window** | The AI can only hold so much of your codebase in working memory at once. A sweeping refactor exceeds this and the AI starts forgetting earlier files. |
| **Coupling complexity** | In "slop" code, UI markup is tangled with state management, data fetching, and routing. The AI cannot cleanly separate "visual layer" from "logic layer" without explicit instructions. |

The fix isn't a smarter prompt — it's a smaller prompt, run many times, with strict scope.

---

## 3. Core Principle: Manage the Blast Radius

Every prompt you give Claude during a refactor should answer three questions:

1. **What exactly are you changing?** (One file? One component? One token?)
2. **What must NOT change?** (Props, hooks, side effects, exported APIs.)
3. **What is the source of truth?** (Which Figma frame? Which token file?)

If a prompt is vague on any of these, the AI will improvise — and improvisation in a refactor means destruction.

---

## 4. Phase 0 — Assess Before You Touch

Before any refactoring, get the AI to map the territory. This is a read-only phase.

### Step 0.1 — Audit the Existing Codebase

```
> Do not modify any files. Analyze the codebase and produce a report at
> docs/refactor-audit.md containing:
>
> 1. A list of all UI components in components/ and their current responsibilities.
> 2. Which components are "pure" (presentational) vs which mix logic and UI.
> 3. The current styling approach (inline styles, CSS modules, Tailwind, etc.)
>    and any inconsistencies.
> 4. Components that are reused in 3+ places (high-risk refactor targets).
> 5. Components only used once (safe refactor targets to start with).
```

This audit becomes your roadmap. Refactor the **safe targets first** to build confidence in the pipeline, then graduate to the high-risk components.

### Step 0.2 — Audit the Figma File

```
> Use the Figma MCP to inspect [Figma file URL]. Produce a report at
> docs/figma-inventory.md listing every component frame, its purpose,
> and which existing codebase component it likely maps to.
```

Now you have a two-column mapping: *current slop component → target Figma frame*. This is your refactor backlog.

### Step 0.3 — Create a Refactor Branch

```bash
git checkout -b refactor/figma-redesign
```

Never refactor on `main`. Every component refactor gets its own commit on this branch.

---

## 5. Phase 1 — Establish Global Truth

Before any component is touched, lock down the design tokens. This must happen first or every component refactor will hardcode slightly different values.

```
> Use the Figma MCP to extract the complete design token set from
> [Figma file URL]: colors, typography scales, border radii, spacing,
> shadows. Update tailwind.config.ts and app/globals.css to make these
> the single source of truth.
>
> Do not touch any component files in this prompt.
```

**Commit immediately.** This is the foundation every subsequent prompt builds on.

```bash
git add tailwind.config.ts app/globals.css
git commit -m "refactor(tokens): sync design tokens with Figma source of truth"
```

---

## 6. Phase 2 — Atomic Component Refactoring

Start at the bottom of the component tree. Buttons. Inputs. Badges. Cards. The dumb, primitive components that everything else depends on.

### The Atomic Prompt Template

```
> Use the Figma MCP to inspect the [ComponentName] frame in [Figma file].
> Find the existing component at [exact/path/to/Component.tsx].
>
> Rewrite ONLY the JSX and Tailwind classes to match the Figma design,
> using the tokens already in tailwind.config.ts.
>
> Strict preservation requirements:
> - Do not change the component's exported name or default export.
> - Do not change the TypeScript props interface.
> - Do not remove or rename any props.
> - Preserve all event handlers (onClick, onChange, onSubmit, etc.).
> - Preserve all ref forwarding.
> - Preserve all aria-* and accessibility attributes.
>
> Do not modify any other files.
```

### Why the Strict Preservation Block Matters

The most common AI failure during redesigns is **prop dropping** — the AI builds a beautiful new button and silently forgets to wire up the `isLoading` spinner state, or drops the `disabled` styling, or removes the `aria-label` forwarding. The preservation block prevents this.

### Verification Loop

After each atomic refactor:

1. Read the diff manually. Confirm the props interface is identical.
2. Run `npm run build` or `tsc --noEmit` to catch type errors.
3. Visually spot-check one usage site in the running app.
4. Commit.

```bash
git add components/ui/Button.tsx
git commit -m "refactor(ui): redesign Button to match Figma, preserve API"
```

---

## 7. Phase 3 — The Logic vs. Visual Split

Once your primitive components are clean, you'll hit the messy files — `Dashboard.tsx`, `Settings.tsx`, the 800-line `UserProfile.tsx` with hooks, fetches, and JSX all tangled together.

For these, you must explicitly tell Claude where the seam is.

### The Logic/Visual Split Prompt

```
> I want to redesign [file/path/Dashboard.tsx] to match the Figma
> Dashboard frame via the MCP.
>
> Step 1: Before writing any code, analyze the current file and list
> for me:
>   - All useState / useReducer calls
>   - All useEffect and other side-effect hooks
>   - All data fetching (fetch, axios, react-query, server actions)
>   - All event handlers and callbacks
>   - All imports
>
> Step 2: Wait for my confirmation before proceeding.
```

**Read the analysis.** This is the AI showing you its understanding of what must be preserved. If it missed something, correct it before letting it write code.

Then:

```
> Confirmed. Now rewrite ONLY the return (...) JSX block to match the
> Figma layout. Translate Auto-Layout properties into matching flex/grid
> Tailwind classes using tokens from tailwind.config.ts.
>
> Do not modify any hook calls, data fetching, state variables, or
> handler functions. Do not change any imports unless adding new UI
> primitives from components/ui/.
>
> Show me the diff before writing.
```

The two-step analyze-then-write pattern is the single highest-leverage technique for surviving complex file refactors.

---

## 8. Phase 4 — Page-Level Assembly

Once primitives and complex components are refactored, page-level changes become almost trivial — most of the page is now composed of components that already match Figma.

At this stage you can give Claude slightly broader scope:

```
> Refactor the layout of app/(dashboard)/page.tsx to match the Figma
> Dashboard frame. The child components (Sidebar, Header, StatCard,
> ActivityFeed) have already been refactored to match Figma — use them
> as-is. Only adjust the page-level layout, spacing, and composition.
```

If you've done Phases 1–3 properly, Phase 4 is mostly arranging finished Lego bricks.

---

## 9. Golden Rules

| Rule | Why It Matters |
|------|---------------|
| **Commit after every successful refactor.** | If the next prompt butchers a file, `git restore` recovers in 2 seconds. Without commits, you're untangling a 15-file mess. |
| **One or two files per prompt. Maximum.** | "Refactor the Sidebar" is fine. "Refactor the navigation system" is a disaster. |
| **Always include the preservation block.** | Prop dropping is the #1 failure mode. Explicit preservation requirements eliminate it. |
| **Run the type checker between prompts.** | `tsc --noEmit` catches broken prop chains before they compound across files. |
| **Refactor bottom-up: primitives → composites → pages.** | Top-down refactors force the AI to refactor children it hasn't seen yet, which causes hallucination. |
| **Never let Claude refactor and add features in the same prompt.** | Refactors should be visual-only. Behavior changes get their own prompts, on their own commits. |
| **Read every diff before accepting.** | The AI is a contractor. You are the inspector. Diffs are your inspection. |

---

## 10. Failure Modes & Recovery

| Failure | Symptom | Recovery |
|---------|---------|----------|
| **Prop dropping** | A previously working feature (loading spinner, disabled state) silently stops working. | `git restore` the file. Re-prompt with a more explicit preservation block listing the dropped prop by name. |
| **Hallucinated imports** | Build fails with "Cannot find module" for components that don't exist. | The AI invented a component it expected to find. Either create the component or re-prompt with the actual available imports. |
| **Token drift** | Hex codes appear hardcoded in components instead of Tailwind classes. | Re-prompt: "Replace any hardcoded color/spacing values with the corresponding tokens from tailwind.config.ts." |
| **Context exhaustion mid-refactor** | Output gets truncated or the AI starts forgetting earlier instructions. | Stop. Commit what works. Start a fresh session for the next component. |
| **Logic accidentally rewritten** | A `useEffect` now has different dependencies, or a fetch call uses a different endpoint. | `git restore`. Re-run Phase 3's analyze-then-write pattern. The AI skipped the analysis step. |
| **Layout breaks responsiveness** | Desktop looks right, mobile is broken. | Re-prompt with explicit breakpoints: "Ensure the layout uses Tailwind's `sm:`, `md:`, `lg:` prefixes to match the Figma mobile/tablet/desktop frames." |

---

## 11. Quick Reference Cheat Sheet

### The Refactor Loop

```
1. git checkout -b refactor/figma-redesign     [once, per refactor effort]
2. Audit codebase + Figma → produce mapping    [Phase 0]
3. Sync tokens to tailwind.config.ts           [Phase 1]
4. Refactor primitives, one at a time          [Phase 2]
   ├─ Atomic prompt with preservation block
   ├─ tsc --noEmit
   ├─ visual spot-check
   └─ git commit
5. Refactor complex files with logic/visual split  [Phase 3]
   ├─ Analyze step (read-only)
   ├─ Write step (JSX only)
   └─ git commit
6. Refactor page layouts                        [Phase 4]
7. Merge to main when stable
```

### The Universal Preservation Block

Paste this into any component refactor prompt:

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

### The One Rule to Remember

> **You are the project manager. Claude is the contractor.**
> **Figma is the blueprint. Git is the safety net.**
> **Small scope, strict preservation, constant commits.**

---

*Companion to [`design-to-code-pipeline.md`](./design-to-code-pipeline.md). Last updated: 2026.*
