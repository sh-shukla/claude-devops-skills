---
name: terraform-review
description: Use when reviewing a Terraform plan, diff or PR before apply — ranks destructive changes, security regressions and state risks. Review only; never applies.
---

# Terraform plan review

Input: `terraform plan` output, a plan JSON (`terraform show -json`), or a `.tf` diff. Output: a ranked risk report. This skill never runs `apply`.

## 1. Triage the plan summary

```bash
terraform plan -out=tfplan && terraform show -json tfplan > plan.json
jq -r '.resource_changes[] | select(.change.actions != ["no-op"]) | (.change.actions|join(",")) + "  " + .address' plan.json | sort
```

Flag in this order of severity:

1. **delete** and **replace** (`delete,create` / `create,delete`) — name every resource and why it's being replaced. Replacements hide behind innocent-looking changes to immutable fields (name, AZ, engine version, launch template).
2. **IAM & security groups** — any change, even "add".
3. **Data-bearing resources** — RDS, EBS, S3, DynamoDB: a replace here is data loss unless snapshots/deletion protection are proven in the plan.

## 2. Security regressions

- Ingress `0.0.0.0/0` (or `::/0`) on anything that isn't a public LB on 80/443.
- IAM policies gaining `"Action": "*"`, `"Resource": "*"`, or `iam:PassRole` without a condition.
- S3: `force_destroy = true`, public ACLs, dropped `block_public_*` settings.
- Secrets appearing as plaintext in values — flag for rotation, quote only the key name, never the value.
- Encryption flags flipping off (`storage_encrypted`, `kms_key_id` removals).

## 3. State and blast-radius checks

- `moved` blocks vs delete+create: a rename without `moved` destroys and recreates.
- Count/for_each key changes that re-index whole collections.
- Provider or module version bumps in the same PR as resource changes — ask to split.
- Changes outside the stated scope of the PR — list them as "unrelated drift picked up by plan".

## 4. Report format

| Risk | Resource | Action | Why it matters | Mitigation |
|---|---|---|---|---|
| 🔴 | aws_db_instance.main | replace | engine bump forces new instance → data loss window | snapshot + blue/green, or pin version |

End with one verdict line: **safe to apply / apply with steps / do not apply**, plus the exact pre-apply steps (snapshot, state mv, targeted apply order).
