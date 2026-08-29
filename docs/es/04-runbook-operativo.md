# 04 - Runbook Operativo

## Proposito

Este runbook publico resume como se opera el homelab sin exponer detalles sensibles. No reemplaza la documentacion interna completa; funciona como version presentable y defendible.

## Objetivos del runbook

- revisar el estado general
- diagnosticar incidentes sin improvisar
- validar capacidad operativa basica
- reducir dependencia de memoria informal
- ordenar la respuesta ante fallos
- mantener separada la operacion real de la version publica

## Orden de diagnostico

```mermaid
flowchart LR
    A[Energia / host] --> B[Red basica]
    B --> C[DNS interno]
    C --> D[Servicio especifico]
    D --> E[Aplicacion / datos]
    E --> F[Backup / recuperacion si aplica]
```

## Checklist diario

| Control | Resultado esperado |
|---|---|
| Hypervisor accesible | operativo |
| DNS interno resolviendo | correcto |
| Plataforma de apps arriba | correcta |
| Storage accesible | correcto |
| SIEM / monitoreo respondiendo | correcto |
| Estado del ultimo backup | validado en evidencia privada |
| Estado de copia offsite | monitoreado cuando esta habilitada |
| Alertas criticas | revisadas |

## Checklist semanal

| Control | Resultado esperado |
|---|---|
| espacio en storage | suficiente |
| crecimiento de backups | bajo control |
| limpieza operativa | ejecutada |
| archivo reciente legible | valido |
| estado general de servicios | estable |
| revision de pendientes criticos | actualizada |
| documentacion privada | evidencia actualizada |
| documentacion publica | sin datos sensibles |

## Escenarios operativos tipicos

### 1. No responde un servicio web

Validar en este orden:

1. resolucion de nombre
2. reachability por red
3. estado de la VM o contenedor
4. proxy o publicacion
5. logs del servicio
6. dependencia de storage o DNS

### 2. Falla DNS interno

Validar:

1. estado del servicio DNS
2. puertos de escucha
3. cliente apuntando al DNS correcto
4. resolucion por nombre desde un origen confiable
5. impacto sobre servicios dependientes

### 3. Problema de salida o conectividad entre zonas

Validar:

1. gateway de la zona
2. forwarding
3. NAT
4. reglas permitidas entre zonas
5. diferencia entre problema DNS y problema de transito

### 4. Storage lleno o backups fallando

Validar:

1. espacio libre
2. crecimiento por dominio
3. staging o residuos viejos
4. politica de retencion
5. integridad del ultimo backup bueno
6. estado de copia offsite

### 5. Dashboard o monitoreo degradado

Validar:

1. fuente de metricas
2. collector/exporter
3. scrape o ingesta
4. dashboard y query
5. si el dato representa operacion real o solo ausencia de metrica

## Matriz rapida de decision

| Sintoma | Primera sospecha |
|---|---|
| por red funciona, por nombre no | DNS |
| varios servicios caidos a la vez | hypervisor o red |
| backup corre pero contenido inconsistente | pipeline, staging o validacion |
| acceso remoto parcial | rutas anunciadas, politica de la malla o NAT |
| servicio web no abre pero VM responde | proxy o aplicacion |
| dashboard en rojo con servicio sano | metrica, exporter o query |

## Controles minimos por componente

| Componente | Que validar |
|---|---|
| Hypervisor | VMs arriba, red, storage, transito |
| DNS interno | servicio, resolucion, puertos |
| Plataforma de apps | contenedores, proxy, recursos |
| Storage | espacio, shares, directorios de backup |
| SIEM | servicios principales, agentes e ingesta |
| Acceso remoto | estado del nodo, **rutas anunciadas y aprobadas**, y alcance real al destino esperado |
| Dashboards | datos reales, frescura y utilidad |

## Rollback y recuperacion

Cuando considerar rollback:

- el cambio reciente es claramente el origen del problema
- reparar en caliente aumenta riesgo
- existe snapshot o backup reciente confiable
- el costo de volver atras es menor que seguir tocando

## Lecciones operativas incorporadas

- snapshot no reemplaza backup
- backup generado no equivale a backup confiable
- cron ejecutado no equivale a validacion
- dashboard verde no equivale a recuperabilidad
- si no esta validado, no existe
- el orden de diagnostico importa

## Que demuestra este runbook

Este documento muestra que el entorno no fue solo instalado. Fue pensado para ser operado, observado y recuperado con criterio.
