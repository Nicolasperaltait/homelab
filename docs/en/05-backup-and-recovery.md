# 05 — Backup and Recovery

## Philosophy

Backups are treated as an operational control, not a checkbox.

This document focuses on:

- what needs to be recoverable
- where backup layers exist
- how recovery paths differ
- why restore confidence matters more than archive count

---

## Backup layers

| Layer | Scope | Why it exists |
|---|---|---|
| VM / infrastructure | full VM state | broad recovery from host or VM failure |
| Application / service | app data and config | faster service-level recovery |
| Archive layer | compressed/exported copies | portable recovery points |
| Offsite encrypted copy | remote resilience | disaster tolerance |

---

## Public-safe backup flow

```mermaid
flowchart LR
    SRC[Service or Data Source] --> STG[Local Staging]
    STG --> ARC[Archive / Package]
    ARC --> NAS[Storage Repository]
    NAS --> OFF[Encrypted Offsite Copy]
```

---

## Why snapshots are not enough

Snapshots are useful, but they are not the same as backups.

| Snapshot | Backup |
|---|---|
| fast rollback | durable recovery artifact |
| tied to platform context | more portable |
| good for change windows | good for failure recovery |
| not a full resilience strategy | part of a resilience strategy |

---

## Recovery categories

### 1. Service-level recovery
Use when:

- the platform is healthy
- the application state is damaged
- a targeted restore is faster than rolling back the whole VM

### 2. VM-level recovery
Use when:

- the service host is damaged
- the operating system or VM state is compromised
- broad rollback is faster and safer

### 3. Offsite recovery
Use when:

- local storage is unavailable
- a major platform or site failure affects primary recovery paths

---

## Recovery confidence model

| Level | Meaning |
|---|---|
| low | artifacts exist, but restore has not been demonstrated |
| medium | backup workflow validated, limited restore confidence |
| high | restore has been tested and documented |

The most important maturity step is moving from **artifact presence** to **tested recovery**.

---

## Core backup design ideas

- separate backup scope by role
- keep archives understandable
- validate that output is readable
- protect critical recovery configuration
- distinguish convenience from recoverability

---

## What a professional backup story should include

- scope
- retention logic
- storage destination
- offsite strategy
- validation method
- restore method
- known recovery limits

This document intentionally emphasizes the last two.

---

## Known backup risks

| Risk | Why it matters |
|---|---|
| backup exists but restore is untested | false confidence |
| offsite path exists but key config is missing | recovery gap |
| retention is unclear | operator confusion during cleanup |
| snapshots are treated as backups | weak resilience model |

---

## Public-safe recovery scenarios

### Scenario A — Application corruption
Preferred response:
- preserve evidence if needed
- restore the service data set or archive
- validate application startup
- validate business function

### Scenario B — VM failure
Preferred response:
- restore VM-level backup or clean platform state
- revalidate dependencies
- revalidate service exposure and monitoring

### Scenario C — Local storage failure
Preferred response:
- use offsite encrypted recovery path
- restore in stages
- validate configuration dependencies first

---

## What remains the most important open item

A mature backup system is not defined by:

- the number of zip files
- the number of remotes
- the number of cron jobs

It is defined by:

> whether a restore can be performed cleanly, intentionally, and without improvisation

That is why restore testing remains the key maturity marker.

---

## Summary

The backup design is already meaningful.  
The next real milestone is turning it into a **measured recovery practice**.
