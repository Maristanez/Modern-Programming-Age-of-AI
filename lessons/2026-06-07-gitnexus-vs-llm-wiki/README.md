# GitNexus vs. LLM Wiki — Two Kinds of External Brain for AI Agents

**Date:** 2026-06-07
**Source:** Comparison notes (GitNexus + Karpathy-style "LLM Wiki"). Primary references:
[GitNexus repo](https://github.com/abhigyanpatwari/GitNexus),
[GitNexus research writeup](https://rywalker.com/research/gitnexus),
[Obsidian](https://obsidian.md/).
**Type:** Tooling comparison / decision guide

## TL;DR

GitNexus and an "LLM Wiki" are **not competitors** — they solve different problems and
are often run together. GitNexus is a **code intelligence engine**: it parses a repo into
a knowledge graph (ASTs, call chains, imports) so an agent can query *structure* instead
of dumping the whole codebase into context. An LLM Wiki is a **general-purpose memory
system**: a folder of Markdown files an agent reads from and writes to, accumulating
research, decisions, and notes across sessions. Pick GitNexus when agents hallucinate
about dependencies or break code during refactors; pick an LLM Wiki when agents "forget"
prior research and decisions. Use both when you want structural truth *and* conceptual
truth side by side.

## Why it matters

Two of the most common failure modes when working with AI coding/research agents are:

1. **Structural blindness** — the agent doesn't actually know how the code fits together,
   so it hallucinates function signatures, misses callers, or breaks things during a
   refactor because it can't see the blast radius.
2. **Amnesia** — the agent starts every session from zero. Research, constraints, and
   decisions from days ago are gone, so you re-explain context over and over.

These look similar ("the agent doesn't know enough") but have *opposite* fixes. The first
is solved by giving the agent a precise, queryable model of the code's structure. The
second is solved by giving the agent a durable, human-readable place to write down what it
learns. Conflating them leads to picking the wrong tool. This lesson keeps them straight.

## How it works

### Side-by-side

| Feature | GitNexus | LLM Wiki (Karpathy-style) |
| --- | --- | --- |
| **Core purpose** | Code intelligence, structural mapping, dependency analysis | Long-term context storage, research compounding, memory |
| **Primary data** | Source code (ASTs, call chains, imports) | Text: research papers, docs, web clips, notes |
| **Underlying tech** | Graph DB (LadybugDB / KuzuDB), Tree-sitter, Model Context Protocol (MCP) | Markdown files + Git, often SQLite / hybrid BM25 search |
| **Agent role** | Graph RAG — agent runs structural queries | Read/Write RAG — agent actively updates the knowledge base |
| **Best for** | Refactoring, blast-radius analysis, deep audits | Aggregating research, multi-session memory |

### GitNexus — the code intelligence engine

A zero-server engine that maps a repository's architecture into a knowledge graph and acts
as an "expert guide" for AI coding agents.

- **Mechanism:** A Tree-sitter / AST parser maps how code connects — dependencies,
  function calls, execution flows — into graph nodes and edges.
- **Why it helps coding:**
  - **Blast-radius analysis:** Before an agent makes a change, it queries the graph to see
    which functions and upstream modules could break.
  - **Reduced token usage:** Instead of feeding the whole codebase to the LLM, the agent
    pulls only the relevant nodes/edges — saving context window.
  - **Architectural insight:** The agent reasons about the code's *structure and intent*,
    not just surface-level text.
- **Best scenario:** Actively refactoring or building complex software with assistants
  like Claude Code, [Cursor](https://cursor.com/), or [Windsurf](https://codeium.com/windsurf).

### LLM Wiki — the persistent external brain

Inspired by concepts popularized by Andrej Karpathy: treat a directory of Markdown files as
a persistent "external brain" the agent reads from and writes to.

- **Mechanism:** Agents get read/write access to a folder of Markdown. Over time they add
  notes, summaries, and findings, compounding a body of knowledge.
- **Why it helps research:**
  - **Human-readable memory:** It's just Markdown — open it in [Obsidian](https://obsidian.md/)
    (or any editor) to review what the AI has learned.
  - **Compounding context:** The agent reads its previous notes instead of starting fresh
    each session.
  - **Broad ingestion:** Not restricted to code — research papers, articles, docs all fit.
- **Best scenario:** Long-term projects and research that span many sessions or weeks,
  where maintaining context and decisions matters.

### They compose

The two are frequently used in tandem:

- **Bridge feature:** GitNexus can auto-generate a static **code wiki** from its knowledge
  graph — exporting structural insight into a Markdown form an LLM Wiki can read.
- **Division of labor:** Let GitNexus own the code architecture (the *structural truth*)
  while the LLM Wiki holds high-level project notes, research findings, and TODOs (the
  *conceptual truth*).

> Note: this repo itself is an LLM-Wiki-shaped system — a growing tree of Markdown lessons
> and workspaces with no backend. The [folder-system-architecture lesson](../2026-05-13-folder-system-architecture/)
> is the structural pattern behind it.

## How to apply it

Decide by your **primary symptom**, then optionally layer the second tool:

1. **Agents hallucinate dependencies / break code on refactor → start with GitNexus.**
   Point it at the repo, let it build the graph, and expose it to your agent over MCP. Have
   the agent query the graph for callers/blast radius *before* editing.
2. **Agents forget prior research, constraints, or decisions → start with an LLM Wiki.**
   Create a Markdown folder, give the agent read/write access, and make "read the wiki
   first, write findings back" part of every session's loop. Review in Obsidian.
3. **Both problems on a complex, long-running codebase → run them together.** Use GitNexus
   for structural truth; auto-generate its code wiki and keep your conceptual notes
   (decisions, research, TODOs) in the LLM Wiki alongside it.
4. **Just getting started with an agent-friendly environment?** Stand up the LLM Wiki first
   (cheap, immediate memory payoff), then add GitNexus once the codebase is large enough
   that structural queries earn their keep.

## Open questions / follow-ups

- **Graph freshness:** How does GitNexus keep the graph in sync as code changes — on-demand
  reparse, watch mode, or manual rebuild? Staleness would undermine blast-radius queries.
- **Wiki write discipline:** Unstructured agent writes can rot a Markdown wiki over time.
  What conventions (frontmatter, indexes, pruning) keep it queryable? Compare with this
  repo's lesson/workspace conventions and the memory-file pattern.
- **Retrieval quality:** LLM Wiki search is often BM25/hybrid; how does that compare to
  embedding-based retrieval for surfacing the right note at the right time?
- **Could the two merge?** If GitNexus exports a code wiki, is there value in a single tool
  that unifies structural graph + conceptual notes, or does separation of concerns win?
- Verify the underlying-tech details (LadybugDB / KuzuDB) against current GitNexus docs —
  these were drawn from secondary notes.
