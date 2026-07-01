<p align="center">
  <img src="harness_banner.png" alt="Harness Banner" width="600">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Version-2.1.0-brightgreen.svg" alt="Version">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" alt="License"></a>
  <img src="https://img.shields.io/badge/pi-Package-purple.svg" alt="pi Package">
  <img src="https://img.shields.io/badge/Patterns-6_Architectures-orange.svg" alt="6 Architecture Patterns">
  <img src="https://img.shields.io/badge/Modes-single%20%7C%20parallel%20%7C%20chain-green.svg" alt="Delegation Modes">
  <a href="https://github.com/revfactory/harness/stargazers"><img src="https://img.shields.io/github/stars/revfactory/harness?style=social" alt="GitHub Stars"></a>
</p>

<p align="center">
  <a href="#category--where-harness-sits"><img src="https://img.shields.io/badge/Layer-L3%20Meta--Factory-orange" alt="Layer"></a>
  <a href="#category--where-harness-sits"><img src="https://img.shields.io/badge/Sub--layer-Team--Architecture%20Factory-teal" alt="Sub-layer"></a>
  <a href="#"><img src="https://img.shields.io/badge/README-EN%20%7C%20KO%20%7C%20JA-lightgrey" alt="i18n"></a>
</p>

# pi-agent-harness — The Team-Architecture Factory for pi

**English** | [한국어](README_KO.md) | [日本語](README_JA.md)

