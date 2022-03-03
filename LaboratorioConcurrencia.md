# Laboratorio tema Concurrencia 

__1. Preliminares__
* Abre una sesión con tu usuario ABDn. 
* Utilizaremos  los  dos  usuarios  creados  en  el  primer  laboratorio,  usuarioAn  y  usuarioBn.  
Los dos con el role ABDUSER. 
* Abrir una sesión para dichos usuarios. 
* Obtener  los  identificadores  de  sesión  de  las  dos  sesiones.  En  nuestro  caso  los  usuarios   son  usuarioAn  y  usuarioBn  (recuerda  que  los  nombres  de  los  usuarios  siempre  van  en  mayúsculas)

USUARIOA04:
|Username|Sid (Session id)|
|-|-|
|USUARIOA04|189|

USUARIOB04:

|Username|Sid (Session id)|
|-|-|
|USUARIOB04|303|

__2. Control de la concurrencia__

En la sesión del usuarioAn:
- Crear la tabla CUENTA según la siguiente especificación e insertar dos tuplas en la tabla 
según las instrucciones siguientes:

```
CREATE TABLE CUENTA (numero number primary key, saldo number); 
INSERT INTO CUENTA VALUES (11,1000); 
INSERT INTO CUENTA VALUES (22,2000); 
COMMIT; 
```
- Modificar el saldo de la cuenta número 11:

``
UPDATE CUENTA SET saldo = 9000 WHERE Numero = 11; (no hacer commit) 
``

- Comprobar que se ha modificado 
- En  la  sesión  del  usuario  ABDn  obtener  información  sobre  bloqueos  para  la  sesión  del usuarioAn.

```
SELECT sid, id1, id2, lmode, request, type, ctime, block 
FROM V$LOCK 
WHERE sid=número de sesión;
```

|SID|ID1|ID2|LMODE|REQUEST|TYPE|CTIME|BLOCK|
|-|-|-|:-:|:-:|-|-|-|
|189|1427337|0|3|0|TM|170|0|
|189|133|0|4|0|AE|595|0|
|189|196623|2161486|6|0|TX|170|0|

- Desde la sesión del usuarioAn asignar al usuarioBn los privilegios de lectura y 
modificación sobre la tabla CUENTA

- En  la  sesión  del  usuario  ABDn  obtener  de  nuevo  información  sobre  bloqueos  para  la  sesión del usuarioAn. ¿Qué ha ocurrido?

|SID|ID1|ID2|LMODE|REQUEST|TYPE|CTIME|BLOCK|
|-|-|-|:-:|:-:|-|-|-|
|189|	133|	0|	4|	0|	AE|	1015|	0|

- Modificar de nuevo el saldo de la cuenta número 11 desde la sesión del usuarioAn

`UPDATE CUENTA SET saldo = 9100 WHERE Numero = 11; (sin commit) `

- Comprobar qué se ha modificado.

- En  la  sesión  del  usuario  ABDn  obtener  de  nuevo  información  sobre  bloqueos  para  la  sesión del usuarioAn. 


|SID|ID1|ID2|LMODE|REQUEST|TYPE|CTIME|BLOCK|
|-|-|-|:-:|:-:|-|-|-|
|189|	133|	0|	4|	0|	AE|	1380|	0|
|189|	720916|	2286173|	6|	0|	TX|	13|	0|
|189|	1427337	|0|	3|	0|	TM|	13	|0|


En la sesión del usuarioBn:
- Obtener los valores de la tabla CUENTA del usuarioAn.

SELECT * from usuarioAn.Cuenta;

- ¿Se  podrá  modificar  el  valor  de  saldo  de  la  cuenta  número  11  con  la  siguiente instrucción? ¿Por qué?

`UPDATE usuarioan.CUENTA SET saldo = 5000 WHERE Numero = 11; (sin commit)`

No deberia de poderse hacer este update debiado a q USUARIOA04 todavia tiene bloqueada la tabla CUENTA junto a la tubla donde el numero es 11. Por lo que la operacion quedara bloqueada hasta q USUARIOA04 la desbloque.
```
SELECT sid, id1, id2, lmode, request, type, ctime, block 
FROM V$LOCK 
WHERE sid=número de sesión de cada usuario;
```


USUARIOA04:
|SID|ID1|ID2|LMODE|REQUEST|TYPE|CTIME|BLOCK|
|-|-|-|:-:|:-:|-|-|-|
|189|	133|	0|	4|	0|	AE|	246|	0|
|189|	1427337|	0|	3|	0|	TM|	193|	0|
|189|	196609|	2162898|	6|	0|	TX|	193|	1|

USUARIOB04:

|SID|ID1|ID2|LMODE|REQUEST|TYPE|CTIME|BLOCK|
|-|-|-|:-:|:-:|-|-|-|
|303|	133|	0|	4|	0|	AE|	90|	0|
|303|	1427337|	0|	3|	0|	TM|	27|	0|
|303|	196609|	2162898|	0|	6|	TX|	27|	0|

También puedes consultar en la vista v$session qué sesión está siendo bloqueada y por cuál 
```
SELECT sid, username, blocking_session 
FROM V$SESSION 
WHERE sid=número de sesión; 
```

|SID|USERNAME|BLOCKING_SESSION|
|-|-|:-:|
|303|	USUARIOB04|	189|

