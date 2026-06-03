# 07 - Architecture Decisions

## Purpose

This document summarizes the main architecture decisions behind the public homelab design. It is intended for technical review and reuse.

## Decision matrix

| Decision | Rationale | Tradeoff | Public evidence |
|---|---|---|---|
| Segment by function | reduces ambiguity and unnecessary lateral movement | more routing and access design to manage | architecture and security docs |
| Treat hypervisor as control plane | compute, transit and recovery depend on it | increases sensitivity of hypervisor changes | executive architecture |
| Keep admin surfaces private | reduces attack surface | requires internal access pattern discipline | security and access model |
| Use centralized internal DNS | improves consistency and service access | creates a dependency that must be monitored | architecture docs |
| Handle container hosts differently | generic firewalling can break bridge/proxy behavior | requires role-aware hardening | security docs |
| Separate dashboards from SIEM | metrics and security evidence have different jobs | two visibility paths to maintain | observability docs |
| Treat offsite as encrypted recovery path | protects data outside local storage | recovery config becomes critical | backup docs |
| Keep public docs sanitized | technical clarity without exposing the lab | less implementation detail in public | README and public docs |

## What this shows

These decisions show architecture thinking:

- defining boundaries before adding tools
- explaining why flows exist
- accepting residual risks explicitly
- treating documentation as part of operations
- using private evidence without publishing sensitive details

## How to read it

The strongest framing is:

> The lab is small, but it is designed as a platform: segmented, monitored, recoverable in design and documented with explicit tradeoffs.
