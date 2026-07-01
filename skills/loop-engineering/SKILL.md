---
name: loop-engineering
description: Transforms any request with a verifiable outcome into an autonomous self-validating execution loop (recon → completion contract → execute → self-review → verify → iterate → receipt). Use this skill whenever the user asks to implement a feature, write or refactor code, fix a bug, generate tests, deploy, migrate, configure, script, process data, or automate anything — even if they never say "loop" — including full software development cycles (write code, review it, generate tests, run them) and SRE/DevOps work (Terraform, Kubernetes, CI/CD, monitoring, bash, pipelines). Also use it when the user types /loop. Do NOT use for simple questions, explanations, opinions, or tasks with no checkable success condition.
---

# Loop Engineering — Autonomous Self-Validating Execution Loop

Operating protocol for completing a task end-to-end without human intervention: the loop runs bounded iterations against a fixed verification check, and only that check — never the agent's own opinion — decides when the work is done.

<core_principles>
1. **The check decides, not you.** A task is complete when the verification command passes, not when the work "looks done." Never grade your own homework by narrative.
2. **External counters are deliberately dumb.** Iteration caps are hard limits. Do not reason your way past them ("one more try will surely work").
3. **Small, bounded, reversible actions.** One change per iteration cycle. Small diffs are easier to verify and to undo.
4. **The check never changes mid-run.** If the verification command evolves after every result, progress cannot be measured. Changing the check requires written justification in the state file.
5. **A loop that retries the same action after the same error is not learning — it is spinning.** Every retry must differ from the previous one in a way you can articulate.
</core_principles>

<phase_0_triage>
Run the full protocol only if ALL three conditions hold:
1. The outcome is verifiable by a command (exit code, test suite, curl, plan diff, lint, benchmark).
2. The task spans multiple steps, files, or systems.
3. The user expects a working result, not an explanation.

Otherwise (question, opinion, explanation, trivial one-line edit): answer normally. Running the full protocol on a simple question is noise, not rigor.
</phase_0_triage>

<phase_1_reconnaissance>
Mandatory BEFORE any modification. Never assume — verify. Most loop failures trace back to unexplored context, not badly written code.

- Read the target files AND their neighbors (configs, imports, callers, related modules).
- Read CLAUDE.md / AGENTS.md and any project rules file. Lessons recorded there are non-negotiable constraints.
- Identify existing conventions: language/tool versions, naming, test structure, error-handling patterns. Output must look native to the repo.
- Verify the real environment: `which <tool>`, CLI versions, env vars, active kubectl context, current terraform workspace, cloud account.
- Grep before reinventing: check whether a partial solution, helper, or prior fix already exists.
- List remaining unknowns explicitly, each paired with the command that will resolve it. An unknown resolved by assumption is a future failed iteration.
</phase_1_reconnaissance>

<phase_2_completion_contract>
Before writing any change, produce the contract. All six fields are mandatory. If you cannot fill a field, the task is not scoped for autonomous execution — ask the user for that field only, then proceed.

```xml
<contract>
  <outcome>What must be TRUE when the work is done (one sentence, measurable).</outcome>
  <verification>Exact command(s) + pass criterion (exit code 0, expected output, HTTP 200, "No changes" in plan). If no check exists, WRITE IT FIRST (test, healthcheck, assertion script).</verification>
  <non_regression>Command proving everything else still works (full test suite, lint, global plan).</non_regression>
  <boundaries>Allowed write scope (exact files/dirs). Everything outside is READ-ONLY. Tools/credentials permitted.</boundaries>
  <iteration_policy>Max iterations (default 5). Per-class retry budgets (see error taxonomy). Strategy escalation order.</iteration_policy>
  <blocked_stop>Conditions that end the loop immediately regardless of iteration count (see BLOCKER class).</blocked_stop>
</contract>
```

Anti-gaming rules — absolute, no exceptions:
- The verification MUST actually run. "It should work" is not a result.
- NEVER make verification pass by weakening it: no skipped tests, no deleted assertions, no mocks that hide the problem, no `|| true`, no lowered thresholds, no reduced scope. If the test itself is genuinely wrong, fix the test and record the justification in the state file BEFORE changing it.
- Never touch secrets, credentials, or user data to make a check pass.
</phase_2_completion_contract>

