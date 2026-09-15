# Gestión de Usuarios en MySQL

La gestión de usuarios en MySQL permite controlar quién puede acceder al servidor, desde dónde puede conectarse y qué operaciones tiene autorizadas.

---

## 1. ¿Qué es un usuario en MySQL?

Un usuario es una cuenta que puede acceder al servidor MySQL.

Cada usuario se define mediante:

- **Nombre de usuario**
- **Host** desde el que puede conectarse

La estructura general es:

```text
'usuario'@'host'
```

> [!IMPORTANT]
> En MySQL, `admin@localhost` es distinto de `admin@'%'`.

```text
'admin'@'localhost'  → Solo puede conectarse desde el servidor local
'admin'@'%'          → Puede conectarse desde cualquier host
```

---

## 2. Crear usuarios

### Sintaxis básica

```sql
CREATE USER 'usuario'@'host'
IDENTIFIED BY 'contraseña';
```

### Ejemplo

```sql
CREATE USER 'maria'@'localhost'
IDENTIFIED BY 'segura123';
```

---

## 3. Privilegios y roles

Los privilegios determinan qué operaciones puede realizar un usuario en una base de datos.

### Asignar privilegios

```sql
GRANT SELECT, INSERT
ON base_datos.*
TO 'usuario'@'host';
```

### Ver privilegios

```sql
SHOW GRANTS
FOR 'usuario'@'host';
```

### Revocar privilegios

```sql
REVOKE INSERT
ON base_datos.*
FROM 'usuario'@'host';
```

---

## 4. Eliminar usuarios

```sql
DROP USER 'usuario'@'host';
```

### Ejemplo

```sql
DROP USER 'consulta'@'%';
```

---

## 5. Cambiar una contraseña

```sql
ALTER USER 'usuario'@'host'
IDENTIFIED BY 'nueva_contraseña';
```

### Ejemplo

```sql
ALTER USER 'maria'@'localhost'
IDENTIFIED BY 'nuevaClaveSegura';
```

---

## 6. Buenas prácticas

- Nunca utilizar el usuario `root` para tareas comunes.
- Asignar únicamente los privilegios mínimos necesarios.
- Utilizar contraseñas seguras y difíciles de adivinar.
- Evitar el acceso desde cualquier host cuando no sea imprescindible.
- Revisar y auditar los permisos periódicamente.
- Eliminar usuarios que ya no sean necesarios.
- Mantener actualizado el servidor MySQL.

---

# Actividad práctica

Realiza las siguientes tareas:

1. Crear dos usuarios:
   - `alumno1@localhost` con acceso completo a la base de datos `escuela`.
   - `consulta@'%'` con permisos de solo lectura sobre la base de datos `escuela`.
2. Verificar los permisos asignados.
3. Modificar la contraseña de `alumno1`.
4. Eliminar el usuario `consulta`.

---

# Scripts SQL de ejemplo

## Script 1: Crear una base de datos y usuarios

### Crear una base de datos de ejemplo

```sql
CREATE DATABASE IF NOT EXISTS escuela;
```

### Crear un usuario con todos los privilegios sobre `escuela`

```sql
CREATE USER 'alumno1'@'localhost'
IDENTIFIED BY 'alumno123';

GRANT ALL PRIVILEGES
ON escuela.*
TO 'alumno1'@'localhost';
```

### Crear un usuario con permisos de solo lectura

El usuario podrá conectarse desde cualquier host y únicamente podrá realizar consultas `SELECT`.

```sql
CREATE USER 'consulta'@'%'
IDENTIFIED BY 'lectura123';

GRANT SELECT
ON escuela.*
TO 'consulta'@'%';
```

---

## Script 2: Verificar privilegios

### Mostrar los privilegios de `alumno1`

```sql
SHOW GRANTS
FOR 'alumno1'@'localhost';
```

### Mostrar los privilegios de `consulta`

```sql
SHOW GRANTS
FOR 'consulta'@'%';
```

---

## Script 3: Modificar un usuario y su contraseña

### Cambiar la contraseña de `alumno1`

```sql
ALTER USER 'alumno1'@'localhost'
IDENTIFIED BY 'nuevoAlumno123';
```

### Alternativa para versiones antiguas

> [!WARNING]
> La función `PASSWORD()` puede no estar disponible o recomendada en versiones modernas de MySQL.

```sql
SET PASSWORD FOR 'alumno1'@'localhost'
= PASSWORD('nuevoAlumno123');
```

---

## Script 4: Revocar permisos y eliminar un usuario

### Revocar el permiso `SELECT`

```sql
REVOKE SELECT
ON escuela.*
FROM 'consulta'@'%';
```

### Eliminar el usuario

```sql
DROP USER 'consulta'@'%';
```

---

## Script 5: Crear un usuario con privilegios limitados a una tabla

### Crear el usuario

```sql
CREATE USER 'reportes'@'localhost'
IDENTIFIED BY 'reporte2025';
```

### Asignar permisos únicamente sobre una tabla

```sql
GRANT SELECT, INSERT
ON escuela.alumnos
TO 'reportes'@'localhost';
```

