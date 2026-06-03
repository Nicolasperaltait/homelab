# 05 - Backup and Recovery

## Purpose

Describe the public backup and recovery strategy for the homelab.

## Principles

- recovery matters more than having a copy
- snapshots and backups are not the same thing
- offsite copy without restore capability is not enough
- periodic validation is part of the design
- a backup window should answer a clear operational question
- configuration needed to decrypt or restore is also critical

## Backup model layers

| Layer | Scope |
|---|---|
| Infrastructure | VMs, disks and platform rollback |
| Application | critical service data |
| Local storage | operational backup repository |
| Archive/package | domain-level consolidation |
| Encrypted offsite | external copy for major-loss scenarios |
| Evidence | logs, status, alerts and validation |

## Logical flow

```mermaid
flowchart LR
    A[Data source] --> B[Local backup]
    B --> C[Storage]
    C --> D[Archive package]
    D --> E[Encrypted offsite]
    D --> F[Validation]
    F --> G[Event / evidence]
```

## Operational backup window

The public design does not publish real schedules. The operational criteria are:

- run critical backups during a low-activity window
- separate small and heavy backups to improve visibility
- refresh metrics after the window should be complete
- answer the question: can I operate today with confidence?

## What should be protected

- VM state
- critical service data
- configuration required for recovery
- minimum operational continuity
- ability to return to a known good state
- evidence that a backup was not only created, but validated

## Key distinction

| Concept | Correct use |
|---|---|
| Snapshot | fast rollback for specific changes |
| Backup | more portable recovery artifact |
| Offsite | resilience against local loss |
| Restore test | evidence that recovery is real |
| Alert | actionable signal, not a substitute for validation |

## Encrypted offsite copy

The external copy is treated as a resilience control, not just storage.

Public principles:

- offsite content should be encrypted
- recovery keys/configuration are critical material
- remote names, real paths and full configs are not published
- offsite presence is validated, but it does not replace restore testing

## Recovery scenarios

### Scenario A - Service failure

- preserve evidence if needed
- validate network, DNS and storage dependencies
- restore from domain-level copy if appropriate
- confirm that the application returns healthy

### Scenario B - VM failure

- evaluate fast rollback
- restore VM backup where appropriate
- validate network, boot and reachability

### Scenario C - Partial storage loss

- isolate impact
- recover from local archive or external copy
- rebuild the minimum operational flow

### Scenario D - Broad environment loss

- reinstall base platform
- restore priority components
- rebuild connectivity and DNS
- recover critical services by priority

## Recovery priority

| Priority | Component |
|---|---|
| High | hypervisor, internal DNS, storage, application platform |
| High | critical service data and configuration |
| Medium | observability and dashboards |
| Variable | auxiliary or lab services |

## Open risks

| Risk | State |
|---|---|
| formal restore test | pending |
| stronger automated validation | future improvement |
| physical storage redundancy | evolution pending |
| SIEM alerting and evidence | maturing |
| dependency on critical offsite configuration | controlled in private documentation |

## Desired operational indicators

- recent backup visible
- recent archive readable
- recent external copy
- logs without critical error
- understandable status event
- periodic restore test documented

## Core idea

This design focuses on the outcome that matters:

> recovery and continuity awareness.