> **pi-agent-harness is a team-architecture factory for [pi](https://github.com/earendil-works/pi), the minimal terminal coding agent.** Say **"build a harness for this project"** (English) or **"하네스 구성해줘"** (한국어) or **"ハーネスを構成して"** (日本語), and it turns your domain description into pi agent definitions, the skills they use, and the orchestration prompts that drive them — picked from six pre-defined team-architecture patterns.

> **Ported from Claude Code.** This package is a pi-only port of the original [Harness](https://github.com/revfactory/harness) (a Claude Code plugin). It maps the same factory onto pi's primitives — `.pi/agents/`, `.pi/skills/`, `.pi/prompts/`, and the bundled `subagent` extension — because pi deliberately ships **no built-in subagents, no MCP, no plan mode, no todos**.

## Overview

pi is intentionally minimal: it gives the model four tools (`read`, `write`, `edit`, `bash`) and everything else is added via skills, prompt templates, extensions, and packages. This harness leverages that extensibility to decompose complex tasks into coordinated specialist agents. Say "build a harness for this project" and it generates:

- **Agent definitions** (`.pi/agents/*.md`) — specialist personas with `tools`/`model` frontmatter; each runs in an isolated `pi` process
- **Skills** (`.pi/skills/*/SKILL.md`) — Agent Skills standard procedural knowledge
- **Orchestration prompts** (`.pi/prompts/*.md`) — `/commands` that drive the `subagent` tool in `single` / `parallel` / `chain` modes

The bundled **`subagent` extension** (`extensions/subagent/`) supplies the delegation tool pi lacks natively, so generated harnesses run out of the box.

## Category — Where Harness Sits

Harness lives at the **L3 Meta-Factory** layer — the layer that generates other harnesses rather than being one. Inside L3, we pick a specific sub-layer: **Team-Architecture Factory**. This package targets the **pi runtime**.

| Layer | What it does | Neighbors |
|-------|--------------|-----------|
| **L3 — Meta-Factory / Team-Architecture Factory** (us) | Domain sentence → pi agent team + skills + orchestration prompts, via 6 team patterns | — |
| L3 — Meta-Factory / Runtime-Configuration Factory | Deterministic, repeatable runtime configurations | [coleam00/Archon](https://github.com/coleam00/Archon) |
| L3 — Meta-Factory / Claude Code original | Same concept, Claude Code runtime | [revfactory/harness](https://github.com/revfactory/harness) |
| L2 — Cross-Harness Workflow | Standardize skills/rules/hooks across harnesses | [affaan-m/ECC](https://github.com/affaan-m/everything-claude-code) |

## Key Features

- **Agent Design** — 6 architectural patterns: Pipeline, Fan-out/Fan-in, Expert Pool, Producer-Reviewer, Supervisor, and Hierarchical Delegation — mapped to pi's `single`/`parallel`/`chain` delegation modes
- **Skill Generation** — Auto-generates Agent-Skills-standard skills with Progressive Disclosure for efficient context management
- **Orchestration** — `subagent`-tool prompts with `{previous}` chaining, `_workspace/` file handoff, and error handling
- **Model Tiering** — Per-agent model selection (`claude-haiku-4-5` for recon, `claude-sonnet-4-5` for work, `claude-opus-4-7` for hard reasoning)
- **Validation** — Trigger verification, dry-run testing, and with-skill vs without-skill comparison via the `subagent` tool

## Why this vs other pi harnesses

The pi.dev gallery has 40+ "harness" packages. Most are **single-agent guardrail/workflow tools for software coding only**. pi-agent-harness is positioned differently (full landscape in [`docs/competitive-analysis.md`](docs/competitive-analysis.md)):

- **Zero companion dependencies.** Multi-agent harnesses like `skynex-pi` require 4 extra packages to install alongside. We **bundle** the `subagent` delegation tool — `pi install` and go.
- **Domain-agnostic, not code-only.** Every direct competitor targets software engineering. This factory builds teams for **any** domain — research, novels, webtoons, marketing, *and* code.
- **A team-architecture factory, not a profile manager.** vs `pi-harness-factory` (single-agent modes) and governance-only harnesses: we *decompose a domain into a coordinated team* with 6 named patterns.
- **Lean surface.** vs `ultimate-pi`'s 20+ commands: one orchestration prompt per domain, generated to fit.

We also **adopt** what works across the ecosystem: complexity triage (scale process to task size), a structured inter-agent handoff schema, run manifests (artifacts-as-truth), a verification gate + bounded repair loop, HITL checkpoints, and no-auto-commit governance.

## Workflow

```
Phase 1: Domain Analysis
    ↓
Phase 2: Architecture Design (single / parallel / chain delegation)
    ↓
Phase 3: Agent Definition Generation (.pi/agents/)
    ↓
Phase 4: Skill Generation (.pi/skills/)
    ↓
Phase 5: Orchestration Prompts (.pi/prompts/) + AGENTS.md pointer
    ↓
Phase 6: Validation & Testing
    ↓
Phase 7: Harness Evolution (feedback-driven)
```

## Installation

Install as a pi package (the bundled `subagent` extension and the `harness` skill load automatically):

```bash
# From npm (pi.dev gallery)
pi install npm:@baryonlabs/pi-agent-harness

# Or from git
pi install git:github.com/baryonlabs/pi-agent-harness

# Or a local checkout
pi install /absolute/path/to/pi-agent-harness

# Project-local install (shareable with your team via .pi/settings.json)
pi install -l npm:@baryonlabs/pi-agent-harness
```

Try it without installing:

```bash
pi -e /absolute/path/to/pi-agent-harness
```

Then, inside pi, trigger it:

```
하네스 구성해줘
Build a harness for this project
```

## Package Structure

```
pi-agent-harness/
├── package.json                    # pi manifest (extensions + skills)
├── extensions/
│   └── subagent/                   # Bundled delegation tool (single/parallel/chain)
│       ├── index.ts
│       ├── agents.ts
│       └── README.md
├── skills/
│   └── harness/
│       ├── SKILL.md                # Main skill (7-Phase workflow)
│       └── references/
│           ├── agent-design-patterns.md   # 6 patterns + Claude→pi mapping
│           ├── orchestrator-template.md   # Orchestration prompt templates
│           ├── team-examples.md           # 5 real-world configurations
│           ├── skill-writing-guide.md     # Skill authoring guide
│           ├── skill-testing-guide.md     # Testing & evaluation methodology
│           └── qa-agent-guide.md          # QA agent integration guide
└── README.md
```

## Usage

### Delegation Modes

pi has no built-in agent teams or live messaging. Collaboration is implemented by the main agent delegating via the bundled `subagent` tool and exchanging results through `_workspace/` files. The orchestration prompt must pass `agentScope: "both"` so project-level `.pi/agents/` are discovered.

| Mode | Tool params | Use for |
|------|-------------|---------|
| **single** | `{ agent, task }` | One-off isolated task, expert-pool selection |
| **parallel** | `{ tasks: [...] }` (≤8, 4 concurrent) | Fan-out/fan-in independent work; main integrates |
| **chain** | `{ chain: [...] }` with `{previous}` | Sequential pipeline, producer-reviewer loops |

<p align="center">
  <img src="harness_team.png" alt="Harness Agent Team" width="500">
</p>

### Architecture Patterns → pi modes

| Pattern | pi mapping |
|---------|------------|
| Pipeline | `chain` |
| Fan-out/Fan-in | `parallel` → main integrates |
| Expert Pool | `single` (selective) |
| Producer-Reviewer | `chain` (worker → reviewer → worker) |
| Supervisor | main agent's dynamic `parallel` loop |
| Hierarchical Delegation | 2-level delegation |

## Output

Files generated by Harness in your project:

```
your-project/
├── .pi/
│   ├── agents/          # Agent definitions (name, description, tools, model)
│   │   ├── analyst.md
│   │   ├── builder.md
│   │   └── qa-inspector.md
│   ├── skills/          # Skill files
│   │   ├── analyze/
│   │   │   └── SKILL.md
│   │   └── build/
│   │       ├── SKILL.md
│   │       └── references/
│   └── prompts/         # Orchestration prompts (/commands)
│       └── build-feature.md
└── AGENTS.md            # Harness pointer (trigger rule + change log)
```

## Examples

A runnable, generated harness lives in [`examples/`](examples/):

- **[`repo-audit/`](examples/repo-audit/)** — audits any codebase from architecture, security, and quality angles (parallel fan-out), then integrates and adversarially verifies into one prioritized `AUDIT.md`. Runs with **zero companion dependencies** — just pi + this package. Read it to see exactly what the factory produces (`.pi/agents/`, `.pi/prompts/`, structured handoff, model tiering, run manifest, no-auto-commit).

```bash
cd examples/repo-audit && pi
# then: /repo-audit .
```

## Use Cases — Try These Prompts

After installing, trigger any of these inside pi:

**Deep Research** — `Build a harness for deep research: parallel agents investigating a topic from official, media, and community angles, then the main agent cross-validates and writes a report.`

**Website Development** — `Build a harness for full-stack website development as a chain: design → frontend (React/Next.js) → backend API → QA testing.`

**Code Review** — `Build a harness for comprehensive code review: parallel agents checking architecture, security, and performance, then merged into a single report.`

**Webtoon Production** — `Build a harness for webtoon episodes: a producer-reviewer chain where an artist agent generates panels and a reviewer agent enforces style consistency.`

**Marketing Campaign** — `Build a harness for marketing campaigns: research the market, write ad copy, design visual concepts, and plan A/B tests with iterative review.`

## Requirements

- [pi coding agent](https://github.com/earendil-works/pi) installed (`npm install -g --ignore-scripts @earendil-works/pi-coding-agent`)
- A model provider authenticated in pi (`/login` or an API key such as `ANTHROPIC_API_KEY`)
- The bundled `subagent` extension (installed automatically with this package) — no extra setup

> **Security note:** the `subagent` tool runs project-local agents (`.pi/agents/*.md`) only when invoked with `agentScope: "both"`/`"project"`, and prompts for confirmation in interactive mode. Only enable for repositories you trust. See `extensions/subagent/README.md`.

## Built with Harness (Claude Code original)

The original Claude Code harness was validated in a controlled A/B study and shipped a large catalog. Those artifacts target Claude Code; the methodology carries over to this pi port.

- **[revfactory/harness-100](https://github.com/revfactory/harness-100)** — 100 production-ready agent team harnesses across 10 domains (EN + KO).
- **[revfactory/claude-code-harness](https://github.com/revfactory/claude-code-harness)** — A/B experiment (n=15): +60% avg quality (49.5 → 79.3), 15/15 win-rate, −32% variance (author-measured, third-party replications pending).

> Full paper: *Hwang, M. (2026). Harness: Structured Pre-Configuration for Enhancing LLM Code Agent Output Quality.*

## FAQ

<details>
<summary><b>Q1. How is this different from the original Harness?</b></summary>

**A.** Same factory, different runtime. The original [revfactory/harness](https://github.com/revfactory/harness) is a Claude Code plugin that generates `.claude/agents/` + `.claude/skills/` and relies on Claude Code's native agent teams (`TeamCreate`/`SendMessage`/`TaskCreate`). This package is a **pi-only port**: it generates `.pi/agents/` + `.pi/skills/` + `.pi/prompts/` and implements collaboration through the bundled `subagent` extension (`single`/`parallel`/`chain`) plus `_workspace/` file handoff, because pi has no built-in subagents or live team messaging.
</details>

<details>
<summary><b>Q2. pi has no subagents — how do agent teams work?</b></summary>

**A.** This package bundles a `subagent` extension that spawns a separate `pi` process per delegation, each with an isolated context window. The main agent acts as coordinator: it fans out with `parallel`, chains dependent steps with `chain` (passing `{previous}`), and integrates results itself (pi has no team-consensus channel). See `skills/harness/references/agent-design-patterns.md` for the full Claude→pi mapping.
</details>

<details>
<summary><b>Q3. Can I reuse my existing Claude Code skills?</b></summary>

**A.** Yes. pi can load skills from other harnesses by adding their directories to `settings.json` (e.g. `"skills": ["~/.claude/skills"]`). Agent Skills standard SKILL.md files are largely portable between Claude Code and pi.
</details>

<details>
<summary><b>Q4. How is this different from <code>pi-harness-factory</code>?</b></summary>

**A.** They solve different problems and are complementary.

- **[`pi-harness-factory`](https://github.com/kswift1/pi-harness-factory)** is a **single-agent profile manager**: it defines "working modes" (presets like `tdd`, `safe-coder`, `code-reviewer`, `researcher`) that constrain *one* agent's tool access, mutation scope, validation strictness, and safety gates. It answers **"what is this one agent allowed to do, and how strict should it be?"** — a *runtime-configuration* factory (guardrails).
- **`pi-agent-harness`** (this project) is a **multi-agent team factory**: from one domain sentence it generates a *team* of specialist agents (`.pi/agents/`), their skills (`.pi/skills/`), and orchestration prompts (`.pi/prompts/`) that fan out and chain work via the bundled `subagent` tool. It answers **"how do I decompose this domain into a coordinated team of agents?"** — a *team-architecture* factory.

| | `pi-harness-factory` | `pi-agent-harness` (this) |
|---|---|---|
| Unit of work | One agent, many modes | Many agents, one team |
| Produces | JSON profiles (`.pi/harness-factory/`) | Agents + skills + orchestration prompts (`.pi/`) |
| Focus | Guardrails / constraints | Decomposition / orchestration |
| Layer | Runtime-configuration factory | Team-architecture factory |

You can use both: constrain each generated agent with `pi-harness-factory` modes while `pi-agent-harness` designs and orchestrates the team.
</details>

## License

Apache 2.0
