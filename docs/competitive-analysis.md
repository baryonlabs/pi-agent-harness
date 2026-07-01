# Competitive Analysis — pi.dev "harness" Ecosystem

> **Purpose (internal).** Map the pi.dev package gallery's `harness` landscape, extract strengths worth adopting and weaknesses worth avoiding, and record how `pi-agent-harness` differentiates. Source: `https://pi.dev/packages?name=harness` (surveyed 2026-07-01).

## TL;DR

- The gallery has **40+ "harness" packages**. Most are **single-agent governance/workflow** tools for **software coding only**.
- Our closest direct competitors (multi-agent teams): **zob-harness**, **skynex-pi**, **ultimate-pi**, **gentle-pi**, **dev-loops**.
- **What we adopt:** complexity triage, a structured inter-agent handoff schema, run manifests (artifacts-as-truth), a verification gate + bounded repair loop, HITL checkpoints, no-auto-commit governance.
- **Where we stay different:** (a) **zero companion dependencies** — we bundle `subagent`; (b) **domain-agnostic** — novels, webtoons, research, not just code; (c) a **team-architecture factory** with 6 patterns, not a single-agent profile/guardrail tool.

## Landscape (by download volume)

| Package | Author | Category | Downloads/mo |
|---------|--------|----------|-------------|
| gentle-pi | Gentleman-Programming | senior-architect harness (SDD/OpenSpec, TDD) | 11.5K |
| zob-harness | cgarrot | **governed agent factory / communicating teams** | 5.3K |
| ultimate-pi | aryaniyaps | governed plan→execute→review pipeline | 2.8K |
| dev-loops | mfittko | GitHub issue→PR lifecycle (harness-agnostic) | 2.1K |
| arey-pi | alereyleyva | Gherkin specs + non-negotiable TDD | 1.3K |
| skynex-pi | joeldevz | **multi-agent: 17 skills + 11 sub-agents + 7 ext** | 817 |
| pi-harness-factory | kswift120 | single-agent profile manager | 204 |
| … (30+ more) | | eval, security, browser, tmux, memory, providers | tail |

Adjacent (not competitors, but part of the mental model): `@marcfargas/pi-test-harness` (extension testing), `@artale/pi-eval` (session evaluation), `pi-sub-agent` (raw delegation tool), `pi-browser-harness`, `pi-tmux-harness`.

## Deep dives — direct competitors

### zob-harness — "A governed Agent Factory for Pi" (closest competitor)

**Concept:** turns Pi "from a single coding-agent chat into a supervised factory system: define a team, start a run, watch agent communication, review artifacts, and package successful workflows into factories."

Strengths worth adopting:
1. **Visible, structured inter-agent communication.** Agents coordinate with parent-visible messages in a fixed schema: `CONTEXT / ASK / EVIDENCE / URGENCY / BLOCKER`. "Agents coordinate through parent-visible messages instead of hidden worker chat." → **Directly fixes our weakest area** (pi has no live messaging; we hand off via files). A structured handoff schema makes our `_workspace/` files first-class.
2. **Run artifacts as truth.** "manifests, kickoff files, workgraphs, status files, validation output, and oracle reports outlive the chat," stored under `.pi/reports/<run_id>/`.
3. **Governed by default.** Scoped tools, path rules, evidence gates, **no-ship review**, governed commits — "keep the owner in control."
4. **Factories create factories.** A successful run becomes a reusable manifest/launcher — mirrors our "harness evolution" idea.
5. **A runnable demo** (six-agent Pac-Man factory) for instant understanding.

Weaknesses / where we differ: heavy **tmux dependency** for persistent roles; **software-only**; steep concept count (ZAgents/ZTeams/workgraphs/oracles).

### skynex-pi — "Multi-agent coding harness — triage + 17 skills + 11 sub-agents"

Strengths worth adopting:
1. **Complexity triage.** Every prompt is auto-classified into `conversational / small / medium / substantial`, and the matching workflow is injected: `medium → discover→plan→build→validate`, `substantial → discover→propose→specify→plan→build→validate`. **Scale the process to the task.**
2. **HITL gate modes** via `SKYNEX_HITL=single|strict|none`.
3. Discipline skills: `tdd-discipline`, `verification-before-completion`, `adversarial-review`, `security`.

