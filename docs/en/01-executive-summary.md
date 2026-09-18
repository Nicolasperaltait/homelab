# 01 - Executive Summary

> State described: September 2026.

## Goal

This document summarizes the homelab at an executive and technical level. The
public version shows architecture, operations, security and recovery judgment
without exposing data that would allow mapping or reproducing the real
environment.

## Overview

The homelab is a practice environment for infrastructure and cybersecurity that
also runs daily-use services: in-house applications, the code remote and the
workstation backups. The goal is not to accumulate services, but to demonstrate
the ability to:

- design a segmented architecture
- operate services with judgment and retire those that are not used
- reduce exposure: no inbound port at the edge
- observe the environment and get notified only when something fails
- back up, restore and **test** the restore
- work with AI agents under least privilege
- document decisions, incidents and limits, including one's own mistakes

## Public / private separation

| Layer | Purpose |
|---|---|
| Private documentation | real operations, evidence, incidents, paths, scripts and critical data |
| Public repository | sanitized version for technical explanation and architecture review |

Real operational information is not copied into the public repository. It is
first transformed into patterns, decisions and lessons with no identifying data.

## Main components

| Layer | Function |
|---|---|
| Hypervisor | virtualization, transit between segments, central control point |
| Internal DNS | centralized resolution and filtering, served to the whole network |
| Container platform | in-house applications, observability and reverse proxy |
| Code remote | own repositories with continuous integration, at home |
| Security | SIEM with agents on the hosts and on the workstation |
| Observability | metrics, dashboards and failure alerts to a messaging channel |
| Storage | NAS with backups, workstation mirror and encrypted offsite copy |
| Recovery | machine dedicated to test restores, isolated from production |
| Remote access | overlay mesh with per-node identity and a dedicated gateway |
| Automated access | jump host for AI agents, with a manual switch |

## Current state

### Working and verified

- logical segmentation by zone on the hypervisor
- remote access through the mesh, with no inbound ports and policy as code
- key-only SSH and system auditing on infrastructure hosts
- AI-agent access through a single jump host, with scoped privileged reading
- automatic security patching on Linux hosts and weekly container image updates
  with automatic rollback
- failure alerts to a messaging channel (late backups, exporters down, failing
  probes, unapplied patches)
- backup chain rebuilt with a content-age metric, not an archive-age one
- encrypted nightly backup of the workstation configuration, with an alert
- encrypted offsite copy, verified by metric
- own code remote with continuous integration
- two in-house web applications in production, deployed from commits
- workstation migration tested on a clean machine

### Open, with known risk

- the automatic restore test run is interrupted; resume it
- two machines that do not start on their own after a hypervisor reboot
- image-level backup for machines that today only back up data
- offsite copy of the code remote
- a second DNS resolver
- replacing a support disk that failed
- completing per-host network filtering
- built-in authentication for the web applications (today reachable only by tunnel)
- reducing SIEM noise and ingesting system auditing

## Canonical project state

| Aspect | State |
|---|---|
| Architecture | stable and documented |
| Access | no inbound exposure; automation separated from the operator |
| Observability | working; alerts only on failures |
| Backups | operational and measured by content, with offsite copy |
| Recovery | tested with measured RTO; periodic run to be resumed |
| Public documentation | sanitized and updated to September 2026 |

## The right reading

This homelab does not try to look enterprise through decoration. It tries to
show something more serious:

> a small infrastructure that is reasoned, operable, explainable, and honest
> about what is not solved yet.
