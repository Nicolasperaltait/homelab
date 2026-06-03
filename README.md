# Homelab Prod

![Public Docs](https://img.shields.io/badge/Public%20Docs-Yes-0A66C2?style=for-the-badge)
![Sanitized](https://img.shields.io/badge/Sanitized-Yes-2E8B57?style=for-the-badge)
![Bilingual](https://img.shields.io/badge/Bilingual-ES%20%7C%20EN-6A5ACD?style=for-the-badge)
![Infrastructure](https://img.shields.io/badge/Focus-Infrastructure-444444?style=for-the-badge)
![Security](https://img.shields.io/badge/Focus-Security-B22222?style=for-the-badge)

> A sanitized infrastructure and security architecture record documenting segmentation, operations, observability, backup strategy and a risk-driven roadmap.

---

## What this documents

This is not a tool list. It documents how a small infrastructure environment is reasoned, operated and explained without exposing sensitive implementation details:

- designed a segmented internal platform with explicit trust boundaries
- defined security boundaries and a controlled access model
- built repeatable operations and recovery practices
- separated private operational evidence from public documentation
- documented architectural decisions, tradeoffs, residual risks and roadmap
- translated real operational lessons into sanitized case studies

The real environment is documented privately. This repository publishes the architecture reasoning, operational model and security posture at a sanitized level.

---

## Architecture story

The project is built around a small but realistic infrastructure model:

- a hypervisor as the compute, transit and control-plane anchor
- internal DNS as a critical dependency for service access
- separated service, security, storage and remote-access concerns
- observability designed to answer operational questions
- SIEM treated as evidence and security visibility, not just another dashboard
- backup strategy focused on recoverability rather than archive count

```mermaid
flowchart LR
    OP[Operator] --> MGMT[Management Access]
    MGMT --> HV[Hypervisor / Control Plane]
    HV --> APP[Services Zone]
    HV --> SEC[Security Zone]
    HV --> VPN[Remote Access Zone]
    HV --> STO[Storage / Backup Zone]

    APP --> MON[Operational Dashboards]
    SEC --> SIEM[Security Monitoring]
    STO --> DR[Backup / Recovery Practice]
```

---

## Reading map

### English
- [00 - Executive architecture](docs/en/00-executive-architecture.md)
- [01 - Executive summary](docs/en/01-executive-summary.md)
- [02 - Architecture and network](docs/en/02-architecture-and-network.md)
- [03 - Security and access](docs/en/03-security-and-access.md)
- [04 - Operations runbook](docs/en/04-operations-runbook.md)
- [05 - Backup and recovery](docs/en/05-backup-and-recovery.md)
- [06 - Observability and roadmap](docs/en/06-observability-and-roadmap.md)
- [07 - Architecture decisions](docs/en/07-architecture-decisions.md)
- [Case studies](docs/en/case-studies/README.md)

### Espanol
- [00 - Arquitectura ejecutiva](docs/es/00-arquitectura-ejecutiva.md)
- [01 - Resumen ejecutivo](docs/es/01-resumen-ejecutivo.md)
- [02 - Arquitectura y red](docs/es/02-arquitectura-y-red.md)
- [03 - Seguridad y accesos](docs/es/03-seguridad-y-accesos.md)
- [04 - Runbook operativo](docs/es/04-runbook-operativo.md)
- [05 - Backup y recuperacion](docs/es/05-backup-y-recuperacion.md)
- [06 - Observabilidad y roadmap](docs/es/06-observabilidad-y-roadmap.md)
- [07 - Decisiones arquitectonicas](docs/es/07-decisiones-arquitectonicas.md)
- [Casos de estudio](docs/es/casos-de-estudio/README.md)

---

## What is intentionally private

This repository does not publish:

- real IP addresses, hostnames or users
- credentials, tokens, keys or webhooks
- complete firewall, VPN, SIEM or backup configurations
- raw logs, screenshots or alert payloads
- exact rollback paths, scripts or operational evidence
- private incident records

That omission is part of the security model, not a documentation gap.

---

## Repository structure

```text
homelab/
├── README.md
├── LICENSE.md
└── docs/
    ├── en/
    │   ├── 00-executive-architecture.md
    │   ├── 01-executive-summary.md
    │   ├── 02-architecture-and-network.md
    │   ├── 03-security-and-access.md
    │   ├── 04-operations-runbook.md
    │   ├── 05-backup-and-recovery.md
    │   ├── 06-observability-and-roadmap.md
    │   ├── 07-architecture-decisions.md
    │   └── case-studies/
    └── es/
        ├── 00-arquitectura-ejecutiva.md
        ├── 01-resumen-ejecutivo.md
        ├── 02-arquitectura-y-red.md
        ├── 03-seguridad-y-accesos.md
        ├── 04-runbook-operativo.md
        ├── 05-backup-y-recuperacion.md
        ├── 06-observabilidad-y-roadmap.md
        ├── 07-decisiones-arquitectonicas.md
        └── casos-de-estudio/
```
