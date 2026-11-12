# Variables en MariaDB

## Objetivo

Aprender a identificar, utilizar y modificar los distintos tipos de variables que ofrece MariaDB, comprendiendo su alcance, aplicación práctica y cómo afectan el comportamiento del servidor o las operaciones SQL.

## 1. Clasificación general de variables

| Tipo de variable | Ámbito de visibilidad | Persistencia | Uso principal y arquitectura |
|---|---|---|---|
| Variables del sistema | Global o sesión | Temporal (vuelve al valor por defecto al reiniciar) o permanente (vía `my.cnf`) | Configuración del comportamiento general del motor, gestión de memoria, buffers y restricciones de seguridad. |
| Variables de usuario | Sesión actual (conexión) | Temporal (se destruyen al cerrar la conexión o cliente) | Almacenamiento rápido de resultados intermedios, control de contadores en consultas y paso de datos entre sentencias. |
| Variables locales | Bloques `BEGIN...END` (procedimientos, funciones, triggers) | Estrictamente efímera (existe solo durante la ejecución del bloque) | Lógica interna de negocio, procesamiento algorítmico y manipulación de datos transaccionales. |

## 2. Variables del sistema (*System Variables*)

### ¿Qué son y cómo operan a bajo nivel?

Las variables del sistema son parámetros internos del motor de MariaDB que configuran aspectos críticos como la asignación de memoria RAM para los pools de InnoDB, los límites de concurrencia, las políticas de codificación de caracteres y los modos de cumplimiento de SQL (`sql_mode`).

Cuando MariaDB arranca, lee estos parámetros desde los archivos de configuración (`my.cnf` o `my.ini`). Durante la ejecución, se dividen en dos categorías de visibilidad:

- **GLOBAL:** Afectan a todo el servidor. Modificar una variable global cambia el comportamiento predeterminado para todas las conexiones nuevas que se establezcan a partir de ese momento. No alteran de manera retroactiva las sesiones que ya se encuentran activas.
- **SESSION** (o **LOCAL**): Afectan exclusivamente a la conexión actual con el cliente. Cada hilo (*thread*) de conexión puede sobrescribir variables de sesión de forma independiente sin afectar a otros usuarios conectados al mismo servidor.

### Sintaxis y ejemplos de consulta

```sql
-- Consultar todas las variables del sistema y sus valores actuales
SHOW VARIABLES;

-- Consultar una variable específica (busca en global o sesión según el contexto)
SHOW VARIABLES LIKE 'max_connections';

-- Consultar explícitamente el valor GLOBAL de una variable de configuración
SHOW GLOBAL VARIABLES LIKE 'sql_mode';

-- Consultar explícitamente el valor de la SESIÓN actual
SHOW SESSION VARIABLES LIKE 'sql_mode';
```

### Modificación y persistencia

Modificar variables en caliente permite ajustar el rendimiento sin reiniciar el servicio, aunque muchas configuraciones profundas de almacenamiento requieren persistencia en disco:

```sql
-- Cambiar una variable únicamente para la sesión activa actual
SET SESSION sql_mode = 'STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION';

-- Cambiar una variable a nivel global para futuras conexiones (se pierde si se reinicia el servidor)
SET GLOBAL max_connections = 300;
```

> **Nota:** Para garantizar que los cambios globales resistan un reinicio del servidor, debes declararlos explícitamente en el archivo de configuración del sistema operativo (`/etc/mysql/my.cnf` o equivalente):

```ini
[mysqld]
max_connections = 300
```

## 3. Variables de usuario (*User-Defined Variables*)

### ¿Qué son y cuál es su alcance?

Son variables dinámicas definidas por el usuario que operan exclusivamente en el ámbito de la sesión actual. Su característica principal es la flexibilidad: no requieren una declaración de tipo previa (su tipo de datos se infiere automáticamente según el valor asignado: entero, decimal, cadena, booleano o `NULL`).

- Se identifican siempre por ir precedidas por el carácter arroba (`@`).
- Son sumamente útiles para optimizar consultas complejas, almacenar contadores incrementales sobre conjuntos de resultados (*result sets*) o simular variables de estado en scripts complejos sin necesidad de crear procedimientos almacenados.

### Ejemplos prácticos

```sql
-- Asignación directa de valores escalares
SET @nombre = 'Leonardo';
SET @monto = 1500.00;

-- Uso de variables dentro de expresiones y funciones
SELECT CONCAT('Bienvenido al sistema, ', @nombre) AS saludo;

-- Cálculo e incremento dinámico
SET @total = @monto * 1.21;
SELECT @total AS total_con_iva;

-- Patrón de contador acumulativo en consultas
SET @contador = 0;
SELECT @contador := @contador + 1 AS id_secuencial, nombre_producto
FROM productos
LIMIT 5;
```

## 4. Variables locales (*Stored Routine & Trigger Variables*)

### ¿Qué son y por qué son estrictas?

A diferencia de las variables de usuario, las variables locales están diseñadas para la programación estructurada dentro de la base de datos (*Stored Procedures*, *Functions* y *Triggers*). Su tipo es estricto: deben ser declaradas explícitamente indicando su tipo de dato exacto antes de ser utilizadas.

### Sintaxis de declaración

```sql
DECLARE nombre_variable TIPO_DE_DATO [DEFAULT valor_inicial];
```

- **Regla de ámbito:** Solo existen y son accesibles dentro del bloque compuesto delimitado por `BEGIN ... END`. Una vez que el procedimiento o trigger finaliza su ejecución, la variable local se destruye y libera de la memoria del servidor.
- Si no se especifica un valor mediante `DEFAULT`, la variable toma automáticamente el valor `NULL`.

### Ejemplo 1: Procedimiento almacenado con lógica matemática

