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


---

## Acceso de automatizacion y de agentes de IA

Las cuentas que operan de forma automatizada -incluido un asistente de IA- **no
comparten el modelo de acceso de la persona**. El criterio es que apagar el
acceso automatizado no debe apagar el del operador, y viceversa.

| Propiedad | Como se resuelve |
|---|---|
| Un solo camino | Todo el acceso automatizado pasa por un host de salto dedicado. No hay acceso directo a los destinos |
| Interruptor fisico | Ese host **no arranca solo**. Lo enciende el operador desde el hipervisor, fuera del alcance del agente |
| Credenciales confinadas | Las credenciales hacia el resto viven **solo dentro** del host de salto |
| Restriccion por origen | Aun filtrada, una credencial no sirve desde otro lugar |
| Sin reenvio | El host de salto no permite reenvio de puertos ni de agente: con reenvio, la credencial volveria a la estacion del operador |
| Trazabilidad por cuenta | Los eventos salen al SIEM en el momento, distinguiendo que cuenta hizo cada cosa |
| Sin via de emergencia | Decision explicita. Si el host de salto esta apagado, se pide encenderlo |

### Lectura privilegiada sin escritura

La mayor parte del trabajo util de una cuenta de automatizacion es **leer**. Un
permiso acotado a comandos concretos genera friccion constante por operaciones
inofensivas, y la salida facil -autorizar un lector generico con privilegio- es
**equivalente a dar acceso total**, porque permite leer el archivo de
contrasenas.

Se resuelve con un envoltorio propio de solo lectura, con lista de exclusion para
el material critico, que **normaliza rutas** antes de decidir e **inspecciona
contenido** en vez de confiar en el nombre del archivo.

### Los permisos se dimensionan midiendo

La lista de comandos privilegiados permitidos **no se disena en abstracto**: sale
de leer que se invoco realmente. En la revision de 2026 ese ejercicio mostro que
un permiso amplio concedido "por las dudas" **nunca habia hecho falta**: casi
todo el uso era lectura, parte no necesitaba privilegio, y quedaban dos acciones
concretas.

**Una regla que el propio agente cumple voluntariamente no es un control.** Si un
permiso no se usa, no tiene por que existir.

Detalle en [Caso 05](casos-de-estudio/05-acceso-de-agentes-de-ia-y-minimo-privilegio.md).

---

## Verificacion de controles

Un control que no se prueba es una suposicion documentada.

La practica de este entorno es que **todo script de seguridad comprueba lo que
tiene que FALLAR**, no solo lo que tiene que funcionar:

- el envoltorio de lectura verifica que **deniega** el archivo de contrasenas,
  una clave privada y una ruta que intente evadirlo;
- la reduccion de privilegios verifica que un lector generico y un interprete de
  comandos **quedan denegados**;
- la rotacion de credenciales verifica que la credencial **vieja deja de
  funcionar**, porque crear una nueva no es rotar;
- el endurecimiento del acceso remoto consulta la **configuracion efectiva** del
  servicio, no el archivo que se escribio.

El motivo es empirico: en una sola semana de endurecimiento, **siete
verificaciones informaron un resultado que no correspondia con la realidad**.
Ninguna fallo por un error de implementacion; todas verificaban la accion en vez
del efecto.

Detalle en [Caso 06](casos-de-estudio/06-cuando-un-control-no-mide-lo-que-dice-medir.md).

### Retiro de credenciales

Orden innegociable, adoptado despues de perder acceso a un host dos veces:

1. montar el reemplazo;
2. **probarlo desde donde se va a usar**;
3. recien entonces retirar el anterior.

Los scripts que retiran un acceso **se niegan a ejecutarse** si el reemplazo no
esta instalado y verificado.
