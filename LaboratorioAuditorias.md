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

|  USUARIOA04 | USUARIOA04   |
|---|---|
|``update ABDn.DEPARTAMENTOS set PRESUPUESTO=99999 where NOMBRE='LSI';`` | ``update ABDn.DEPARTAMENTOS set PRESUPUESTO=88888 where NOMBRE='ATC'; ``|
``update ABDn.DEPARTAMENTOS set PRESU= 99999 where NOMBRE='LSI';`` | ``update ABDn.DEPARTAMENTOS set PRESU= 88888 where NOMBRE='ATC';``   |
| | ``update into ABDn.DEPARTAMENTOS set PRESUPUESTO= 88888 where NOMBRE='ATC';`` |
| | ``update ABDn.EMPLEADOS set SALARIO= 2500 where DNI=123456;``  |
| | ``update ABDn.DEPTOS set PRESUPUESTO= 88888 where NOMBRE='ATC';`` |

- Comprueba con SYS.DBA_AUDIT_TRAIL (con WHERE OWNER = 'ABDn') qué registros se
obtienen y explica por qué.

| Comandos  | Registros de auditorias  |
|---|:---:|
|  - |  USERNAME OWNER OBJ_NAME ACTION_NAME TIMESTAMP RETURNCODE |
