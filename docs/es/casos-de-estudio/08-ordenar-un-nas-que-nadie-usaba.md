# Caso 08 - Ordenar un NAS que Nadie Usaba

## Contexto

El laboratorio tiene un NAS virtualizado que recibe los respaldos de la estacion de
trabajo. Funcionaba: los respaldos corrian todas las noches y las metricas estaban en
verde. Pero acumulaba meses de capas:

- cuentas de un trabajo anterior que ya no aplicaba;
- recursos compartidos que apuntaban a carpetas que **ya no existian**;
- tareas programadas que notificaban a un canal de mensajeria dado de baja semanas antes;
- servicios de descubrimiento de red que nadie usaba;
- decenas de actualizaciones pendientes;
- y rutas de red hacia la zona de seguridad que **desaparecian** cada vez que el sistema
  reiniciaba su pila de red.

Y, sobre todo, **no tenia ninguna utilidad visible para su dueno**. Era un deposito de
respaldos que nadie abria.

El objetivo de la sesion fue doble: **ordenarlo con minimo privilegio**, y **darle un uso
real** sin comprometer lo que ya hacia bien.

## La regla de trabajo

Nada se aplico sin un plan escrito con reversion. Cada tanda fue un script que:

1. guarda un respaldo de lo que va a tocar;
2. aplica el cambio;
3. **prueba el efecto, no la accion**, y cuando puede, **prueba que lo que tiene que fallar
   falle**;
4. y si una verificacion no da, se detiene y dice donde quedo el respaldo.

El asistente de IA prepara y verifica en modo lectura; **todo cambio con privilegio lo
ejecuta el operador**, con su contrasena, en su terminal.

## 1. Rutas que no sobrevivian a la red

Las rutas hacia las zonas internas las creaba un servicio de un solo disparo al arrancar.
Cubria el arranque y nada mas. Una actualizacion de una biblioteca base del sistema
reinicio el gestor de red y las borro; el agente del SIEM en ese host quedo desconectado
hora y media, y el sintoma aparecio en **otro** sistema.

**Correccion:** las rutas pasan a la configuracion declarativa de red, en un archivo
propio que el gestor del appliance no genera ni borra.

**Prueba**, en tres niveles de dureza creciente:

| Prueba | Resultado |
|---|---|
| Reiniciar el gestor de red (lo que hizo la actualizacion) | rutas de vuelta solas |
| Redesplegar la red desde el panel del appliance | el archivo sigue, rutas presentes |
| **Reiniciar la maquina** | rutas de vuelta solas |

Si alguna fallaba, el script revertia solo.

## 2. Cuentas por funcion

| Rol | Consola | Privilegio | Recursos compartidos |
|---|---|---|---|
| Administrador interno | no | total | no |
| Operador humano | si, con clave | **unico con elevacion** | excluido a proposito |
| Acceso al NAS | **sin consola** | ninguno | lectura/escritura |
| Acceso al NAS de solo lectura | sin consola | ninguno | lectura |
| Agente de IA | solo desde la VM de salto | lectura acotada | no |

Dos cuentas se borraron **despues** de verificar que no tenian procesos ni archivos. Una
que parecia prescindible **se conservo**: la usaban dos tareas automaticas de la
estacion de trabajo. Se le quito la consola, porque esas tareas no la necesitan.

**Criterio:** antes de retirar una identidad, preguntar que mas abria. Antes de conservarla,
preguntar que necesita realmente.

## 3. Endurecimiento

- tareas programadas muertas **movidas, no borradas**, y el archivo de configuracion del
  canal viejo movido **sin leerlo** (podia contener un secreto);
- descubrimiento de red y servicios sin uso apagados;
- protocolo minimo de archivos compartidos elevado a la version moderna;
- el acceso desde fuera de casa solo por la malla de acceso remoto, **nunca expuesto a
  internet**;
- todas las actualizaciones aplicadas con **instantanea previa de la VM**, reinicio, y
  un chequeo posterior que mira **conectividad**, no solo servicios: rutas, alcance a las
  zonas internas y a internet, y el agente del SIEM conectado **segun su propio estado**,
  no segun el gestor de servicios;
- la instantanea se **borro** al verificar: en un disco virtual de copia en escritura, una
  instantanea olvidada retiene bloques y ya habia llenado el almacenamiento una vez;
- auditoria del sistema enviada al SIEM y reglas de auditoria **inmutables** hasta el
  proximo arranque;
- una alerta nueva: **agente del SIEM desconectado**. La metrica existia; la regla no.