Podemos observar que USUARIOB04 esta siendo bloqueado por USUARIOA04

- Para ver qué sentencia está bloqueada:

```
SELECT SQL_TEXT 
FROM v$SQLTEXT a, v$SESSION b  
WHERE a.address = b.sql_address and b.sid = numero de sesión;
```

|SLQ TEXT|
|-|
|UPDATE USUARIOA04.CUENTA SET saldo = 5000 WHERE Numero = 11|

- Solucionar el problema haciendo commit en la sesión del usuarioAn y consultar de nuevo  la vista V$LOCK.

`COMMIT;`

- Ver el contenido de CUENTA desde las sesiones de los dos usuarios. ¿Es igual en las dos  sesiones? 

No es igual, este se puede deber a que USUARIOB04 no ha realizado un commit todavia.

- Termina la transacción del usuarioBn y mira de nuevo el contenido de CUENTA desde las 
sesiones de los dos usuarios

Explica brevemente qué ha pasado e interpreta por qué los valores de la tabla CUENTA son 
los que son para cada usuario y por qué responde o no el sistema (según haya sido el caso).

USUARIOA04  independientemente de los commits siempre vera el ultimo valor asigando, en el caso de los otros usarios del sistem estos no veran el cambio hasta acabar la transaccion. El sistema a sido bloqueado cuando USUARIOB04 a intentado hacer un update. Esto ha pasado debia a que la tupla a actualizar seguia bloqueada por otro usuario, hasta q este no acabo la sensentica permanecio bloqueada.


En la sesión del usuarioAn
- Modificar el valor de saldo de la tupla con número = 11
`UPDATE CUENTA SET saldo = 8500 WHERE Numero = 11; (sin commit)`

En la sesión del usuarioBn
- Modificar el valor de saldo de la tupla con número = 22. ¿Qué ocurre ahora? Interpretar el resultado.

Al hacer update a una tupla que no ejerce un conflicto debido a que esta no esta bloqueada, la sensentencia no se bloquea y procede a actualizara el valor para la tupla con el numero 22.

Mira  el  contenido  de  la  tabla  CUENTA  en  las  sesiones  de  los  dos  usuarios  usuarioAn  y   usuarioBn  antes  y  después  de  hacer  el  commit.  Observa  también  los  cambios  de  la  tabla  
V$LOCK

Explica brevemente qué ha pasado e interpreta por qué los valores de la tabla CUENTA son 
los que son para cada usuario y por qué responde o no el sistema (según haya sido el caso).


USUARIOA04 y USUARIOB04 no veran los cambioes en la tuplas hasta q el otro haga el commit, ya que que la sentencia no termina hasta q se haga un commit.

__3.Provocar un interbloqueo__

- En la sesión del usuarioAn

`UPDATE CUENTA SET saldo = 5550 WHERE Numero = 11; (sin commit) `
- En la sesión del usuarioBn

`UPDATE usuarioan.CUENTA SET saldo = 3333 WHERE Numero = 22; (sin commit) `
- En la sesión del usuarioAn

`UPDATE CUENTA SET saldo = 3456 WHERE Numero = 22; (sin commit) `
- En la sesión del usuarioBn

`UPDATE usuarioan.CUENTA SET saldo = 1111 Where Numero = 11; (sin commit) `
- En la sesión del usuario ABDn consulta la vista V$LOCK y también la espera de las sesiones 
de cada usuario mediante la instrucción: 
```
SELECT sid, seconds_in_wait, event 
FROM V$session_wait 
WHERE sid=numerosesión de usuario; 
```

USUARIOA04:

|SID|SECONDS IN TIME| EVENT
|-|:-:|-|
|189|	81|	SQL*Net message from client|

USUARIOB04:
|SID|SECONDS IN TIME| EVENT
|-|:-:|-|
|303|	124|	enq: TX - row lock contention|

Explica qué ha pasado y por qué responde o no el sistema (según haya sido el caso).

Primero USUARIOA04 no ha terminado la transaccion de update del numero 11, lo mismo con el numero 22 desde USUARIOB04. Entonces cuando USUARIOA04  a intentado hacer un update al numero 22 y USUARIOB04 al numero 11, eston 2 han quedado totalmente bloqueados por completo llegando al punto de un deadlock.

__4. Bloqueo de intención.__

En la sesión del usuarioAn realiza un 

`SELECT FOR UPDATE SELECT * FROM CUENTA WHERE numero=11 FOR UPDATE;` 
En la sesión del usuarioBn 
- ¿Se podrá ver el contenido de la tabla CUENTA?  
Si, ya que el select no interfiere con el update
- ¿Se podrá modificar alguna tupla? ¿Por qué? 

Podriamos modificar una tupla que no interfiriera con la de arriba, es decir todas menos la que contiene el numero 11.

En la sesión del usuario ABDn consulta la vista V$LOCK. 

|SID|ID1|ID2|LMODE|REQUEST|TYPE|CTIME|BLOCK|
|-|-|-|:-:|:-:|-|-|-|
|189|	133|	0|	4|	0|	AE|	3032|	0|
|189|	131074|	2198869|	6|	0|	TX|	1165|	|0|
|189|	1427337|	0|	3|	0|	TM|	1165|	0|

Explica  brevemente  qué  ha  ido  pasando  y  por  qué  responde  o  no  el  sistema  (según  haya  
sido el caso).