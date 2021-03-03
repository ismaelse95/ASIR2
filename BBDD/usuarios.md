# Gestión de usuarios

## Oracle

1. Realiza un procedimiento llamado MostrarObjetosAccesibles que reciba un nombre de usuario y
muestre todos los objetos a los que tiene acceso.

Vamos a crear el procedimiento.
~~~
CREATE OR REPLACE PROCEDURE MostrarObjetosAccesibles(p_usuario VARCHAR2)
is
    v_privilegio    VARCHAR2(100);
    v_tabla         VARCHAR2(100);
    v_propietario   VARCHAR2(100);
    cursor c_cursor is
    SELECT table_name, privilege, owner
    FROM dba_tab_privs
    WHERE grantee=p_usuario;
    v_cursor c_cursor%ROWTYPE;
begin
	for v_cursor in c_cursor loop
		v_tabla:=v_cursor.table_name;
		v_privilegio:=v_cursor.privilege;
		v_propietario:=v_cursor.owner;
    	dbms_output.put_line('Nombre de la tabla: '||v_tabla||' Privilegio: '||v_privilegio||' Propietario: '||v_propietario);
    end loop;
end;
/
~~~

2. Realiza un procedimiento que reciba un nombre de usuario, un privilegio y un objeto y nos muestre
el mensaje 'SI, DIRECTO' si el usuario tiene ese privilegio sobre objeto concedido directamente, 'SI,
POR ROL' si el usuario lo tiene en alguno de los roles que tiene concedidos y un 'NO' si el usuario
no tiene dicho privilegio.

~~~
create or replace procedure PrivilegiosUsuarios(p_usuario VARCHAR2, p_privilegio VARCHAR2, p_objeto VARCHAR2)
is
    v_contador number:=0;
begin
    SELECT count(*) into v_contador
    FROM dba_tab_privs
    WHERE grantee=upper(p_usuario)
    and privilege=upper(p_privilegio)
    and table_name=upper(p_objeto);
    if v_contador=0 then
        PrivilegiosRol(p_usuario,p_privilegio,p_objeto);
    else
        dbms_output.put_line('SI, DIRECTO');
    end if;
end;
/

create or replace procedure PrivilegiosRol(p_usuario VARCHAR2, p_privilegio VARCHAR2, p_objeto VARCHAR2)
is
    v_contador  number:=0;
begin
    SELECT count(*) into v_contador
    FROM dba_role_privs
    WHERE grantee=upper(p_usuario)
    and granted_role in (SELECT role
                         FROM role_tab_privs
                         WHERE privilege=upper(p_privilegio)
                         and table_name=upper(p_objeto));
    if v_contador=0 then
        dbms_output.put_line('NO');
    else
        dbms_output.put_line('SI, POR ROL');
    end if;
end;
/
~~~

3. (ORACLE, Postgres, MySQL) Escribe una consulta que obtenga un script para quitar el privilegio de
borrar registros en alguna tabla de SCOTT a los usuarios que lo tengan.

4. (ORACLE) Crea un tablespace TS2 con tamaño de extensión de 256K. Realiza una consulta que
genere un script que asigne ese tablespace como tablespace por defecto a los usuarios que no
tienen privilegios para consultar ninguna tabla de SCOTT, excepto a SYSTEM.

5. (ORACLE) La vida de un DBA es dura. Tras pedirlo insistentemente, en tu empresa han contratado
una persona para ayudarte. Decides que se encargará de las siguientes tareas:
- Resetear los archivos de log en caso de necesidad.
- Crear funciones de complejidad de contraseña y asignárselas a usuarios.
- Eliminar la información de rollback. (este privilegio podrá pasarlo a quien quiera)
- Modificar información existente en la tabla dept del usuario scott. (este privilegio podrá
pasarlo a quien quiera)
- Realizar pruebas de todos los procedimientos existentes en la base de datos.
- Poner un tablespace fuera de línea.
Crea un usuario llamado Ayudante y, sin usar los roles predefinidos de ORACLE, dale los
privilegios mínimos para que pueda resolver dichas tareas.
Pista: Si no recuerdas el nombre de un privilegio, puedes buscarlo en el diccionario de datos.

6. (ORACLE) Muestra el texto de la última sentencia SQL que se ejecuto en el servidor, junto con el
número de veces que se ha ejecutado desde que se cargó en el Shared Pool y el tiempo de CPU
empleado en su ejecución.