## 4. Darle utilidad: una carpeta, tres puertas

Se creo una unica carpeta de intercambio con tres formas de llegar:

| Puerta | Para que |
|---|---|
| Navegador web | subir y bajar desde cualquier equipo, en casa o por la malla remota |
| Sincronizacion de archivos | lo que se deja en la PC aparece en el NAS y en el telefono |
| Recurso compartido | como una unidad de red mas |

**El explorador web corre en un contenedor que solo monta esa carpeta.** Se probo el
aislamiento dejando un archivo testigo: el contenedor lo ve, y **no ve** ninguna carpeta de
respaldos.

**Y se hizo lugar antes de invitar a usarla.** La carpeta vive en el mismo disco que los
respaldos, cuyo trabajo nocturno **no corre** si el libre baja de un umbral. El margen era
de un 6 %. Se amplio el disco virtual en caliente, verificando antes que no hubiera
instantaneas vivas, y el margen paso a mas de un 40 %.

## Lo que salio mal, contado sin adornos

### El control que fallo sobre un cambio correcto

La verificacion del protocolo minimo comparaba contra un texto literal. La herramienta
informa el mismo valor con un sufijo de version. **El cambio estaba bien; el control
fallo** y corto la verificacion que venia despues, que hubo que completar a mano.

### El puerto abierto que no estaba abierto

Se abrio el puerto del explorador web y desde la red daba tiempo agotado, mientras que
**desde el propio host respondia bien**. El motor de contenedores publica el puerto
reescribiendo el destino: la conexion viaja por la cadena de **reenvio** del firewall, no
por la de **entrada**. La regla agregada nunca se evaluaba.

Efecto lateral bueno: la cuenta de administracion del explorador todavia tenia **la clave
de fabrica**, y por ese error **nunca fue alcanzable**. La leccion igual vale: el orden
correcto era cambiar la clave **antes** de abrir, y probar el puerto **desde otra maquina**.

### Un tunel que el propio endurecimiento prohibia

Para cambiar esa clave con el puerto cerrado se propuso un tunel por el acceso remoto
seguro. **El endurecimiento del mismo host prohibe los tuneles.** Revisar la configuracion
efectiva antes de proponer un camino.

### La sincronizacion cortada por un campo invisible

Al aceptar la carpeta de intercambio en la estacion de trabajo, **la sincronizacion de
todos los respaldos** empezo a cortarse cada pocos segundos, durante 17 minutos.

La causa: una plantilla heredaba una clave de cifrado en la entrada **del propio
dispositivo**. La interfaz grafica solo muestra la de los otros dispositivos, asi que en
pantalla todo se veia vacio. Se encontro leyendo la configuracion cruda por la API.

**Ya estaba documentado** un caso casi igual de semanas atras. Esta vez la trampa estaba un
lugar mas alla de lo que la interfaz deja ver. La correccion incluyo la plantilla, para
que la proxima carpeta no la herede.

### Un silencio leido como evidencia

Se concluyo que un dispositivo "ni siquiera intentaba conectarse" porque el registro no
mostraba nada. **La cuenta que consultaba no tenia permiso para leer ese registro**: la
respuesta iba a ser vacia siempre.

**Una salida vacia solo es evidencia si la misma consulta es capaz de ver algo cuando
algo pasa.**

### El boton que iba a deshacer el endurecimiento

Al terminar, el panel del appliance mostraba "cambios pendientes de aplicar". Aplicarlos
**habria vuelto a encender** un servicio que se habia apagado a mano: el gestor de
configuracion lo reactiva salvo que su propia configuracion diga lo contrario.

**Correccion:** el apagado se movio a la configuracion del gestor, y recien despues se
aplico, verificando que el servicio siguiera apagado.

## Una semana despues: la utilidad que no fue

Ocho dias mas tarde, la carpeta de intercambio **estaba vacia**. Nadie la habia usado.
Tres puertas para una carpeta no eran lo que hacia falta.

La pregunta cambio de "que se le puede agregar al NAS" a **"que problema real tiene su
dueno"**. La respuesta fue concreta: *me piden un documento y no lo puedo conseguir*, y
*quiero poder volver a una version vieja*. Se descartaron a proposito las salidas
tipicas: galeria de fotos, servidor de peliculas y una suite tipo nube completa.

### Un disco de red, el mismo en todos lados

La solucion fue la mas simple: **montar el NAS entero como un disco mas** en cada
equipo, siempre por la direccion del NAS dentro de la malla de acceso remoto. Es la
misma direccion en casa y afuera, y el trafico va cifrado.

