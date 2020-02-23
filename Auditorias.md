# Práctica de Auditorías de Base de Datos

Vamos a realizar una seria ejercicios para practicar algunos conceptos básicos de auditoría sobre una base de datos 
Oracle:

1. Activa desde SQL*Plus la auditoría de los intentos de acceso fallidos al sistema. Comprueba su funcionamiento.

2. Realiza un procedimiento en PL/SQL que te muestre los accesos fallidos junto con el motivo de los mismos, transformando el código de error almacenado en un mensaje de texto comprensible.

3. Activa la auditoría de las operaciones DML realizadas por SCOTT. Comprueba su funcionamiento.

4. Realiza una auditoría de grano fino para almacenar información sobre la inserción de empleados del departamento 10 en la tabla emp de scott.

5. Explica la diferencia entre auditar una operación by access o by session.

6. Documenta las diferencias entre los valores db y db, extended del parámetro audit_trail de ORACLE. Demuéstralas poniendo un ejemplo de la información sobre una operación concreta recopilada con cada uno de ellos.

7. Localiza en Enterprise Manager las posibilidades para realizar una auditoría e intenta repetir con dicha herramienta los apartados 1, 3 y 4.

8. Averigua si en Postgres se pueden realizar los apartados 1, 3 y 4. Si es así, documenta el proceso adecuadamente.

9. Averigua si en MySQL se pueden realizar los apartados 1, 3 y 4. Si es así, documenta el proceso adecuadamente.

10. Averigua las posibilidades que ofrece MongoDB para auditar los cambios que va sufriendo un documento.

11. Averigua si en MongoDB se pueden auditar los accesos al sistema.
