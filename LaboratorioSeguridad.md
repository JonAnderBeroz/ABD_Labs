## Laboratorio tema seguridad 

1. Creación de cuenta s para usuarios. Desde la sesión de ABDn crea dos usuarios
USUARIOAn, USUARIOB n donde n se corresponde con el de tu usuario ABD n. Usa la
contraseña del usuario que decidas y únicamente especifica el tablespace por defecto, que
será USERS.


``
	create user USUARIOA04
	identified by admin;
``

``
	create user USUARIOB04
	identified by admin;
``

- Abrir otra sesión diferente con el usuario recién creado USUARIOAn. ¿Qué ocurre? Solucionar el problema para abrir una sesión para cada uno de los usuarios creados

A pesar de haber creado un usuario este no tendria que tener en principio privilegios para conectarse, por lo que el creador de estos deberia de darles el permiso de conexion. Con el siguiente comando:

``
Grant create session to [username];
``

2. Dar permisos para crear tablas. Intenta crear una tabla e introducir al menos una tupla
(en la sesión del USUARIOAn) tal y como se indica más abajo.

```
CREATE TABLE Asignaturas (nombre varchar2(20) primary key);
INSERT INTO Asignaturas VALUES ('nombre de la asignatura');
COMMIT;
```

- ¿Qué ocurre? Otorgar los permisos necesarios pa ra solucionar el problema y crear la tabla
con su tupla.

Lo que ocurriria en esta caso es que el nuevo usuario no podria ni crear ni insertar en la tabla, ya que a este no se la han dado permisos de create e insert. Para darles dichos permisos usariamos el siguiente comando:

``
GRANT CREATE TABLE TO USUARIOA04;
``

``
GRANT INSERT ON USUARIOA04.ASIGNATURAS TO USUARIOA04;   
``

Solo con esto no seria suficiente ya que al no asignarle una quota al usuario no tendria permisos para ocupar un espacio en tablas. Para solucionar esto usariamos el siguiente comando:

``
ALTER USER USUARIOA04 
QUOTA UNLIMITED ON USERS;
``

3. Otorgar permisos para utilizar tablas. Quién tiene derecho a dar los permisos.
Comprueba los permisos del USUARIOBn y otórgale los mismos que al USUARIOAn.


En este caso el unico que tendria derecho a dar los permisos seria la cuenta ABDn, ya que no se les dio el ``GRANT OPTION`` a los usuarios creados por este. ?

```
SELECT * FROM DBA_SYS_PRIVS WHERE GRANTEE ='USUARIOAN';
SELECT * FROM DBA_SYS_PRIVS WHERE GRANTEE ='USUARIOBN';
```

Para otorgar los permisos al usuario USUARIOBn cambiaremos el to de las sentencias previamente creadas.

- Crea otro usuario USUARIOCn y otórgale tambié n los mimos permisos (pero hazlo utilizando
la herramienta gráfica de SQL Developer).

- Abrir otra sesión para el USUARIOBn y crear otra tabla ASIGNATURAS para este USUARIOBn
e introducir tuplas. Intentar acceder a ella desde la sesión abierta por el USUARIOAn.
Para acceder a una tabla de otro usuario ha de ponerse como prefijo el nombre del
propietario del objeto.
    - ¿Qué ocurre? Soluciona el problema otorgando los permisos correspondientes desde la
sesión USUARIOBn.

	El USUARIOA04 al intentar hacer un select en la tabla asignaturas veria un error, ya que este no tendria permiso para  hacer un select sobre la tabla. Para solucionar este problema el USUARIOB04 o ADB04 daran permiso de select al USUARIOA04.

	``
	GRANT   SELECT ON  ASIGNATURAS to USUARIOA04;
	``

Comprobar que si el otro usuario no ha hecho el COMMIT sus modificaciones no son todavía
visibles.

Efectivamente otro Usuario no vera los cambios hasta q el otro usuario (propietario de la tabla) no haga COMMIT de sus modificaciones. Podemos usar lo siguiente para comprobarlo:

Supongamos que ya existe una tupla en la tabla con valor __"test"__

USUARIOB04: ``Insert into asignaturas values ('a');``

USUARIOA04: ``Select *
from USUARIOB04.ASIGNATURAS;`` ==> (En consola) test.

USUARIOB04: ``Insert into asignaturas values ('b');
COMMIT;``

USUARIOA04: ``Select *
from USUARIOB04.ASIGNATURAS;`` ==> (En consola) b a test.


4. Crear otra tabla en la sesión del USUARIOAn

```
CREATE TABLE Estudiantes (
dni number primary key,
asignaturafavorita REFERENCES USUARIOBn.Asignaturas)
```

¿Qué ocurre? Soluciona el problema otorga ndo los permisos correspondientes e inserta
alguna tupla para comprobar el correcto funcionamiento.

Esta consulta daria error ya que el USUARIOA04 no tiene permisos para referenciar la tabla asignaturas del USUARIOB04. Para solucionar esto usaremos el siguiente comando:

