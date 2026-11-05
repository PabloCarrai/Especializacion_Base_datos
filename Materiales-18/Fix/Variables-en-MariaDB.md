# Variables en MariaDB

**Objetivo**

Aprender a identificar, utilizar y modificar los distintos tipos de variables que ofrece MariaDB, comprendiendo su alcance, aplicación práctica y cómo afectan el comportamiento del servidor o las operaciones SQL.

---

## 1. Clasificación General de Variables

| Tipo de Variable | Ámbito | Persistencia | Uso principal |
| --- | --- | --- | --- |
| **Variables del sistema** | Global o sesión | Temporal o permanente | Configuración del servidor y sesiones |
| **Variables de usuario** | Sesión actual | Temporal | Almacenar valores personalizados temporales |
| **Variables locales** | Procedimientos y triggers | Temporal | Uso interno en bloques SQL |

---

## 2. Variables del Sistema

### ¿Qué son?

Son parámetros de configuración que controlan el funcionamiento de MariaDB (buffers, formatos, modos SQL, conexiones, etc.).

### Tipos:

* **`GLOBAL`**: Afectan a todo el servidor hasta que se reinicie. Afectan a todas las conexiones nuevas. No modifican las sesiones ya activas.
* **`SESSION`**: Afectan a una conexión específica. Afectan únicamente a la conexión actual (sesión). Cada usuario conectado puede tener un valor distinto para una misma variable `SESSION`.

### Ejemplos:

Ver todas las variables del sistema:

```sql
SHOW VARIABLES;

```

Ver una variable específica:

```sql
SHOW VARIABLES LIKE 'max_connections';

```

Ver variables globales vs de sesión:

```sql
SHOW GLOBAL VARIABLES LIKE 'sql_mode';
SHOW SESSION VARIABLES LIKE 'sql_mode';

```

### Modificar variables del sistema:

Cambiar variable de sesión:

```sql
SET SESSION sql_mode = 'STRICT_TRANS_TABLES';

```

Cambiar variable global (no persistente):

```sql
SET GLOBAL max_connections = 300;

```

Para hacerla persistente, editar el archivo `my.cnf` o `my.ini`:

```ini
[mysqld]
max_connections = 300

```

---

## 3. Variables de Usuario

### ¿Qué son?

Son variables temporales específicas de una sesión, útiles para almacenar resultados intermedios, usar en scripts y procedimientos.

### Sintaxis:

Comienzan con `@` y no requieren declaración previa.

### Ejemplos:

Asignación directa:

```sql
SET @nombre = 'Leonardo';
SET @monto = 1500;

```

Uso en consultas:

```sql
SELECT CONCAT('Bienvenido, ', @nombre) AS saludo;
SET @total = @monto * 1.21;
SELECT @total AS total_con_iva;

```

Reasignación:

```sql
SET @contador = @contador + 1;

```

---

## 4. Variables Locales (en Procedimientos y Triggers)

### ¿Qué son?

Variables internas declaradas explícitamente dentro de procedimientos almacenados o triggers.

### Sintaxis:

```sql
DECLARE nombre_variable TIPO [DEFAULT valor];

```

### Detalle:

| Parte | Significado |
| --- | --- |
| **`DECLARE`** | Palabra clave que indica la declaración de una variable local. |
| **`nombre_variable`** | Es el nombre de la variable que vas a usar. |
| **`TIPO`** | Es el tipo de dato que almacenará la variable (ej: `INT`, `DECIMAL`, `VARCHAR(100)`, etc). |
| **`[DEFAULT valor]`** | Es opcional. Sirve para inicializar la variable con un valor por defecto. |

> **Nota:** Esta sintaxis solo se puede usar dentro de bloques `BEGIN...END`, como los que aparecen en procedimientos o triggers.

### Ejemplo 1 – Sin valor por defecto

```sql
DECLARE cantidad INT;

```

Se declara una variable llamada `cantidad` de tipo entero. Su valor inicial será `NULL` hasta que se le asigne otro valor.

### Ejemplo 2 – Con valor por defecto

```sql
DECLARE precio DECIMAL(10,2) DEFAULT 100.00;

```

Declara la variable `precio` como decimal y la inicializa automáticamente con el valor `100.00`.

### Ejemplo para un procedimiento:

