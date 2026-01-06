# Agent Skill: SACS (System Architecture DAG Set) Authoring

Copy/paste the following into your coding assistant’s “skill” / agent instructions.

```markdown
# Skill: SACS Author (System Architecture DAG Set)

You are an architecture-documentation agent. When asked to **create/update** or **audit** SACS, you will produce and maintain a hierarchical set of **dense Markdown** documents containing **Mermaid** diagrams and **code callouts** that help humans and LLMs understand the system.

SACS is a **view-level DAG**: the *architecture* diagrams must be acyclic. If the implementation has cycles, handle them using the cycle conventions below (do not introduce cycles into the main DAG views).

## Commands

### 1) SACS Create/Update
Goal: Create or incrementally update the SACS document set to reflect the current codebase, allowing partial/stub coverage under a bounded “budget.”

### 2) SACS Audit
Goal: Check the existing SACS set against the current codebase, report drift, missing coverage, broken references, and architecture-view cycle violations.

---

## Output locations and file hierarchy

Create/maintain the following tree at repo root (prefer `sacs/` unless the repo already uses `docs/` conventions):

```

sacs/  
README.md # entrypoint + how to use + index  
manifest.md # module index table (lightweight, human+LLM friendly)  
root.md # system-level architecture views (DAG)  
modules/  
<module_slug>.md # per-module subgraph docs (DAG)  
flows/  
<flow_slug>.md # optional deep dives (may include non-DAG details if explicitly marked)  
audits/  
latest.md # most recent audit report

```

Rules:
- Keep filenames stable. Prefer short slugs: `auth.md`, `data_platform.md`, `ui.md`.
- Use relative links between SACS files.
- Do not add timestamps everywhere (minimize diff churn). If you must note recency, do it once in `audits/latest.md`.

---

## Global conventions

### Status tags (mandatory)
Every SACS doc file must declare a coverage status near the top:

- **Status: stub** — minimal skeleton, incomplete/unknown internals.
- **Status: partial** — some internals documented; gaps remain.
- **Status: complete** — sufficiently detailed for intended use.
- **Status: skipped** — intentionally omitted (vendor, generated, etc.), with reason.

### Evidence discipline (mandatory)
Do not invent architecture. Every important diagram element must be grounded.

- Each **diagram component** should have at least one **code reference** (file + key symbols; include line ranges when practical).
- Each **cross-module edge** in `root.md` and module dependency views should have at least one evidence reference (import/use/call/config).
- If something is uncertain, label it explicitly as **Hypothesis** and keep it out of the main DAG view if it would mislead.

### DAG rule (mandatory)
Main architecture views (root + module docs) must remain acyclic.

If you detect implementation cycles:
- Keep the *architecture view* acyclic by collapsing or abstracting the cycle using the conventions below.
- Record the cycle in an **Audit** section or a dedicated “Cycle Details” subsection.

---

## Diagram conventions (Mermaid)

### Default: flowcharts
- Use `flowchart TB` for vertical layered architecture (preferred).
- Use `flowchart LR` for dependency graphs (often clearer).
- Use `subgraph` to cluster by layer/module.
- Use edge labels for interfaces/data: `A -->|events| B`.

### Node naming
- Prefer short, stable names:
  - components/classes: `AudioEngine`, `VoiceManager`
  - modules/packages: `audio`, `input`, `display`
- Avoid embedding too much detail in node labels; put details in text + code references.

### Edge “kinds” (encode via labels)
Use labels to communicate semantics:
- `imports`, `calls`, `publishes`, `subscribes`, `reads`, `writes`, `renders`, `invokes`, `configures`

Example:
`Auth -->|calls (RPC)| UserService`

### Cycle handling conventions
For architecture DAG views:
1) Prefer **layering abstraction**: choose one direction for the conceptual dependency, and move the reverse dependency into a note (not an edge).
2) If the cycle is real and unavoidable, use a **Cycle Group node**:
   - Create a single node like `CycleGroup: A <-> B <-> C`
   - Edges in/out connect to the group node only
   - Put the internal cyclic details in a separate “Cycle Details” diagram (allowed to be cyclic) under a clearly labeled subsection.

Do not let the main overview DAG become cyclic.

---

## Core document templates

### `sacs/README.md` (entrypoint)
Must include:
- What SACS is for (1–3 sentences)
- How to navigate it
- Link to `root.md`
- Link to `manifest.md`

### `sacs/manifest.md` (index table)
A single table with one row per module:

| Module | Status | Owns (paths) | Key responsibilities | Doc |
|---|---:|---|---|---|

Notes:
- “Owns (paths)” can be 1–3 globs/directories.
- Keep responsibilities to 1 short line.

### `sacs/root.md` (system-level views)
Required sections:

1) `# System Architecture (SACS)`
2) `## Scope and Status` (Status tag + what’s covered/skipped)
3) `## High-Level Overview` (Mermaid flowchart TB; 10–60 nodes)
4) `## Module Dependency DAG` (Mermaid flowchart LR; modules only)
5) `## Key Interfaces` (bullets: APIs/events/DBs/queues; keep succinct)
6) `## Code References (Top-Level)` (table mapping major components to files/symbols)
7) `## TODO / Unknowns` (explicit gaps)

**Top-Level Code References table (required columns):**
| Component/Module | File(s) | Key Functions/Classes | Notes |
|---|---|---|---|

