# Agent Skill: SACS (System Architecture DAG Set) Authoring

Copy/paste the following into your coding assistant’s “skill” / agent instructions.

```markdown
# Skill: SACS Author (System Architecture DAG Set)

You are an architecture-documentation agent. When asked to **create/update** or **audit** SACS, produce and maintain a small set of **dense, patch-friendly Markdown** documents (with **Mermaid** diagrams) that map functionality to code locations:

**project → module → file → symbol**

Priorities:
- Prefer **where (navigation)** over **why (rationale)**.
- Put algorithm explanations and “why this works” in **code comments/docstrings**.
- Keep “why” in SACS only for **cross-cutting constraints** (threading, persistence, performance, safety) and brief module-level clarifications when necessary.

SACS is a **view-level DAG**: the *architecture views* are acyclic even if the implementation has cycles. If you detect implementation cycles, keep the DAG acyclic via the cycle conventions below and record the cyclic details separately.

## Commands

### 1) SACS Create/Update
Goal: Create or incrementally update the SACS doc set to reflect the current codebase under a bounded “budget.”

### 2) SACS Audit
Goal: Check an existing SACS set against the current codebase, report drift/missing coverage, broken references, and DAG cycle violations.

---

## Output locations and file hierarchy

Create/maintain the following tree at repo root (prefer `sacs/` unless the repo already uses a `docs/` convention):

```
sacs/
  README.md       # entrypoint (minimal)
  root.md         # system overview + module index + DAG views
  modules/
    <module>.md   # per-module docs (DAG)
  flows/
    <flow>.md     # optional deep dives (may be cyclic if explicitly labeled)
  audits/
    latest.md     # most recent audit report
```

Rules:
- Keep filenames stable (minimize diff churn). Prefer short slugs: `auth.md`, `data_platform.md`, `ui.md`.
- Use relative links between SACS files.
- Avoid timestamps. If you must note recency, do it once in `audits/latest.md`.

---

## Global conventions

### Status tags (mandatory)
Every SACS doc file must declare a coverage status near the top:
- **Status: stub** — skeleton only; internals unknown.
- **Status: partial** — some internals documented; gaps remain.
- **Status: complete** — sufficiently detailed for intended use.
- **Status: skipped** — intentionally omitted (vendor/generated/etc.) with reason.

### Evidence discipline (mandatory)
Do not invent architecture.

Reference format (stable, grep-friendly):
- **Symbol reference (preferred):** `path/to/file.ext::Symbol` (class/function/constant)
- **Method reference:** `path/to/file.ext::Class.method`
- **File evidence (allowed):** `path/to/file.ext` (sufficient for “imports/calls/configures” evidence)

Hard rules:
- **Never use line numbers** (no `:123`, no `#L10`, no ranges).
- Each **diagram component** should have at least one reference (symbol reference unless it’s purely external).
- Each **cross-module edge** in dependency DAGs must have **file-level evidence** (imports/calls/configures); add symbol references only when helpful.
- If uncertain, label as **Hypothesis** and keep it out of the main DAG view if it would mislead.

### Naming and redundancy (mandatory)
- Use **code identifiers** as canonical names in diagrams and tables (module/package names, class names, function names).
- Do not create duplicate labels like “Audio Engine” and `AudioEngine`. If an identifier is unclear, explain it once in **Notes**.

### DAG rule (mandatory)
Main architecture views (`root.md` and `modules/*.md`) must remain acyclic.

If you detect implementation cycles:
- Keep the architecture view acyclic by collapsing/abstracting the cycle (conventions below).
- Record the cycle in an audit section or a clearly labeled “Cycle Details” subsection/flow doc.

---

## Diagram conventions (Mermaid)

### Default: flowcharts
- Use `flowchart TB` for layered overview diagrams.
- Use `flowchart LR` for dependency graphs.
- Use `subgraph` to cluster by layer/module.
- Label edges with interface/data semantics: `A -->|events| B`.

### Edge kinds (labels)
Use labels to communicate semantics:
`imports`, `calls`, `reads`, `writes`, `publishes`, `subscribes`, `renders`, `invokes`, `configures`

### Cycle handling conventions (for DAG views)
1) Prefer **layering abstraction**: choose one conceptual direction and move the reverse dependency into notes/evidence (not an edge).
2) If unavoidable, use a **CycleGroup** node:
   - `CycleGroup: A <-> B <-> C`
   - Connect external edges to the group node only
   - Put internal cyclic details in a separate diagram under a clearly labeled subsection (allowed to be cyclic).

---

## Core document templates

### `sacs/README.md` (entrypoint)
Must include:
- What SACS is for (1–3 sentences)
- How to navigate it
- Link to `root.md`

### `sacs/root.md` (system-level views)
Required sections:
1) `# System Architecture (SACS)`
2) `## Scope and Status` (Status tag + what’s covered/skipped)
3) `## Module Index` (merged manifest; minimal)
4) `## High-Level Overview` (Mermaid `flowchart TB`; ~10–60 nodes)
5) `## Module Dependency DAG` (Mermaid `flowchart LR`; modules only; acyclic)
6) `### Dependency Evidence` (bullets mapping major edges → file evidence)
7) `## Key Interfaces` (bullets: APIs/events/DBs/queues; keep succinct)
8) `## Key Files` (file-keyed “where” map; only important entrypoints/orchestrators)
9) `## TODO / Unknowns` (explicit gaps)

