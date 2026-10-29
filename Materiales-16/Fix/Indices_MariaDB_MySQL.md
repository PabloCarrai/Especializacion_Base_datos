# Índices en MySQL

## ¿Qué es un Índice?

Un índice es una estructura de datos auxiliar que **acelera las consultas de búsqueda y ordenamiento** en una tabla.

> Funciona como el índice de un libro: permite localizar información rápidamente sin tener que leer todas las páginas.

---

## ¿Por qué usar índices?

### Ventajas principales

* Búsqueda más rápida con cláusulas `WHERE`
* Mejor rendimiento en operaciones `JOIN`
* Aceleración de operaciones `ORDER BY` y `GROUP BY`
* Menor lectura de datos en disco

---

## Tipos de Índices en MySQL

### 1. `PRIMARY KEY`

* **Descripción:** Es un tipo especial de índice que identifica de forma única cada fila en una tabla.
* **Características:**
* No permite valores `NULL`.
* Solo puede existir una clave primaria por tabla.
* Es automáticamente un índice `UNIQUE`.


* **Ejemplo:**

```sql
CREATE TABLE empleados (
  legajo INT PRIMARY KEY,
  nombre VARCHAR(100)
);

```

### 2. `UNIQUE`

* **Descripción:** Impide la duplicación de valores en una o más columnas.
* **Características:**
* Se pueden definir varios índices `UNIQUE` en una tabla.
* Acepta valores `NULL` (excepto en `PRIMARY KEY`).


* **Ejemplo:**

```sql
CREATE TABLE usuarios (
  id INT AUTO_INCREMENT PRIMARY KEY,
  email VARCHAR(255) UNIQUE
);

```

### 3. `INDEX` o `KEY`

* **Descripción:** Índice estándar utilizado para mejorar el rendimiento de las consultas.
* **Características:**
* Permite valores duplicados y `NULL`.
* Se puede crear en una o varias columnas.


* **Ejemplo:**

```sql
CREATE INDEX idx_apellido ON empleados(apellido);

```

### 4. `FULLTEXT`

* **Descripción:** Índice utilizado para búsquedas de texto completo en columnas `CHAR`, `VARCHAR` o `TEXT`.
* **Características:**
* Permite hacer búsquedas con `MATCH ... AGAINST`.
* Disponible en motores como InnoDB (desde MySQL 5.6) y MyISAM.


* **Ejemplo:**

```sql
CREATE TABLE articulos (
  id INT PRIMARY KEY,
  contenido TEXT,
  FULLTEXT(contenido)
);

```

---

## ¿Dónde conviene agregar índices?

* Se realizan búsquedas frecuentes por una columna (`WHERE campo = valor`).
* La columna se usa en `joins` entre tablas.
* Se necesita ordenar o agrupar por una columna (`ORDER BY`, `GROUP BY`).
* Se realizan búsquedas por rango (`BETWEEN`, `>`, `<`).
* Columnas utilizadas en `WHERE`, `JOIN`, `ORDER BY` o `GROUP BY`.
* Claves foráneas (*foreign keys*).

---

## ¿Cuándo evitar índices?

* Columnas con baja selectividad.
* Tablas pequeñas.
* Si se prioriza el rendimiento de `INSERT`, `UPDATE` o `DELETE`.

---

## Buenas prácticas

* Usar nombres significativos para los índices.
* Revisar con `SHOW INDEXES FROM tabla;` y `EXPLAIN`.
* Eliminar índices no utilizados.

---

## El Costo Oculto de los Índices (Impacto y Overhead)

Aunque los índices aceleran dramáticamente las consultas de lectura (`SELECT`), no son gratuitos. Cada índice adicional introduce costos operativos que deben evaluarse cuidadosamente:

### 1. Impacto en Operaciones de Escritura (`INSERT`, `UPDATE`, `DELETE`)

* Cada vez que se inserta una fila, MariaDB no solo debe escribir los datos en la tabla, sino también actualizar cada uno de los índices asociados a esa tabla.
* Si una tabla tiene una clave primaria y 4 índices secundarios, un simple `INSERT` puede desencadenar múltiples escrituras adicionales en disco para reordenar y balancear las estructuras de los índices.
* Las operaciones `UPDATE` que modifican columnas indexadas fuerzan al motor a eliminar y reinsertar las entradas en el índice, degradando fuertemente el rendimiento.

### 2. Consumo de Almacenamiento (Disco)

* Los índices se almacenan físicamente en el disco. En tablas con millones de registros, el espacio ocupado por los índices puede igualar o incluso superar el tamaño de los datos reales de la tabla.

### 3. Presión sobre la Memoria RAM (*Buffer Pool*)

* Para que un índice sea realmente rápido, MariaDB intenta mantener las páginas del índice en la memoria RAM (en el *Buffer Pool* de InnoDB).
* Si hay un exceso de índices inútiles o sobredimensionados, estos compiten por el espacio en memoria, provocando un aumento en las operaciones de lectura/escritura en disco (*Disk I/O*) y ralentizando el servidor en general.

---

## ¿Cómo Funcionan por Dentro? (Estructura B-Tree)