```sql
DELIMITER //

CREATE PROCEDURE calcular_total()
BEGIN
  DECLARE cantidad INT DEFAULT 3;
  DECLARE precio DECIMAL(10,2) DEFAULT 100.00;
  DECLARE total DECIMAL(10,2);
  
  SET total = cantidad * precio;
  
  SELECT total AS 'Total calculado';
END //

DELIMITER ;

```

### Para un trigger:

```sql
DELIMITER //

CREATE TRIGGER antes_insertar_venta
BEFORE INSERT ON ventas
FOR EACH ROW
BEGIN
  DECLARE impuesto DECIMAL(10,2);
  SET impuesto = NEW.total * 0.21;
  SET NEW.total = NEW.total + impuesto;
END //

DELIMITER ;

```

---

## 5. Buenas prácticas

* **Para variables globales:** documentar cada cambio y aplicarlo al archivo `my.cnf` para mantenerlo tras reinicios.
* **Para variables de usuario:** usar nombres descriptivos y evitar conflictos si hay muchas sesiones concurrentes.
* **Para procedimientos:** siempre inicializar las variables locales para evitar comportamientos inesperados.

---

## Tabla de variables comunes en MariaDB

| Variable | Ejemplo de consulta | Descripción |
| --- | --- | --- |
| `version` | `SHOW VARIABLES LIKE 'version';` | Versión exacta del servidor MariaDB. |
| `datadir` | `SHOW VARIABLES LIKE 'datadir';` | Carpeta donde se almacenan los archivos de datos. |
| `basedir` | `SHOW VARIABLES LIKE 'basedir';` | Carpeta de instalación de MariaDB. |
| `port` | `SHOW VARIABLES LIKE 'port';` | Puerto de escucha (por defecto 3306). |
| `max_connections` | `SHOW VARIABLES LIKE 'max_connections';` | Número máximo de conexiones simultáneas permitidas. |
| `wait_timeout` | `SHOW VARIABLES LIKE 'wait_timeout';` | Tiempo en segundos que una conexión inactiva permanece abierta. |
| `innodb_buffer_pool_size` | `SHOW VARIABLES LIKE 'innodb_buffer_pool_size';` | Memoria asignada al buffer de InnoDB (clave para rendimiento). |
| `innodb_log_file_size` | `SHOW VARIABLES LIKE 'innodb_log_file_size';` | Tamaño de cada archivo de log de InnoDB. |
| `innodb_flush_log_at_trx_commit` | `SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';` | Controla seguridad/rendimiento en commits de InnoDB. |
| `character_set_server` | `SHOW VARIABLES LIKE 'character_set_server';` | Charset por defecto de la base de datos. |
| `collation_server` | `SHOW VARIABLES LIKE 'collation_server';` | Ordenamiento de texto por defecto. |
| `sql_mode` | `SHOW VARIABLES LIKE 'sql_mode';` | Modo SQL (ej: `STRICT_TRANS_TABLES`, `NO_ZERO_DATE`, etc.). |
| `log_error` | `SHOW VARIABLES LIKE 'log_error';` | Ubicación del archivo de log de errores. |
| `slow_query_log` | `SHOW VARIABLES LIKE 'slow_query_log';` | Si el log de consultas lentas está habilitado (`ON`/`OFF`). |
| `slow_query_log_file` | `SHOW VARIABLES LIKE 'slow_query_log_file';` | Ruta del archivo donde se guardan las consultas lentas. |
| `long_query_time` | `SHOW VARIABLES LIKE 'long_query_time';` | Tiempo (en segundos) a partir del cual una consulta se considera lenta. |

---

## Ejemplo de calculadora usando variables

```sql
-- Ponemos valor a las variables (puede venir de una tabla o consulta)
SET @a = 120;
SET @b = 499;

-- Realizamos las operaciones matemáticas
SET @suma = @a + @b;
SET @resta = @a - @b;
SET @multiplicacion = @a * @b;
SET @division = IF(@b != 0, @a / @b, NULL);
SET @potencia = POW(@a, @b);
SET @modulo = MOD(@a, @b);

-- Mostramos los resultados
SELECT 'Suma' AS operacion, @suma AS resultado
UNION ALL
SELECT 'Resta', @resta
UNION ALL
SELECT 'Multiplicación', @multiplicacion
UNION ALL
SELECT 'División', @division
UNION ALL
SELECT 'Potencia', @potencia
UNION ALL
SELECT 'Módulo', @modulo;

```
