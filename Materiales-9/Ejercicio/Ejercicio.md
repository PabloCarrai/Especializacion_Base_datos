> Ejercicio 54

[Ejercicio](https://github.com/PabloCarrai/Especializacion_Base_datos/blob/main/Materiales-9/Fix/Tipos-de-datos-en-MariaDB.md#53-actividades-pr%C3%A1cticas-y-autoevaluaci%C3%B3n)


[Ejercicio 1](https://github.com/PabloCarrai/Especializacion_Base_datos/blob/main/Materiales-9/Fix/Tipos-de-datos-en-MariaDB.md#ejercicio-1--identificar-el-tipo-%C3%B3ptimo
) — Identificar el tipo óptimo

| Campo | Tipo Propuesto |
|---|---|
| Nombre  |  varchar(50)  |
| Edad    |  tinyint unsigned |
| Sueldo  |  decimal(12,2)  |
| Fecha de nacimiento    |   date  |
| Teléfono     |   char(10)    |
| Activo  |  boolean  |
| Código postal  |   char(8)  |
| Observaciones   |  text  |
| Cantidad de productos  |   smallint unsigned |
| Hora de entrada |  time  |


[Ejercicio 2](https://github.com/PabloCarrai/Especializacion_Base_datos/blob/main/Materiales-9/Fix/Tipos-de-datos-en-MariaDB.md#ejercicio-2--dise%C3%B1o-de-tabla-productos) — Diseñar una tabla

Crear una tabla llamada:
productos
que contenga:
id
codigo
nombre
descripcion
precio
stock
activo
fecha_alta
Los alumnos deben decidir:
- tipo de cada columna;
- longitud;
- NULL/NOT NULL;
- DEFAULT;
- clave primaria;
- AUTO_INCREMENT.


```sql

create table productos(
	id int auto_increment primary key,
	codigo char(14) not null,
	nombre varchar(50) not null,
	descripcion text,
	precio decimal(12,2)  not null,
	stock tinyint unsigned default 0, 
	activo boolean default true,
	fecha_alta date
);


```





 Ejercicio 3 — Detectar errores

```sql

Analizar la siguiente tabla:
CREATE TABLE empleados (
    id BIGINT,
    nombre TEXT,
    edad VARCHAR(3),
    sueldo FLOAT,
    fecha_nacimiento VARCHAR(10),
    telefono BIGINT,
    activo VARCHAR(5)
);

```

Preguntas:
1. ¿Qué tipos podrían modificarse?
> **Respuesta:** id, nombre, sueldo,fecha_nacimiento,telefono, activo 

2. ¿Qué columnas deberían ser NOT NULL?
> **Respuesta:** nombre, sueldo, fecha_nacimiento.

3. ¿Cuál debería ser la clave primaria?
> **Respuesta:** id.

4. ¿Dónde utilizarían UNSIGNED?
> **Respuesta:** id, sueldo.

5. ¿Qué tipo utilizarían para el sueldo?
> **Respuesta:** decimal.

6. ¿Qué tipo utilizarían para la fecha?
> **Respuesta:** date.

7. ¿Qué tipo utilizarían para activo?
> **Respuesta:** boolean.

8. ¿Qué problema puede existir con el teléfono?
> **Respuesta:** No permite caracter +.

9. ¿Realmente necesitamos almacenar la edad?
> **Respuesta:** No hace falta ya que existe fecha_nacimiento y se puede calcular la edad.


Texto →

Ejercicio 4 — ¿Número o texto?
Indicar si cada dato debería analizarse como número o como texto:
12345
0012345
+54 11 4567-8900
100
000100
2026
00123456

La clave del ejercicio es que no debemos mirar solamente los caracteres que forman el valor.
Debemos preguntarnos:
¿Qué representa el dato?




#	Falta lo de abajo








. Ejercicio 5 — Crear e insertar datos


> Crear la tabla:


```sql


CREATE TABLE personas (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    apellido VARCHAR(100) NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    fecha_nacimiento DATE,
    salario DECIMAL(12,2),
    activo BOOLEAN DEFAULT TRUE
);

```



Insertar al menos cinco personas:



```sql


INSERT INTO personas
(apellido, nombre, fecha_nacimiento, salario, activo)
VALUES
('García', 'Juan', '1985-03-12', 850000.50, TRUE),
('Pérez', 'María', '1990-08-25', 920000.00, TRUE),
('López', 'Carlos', '1978-11-03', 1100000.75, TRUE),
('Gómez', 'Ana', '1995-06-18', 780000.00, FALSE),
('Díaz', 'Pedro', '1969-04-15', 1250000.25, TRUE);


```





Luego ejecutar:


```sql

SELECT *
FROM personas;


```



59. Ejercicio 6 — Consultas
Realizar las siguientes consultas:

Personas activas



```sql



SELECT *
FROM personas
WHERE activo = TRUE;



```



Personas con salario superior a determinado valor

```sql

SELECT *
FROM personas
WHERE salario > 900000;

```



Ordenar por salario

```sql

SELECT *
FROM personas
ORDER BY salario DESC;

```



Obtener el salario promedio

```sql

SELECT AVG(salario)
FROM personas;


```



Obtener el año de nacimiento


```sql

SELECT
    nombre,
    apellido,
    YEAR(fecha_nacimiento) AS anio_nacimiento
FROM personas;



```

60. Actividad final

Responder:

1. ¿Por qué no utilizar VARCHAR para almacenar un precio?
2. ¿Por qué un número de teléfono puede ser texto?
3. ¿Por qué DATE es preferible a VARCHAR para una fecha?
4. ¿Por qué una edad puede ser TINYINT UNSIGNED?
5. ¿Por qué un código 001234 podría ser CHAR o VARCHAR y no INT?
6. ¿Qué diferencia conceptual existe entre:

0
FALSE
NULL
''

7. ¿Por qué puede ser problemático almacenar simultáneamente la fecha de nacimiento y la edad?

