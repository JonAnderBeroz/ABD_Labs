# Laboratorio __Arquitectura del SGBD Oracle__

__1. Entender la organización de Oracle__

- Buscar los ficheros de datos, redo log, control. Para ello en la sesión de SYS acceder a las
vistas v$datafile y v$controlfile (atributo name), y v$logfile (atributos group# y member). Para buscar el fichero de parámetros de inicialización show parameter spfile y comprueba su ubicación en la carpeta correspondiente.

||Carpeta|Ficheros|
|-|-|-|
|Ficheros Redo log|||
|Ficheros de control|||
|Ficheros de Datos|||
|Ficheros de Parámetros de Inicialización|||

- Para comprobar el tamaño de la memoria SGA (System Global Area): __show sga__

||Tamaño|
|-|-|
|SGA||
|Buffers de la BD||
|Buffers de Redo||
|Resto||

- Para comprobar el tamaño de la memoria SGA (System Global Area): show sga