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