| Equipo | Como |
|---|---|
| PC de escritorio | unidad de red, como antes |
| Notebook Linux | montaje automatico: se conecta al abrir la carpeta y se suelta sin uso |
| Telefono | explorador de archivos con cliente de red |

Sin copia local: el dueno no trabaja sin conexion, y montado no hay nada que
sincronizar. Una carpeta nueva aparece sola en los tres.

**Las versiones viejas ya existian.** El espejo de respaldos guardaba versiones desde
hacia semanas; lo que faltaba era **poder llegar a ellas**: el recurso compartido oculta
las carpetas que empiezan con punto, y ahi viven. Desde la notebook y el telefono se ven.

Se comparo contra una suite de nube completa (web, apps, links para compartir). Se
descarto por el costo: base de datos, cache, mas memoria y actualizaciones mayores que
mantener, para dos necesidades que un disco de red ya cubria. Queda la puerta abierta:
puede montarse despues sobre las mismas carpetas.

### Lo que habia quedado mal la semana anterior

**El explorador web "por la malla" nunca habia funcionado.** Se probo el 25/09 solo desde
casa. Desde la malla, la conexion colgaba con la regla del firewall del host bien
escrita: la **politica de la malla** no permitia ese puerto, y como el paquete no llega
al host, su firewall no registra nada. Lo mismo le pasaba a la sincronizacion del
telefono fuera de casa. Se agrego el permiso **y una prueba en la propia politica**, que
hace rechazar cualquier cambio futuro que lo vuelva a cerrar.

**Lo sincronizado no aparecia en el recurso compartido.** Estaba en el disco y no en la
unidad de red. Se corrigieron los permisos de la carpeta y la mascara con la que el
servicio de sincronizacion crea los archivos. Quedo funcionando, **con la causa exacta
sin confirmar**: la hipotesis inicial no cerraba con los permisos que tenia el archivo.
Se registro asi, sin inventar una explicacion.

### Errores propios, sin adornos

- Un reemplazo de texto con una ruta de Windows interpreto las barras invertidas como
  secuencias de escape y **rompio dos documentos**. Se repararon armando la barra fuera
  del reemplazo.
- Un bloque para la notebook **no decia en que maquina correrlo**, y se corrio en la PC.
- Otro bloque usaba un formato de ruta que la terminal del operador no entiende.

### Lo que queda abierto, dicho de frente

- Lo que el dueno cree directamente en el NAS **no tiene respaldo**: los respaldos
  copian rutas fijas que vienen de la PC.
- La notebook guarda una credencial con escritura sobre todo el NAS y **su disco no esta
  cifrado**. Paso a prioridad alta.

## Patrones

1. **Persistir en la configuracion que manda.** Una ruta agregada a mano, un servicio
   enmascarado por fuera del gestor: duran hasta el proximo reinicio de red o el proximo
   "aplicar".
2. **Probar desde donde prueba el usuario.** Un puerto se prueba desde otra maquina; una
   ruta, reiniciando de verdad.
3. **Hacer lugar antes de invitar a usar.** La utilidad nueva compartia disco con los
   respaldos; ampliar primero evito que el exito de la carpeta frenara el respaldo nocturno.
4. **La interfaz no es la configuracion.** Cuando pantalla y sintoma no coinciden, leer la
   configuracion cruda.
5. **Un vacio no es un "no".** Verificar primero que la consulta puede ver.
6. **La utilidad se mide en uso, no en funciones.** Una carpeta vacia a la semana dice mas
   que tres puertas funcionando. Preguntar por el problema, no por la herramienta.
7. **Desde la malla hay dos filtros.** Un puerto para usar desde afuera necesita la regla
   del host **y** el permiso de la malla, y se prueba contra la direccion de la malla.

## Resultado

| Antes | Despues |
|---|---|
| rutas que se perdian con cada reinicio de red | rutas declarativas, probadas con reinicio real |
| cuentas de un trabajo anterior | cinco identidades, una funcion cada una |
| recursos compartidos hacia carpetas inexistentes | un recurso con lectura/escritura y lectura separadas |
| decenas de actualizaciones pendientes | cero, con reinicio verificado |
| agente del SIEM caido sin aviso | alerta dedicada |
| margen del 6 % sobre el umbral de respaldo | margen de mas del 40 % |
| sin uso para su dueno | carpeta de intercambio con navegador, sincronizacion y red |
| carpeta de intercambio vacia a la semana | el NAS entero como disco en PC, notebook y telefono, con versiones al alcance |