---

# Ejemplos de usuarios con diferentes opciones de acceso

## 1. Usuario básico con contraseña

```sql
CREATE USER 'juan'@'localhost'
IDENTIFIED BY 'claveSegura123';
```

El usuario `juan` solo podrá conectarse desde el mismo servidor.

---

## 2. Usuario con acceso desde cualquier host

```sql
CREATE USER 'ana'@'%'
IDENTIFIED BY 'claveAna456';
```

El símbolo `%` permite la conexión desde cualquier dirección IP.

> [!WARNING]
> Utilizar `'%'` puede aumentar la superficie de ataque. Siempre que sea posible, se recomienda especificar hosts o direcciones IP concretas.

---

## 3. Usuario con un plugin de autenticación específico

```sql
CREATE USER 'miguel'@'localhost'
IDENTIFIED WITH mysql_native_password
BY 'claveMiguel789';
```

Este comando fuerza el uso del plugin `mysql_native_password`, utilizado principalmente para mejorar la compatibilidad con aplicaciones antiguas.

---

## 4. Usuario sin contraseña

```sql
CREATE USER 'sinpass'@'localhost';
```

No se define una contraseña para el usuario.

> [!CAUTION]
> Esta configuración representa un riesgo elevado de seguridad. Solo debería utilizarse en entornos de desarrollo muy controlados.

---

## 5. Usuario con contraseña que debe cambiarse en el primer inicio de sesión

```sql
CREATE USER 'lucia'@'localhost'
IDENTIFIED BY 'lucia123'
PASSWORD EXPIRE;
```

El usuario deberá cambiar su contraseña la primera vez que inicie sesión.

---

## 6. Usuario con contraseña que caduca después de 30 días

```sql
CREATE USER 'daniel'@'localhost'
IDENTIFIED BY 'daniel123'
PASSWORD EXPIRE INTERVAL 30 DAY;
```

La contraseña deberá renovarse automáticamente después de 30 días.

---

## 7. Usuario bloqueado

```sql
CREATE USER 'bloqueado'@'localhost'
ACCOUNT LOCK;
```

El usuario existe, pero no podrá iniciar sesión hasta que sea desbloqueado.

---

## 8. Usuario desbloqueado

```sql
CREATE USER 'activo'@'localhost'
IDENTIFIED BY 'activo123'
ACCOUNT UNLOCK;
```

El usuario se crea habilitado para conectarse.

---

## 9. Usuario con nombre en mayúsculas

```sql
CREATE USER "ADMINISTRADOR"@'localhost'
IDENTIFIED BY 'AdminPass2025';
```

Este ejemplo utiliza comillas dobles para definir el nombre del usuario.

> [!NOTE]
> El comportamiento de las comillas y de los nombres de usuario puede depender de la configuración del modo SQL. Se recomienda utilizar comillas simples para los nombres de usuario en la sintaxis habitual de MySQL.

---

## 10. Usuario restringido a una dirección IP específica

```sql
CREATE USER 'seguridad'@'192.168.1.100'
IDENTIFIED BY 'ipFija123';
```

El usuario solo podrá conectarse desde la dirección IP `192.168.1.100`.

---

## 11. Usuario con el plugin `caching_sha2_password`

```sql
CREATE USER 'moderno'@'localhost'
IDENTIFIED WITH caching_sha2_password
BY 'claveModerna';
```

`caching_sha2_password` es un plugin de autenticación moderno y el método predeterminado en muchas instalaciones de MySQL 8.

---

## 12. Usuario creado sin contraseña y configurado posteriormente

### Crear el usuario

```sql
CREATE USER 'pedro'@'localhost';
```

### Asignar una contraseña posteriormente

```sql
ALTER USER 'pedro'@'localhost'
IDENTIFIED BY 'claveFinal123';
```

Este ejemplo demuestra que la contraseña puede agregarse o modificarse después de crear el usuario.

---

## 13. Usuario para replicación

### Crear el usuario

```sql
CREATE USER 'replicador'@'%'
IDENTIFIED BY 'replica123';
```

### Asignar privilegios de replicación

```sql
GRANT REPLICATION SLAVE
ON *.*
TO 'replicador'@'%';
```

Este usuario recibe los permisos necesarios para participar en la replicación entre servidores.

> [!WARNING]
> Para entornos actuales, conviene revisar los privilegios específicos requeridos por la versión de MySQL y por el tipo de replicación utilizado.

---

# Resumen de comandos principales

| Acción | Comando |
|---|---|
| Crear usuario | `CREATE USER` |
| Asignar privilegios | `GRANT` |
| Consultar privilegios | `SHOW GRANTS` |
| Revocar privilegios | `REVOKE` |
| Cambiar contraseña | `ALTER USER` |
| Eliminar usuario | `DROP USER` |
| Bloquear usuario | `ACCOUNT LOCK` |
| Desbloquear usuario | `ACCOUNT UNLOCK` |

---

## Documentación complementaria

- [Documentación oficial de MySQL — Account Management](https://dev.mysql.com/doc/refman/8.4/en/account-management-statements.html)
