# 07 - Decisiones Arquitectonicas

## Proposito

Este documento resume las decisiones principales detras del diseno publico del homelab. Esta pensado para revision tecnica y reutilizacion.

## Matriz de decisiones

| Decision | Razonamiento | Tradeoff | Evidencia publica |
|---|---|---|---|
| Segmentar por funcion | reduce ambiguedad y lateralidad innecesaria | requiere disenar rutas y accesos | docs de arquitectura y seguridad |
| Tratar hypervisor como control plane | computo, transito y recuperacion dependen de el | vuelve sensibles los cambios en hypervisor | arquitectura ejecutiva |
| Mantener privadas las superficies admin | reduce superficie de ataque | exige disciplina de acceso interno | modelo de seguridad |
| Centralizar DNS interno | mejora consistencia de acceso | crea dependencia que debe monitorearse | docs de arquitectura |
| Tratar hosts de contenedores distinto | firewall generico puede romper bridge/proxy | requiere hardening por rol | docs de seguridad |
| Separar dashboards de SIEM | metricas y evidencia de seguridad tienen trabajos distintos | hay dos caminos de visibilidad | docs de observabilidad |
| Tratar offsite como camino cifrado de recuperacion | protege datos fuera del storage local | la config de recuperacion se vuelve critica | docs de backup |
| Mantener docs publicas sanitizadas | claridad tecnica sin exponer el lab | menos detalle de implementacion publica | README y docs publicas |

## Que demuestra

Estas decisiones muestran pensamiento de arquitectura:

- definir limites antes de agregar herramientas
- explicar por que existen los flujos
- aceptar riesgos residuales explicitamente
- tratar documentacion como parte de la operacion
- usar evidencia privada sin publicar datos sensibles

## Como leerlo

El encuadre mas fuerte es:

> El lab es pequeno, pero esta disenado como plataforma: segmentado, observable, recuperable en diseno y documentado con tradeoffs explicitos.
