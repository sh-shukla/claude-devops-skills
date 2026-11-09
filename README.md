# claude-devops-skills

Hand-written [Claude Code](https://claude.com/claude-code) skills for DevOps and platform engineers — the workflows I actually run: Kubernetes triage, Terraform plan review, incident command, AWS cost audits, CI/CD debugging, and turning the work into words.

Each skill is a `SKILL.md` following the [Agent Skills](https://github.com/anthropics/skills) format: a description that tells Claude *when* to load it, and a body that encodes *how* a senior engineer does the job — including what **not** to do.

## The skills

| Skill | When Claude uses it |
|---|---|
| [`k8s-debug`](skills/k8s-debug/SKILL.md) | CrashLoopBackOff, pending pods, OOMKills, DNS/RBAC failures — reads before it touches |
| [`terraform-review`](skills/terraform-review/SKILL.md) | Reviews plans/diffs for destroys, replacements and security regressions — never applies |
| [`incident-commander`](skills/incident-commander/SKILL.md) | Severity call, first-15-minutes checklist, comms cadence, blameless postmortem |
| [`aws-cost-audit`](skills/aws-cost-audit/SKILL.md) | Read-only sweep of the usual money leaks, ending in a savings table |
| [`cicd-doctor`](skills/cicd-doctor/SKILL.md) | Red or flaky pipelines — classifies the failure before fixing the class |
| [`linkedin-devops-post`](skills/linkedin-devops-post/SKILL.md) | Turns real engineering work into a post engineers don't scroll past |

## Install

```bash
git clone https://github.com/sh-shukla/claude-devops-skills.git
cp -r claude-devops-skills/skills/* ~/.claude/skills/        # all of them, every project
# or per-project: cp -r claude-devops-skills/skills/k8s-debug <repo>/.claude/skills/
```

Claude Code picks them up automatically — mention a crashlooping pod and `k8s-debug` loads itself.

## Design principles

- **Read-only by default.** Diagnosis never mutates: no deletes as diagnostics, no applies during review, no cost "fixes" during an audit.
- **Classify, then fix the class.** A retried flaky test is still flaky; a bumped memory limit without the usage curve is still a leak.
- **Evidence before verdict.** Every skill ends in a report format that forces exact log lines, numbers and blast radius.

## Worth knowing in the ecosystem

- [anthropics/skills](https://github.com/anthropics/skills) — the official spec and reference skills
- [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) · [VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills) · [gmh5225/awesome-skills](https://github.com/gmh5225/awesome-skills) — curated catalogs
- Registries like [claudeskills.info](https://claudeskills.info/best/devops-skills/) index thousands of DevOps-tagged skills — read the source of anything before installing it

## License

[MIT](LICENSE)
