# Laboratio tema Audotioría

1. Parámetros AUDIT_TRAIL y estado actual de AUD$ 
```
select USERNAME, OWNER, OBJ_NAME, ACTION_NAME, TIMESTAMP, RETURNCODE
from SYS.DBA_AUDIT_TRAIL
where OWNER  ='ABD04';

select *
from SYS.DBA_STMT_AUDIT_OPTS;

select *
from SYS.DBA_PRIV_AUDIT_OPTS;

select object_name, OBJECT_TYPE, DEL, INS, SEL, UPD
from SYS.DBA_OBJ_AUDIT_OPTS
where OWNER = 'ABD04';

select ALT, AUD, COM, DEL, GRA, IND, INS, LOC, REN, SEL, UPD, REF, EXE
from SYS.ALL_DEF_AUDIT_OPTS;
```
2. __BY  SESSION  vs  BY  ACCESS.__ Auditar  desde  la  sesión  de  ABDn  el uso  de  las  sentencias  ``SELECT  TABLE``  e  ``INSERT TABLE``  en cualquier caso (fallido o no). En  el caso de ``SELECT`` por sesión  y  en  el  caso  de  ``INSERT``  por  sentencia.  Asegúrate  primero  que  USUARIOAn  no  tiene sesión abierta.

```
AUDIT SELECT TABLE BY SESSION 
AUDIT INSERT TABLE BY ACCESS
COMMIT;
```

- Comprueba si hay diferencia entre estas dos opciones observando las dos nuevas filas de la
tabla ``SYS.DBA_STMT_AUDIT_OPTS``

|  User Name | Audit Option  | Success  | Failure  |
|---|---|---|---|
| null  | SELECT TABLE  | BY SESSION  | BY SESSION  |
| null  | INSERT TABLE  | BY ACCESS   | BY ACCESS   | 

- Abrir una sesión con USUARIOAn y realizar una sentencia SELECT a la tabla
ABDn.DEPARTAMENTOS e insertar varias tuplas en la tabla y realizar otro SELECT. Intenta también realizar un SELECT a la tabla EMPLEADOS. Consultar los registros de auditoría que se han generado para estas operaciones (vista ``SYS.DBA_AUDIT_TRAIL con WHERE OWNER = 'ABDn'``).

| Comandos   | Registro de auditoria   |
|---|:---:|
|  - | USERNAME OWNER OBJ_NAME ACTION_NAME TIMESTAMP RETURNCODE SES_ACTIONS  |
|  ``insert into ABD04.departamentos values ('a',6);``  | USUARIOA04 ABD04 DEPARTAMENTOS INSERT 09/02/22 0 null|
| ``insert into ABD04.departamentos values ('b',6);`` | USUARIOA04 ABD04 DEPARTAMENTOS INSERT 09/02/22 0 null |
|``select * from ABD04.departamentos;``| USUARIOA04	ABD04	DEPARTAMENTOS	SESSION REC	09/02/22	0	---------S------ |
|``select * from ABD04.Empleados;``|USUARIOA04	ABD04	EMPLEADOS	SESSION REC	09/02/22	2004	---------F------|

3. __WHENEVER SUCCESSFUL vs WHENEVER NOT SUCCESSFUL.__ Desde la sesión de ABDn
realizamos las siguientes auditorías:

```
AUDIT UPDATE TABLE BY USUARIOAn BY ACCESS WHENEVER SUCCESSFUL;
AUDIT UPDATE TABLE BY USUARIOBn BY ACCESS WHENEVER NOT SUCCESSFUL;
```

| USER_NAME   | AUDIT_OPTION  | SUCESS  | FAILURE   |
|---|---|---|---|
| USUARIOB04  | UPDATE TABLE   | NOT SET   |  BY ACCESS |
| USUARIOA04  | UPDATE TABLE   | BY ACCESS  |   NOT SET |

- Otorga el privilegio necesario (UPDATE) sobre la tabla ABDn.DEPARTAMENTOS al
USUARIOBn. Comprueba que la tabla DEPARTAMENTOS DE ABDn contiene una tupla con el departamento de nombre 'LSI' y otra con el nombre 'ATC' (en caso contrario haz el INSERT) y que la tabla EMPLEADOS contiene un empleado con dni =123456 . Realiza algunas modificaciones abriendo sesiones desde USUARIOAn y USUARIOBn de manera que unas terminen correctamente y otras no.
Por ejemplo

|  USUARIOA04 | USUARIOB04   |
|---|---|
|``update ABDn.DEPARTAMENTOS set PRESUPUESTO=99999 where NOMBRE='LSI';`` | ``update ABDn.DEPARTAMENTOS set PRESUPUESTO=88888 where NOMBRE='ATC'; ``|
``update ABDn.DEPARTAMENTOS set PRESU= 99999 where NOMBRE='LSI';`` | ``update ABDn.DEPARTAMENTOS set PRESU= 88888 where NOMBRE='ATC';``   |
| | ``update into ABDn.DEPARTAMENTOS set PRESUPUESTO= 88888 where NOMBRE='ATC';`` |
| | ``update ABDn.EMPLEADOS set SALARIO= 2500 where DNI=123456;``  |
| | ``update ABDn.DEPTOS set PRESUPUESTO= 88888 where NOMBRE='ATC';`` |

- Comprueba con SYS.DBA_AUDIT_TRAIL (con WHERE OWNER = 'ABDn') qué registros se
obtienen y explica por qué.

USUARIOA04:

| Comandos  | Registros de auditorias  |
|---|:---:|
|  - |  USERNAME OWNER OBJ_NAME ACTION_NAME TIMESTAMP RETURNCODE |
| ``update ABDn.DEPARTAMENTOS set PRESUPUESTO=99999 where NOMBRE='LSI';`` | - |
| ``update ABDn.DEPARTAMENTOS set PRESU= 99999 where NOMBRE='LSI';``| -|


