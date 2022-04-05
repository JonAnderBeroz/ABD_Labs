# Laboratorio __Arquitectura del SGBD Oracle__

__1. Entender la organización de Oracle__

- Buscar los ficheros de datos, redo log, control. Para ello en la sesión de SYS acceder a las
vistas v$datafile y v$controlfile (atributo name), y v$logfile (atributos group# y member). Para buscar el fichero de parámetros de inicialización show parameter spfile y comprueba su ubicación en la carpeta correspondiente.

||Carpeta|Ficheros|
|-|-|-|
|Ficheros Redo log|ORADATA|REDO01.LOG<br>REDO02.LOG<br>REDO03.LOG|
|Ficheros de control|ORADATA<br>FAST_RECOVERY_AREA|CONTROL01.CTL<br>CONTROL02.CTL |
|Ficheros de Datos|ORADATA|SYSTEM01.DBF<br>SYSAUX01.DBF<br>UNDOTBS01.DBF<br>USERS01.DBF|
|Ficheros de Parámetros de Inicialización| DATABASE| SPFILEABDLAB.ORA|

- Para comprobar el tamaño de la memoria SGA (System Global Area): __show sga__

||Tamaño|
|-|-|
|SGA| 838860800 bytes  |
|Buffers de la BD|318767104 bytes |
|Buffers de Redo|5337088 bytes  |
|Resto|3051184 bytes + 511705424 bytes|

- Para comprobar las características de la Base de Datos, acceder a la vista __v$database__ con
estos atributos: __dbid__ (identificador interno de la BD), __name__, __created__ (fecha de creación), __log_mode__, __open_mode__.

||v$database|
|-|-|
|dbid: 973164796|log_mode: NOARCHIVELOG|
|name: ABDLAB|open_mode: READ WRITE|
|created: 24/07/17 ||

- Para comprobar las características de la instancia acceder a la vista __v$thread__ con estos atributos: __thread#__, __status__, __enabled__ (qué usuarios pueden acceder), __groups__ (cuántos grupos de redo log tiene definidos), __instance__ (nombre de la instancia), __open_time__, __current_group#__ (de entre los grupos de redo log, cuál es el actual, en cuál está escribiendo el sistema), __sequence#__ (número de secuencia del grupo de redo actual), __checkpoint_change#__ (el número del último cambio al realizar un punto de sincronización), __checkpoint_time__ (cuándo se ha realizado el último punto de sincronización)

||v$thread|
|-|-|
|thread#: 1|open_time: 04/04/22|
|status: OPEN|current_group#: 3|
|enabled: PUBLIC|sequence#: 24|
|groups: 3|checkpoint_change#: 972761|
|instance: abdlab |checkpoint time:04/04/22 |

- Para comprobar los parámetros de inicialización: El fichero de parámetros ha sido
construido por el asistente de creación de bases de datos en función de las opciones
elegidas en las diferentes etapas por las que pasa el asistente en el proceso de construcción
de una nueva base de datos. Ejecuta create pfile from spfile; y estudia los parámetros
recogidos en el fichero C:\oracle\db12\database\initabdlab.ora que se ha generado.
Los parámetros que van precedidos por abdlab.__ son parámetros a los que ha asignado
valor automáticamente la propia base de datos. Toma nota de sus valores. (Podría verse
ejecutando show parameter pero aparecen muchos más)

abdlab.__data_transfer_cache_size=0
abdlab.__db_cache_size=289406976
abdlab.__java_pool_size=4194304
abdlab.__large_pool_size=8388608
abdlab.__oracle_base='C:\oracle'#ORACLE_BASE set from environment
abdlab.__pga_aggregate_target=335544320
abdlab.__sga_target=503316480
abdlab.__shared_io_pool_size=16777216
abdlab.__shared_pool_size=176160768
abdlab.__streams_pool_size=0
*.audit_file_dest='C:\oracle\admin\abdlab\adump'
*.audit_trail='db'
*.compatible='12.1.0.2.0'
*.control_files='C:\oracle\oradata\abdlab\control01.ctl','C:\oracle\fast_recovery_area\abdlab\control02.ctl'
*.db_block_size=8192
*.db_domain='si.ehu.es'
*.db_name='abdlab'
*.db_recovery_file_dest='C:\oracle\fast_recovery_area'
*.db_recovery_file_dest_size=24080m
*.diagnostic_dest='C:\oracle'
*.dispatchers='(PROTOCOL=TCP) (SERVICE=abdlabXDB)'
*.memory_target=800m
*.nls_language='SPANISH'
*.nls_territory='SPAIN'
*.open_cursors=300
*.processes=300
*.remote_login_passwordfile='EXCLUSIVE'
*.undo_tablespace='UNDOTBS1'

- Puedes ver que la instalación de Oracle con la que se ha trabajado en los otros laboratorios
es similar a la de esta máquina virtual. Desde una sesión con tu usuario ABDn puedes
realizar las mismas consultas de los apartados a-d. Para ver los parámetros de inicialización
puedes realizar la consulta __select name, value from v$parameter where isdefault='FALSE'__

__2. Espacios de tablas (tablespaces)__

Las tablas y otros objetos de la base de datos se almacenan en las secciones de disco
denominadas ‘espacios de tablas’ (tablespace). En este ejercicio crearemos uno de esos
espacios y guardaremos ahí una de nuestras tablas.

- Aparte de los espacios de tablas que están ya creados en la base de datos, crea un espacio
de tablas, con nombre __espacioN__ (donde N es el número de tu usuario ABDn). Para ello,
ejecutar esta instrucción:

__create tablespace espacioN datafile 'C:\ORACLE\ORADATA\ABDLAB\espacioN.dbf' size 200k;__

- Comprobar que el fichero se ha creado en la carpeta indicada. Así mismo, con la conexión
como DBA en la carpeta “Almacenamiento+Tablespaces”, comprobar las características del
nuevo espacio de tablas.

- Asigna al usuario __usuario1__ su cuota ‘unlimited’ sobre el espacio de tablas __espacioN__.

- En la sesión del usuario1 c rear un a nueva tabla en el espacio espacioN, con la siguiente
instrucción:

__create table PRUEBA (COD NUMBER) tablespace ESPACION__

Insertar alguna tupla en la tabla creada. Ahora, ¿cuántos bytes tiene ocupados el espacio de
tablas?

- Desde la conexión como DBA borrar el espacio de tablas que acabamos de crear (borrando
también los objetos incluidos en él) pero manteniendo el fichero físico espacioN.dbf.
Comprueba que así ha sido. ('C:\ORACLE\ORADATA\ABDLAB\espacioN.dbf'). Asignar de
nuevo al usuario1 el tablespace USERS con cuota ilimitada.