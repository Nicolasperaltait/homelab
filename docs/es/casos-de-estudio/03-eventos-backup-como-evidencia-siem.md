# Caso 03 - Eventos de Backup como Evidencia SIEM

## Contexto

Los backups son controles operativos, pero sus fallos tambien deben verse como eventos de seguridad y resiliencia. El lab trata la visibilidad SIEM como evidencia, no solo como dashboard.

## Sintoma

Los workflows de backup podian generar estado local, pero habia una brecha entre detectar una falla tactica y conservar evidencia durable para revision.

## Decision

El patron de diseno fue:

- generar eventos estructurados de estado de backup
- enviar eventos criticos al monitoreo de seguridad
- separar alerta inmediata de evidencia historica
- no publicar reglas, payloads ni rutas reales
- usar SIEM para apoyar revision tipo auditoria

## Validacion

El patron publico de validacion es:

- una falla critica de backup genera evento
- el monitoreo de seguridad ingiere el evento
- severidad y agrupacion son significativas
- el evento permite revision y reporting posterior

## Leccion aprendida

El valor de SIEM aumenta cuando conecta monitoreo de seguridad con riesgo operativo real, como falla de backup y confianza de recuperacion.
