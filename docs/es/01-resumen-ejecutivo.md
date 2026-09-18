# 01 - Resumen Ejecutivo

> Estado descrito: septiembre de 2026.

## Objetivo

Este documento resume el homelab de forma ejecutiva y tecnica. La version publica
muestra criterio de arquitectura, operacion, seguridad y recuperacion sin exponer
datos que permitan mapear o reproducir el entorno real.

## Vision general

El homelab es un entorno de practica para infraestructura y ciberseguridad que
ademas corre servicios de uso diario: aplicaciones propias, el remoto de codigo y
los respaldos de la estacion de trabajo. La meta no es acumular servicios, sino
demostrar capacidad para:

- disenar una arquitectura segmentada
- operar servicios con criterio y dar de baja los que no se usan
- reducir exposicion: ningun puerto entrante en el borde
- observar el entorno y recibir aviso solo cuando algo falla
- respaldar, restaurar y **probar** la restauracion
- trabajar con agentes de IA bajo minimo privilegio
- documentar decisiones, incidentes y limites, incluidos los errores propios

## Separacion publico / privado

| Capa | Proposito |
|---|---|
| Documentacion privada | operacion real, evidencias, incidentes, rutas, scripts y datos criticos |
| Repositorio publico | version sanitizada para explicacion tecnica y revision de arquitectura |

La informacion operativa real no se copia al repositorio publico. Primero se
transforma en patrones, decisiones y aprendizajes sin datos identificables.

## Componentes principales

| Capa | Funcion |
|---|---|
| Hipervisor | virtualizacion, transito entre segmentos, punto de control central |
| DNS interno | resolucion centralizada y filtrado, servido a toda la red |
| Plataforma de contenedores | aplicaciones propias, observabilidad y proxy inverso |
| Remoto de codigo | repositorios propios con integracion continua, dentro de casa |
| Seguridad | SIEM con agentes en los hosts y en la estacion de trabajo |
| Observabilidad | metricas, dashboards y alertas de fallo a un canal de mensajeria |
| Storage | NAS con respaldos, espejo de la estacion de trabajo y copia cifrada fuera del sitio |
| Recuperacion | maquina dedicada a probar restauraciones, aislada de produccion |
| Acceso remoto | malla superpuesta con identidad por nodo y una puerta dedicada |
| Acceso automatizado | host de salto para agentes de IA, con interruptor manual |

## Estado actual

### Funcionando y verificado

- segmentacion logica por zonas en el hipervisor
- acceso remoto por malla, sin puertos entrantes, con politica como codigo
- autenticacion SSH solo por clave y auditoria del sistema en los hosts de infraestructura
- acceso de agentes de IA por un unico host de salto, con lectura privilegiada acotada
- parcheo de seguridad automatico en los hosts Linux y actualizacion semanal de
  imagenes de contenedores con reversion automatica
- alertas de fallo a un canal de mensajeria (backups atrasados, exportadores
  caidos, pruebas que fallan, parches sin aplicar)
- cadena de backup reconstruida con metrica de edad del contenido, no del archivo
- respaldo nocturno cifrado de la configuracion de la estacion de trabajo, con alerta
- copia cifrada fuera del sitio, verificada por metrica
- remoto de codigo propio con integracion continua
- dos aplicaciones web propias en produccion, desplegadas desde commits
- migracion de la estacion de trabajo probada en una maquina limpia

### Abierto, con riesgo conocido

- corrida automatica de las pruebas de restauracion interrumpida; reanudarla
- dos maquinas que no arrancan solas tras un reinicio del hipervisor
- respaldo a nivel imagen para las maquinas que hoy solo respaldan datos
- copia fuera de casa del remoto de codigo
- segundo resolver DNS
- reemplazo de un disco de soporte que fallo
- completar el filtrado de red por host
- autenticacion propia en las aplicaciones web (hoy solo son alcanzables por tunel)
- reducir el ruido del SIEM e ingerir la auditoria del sistema

## Estado canonico del proyecto

| Aspecto | Estado |
|---|---|
| Arquitectura | estable y documentada |
| Acceso | sin exposicion entrante; automatizacion separada del operador |
| Observabilidad | funcional; alerta solo fallos |
| Backups | operativos y medidos por contenido, con copia fuera del sitio |
| Recuperacion | probada con RTO medido; corrida periodica a reanudar |
| Documentacion publica | sanitizada y actualizada a septiembre de 2026 |

## Lectura correcta

Este homelab no busca parecer enterprise por decoracion. Busca mostrar algo mas
serio:

> una infraestructura pequena, razonada, operable, explicable, y honesta sobre
> lo que todavia no esta resuelto.
