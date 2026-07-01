# Quickstart — 5 Minutes to Your First Harness (pi)

> **Time budget: 5 minutes (strict).** If you are not at Step 5 within 5 minutes, stop and file an issue — that is a bug in this document, not a bug in you.

**What you will have at the end:** a working `.pi/agents/` directory with 3–5 domain-specialized agents, their skills in `.pi/skills/`, and an orchestration prompt in `.pi/prompts/` — generated from a single-sentence prompt, ready to run on a sample task.

**Prerequisites (check before starting):**
- [pi coding agent](https://github.com/earendil-works/pi) installed (`pi --version` should return a version)
- A model provider authenticated in pi (`/login`, or an API key such as `ANTHROPIC_API_KEY`)
- Network access to `github.com` and your provider's API

> **No experimental flags.** Unlike the Claude Code original, this port bundles a `subagent` extension that supplies multi-agent delegation. There is nothing extra to enable.

---

## Step 1 — Install the harness package (60 seconds)

```bash
pi install git:github.com/baryonlabs/pi-agent-harness
```

**What this does:** Installs the `pi-agent-harness` package into your user settings (`~/.pi/agent/settings.json`). The bundled `subagent` extension and the `harness` skill load automatically on the next `pi` start.

**Expected output:** confirmation that the package was installed. Verify with `pi list`.

*(Prefer a project-local install shareable with your team? Use `pi install -l git:github.com/baryonlabs/pi-agent-harness` — it writes to `.pi/settings.json`.)*

---

## Step 2 — Start pi and confirm the tools (30 seconds)

```bash
pi
```

Inside pi, the `subagent` tool and the `harness` skill should now be available (the skill appears in the system prompt's skill list; force-load it any time with `/skill:harness`).

**Failure FAQ #1 — `subagent` tool missing / harness skill not listed**
**Cause:** the package didn't load (install wrote to a different scope, or pi cached an old session).
**Fix:** run `pi list` to confirm `pi-agent-harness` is present, then `/reload` inside pi (or restart `pi`).

---

## Step 3 — Generate a harness from one sentence (2 minutes)

Inside pi, type:

```
build a harness for a fintech risk-assessment team
```

**What this does:** Triggers the `harness` meta-skill, which analyzes your domain sentence and scaffolds a team of specialized agents + their skills + an orchestration prompt into `.pi/agents/`, `.pi/skills/`, and `.pi/prompts/` in the current directory.

**Try these alternate prompts** — any of them work:
- `하네스 구성해줘 — 핀테크 리스크 평가 팀` (Korean also works)
- `build a harness for an e-commerce fraud-detection workflow`
- `design an agent team for technical due diligence on open-source repos`

**Expected output:** a plan, then confirmation that 3–5 agent `.md` files, their skills, and an orchestration prompt were written.

---

## Step 4 — Verify the generated files (30 seconds)

From another shell (or `!ls` inside pi):

```bash
ls -la .pi/agents/
ls -la .pi/skills/
ls -la .pi/prompts/
```

**Expected output:** 3–5 agent files (e.g., `risk-analyst.md`, `compliance-reviewer.md`, `portfolio-monitor.md`), matching skill directories, and an orchestration prompt (e.g., `risk-assessment.md`). Each agent file has `name`/`description` and usually `tools`/`model` frontmatter.

**Failure FAQ #2 — "Nothing was generated" / directories are empty**
**Cause:** the harness skill did not trigger, or you were in a directory pi hasn't trusted.
**Fix:** trust the project when prompted, then force-load the skill with `/skill:harness` and repeat Step 3.

---

## Step 5 — Run the generated team against a sample task (90 seconds)

The harness registers an orchestration prompt under `.pi/prompts/`. Invoke it as a `/command` (filename without `.md`), e.g.:

```
/risk-assessment Ticket FIN-427: a new corporate customer (mid-cap manufacturer, $80M revenue, South Korea) applied for a $5M working-capital line. Produce a risk assessment covering (1) credit-history red flags, (2) sector concentration vs. our existing book, (3) regulatory exposure (KFTC, FSC). Output: a 1-page memo with a go/no-go recommendation.
```

**What this does:** the main agent runs the orchestration prompt, delegating to the generated agents via the `subagent` tool (`single`/`parallel`/`chain`, with `agentScope: "both"` so project `.pi/agents/` are discovered), and returns a structured memo. Intermediate artifacts land in `_workspace/`.

**Failure FAQ #3 — "The team doesn't run / project agents aren't found"**
**Cause:** the orchestration prompt invoked `subagent` without `agentScope: "both"`, so only user-level agents (`~/.pi/agent/agents/`) were visible; or the project isn't trusted.
**Fix:** ensure the generated prompt passes `agentScope: "both"` (the harness writes this by default — re-run Step 3 to regenerate if it's missing), and trust the project.

**Failure FAQ #4 — "Too many API calls / cost anxiety"**
**Cause:** `parallel` delegation fans out to up to 8 tasks (4 concurrent), each a separate `pi` process. A complex ticket can consume 50K–200K tokens.
**Fix:** run one task per invocation, and tier models down (`claude-haiku-4-5` for recon agents) via each agent's `model` frontmatter.

---

## You're done

At this point you should have:

- [x] A `.pi/agents/` directory with domain-specialized agents
- [x] A `.pi/skills/` directory with their supporting skills
- [x] A `.pi/prompts/` orchestration prompt (`/command`)
- [x] One successful sample-task execution

**Next reads:**
- [`docs/experimental-dependency.md`](./experimental-dependency.md) — the runtime dependency (pi + the bundled `subagent` extension) and how we adapt when pi's extension API changes
- [`revfactory/harness-100`](https://github.com/revfactory/harness-100) — Catalog of 100+ pre-built domain harnesses (Claude Code format; portable methodology)
- [`revfactory/claude-code-harness`](https://github.com/revfactory/claude-code-harness) — The A/B test harness used to measure +60% quality on 15 tasks (Claude Code original)

**If you hit something this guide didn't cover:** open an issue with the `quickstart-gap` label and include: (a) which step failed, (b) `pi --version`, (c) the exact error message. The SLA for quickstart-gap issues is **48 hours** to first response (see `CONTRIBUTING.md`).