Weaknesses / where we differ: **requires 4 companion packages** (`pi-sub-agent`, `@juicesharp/rpiv-todo`, `pi-web-access`, `pi-mcp-adapter`) — "NOT bundled to avoid version conflicts." Heavy setup burden. **We bundle `subagent`; zero companion deps.** Also software-only.

### ultimate-pi — "Governed coding harness: bootstrap, plan, execute, review, steer"

Strengths worth adopting:
1. **PlanPacket gate.** "execute only against an approved PlanPacket, then run an independent review gate before merge." Plan → approve → execute → review.
2. **Repair loop.** `/harness-steer` reads `repair-brief.yaml` and "respawns the executor in repair mode **without widening the approved plan scope**." Bounded repair.
3. **Never auto-merges** — human owns the merge. Run context in `.pi/harness/active-run.json`, artifacts under `.pi/harness/runs/<run_id>/`.
4. **Idempotent bootstrap** (`/harness-setup`).

Weaknesses / where we differ: **very large command surface** (auto/plan/run/review/steer/abort/clear/trace/incident/…) — high cognitive load. We keep a small prompt surface.

### gentle-pi — highest downloads (11.5K/mo)

"Turn Pi into el Gentleman: a senior-architect development harness with SDD/OpenSpec, subagents, strict TDD evidence, review guardrails, and skill discovery." Lesson: **strong branding/persona + spec-driven development** drives adoption. Software-only.

### dev-loops — GitHub issue → merged PR

Seven deterministic lifecycle phases (intake → refinement → implementation → draft_gate → feedback_resolution → pre_approval_gate → merge) with self-correcting review gates and a single `dev-loop` entrypoint. Lesson: **deterministic routing + one entrypoint** beats exposing internal strategy names. Harness-agnostic (Pi + Claude Code).

## What we adopt (mapped to our files)

| # | Adopted strength | Source | Where it lands |
|---|------------------|--------|----------------|
| A | **Complexity triage** — classify small/standard/substantial, scale team + phases | skynex | `orchestrator-template.md` (Phase 0.5) + SKILL.md Phase 2 pointer |
| B | **Structured handoff schema** — CONTEXT/GOAL/INPUT/OUTPUT/EVIDENCE/NEXT in `_workspace/` files | zob | `agent-design-patterns.md` (handoff) + agent def template |
| C | **Run manifest + run IDs** — `_workspace/<run_id>/manifest.md` as source of truth | zob, ultimate | `orchestrator-template.md` (Phase 1) |
| D | **Verification gate + bounded repair** — evidence-checked review, ≤2 repair passes, no scope-widening | ultimate, skynex | `orchestrator-template.md` (producer-reviewer) |
| E | **No auto-commit / no-ship review** — human owns merge; harness never commits unprompted | zob, ultimate | SKILL.md principles + orchestrator error handling |
| F | **HITL checkpoints** — explicit human approval points, tunable | skynex | `orchestrator-template.md` |

## What we deliberately keep different (our moat)

1. **Zero companion dependencies.** skynex needs 4 extra packages; we bundle `subagent`. `pi install` and go.
2. **Domain-agnostic.** Every direct competitor is software-coding-only. We generate teams for **any** domain — research, novels, webtoons, marketing, code. This is our widest differentiator; lead with it.
3. **Team-architecture factory, not a guardrail tool.** vs `pi-harness-factory` (single-agent profiles) and vs governance-only harnesses: we *decompose a domain into a coordinated team* with 6 named patterns.
4. **Lean surface.** vs ultimate-pi's 20+ commands: one orchestration prompt per domain, generated to fit.

## Backlog (not yet adopted)

- **Runnable demo** (à la zob Pac-Man) — a `examples/` domain harness users can run in 2 minutes.
- **Spec-driven option** (SDD/OpenSpec, à la gentle-pi) — optional spec artifact before team design.
- **Session evaluation** hook (à la `@artale/pi-eval`) — score a harness run's success/efficiency.
- **Branding/persona** pass — a memorable identity (gentle-pi's "el Gentleman" lesson).
