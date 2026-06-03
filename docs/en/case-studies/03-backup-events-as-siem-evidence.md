# Case Study 03 - Backup Events as SIEM Evidence

## Context

Backups are operational controls, but failures should also be visible as security and resilience events. The lab treats SIEM visibility as evidence, not just a dashboard.

## Symptom

Backup workflows could generate local status, but there was a gap between tactical failure detection and durable evidence for review.

## Decision

The design pattern became:

- generate structured backup status events
- send critical events into security monitoring
- separate immediate alerting from historical evidence
- avoid publishing real rules, payloads or paths
- use SIEM visibility to support audit-style review

## Validation

The public validation pattern is:

- critical backup failure creates an event
- security monitoring ingests the event
- severity and grouping are meaningful
- the event supports later review and reporting

## Lesson learned

SIEM value increases when it connects security monitoring with real operational risk, such as backup failure and recovery confidence.