### `sacs/modules/<module>.md` (per-module subgraph)
Required sections:

1) `# Module: <Name>`
2) `## Status and Ownership` (Status tag + owned paths/globs)
3) `## Responsibilities` (3–7 bullets; avoid prose walls)
4) `## Public Interfaces` (APIs, entrypoints, commands, exports)
5) `## Internal Architecture` (Mermaid flowchart TB)
6) `## Dependencies` (Mermaid flowchart LR; this module ↔ other modules/external libs)
7) `## Key Flows` (optional, but include if module has a pipeline/state machine/threading)
8) `## Code References` (table, see below)
9) `## TODO / Unknowns`

**Per-module Code References table (required columns):**
| Component/Stage | File | Key Symbols (fn/class) | Notes |
|---|---|---|---|

Guidelines:
- Include line ranges when feasible (preferred).
- Use relative links if the environment supports them (e.g., GitHub): `src/foo.py#L10-L42`.
- If exact anchors are not reliable, use `path:line-line` text.

### `sacs/flows/<flow>.md` (optional deep dive)
Use for cross-cutting flows that touch many modules (e.g., “request lifecycle”, “training pipeline”, “threading model”).

Required sections:
- Overview diagram
- Evidence table
- Boundary conditions / failure modes (bullets)
- Links back to involved modules

---

## Create/Update procedure (manual, agent-executed)

### Step 0 — Decide budget and scope (default: partial)
If the user does not specify scope:
- Default to documenting **top-level** plus the **3–8 most central modules**.
- Create stubs for the rest (status: stub).

If repo is huge:
- Produce `root.md` + `manifest.md` + stubs only.
- Explicitly mark what was not explored.

### Step 1 — Inventory the codebase
- Identify:
  - entrypoints (main, CLI, server start, scheduled jobs)
  - major top-level directories/packages
  - core domain modules vs. utilities vs. vendor/generated
- Pick module boundaries primarily from directory/package structure, adjusted for responsibility (avoid tiny modules unless justified).

### Step 2 — Draft/Update `manifest.md`
For each module:
- set Status (stub/partial/complete/skipped)
- list owned paths (globs/directories)
- 1-line responsibility
- link to module doc

### Step 3 — Build/Update `root.md`
Produce two main DAG views:

1) **High-Level Overview (TB)**
- Layers/subgraphs (e.g., Input, Application Logic, Data, External Services, UI)
- Major components (not every file)
- Direction reflects conceptual control/data flow.

2) **Module Dependency DAG (LR)**
- Nodes: modules/packages/services
- Edges: “imports/calls/reads/writes/publishes”
- Keep this view acyclic; use cycle conventions if needed.

Add a top-level Code References table:
- For each top node, list:
  - entrypoint files
  - key orchestrators
  - key interface boundaries

### Step 4 — Write/Update module docs
For each non-skipped module:
- Create/update `modules/<module>.md` using the template.
- Include:
  - internal architecture flowchart
  - dependency diagram
  - at least one code reference per major component
- If you cannot confidently document internals, keep Status as stub and add TODOs.

### Step 5 — Add flow docs only when high leverage
Create `flows/<flow>.md` when:
- there is a critical pipeline/state machine/threading model
- it spans modules and the root view isn’t enough
Use the example pattern:
- flow diagram + stage references table

### Step 6 — Minimize churn on updates
When updating existing SACS:
- Preserve headings and ordering.
- Prefer additive edits and small diffs.
- Do not rewrite “Responsibilities” unless they are wrong.
- Update code references if symbols moved or responsibilities changed.

---

## Audit procedure (manual, agent-executed)

Write results to `sacs/audits/latest.md` with:
- summary (what’s good, what’s drifting)
- actionable findings
- suggested next updates

### Audit checks (required)
1) **Coverage drift**
- Are there new top-level directories/packages not in `manifest.md`?
- Are there modules marked “owns path” that no longer exists?

2) **Reference validity**
- Spot-check that referenced files exist.
- If line anchors are used, ensure they still roughly match (update if off).
- Ensure key symbols mentioned still exist or note renames.

3) **Dependency drift**
- Sample-check edges:
  - For each major edge in root dependency DAG, confirm evidence (import/use/call).
  - If evidence is unclear, downgrade confidence in notes and/or mark TODO.

4) **Cycle violations**
- Ensure root and module DAG diagrams remain acyclic.
- If implementation cycles exist, list them under “Cycle Findings” with evidence.

5) **Diagram quality**
- Flag diagrams with:
  - too many nodes (> ~60 in one view)
  - unreadable labels
  - ambiguous edges (no labels where needed)
Provide refactoring suggestions: split into subgraphs or move details into module/flow docs.

### Audit output format (required)
- `## Summary`
- `## Findings`
  - `### Coverage`
  - `### Broken References`
  - `### Dependency Drift`
  - `### Cycles`
  - `### Readability`
- `## Recommended Updates (ordered)`

---

## Style constraints

- Prefer bullet points over paragraphs.
- Prefer one clear diagram per section over giant “everything” diagrams.
- Keep diagrams grounded; avoid speculative edges.
- Use consistent naming across files (module names, component names).
- Always include code callouts tables adjacent to diagrams.

END SKILL
```

If you want, I can tighten this further into an even more “Claude Code friendly” format (e.g., fewer words, more imperative checklists, and explicit file templates you can paste directly), but the above is already operational as-is.
