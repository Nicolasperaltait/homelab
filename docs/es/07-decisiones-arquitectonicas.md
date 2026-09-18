# 07 - Decisiones Arquitectonicas

> Estado descrito: septiembre de 2026.

## Proposito

Resumir las decisiones principales detras del diseno, con su razonamiento, su
costo y donde se documentan.

## Matriz de decisiones

| Decision | Razonamiento | Tradeoff | Donde se ve |
|---|---|---|---|
| Segmentar por funcion en el hipervisor | reduce ambiguedad y lateralidad | requiere disenar rutas y accesos | 02 |
| Tratar el hipervisor como control plane | computo, transito y recuperacion dependen de el | sus cambios son sensibles | 00, 02 |
| Acceso remoto por malla sin puertos entrantes | el borde queda cerrado | dependencia de un plano de control externo | 03 |
| Rutas por host en la malla | funciona desde redes ajenas con el mismo rango | mas entradas que mantener | 03 |
| Bloqueo de la malla con nodos firmantes | una credencial valida no alcanza para entrar | el alta de un equipo exige un paso extra | 03 |
| Un unico camino para los agentes de IA | apagar la automatizacion no apaga al operador | el operador enciende el host a mano | 03, caso 05 |
| Minimo privilegio medido, no disenado | un permiso que no se usa no debe existir | revisarlo cuando cambia el uso | caso 05 |
| Verificar controles haciendolos fallar | una verificacion puede mentir | scripts de prueba mas largos | caso 06 |
| Tratar el host de contenedores distinto | un firewall generico rompe la red de contenedores | hardening por rol | 03 |
| Parcheo automatico solo de seguridad y sin reinicio | seguridad al dia sin cortes sorpresa | los reinicios se acumulan para una ventana | 03 |
| Alertar solo fallos | un canal ruidoso se ignora | lo sano se mira en el dashboard | 06 |
| Toda tarea automatica deja una metrica | una tarea que deja de correr es invisible | un exportador mas por tarea | 06 |
| Backups medidos por su contenido | un archivo reciente no prueba datos nuevos | metricas mas especificas | 05, caso 07 |
| Pruebas de restauracion sin credenciales | la validacion no se vuelve un secreto a proteger | no prueba un inicio de sesion real | 05 |
| Remoto de codigo propio | el historial no depende de un servicio externo | hay que respaldarlo y sacarlo de casa | 05 |
| Borrar solo con cuarentena y copia verificada | un borrado apurado no tiene vuelta | espacio ocupado treinta dias | 04, caso 07 |
| Archivar en vez de borrar lo que es ultima copia | el historial no se pierde | repositorios inactivos visibles | caso 07 |
| Dar de baja lo que no se usa | menos superficie y menos mantenimiento | decidirlo con datos de uso | 02 |
| Probar la migracion en una maquina limpia con las mismas cuentas | los defectos de privilegios solo aparecen asi | una prueba mas larga | caso 07 |
| Documentacion publica sanitizada | claridad tecnica sin exponer el entorno | menos detalle de implementacion | README |

## Decisiones revertidas

Tambien se documenta lo que se decidio y despues se deshizo:

| Decision original | Por que se revirtio |
|---|---|
| Tunel VPN propio con puerto entrante | contradecia el principio de borde cerrado |
| Canal de chat para alertas | nunca llego a entregar un aviso |
| Consola dedicada para dashboards | el aviso por mensajeria la volvio innecesaria |
| Servicios de IA local en el laboratorio | no se usaban; se retiraron con su historial archivado |
| Un servicio de credenciales autoalojado | no se usaba; su baja esta en curso |

## Como leerlo

> El lab es pequeno, pero esta disenado como plataforma: segmentado, sin
> exposicion entrante, observable, recuperable con evidencia, y documentado con
> sus tradeoffs, sus errores y lo que todavia falta.
