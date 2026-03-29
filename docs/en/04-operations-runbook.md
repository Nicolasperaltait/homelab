# 04 — Operations Runbook

## Purpose

This document answers a simple operational question:

> If something stops working, what do I check first, in what order, and how do I avoid making it worse?

That is the real value of a runbook.

---

## Operational priorities

Use this order during troubleshooting:

1. hypervisor / host state
2. network reachability
3. DNS resolution
4. service status
5. data path / storage path
6. backup or recovery path

This order prevents random restarts and guesswork.

---

## Daily checks

| Check | Expected result |
|---|---|
| Hypervisor reachable | yes |
| Internal DNS responding | yes |
| Storage/NAS healthy | yes |
| App platform healthy | yes |
| Security monitoring reachable | yes |
| Latest backup artifacts present | yes |

### Daily checklist
- [ ] verify hypervisor access
- [ ] verify internal DNS
- [ ] verify storage availability
- [ ] verify app platform containers/services
- [ ] verify security monitoring availability
- [ ] verify recent backup presence

---

## Weekly checks

| Check | Why it matters |
|---|---|
| storage capacity | backup failures usually arrive late, not politely |
| archive growth | retention drift is operational debt |
| last successful backup window | confirms jobs are not silently stale |
| last readable archive | checks artifact integrity at a practical level |
| snapshot / restore posture | confirms recovery options still exist |

### Weekly checklist
- [ ] review storage utilization
- [ ] confirm backup windows completed
- [ ] confirm latest archive is readable
- [ ] review logs for repeated warning patterns
- [ ] review top operational risks

---

## First 10 minutes of an incident

### Step 1 — define the symptom
Write down:

- what is failing
- from where it is failing
- whether it fails by name, by IP, or both
- whether the issue affects one service or many

### Step 2 — define the scope
Classify the issue:

- one service
- one VM
- one zone
- multiple zones
- whole environment

### Step 3 — verify the platform
Check:

- hypervisor health
- VM state
- internal routing/gateway behavior

### Step 4 — verify DNS
If IP works but name fails, treat it as DNS first.

### Step 5 — verify storage if the symptom involves backups, NAS, archives, or application data

### Step 6 — avoid mass restarts
Do not restart everything because “something feels weird.”

That is how small incidents become archaeology.

---

## Typical failure scenarios

## Scenario A — Web service unavailable

### Check order
1. name resolution
2. IP reachability
3. VM state
4. reverse proxy path
5. application/container state

### Likely root causes
- DNS issue
- proxy routing issue
- service process failure
- VM unavailable
- network path broken

---

## Scenario B — No outbound connectivity from a VM or service

### Check order
1. gateway presence
2. hypervisor forwarding/routing logic
3. DNS vs pure connectivity distinction
4. service-level retry behavior

### Likely root causes
- routing issue
- NAT/forwarding issue
- DNS failure misread as connectivity loss

---

## Scenario C — Internal DNS failure

### Symptoms
- internal names fail
- IP access still works
- multiple “apps” appear down at once

### Interpretation
This is often a DNS event, not a multi-service outage.

---

## Scenario D — Backup failure or missing artifact

### Check order
1. source data path
2. staging or transfer path
3. archive creation
4. offsite path
5. retention or capacity issue

### Operational reminder
A job timer firing is not proof of a good backup.

---

## Scenario E — Storage pressure

### Signs
- backup jobs fail unexpectedly
- archives stop rotating cleanly
- staging accumulates
- services report write or upload errors

### Response
- measure first
- identify largest consumers
- preserve latest healthy recovery point
- clean with intent, not panic

---

## Rollback vs restore

| Option | Use when |
|---|---|
| rollback / snapshot return | a recent change clearly caused the issue |
| VM-level restore | host/VM state is badly degraded |
| app/data restore | service state is damaged but platform is usable |

---

## Public-safe command categories

This public runbook avoids embedding environment-specific secrets.  
Instead, it recommends categories of checks:

- VM state checks
- service status checks
- DNS checks
- routing checks
- storage capacity checks
- archive listing and validation checks

Detailed implementation commands may remain in private notes if they contain sensitive identifiers.

---

## What makes this runbook useful

A good runbook is not a command dump.

It should:

- reduce panic
- reduce guesswork
- improve triage order
- make recovery decisions easier
- preserve operator confidence

---

## Summary

This runbook exists to prove that the lab is **operated**, not just assembled.
