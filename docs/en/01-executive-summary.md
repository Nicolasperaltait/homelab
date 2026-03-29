# 01 — Executive Summary

## Purpose

This document explains the homelab at the highest useful level:

- what it is
- what problems it is meant to solve
- what technical skills it demonstrates
- what its current maturity level is

---

## What this homelab is

This is a **segmented infrastructure and security lab** built to simulate a small but realistic environment for:

- infrastructure administration
- network segmentation
- service isolation
- security monitoring
- observability
- backup and recovery
- disciplined operations

It is not meant to imitate enterprise complexity for its own sake.  
It is meant to demonstrate **good technical judgment**.

---

## Core design idea

The lab is built around a virtualization host that acts as:

- compute platform
- internal gateway
- segmentation control point

Around that core, services are separated by role:

| Zone | Purpose | Typical workloads |
|---|---|---|
| Management / LAN | administration and local access | workstation, DNS, NAS, hypervisor access |
| Applications | internal services platform | containers, reverse proxy, dashboards |
| Security | detection and visibility | SIEM, agents, security dashboards |
| VPN | remote entry model | remote access service |

---

## What the project demonstrates

### Infrastructure
- virtualization design
- host and service dependency mapping
- controlled service publication
- storage-aware operations

### Networking
- logical segmentation
- gateway-based inter-zone control
- internal DNS strategy
- role-based connectivity model

### Security
- least privilege thinking
- host-role-aware hardening
- centralized security monitoring
- sanitization discipline for public documentation

### Operations
- repeatable runbook structure
- incident-first troubleshooting order
- baseline-driven rollback thinking
- backup validation instead of blind trust

### Recovery
- distinction between snapshots and backups
- application-level vs VM-level recovery
- encrypted offsite strategy
- restore maturity tracking

---

## Current maturity assessment

| Area | Current posture |
|---|---|
| Architecture | stable and explainable |
| Segmentation | implemented |
| Security baseline | functional, with role-based refinement ongoing |
| Monitoring | operational |
| Backups | operational and validated at workflow level |
| Restore testing | still the most important open item |
| Documentation | now intentionally simplified for public use |

---

## What changed in the public version

The original public repository structure was broad and technically correct, but it was harder to explain quickly.

This version improves that by making the repo:

- easier to navigate
- easier to defend in an interview
- easier to maintain over time
- easier to keep sanitized

---

## One-paragraph project pitch

> This homelab is a hypervisor-based segmented environment designed to practice infrastructure architecture, security operations, observability, backup discipline, and technical troubleshooting. The public repo is sanitized, but it keeps the actual design logic, operating model, recovery strategy, and decision-making process visible.

---

## Suggested interview narrative

Use this sequence:

1. Show the executive summary.
2. Show the architecture diagram.
3. Show the runbook.
4. Show the backup and recovery model.
5. Close with roadmap and open risks.

That sequence tells a coherent story:

**design → controls → operations → recovery → next steps**
