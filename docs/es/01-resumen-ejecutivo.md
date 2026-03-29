# 01 · Resumen Ejecutivo

## Objetivo

Este documento resume el homelab de forma ejecutiva y técnica.  
Está pensado para que cualquier persona pueda entender:

- qué problema resuelve
- cómo está organizado
- qué capacidades demuestra
- cuál es su estado actual

## Visión general

Este homelab fue diseñado como un entorno de práctica para infraestructura, seguridad y operación.  
La meta no es “tener muchos servicios”, sino demostrar capacidad para:

- diseñar una arquitectura segmentada
- operar servicios con criterio
- reducir exposición innecesaria
- centralizar resolución interna
- observar el entorno
- respaldar y recuperar componentes críticos
- documentar decisiones y límites

## Qué demuestra este proyecto

| Capacidad | Cómo se refleja en el repo |
|---|---|
| Arquitectura | Segmentación por zonas, dependencias claras, publicación controlada |
| Operación | Runbook, controles diarios/semanales, troubleshooting por escenario |
| Seguridad | Acceso por mínimo privilegio, servicios administrativos no expuestos, endurecimiento por rol |
| Observabilidad | Separación entre monitoreo de infraestructura y visibilidad de seguridad |
| Continuidad | Backups, snapshots, enfoque DRP, distinción entre copia y recuperación |
| Comunicación técnica | Documentación clara, sanitizada y defendible |

## Componentes principales

| Capa | Función |
|---|---|
| Hypervisor | Virtualización, tránsito entre segmentos, punto de control central |
| DNS interno | Resolución centralizada y consistencia de acceso |
| Plataforma de aplicaciones | Host para servicios internos |
| Seguridad | SIEM y telemetría de eventos |
| Monitoreo | Métricas, dashboards y visibilidad operativa |
| Storage | Repositorio de datos y nodo de backups |
| Acceso remoto | Diseño de acceso seguro bajo modelo VPN |

## Estado actual resumido

### Ya resuelto

- segmentación lógica por zonas
- DNS interno centralizado
- plataforma de aplicaciones operativa
- observabilidad base operativa
- almacenamiento y pipeline de backup documentados
- runbook maestro traducido a un formato público
- enfoque documental más maduro y entendible

### Pendientes de alto valor

- prueba real de restore
- alertas operativas más accionables
- redundancia DNS
- mejora del acceso remoto bajo restricciones de conectividad
- endurecimiento v2 por rol

## Topología lógica simplificada

```mermaid
flowchart LR
    A[Workstation de administración] --> B[DNS interno]
    A --> C[Hypervisor]
    C --> D[Zona de servicios]
    C --> E[Zona de seguridad]
    C --> F[Zona VPN]
    C --> G[Storage / NAS]

    D --> H[Aplicaciones internas]
    D --> I[Reverse proxy]
    D --> J[Monitoreo]

    E --> K[SIEM]
    F --> L[Acceso remoto seguro]
```

## Estado canónico del proyecto

| Aspecto | Estado |
|---|---|
| Arquitectura | Estable y documentada |
| Seguridad base | Aplicada con enfoque por rol |
| Observabilidad | Funcional |
| Backups | Operativos y documentados |
| Recuperación | Parcialmente demostrada, con restore test pendiente |
| Madurez documental | Alta para uso público |

## Lectura correcta del proyecto

Este homelab no busca parecer “enterprise” por decoración.  
Busca mostrar algo más serio:

> una infraestructura pequeña, razonada, operable y explicable.
