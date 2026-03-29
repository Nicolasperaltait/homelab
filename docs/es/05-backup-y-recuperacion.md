# 05 · Backup y Recuperación

## Propósito

Describir la estrategia pública de backup y recuperación del homelab.

## Principios

- recuperar vale más que “tener una copia”
- snapshot y backup no son lo mismo
- offsite sin capacidad real de restore no alcanza
- la validación periódica es parte del diseño

## Capas del modelo de backup

| Capa | Qué cubre |
|---|---|
| Infraestructura | VMs, discos y rollback de plataforma |
| Aplicación | datos críticos de servicios |
| Storage local | repositorio operativo de backup |
| Archivo empaquetado | consolidación por dominio |
| Offsite cifrado | copia externa para escenarios de pérdida mayor |

## Flujo lógico

```mermaid
flowchart LR
    A[Origen de datos] --> B[Backup local]
    B --> C[Storage]
    C --> D[Archivo empaquetado]
    D --> E[Offsite cifrado]
```

## Qué se busca proteger

- estado de máquinas virtuales
- datos de servicios críticos
- configuraciones necesarias para recuperación
- continuidad operativa mínima
- capacidad de volver a un punto conocido bueno

## Distinción clave

| Concepto | Uso correcto |
|---|---|
| Snapshot | rollback rápido, cambios puntuales |
| Backup | recuperación portable y más robusta |
| Offsite | resiliencia ante pérdida local |
| Restore test | evidencia de que la recuperación es real |

## Escenarios de recuperación

### Escenario A · Falla de servicio

- revisar logs
- validar datos
- recuperar desde copia por dominio si corresponde
- confirmar que la aplicación vuelve en estado sano

### Escenario B · Falla de VM

- evaluar rollback rápido
- restaurar VM desde backup cuando corresponda
- validar red, arranque y reachability

### Escenario C · Pérdida parcial de storage

- aislar impacto
- recuperar desde archivo local o copia externa
- reconstruir el flujo operativo mínimo

### Escenario D · Pérdida amplia del entorno

- reinstalar plataforma base
- restaurar componentes prioritarios
- reconstruir conectividad y DNS
- recuperar servicios críticos según prioridad

## Prioridad de recuperación

| Prioridad | Componente |
|---|---|
| Alta | hypervisor, DNS interno, storage, plataforma de apps |
| Media | observabilidad y dashboard |
| Variable | servicios auxiliares o de laboratorio |

## Riesgos todavía abiertos

| Riesgo | Estado |
|---|---|
| restore test formal | pendiente |
| cifrado local integral | parcialmente pendiente |
| redundancia física de storage | pendiente de evolución |
| validación automática más fuerte | mejora futura |

## Indicadores operativos deseados

- backup reciente visible
- archivo reciente legible
- copia externa reciente
- logs sin error crítico
- restore test periódico documentado

## Idea central

Este diseño no vende “backups bonitos”.  
Vende algo mejor:

> conciencia de recuperación y continuidad.
