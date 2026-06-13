# 05 - Backup y Recuperacion

## Proposito

Describir la estrategia publica de backup y recuperacion del homelab.

## Principios

- recuperar vale mas que tener una copia
- snapshot y backup no son lo mismo
- offsite sin capacidad real de restore no alcanza
- la validacion periodica es parte del diseno
- una ventana de backup debe representar una respuesta operativa clara
- la configuracion necesaria para descifrar o restaurar tambien es critica

## Capas del modelo de backup

| Capa | Que cubre |
|---|---|
| Infraestructura | VMs, discos y rollback de plataforma |
| Aplicacion | datos criticos de servicios |
| Storage local | repositorio operativo de backup |
| Archivo empaquetado | consolidacion por dominio |
| Offsite cifrado | copia externa para escenarios de perdida mayor |
| Evidencia | logs, estado, alertas y validaciones |

## Flujo logico

```mermaid
flowchart LR
    A[Origen de datos] --> B[Backup local]
    B --> C[Storage]
    C --> D[Archivo empaquetado]
    D --> E[Offsite cifrado]
    D --> F[Validacion]
    F --> G[Evento / evidencia]
```

## Ventana operativa de backups

El diseno publico no documenta horarios reales. El criterio operativo es:

- ejecutar backups criticos en una ventana de baja actividad
- separar backups pequenos y pesados para mejorar visibilidad
- refrescar metricas despues de que la ventana deberia haber terminado
- responder a la pregunta: puedo operar hoy con confianza?

## Que se busca proteger

- estado de maquinas virtuales
- datos de servicios criticos
- configuraciones necesarias para recuperacion
- continuidad operativa minima
- capacidad de volver a un punto conocido bueno
- evidencia de que el backup no solo existio, sino que fue validado

## Distincion clave

| Concepto | Uso correcto |
|---|---|
| Snapshot | rollback rapido, cambios puntuales |
| Backup | recuperacion portable y mas robusta |
| Offsite | resiliencia ante perdida local |
| Restore test | evidencia de que la recuperacion es real |
| Alerta | senal accionable, no reemplazo de validacion |

## Offsite cifrado

La copia externa se trata como un control de resiliencia, no como almacenamiento cualquiera.

Principios publicos:

- el contenido offsite debe estar cifrado
- las claves/configuracion de recuperacion son material critico
- no se publican nombres de remotos, rutas reales ni configuraciones completas
- se valida presencia offsite, pero eso no reemplaza restore test

## Escenarios de recuperacion

### Escenario A - Falla de servicio

- preservar evidencia si corresponde
- validar dependencia de red, DNS y storage
- recuperar desde copia por dominio si corresponde
- confirmar que la aplicacion vuelve en estado sano

### Escenario B - Falla de VM

- evaluar rollback rapido
- restaurar VM desde backup cuando corresponda
- validar red, arranque y reachability

### Escenario C - Perdida parcial de storage

- aislar impacto
- recuperar desde archivo local o copia externa
- reconstruir el flujo operativo minimo

### Escenario D - Perdida amplia del entorno

- reinstalar plataforma base
- restaurar componentes prioritarios
- reconstruir conectividad y DNS
- recuperar servicios criticos segun prioridad

## Prioridad de recuperacion

| Prioridad | Componente |
|---|---|
| Alta | hypervisor, DNS interno, storage, plataforma de apps |
| Alta | datos y configuracion de servicios criticos |
| Media | observabilidad y dashboards |
| Variable | servicios auxiliares o de laboratorio |

## Riesgos todavia abiertos

| Riesgo | Estado |
|---|---|
| restore test formal | pendiente (ruta critica activa) |
| validacion automatica mas fuerte | en maduracion (evidencia por ciclo implementada) |
| redundancia fisica de storage | pendiente de evolucion |
| alertas y evidencia SIEM | en maduracion |
| dependencia de configuracion critica para offsite | controlada en documentacion privada |

## Indicadores operativos deseados

- backup reciente visible
- archivo reciente legible
- copia externa reciente
- logs sin error critico
- evento de estado entendible
- restore test periodico documentado

## Idea central

Este diseno se enfoca en el resultado que importa:

> conciencia de recuperacion y continuidad.
