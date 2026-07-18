# Caso 04 - Migracion de Storage, Observabilidad y Reboot Controlado

## Contexto

Una plataforma de laboratorio productivo necesitaba liberar capacidad de storage
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
- validar observabilidad con metricas especificas, no con estados generales;
- ordenar el autostart de maquinas virtuales por dependencia;
- probar el comportamiento con DNS primario caido;
- confirmar reboot real con boot time y kernel activo.

## Validacion

El patron de validacion publico fue:

- confirmar que el nuevo storage queda montado en el path esperado;
- verificar que los servicios dependientes siguen activos;
- comprobar que dashboards y metricas consultan el recurso correcto;
- apagar temporalmente el DNS primario y validar fallback;
- reiniciar el hypervisor y verificar autostart;
- confirmar kernel activo, no solo que la sesion remota se haya cortado.

## Leccion aprendida

Un cambio de storage no termina cuando los archivos fueron copiados. Termina
cuando las automatizaciones, los dashboards, los servicios criticos, el rollback
y el arranque posterior quedaron validados con evidencia.

Tambien quedo una regla practica: una caida de SSH no prueba un reboot. El
reboot se confirma con boot time, uptime y version activa del kernel.

## Resultado

El entorno quedo operativo, con storage liberado de forma controlada,
observabilidad corregida y arranque automatico validado. El storage anterior no
se borro inmediatamente: se conservo como rollback hasta completar la ventana de
observacion.
