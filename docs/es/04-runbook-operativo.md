# 04 · Runbook Operativo

## Propósito

Este runbook público resume cómo se opera el homelab sin exponer detalles sensibles.  
No reemplaza la documentación interna completa; funciona como versión presentable y defendible.

## Objetivos del runbook

- revisar el estado general
- diagnosticar incidentes sin improvisar
- validar capacidad operativa básica
- reducir dependencia de memoria informal
- ordenar la respuesta ante fallos

## Orden de diagnóstico

```mermaid
flowchart LR
    A[Energía / host] --> B[Red básica]
    B --> C[DNS interno]
    C --> D[Servicio específico]
    D --> E[Aplicación / datos]
```

## Checklist diario

| Control | Resultado esperado |
|---|---|
| Hypervisor accesible | operativo |
| DNS interno resolviendo | correcto |
| Plataforma de apps arriba | correcta |
| Storage accesible | correcto |
| SIEM / monitoreo respondiendo | correcto |
| Último backup visible | correcto |
| Última copia offsite reciente | correcto |

## Checklist semanal

| Control | Resultado esperado |
|---|---|
| espacio en storage | suficiente |
| crecimiento de backups | bajo control |
| limpieza operativa | ejecutada |
| ZIP reciente legible | válido |
| estado general de servicios | estable |
| revisión de pendientes críticos | actualizada |

## Escenarios operativos típicos

### 1. No responde un servicio web

Validar en este orden:

1. resolución de nombre
2. reachability por IP
3. estado de la VM o contenedor
4. proxy o publicación
5. logs del servicio

### 2. Falla DNS interno

Validar:

1. estado del servicio DNS
2. puertos de escucha
3. cliente apuntando al DNS correcto
4. resolución por nombre desde un origen confiable

### 3. Problema de salida o conectividad entre zonas

Validar:

1. gateway de la zona
2. forwarding
3. NAT
4. diferencia entre problema DNS y problema de tránsito

### 4. Storage lleno o backups fallando

Validar:

1. espacio libre
2. crecimiento por dominio
3. staging o residuos viejos
4. política de retención
5. integridad del último backup bueno

## Matriz rápida de decisión

| Síntoma | Primera sospecha |
|---|---|
| por IP funciona, por nombre no | DNS |
| varios servicios caídos a la vez | hypervisor o red |
| backup corre pero contenido inconsistente | pipeline o staging |
| acceso remoto parcial | VPN, rutas o NAT |
| servicio web no abre pero VM responde | proxy o aplicación |

## Controles mínimos por componente

| Componente | Qué validar |
|---|---|
| Hypervisor | VMs arriba, red, storage, tránsito |
| DNS interno | servicio, resolución, puertos |
| Plataforma de apps | contenedores, proxy, recursos |
| Storage | espacio, shares, directorios de backup |
| SIEM | servicios principales y reachability |
| VPN | interfaz, handshake, reachability esperada |

## Rollback y recuperación

Cuándo considerar rollback:

- el cambio reciente es claramente el origen del problema
- reparar “en caliente” aumenta riesgo
- existe snapshot o backup reciente confiable
- el costo de volver atrás es menor que seguir tocando

## Lecciones operativas incorporadas

- snapshot no reemplaza backup
- backup generado no equivale a backup confiable
- cron ejecutado no equivale a validación
- si no está validado, no existe
- el orden de diagnóstico importa

## Qué demuestra este runbook

Este documento muestra que el entorno no fue solo instalado.  
Fue pensado para ser **operado**.