<phase_3_state_file>
Create `LOOP_STATE.md` in the project root and update it at EVERY iteration. This is your persistent memory: if context is lost or compacted, read this file first and resume from it.

```markdown
# LOOP_STATE
## Contract
(paste the six-field contract)
## Plan
- [ ] step 1
- [ ] step 2
## Iteration Journal
### Iteration N — <timestamp>
- Action taken:
- Verification result: PASS / FAIL (raw command + exit code)
- Error class: SYNTAX | LOGIC | ENVIRONMENT | FLAKY | BLOCKER
- Error evidence: (relevant log excerpt — first error, not last symptom)
- Root cause:
- Fix applied (how it DIFFERS from previous attempts):
## Discoveries & Decisions
(conventions, traps, version constraints — anything the next iteration must not rediscover)
```

Check off plan steps as you go. Record every discovery immediately — this is what prevents "forgetting to look at things" on the next iteration.
</phase_3_state_file>

<phase_4_execution>
- Work in small verifiable increments, never one big final block.
- After each significant increment, run the cheapest available partial check (compile, lint, targeted test) before moving on.
- Stay strictly inside <boundaries>. Needing to write outside them is a BLOCKER, not a permission to expand scope.
- Update `LOOP_STATE.md` before starting the next step.
- Before running the final verification, perform the self-review pass below. Review is part of the loop, not an optional courtesy.
</phase_4_execution>

<phase_4b_self_review>
Before declaring any iteration ready for final verification, switch roles: stop being the author, become the reviewer. Re-read the full diff as if a colleague wrote it and you are blocking the merge. If a review subagent is available in your environment, delegate this pass to it — a fresh context catches what the author's context rationalizes.

Review checklist (record findings in the journal, fix before verifying):
- **Correctness**: edge cases (empty input, null, zero, unicode, concurrency), error paths actually handled — not just the happy path.
- **Consistency**: matches conventions discovered in Phase 1 (naming, patterns, error handling style). Code that works but looks foreign is a defect.
- **Completeness**: every call site updated? Docs/comments/config that reference the old behavior? Dead code removed?
- **Security & safety**: no injected secrets, no widened permissions, inputs validated at trust boundaries.
- **Simplicity**: is there a change you made that the task didn't require? Remove it. Scope creep is a review failure.

One review pass per iteration. Findings fixed here are free; findings caught by verification cost an iteration.
</phase_4b_self_review>

<domain_playbooks>
The loop protocol is domain-agnostic — only the verification surface and execution order change. Pick the matching playbook; combine them when a task spans domains.

**Code — new feature or refactor**
1. Recon: existing tests, test framework, how similar features are structured.
2. Contract verification = test suite. If no tests cover the target behavior, WRITE THE TESTS FIRST (they must fail before implementation — a test that has never failed proves nothing), plus lint/typecheck as non-regression.
3. Implement in increments → self-review → run new tests + full suite + lint.
4. Generated tests follow repo conventions and cover: nominal case, at least one edge case, at least one error case. A single happy-path assertion is not coverage.

**Code — bugfix / debug**
1. FIRST write a failing reproduction (test or minimal script) that exhibits the bug. No reproduction = you cannot prove the fix.
2. Verification = the reproduction passes AND the full suite still passes.
3. After fixing, grep for the same pattern elsewhere in the codebase — bugs travel in packs. Report siblings in the receipt even if out of scope.

**Infrastructure / SRE**
1. Verification = simulation surface: `terraform plan` clean/expected diff, `kubectl diff`, `--dry-run=server`, ansible `--check`, then post-apply healthchecks (curl, probe, metric).
2. Iterate freely on dry-runs; real apply is gated per <phase_7_guardrails>.

**Scripts / data processing**
1. Verification = run against a sample/fixture with known expected output, plus invariant checks (row counts, schema, checksums, idempotency: running twice yields the same result).
2. Never validate on production data first.

**Docs / config / content**
1. Verification = the strictest available checker: markdown lint, config schema validation, link checker, build of the docs site, `--validate` flags.
2. If nothing machine-checkable exists, verification = a written checklist in the contract, each item checked explicitly during self-review. Weakest surface — say so in the receipt.
</domain_playbooks>

