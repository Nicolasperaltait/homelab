# Sanitized Case Studies

These case studies translate private operational lessons into public, sanitized architecture material.

They intentionally omit real IPs, hostnames, paths, logs, commands and sensitive configurations.

## Cases

| Case | What it demonstrates |
|---|---|
| [01 - Storage pressure and backup recovery posture](01-storage-pressure-and-backup-recovery-posture.md) | recovery thinking, storage pressure, validation discipline |
| [02 - Monitoring path blocked by segmentation](02-monitoring-path-blocked-by-segmentation.md) | segmentation tradeoffs and justified cross-zone flows |
| [03 - Backup events as SIEM evidence](03-backup-events-as-siem-evidence.md) | security visibility as operational evidence |
| [04 - Storage migration, observability and controlled reboot](04-storage-migration-observability-and-controlled-reboot.md) | sensitive changes with temporary rollback, controlled retirement, observability and post-reboot validation |
| [05 - AI agent access under least privilege](05-ai-agent-access-under-least-privilege.md) | a single path with a physical off switch, scoped privileged reading, and permissions sized from measured use |
| [06 - When a control does not measure what it claims to](06-when-a-control-does-not-measure-what-it-claims.md) | seven verifications that lied, the shared pattern, and the practice of checking what must fail |
