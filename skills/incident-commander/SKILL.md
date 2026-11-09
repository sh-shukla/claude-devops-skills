---
name: incident-commander
description: Use when production is degraded or down — severity call, first-15-minutes checklist, stakeholder comms on a cadence, timeline keeping, and the blameless postmortem afterwards.
---

# Incident command

Structure beats heroics. This skill drives the process so engineers can drive the fix.

## 1. Severity, in 60 seconds

| Sev | Definition | Response |
|---|---|---|
| SEV1 | Customer-facing outage or data loss, no workaround | All hands, comms every 30 min, leadership paged |
| SEV2 | Major degradation, workaround exists | IC + owning team, comms hourly |
| SEV3 | Minor/internal impact | Owning team, async updates |

When in doubt between two levels, take the higher one — downgrading later is free, upgrading late is not.

## 2. First 15 minutes

1. Name an **Incident Commander** (coordinates, doesn't debug) and a **scribe**.
2. Open a dedicated channel: `#inc-<date>-<slug>`. All discussion moves there.
3. Freeze deploys. Ask first: **"what changed?"** — deploy, flag, config, infra, certificate, quota.
4. Mitigate before root-causing: rollback, flag off, scale up, fail over. The cause can wait; the customers can't.
5. Post the first status update even if it only says "investigating" — silence reads as chaos.

## 3. Comms template (repeat on cadence)

```
[SEV1][Investigating|Mitigating|Monitoring|Resolved] <one-line symptom>
Impact: who/what, since when, % affected if known
Current theory: <or "none yet">
Next update: <time>
```

Never promise an ETA you don't have evidence for. "Next update at :30" is a commitment you can always keep.

## 4. Timeline (scribe keeps it live)

`HH:MM TZ — event` — detections, decisions, actions, rollbacks. Written during, not reconstructed after. This becomes the postmortem's backbone.

## 5. Blameless postmortem (within 5 working days)

Sections: Impact (numbers, duration, customers) · Timeline · Root cause (the system's, never a person's) · What went well · What went poorly · Action items — each with an owner and a date, each one either **detect sooner, mitigate faster, or prevent**. An action item without an owner is a wish.

The test of blameless: the same person, same information, same tools would make the same call. Fix the system that made the wrong call easy.
