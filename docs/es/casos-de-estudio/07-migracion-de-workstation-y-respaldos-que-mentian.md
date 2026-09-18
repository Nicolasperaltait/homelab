# Caso 07 - Migracion de la Workstation y Respaldos que Mentian

## Contexto

La estacion de trabajo principal del laboratorio necesitaba poder formatearse.
Nadie se animaba, porque nadie sabia que se perderia.

La pregunta "que se pierde si formateo" se contesto midiendo, no recordando. El
resultado fue peor de lo esperado:

- **la mayoria de los repositorios de trabajo no tenia ningun remoto.** El
  historial de meses vivia en un solo disco;
- **el respaldo que debia cubrirlo llevaba casi cuatro meses congelado**, y el
  monitoreo lo reportaba como sano todos los dias.

El objetivo quedo definido asi: que formatear no pierda nada, y que rearmar el
equipo se haga siguiendo un documento, sin depender de la memoria de nadie.

## Hallazgo 1 - El respaldo que media la edad equivocada

La cadena de respaldo tenia cinco eslabones. El primero, una tarea programada en
la estacion de trabajo, apuntaba a un script que ya no existia. Fallaba todos los
dias, sin avisar a nadie.

Los eslabones siguientes seguian corriendo: el almacenamiento comprimia cada
madrugada el espejo congelado, el hash verificaba y la metrica informaba un
respaldo de pocas horas.

**La metrica media la edad del archivo comprimido, no la de su contenido.**
Estaba comprobando que el ultimo paso de la cadena habia corrido, no que el
respaldo tuviera datos nuevos.

Correccion: la cadena se reemplazo por una sincronizacion directa, y se agrego una
unica metrica nueva, **la edad del archivo mas reciente dentro del respaldo**, con
alerta si se atrasa.

## Hallazgo 2 - El envio fuera de casa que fallaba en silencio

La copia fuera del sitio fallo varios dias seguidos sin generar una alerta.

El script usaba el modo estricto de la shell y una trampa para errores. Pero
**esa trampa no se dispara cuando el propio script termina con un codigo de
salida explicito**, y justamente asi abortaba cuando faltaba una carpeta de
origen. El script informaba fallo al sistema, pero la ruta que avisaba nunca
se ejecutaba.

Correccion: la notificacion se engancha a la **salida del proceso**, que corre
siempre, y decide segun el codigo de salida. La carpeta opcional dejo de ser
motivo para abortar todo el envio.

## Hallazgo 3 - El disco lleno que no se vaciaba al borrar

El almacenamiento del hipervisor llego al cien por ciento. Borrar archivos
dentro de la maquina virtual no liberaba nada.

La causa era una **instantanea olvidada**. Mientras existe una instantanea, el
disco virtual conserva los bloques viejos: lo que se borra adentro sigue ocupando
lugar afuera.

Correccion: retirar las instantaneas vencidas y los respaldos antiguos, y recien
despues propagar el descarte de bloques desde el invitado. Las instantaneas pasan
a tener fecha de retiro desde el momento en que se crean.

## La migracion en fases

| Fase | Que hizo | Criterio de cierre |
|---|---|---|
| 0 | Salvar lo que no tenia copia, con el material criptografico cifrado aparte | cada archivo **abrio**, no solo existia |
| 1 | Limpiar, moviendo a cuarentena en vez de borrar | espacio medido y cero tareas apuntando a rutas inexistentes |
| 2 | Remoto de git propio, autoalojado | clonar desde otra maquina y comparar el historial |
| 3 | Respaldo simple con metrica de contenido | la metrica confirma sola que el respaldo es de hoy |
| 4 | Empaquetar las aplicaciones | desplegar sin tocar nada a mano |
| 5 | Prueba real en una maquina limpia | levantarla leyendo solo el documento |

Dos reglas cruzaron todas las fases:

- **nada se borra directo.** Todo pasa por una cuarentena de treinta dias con un
  manifiesto de origen y destino;
- **ningun paso destructivo corre si su copia no se verifico abriendola.**

Un ejemplo de por que la segunda regla importa: una copia a cuarentena se dio por
completa y le faltaban miles de archivos. Se detecto porque el borrado posterior
exigia una comparacion archivo por archivo.

## La prueba en una maquina limpia

La Fase 5 se hizo en una maquina virtual creada desde cero, **con las mismas dos
cuentas que el equipo real**: una administradora y una de uso diario sin
privilegios. Esa decision encontro defectos que una maquina de una sola cuenta
nunca habria mostrado:

1. **Una cuenta estandar no puede registrar tareas programadas.** La
   restauracion quedo en tres partes: administradora, usuario y otra vez
   administradora, que registra las tareas a nombre de la cuenta diaria.
2. **Un paquete instalado desde la tienda solo existe para quien lo instalo.** La
   cuenta diaria no veia la herramienta que necesitaban sus propias tareas. Se
   fuerza el instalador clasico y el script busca la herramienta en una ruta
   valida para cualquier cuenta.
3. **La restauracion corria en el perfil equivocado.** El script ahora verifica
   la cuenta antes de empezar y se corta si no es la correcta.
4. **Un instalador fallo una vez de forma transitoria.** Se reintenta una vez
   antes de declarar el fallo.
5. **Un error propio en el script de primer arranque** hacia que el instalador
   de paquetes no se ejecutara nunca, sin mostrar ningun error. Se agrego una
   verificacion explicita al final.

## El respaldo nocturno con archivos abiertos

Parte de la configuracion que habia que respaldar pertenece a herramientas que
**nunca se cierran**. La primera version aceptaba en silencio los archivos
bloqueados y habria producido, algun dia, un respaldo al que le faltaba justo lo
importante.

Criterio adoptado, definido por el dueno del equipo: **perder unas horas de
trabajo es aceptable; restaurar algo de hace meses creyendo que es de ayer, no.**

- se leen los archivos abiertos en modo compartido;
- las bases de datos incluidas se comprueban **extrayendolas del respaldo**, no
  en el origen;
- si algo quedo incompleto, el archivo se marca como incompleto y **no renueva la
  metrica de exito**. Si eso se repite, la alerta avisa sola.

## Remoto propio: archivar no es borrar

Con el remoto autoalojado en marcha se limpiaron los repositorios:

- se borro **solo** un repositorio cuyo contenido estaba integro en otro, lo que
  se **verifico commit a commit** y no por el parecido de los nombres;
- los repositorios de proyectos dados de baja se **archivaron**, no se borraron.
  Sus copias locales estaban en cuarentena: cuando venciera, el repositorio iba a
  ser la unica copia de ese historial. Borrarlos no liberaba espacio util y no
  tenia vuelta atras.

Un remoto dentro de la misma casa **no es una copia fuera del sitio**. Por eso el
remoto entra en los respaldos del hipervisor, y queda pendiente una copia a un
medio externo.

## Patron comun

Los tres hallazgos comparten la leccion del [Caso 06](06-cuando-un-control-no-mide-lo-que-dice-medir.md):
**la senal de exito venia del paso equivocado.**

| Control | Que media | Que tenia que medir |
|---|---|---|
| Metrica del respaldo | edad del comprimido | edad del contenido |
| Aviso del envio fuera de casa | errores de un comando | resultado del proceso completo |
| Espacio libre tras borrar | lo borrado dentro del invitado | lo liberado en el anfitrion |

## Leccion

**Un plan de migracion que nunca se probo es una hipotesis.** La unica
autorizacion para formatear es una maquina que nunca fue la propia, levantada
entera desde el documento. Y un respaldo solo esta probado cuando alguien abrio
lo que contiene.
