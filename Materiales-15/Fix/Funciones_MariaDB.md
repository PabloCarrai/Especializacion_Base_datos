# Funciones en MariaDB

## ¿Qué es una función?

Una función en MariaDB es un bloque de código reutilizable que realiza una operación y devuelve un único valor. A diferencia de los procedimientos almacenados, las funciones se pueden utilizar dentro de consultas SQL.

---

## Tipos de funciones en MariaDB

### Funciones nativas (*built-in*)

MariaDB incluye muchas funciones integradas que permiten realizar operaciones sobre:

* **Texto**: `CONCAT`, `SUBSTRING`, `LENGTH`
* **Números**: `ROUND`, `CEIL`, `FLOOR`
* **Fechas**: `NOW`, `CURDATE`, `DATEDIFF`
* **Agregación**: `SUM`, `AVG`, `COUNT`, `MAX`, `MIN`
* **Control**: `IF`, `CASE`

---

## Ejemplos de Funciones Nativas

### Funciones de Cadena (*String*)

* **`CONCAT(str1, str2, ...)`**
* **Descripción:** Concatena dos o más cadenas de texto en una sola.
* **Ejemplo:**
```sql
SELECT CONCAT('Hola', ' ', 'Mundo') AS resultado;
-- Resultado: 'Hola Mundo'

```




* **`SUBSTRING(str, pos, len)`**
* **Descripción:** Extrae una porción de una cadena a partir de una posición específica y con una longitud dada.
* **Ejemplo:**
```sql
SELECT SUBSTRING('MariaDB', 1, 5) AS resultado;
-- Resultado: 'Maria'

```




* **`LENGTH(str)`**
* **Descripción:** Devuelve la longitud de una cadena en bytes.
* **Ejemplo:**
```sql
SELECT LENGTH('Base de datos') AS resultado;
-- Resultado: 13

```





---

### Funciones Numéricas y Matemáticas

* **`ROUND(X, D)`**
* **Descripción:** Redondea un número a un número específico de decimales.
* **Ejemplo:**
```sql
SELECT ROUND(123.4567, 2) AS resultado;
-- Resultado: 123.46

```




* **`ABS(X)`**
* **Descripción:** Devuelve el valor absoluto de un número (su valor sin signo).
* **Ejemplo:**
```sql
SELECT ABS(-15.5) AS resultado;
-- Resultado: 15.5

```




* **`CEIL(X)`**
* **Descripción:** Redondea un número hacia arriba al entero más cercano.
* **Ejemplo:**
```sql
SELECT CEIL(4.2) AS resultado;
-- Resultado: 5

```





---

### Funciones de Fecha y Hora

* **`NOW()`**
* **Descripción:** Retorna la fecha y hora actual del sistema.
* **Ejemplo:**
```sql
SELECT NOW() AS resultado;
-- Resultado: 2026-10-22 13:52:00 (según la hora actual)

```




* **`DATEDIFF(expr1, expr2)`**
* **Descripción:** Calcula la diferencia en días entre dos fechas (restando `expr2` de `expr1`).
* **Ejemplo:**
```sql
SELECT DATEDIFF('2026-12-31', '2026-10-22') AS resultado;
-- Resultado: 70

```




* **`DATE_ADD(date, INTERVAL expr unit)`**
* **Descripción:** Suma un intervalo de tiempo (días, meses, horas, etc.) a una fecha dada.
* **Ejemplo:**
```sql
SELECT DATE_ADD('2026-10-22', INTERVAL 1 MONTH) AS resultado;
-- Resultado: '2026-11-22'

```





---

### Funciones de Control de Flujo

* **`IF(expr1, expr2, expr3)`**
* **Descripción:** Si la expresión `expr1` es verdadera, devuelve `expr2`; de lo contrario, devuelve `expr3`.
* **Ejemplo:**
```sql
SELECT IF(10 > 5, 'Mayor', 'Menor') AS resultado;
-- Resultado: 'Mayor'

```




* **`COALESCE(val1, val2, ...)`**
* **Descripción:** Retorna el primer valor no nulo (`NOT NULL`) de la lista.
* **Ejemplo:**
```sql
SELECT COALESCE(NULL, NULL, 'Valor por defecto') AS resultado;
-- Resultado: 'Valor por defecto'

```





---

### Funciones de Información y Sistema

* **`VERSION()`**
* **Descripción:** Devuelve una cadena con la versión actual del servidor MariaDB.
* **Ejemplo:**
```sql
SELECT VERSION() AS resultado;
-- Resultado: '10.11.2-MariaDB' (o la versión específica instalada)

```




