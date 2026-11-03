# Práctica de índices

## Creación de la Base de Datos y Estructura Inicial

Ejecutar el siguiente bloque de código para crear un entorno de pruebas limpio.

```sql
-- Creamos y seleccionamos la base de datos
DROP DATABASE IF EXISTS db_indices_clase;
CREATE DATABASE db_indices_clase;
USE db_indices_clase;

-- Tabla de Empleados (Demostración de PRIMARY KEY y UNIQUE)
CREATE TABLE empleados (
    legajo INT AUTO_INCREMENT PRIMARY KEY,
    dni VARCHAR(20) NOT NULL UNIQUE,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    activo TINYINT DEFAULT 1
);

-- Tabla de Ventas (Para ver rendimiento, EXPLAIN y Cardinalidad)
CREATE TABLE ventas (
    id_venta INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT NOT NULL,
    monto DECIMAL(10,2) NOT NULL,
    fecha DATE NOT NULL
);

-- Tabla de Artículos (Para Fulltext Search)
CREATE TABLE articulos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    titulo VARCHAR(150),
    contenido TEXT,
    FULLTEXT(contenido)
);

```

## Carga de Datos Masivos para Pruebas

Para notar la utilidad de los índices, necesitamos un volumen de datos aceptable. Vamos a insertar registros de prueba en la tabla ventas.

```sql
-- Insertamos registros simulados en ventas (aprox 10,000 filas con un procedimiento o inserción múltiple)
-- Se puede correr este bloque un par de veces:
INSERT INTO ventas (cliente_id, monto, fecha) VALUES 
(10, 1500.50, '2026-01-01'),
(25, 3200.00, '2026-01-02'),
(10, 450.00, '2026-01-03'),
(99, 12000.00, '2026-01-04'),
(15, 890.50, '2026-01-05');

-- Duplicamos para simular más volumen rápidamente (ejecutar varias veces también)
INSERT INTO ventas (cliente_id, monto, fecha) 
SELECT cliente_id + 1, monto * 1.1, fecha FROM ventas;
-- (Ejecutar el INSERT de arriba unas 5 o 6 veces para acumular registros y notar el efecto en EXPLAIN).

```

## Análisis de Rendimiento con EXPLAIN

El comando EXPLAIN es sirve para saber si MariaDB está usando un índice o haciendo un Full Table Scan.

### Consulta sin índice en `cliente_id`

```sql
-- Analicemos cómo busca el motor antes de indexar cliente_id
EXPLAIN SELECT * FROM ventas WHERE cliente_id = 10;

```

> **Observación en Workbench:** En la columna `type` dice `ALL` (significa búsqueda completa de tabla) y en `rows` una cantidad alta de registros recorridos.

### Creando un índice estándar (INDEX)

```sql
-- Creamos un índice explícito para cliente_id
CREATE INDEX idx_cliente_id ON ventas(cliente_id);

-- Volvemos a ejecutar el EXPLAIN
EXPLAIN SELECT * FROM ventas WHERE cliente_id = 10;

```

> **Observación en Workbench:** Ahora la columna `type` cambiará a `ref` o `range` y la columna `key` mostrará `idx_cliente_id`. El motor encontró los datos directamente sin leer toda la tabla.

Para ver cuánto espacio ocupan los índices de tus tablas directamente en MySQL Workbench, podés ejecutar una consulta SQL que consulte el diccionario de datos del sistema (`information_schema`).

```sql
SELECT 
    TABLE_NAME AS `Tabla`,
    ROUND(SUM(INDEX_LENGTH) / 1024 / 1024, 2) AS `Tamaño_Indices_MB`,
    ROUND(SUM(DATA_LENGTH) / 1024 / 1024, 2) AS `Tamaño_Datos_MB`,
    ROUND(SUM(DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024, 2) AS `Tamaño_Total_MB`
FROM 
    information_schema.TABLES
WHERE 
    TABLE_SCHEMA = 'db_indices_clase'  -- <--- Cambia esto por tu base de datos
GROUP BY 
    TABLE_NAME
ORDER BY 
    Tamaño_Indices_MB DESC;

```

## Mantenimiento: `ANALYZE TABLE` y `OPTIMIZE TABLE`

```sql
-- Actualiza las estadísticas internas para que el optimizador elija el mejor plan
ANALYZE TABLE ventas;

-- Reorganiza físicamente la tabla y desfragmenta el espacio en disco (ideal tras borrados masivos)
OPTIMIZE TABLE ventas;

```

Este mensaje no es un error, sino una notificación informativa (`status: OK`) del motor de base de datos.

Significa que el motor de almacenamiento de la tabla (InnoDB en MariaDB/MySQL) no realiza una optimización "en el lugar" tradicional para ese comando en particular, por lo que el sistema automáticamente aplica una estrategia equivalente y segura: recrear la tabla y reanalizar sus estadísticas de forma transparente para vos. Los datos se han optimizado correctamente.

### ¿Cada cuánto tiempo se recomienda ejecutar `OPTIMIZE TABLE` en producción?

