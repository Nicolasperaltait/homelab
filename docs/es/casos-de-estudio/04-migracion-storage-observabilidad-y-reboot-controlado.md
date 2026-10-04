# Caso 04 - Migracion de Storage, Observabilidad y Reboot Controlado

## Contexto

Una infraestructura productiva necesitaba liberar capacidad de storage
sin romper automatizaciones, backups ni observabilidad. El entorno tenia una
dependencia fuerte entre storage, DNS, dashboards, servicios de seguridad y
arranque automatico de maquinas virtuales.

## Problema

El riesgo no era solamente mover datos. El riesgo real era perder continuidad
operativa por alguno de estos puntos:

- rutas estables usadas por automatizaciones;
- dashboards apuntando al filesystem equivocado;
- servicios de seguridad con metricas desactualizadas;
- maquinas virtuales sin autostart correcto;
- dependencia no probada del DNS primario;
- reboot de hypervisor sin evidencia de arranque correcto.

## Decision

La migracion se trato como un cambio operacional controlado:

- preservar el path logico usado por los procesos;
- crear rollback antes de retirar el storage anterior;
- retirar el rollback local solo despues de una confirmacion separada y pruebas
  funcionales;
- reutilizar el disco liberado con un rol claro, separando discos de maquinas
  virtuales, backups, ISOs y storage de NAS;
- validar observabilidad con metricas especificas, no con estados generales;
- ordenar el autostart de maquinas virtuales por dependencia;
- probar el comportamiento con DNS primario caido;
- confirmar reboot real con boot time y kernel activo.

## Validacion

El patron de validacion publico fue:

- confirmar que el nuevo storage queda montado en el path esperado;
- verificar que los servicios dependientes siguen activos;
- comprobar que dashboards y metricas consultan el recurso correcto;
- ejecutar anticipadamente automatizaciones criticas y validar artefactos
  restaurables;
- comprobar que no quedan discos activos ni snapshots reteniendo el storage
  anterior;
- revisar discos base, cloud-init y snapshots antes de borrar archivos que
  parecen duplicados;
- verificar uso final por storage y confirmar que las maquinas siguen en el
  estado esperado;
- apagar temporalmente el DNS primario y validar fallback;
- reiniciar el hypervisor y verificar autostart;
- confirmar kernel activo, no solo que la sesion remota se haya cortado.

## Leccion aprendida

Un cambio de storage no termina cuando los archivos fueron copiados. Termina
cuando las automatizaciones, los dashboards, los servicios criticos, el rollback
y el arranque posterior quedaron validados con evidencia.

Tambien quedo una regla practica: una caida de SSH no prueba un reboot. El
reboot se confirma con boot time, uptime y version activa del kernel.

Otra leccion fue no asumir que dos archivos de disco representan duplicados. Un
disco base puede ser un resto eliminable solo si no hay referencias ni backing
files; un disco Cloud-Init pequeno puede estar activamente referenciado y debe
conservarse.

## Resultado

El entorno quedo operativo, con storage liberado de forma controlada,
observabilidad corregida y arranque automatico validado. El storage anterior se
conservo primero como rollback, luego se retiro definitivamente tras validacion
funcional y confirmacion separada.

Despues, el disco liberado se formateo y se reutilizo como storage principal de
maquinas virtuales. El storage local quedo reservado para componentes minimos y
criticos, mientras que backups, imagenes de instalacion y storage de NAS quedaron
separados en un storage de soporte.