~~~
Select distinct sql_text,executions,CPU_TIME
from v$sqlarea
order by first_load_TIME desc
fetch first 1 rows only;
~~~

7. (ORACLE) Realiza un procedimiento que genere un script que cree un rol conteniendo todos los
permisos que tenga el usuario cuyo nombre reciba como parámetro, le hayan sido asignados a
aquél directamente o a traves de roles. El nuevo rol deberá llamarse BackupPrivsNombreUsuario.

## MongoDB

8. Averigua si existe la posibilidad en MongoDB de limitar el acceso de un usuario a los datos de una
colección determinada.

Podemos limitar el acceso creando un usuario y dándole el rol de lectura. Para ello crearemos el usuario de la siguiente forma.
~~~
> db.createUser(
... {
... user: "Prueba",
... pwd: "Prueba",
... roles:[{role: "read", db: "eolicas"}]
... })
~~~

9. Averigua si en MongoDB existe el concepto de privilegio del sistema y muestra las diferencias más
importantes con ORACLE.

En MongoDB el concepto privilegio del sistema depende del rol que tenga cada usuario.

Las diferencias que hay con Oracle es a la hora de definir cada concepto.

**MongoDB - Oracle**
db.createCollection()	- create table
db.createUser()	- create user
db.dropRole()	- drop role
db.startSession()	- create session
db.createView()	- create view
db.dropUser()	- drop user

10. Explica los roles por defecto que incorpora MongoDB y como se asignan a los usuarios.

Roles de administración de usuarios:
- read: Permite la lectura en la base de datos (find, listar índices, coleccione)(Sobre una base de datos).
- readWrite: Lectura y escritura (creación y borrado de índices, drop, remove, update)(Sobre una base de datos).
- readWriteAnyDatabase: Lectura y escritura en todas las bases de datos.
- readAnyDatabase: Lectura en todas las bases de datos.

Roles de administración de base de datos sobre la base de datos que se asigne:
- dbAdmin: Permite la gestión sobre las bases de datos system.indexes, system.namespaces y system.profile y además permiso de creación borrado de índices y colecciones en cualquier base de datos (pero no permiso de lectura).
- dbOwer: Podremos realizar cualquier acción administrativa en la base de datos.
- userAdmin: Manejo de usuarios, prermite crear, eliminar usuarios y roles. También puede otorgar privilegios.

Roles de administración de base de datos sobre todas las bases de datos:
- dbAdminAnyDatabase: Podremos gestionar los datos pero no se podrá acceder a la información sobre los usuarios.
- userAdminAnyDatabase: Podremos crear, eliminar y modificar roles o usuarios.

Roles de administración de cluster:
- clusterMonitor: Lectura sobre las herramientas de monitorización.
- hostManager: Puede modificar y administrar servidores.
- clusterManager: Manejo y monitorización del cluseter. Puede acceder a las bases de datos config y local.
- clusterAdmin: Permisos de clusterManager, clusterMonitor, hostManager y también podremos usar la acción dropDatabase.

Roles de Copia y restauración:
- backup: Privilegios para hacer backups (colección admin.mms.backup).
- restore: permiso para la recuperación de backups (mongorestore sin opción --oplogReplay).

Roles de super usuario:
- root: Combina readWriteAnyDatabase, dbAdminAnyDatabase, userAdminAnyDatabase, clusterAdmin, y restore (Rol de super usuario).

Cuando creamos un usuario tiene privilegios para crear/borrar usuarios, pero no para leer o escribir en las bases de datos. Para por ejemplo crear un usuario que si pueda acceder a cualquier base de datos lo crearemos de la siguiente forma.
~~~
db.createUser({user:'prueba', pwd:'prueba', roles: ['readWriteAnyDatabase']})
~~~

11. Explica como puede consultarse el diccionario de datos de MongoDB para saber que roles han sido
concedidos a un usuario y qué privilegios incluyen.

Para consultar los roles de un usuario podremos verlo con show users.
~~~
{
	"_id" : "admin.prueba",
	"userId" : UUID("7a985d2b-adc2-4a02-9d9d-cdadfcc2ee4d"),
	"user" : "prueba",
	"db" : "admin",
	"roles" : [
		{
			"role" : "read",
			"db" : "admin"
		}
	],
	"mechanisms" : [
		"SCRAM-SHA-1",
		"SCRAM-SHA-256"
	]
}
~~~

Para ver los privilegios que incluyen.
~~~
db.getRole("readWrite", {showPrivileges: true})
~~~
