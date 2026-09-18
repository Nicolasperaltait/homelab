# 07 - Architecture Decisions

> State described: September 2026.

## Purpose

Summarize the main decisions behind the design, with their reasoning, their cost
and where they are documented.

## Decision matrix

| Decision | Reasoning | Tradeoff | Where to see it |
|---|---|---|---|
| Segment by function on the hypervisor | reduces ambiguity and lateral movement | requires designing routes and access | 02 |
| Treat the hypervisor as control plane | compute, transit and recovery depend on it | its changes are sensitive | 00, 02 |
| Remote access through a mesh with no inbound ports | the edge stays closed | dependency on an external control plane | 03 |
| Per-host routes on the mesh | works from foreign networks with the same range | more entries to maintain | 03 |
| Mesh lock with signing nodes | a valid credential is not enough to join | adding a device takes an extra step | 03 |
| A single path for AI agents | turning off automation does not turn off the operator | the operator turns the host on by hand | 03, case 05 |
| Least privilege measured, not designed | an unused permission should not exist | revisit when usage changes | case 05 |
| Verify controls by making them fail | a verification can lie | longer test scripts | case 06 |
| Treat the container host differently | a generic firewall breaks container networking | role-based hardening | 03 |
| Automatic patching, security only, no reboot | security up to date without surprise outages | reboots pile up for a window | 03 |
| Alert only on failures | a noisy channel gets ignored | health is checked on the dashboard | 06 |
| Every automated task leaves a metric | a task that stops running is invisible | one more exporter per task | 06 |
| Backups measured by content | a recent archive does not prove new data | more specific metrics | 05, case 07 |
| Restore tests without credentials | validation does not become a secret to protect | it does not test a real login | 05 |
| Own code remote | history does not depend on an external service | it must be backed up and taken offsite | 05 |
| Delete only through quarantine and verified copy | a rushed deletion cannot be undone | space held for thirty days | 04, case 07 |
| Archive instead of deleting what is the last copy | history is not lost | inactive repositories stay visible | case 07 |
| Retire what is not used | less surface and less maintenance | decide it with usage data | 02 |
| Test the migration on a clean machine with the same accounts | privilege defects only show up that way | a longer test | case 07 |
| Sanitized public documentation | technical clarity without exposing the environment | less implementation detail | README |

## Reverted decisions

What was decided and later undone is also documented:

| Original decision | Why it was reverted |
|---|---|
| Own VPN tunnel with an inbound port | contradicted the closed-edge principle |
| Chat channel for alerts | never delivered a single notice |
| Dedicated dashboard console | messaging alerts made it unnecessary |
| Local AI services in the lab | not used; retired with their history archived |
| A self-hosted credentials service | not used; its retirement is in progress |

## How to read it

> The lab is small, but it is designed as a platform: segmented, with no inbound
> exposure, observable, recoverable with evidence, and documented with its
> tradeoffs, its mistakes and what is still missing.
