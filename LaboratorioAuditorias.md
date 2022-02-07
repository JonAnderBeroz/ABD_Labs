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