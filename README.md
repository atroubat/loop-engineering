# loop-engineering

A [Claude Code](https://claude.com/claude-code) skill that turns any request with a verifiable outcome into an autonomous, self-validating execution loop: recon → completion contract → execute → self-review → verify → iterate → receipt.

Instead of stopping when the work "looks done," the agent runs bounded iterations against a fixed verification check (tests, lint, exit code, plan diff, curl, benchmark...) and only that check decides when the task is complete.

## Core principles

- **The check decides, not the agent.** No self-graded homework.
- **Hard iteration caps.** Default max 5 iterations, enforced externally.
- **Small, bounded, reversible actions.** One change per iteration.
- **The verification command never changes mid-run** without written justification.
- **No spinning.** Two identical failures in a row triggers an early `STALLED` exit instead of burning the full budget.

## Install

Copy the skill into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills
cp -r skills/loop-engineering ~/.claude/skills/loop-engineering
```

Claude Code will pick it up automatically on the next session (or invoke it directly with `/loop`).

## Usage

The skill triggers automatically on any task with a checkable success condition: implementing a feature, fixing a bug, writing tests, deploying, migrating, scripting, automating infra (Terraform, Kubernetes, CI/CD)...

It does **not** trigger on simple questions, explanations, or opinions with no verifiable outcome.

## License

MIT