USUARIOB04:

| Comandos  | Registros de auditorias  |
|---|:---:|
|  - |  USERNAME OWNER OBJ_NAME ACTION_NAME TIMESTAMP RETURNCODE |
| ``update ABDn.DEPARTAMENTOS set PRESUPUESTO=88888 where NOMBRE='ATC';`` | - |
| ``update ABDn.DEPARTAMENTOS set PRESU= 88888 where NOMBRE='ATC';``| USUARIOB04	ABD04	DEPARTAMENTOS	UPDATE	10/02/22	904	|
|``update into ABDn.DEPARTAMENTOS set PRESUPUESTO= 88888 where NOMBRE='ATC';``| USUARIOB04	ABD04	DEPARTAMENTOS	UPDATE	10/02/22	904	|
|``update ABDn.EMPLEADOS set SALARIO= 2500 where DNI=123456;``| USUARIOB04	ABD04	EMPLEADOS	UPDATE	10/02/22	2004 |
|``update ABDn.DEPTOS set PRESUPUESTO= 88888 where NOMBRE='ATC';``| - |

En las consultas hechas desde  USUARIOA04 no se van a dejar trazas ya que se uso la opcion __WHENEVER SUCCESSFUL__, esto quiere decir q la traza solo se creara cuando la consulta termine correctamente.

En el caso de USUARIOB04 hay algunas trazan que no aparecen a pesar de tener la opcion __WHENEVER NOT SUCCESSFUL__ y haber fallado en su ejecucion. Esto puede suceder pq la tabla a la que se intenta hacer un update no existe por lo q no se puede crear una traza a algo inexistente.

4. __Transacciones y auditorías.__ Desde la sesión de ABDn definimos una auditoría sobre un
objeto concreto, la tabla EMPLEADOS mediante la sentencia:

``AUDIT INSERT ON ABDn.EMPLEADOS BY ACCESS WHENEVER SUCCESSFUL;``

Comprobar en ``SYS.DBA_OBJ_AUDIT_OPTS con OWNER='ABDn'`` su activación

| Object Name   | Object Type  | Del   |    Ins  | Sel  | Upd  |   
|---|---|---|---|---|---|
| EMPLEADOS  | TABLE  | -/-  | A/-  | -/-  | -/-  |

- Otorga el privilegio necesario a USARIOAn para que pueda insertar en la tabla
EMPLEADOS. Comprobar qué ocurre si realizamos los siguientes INSERT en la tabla
ABDn.EMPLEADOS (abriendo sesión para USUARIOAn) cumpliéndose que una está y
otra no:

```
INSERT INTO ABDn.EMPLEADOS VALUES (123456, 1000, 'LSI'); (tupla existente)
INSERT INTO ABDn.EMPLEADOS VALUES (222222, 1000, 'ATC'); (tupla inexistente)
```

Comprueba con SYS.DBA_AUDIT_TRAIL (con OWNER = 'ABDn') cuántos registros se
obtienen y explica por qué.

|Comandos|Registro auditoria|
|--|--|
| -|USERNAME OWNER OBJ_NAME ACTION_NAME TIMESTAMP RETURNCODE|
|``INSERT INTO ABDn.EMPLEADOS VALUES (123456, 1000, 'LSI');``| - |
|``INSERT INTO ABDn.EMPLEADOS VALUES (222222, 1000, 'ATC'); ``| USUARIOA04	ABD04	EMPLEADOS	INSERT	10/02/22	0 |

Al ser __WHENEVER SUCCESSFULL__ solo se creara una traza cuando la consulta sea complete correctamente.

* Abre una sesión para USUARIOAn y realiza las siguientes transacciones insertando
nuevas tuplas en la tabla.  

```
INSERT INTO ABDn.EMPLEADOS VALUES (111111, 2000, 'LSI');
COMMIT;
INSERT INTO ABDn.EMPLEADOS VALUES (333333, 3000, 'ATC');
ROLLBACK;
```
|Comandos|Registro de auditoría|
|--|--|
|``INSERT INTO ABDn.EMPLEADOS VALUES (111111, 2000, 'LSI'); COMMIT; INSERT INTO ABDn.EMPLEADOS VALUES (333333, 3000, 'ATC'); ROLLBACK;`` |USUARIOA04	ABD04	EMPLEADOS	INSERT	10/02/22	0	 |

5. 

6. 

7. __Auditorias de sentencia.__ Otorgar a USUARIOAn los privilegios que le permitan realizar
CREATE USER y ALTER USER. Definir un auditoría para las sentencias de usuario desde la
sesión de ABDn mediante

``AUDIT USER BY USUARIOAn``

|Registros|
|--|
| USUARIOA04		USUARIOD04	DROP USER	14/02/22	0 |
| USUARIOA04		USUARIOD04	DROP USER	14/02/22	1031|
|USUARIOA04		USUARIOD04	ALTER USER	14/02/22	0|
| USUARIOA04		USUARIOD04	CREATE USER	14/02/22	0  |

Como podemos ver se han creado 4 registros al auditar en USER, con esto podemos intuir esta auditoria reacciona a las acciones de crear, borrar y editar un usuario.

Supón  que  queremos  eliminar  la  auditoría  sobre  ALTER  USER  y  mantener  las  de  DROP  
USER y CREATE USER. ¿Es posible?

No seria posible, ya que no hay una accion de auditoria q solo incluya el drop y el creat. La unica opcion que tenemos es auditar sobre user y este reacciona a las acciones de crear, borrar y editar un usuario








