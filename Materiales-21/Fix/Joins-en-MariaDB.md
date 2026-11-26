# Joins en MaríaDB

Para comprender cómo funcionan los JOINs en SQL, el secreto es entender que no estamos fusionando tablas al azar, sino buscando coincidencias basándonos en un dato común.

Por ejemplo si tenemos una lista de Clientes (Tabla A) y una lista de Pedidos (Tabla B). Para saber qué compró cada persona, ambas tablas deben compartir una columna de conexión, que en este caso es el ID del cliente (`ID = Cliente_ID`).

A continuación, se detalla qué hace el motor de la base de datos en cada caso:

## 1. INNER JOIN (La Intersección Exacta)

Busca únicamente las filas que tienen una coincidencia exacta en ambas tablas.

* **Cómo funciona:** El motor toma a "Ana" (ID 1) y busca si hay un ID 1 en Pedidos (lo encuentra: "laptop"). Hace lo mismo con "Beto" (ID 2  "mouse"). Al llegar a "Clara" (ID 3), como no tiene ningún pedido asignado en la otra tabla, la ignora por completo. Lo mismo ocurre con el pedido del "teclado" (ID 4), que al no tener un cliente registrado, no se muestra.
* **Cuándo usarlo:** Cuando se necesita  información completa y relacionada, donde la falta de un dato anula la utilidad de la fila (por ejemplo, facturas que sí o sí deben tener un cliente asociado).

## 2. LEFT JOIN (La Prioridad de la Izquierda)

Garantiza que la tabla de la izquierda (la que se escribe primero en el FROM) se muestre completa, sin importar si tiene o no relación con la otra.

* **Cómo funciona:** Muestra a todos los clientes: "Ana", "Beto" y "Clara". Como "Clara" no tiene pedidos, el motor de base de datos no puede dejar ese espacio vacío, de modo que lo rellena con un valor especial: `NULL` (que significa "sin datos"). El pedido de "teclado" (ID 4) sigue quedando afuera porque pertenece a la tabla derecha y no tiene coincidencia a la izquierda.
* **Cuándo usarlo:** Cuando se necesita un listado completo (por ejemplo, todos tus alumnos, todos tus clientes o todos tus productos) y querés ver qué tienen asociado, sin que los que no tienen actividad queden invisibilizados.

## 3. RIGHT JOIN (La Prioridad de la Derecha)

Es el hermano invertido del LEFT JOIN. Prioriza que la tabla de la derecha (la que se escribe después del JOIN) aparezca de forma íntegra.

* **Cómo funciona:** El motor se asegura de que todos los pedidos se muestren: "laptop", "mouse" y "teclado". Al llegar al "teclado" (ID 4), como no hay un cliente con ID 4 en la Tabla A, completa los datos del cliente con `NULL`. En este caso, "Clara" queda totalmente afuera de los resultados porque es de la izquierda y no compró nada.
* **Cuándo usarlo:** Para control de calidad o auditorías (por ejemplo, encontrar "pedidos huérfanos" que no tienen un cliente asignado, lo que podría indicar un error en el sistema).

## 4. FULL OUTER JOIN (La Unión Total)

No quiere dejar a nadie afuera. Trae todos los registros de la Tabla A y todos los de la Tabla B, coincidan o no.

* **Cómo funciona:** Muestra las uniones que sí existen ("Ana" con "laptop", "Beto" con "mouse") y, para los casos sin pareja, rellena con `NULL` en el lado que haga falta. Así, verás a "Clara" con su pedido en `NULL` y también al "teclado" con su cliente en `NULL`. (Nota: Recordá que en MariaDB esto se simula uniendo un LEFT JOIN y un RIGHT JOIN mediante un UNION).
* **Cuándo usarlo:** Para diagnósticos profundos de datos o para fusionar sistemas independientes donde necesitás una vista global de lo que coincide y de lo que quedó suelto en ambos lados.

---

## Ejemplo

Para probar directamente estas consultas en MariaDB, primero se presenta el script de creación e inserción de las tablas con los mismos datos del esquema (clientes y pedidos), seguido del código SQL de cada tipo de JOIN.

### Script de preparación del entorno (MariaDB)

> (Nota: Para permitir el pedido huérfano con `cliente_id = 4` en la demostración del RIGHT JOIN, se omite intencionalmente la restricción `FOREIGN KEY` en la tabla pedidos).

```sql
CREATE DATABASE IF NOT EXISTS ejemplos_joins;
USE ejemplos_joins;

CREATE TABLE clientes (
    id INT PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL
);

CREATE TABLE pedidos (
    cliente_id INT,
    producto VARCHAR(50) NOT NULL
);

INSERT INTO clientes (id, nombre) VALUES 
(1, 'Ana'),
(2, 'Beto'),
(3, 'Clara');

INSERT INTO pedidos (cliente_id, producto) VALUES 
(1, 'laptop'),
(2, 'mouse'),
(4, 'teclado');

```

### INNER JOIN (Solo coincidencias exactas)

Devuelve únicamente a Ana y a Beto, ya que son los únicos registros cuyo identificador coincide en ambas tablas.

```sql
SELECT 
    c.id,
    c.nombre,
    p.producto
FROM clientes c
INNER JOIN pedidos p ON c.id = p.cliente_id;

```

### LEFT JOIN (Toda la izquierda + coincidencias)

Muestra todos los clientes de la tabla izquierda (clientes). Como Clara (`id = 3`) no tiene pedidos registrados, la columna producto devuelve `NULL`.

```sql
SELECT 
    c.id,
    c.nombre,
    p.producto
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id;

```

### RIGHT JOIN (Toda la derecha + coincidencias)

Muestra todos los pedidos de la tabla derecha (pedidos). Al llegar al registro del teclado (`cliente_id = 4`), como no existe un cliente con ese ID en clientes, los campos id y nombre devuelven `NULL` (en este caso se proyecta `p.cliente_id` para visualizar el identificador huérfano).

```sql
SELECT 
    p.cliente_id,
    c.nombre,
    p.producto
FROM clientes c
RIGHT JOIN pedidos p ON c.id = p.cliente_id;

```

### FULL OUTER JOIN (Simulación en MariaDB)

Dado que MariaDB no admite la palabra clave FULL OUTER JOIN de manera nativa, se emula combinando el resultado de un LEFT JOIN con el de un RIGHT JOIN mediante el operador UNION (que automáticamente elimina las filas duplicadas de Ana y Beto).

```sql
SELECT 
    c.id,
    c.nombre,
    p.producto
FROM clientes c
LEFT JOIN pedidos p ON c.id = p.cliente_id

UNION

SELECT 
    p.cliente_id AS id,
    c.nombre,
    p.producto
FROM clientes c
RIGHT JOIN pedidos p ON c.id = p.cliente_id;

```