* **`USER()`**
* **Descripción:** Devuelve el nombre de usuario y el host actual con el que se realizó la conexión.
* **Ejemplo:**
```sql
SELECT USER() AS resultado;
-- Resultado: 'root@localhost'

```





---

## Funciones definidas por el usuario (UDF)

Son funciones creadas por el usuario utilizando `CREATE FUNCTION`. Permiten encapsular lógica y reutilizarla.

### Sintaxis básica

```sql
DELIMITER //

CREATE FUNCTION calcular_iva(precio DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
  DECLARE resultado DECIMAL(10,2);
  SET resultado = precio * 0.21;
  RETURN resultado;
END //

DELIMITER ;

```

### Cómo se usan

```sql
SELECT calcular_iva(100); -- devuelve 21.00

```

---

## Diferencias entre funciones y procedimientos

| Característica | Función | Procedimiento |
| --- | --- | --- |
| **Devuelve un valor** | Sí (obligatorio) | No necesariamente |
| **Uso en SELECT** | Sí | No |
| **Sintaxis de llamada** | `SELECT mi_funcion();` | `CALL mi_procedimiento();` |

---

## Usos comunes de funciones

* Realizar cálculos dentro de una consulta
* Validar o transformar datos
* Encapsular lógica de negocio reutilizable
* Componer expresiones complejas

### Ejemplo de una función dentro de una consulta

```sql
SELECT nombre, calcular_iva(precio) AS precio_con_iva
FROM productos;

```

---

## Buenas prácticas

* Usar nombres descriptivos.
* Documentar qué hace la función.
* Marcar como `DETERMINISTIC` si no cambia con el tiempo.
* Evitar funciones que realicen operaciones de escritura.

---

## Uso de DETERMINISTIC y NOT DETERMINISTIC en funciones

En MariaDB, al crear una función, se debe indicar si es determinística o no determinística:

* **`DETERMINISTIC`**: significa que la función siempre devuelve el mismo resultado para los mismos valores de entrada.
* **`NOT DETERMINISTIC`**: significa que el resultado puede cambiar, aunque los argumentos sean iguales.

### ¿Por qué es importante?

Esto ayuda al optimizador de consultas a decidir si puede almacenar en caché los resultados o si debe recalcularlos cada vez.

### Ejemplo de función determinística

```sql
DELIMITER //

CREATE FUNCTION doble(valor INT)
RETURNS INT
DETERMINISTIC
BEGIN
  RETURN valor * 2;
END //

DELIMITER ;

```

*(Este resultado siempre será igual si el valor de entrada es el mismo)*

### Ejemplo de función no determinística

```sql
DELIMITER //

CREATE FUNCTION hoy()
RETURNS DATE
NOT DETERMINISTIC
BEGIN
  RETURN CURDATE();
END //

DELIMITER ;

```

*(Aunque no reciba parámetros, devuelve el valor actual del sistema, que cambia cada día)*

---

## Otros ejemplos de UDF

### 1. Calcular edad

```sql
DELIMITER //

CREATE FUNCTION calcular_edad(fecha_nac DATE)
RETURNS INT
DETERMINISTIC
BEGIN
  DECLARE edad INT;
  SET edad = TIMESTAMPDIFF(YEAR, fecha_nac, CURDATE());
  RETURN edad;
END //

DELIMITER ;

```

**Uso:**

```sql
SELECT nombre, calcular_edad(fecha_nacimiento) AS edad FROM personas;

```

### 2. Validar un DNI (solo numérico y longitud entre 7 y 8 dígitos)

```sql
DELIMITER //

CREATE FUNCTION validar_dni(dni VARCHAR(10))
RETURNS BOOLEAN
DETERMINISTIC
BEGIN
  DECLARE es_valido BOOLEAN;
  SET es_valido = dni REGEXP '^[0-9]{7,8}$';
  RETURN es_valido;
END //

DELIMITER ;

```

**Uso:**

```sql
SELECT nombre, validar_dni(dni) AS dni_valido FROM ciudadanos;

```

### 3. Función con CASE para clasificar edad

```sql
DELIMITER //

CREATE FUNCTION clasificar_edad(edad INT)
RETURNS VARCHAR(20)
DETERMINISTIC
BEGIN
  RETURN CASE
    WHEN edad < 13 THEN 'Niño'
    WHEN edad BETWEEN 13 AND 17 THEN 'Adolescente'
    WHEN edad BETWEEN 18 AND 64 THEN 'Adulto'
    ELSE 'Mayor'
  END;
END //

DELIMITER ;

```

**Uso combinado:**

```sql
SELECT nombre, calcular_edad(fecha_nacimiento) AS edad,
       clasificar_edad(calcular_edad(fecha_nacimiento)) AS grupo_edad
FROM personas;

```

