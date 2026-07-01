# Runtime Dependency (pi)

> **Status:** Active · **Owner:** baryonlabs · **Last updated:** 2026-07-01 · **SLA:** See [Monitoring Commitment](#monitoring-commitment)

This document explains what `pi-agent-harness` depends on at runtime, why there is **no experimental flag** (unlike the Claude Code original), and how this repository adapts when its upstream — the [pi coding agent](https://github.com/earendil-works/pi) — changes.

> **Ported from Claude Code.** The original [Harness](https://github.com/revfactory/harness) depended on Claude Code's experimental Agent Teams API (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, `TeamCreate`/`SendMessage`/`TaskCreate`). This pi port removes that dependency entirely: pi has no built-in subagents, so multi-agent delegation is supplied by a **bundled `subagent` extension**. Nothing to enable, no preview flags.

---

## Current State

### What the harness depends on

| Dependency | Purpose | Gated? |
|-----------|---------|--------|
| **pi coding agent** | Host runtime; loads skills, prompts, extensions, context files | No — stable install |
| **Bundled `subagent` extension** (`extensions/subagent/`) | Registers the `subagent` tool for `single`/`parallel`/`chain` delegation | No — ships in this package |
| **pi extension API** (`@earendil-works/pi-coding-agent`, `pi-agent-core`, `pi-ai`, `pi-tui`, `typebox`) | The `subagent` extension imports these (declared as `peerDependencies`) | No — provided by pi at runtime |
| **Agent Skills standard** | `.pi/skills/*/SKILL.md` format | No — versioned spec ([agentskills.io](https://agentskills.io/specification)) |

There is **no environment variable to set**. After `pi install`, the `subagent` tool and `harness` skill load automatically.

### The one caveat: pi's extension API is young

pi is a fast-moving, minimal harness. The `subagent` extension imports internal symbols from `@earendil-works/pi-coding-agent` (e.g. `getAgentDir`, `parseFrontmatter`, `CONFIG_DIR_NAME`, `ExtensionAPI`). If pi renames or changes these, the extension can break even though generated `.pi/agents`/`.pi/skills` (plain Markdown) remain valid.

---

## Dependency Graph

```
pi-agent-harness (v2.0.0)
  ├── harness skill (.pi/skills discovery)          ← Agent Skills standard
  ├── generated .pi/agents · .pi/skills · .pi/prompts ← plain Markdown, runtime-independent
  └── subagent extension (extensions/subagent)
        └── pi extension API (peerDependencies)
              └── pi release cadence
                    ├── Scenario A: pi adds first-class subagents
                    ├── Scenario B: pi extension API stabilizes / is versioned
                    └── Scenario C: breaking API change (symbol rename/signature)
```

**Read top-down:** the durable output (agents/skills/prompts Markdown) does not depend on pi internals. Only the `subagent` extension does. If pi's extension API changes, this repository adapts within the SLA below.

---

## Scenarios

### Scenario A — pi ships first-class subagents

**Trigger detection:** pi Changelog announces a built-in subagent/delegation tool, or an official pi package supersedes our bundled extension.

**Probability (subjective):** Medium. pi's philosophy is "we skip subagents; build or install your own," so a first-party version is plausible but not guaranteed.

| Checkpoint | Action | Artifact |
|------------|--------|----------|
| **T+48h** | Open `feat/native-subagent` branch. Evaluate mapping our `single`/`parallel`/`chain` contract onto the native tool; keep the bundled extension as fallback. | Branch + PR (draft) |
| **T+72h** | Update SKILL.md + references if the native tool changes the orchestration-prompt shape. Ship a minor release with a CHANGELOG note. | Minor release |

**Adopter impact:** Positive — one less bundled component to maintain. Generated harnesses keep working.

### Scenario B — pi extension API is versioned/stabilized

**Trigger detection:** pi publishes a stable, documented extension API surface (semver'd exports).

| Checkpoint | Action | Artifact |
|------------|--------|----------|
| **T+72h** | Pin `peerDependencies` to the stable range; document the minimum pi version in README Requirements. | Version-pinned release |

**Adopter impact:** Positive — fewer surprise breakages.

### Scenario C — breaking change (symbol rename / signature change)

**Trigger detection:** the `subagent` extension fails to load against pi's latest release, or an import throws.

**Probability (subjective):** Medium. Minimal fast-moving tools rename internals without deprecation windows.

| Checkpoint | Action | Artifact |
|------------|--------|----------|
| **T+0–24h** | Open `hotfix/pi-compat-<date>`. Patch affected imports in `extensions/subagent/`. Verify the `subagent` tool registers and a sample delegation runs. | Hotfix branch |
| **T+24h** | Merge hotfix, push a patch tag, note the affected pi version in the README Requirements. | Patch release |

**Adopter impact:** Users pinned to the prior pi version are unaffected; users on latest get a same-week patch.

---

## Monitoring Commitment

| Event | SLA | Measurement |
|-------|-----|-------------|
| pi publishes a subagent/extension-API change in its Changelog | This document updated within **72 hours** | Changelog timestamp vs. this file's `Last updated` line |
| `subagent` extension fails to load against latest pi | Hotfix branch open within **24 hours** | Report/CI timestamp vs. branch creation |
| New pi minor/major release | README Requirements min-version row reviewed within **7 days** | Release timestamp vs. README diff |

**Sources we monitor:**

- pi releases — [earendil-works/pi](https://github.com/earendil-works/pi) GitHub Releases / CHANGELOG
- `@earendil-works/pi-coding-agent` on npm (peer API surface)
- pi [Discord](https://discord.com/invite/3cU7Bz4UPx) community signal

---

## FAQ for Adopters

### Q1. Is there any preview/experimental flag to enable, like the Claude Code version?

**A.** No. The Claude Code original required `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`. This pi port bundles the `subagent` extension, so delegation works out of the box after `pi install`.

### Q2. If the `subagent` extension breaks, do my generated harnesses break?

**A.** Your `.pi/agents/*.md`, `.pi/skills/*`, and `.pi/prompts/*.md` are plain Markdown and stay valid. Only the delegation runtime (the extension) would need a patch — see Scenario C. Single-agent skill usage keeps working regardless.

### Q3. Can I use project-local agents safely?

**A.** The `subagent` tool loads project-local `.pi/agents/*.md` only when invoked with `agentScope: "both"`/`"project"`, and it prompts for confirmation in interactive mode. Enable only for repositories you trust. See `extensions/subagent/README.md`.

---

**Related documents:**
- [`docs/quickstart.md`](./quickstart.md) — 5-minute install walkthrough
- [`extensions/subagent/README.md`](../extensions/subagent/README.md) — the bundled delegation tool
- [`skills/harness/references/agent-design-patterns.md`](../skills/harness/references/agent-design-patterns.md) — full Claude → pi mapping