``GRANT REFERENCES  ON USUARIOB04.Asignaturas TO USUARIOA04;``

5. __Posibilidad de ampliar permisos.__
- Abrir una nueva sesión con el USUARIOCn. Este usuario quiere realizar tanto IN SERT
como SELECT sobre USUARIOBn.ASIGNATURAS. Supongamos que queremos que estos
privilegios se le otorguen a través del USUARIOAn. ¿Qué tendría que hacer el
USUARIOBn para que el USUARIOAn pueda propagar este privilegio? Hazlo y comprueba
los permisos otorgados.

Para que USUARIOA04 pueda propagar permisos sobre la tabla ASIGNATURAS del USUARIOB04 necesitariamos darle los permismos pero con el ``GRANT OPTION`` para q asi al usuario al q asignamos estos permisos los pueda propagar.

- Supongamos que queremos mantener el permiso del USUARIOAn para añadir filas a la
tabla, pero sin la ampliación concedida. Comprueba que no es posible realizar REVOKE
GRANT OPTION, sino que ha de hacerse en dos pasos: quitar el privilegio completo
y concederlo de nuevo, sin la ampliación.

Efectivamen si intentamos hacer un revoke de ``GRANT OPTION`` este da error por lo que usaremos los siguiente comandos para que el USUARIOA04 no pueda propagar los permisos:

``REVOKE INSERT,SELECT ON ASIGNATURAS FROM USUARIOA04;``

``GRANT INSERT,SELECT ON ASIGNATURAS TO USUARIOA04;``

6. __Información sobre privilegios.__

- También se pueden conceder privilegios sobre un atributo (columna). Pruébalo.	

``GRANT UPDATE (nombre) ON ASIGNATURAS to USUARIOA04;``

- Acceder a las tablas del __diccionario__ (desde la sesión del usuario ABDn).

	```
	SELECT * FROM SYS.DBA_TAB_PRIVS WHERE GRANTEE = 'USUARIOBN';
	SELECT * FROM SYS.DBA_SYS_PRIVS WHERE GRANTEE = 'USUARIOBN';
	SELECT * FROM SYS.DBA_TAB_PRIVS WHERE GRANTEE = 'USUARIOAN';
	SELECT * FROM SYS.DBA_COL_PRIVS WHERE GRANTEE='USUARIOAN';
	SELECT TABLE_NAME, PRIVILEGE, GRANTEE FROM SYS.DBA_TAB_PRIVS WHERE GRANTOR
	='USUARIOBN';
	SELECT PRIVILEGE FROM SYS.DBA_SYS_PRIVS WHERE GRANTEE = 'USUARIOBN';
	```
	(__grantor__ da los privilegios, __grantee__: recibe los privilegios)


- Puedes asignar otros privilegios para ver las modificaciones. Por ejemplo desde ABDn
concede permiso al USUARIOAn para poder acceder y eliminar en la tabla EMPLEADOS.

``GRANT SELECT, DELETE ON EMPLEADOS TO USUARIOA04;``

7. __Quitar permisos.__

- Comprobar si ORACLE utiliza la opción CASCADE por defecto. Qué ocurre si el
USUARIOBn retira el privileg io SELECT concedido al USUARIOAn. ¿Puede el USUARIOCn
acceder a la tabla USUARIOBn.ASIGNATURAS

Comando para quitarle permiso al USUARIOA04: ``REVOKE SELECT ON ASIGNATURAS FROM USUARIOA04;``

Al quitarle los permisos a USAURIOA04, USUARIOC04 tb perdio dichos privilegios por ende podriamos asumir que oracle usa por defecto la clausula ``CASCADE``.

- Podemos eliminar todos los permisos concedidos sobre un objeto con REVOKE ALL

```
ABDN: REVOKE ALL ON EMPLEADOS FROM USUARIOAn;
USUARIOBN: REVOKE ALL ON ASIGNATURAS FROM USUARIOAn;
```

¿Qué ha o currido? ¿Por qué? ¿Qué ocurre al incluir CASCADE CONSTRAINTS con la
restricción de integridad refe rencial definida en la tabla? Incluye una tupla que viole
dicha restricción.

El revoke hecho por ABDN no ha dado ninguna error, pero el de USUARIO04B ha dado error ya que unas de sus tables es referencia como __FOREIGN KEY__ en la tabla Estudiantes	 del USUARIOA04 asi que nos insta a usar ``CASCADE CONSTRAINTS`` para poder revokar todos los privilegios. Una  tupla que viole el uso referencial podria ser la siguiente:

``INSERT INTO estudiantes 	VALUES(3, "manolo");``

8. __Permisos para public (todo el mundo).__ En la sesión del USUARIOCn crear una nueva
tabla NOTAS (DNI, nota) con tuplas.

```
CREATE TABLE NOTAS(
DNI number primary key,
nota number(2)
);
COMMIT;
```
- Otorgar el privilegio de seleccionar sobre dicha tabla al USUARIOAn

``GRANT SELECT ON NOTAS TO USUARIOA04;``

