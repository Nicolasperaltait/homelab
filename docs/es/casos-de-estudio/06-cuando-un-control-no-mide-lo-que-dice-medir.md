# Caso 06 - Cuando un Control No Mide lo que Dice Medir

## Contexto

Durante una semana de endurecimiento sobre una plataforma de laboratorio
productivo se aplicaron cambios en accesos, privilegios, auditoria y filtrado de
red. Cada cambio se acompano de un script que verificaba su propio resultado.

**El patron mas frecuente de la semana no fue un cambio que fallara. Fue una
verificacion que mentia.**

Aparecio siete veces. Vale la pena escribirlo porque no es un error de
implementacion: es un error de razonamiento, y se repite.

## Los casos

### 1. El endurecimiento que informo exito sin haber endurecido nada

Un script cerraba la autenticacion por contrasena y confirmaba que habia escrito
el archivo correctamente. Lo habia escrito. **Y la contrasena seguia
funcionando.**

En ese servicio de acceso remoto **gana la PRIMERA aparicion de cada directiva**,
y los fragmentos de configuracion se cargan en orden alfabetico. Un archivo
generado por el aprovisionamiento automatico, con un prefijo numerico mas bajo,
le ganaba al del endurecimiento.

**El script verificaba que habia escrito, no que el sistema hubiera cambiado.**

Correccion: el fragmento pasa a cargarse primero, y la verificacion consulta la
**configuracion efectiva** del servicio en vez de leer el archivo.

### 2. El guardia que nunca comprobo nada

Antes de retirar una cuenta en desuso, un script contaba archivos, procesos y
tareas programadas para confirmar que estaba inerte.

La utilidad de conteo **imprime cero y a la vez informa fallo**. La forma
defensiva que se habia usado producia dos lineas en vez de un numero, la
comparacion numerica fallaba, **y una comparacion que falla se lee como
negativa**. La conclusion "esta inerte" salia sin haber comprobado nada.

**Un guardia que falla hacia el lado permisivo es peor que no tener guardia**,
porque produce una afirmacion que nadie vuelve a cuestionar.

Correccion: normalizar los contadores y **bloquear explicitamente** si el valor
no es numerico.

### 3. La verificacion que revirtio un cambio correcto

Tras aplicar una credencial nueva a un servicio, la comprobacion buscaba errores
en el registro del contenedor. Encontro uno y revirtio el cambio.

**El error era de horas antes.** La verificacion leia el registro completo, sin
acotar al arranque posterior al cambio.

Correccion: la fuente de verdad pasa a ser **el valor efectivo en el entorno del
proceso**, comparado byte a byte. El registro queda como informacion, acotado al
arranque nuevo.

### 4. El aviso de seguridad que rompio la automatizacion

Se agrego un mensaje de acceso restringido, como pide el estandar de
endurecimiento.

Ese mensaje **se emite antes de autenticar, en toda conexion**, incluidas las no
interactivas. Cada comando automatizado empezo a devolver siete lineas de aviso
mezcladas con su salida.

**El control funcionaba exactamente como se le pidio. El problema era el
alcance.**

Correccion: excepcion explicita para las cuentas de automatizacion.

### 5. La red de seguridad que causaba el mismo dano del que rescataba

Al preparar un cambio de filtrado de red se diseno un temporizador de rescate:
si el operador quedaba sin acceso, a los quince minutos se revertian las reglas
solas.

**El comando de rescate borraba TODAS las tablas de filtrado del host**, no solo
las propias. En un host con contenedores eso significa la red de contenedores y
los puertos publicados. En el nodo de acceso remoto, el tunel entero.

**El rescate habria causado exactamente el mismo dano que la falla de la que
rescataba.**

Correccion: crear, borrar y recrear **unicamente la tabla propia**. Nadie habia
mirado que hacia el comando de rescate en un host con contenedores: la red de
seguridad se habia escrito, no probado.

### 6. La validacion que pasaba por pieza y fallaba en conjunto

Un script generaba dos archivos de permisos, uno por cada cuenta de
automatizacion, y validaba cada uno antes de instalarlo. Los dos validaban.

**El conjunto no.** Los dos definian el mismo alias, y definir un alias dos veces
es un error.

**Validar la pieza no valida el todo.**

Y el error mas grave fue otro: **al detectar el problema, el script aviso y
salio, dejando los archivos rotos puestos.** Otro script escrito la misma manana
si revertia.

**Avisar no es revertir.** Un script que detecta que rompio algo y no lo deshace
es peor que uno que no lo detecta, porque genera confianza en una comprobacion
que no protege.

### 7. Y la variante inversa: el control que grito sobre un sistema sano

Al desplegar la auditoria del sistema, la verificacion informo que en un host
solo se habian cargado ocho reglas de veintitres.

Se investigo el host. Todo estaba bien.

**La causa era la verificacion.** Esperaba un tiempo fijo y despues contaba. Las
reglas se cargan de a una, y en el host mas lento ese tiempo capturo la carga
**por la mitad**. Las ocho eran **las ocho primeras del archivo, en orden**: la
pista estaba en el resultado desde el principio.

**Un control que reporta falla en un sistema sano es tan defectuoso como uno que
calla ante una falla real**, y peor a mediano plazo: entrena a ignorarlo.

Correccion: esperar a que el conteo **se estabilice**, y distinguir "puede seguir
cargando" de "fallo".

