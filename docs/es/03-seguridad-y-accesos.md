# 03 - Seguridad y Accesos

## Proposito

Documentar el enfoque de seguridad del homelab sin exponer detalles sensibles.

## Modelo de seguridad

El entorno sigue un modelo pequeno pero explicito:

- segmentacion por rol
- minimo privilegio
- reduccion de lateralidad
- administracion no expuesta publicamente
- acceso remoto bajo patron VPN
- excepciones documentadas cuando un flujo entre zonas es necesario
- separacion entre documentacion privada y documentacion publica

## Principios aplicados

| Principio | Aplicacion |
|---|---|
| Minimo privilegio | accesos administrativos controlados |
| Segmentacion | separacion de zonas con proposito |
| Seguridad por diseno | servicios criticos no expuestos |
| Excepciones documentadas | flujos permitidos solo por necesidad funcional |
| Hardening por rol | no todos los hosts reciben el mismo tratamiento |
| Sanitizacion publica | publicar intencion y criterio, no implementacion sensible |

## Que no se publica

- claves privadas
- secretos
- tokens
- credenciales
- endpoints reales de administracion
- configuracion completa de VPN
- rutas internas de backups o logs
- configuraciones completas de firewall, SIEM o monitoreo
- detalles sensibles del sistema de backup offsite

## Accesos

### Acceso administrativo

- acceso directo solo desde origen autorizado
- prioridad a SSH y paneles internos bajo control
- sin publicacion directa a Internet de interfaces administrativas

### Acceso de aplicaciones

- preferencia por nombres internos
- reverse proxy para servicios web seleccionados
- puertos directos solo cuando existe justificacion operativa

### Acceso remoto

- modelo basado en VPN
- publicacion externa solo bajo diseno controlado
- consideracion explicita de restricciones de conectividad y upstream networking

## Hardening por rol

Una leccion importante del proyecto es que un hardening generico no sirve para todos los hosts.

| Tipo de host | Enfoque recomendado |
|---|---|
| DNS interno | permitir puertos funcionales de resolucion y administracion minima |
| NAS / storage | proteger shares y paneles segun origen |
| SIEM | exponer solo puertos requeridos para dashboard y agentes |
| Host de contenedores | tratamiento especial por networking, bridge y proxy |
| Hypervisor | control fino de firewall, forwarding y transito |

## Flujos legitimos entre zonas

Hay casos donde una zona debe hablar con otra. Eso no contradice la segmentacion; la vuelve realista.

Criterio:

- permitir solo el flujo necesario
- documentar por que existe
- validar impacto operativo
- revisarlo periodicamente
- no publicar reglas reales ni origen/destino exactos

## Seguridad como evidencia operativa

El SIEM no se usa solo como herramienta de visualizacion. Su rol esperado es registrar eventos relevantes para operacion y seguridad, por ejemplo:

- fallos de backup
- cambios en componentes criticos
- agentes desconectados
- eventos de autenticacion relevantes
- alertas que requieren accion humana

En la version publica se describe el patron, no las reglas reales ni eventos crudos.

## Riesgos conocidos

| Riesgo | Mitigacion actual | Pendiente |
|---|---|---|
| Punto unico de falla DNS | DNS interno central | redundancia DNS |
| Acceso remoto limitado por conectividad | diseno VPN local | relay o mejora de salida |
| Recuperacion no completamente demostrada | backups y DRP documentados | restore test real |
| Alertas con demasiado ruido | criterio de alertas accionables | refinamiento continuo |
| Complejidad creciente | runbook y documentacion | revision continua de alcance |

## Threat model lite

```mermaid
flowchart TD
    A[Internet] -->|No acceso admin directo| B[Servicios expuestos por diseno]
    C[Cliente interno] --> D[DNS interno]
    C --> E[Aplicaciones internas]
    C --> F[Paneles administrativos autorizados]
    G[VPN] --> C
    H[Actor no autorizado] -.->|bloqueado o no publicado| F
```

## Idea central

La seguridad de este homelab no se apoya en una sola herramienta. Se apoya en una combinacion de:

- segmentacion
- publicacion controlada
- endurecimiento segun rol
- operacion disciplinada
- observabilidad de seguridad
- documentacion clara y segura
