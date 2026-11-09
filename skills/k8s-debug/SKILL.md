---
name: k8s-debug
description: Use when debugging Kubernetes workloads — CrashLoopBackOff, pending pods, OOMKills, failing probes, DNS/networking or RBAC errors. Structured triage that reads before it touches.
---

# Kubernetes workload triage

Diagnose in this order. Read-only first: no deletes, no restarts, no edits until the cause is named.

## 1. Establish the blast radius

```bash
kubectl get pods -n <ns> -o wide | grep -v Running
kubectl get events -n <ns> --sort-by=.lastTimestamp | tail -20
kubectl get nodes -o wide   # NotReady? cordoned? version skew?
```

One pod, one node, or everywhere? That answer halves the search space before anything else.

## 2. Classify the symptom

| Symptom | First command | Usual causes |
|---|---|---|
| CrashLoopBackOff | `kubectl logs <pod> --previous` | app exits on boot: bad config/env, missing dependency, migration failure |
| OOMKilled (exit 137) | `kubectl describe pod` → Last State | limits too low, real leak, JVM/Go heap not matching cgroup |
| Pending | `kubectl describe pod` → Events | no schedulable node: resources, taints, affinity, PVC unbound |
| ImagePullBackOff | `kubectl describe pod` → Events | wrong tag, private registry secret missing/expired |
| Probe failures | `kubectl describe pod` → Events | probe too aggressive at startup, app not listening on probed port |
| Service unreachable | `kubectl get endpoints <svc>` | empty endpoints = selector/label mismatch or no ready pods |

## 3. Dig where the classification points

- **Logs:** always check `--previous` for crash loops — the live container hasn't failed yet.
- **DNS:** `kubectl run tmp --rm -it --image=busybox:1.36 --restart=Never -- nslookup <svc>.<ns>.svc.cluster.local`
- **RBAC:** `kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<ns>:<sa>`
- **Resources:** `kubectl top pods -n <ns>`; compare against requests/limits in the spec, not against the node.
- **NetworkPolicy:** if traffic dies between two namespaces, list policies in both — default-deny in one direction is the classic.

## 4. Report before fixing

State: symptom → evidence (exact event/log lines) → cause → proposed fix → blast radius of the fix.
Fixes go through the deploy path (git), not `kubectl edit`. Suggest `kubectl rollout undo` only when the breakage correlates with a fresh rollout and the user confirms.

## Never

- Never `kubectl delete pod` as a diagnostic — it destroys the evidence.
- Never raise memory limits without reading the actual usage curve first.
- Never conclude from an empty `kubectl logs` — check the container actually started (`describe` → State).