```sql
DELIMITER //

CREATE PROCEDURE calcular_total()
BEGIN
    -- Declaración estricta de variables locales con tipo y valores por defecto
    DECLARE cantidad INT DEFAULT 3;
    DECLARE precio DECIMAL(10,2) DEFAULT 100.00;
    DECLARE total DECIMAL(10,2);

    -- Operación lógica interna
    SET total = cantidad * precio;

    -- Retorno del resultado estructurado
    SELECT total AS 'Total calculado';
END //

DELIMITER ;
```

### Ejemplo 2: Trigger operando con contextos `NEW`

```sql
DELIMITER //

CREATE TRIGGER antes_insertar_venta
BEFORE INSERT ON ventas
FOR EACH ROW
BEGIN
    -- Declaración de variable local para cálculo de impuestos
    DECLARE impuesto DECIMAL(10,2);

    -- Cálculo basado en el registro que está ingresando (NEW)
    SET impuesto = NEW.total * 0.21;
    SET NEW.total = NEW.total + impuesto;
END //

DELIMITER ;
```

## 5. Tabla de variables del sistema comunes en MariaDB

Esta tabla resume los parámetros de configuración más importantes que todo administrador o desarrollador debe monitorear y comprender para afinar el rendimiento del servidor:

| Variable del sistema | Ejemplo de consulta | Descripción técnica e impacto |
|---|---|---|
| `version` | `SHOW VARIABLES LIKE 'version';` | Versión exacta y compilación del servidor MariaDB en ejecución. |
| `datadir` | `SHOW VARIABLES LIKE 'datadir';` | Directorio físico del sistema de archivos donde se almacenan las bases de datos y tablas. |
| `basedir` | `SHOW VARIABLES LIKE 'basedir';` | Ruta raíz de la instalación de los binarios de MariaDB. |
| `port` | `SHOW VARIABLES LIKE 'port';` | Puerto de red TCP/IP de escucha para conexiones entrantes (por defecto 3306). |
| `max_connections` | `SHOW VARIABLES LIKE 'max_connections';` | Límite superior de conexiones de clientes concurrentes permitidas simultáneamente. |
| `wait_timeout` | `SHOW VARIABLES LIKE 'wait_timeout';` | Segundos que el servidor aguarda con una conexión inactiva antes de cerrarla automáticamente. |
| `innodb_buffer_pool_size` | `SHOW VARIABLES LIKE 'innodb_buffer_pool_size';` | Crítico para rendimiento: memoria RAM reservada para almacenar cachés de datos e índices de tablas InnoDB. |
| `innodb_log_file_size` | `SHOW VARIABLES LIKE 'innodb_log_file_size';` | Tamaño de cada archivo de registro de transacciones (*redo log*) de InnoDB. |
| `innodb_flush_log_at_trx_commit` | `SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';` | Equilibrio entre durabilidad transaccional (seguridad ante cortes de luz) y velocidad de escritura. |
| `character_set_server` | `SHOW VARIABLES LIKE 'character_set_server';` | Conjunto de caracteres predeterminado a nivel global para las nuevas bases de datos. |
| `collation_server` | `SHOW VARIABLES LIKE 'collation_server';` | Reglas de cotejo y ordenamiento de texto predeterminadas del servidor. |
| `sql_mode` | `SHOW VARIABLES LIKE 'sql_mode';` | Define reglas sintácticas estrictas y validaciones de datos (ej. rechazar ceros en fechas). |
| `log_error` | `SHOW VARIABLES LIKE 'log_error';` | Ruta física y archivo donde el servidor registra advertencias y errores críticos. |
| `slow_query_log` | `SHOW VARIABLES LIKE 'slow_query_log';` | Interruptor (ON/OFF) para activar el registro de consultas que demoran demasiado. |
| `slow_query_log_file` | `SHOW VARIABLES LIKE 'slow_query_log_file';` | Ubicación del archivo de texto donde se acumulan las consultas lentas diagnosticadas. |
| `long_query_time` | `SHOW VARIABLES LIKE 'long_query_time';` | Umbral en segundos a partir del cual una consulta SQL se cataloga oficialmente como "lenta". |

## 6. Buenas prácticas

1. **Variables globales:** Evita modificar parámetros críticos de memoria o conexiones en caliente en servidores de producción altamente concurridos sin antes probar el impacto en un entorno de ensayo (*staging*). Documentar siempre los cambios y asegúrate de reflejarlos en `my.cnf`.
2. **Variables de usuario:** Utilizar nombres descriptivos y evitar reutilizar la misma variable `@` para fines distintos en diferentes capas de la aplicación para prevenir lógica errónea o colisiones de datos.
3. **Variables locales:** Inicializar siempre las variables locales con un valor mediante `DEFAULT` o una sentencia `SET` inicial. Esto previene que errores lógicos silenciosos operen con valores `NULL` no deseados dentro de los procedimientos almacenados.

## 7. Ejemplo práctico: Calculadora con variables de usuario

Este script demuestra cómo las variables de usuario permiten procesar operaciones matemáticas complejas, control de errores (como evitar divisiones por cero mediante la función condicional `IF`) y consolidar conjuntos de datos tabulares consolidados:

```sql
-- Inicialización de operadores de trabajo
SET @a = 12;
SET @b = 49;

-- Ejecución y almacenamiento de operaciones matemáticas avanzadas
SET @suma = @a + @b;
SET @resta = @a - @b;
SET @multiplicacion = @a * @b;
SET @division = IF(@b != 0, @a / @b, NULL); -- Control preventivo de división por cero
SET @potencia = POW(@a, @b);
SET @modulo = MOD(@a, @b);

-- Consolidación y presentación unificada de resultados mediante UNION ALL
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