---

## Más ejemplos

### Funciones `DETERMINISTIC`

#### Función para convertir Celsius a Fahrenheit

```sql
DELIMITER //

CREATE FUNCTION celsius_a_fahrenheit(celsius DECIMAL(5,2))
RETURNS DECIMAL(5,2)
DETERMINISTIC
BEGIN
  RETURN (celsius * 9 / 5) + 32;
END //

DELIMITER ;

```

#### Función para calcular el área de un círculo

```sql
DELIMITER //

CREATE FUNCTION area_circulo(radio DECIMAL(5,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
  RETURN PI() * POW(radio, 2);
END //

DELIMITER ;

```

---

### Funciones `NOT DETERMINISTIC`

#### Función que devuelve la fecha y hora actuales

```sql
DELIMITER //

CREATE FUNCTION obtener_fecha_hora()
RETURNS DATETIME
NOT DETERMINISTIC
BEGIN
  RETURN NOW();
END //

DELIMITER ;

```

*(Cada vez que se llama, puede devolver un valor diferente porque depende del reloj del sistema).*

#### Función que devuelve un valor aleatorio

```sql
DELIMITER //

CREATE FUNCTION numero_azar(limite INT)
RETURNS INT
NOT DETERMINISTIC
BEGIN
  RETURN FLOOR(RAND() * limite);
END //

DELIMITER ;

```

*(RAND() devuelve un número decimal aleatorio entre 0 y 1, por lo tanto, el resultado no es predecible).*

* **`RAND()`**: Devuelve un número decimal aleatorio entre 0 y 1 (por ejemplo: `0.736`).
* **`RAND() * limite`**: Escala ese número aleatorio al rango deseado (por ejemplo, si `limite = 10`, el resultado será entre 0 y 9.999).
* **`FLOOR(...)`**: Aplica la función piso, que elimina la parte decimal y devuelve solo la parte entera (por ejemplo, `FLOOR(9.999) → 9`).

---

## ¿Para qué se usa `DELIMITER //` en MariaDB?

Cuando se define una función o un procedimiento almacenado, se usan múltiples líneas de código que incluyen puntos y coma (`;`) dentro del cuerpo de la función.

Como el punto y coma es el delimitador estándar de comandos en SQL, si no lo cambiamos, MariaDB pensaría que la función termina en la primera línea que contenga `;`.

Por eso usamos `DELIMITER //` (o cualquier otro símbolo que no sea `;`) para indicar a MariaDB dónde realmente termina la definición de la función.

### Al final, se vuelve a definir el delimitador estándar:

```sql
DELIMITER ;

```

* Se cambia el delimitador de comandos SQL para que MariaDB no se confunda con los puntos y coma (`;`) que están dentro del cuerpo de la función. Pero una vez que termina la definición, es necesario volver al delimitador por defecto (`;`) porque de no hacerlo:
* Todas las sentencias siguientes (`SELECT`, `INSERT`, `UPDATE`, etc.) usarán `//` si no lo cambiás de nuevo.
* Se pueden generar errores inesperados al tratar de ejecutar consultas simples y MariaDB sigue esperando el `//` como final.


* Facilita la lectura y mantenimiento del código, sobre todo si se ejecuta como script.
* No es obligatorio en términos del lenguaje, pero sí altamente recomendado y esperado para evitar problemas.

---

## Creación de las tablas para los ejemplos

```sql
-- Crear la base de datos
CREATE DATABASE IF NOT EXISTS funciones_db;
USE funciones_db;

-- Tabla 1: personas (para funciones calcular_edad y clasificar_edad)
CREATE TABLE personas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50),
    fecha_nacimiento DATE
);

INSERT INTO personas (nombre, fecha_nacimiento) VALUES
('Juan Pérez', '1990-05-10'),
('Ana Gómez', '2010-08-25'),
('Carlos Ruiz', '1955-01-15'),
('Lucía Torres', '2007-12-01');

-- Tabla 2: ciudadanos (para validar_dni)
CREATE TABLE ciudadanos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50),
    dni VARCHAR(10)
);

INSERT INTO ciudadanos (nombre, dni) VALUES
('Miguel Herrera', '30544879'),
('María Blanco', '123456'),
('Sofía Martín', '45678901'),
('Pedro Díaz', '78A45632');

-- Tabla 3: productos (para calcular_iva y otros)
CREATE TABLE productos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50),
    precio DECIMAL(10,2)
);

INSERT INTO productos (nombre, precio) VALUES
('Laptop', 1200.00),
('Mouse', 25.50),
('Teclado', 55.00),
('Monitor', 300.00);

```