En MariaDB (específicamente en el motor InnoDB), la gran mayoría de los índices se implementan utilizando una estructura de datos llamada Árbol B+ (*B-Tree*).

* **Organización jerárquica:** Los datos en el índice se estructuran en nodos jerárquicos (raíz, ramas y hojas) ordenados de forma secuencial.
* **Búsquedas eficientes:** En lugar de realizar un recorrido secuencial (*Full Table Scan*), el motor desciende por el árbol descartando la mitad de las ramas en cada paso.
* **Claves Primarias (*Clustered Index*):** En InnoDB, la tabla misma está organizada como un árbol B+ basado en la Clave Primaria. Esto significa que las filas de datos se almacenan físicamente en el mismo orden que el índice primario, haciendo que las búsquedas por `PRIMARY KEY` sean extremadamente rápidas.
* **Índices Secundarios:** Los índices secundarios (como `UNIQUE` o `INDEX`) no almacenan las filas completas de datos, sino una copia de las columnas indexadas junto con el valor de la Clave Primaria correspondiente para poder ubicar el registro final.

---

## Cardinalidad y Selectividad

Un concepto teórico fundamental para un administrador de bases de datos es entender qué hace que un índice sea efectivo:

* **Cardinalidad:** Es el número de valores únicos que existen en una columna.
* *Ejemplo:* Una columna `dni` tiene alta cardinalidad (casi tantos valores únicos como filas). Una columna `genero` o `activo` (con valores `0` o `1`) tiene baja cardinalidad.


* **Selectividad:** Mide qué tan exclusivo es el resultado de una consulta filtrada por ese índice.
* **¿Por qué importa?** Si creas un índice en una columna de baja cardinalidad (ej. un campo booleano donde el 95% de los registros son `1`), el optimizador de MariaDB a menudo ignorará el índice. Considerará que es más rápido hacer un recorrido completo de la tabla (*Full Table Scan*) que buscar en el índice y luego ir a buscar los registros a disco, ya que el índice no logra filtrar casi nada.

---

## Índices Invisibles (*Invisible Indexes*) en MariaDB

Una característica moderna y muy útil en la administración de bases de datos es la capacidad de hacer que un índice sea invisible.

* **¿Para qué sirve?** Permite probar qué sucede si eliminas un índice sin tener que borrarlo definitivamente. Si una consulta se vuelve lenta o falla, puedes volver a hacerlo visible de inmediato.
* **Sintaxis de ejemplo:**

```sql
-- Volver un índice invisible para el optimizador
ALTER TABLE empleados ALTER INDEX idx_apellido INVISIBLE;

-- Volver a hacerlo visible
ALTER TABLE empleados ALTER INDEX idx_apellido VISIBLE;

```

---

## Manejo de índices

### Ver índices existentes en una tabla

```sql
SHOW INDEX FROM nombre_tabla;

```

### Eliminar (`DROP`) un índice

```sql
DROP INDEX nombre_indice ON nombre_tabla;

```

### Ver estadísticas de uso de índice

```sql
EXPLAIN SELECT * FROM ventas WHERE cliente_id = 10;
-- Muestra si se está usando algún índice en una consulta.

```

---

## Optimizando los índices

### ¿Qué es `OPTIMIZE TABLE`?

`OPTIMIZE TABLE` reorganiza físicamente la tabla y sus índices. Elimina el espacio libre dejado por operaciones `DELETE` o `UPDATE` y mejora el rendimiento al reconstruir la tabla.

#### Cuándo usar `OPTIMIZE TABLE`

* Después de muchas operaciones `DELETE` o `UPDATE`.
* Cuando se detecta fragmentación.
* Cuando las consultas se vuelven lentas y hay espacio desperdiciado.

#### Ejemplo de uso

```sql
OPTIMIZE TABLE ventas;

```

---

### ¿Qué es `ANALYZE TABLE`?

`ANALYZE TABLE` recalcula las estadísticas internas de la tabla para que el optimizador de consultas pueda elegir los mejores planes de ejecución. No modifica la estructura ni reorganiza los datos.

#### Cuándo usar `ANALYZE TABLE`

* Después de agregar o eliminar grandes volúmenes de datos.
* Cuando las consultas no están usando los índices correctamente.
* Antes de ejecutar consultas complejas luego de cambios masivos.

#### Ejemplo de uso

```sql
ANALYZE TABLE ventas;

```

---

### Comparativa entre OPTIMIZE y ANALYZE

| Característica | `OPTIMIZE TABLE` | `ANALYZE TABLE` |
| --- | --- | --- |
| **Reorganiza datos** | ✅ Sí | ❌ No |
| **Recalcula estadísticas** | ✅ A veces | ✅ Siempre |
| **Mejora espacio en disco** | ✅ Sí | ❌ No |
| **Bloqueo posible** | ✅ Puede bloquear | ⚠️ Generalmente no bloquea |
| **Uso recomendado** | Tras `UPDATE`/`DELETE` masivos | Tras grandes cambios en datos |

---

## Conclusión

Los índices aceleran las consultas `SELECT`, pero deben usarse estratégicamente para no afectar las escrituras.
