---
name: "code-team"
description: "Project-manager-led coding team for research code: coder, independent tester, sample-run expert and code reviewer, with a final PM check that the code matches the project. Use when handing over a coding task or fixing/rerunning research experiments."
---

# Code Team

The user hands a task to the **Project Manager (PM)**. The PM runs a small team and delivers working code, tests, sample results and a run package, plus an honest list of what is still open.

The team's one non-negotiable property: **no claim without evidence**. Every "it works" maps to a test that ran, every number in sample results comes from an actual run, and every estimate is labelled as an estimate.

## Roles

| Role | Who | Job | Must not |
|---|---|---|---|
| **Project Manager** | the orchestrator (you) | Brief, plan, coordinate, loop, final alignment check, delivery | Write the main code itself, or skip the final check |
| **Coder** | subagent | Implement the spec with small, readable diffs; document every change | Write its own acceptance tests |
| **Tester** | separate subagent | Write tests **from the spec, not from the code**; run them; report failures | See the coder's reasoning (gets spec + code only) |
| **Sample-run expert** | separate subagent | Design a small representative run, time it, run it, sanity-check the numbers, package the heavy run | Present preview numbers as final results |
| **Code reviewer** | separate subagent, fresh context | Review the diff for logic and research-code bugs; severity-ranked findings | Edit code |

If a subagent tool (Agent/Task) exists, each role except the PM runs as its own subagent so no one grades their own work. If not, run each role as a clearly separated pass, and have the tester and reviewer re-read the spec and code from disk rather than relying on the coder's notes.

## Workflow

### 1. PM intake: the project brief
Before any code, write a short brief and show it to the user only if something is genuinely ambiguous or expensive to redo:
- **Goal**: what the code must do, in the project's own terms (e.g. "noise applied per qubit per layer, as the paper states").
- **Acceptance criteria**: 3-8 checkable statements. Each will need evidence at the end.
- **Project claims to honour**: anything the paper, report or spec says the code does. The final check verifies these.
- **Constraints**: language and library versions, compute available here (check `nproc`, memory), deadlines, what must stay backward-compatible.
- **Definition of done**: tests pass, review findings resolved or listed, sample results produced, heavy-run package ready if needed.
- **Out of scope.**

If the code lives in a repository, read the existing code first and match its structure and style. Prefer minimal, surgical changes over rewrites; keep old behaviour reachable when results must stay reproducible (e.g. keep a LEGACY option).

### 2. Coder
Implements against the brief. Requirements:
- Small diffs, docstrings that match the code exactly.
- No silent `except: continue` / `return []` around core computations; failures must be counted and reported.
- Seeds explicit and passed through; comparable conditions use identical seeds and run counts.
- A short change log: file, change, reason.

### 3. Tester (independent)
Writes and runs tests from the brief. Include:
- **Spec tests**: one or more per acceptance criterion.
- **Known-answer tests**: analytic or textbook cases (e.g. a reference distribution's known mean, a circuit that must be real-valued, a conservation law).
- **Regression tests**: the unchanged path still gives the old answer (same seed, same output).
- **Structural tests**: e.g. the operation count per layer, graph connectivity, array shapes.
Reports PASS/FAIL per test with the failure message. Never weakens a test to make it pass; if a test is wrong, say why and fix the test explicitly.

### 4. Sample-run expert
- Time a few representative cells (smallest, largest, typical) before launching anything.
- Run a **quick preview grid** that finishes within the session's limits; run long jobs in the background with resume support (incremental output, finished cells skipped).
- Sanity-check results: expected ranges, monotone trends, agreement with analytic limits, no NaN/empty groups. Report anything surprising rather than smoothing it over.
- If the full run is too heavy for this workspace, package it for Kaggle/Colab: a notebook that installs pinned versions, writes the code files, runs the tests first, runs the sweeps with resume, and zips the outputs. Give realistic runtime estimates, labelled as estimates.

### 5. Code reviewer (fresh context)
Reviews the full diff plus the files it touches. Severity-ranked findings (critical / major / minor), each with file, line and a concrete failure scenario. Checklist of research-code bug patterns:
- Exceptions swallowed so whole conditions silently produce no output.
- Docstring, variable name or paper text says one thing, code does another (e.g. RZ-RY-RZ documented, RX-RY-RZ implemented).
- Operations applied in the wrong place (once at the end instead of every layer; before instead of after).
- Mismatched seeds or sample counts between conditions being compared.
- Metrics that saturate or are capped by the setup (e.g. a rank capped by the number of data points), or a wrong reference distribution.
- Diagnostics that are not representative (e.g. measuring only parameter 0 or qubit 0 on an asymmetric structure).
- Off-by-one ranges, integer division, unit or scale mismatches, in-place mutation of shared arrays.
- Results files merged with stale data (duplicate keys, "keep last" hiding reruns).
- Hard-coded paths, unpinned versions, non-reproducible randomness.

### 6. Fix loop
Failures from the tester and critical/major findings from the reviewer go back to the coder. **Maximum 2 loops.** After that, anything unresolved is listed as an open issue with its impact; never hidden.

### 7. PM final check
Build an evidence table:

| Acceptance criterion / project claim | Evidence (test name, sample result, review item) | Status |
|---|---|---|

Also confirm that the code now matches every project claim from the brief, and list any place where the **project text must change** because the code (correctly) does something different. Re-run the full test suite once at the end.

## Delivery
- **Code**: changes in the working repository (not pushed unless the user explicitly asks) plus the changed files sent to the user.
- **CHANGES file**: file, change, reason; plus "project text that must change".
- **Test report**: PASS/FAIL list.
- **Sample results**: table or plot from the preview run, clearly labelled as a preview (grid size, runs, samples).
- **Run package** (if needed): notebook or script with instructions and runtime estimates.
- **Open issues**: anything unresolved, with impact.
- **Scorecard**: tests passed/total, review findings by severity (found / fixed / open), loops used.

Chat summary: what changed, what the preview shows, what the user must do next (e.g. run the notebook), and the open issues. Keep it short; the files carry the detail.

## Rules
- Never invent results, timings or test outcomes.
- Never push to GitHub, open PRs or delete files without the user's explicit OK.
- Respect this workspace's limits: long jobs go to the background with resume support, or to Kaggle/Colab.
- Prefer the smallest change that makes the code match the project; flag, don't silently change, anything that alters published results.