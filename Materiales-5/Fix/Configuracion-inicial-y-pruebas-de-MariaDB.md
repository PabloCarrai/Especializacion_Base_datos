# Configuración inicial y pruebas de MariaDB

## 1. Verificar que MariaDB está funcionando

Después de la instalación, comprobamos el estado del servicio:

```bash
sudo systemctl status mariadb
```

Deberíamos obtener una salida similar a:

```text
Active: active (running)
```

También podemos comprobar si el servicio se inicia automáticamente con el sistema:

```bash
sudo systemctl is-enabled mariadb
```

Si no estuviera habilitado, lo activamos con:

```bash
sudo systemctl enable mariadb
```

Para iniciar el servicio manualmente:

```bash
sudo systemctl start mariadb
```

---

## 2. Comprobar la versión instalada

Desde Ubuntu, ejecutamos:

```bash
mariadb --version
```

Por ejemplo:

```text
mariadb  Ver 15.1 Distrib 11.x.x-MariaDB
```

> [!IMPORTANT]
> - **MariaDB Server** es el servidor de base de datos.
> - **`mariadb`** es el cliente de línea de comandos que utilizamos para conectarnos al servidor.

---

## 3. Primera conexión

En Ubuntu podemos conectarnos como administrador mediante:

```bash
sudo mariadb
```

Si todo funciona correctamente, veremos algo similar a:

```text
Welcome to the MariaDB monitor.

MariaDB [(none)]>
```

Esto significa que estamos dentro del cliente SQL.

Podemos ejecutar algunas consultas de prueba:

```sql
SELECT VERSION();
```

```sql
SELECT NOW();
```

Para salir del cliente de MariaDB:

```sql
EXIT;
```

---

## 4. Crear una base de datos de prueba

Volvemos a ingresar a MariaDB:

```bash
sudo mariadb
```

Creamos una base de datos llamada `escuela`:

```sql
CREATE DATABASE escuela;
```

Verificamos que se haya creado correctamente:

```sql
SHOW DATABASES;
```

Seleccionamos la base de datos:

```sql
USE escuela;
```

Comprobamos cuál es la base de datos seleccionada:

```sql
SELECT DATABASE();
```

Resultado esperado:

```text
+------------+
| DATABASE() |
+------------+
| escuela    |
+------------+
```

---

## 5. Crear una tabla

Para comenzar a trabajar con datos, creamos la tabla `alumnos`:

```sql
CREATE TABLE alumnos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    edad INT,
    email VARCHAR(100)
);
```

Verificamos la estructura de la tabla:

```sql
DESCRIBE alumnos;
```

También podemos comprobar las tablas disponibles:

```sql
SHOW TABLES;
```

---

## 6. Insertar datos

Insertamos algunos registros de prueba:

```sql
INSERT INTO alumnos (nombre, apellido, edad, email) VALUES
('Juan', 'Perez', 25, 'juan@example.com'),
('Ana', 'Gomez', 30, 'ana@example.com'),
('Carlos', 'Lopez', 28, 'carlos@example.com');
```

Consultamos los datos almacenados:

```sql
SELECT * FROM alumnos;
```

---

## 7. Crear un usuario específico

Creamos un usuario para la aplicación:

```sql
CREATE USER 'appuser'@'localhost'
IDENTIFIED BY 'ClaveSegura123!';
```

Le otorgamos permisos sobre la base de datos `escuela`:

```sql
GRANT ALL PRIVILEGES
ON escuela.*
TO 'appuser'@'localhost';
```

Verificamos los permisos asignados:

```sql
SHOW GRANTS FOR 'appuser'@'localhost';
```

> [!NOTE]
> - **Autenticación:** determina quién puede ingresar.
> - **Autorización:** determina qué puede hacer una vez que ingresó.

---

## 8. Probar la conexión con el nuevo usuario

Primero salimos de MariaDB:

```sql
EXIT;
```

Después nos conectamos utilizando el usuario creado:

```bash
mariadb -u appuser -p escuela
```

Ingresamos la contraseña cuando se solicite.

Una vez dentro, comprobamos la base de datos seleccionada:

```sql
SELECT DATABASE();
```

Consultamos los datos de la tabla:

```sql
SELECT * FROM alumnos;
```

De esta manera comprobamos que el usuario puede acceder correctamente a la base de datos.

---

## 9. Comprobar el puerto de MariaDB

MariaDB utiliza normalmente el puerto:

```text
TCP 3306
```

Desde Ubuntu, podemos comprobar si el puerto está escuchando:

```bash
sudo ss -lntp | grep 3306
```

Podemos obtener una salida similar a:

```text
LISTEN 0 80 127.0.0.1:3306
```

### Interpretación de `127.0.0.1:3306`

- `127.0.0.1` indica que MariaDB escucha únicamente en la máquina local.
- `3306` es el puerto utilizado por MariaDB.

> [!TIP]
> Esta es una buena configuración inicial desde el punto de vista de la seguridad, ya que impide conexiones externas directas al servidor.

---

## 10. Conexiones desde otra PC

Si queremos conectarnos a MariaDB desde otra computadora, debemos revisar si el servidor acepta conexiones remotas.

Primero comprobamos nuevamente el puerto:

```bash
sudo ss -lntp | grep 3306
```

Si aparece:

```text
127.0.0.1:3306
```

MariaDB no está escuchando conexiones externas.

Para aceptar conexiones desde otras máquinas, debemos modificar la configuración del servidor:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Dentro del archivo, buscamos la directiva:

```ini
bind-address = 127.0.0.1
```

Esta configuración limita las conexiones a la propia máquina. Para habilitar conexiones remotas, será necesario ajustar esta directiva y aplicar las medidas de seguridad correspondientes.

---

## 11. Consultar información del servidor

Podemos consultar todas las variables de configuración de MariaDB:

```sql
SHOW VARIABLES;
```

También podemos consultar variables específicas:

```sql
SHOW VARIABLES LIKE 'port';
```

```sql
SHOW VARIABLES LIKE 'bind_address';
```

```sql
SHOW VARIABLES LIKE 'datadir';
```

Estas consultas permiten conocer, entre otros datos:

- El puerto utilizado por MariaDB.
- La dirección en la que escucha el servidor.
- La ubicación del directorio de datos.

---

## 12. Ver las conexiones actuales

Para observar las conexiones y operaciones que está gestionando el servidor, ejecutamos:

```sql
SHOW PROCESSLIST;
```

También podemos consultar las estadísticas generales del servidor:

```sql
SHOW STATUS;
```

> [!NOTE]
> `SHOW PROCESSLIST` permite observar las conexiones activas, mientras que `SHOW STATUS` muestra estadísticas de funcionamiento de MariaDB.