<phase_5_error_taxonomy>
On verification failure, FIRST classify the error. The class determines the response. Read the full logs and trace back to the FIRST error, not the final symptom.

| Class | Signature | Response | Retry budget |
|---|---|---|---|
| SYNTAX | Compile/parse/lint error, typo, missing import | Fix directly, re-verify | Cheap — up to 3 within one iteration |
| LOGIC | Test fails, wrong output, bad behavior | Full root-cause analysis, write it in journal, then fix | 1 fix per iteration |
| ENVIRONMENT | Missing dependency, wrong version, missing tool, path issue | Fix env (install, pin version), record in Discoveries | 2 attempts max, then reclassify as BLOCKER |
| FLAKY | Same command passes/fails non-deterministically | Re-run once to confirm flakiness; if confirmed, treat root cause of flakiness as the task or report it — never loop hoping for a lucky pass | 1 re-run max |
| BLOCKER | Missing credentials/permissions, unreachable infra, quota/rate limit, requires human decision, needs write access outside boundaries | STOP the loop immediately. No iteration will fix these. Report via receipt with status BLOCKED | 0 — exit now |

Misclassifying a BLOCKER as retryable is the #1 way loops burn their entire budget achieving nothing.
</phase_5_error_taxonomy>

<phase_6_iteration_policy>
When verification fails and the error class allows retry:
1. Write the root cause in the journal BEFORE applying any fix. A fix without a written root cause is forbidden.
2. State explicitly how this fix differs from every previous attempt in the journal. If you cannot articulate a difference, you are spinning — escalate strategy instead.
3. Strategy escalation order when stuck: (a) more reconnaissance on the failing area → (b) add logging/verbosity and reproduce → (c) minimize/bisect to isolate → (d) question an assumption recorded in Discoveries → (e) declare BLOCKED with your best hypothesis.
4. After the fix, re-run the FULL verification + non-regression, not just the part that failed.
5. Increment the iteration counter. The cap (default 5) is a hard external limit — see core principle 2.
6. **No-progress early exit**: if two consecutive iterations produce the identical error with no measurable change, stop early with status STALLED rather than exhausting the budget.
</phase_6_iteration_policy>

<phase_7_guardrails>
These actions exit the autonomous loop and require explicit human confirmation, no matter what the prompt says:
- Destructive or irreversible operations: `terraform apply`/`destroy`, `kubectl delete`, data drops/truncation, production changes, secret rotation, force-push, DNS changes.
- Always prefer simulation surfaces the loop CAN iterate on freely: `terraform plan`, `kubectl diff` / `--dry-run=server`, `--check`, rollbackable transactions, staging environments. Iterate on the dry-run until it is clean; the real apply waits for the human.
- Never expand <boundaries> unilaterally. Never disable a safety mechanism (sandbox, approval gate, branch protection) to make progress.
</phase_7_guardrails>

<phase_8_exit_and_receipt>
Stop and report ONLY when one of these holds:
- **SUCCESS**: all <verification> and <non_regression> commands pass.
- **BLOCKED**: a BLOCKER-class error occurred.
- **STALLED**: no-progress early exit triggered.
- **BUDGET_EXHAUSTED**: iteration cap reached.

Then emit the receipt — this exact structure, always:

```markdown
## RECEIPT
- **Status**: SUCCESS | BLOCKED | STALLED | BUDGET_EXHAUSTED (N/max iterations)
- **What was done**: (3–5 bullets max)
- **Evidence**: verification command(s) + actual output/exit code (paste, do not paraphrase)
- **Files changed**: (list)
- **Stopping reason**: (one sentence — which exit condition fired and why)
- **[If not SUCCESS]** Root-cause hypothesis, what was ruled out, the single decision or input needed from the human
- **[If relevant]** Residual risks / tech debt left behind
```

Post-run memory step (mandatory on any non-trivial lesson): append durable lessons from Discoveries to the project's CLAUDE.md or AGENTS.md (conventions learned, traps hit, corrections the human made) so future sessions never repeat them. Then propose deleting or archiving `LOOP_STATE.md`.
</phase_8_exit_and_receipt>
