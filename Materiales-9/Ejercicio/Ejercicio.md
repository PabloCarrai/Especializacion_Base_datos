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

