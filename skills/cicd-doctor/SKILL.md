---
name: cicd-doctor
description: Use when a CI/CD pipeline is red or flaky — GitHub Actions or Jenkins. Classifies the failure from the logs (deps, cache, env, secrets, flaky test, runner) and fixes the class, not the symptom.
---

# CI/CD pipeline doctor

Rule one: read the log of the **first** failing step, not the last red box — downstream failures are usually noise from the first one.

## 1. Pull the evidence

```bash
# GitHub Actions
gh run list --limit 10                       # is it this branch only? all branches? since when?
gh run view <id> --log-failed | tail -100    # the actual error, not the summary
```

For Jenkins: the first `ERROR`/`FAILED` in the console log, plus the stage view to see which stage broke.

## 2. Classify before fixing

| Class | Smell in the log | Fix the class |
|---|---|---|
| Dependency | `404 Not Found`, `ERESOLVE`, version solver errors | pin via lockfile; never "re-run until green" |
| Cache poisoning | works locally, old versions appear in CI | bump the cache key, don't delete steps |
| Environment drift | `command not found`, wrong tool version | pin runner image/tool versions explicitly |
| Secrets | `401/403`, empty env var in masked logs | expired token or wrong scope — rotate/rescope, check environment-level vs repo-level secrets |
| Flaky test | same commit red then green | quarantine with a tracking issue; retries hide, never cure |
| Runner/infra | OOM, disk full, killed at random step | bigger runner or trim the workspace; check `df`/`dmesg` steps |
| Timeout | step killed at a round number | find what got slower first — the timeout is the messenger |

The one-question diagnostic: **"did this ever pass, and what changed since?"** — a never-green pipeline is a build problem; a was-green pipeline is a diff problem (`git log --oneline since-last-green..HEAD`, including CI config and base-image changes).

## 3. Flaky tests deserve their own paragraph

- Reproduce locally with the CI's parallelism: `pytest -n auto --count=20 <test>` (or the runner's equivalent).
- Usual suspects: shared state between tests, time/timezone assumptions, port collisions, order dependence, real network calls.
- Quarantine (skip-with-ticket) beats retry-loops: retries normalize a 5% failure into permanent background noise.

## 4. Verify like production

Re-run the **exact** failed job, not a fresh pipeline — a fresh run can pass for unrelated reasons (new cache, new runner). Green must be attributable to the fix, or it isn't a fix. Never pipe test output through anything that swallows the failure count.