En entornos de producción, la regla de oro con `OPTIMIZE TABLE` es no automatizarlo ni ejecutarlo por rutina (como todas las noches o todas las semanas) sin una justificación real, ya que es una operación pesada.

Recomendaciones clave sobre cuándo y cada cuánto tiempo aplicarlo:

#### 1. ¿Cada cuánto tiempo? (La frecuencia real)

* No hay un período de tiempo fijo (días/meses): Su frecuencia depende enteramente del volumen de operaciones de escritura/borrado, no del reloj.
* Una tabla con baja actividad puede necesitarlo una vez al año (o nunca), mientras que una tabla transaccional con miles de borrados diarios podría requerirlo periódicamente, pero incluso en esos casos, suele hacerse de forma mensual o trimestral, o controlado por métricas.

#### 2. ¿Cuándo SÍ se recomienda ejecutarlo?

Debés correr `OPTIMIZE TABLE` únicamente cuando se cumplan condiciones específicas de fragmentación:

* **Después de operaciones masivas de `DELETE` o `UPDATE`:** Por ejemplo, si a fin de mes borrás millones de registros históricos de una tabla de auditoría o logs.
* **Cuando se nota un desperdicio significativo de espacio:** Puedes consultar el espacio libre o fragmentado en la base de datos (por ejemplo, consultando la tabla `information_schema.TABLES` revisando la columna `Data_free`).
* **Cuando el rendimiento de las consultas comienza a degradarse** a pesar de tener los índices correctos, debido a que las páginas de datos están muy dispersas en el disco.

#### 3. Precauciones importantes en Producción

* **Bloqueo de tablas:** `OPTIMIZE TABLE` (especialmente en InnoDB cuando hace un *recreate*) bloquea la tabla durante el proceso. En tablas muy grandes con millones de filas, esto puede dejar la tabla inaccesible para los usuarios o la aplicación durante varios minutos u horas, provocando cortes o tiempos de espera agotados (*timeouts*).
* **Uso de herramientas alternativas:** En sistemas 24/7 de alta concurrencia donde no se pueden permitir bloqueos prolongados, los administradores suelen preferir herramientas de terceros (como `pt-online-schema-change` de Percona) para recrear tablas y optimizar sin bloquear escrituras.

---

## Práctica de Búsqueda Fulltext

Probemos el índice especializado para textos largos en la tabla artículos.

```sql
-- Insertamos datos de prueba
INSERT INTO articulos (titulo, contenido) VALUES 
('Introducción a SQL', 'MySQL y MariaDB son sistemas de gestión de bases de datos relacionales muy populares.');
('Optimización avanzada', 'Los índices B-Tree y FullText mejoran drásticamente los tiempos de respuesta en consultas complejas.');

-- Buscamos utilizando MATCH ... AGAINST
SELECT * FROM articulos 
WHERE MATCH(contenido) AGAINST('MariaDB bases de datos' IN NATURAL LANGUAGE MODE);

```

Esa consulta sirve para realizar una búsqueda de texto completo (*Full-Text Search*) de manera inteligente y eficiente, en lugar de usar el clásico `LIKE '%palabra%'`.

Vamos a desglosarla parte por parte:

### 1. `SELECT * FROM articulos`

Selecciona todas las columnas de la tabla `articulos` para aquellos registros que cumplan con la condición de búsqueda.

### 2. `WHERE MATCH(contenido)...`

* `MATCH(contenido)`: Le indica a MariaDB que queremos buscar dentro de la columna `contenido`.
* **Requisito obligatorio:** Para que esto funcione, la columna `contenido` debe tener un índice de tipo `FULLTEXT` creado previamente (tal como lo definimos al crear la tabla `articulos`). Si no tuviera ese índice, la consulta daría error.

### 3. `AGAINST('MariaDB bases de datos' IN NATURAL LANGUAGE MODE)`

* `AGAINST(...)`: Es la función que recibe las palabras clave o la frase que estamos buscando.
* `'MariaDB bases de datos'`: Son los términos de búsqueda que el motor va a buscar dentro del texto.
* `IN NATURAL LANGUAGE MODE`: Es el modo de búsqueda por defecto. Funciona interpretando el texto como si fuera lenguaje natural (humano):
* El motor analiza el texto buscado, lo divide en palabras individuales (*tokens*) y busca registros que contengan alguna o todas esas palabras.
* **Relevancia:** A diferencia de un `LIKE`, este modo ordena automáticamente los resultados de mayor a menor relevancia (es decir, las filas que más veces repiten las palabras o donde aparecen de forma más significativa aparecen primero).



### ¿Por qué es mejor que usar `LIKE '%MariaDB%'`?

* **Rendimiento:** Un `LIKE` con comodines al inicio (`%palabra%`) obliga a hacer un *Full Table Scan* (recorrer fila por fila), lo cual es lentísimo en tablas grandes. El índice `FULLTEXT` usa una estructura especial optimizada para texto.
* **Inteligencia:** Entiende frases, busca múltiples palabras a la vez y calcula la relevancia sin necesidad de combinar decenas de `OR LIKE`.