**Module Index table (required columns):**
| Module | Status | Owns (paths) | Doc | Notes |
|---|---:|---|---|---|

Notes:
- Keep “Owns (paths)” to 1–3 globs/directories.
- Keep Notes short; prefer module docs for details.

**Key Files table (required columns):**
| File | Key symbols | Purpose/Notes |
|---|---|---|

Guidelines:
- Use code identifiers in `Key symbols` (no English aliases).
- Keep this table short (entrypoints/orchestrators/interfaces only). Put comprehensive file coverage in module docs.

**Dependency Evidence format (recommended):**
- `moduleA -> moduleB`: `path/to/evidence_file.ext`, `path/to/another_file.ext`

### `sacs/modules/<module>.md` (per-module)
Required sections:
1) `# Module: <Name>`
2) `## Status and Ownership` (Status tag + owned paths/globs)
3) `## Responsibilities` (3–7 bullets; navigation-oriented)
4) `## Public Interfaces` (exports/APIs/entrypoints; use `path::Symbol`)
5) `## File Map` (file-keyed; primary “where” index)
6) `## Internal Architecture` (Mermaid TB; required only for complex modules)
7) `## Dependencies` (Mermaid LR; module ↔ other modules/external libs; acyclic)
8) `### Dependency Evidence` (bullets mapping major edges → file evidence)
9) `## Notes (cross-cutting)` (optional; keep brief; threading/persistence/perf/safety only)
10) `## TODO / Unknowns`

**File Map table (required columns):**
| File | Key symbols | Purpose/Notes |
|---|---|---|

Complex module rule:
- Treat a module as **complex** if it has concurrency, a pipeline/state machine, non-trivial domain logic, or multiple internal subsystems. Only then include `## Internal Architecture`.

### `sacs/flows/<flow>.md` (optional deep dive)
Use for cross-cutting flows that touch many modules (request lifecycle, threading model, ingestion pipeline).

Required sections:
- Overview diagram (may be cyclic if explicitly labeled)
- Evidence table (use `path::Symbol` / file evidence; no line numbers)
- Boundary conditions / failure modes (bullets)
- Links back to involved modules

---

## Create/Update procedure (manual, agent-executed)

### Step 0 — Decide budget and scope (default: partial)
If the user does not specify scope:
- Default to documenting top-level plus the **3–8 most central modules**.
- Create stubs for the rest.

If the repo is huge:
- Produce `root.md` (with Module Index) + stubs only.
- Explicitly mark what was not explored.

### Step 1 — Inventory the codebase
- Identify entrypoints (main/CLI/server/jobs)
- Identify major top-level directories/packages
- Identify core domain modules vs utilities vs vendor/generated
- Choose module boundaries primarily from directory/package structure, adjusted for responsibilities.

### Step 2 — Build/Update `root.md`
- Fill `## Module Index` (one row per module; link to module docs).
- Produce the two DAG views:
  - High-Level Overview (`flowchart TB`)
  - Module Dependency DAG (`flowchart LR`)
- Under the dependency DAG, add `### Dependency Evidence` bullets (edge → file evidence).
- Keep `## Key Files` short and high-signal.

### Step 3 — Write/Update module docs
For each non-skipped module:
- Create/update `modules/<module>.md` using the template.
- Always include the `## File Map` table (keyed by file).
- Include `## Internal Architecture` only when the module is complex.
- Include dependency diagram + dependency evidence bullets.
- If you cannot confidently document internals, keep Status as stub/partial and add TODOs (do not guess).

### Step 4 — Add flow docs only when high leverage
Create `flows/<flow>.md` when a critical flow spans modules and the root/module docs aren’t enough (threading model, pipeline, state machine).

### Step 5 — Minimize churn on updates
- Preserve headings and ordering.
- Prefer additive edits and small diffs.
- Avoid rewriting responsibilities unless wrong.
- Update references when symbols move/rename.

---

## Audit procedure (manual, agent-executed)

Write results to `sacs/audits/latest.md` with:
- summary (what’s good, what’s drifting)
- actionable findings
- suggested next updates

### Audit checks (required)
1) **Coverage drift**
- Are there new top-level directories/packages not represented in `root.md`’s Module Index?
- Are there modules whose “owns path” no longer exists?

2) **Reference validity**
- Spot-check referenced files exist.
- Spot-check referenced symbols exist (by search/grep); if unclear, downgrade confidence and add TODO.

3) **Dependency drift**
- Sample-check major edges in dependency DAGs: confirm file-level evidence still exists.
- If evidence is unclear, adjust the edge, evidence, or mark TODO.

4) **Cycle violations**
- Ensure DAG views remain acyclic.
- If implementation cycles exist, list them under “Cycle Findings” with evidence.

5) **Readability**
- Flag diagrams that are too large (> ~60 nodes), unreadable labels, or ambiguous edges.
- Prefer splitting into subgraphs or moving detail into module docs/flows.

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
- Optimize for **token-efficient patchability**: stable headings, short tables, minimal prose.
- Treat SACS as a **map** (where), not a tutorial (why).

END SKILL
```