### 8. La sanitizacion que solo miraba el arbol de trabajo

**Descubierto el 2026-08-29, y es el caso mas incomodo porque afecta a este mismo
repositorio.**

Este repositorio publico tiene una politica de sanitizacion documentada, con una
lista de patrones sensibles y un escaneo obligatorio. El escaneo se corria sobre
los borradores y sobre el diff antes de cada commit.

**Nunca se corrio contra el historial.**

Meses atras, un commit retiro del arbol de trabajo la nomenclatura interna del
entorno. Los commits anteriores la conservaban, y esos commits **ya estaban
publicados**.

El resultado: el arbol de trabajo estaba limpio, el escaneo pasaba, y el
repositorio mostraba una insignia de "sanitizado" que **era cierta del arbol y
falsa del repositorio**.

**Mismo patron que los siete casos anteriores: se verifico la accion -este commit
esta limpio- en vez del efecto -el repositorio esta limpio-.**

Correccion:

- historial reescrito, con los valores sensibles reemplazados por marcadores de
  redaccion explicitos, para que un commit viejo diga claramente que fue
  saneado en vez de simular que nunca dijo otra cosa;
- respaldo verificado del repositorio completo antes de tocar nada;
- **la comprobacion pasa a recorrer todos los objetos de todos los commits**, no
  solo el diff del cambio que se esta haciendo.

**Severidad honesta:** lo expuesto eran sufijos de nombres internos, no
enrutables y sin valor por si mismos. El costo real no era tecnico: era la
distancia entre lo que el repositorio afirmaba de si mismo y lo que hacia.

### Y la primera correccion tambien fue incompleta

**La reescritura del historial se hizo dos veces el mismo dia.**

La primera paso sobre la nomenclatura interna y **dejo intactos siete nombres de
producto**. El motivo es el mismo defecto una vez mas: **la lista de reemplazos
salio del escaneo previo, y ese escaneo buscaba solo lo que ya se sospechaba.**
La correccion heredo el punto ciego de la busqueda que la definio.

Se detecto verificando **contra el remoto** despues de publicar, no contra la
copia local. Que un envio termine bien no prueba que lo publicado este limpio.

La segunda pasada cambio el metodo: en vez de buscar una lista de terminos
sospechados, **se enumero lo que efectivamente habia** -todas las frases con el
patron de cobertura que el repositorio usaba, y una barrida amplia de nombres de
producto de infraestructura- y recien despues se construyeron los reemplazos.

**Buscar lo que uno sospecha encuentra lo que uno sospecha.** Para saber que hay,
hay que enumerar.

**Severidad, otra vez sin adornos:** nombres de producto no son un dato sensible;
saber que stack corre alguien no abre nada. Lo que estaba mal era la
inconsistencia entre lo que este repositorio declara -no nombrar productos- y lo
que su historial mostraba.

### Y la comprobacion de la correccion tambien estaba mal

La primera verificacion de que el historial habia quedado limpio uso una tuberia
de shell donde **el codigo de salida venia del ultimo comando y no de la
busqueda**. Informaba coincidencias sobre un resultado vacio.

Se rehizo midiendo el contenido en vez del codigo de salida, y se agrego una
segunda comprobacion independiente sobre cada objeto del repositorio.

**Hasta la comprobacion del arreglo necesito ser comprobada.**

## El patron

Los siete casos son el mismo error con distinta ropa:

**Se verifico la accion en vez del efecto.**

- se escribio el archivo -> no: cambio la configuracion efectiva?
- el comando no dio error -> no: el sistema rechaza ahora lo que debe rechazar?
- la pieza valida -> no: el conjunto valida?
- paso el tiempo previsto -> no: la operacion termino?

## La practica que salio de esto

Todo script de este entorno **comprueba lo que tiene que FALLAR**, no solo lo que
tiene que funcionar.

| Control | Que comprueba que falla |
|---|---|
| Envoltorio de lectura privilegiada | Que **deniega** el archivo de contrasenas, una clave privada, la clave del host, y una ruta con `..` que intente evadirlo |
| Reduccion de privilegios | Que un lector generico y un interprete de comandos **quedan denegados** |
| Rotacion de credenciales | Que la credencial **vieja devuelve 401** |
| Endurecimiento de acceso remoto | La configuracion **efectiva**, no el archivo escrito |

El caso de la rotacion lo demuestra solo. La comprobacion tiene dos mitades: que
la credencial nueva funcione, y que **la vieja deje de funcionar**. En la primera
corrida la nueva funcionaba y **la vieja tambien**.

**Un script que solo verificara la primera mitad habria informado "rotacion
completada" con la credencial expuesta todavia valida.**

Crear una credencial nueva no es rotar. **Rotar es que la vieja deje de servir**,
y eso hay que comprobarlo.

## Corolario: los mensajes de error tambien son un control

Uno de los scripts informo *"el conjunto de permisos quedo invalido"*, que era
**cierto**. Se leyo como *"el sistema quedo sin escalada de privilegios"*, que
era **falso**: seguia aplicando las reglas y solo emitia avisos.

La recuperacion "obvia" ante ese mensaje era **restaurar el punto de control de
una maquina que estaba funcionando bien**. El arreglo real eran dos lineas.

**Un error tiene que describir el alcance del dano, no solo su existencia.**
