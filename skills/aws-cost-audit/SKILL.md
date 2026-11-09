---
name: aws-cost-audit
description: Use when AWS spend needs explaining or cutting — a read-only sweep of the usual money leaks (unattached volumes, idle load balancers, oversized instances, gp2, old snapshots) ending in a savings table.
---

# AWS cost audit — read-only

Every command here is `describe|list|get`. The report proposes; a human disposes. Never create, modify or delete anything during an audit.

## 1. Where the money goes

```bash
aws ce get-cost-and-usage --time-period Start=$(date -v-30d +%F),End=$(date +%F) \
  --granularity MONTHLY --metrics UnblendedCost --group-by Type=DIMENSION,Key=SERVICE \
  --query 'ResultsByTime[0].Groups|sort_by(@,&Metrics.UnblendedCost.Amount)[-10:]' --output table
```

Top 10 services first. Audit the top three in depth before touching anything else — a 10% cut there beats a 100% cut of a $4 line item.

## 2. The usual leaks, in payoff order

| Leak | Find it | Typical saving |
|---|---|---|
| Unattached EBS volumes | `aws ec2 describe-volumes --filters Name=status,Values=available` | 100% of each |
| gp2 volumes | `aws ec2 describe-volumes --filters Name=volume-type,Values=gp2` | ~20% via gp3 |
| Unassociated Elastic IPs | `aws ec2 describe-addresses --query 'Addresses[?AssociationId==null]'` | small but pure waste |
| Idle load balancers | ELBs with ~0 RequestCount over 14 days (CloudWatch) | $16–25/mo each |
| Oversized instances | avg CPU < 10% over 14 days (CloudWatch `CPUUtilization`) | ~50% via rightsize |
| Ancient snapshots | `describe-snapshots --owner-ids self` filtered > 180 days | storage line shrinks |
| Previous-gen instance families | t2/m4/r4 in `describe-instances` | ~10–20% via t3/m5/r5 |
| NAT gateway chatter | Cost Explorer: `NatGateway-Bytes` by AZ | VPC endpoints for S3/ECR |

## 3. Don't embarrass the audit

- Check the instance isn't a standby/DR node before calling it "idle" — low CPU is its job.
- Snapshots may be the only backup of a deleted volume: flag age, never assume deletable.
- Reserved/Savings-Plan coverage changes the math — `aws ce get-savings-plans-utilization` before recommending instance changes.

## 4. Report format

| # | Finding | Evidence | Est. $/month | Action | Risk |
|---|---|---|---|---|---|

Sorted by savings, each action phrased as the exact console step or CLI command **for the owner to run** — the audit itself stays read-only. End with: total addressable $/month, and the three actions worth doing this week.
