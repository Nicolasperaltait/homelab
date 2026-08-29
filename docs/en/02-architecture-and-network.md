# 02 — Architecture and Network

## Architecture goals

The architecture is designed around five priorities:

1. security
2. simplicity
3. observability
4. service isolation
5. maintainable growth

---

## High-level topology

```mermaid
flowchart LR
    U[Operations Workstation] --> D[Internal DNS]
    U --> H[Hypervisor / Gateway]
    U --> N[Storage Service]

    H --> A[Applications Zone]
    H --> S[Security Zone]
    H --> V[VPN Zone]

    A --> RP[Reverse Proxy]
    A --> AP[App Platform]
    A --> MO[Monitoring Stack]

    S --> SIEM[Security Monitoring]
    V --> WG[Remote Access Service]
    N --> BK[Backup Repository]
```

---

## Sanitized zone model

| Zone | Public description | Purpose |
|---|---|---|
| Management / LAN | management zone | local admin access, DNS, NAS, hypervisor management |
| Applications | application zone | internal applications and reverse proxy |
| Security | security zone | SIEM and security telemetry |
| ~~VPN~~ | ~~remote access zone~~ | **retired as a network zone.** See the model change in doc 03 |

---

## Logical service map

| Service role | Technology family (conceptual) | Zone | Purpose |
|---|---|---|---|
| Hypervisor | open-source hypervisor | Management | compute, gateway, segmentation control |
| Internal DNS | internal DNS resolver with filtering | Management | name resolution and DNS filtering |
| Storage / NAS | open-source NAS solution | Management | backup target and shared storage |
| App platform | container runtime on a Linux VM | Applications | self-hosted services |
| Reverse proxy | managed reverse proxy | Applications | controlled internal publication |
| Security monitoring | open-source SIEM platform | Security | event collection and visibility |
| Metrics and dashboards | metrics TSDB + dashboard layer | Applications | observability |
| Remote access | **overlay mesh with per-node identity** | Overlay | remote access without exposing inbound ports |

---

## Dependency model

```mermaid
flowchart TD
    P[Hypervisor] --> APPS[Applications Zone]
    P --> SEC[Security Zone]
    P --> VPN[VPN Zone]

    DNS[Internal DNS] --> APPS
    DNS --> SEC
    DNS --> VPN

    APPS --> PROXY[Reverse Proxy]
    APPS --> MON[Monitoring Stack]

    SEC --> WZ[Security Monitoring]

    NAS[Storage / Backup Target] --> BKP[Backup Archives]
    APPS --> NAS
```

---

## Connectivity philosophy

The environment does **not** assume free east-west traffic.  
Instead, the intended model is:

- allow the admin workstation to reach all required systems
- restrict lateral traffic unless justified
- document legitimate cross-zone flows
- prefer internal name resolution over ad-hoc access patterns
- avoid direct Internet exposure of admin interfaces

---

## Example legitimate cross-zone flows

| Source zone | Destination zone | Why it exists |
|---|---|---|
| Management | All zones | administration and validation |
| Applications | Security | service publication or telemetry workflows |
| Applications | Management / Storage | backup or file workflow integration |
| Remote access mesh | Management and selected services, **per advertised route** | secure remote administrative access with no inbound ports |

---

## Why the hypervisor matters so much

The virtualization host is not just “where VMs run.”

It is also:

- the segmentation anchor
- the gateway between internal zones
- a critical dependency for routing/NAT behavior
- a major recovery pivot

That makes it one of the most operationally sensitive parts of the environment.

---

## Why this architecture is still intentionally small

This is not a complexity contest.

The environment is intentionally limited so that each layer remains:

- understandable
- testable
- supportable by one operator
- explainable in public documentation

---

## Architectural strengths

- clean role separation
- low ambiguity in service purpose
- strong fit for public technical explanation
- practical rather than decorative complexity
- room to grow without redesigning from scratch

---

## Architectural risks still tracked

| Risk | Why it matters |
|---|---|
| DNS remains critical | many failures look like “service outage” when they are resolution problems |
| Hypervisor is a control-plane dependency | segmentation and VM availability depend on it |
| Restore confidence must be proven | backup presence is not the same as recovery readiness |
| Remote access design is constrained by upstream networking | VPN design quality depends on the external connectivity model |

---

## Summary

The architecture is strong because it is **coherent**.  
Every major service exists for a reason, belongs to a role, and supports a visible operational outcome.
