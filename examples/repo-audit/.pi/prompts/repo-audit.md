---
description: Audit a codebase from architecture, security, and quality angles, then integrate and verify into one prioritized report
argument-hint: "[path]"
---
Run a codebase audit on `${1:-.}` using the `subagent` tool with `agentScope: "both"`.

## Phase 0 — context & triage
1. Check for an existing `_workspace/` audit run. If the user asked to update a specific area, re-run only that scout; otherwise start a fresh run.
2. Assign a run id: `audit-NN` (increment from existing `_workspace/audit-*` dirs). Create `_workspace/<run_id>/`.
3. Write `_workspace/<run_id>/manifest.md`:
   ```
   # Run: <run_id>
   - target: ${1:-.}
   - team: [architecture-scout, security-scout, quality-scout, audit-synthesizer, skeptic]
   - steps: [ ] scout  [ ] synthesize  [ ] verify  [ ] finalize
   ```

## Phase 1 — parallel scouts (fan-out)
Call `subagent` in **parallel** mode (agentScope: "both"), passing `<run_id>` and target path in each task:
- `architecture-scout` → `_workspace/<run_id>/01_architecture.md`
- `security-scout` → `_workspace/<run_id>/01_security.md`
- `quality-scout` → `_workspace/<run_id>/01_quality.md`

Each scout is read-only and returns a HANDOFF summary. Update the manifest: `scout` done.

## Phase 2 — synthesize + verify (chain)
Read the three `01_*.md` reports, then call `subagent` in **chain** mode:
```
chain: [
  { agent: "audit-synthesizer", task: "Integrate the three scout reports for run <run_id> into AUDIT.md." },
  { agent: "skeptic",           task: "Verify every finding in AUDIT.md against the code. Return FAIL items: {previous}" },
  { agent: "audit-synthesizer", task: "Fix only the FAILed findings (do not widen scope): {previous}" }
]
```
Update the manifest as steps complete. The repair pass is bounded to this single round.

## Phase 3 — report
1. Confirm `AUDIT.md` exists in the project root and reflects the verified findings.
2. Summarize the top Critical/Warning items to the user.
3. **Do not commit, push, or modify source files.** This harness produces a report; the user decides what to do with it.

## Error handling
- A scout that fails → retry once; if it fails again, proceed and note the missing angle in AUDIT.md.
- If the skeptic FAILs the majority of findings → tell the user the draft was weak and offer to re-run the scouts.

## Test scenario
- Normal: `/repo-audit src/` → 3 scouts run in parallel → synthesizer drafts AUDIT.md → skeptic verifies → synthesizer finalizes → top findings reported.
- Error: `security-scout` fails twice → architecture + quality proceed → AUDIT.md notes "security angle not audited".