- Otorgar el mismo privilegio a todos los usuarios. Comprobar si el USUARIOBn puede
acceder a dicha tabla.

``GRANT SELECT ON NOTAS TO PUBLIC;``

USUARIOB04 es capaz de hacer un select sobre el contenido de la tabla.

- Retirar dicho privilegio a todos los usuarios. ¿Qué ocurre con el acceso desde el
USUARIOAn?

``REVOKE SELECT ON NOTAS FROM PUBLIC;``

USUARIOA04 puede seguir viendo la tabla NOTAS, esto es devido a que el propio USUARIOC04 le doy los permisos, por ende, el ``REVOKE PUBLIC`` no tiene efecto sobre USUARIOA04.

9. __Crear grupos de usuarios y sus permisos, roles.__

- Crear un rol USUARIOn y otorgárselo a los usuarios USUARIOAn y USUARIOBn. ¿Podrías
crear un rol ABDn?

```
CREATE ROLE USUARIO04
```

```
ALTER USER USUARIOA04
DEFAULT ROLE USUARIO04;

ALTER USER USUARIOB04
DEFAULT ROLE USUARIO04;

ALTER USER USUARIOC04
DEFAULT ROLE USUARIO04;

COMMIT;
```
- Cuáles son los pasos a seguir para que los usuarios con role USUARIOn puedan acceder
a la tabla DEPARTAMENTOS. Hazlo y comprueba que los usuarios pueden hacer uso de
su permiso. Primero cierra la sesión o haz SET ROLE ALL o bien haz SET ROLE
USUARIOn en la sesión del USUARIOAn y USUARIOBn.

El primero paso para que los usuarios pudieras acceder a la tabla __DEPARTAMENTOS__

`` GRANT SELECT ON DEPARTAMENTOS TO USUARIO04;``

10. __Quitar permisos de objetos vs quitar permisos del sistema__

- Otorgar el privilegio CREATE TABLE siguiendo la siguiente secuencia:

	```
	ABDn: GRANT CREATE TABLE TO USUARIOAn WITH ADMIN OPTION;
	USUARIOAn: GRANT CREATE TABLE TO USUARIOBn WITH ADMIN OPTION;
	USUARIOBn: GRANT CREATE TABLE TO USUARIOCn WITH ADMIN OPTION;
	USUARIOBn: CREATE TABLE NUEVATABLA (CUANTOS NUMBER);
	USUARIOCn: REVOKE CREATE TABLE FROM USUARIOBn; 	
	```

	¿Qué ocurre? Tiene ahora el usuario USUARIOBn posibilidad de crear una nueva tabla
	TABLAB? 

	Despues de que USUARIOC04 le quitara los permisos de crear tablas a USUARIOB04 este no es capaz de crear la tabla TABLAB

- Crear una nueva tabla OTRA en el esquema del USUARIOAn. Otorgar el privilegio SELECT
siguiendo la siguiente secuencia:

	```
	USUARIOAn: GRANT SELECT ON OTRA TO USUARIOBn WITH GRANT OPTION;
	USUARIOBn: GRANT SELECT ON USUARIOAn.OTRA TO USUARIOCn WITH GRANT OPTION;
	USUARIOCn: REVOKE SELECT ON USUARIOAn.OTRA FROM USUARIOBn; 
	```
	¿Qué ocurre? Cuál es la diferencia con el apartado a? 

	En este caso el revoke darie error debido a que USUARIOC04 no ha otorgado el permiso de __SELECT__ a USUARIOB04.

11. __Eliminación de usuarios.__ Eliminar desde la sesión de ABDn el USUARIOCn (sus conexiones

``DROP USER USUARIOC04 CASCADE;``

12. __Crear un perfil.__ Desde la conexión ABDn obtén información de los usuarios a través de la
tabla DBA_USERS:

	```
	SELECT USERNAME, CREATED, ACCOUNT_STATUS, PROFILE FROM DBA_USERS
	WHERE DEFAULT_TABLESPACE='USERS';
	```

	Analiza la información obtenida. Por ejemplo, los usuarios tienen asignado el perfil DEFAULT.

	Comprueba que el usuario USUARIOBn puede abrir dos sesiones diferentes (después cierra
	las dos conexiones). Crear un nuevo perfil, PERFILBn, desde la conexión de ABDn, que
	permita a los usuarios abrir sólo una sesión (parámetro SESSIONS_PER_USER), y asignar
	dicho perfil al usuario USUARIOBn (puedes ver en DBA_USERS cómo ha cambiado). Ahora
	trata de abrir dos sesiones con USUARIOBn. ¿Qué ocurre?

Creacion de perfil:

```
CREATE PROFILE TEST LIMIT
SESSIONS_PER_USER 1;

COMMIT;
```

Atribucion del perfil:

```
ALTER USER USUARIOB04
PROFILE TEST;
```

Al intentar crear una 2da sesion con USUARIOB04 esta dara error ya que al asignarle el perfil TEST este no deja crear mas de una sesion por usuario.













