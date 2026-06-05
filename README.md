# dockit meta-prompting

`dockit` is a prompt toolkit for creating and reviewing design system documentation with AI agents.

This repository is not a web app or SDK. It is a source-of-truth prompt package that defines:

- command entry points (for example: start, import, progress)
- phased documentation workflows
- shared references and templates
- reviewer and explorer agents
- evaluation skills (readability, copy-editing, completeness, voice)
- build tooling to publish agent-targeted prompt bundles

## What this project does

At a high level, this project encodes a complete documentation workflow for design systems:

1. Discover what is being documented (type, tier, audience, use cases)
2. Define structure (which sections the doc should contain)
3. Draft section content
4. Revise for style and clarity
5. Review across multiple quality dimensions
6. Polish and resolve findings

It also supports importing existing documentation into this workflow, so teams can migrate legacy docs instead of starting from scratch.

## Scope

### In scope

- Prompt definitions for `dockit` commands in `src/commands/dockit/`
- Workflow logic in `src/dockit/workflows/`
- Reference standards (phases, sections, style guide, tiers, taxonomy) in `src/dockit/references/`
- Planning/document templates in `src/dockit/templates/`
- Review and utility agents in `src/agents/`
- Skill definitions and deterministic evaluation scripts in `src/skills/`
- Build pipeline that produces agent-specific distributions in `dist/{agent-folder}/`

### Out of scope

- Hosting or rendering a documentation website
- Shipping UI components or design tokens as code artifacts
- Runtime docs serving, search backend, or CMS features
- General-purpose LLM framework functionality outside this doc workflow

## Intended ways of using this repository

### 1) As a source package for AI agent environments

Use this repo as the canonical prompt source, then build target-specific bundles.

The build script compiles TypeScript skills and copies markdown/json/txt assets into four output targets:

- `dist/.claude/`
- `dist/.pi/`
- `dist/.agents/`
- `dist/.opencode/`

During build, `{AGENT_FOLDER}` placeholders are replaced per target so prompt references resolve correctly.

### 2) As a maintained workflow specification

Teams can evolve documentation behavior by editing the source prompts and references:

- command behavior: `src/commands/dockit/*.md`
- workflow transitions: `src/dockit/workflows/*.md`
- writing standards: `src/dockit/references/style-guide.md`
- section taxonomy and discovery framing: `src/dockit/references/*.md`
- planning/report templates: `src/dockit/templates/*.md`

### 3) As a quality-review toolkit

The repository includes deterministic + qualitative review skills:

- readability (`src/skills/readability-evaluation/`)
- copy editing (`src/skills/copy-editing-evaluation/`)
- completeness (`src/skills/completeness-evaluation/`)
- brand voice (`src/skills/brand-voice-evaluation/`)

These are used by reviewer agents during the review phase to create consolidated findings.

### 4) As a structured design-system knowledge explorer

`src/skills/explore-existing-ds-docs/` contains structured Vanilla Framework documentation data (index + per-item JSON files) and guidance for citation-first exploration.

## Command surface

Current command prompt files under `src/commands/dockit/`:

- `/dockit:start`
- `/dockit:import`
- `/dockit:add-section`
- `/dockit:progress`
- `/dockit:help`

The phase model also defines a dedicated review workflow in `src/dockit/workflows/review.md`.

## Project structure (high level)

```text
src/
  agents/                 # reviewer/explorer agent prompts
  commands/dockit/        # command entry prompts
  dockit/
    references/           # phases, sections, style guide, taxonomy
    templates/            # state/checklist/document/review templates
    workflows/            # procedural flow definitions
  skills/                 # evaluation and exploration skills + scripts/data
  build.ts                # multi-target build script
tests/                    # vitest coverage for deterministic evaluators
```

## Development

### Prerequisites

- Node.js 20+
- pnpm (repo is pinned via `packageManager`)

### Install

```bash
pnpm install
```

### Build

```bash
pnpm build
```

This runs `tsx src/build.ts` and generates `dist/.claude`, `dist/.pi`, `dist/.agents`, and `dist/.opencode`.

### Test

```bash
pnpm test
```

Runs `vitest` tests for deterministic evaluation scripts.

## Workflow artifacts created during usage

When users run the `dockit` workflow in a documentation workspace, the process manages:

- `.planning/STATE.md`
- `.planning/docs/{doc-name}/structure.md`
- `.planning/docs/{doc-name}/checklist.md`
- `.planning/docs/{doc-name}/import-report.md` (for imports)
- `.planning/reviews/{doc-name}-review.md` (after review)
- `docs/{doc-name}.md`

These are the operating artifacts the commands read and update over time.

## Example outcome (what you end up with)

After a full run (discovery -> structure -> drafting -> revision -> review -> polish), the workspace typically looks like this:

```text
.planning/
  STATE.md
  docs/
    button/
      structure.md
      checklist.md
      import-report.md      # only when using /dockit:import
  reviews/
    button-review.md
docs/
  button.md
```

### Final document state

- `docs/button.md` contains the user-facing documentation in final prose.
- `.planning/docs/button/checklist.md` has every planned section at `revised` or `complete`.
- `.planning/reviews/button-review.md` tracks all review findings and whether they were fixed, skipped, or sent back.
- `.planning/STATE.md` shows the document phase as `Complete`.

### Typical progression snapshot

During in-progress work, section states in the checklist move like this:

- `pending` -> `draft` -> `revised`

When findings require deeper changes during polish, a section can move backward:

- `revised` -> `draft` (send back to revision)
- `revised` -> `pending` (send back to drafting)

The document is only truly done when all sections are back to `revised` and review findings are resolved.

## Notes for contributors

- Keep workflow, references, and templates aligned: behavior drift creates brittle command execution.
- Prefer updating source prompts under `src/`; generated output lives in `dist/`.
- If you add new skill logic in TypeScript, add tests in `tests/`.
