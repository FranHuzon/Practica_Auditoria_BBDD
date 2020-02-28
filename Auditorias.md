# Práctica de Auditorías de Base de Datos

Vamos a realizar una seria ejercicios para practicar algunos conceptos básicos de auditoría sobre una base de datos 
Oracle:

Primero vamos a comprobar si están avtivas las auditorías para nuestra base de datos:

`SQL> show parameter audit;`

![foto1](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria1.png)

El valor del parámetro audit_trail es el que indica si esta activada o no. En nuestro caso,"db" indica que la auditoría esta activa y los datos se almacenarán en la base de datos de Oracle.

En caso de que el parámetro esté en "none", para activar la auditoría usamos:

`ALTER SYSTEM SET audit_trail={valor} scope=spfile;`

Los distintos valores vienen más detallados [aquí](https://docs.oracle.com/cd/E11882_01/server.112/e40402/initparams017.htm#REFRN10006). Más información sobre el parámetro [scope](https://docs.oracle.com/cd/E11882_01/server.112/e40402/initparams004.htm#REFRN00102)

## Ejercicios

1. Activa desde SQLPlus la auditoría de los intentos de acceso fallidos al sistema. Comprueba su funcionamiento.

Luego, si queremos activar la auditoría de los accesos fallidos a la base de datos usamos el comando:

`audit session whenever not successful;`

Vamos a ejecutar este comando como *sys* y conectaremos de forma errónea con el usuario *fran* para comprobar la auditoría:

```
SQL> audit session whenever not successful;
Auditoría terminada correctamente.
```

```
SQL> connect fran
Introduzca la contraseña: 
ERROR:
ORA-01017: invalid username/password; logon denied
```

```
SQL> Select username,action_name,returncode,extended_timestamp from dba_audit_session where username = 'FRAN' order by extended_timestamp;
```

![foto2](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria2.png)


2. Realiza un procedimiento en PL/SQL que te muestre los accesos fallidos junto con el motivo de los mismos, transformando el código de error almacenado en un mensaje de texto comprensible.

```
create or replace function traducecodigo(p_codigo number)
return varchar2
is
    traduccion varchar2(100):='';
begin
    case p_codigo
        when 1017 then 
            traduccion:='Fallo al introducir la contraseña. Acceso denegado.';
        when 28000 then
            traduccion:='Superado numero de intentos fallidos para el inicio de sesion. La cuenta esta bloqueada';
        when 2004 then
            traduccion:='Error. Violacion de seguridad';
        else
            traduccion:='Error no traducido';
    end case;
return traduccion;
end;
/

create or replace procedure errorestraducidos
is
    cursor c_datos is 
    select username,action_name,returncode,extended_timestamp
    from dba_audit_session 
    where action_name = 'LOGON' 
    and returncode != 0
    order by extended_timestamp;

    v_traduccion varchar2(100);
begin
    dbms_output.put_line('Listado de accesos fallidos'||chr(10));
    dbms_output.put_line('Usuario'||' - '||'Accion'||' - '||'Fecha'||' - '||'Error'||chr(10));
    for i in c_datos loop
        v_traduccion:= traducecodigo(i.returncode);
        dbms_output.put_line(i.username||' - '||i.action_name||' - '||i.extended_timestamp||' - '||v_traduccion||chr(10));
    end loop; 
end;
/
```

![foto3](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria3.png)


3. Activa la auditoría de las operaciones DML realizadas por SCOTT. Comprueba su funcionamiento.

Activamos la auditoria desde un usuario con privilegios:

`AUDIT SELECT TABLE, INSERT TABLE, UPDATE TABLE, DELETE TABLE BY SCOTT;`

![foto4](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria4.png)

Realizamos acciones DML con el usuario SCOTT:

![foto5](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria5.png)

Vemos los registros de la auditoria:

`select object_name,action_name,extended_timestamp from dba_audit_object where owner='SCOTT' order by extended_timestamp;`

![foto6](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria6.png)

4. Realiza una auditoría de grano fino para almacenar información sobre la inserción de empleados del departamento 10 en la tabla emp de scott.

Para crear esta auditoría de una columna específica no se puede usar la auditoría unificada de ORACLE 12c, para ello debemos complementarla con FGA, "Fine-Grained-Auditing". 

Para crear la política de la auditoría usamos:

```
BEGIN
    DBMS_FGA.ADD_POLICY (
        object_schema      =>  'SCOTT',
        object_name        =>  'EMP',
        policy_name        =>  'nuevos_empleados_dept_10',
        audit_condition    =>  'DEPTNO = 10',
        statement_types    =>  'INSERT'
    );
END;
/
```

En la imagen vemos la creación de la política y si se ha creado correctamente en la vista dba_audit_policies

![foto7](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria7.png)

Ahora conectamos con el usuario SCOTT en insertamos un nuevo empleado en la tabla EMP y que pertenezca al departamento 10:

`INSERT INTO emp VALUES(6894,'FRAN','SEGURIDAD',null,sysdate,1500,0,10);`

Nos conectamos como SYS y comprobamos los registros de la auditoria de grano fino:

![foto8](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria8.png)

Aquí dejo un enlace con más información sobre las [auditorías de grano fino](https://docs.oracle.com/database/121/ARPLS/d_fga.htm#ARPLS015).

5. Explica la diferencia entre auditar una operación by access o by session.

Las auditorías pueden realizarse por acceso o por sesión, vamos a explicarlo:

**BY SESSION**: Provoca que Oracle inserte un único registro resumen en la tabla de auditoría aunque la sentencia se ejecute varias veces en la misma sesión.

**BY ACCESS**:  Provoca la escritura de un registro en las tablas de auditoría cada vez que la sentencia se ejecuta. Cuando se especifican auditorías de sentencias DDL o de privilegios del sistema, la auditoría por defecto es por accesos. Cuando se auditan sobre objetos o sentencias DML, la auditoría por defecto es por sesión.

Vamos a ver un ejemplo by access:

Creación de la auditoría:

![foto9](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria9.png)

Acciones con SCOTT:

![foto10](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria10.png)

Comprobación de registros de la auditoría:

![foto11](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria11.png)

Vamos a ver un ejemplo by session:

Creación de la auditoría:

![foto12](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria12.png)

Acciones con SCOTT:

![foto13](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria13.png)

Comprobación de registros de la auditoría:

![foto14](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria14.png)

6. Documenta las diferencias entre los valores db y db, extended del parámetro audit_trail de ORACLE. Demuéstralas poniendo un ejemplo de la información sobre una operación concreta recopilada con cada uno de ellos.

Ambos valores hacen los mismo, registran la auditoría en una tabla de la base de datos, solo que db,extended rellena los campos SQLtext y SQLbind del parámetro audit_trail, dándonos más información. Vamos a ver unos ejemplos.

Por defecto, la base de datos se monta con el valor db. Creamos la auditoría:

`AUDIT CREATE TABLE BY scott BY ACCESS;`

Creamos una tabla con el usuario scott:

`create table scott(id VARCHAR2(1));`

Miramos el registro:

`select username,action_name,extended_timestamp,sql_text,sql_bind from dba_audit_trail where username = 'SCOTT' order by extended_timestamp;`

![foto15](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria15.png)

Ahora, un ejemplo con el valor db, extended:

Modificamos el sistema para que use ese valor en la auditoría:

`alter system set AUDIT_TRAIL=db, extended scope=spfile;`

![foto16](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria16.png)

Para que surta efecto el cambio, debemos realizar un reinicio de la base de datos:

`SQL> shutdown`

`SQL> startup`

Ahora, repetimos el proceso anterior de creación de una tabla con el usuario scott, y vemos el registro de la auditoría:

![foto17](https://github.com/FranHuzon/Practica_Auditoria_BBDD/blob/master/images/auditoria17.png)


**NOTA**: A diferencia de la opción db, también existe la opción xml, que guarda los registros de la auditoría en un fichero xml. En el ejercicio 1 se dejó un enlace a más información sobre estos valores.

7. Localiza en Enterprise Manager las posibilidades para realizar una auditoría e intenta repetir con dicha herramienta los apartados 1, 3 y 4.

8. Averigua si en Postgres se pueden realizar los apartados 1, 3 y 4. Si es así, documenta el proceso adecuadamente.

9. Averigua si en MySQL se pueden realizar los apartados 1, 3 y 4. Si es así, documenta el proceso adecuadamente.

10. Averigua las posibilidades que ofrece MongoDB para auditar los cambios que va sufriendo un documento.

11. Averigua si en MongoDB se pueden auditar los accesos al sistema.
