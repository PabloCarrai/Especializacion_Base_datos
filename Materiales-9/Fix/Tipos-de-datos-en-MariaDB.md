---

## 1. Introducción

Cuando diseñamos una base de datos en MariaDB, cada columna de una tabla debe tener asociado un **tipo de dato**. El tipo de dato indica qué clase de información puede almacenar esa columna y determina, entre otras cosas:

* qué valores son válidos;
* cómo se almacenan físicamente;
* qué operaciones pueden realizarse;
* cómo se comparan y ordenan;
* cuánto espacio pueden ocupar;
* y, en determinados casos, cómo puede afectar su elección al rendimiento.

### Ejemplo práctico

```sql
CREATE TABLE personas (
    id INT,
    nombre VARCHAR(100),
    fecha_nacimiento DATE,
    salario DECIMAL(12,2)
);

```

En esta tabla:

| Columna | Tipo de dato | Qué almacena |
| --- | --- | --- |
| `id` | `INT` | Número entero |
| `nombre` | `VARCHAR(100)` | Texto |
| `fecha_nacimiento` | `DATE` | Fecha |
| `salario` | `DECIMAL(12,2)` | Número decimal |

> **Nota de diseño:** La elección del tipo de dato no debería hacerse solamente pensando en *"qué tipo acepta el valor"*. También debemos preguntarnos: **¿Qué representa realmente ese dato y qué operaciones vamos a realizar sobre él?**

---

## 2. Clasificación de los Tipos de Datos

Los tipos de datos de MariaDB pueden agruparse de la siguiente manera:

```text
TIPOS DE DATOS
├── 🔢 Numéricos
│   ├── Enteros (TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT)
│   └── Decimales (DECIMAL, FLOAT, DOUBLE)
├── 📝 Cadenas de caracteres (CHAR, VARCHAR, TEXT)
├── 📅 Fecha y hora (DATE, TIME, DATETIME, TIMESTAMP, YEAR)
├── ✅ Booleanos (BOOLEAN / BOOL)
├── 📦 Binarios (BINARY, VARBINARY, BLOB)
├── 🔠 Enumeraciones (ENUM)
└── 🌐 Datos semiestructurados (JSON)

```

---

## 3. Tipos Numéricos

Los tipos numéricos permiten almacenar números y se dividen principalmente en dos categorías:

* **Números enteros** (sin parte decimal).
* **Números con parte decimal** (precisión exacta o aproximada).

---

## 4. Números enteros

Los números enteros no tienen parte decimal. Ejemplos comunes: `0`, `1`, `10`, `57`, `1000`, `-25`.

MariaDB dispone de los siguientes tipos enteros:

| Tipo | Tamaño | Rango con signo (`SIGNED`) | Rango sin signo (`UNSIGNED`) |
| --- | --- | --- | --- |
| `TINYINT` | 1 byte | `-128` a `127` | `0` a `255` |
| `SMALLINT` | 2 bytes | `-32.768` a `32.767` | `0` a `65.535` |
| `MEDIUMINT` | 3 bytes | `-8.388.608` a `8.388.607` | `0` a `16.777.215` |
| `INT` | 4 bytes | `-2.147.483.648` a `2.147.483.647` | `0` a `4.294.967.295` |
| `BIGINT` | 8 bytes | `-2⁶³` a `2⁶³-1` | `0` a `2⁶⁴-1` |

---

## 5. TINYINT

`TINYINT` ocupa **1 byte**.

* **Con signo:** `-128` a `127`
* **Sin signo:** `0` a `255`

### Ejemplo de uso

```sql
CREATE TABLE personas (
    nombre VARCHAR(100),
    edad TINYINT UNSIGNED
);

INSERT INTO personas VALUES ('Juan', 35);

```

Una edad puede representarse perfectamente con `TINYINT UNSIGNED`.

### ¿Cuándo utilizarlo?

Puede ser apropiado para:

* Edades
* Meses y días
* Pequeños contadores y valores de estado
* Cantidades pequeñas (ej. `mes TINYINT UNSIGNED`)

---

## 6. SMALLINT

`SMALLINT` ocupa **2 bytes**. Puede ser apropiado para valores que superan el rango de `TINYINT`, pero que tampoco necesitan un `INT`.

### Ejemplos

* `stock SMALLINT UNSIGNED` (si sabemos que el stock nunca va a superar `65.535` unidades).
* Cantidad de alumnos o empleados.
* Años (dependiendo del diseño) y pequeños contadores.

---

## 7. MEDIUMINT

`MEDIUMINT` ocupa **3 bytes**. Su utilización es menos frecuente que `INT`, pero puede ser útil cuando necesitamos un rango intermedio (`cantidad MEDIUMINT UNSIGNED`).

> **Advertencia:** No es necesario utilizarlo simplemente porque existe. En muchos sistemas, `INT` es una opción más sencilla y habitual.

---

## 8. INT

`INT` ocupa **4 bytes**. Es probablemente el tipo entero de propósito general más utilizado.

### Ejemplos

```sql
-- Uso básico
id INT
cantidad INT

-- Como clave primaria autoincremental
id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY

```

Esto último permite disponer de identificadores numéricos generados automáticamente de manera eficiente.

---

## 9. BIGINT

`BIGINT` ocupa **8 bytes**. Se utiliza cuando necesitamos un rango de valores mucho mayor que el proporcionado por `INT` (ej. `id BIGINT UNSIGNED`).

### ¿Cuándo es apropiado?

* Sistemas con enormes cantidades de registros.
* Identificadores generados por determinados sistemas distribuidos.
* Contadores web masivos o métricas a gran escala.

> ⚠️ **No utilizar BIGINT por defecto:** No sería necesario definir `edad BIGINT` porque una edad nunca necesita ese rango. Es preferible utilizar `edad TINYINT UNSIGNED`.

---

## 10. SIGNED y UNSIGNED

Los tipos enteros pueden utilizarse con signo (`SIGNED`) o sin signo (`UNSIGNED`). Por defecto, en MariaDB los tipos enteros son `SIGNED`.

* **Ejemplo con signo:** `edad TINYINT` (permite valores negativos, lo cual no tiene sentido biológico para una edad).
* **Ejemplo sin signo:** `edad TINYINT UNSIGNED` (el rango comienza en `0`).

Otros ejemplos donde tiene sentido `UNSIGNED`: `stock INT UNSIGNED`, `cantidad INT UNSIGNED`, `id INT UNSIGNED`.

> **Regla práctica:** Si un dato conceptualmente nunca puede ser negativo, debemos evaluar si `UNSIGNED` es apropiado.

---

## 11. Un error frecuente: utilizar INT para todo

Un principiante podría cometer el siguiente error de diseño:

```sql
CREATE TABLE productos (
    id BIGINT,
    stock BIGINT,
    categoria BIGINT,
    descuento BIGINT
);

```

Aunque técnicamente algunos valores puedan almacenarse, **no es un buen diseño**. Debemos analizar cada dato en su contexto:

```sql
CREATE TABLE productos (
    id INT UNSIGNED,
    stock INT UNSIGNED,
    categoria INT UNSIGNED,
    descuento DECIMAL(5,2)
);

```

Esta estructura expresa mucho mejor qué representa cada columna y optimiza el uso de memoria.

---

## 12. Tipos Decimales

Cuando necesitamos almacenar números con parte decimal tenemos varias alternativas:

* `DECIMAL` (precisión exacta)
* `FLOAT` (punto flotante aproximado)
* `DOUBLE` (punto flotante de doble precisión)

Ejemplos de valores: `10.50`, `1250.75`, `3.14159`, `0.00125`.

---

## 13. DECIMAL

`DECIMAL` permite almacenar valores decimales con **precisión exacta**. Su sintaxis es:

```sql
DECIMAL(M,D)

```

Donde:

* **`M`** (Precision): Representa la cantidad total de dígitos.
* **`D`** (Scale): Representa la cantidad de dígitos después del separador decimal.

### Ejemplo

```sql
precio DECIMAL(10,2)

```

Significa que se permiten hasta 10 dígitos en total, de los cuales 2 corresponden a la parte decimal (ej. un valor posible sería `12345678.90`).

---

## 14. ¿Por qué utilizar DECIMAL para dinero?

Para importes monetarios normalmente conviene utilizar `DECIMAL` para evitar errores de redondeo inherentes a los tipos flotantes.

### Ejemplo

```sql
CREATE TABLE productos (
    id INT UNSIGNED,
    nombre VARCHAR(100),
    precio DECIMAL(12,2)
);

INSERT INTO productos VALUES (1, 'Teclado', 45999.90);

```

### Es apropiado para:

* Precios, salarios, impuestos y facturación.
* Saldos bancarios y contables.
* Porcentajes que requieran precisión decimal exacta.

> **Regla práctica:** Para valores monetarios, utilizar siempre `DECIMAL` y nunca `FLOAT`.

---

## 15. FLOAT y DOUBLE

`FLOAT` y `DOUBLE` son tipos de **punto flotante**. Están pensados para representar números con una determinada aproximación.

### ¿Cuándo utilizarlos?

* Mediciones científicas o físicas (`temperatura FLOAT`, `distancia DOUBLE`).
* Cálculos donde la representación de punto flotante sea adecuada y la precisión exacta no sea crítica.

```sql
CREATE TABLE mediciones (
    temperatura FLOAT,
    presion DOUBLE
);

```

---

## 16. Tabla Comparativa: DECIMAL vs FLOAT/DOUBLE

| Situación | Tipo habitual |
| --- | --- |
| Dinero, Precios, Salarios, Impuestos | `DECIMAL` |
| Medición aproximada o física | `FLOAT` |
| Cálculos científicos de alta precisión | `DOUBLE` |
| Grandes valores con decimales | `DOUBLE` |

> ⚠️ **No debemos elegir `FLOAT` simplemente porque "tiene decimales".**

---

## 17. Cadenas de Caracteres

Los datos de texto pueden almacenarse principalmente mediante tres tipos:

* `CHAR` (longitud fija)
* `VARCHAR` (longitud variable)
* `TEXT` (textos extensos)

Ejemplos: `Juan`, `Argentina`, `Buenos Aires`, `Administración de Sistemas`.

---

## 18. CHAR

`CHAR` representa cadenas de **longitud fija**.

### Ejemplo

```sql
codigo CHAR(8)

```

Es apropiado cuando el dato tiene una longitud estrictamente fija o muy estable (ej. códigos como `A1234567`, `B1234567`, o códigos postales/provinciales como `provincia_codigo CHAR(2)`).

---

## 19. VARCHAR

`VARCHAR` permite almacenar cadenas de **longitud variable**. Es uno de los tipos de texto más utilizados.

### Ejemplo

```sql
nombre VARCHAR(100)

```

Permite almacenar nombres como `Juan`, `Leonardo`, `Alejandro` o `María` sin necesidad de rellenar con espacios vacíos ni obligar a que todos tengan la misma longitud.

### Otros campos típicos:

`apellido VARCHAR(100)`, `email VARCHAR(150)`, `domicilio VARCHAR(200)`.

---

## 20. CHAR vs VARCHAR: Regla Práctica

* **`CHAR`:** Cuando la longitud es fija (`codigo CHAR(8)`).
* **`VARCHAR`:** Cuando la longitud es variable (`nombre VARCHAR(100)`). No tendría sentido usar `CHAR(100)` si los nombres reales varían drásticamente en longitud.

---

## 21. TEXT

`TEXT` está pensado para almacenar cantidades mayores de texto que superan los límites habituales de `VARCHAR`.

### Ejemplo

```sql
CREATE TABLE noticias (
    id INT UNSIGNED,
    titulo VARCHAR(200),
    contenido TEXT
);

```

### ¿Cuándo utilizarlo?

* Comentarios, observaciones y artículos.
* Descripciones extensas y contenido de documentos.

---

## 22. VARCHAR vs TEXT

* **`VARCHAR`:** Para textos relativamente cortos y con una longitud máxima razonablemente definida (`nombre`, `apellido`, `email`, `usuario`, `domicilio`, `título`).
* **`TEXT`:** Para textos potencialmente mucho más extensos (`observaciones`, `artículos`, `comentarios`, `contenido`).

> ⚠️ **No debemos utilizar `TEXT` para todo simplemente porque "no sabemos cuánto texto habrá".** Los campos `VARCHAR` se procesan de manera más eficiente en memoria temporal y operaciones de ordenamiento.

---

## 23. Caracteres y Bytes

Cuando trabajamos con texto debemos distinguir claramente entre **caracteres** y **bytes**. Una cadena puede contener caracteres acentuados (*Administración*) o símbolos de otros alfabetos.

Por eso son fundamentales el juego de caracteres (`character set`) y la intercalación (`collation`), que determinan cómo se almacenan, comparan y ordenan los textos.

> **Recomendación:** En aplicaciones modernas suele ser conveniente trabajar con **`utf8mb4`** cuando necesitamos una representación amplia y robusta de Unicode (incluyendo emojis y caracteres especiales).

---

## 24. Tipos de Fecha y Hora

MariaDB dispone de varios tipos especializados para gestionar marcas temporales:
`DATE`, `TIME`, `DATETIME`, `TIMESTAMP`, `YEAR`. La elección depende estrictamente de qué deseemos representar.

---

## 25. DATE

`DATE` almacena una fecha calendario sin hora.

* **Formato habitual:** `YYYY-MM-DD` (Ejemplo: `1969-04-15`).
* **Ejemplos:** `fecha_nacimiento DATE`, `fecha_ingreso DATE`, `fecha_vencimiento DATE`.

> Si no necesitamos almacenar la hora, `DATE` es siempre la opción correcta.

---

## 26. TIME

`TIME` almacena una hora exacta o un intervalo de tiempo transcurrido.

* **Ejemplo:** `hora_inicio TIME` (Valor: `08:30:00`).
* **Usos habituales:** Horarios de entrada/salida, turnos, duración de actividades o intervalos.

---

## 27. DATETIME

`DATETIME` almacena fecha y hora combinadas.

* **Formato habitual:** `YYYY-MM-DD HH:MM:SS` (Ejemplo: `2026-09-16 11:30:00`).
* **Usos:** Es indispensable cuando necesitamos conocer exactamente en qué instante preciso ocurrió un evento o transacción.

```sql
CREATE TABLE operaciones (
    id INT UNSIGNED,
    descripcion VARCHAR(200),
    fecha_operacion DATETIME
);

```

---

## 28. TIMESTAMP

`TIMESTAMP` también almacena fecha y hora, pero incluye características específicas relacionadas con la conversión automática de zonas horarias y el manejo de marcas temporales del sistema (`creado`, `actualizado`).

### Ejemplo

```sql
CREATE TABLE usuarios (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    usuario VARCHAR(50),
    creado TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

Al insertar un registro (`INSERT INTO usuarios (usuario) VALUES ('leonardo');`), MariaDB completa automáticamente la fecha y hora exacta de creación.

---

## 29. DATETIME vs TIMESTAMP

No son exactamente equivalentes. Una guía simplificada para elegir:

| Necesidad | Opción recomendada |
| --- | --- |
| Fecha de nacimiento / ingreso / vencimiento | `DATE` |
| Fecha y hora específica de un evento comercial | `DATETIME` |
| Marca temporal automática de auditoría (creación/modificación) | `TIMESTAMP` |

> ❌ **Evitar errores:** Para una fecha de nacimiento (`fecha_nacimiento DATE`) no tiene sentido usar `TIMESTAMP`, ya que no requerimos la hora exacta ni conversiones de zona horaria del servidor.

---

## 30. YEAR

`YEAR` permite representar un año (en formato de 4 dígitos por defecto).

* **Ejemplo:** `anio YEAR`.
* Es útil cuando el dato requerido es exclusivamente un año. Sin embargo, antes de utilizarlo, evalúa si es más conveniente almacenar la fecha completa mediante `DATE`.

---

## 31. BOOLEAN

MariaDB permite utilizar `BOOLEAN` y `BOOL`. Estos tipos se manejan internamente como un alias del tipo numérico **`TINYINT(1)`**.

* `0` = `FALSE`
* `1` = `TRUE`

### Ejemplo

```sql
CREATE TABLE usuarios (
    id INT UNSIGNED,
    nombre VARCHAR(100),
    activo BOOLEAN
);

INSERT INTO usuarios VALUES 
(1, 'Juan', TRUE),
(2, 'Pedro', FALSE);

```

---

## 32. ENUM

`ENUM` permite definir una columna cuyo valor debe pertenecer a un conjunto predefinido de opciones explícitas.

### Ejemplo

```sql
CREATE TABLE empleados (
    id INT UNSIGNED,
    nombre VARCHAR(100),
    estado ENUM('ACTIVO', 'LICENCIA', 'BAJA')
);

INSERT INTO empleados VALUES (1, 'Juan', 'ACTIVO');

```

---

## 33. ¿Cuándo utilizar ENUM?

Resulta útil cuando:

* Existen muy pocas opciones.
* Las opciones están perfectamente definidas y **no cambian frecuentemente**.

> **Advertencia de diseño:** Si los valores pueden cambiar o expandirse con frecuencia, es mucho más flexible utilizar una tabla relacionada en lugar de `ENUM`, evitando alteraciones estructurales (DDL) en producción.

---

## 34. Tipos Binarios

MariaDB dispone de tipos destinados a almacenar datos binarios crudos (secuencias de bytes):
`BINARY`, `VARBINARY`, `TINYBLOB`, `BLOB`, `MEDIUMBLOB`, `LONGBLOB`.

> ⚠️ **Nota:** Los tipos binarios almacenan bytes sin interpretación de caracteres y **no deben confundirse** con los tipos de texto.

---

## 35. BLOB

Los tipos `BLOB` (Binary Large Object) permiten almacenar grandes cantidades de datos binarios (imágenes, archivos PDF, ejecutables).

```sql
foto BLOB

```

> **Buenas prácticas:** Almacenar archivos grandes directamente en una base de datos puede degradar el rendimiento y saturar las copias de seguridad. Una alternativa muy superior es almacenar el archivo en almacenamiento externo (disco, S3) y guardar en la tabla únicamente la ruta o metadatos:

```sql
CREATE TABLE documentos (
    id INT UNSIGNED PRIMARY KEY,
    nombre VARCHAR(200),
    ruta VARCHAR(500)
);

```

---

## 36. JSON

MariaDB soporta de forma nativa el almacenamiento y consulta de documentos `JSON`.

### Ejemplo

```sql
CREATE TABLE configuraciones (
    id INT UNSIGNED PRIMARY KEY,
    datos JSON
);

```

Podemos almacenar estructuras flexibles:

```json
{
    "idioma": "es",
    "notificaciones": true,
    "tema": "oscuro"
}

```

`JSON` es sumamente útil cuando necesitamos persistir información semiestructurada o configuraciones variables entre registros.

---

## 37. JSON no reemplaza al modelo relacional

> ❌ **Anti-patrón:** Pensar *"Como JSON permite guardar cualquier cosa, pondremos toda la información en una columna JSON"*.

Si tenemos datos estrictamente estructurados como empleados (`apellido`, `nombre`, `fecha_ingreso`, `sueldo`), **siempre** debemos utilizar columnas relacionales tipadas:

```sql
apellido VARCHAR(100),
nombre VARCHAR(100),
fecha_ingreso DATE,
sueldo DECIMAL(12,2)

```

El uso de `JSON` debe reservarse exclusivamente para esquemas verdaderamente dinámicos y variables.

---

## 38. El concepto clave: ¿Qué representa realmente el dato?

Consideremos el valor: `00123456`. A primera vista parece un número entero. Sin embargo, debemos hacernos la pregunta fundamental:

> **¿Voy a realizar operaciones matemáticas (sumas, promedios) con él?**

Si representa un código de barras, número de cliente o identificador:

* Si se define como `codigo INT`, **perderemos los ceros iniciales** (`00123456` se convertiría en `123456`).
* En este escenario, el tipo correcto es **`CHAR(8)`** o **`VARCHAR(8)`**, ya que preserva el formato textual exacto.

---

## 39. Ejemplos de datos que parecen números pero son texto

* DNI / CUIT / CUIL
* Códigos postales (`01001`)
* Números de teléfono
* Códigos de producto o expediente

> La regla de oro es: **¿Es una magnitud matemática o es un identificador compuesto por dígitos?**

---

## 40. El caso especial del Teléfono

Nunca se debe definir un número de teléfono como `telefono BIGINT`, ya que los teléfonos pueden contener:

* El signo más (`+`)
* Guiones (`-`) y espacios
* Prefijos internacionales y ceros iniciales (`+54 11 4567-8900`)

Por lo tanto, la elección natural y correcta es **`telefono VARCHAR(30)`**.

---

## 41. No almacenar todo como VARCHAR

Un error amateur frecuente es usar `VARCHAR` para absolutamente todo:

```sql
edad VARCHAR(3),
precio VARCHAR(20),
fecha VARCHAR(10),
cantidad VARCHAR(10)

```

Aunque MariaDB acepte almacenar valores así, **perdemos toda la integridad semántica, validación automática y eficiencia de índices**.

### La forma correcta:

```sql
edad TINYINT UNSIGNED,
precio DECIMAL(12,2),
fecha DATE,
cantidad INT UNSIGNED

```

---

## 42. Impacto en las Consultas y Ordenamiento

Si almacenamos números como texto (`edad VARCHAR(3)` con valores `'8'`, `'25'`, `'100'`), una ordenación alfabética devolverá un resultado incorrecto (`'100'`, `'25'`, `'8'`).

Si utilizamos `edad TINYINT UNSIGNED`, MariaDB ordenará y calculará correctamente como valores numéricos. Esto afecta directamente a cláusulas y funciones como:
`ORDER BY`, `WHERE`, `SUM()`, `AVG()`, `MIN()`, `MAX()`.

---

## 43. No almacenar fechas como texto

Evitar totalmente esquemas como `fecha_nacimiento VARCHAR(10)` con formatos `'15/04/1969'`.

Es preferible utilizar `DATE` (`'1969-04-15'`), lo cual habilita potentes funciones nativas:

```sql
-- Obtener el año de nacimiento
SELECT YEAR(fecha_nacimiento) FROM personas;

-- Rango de fechas optimizado
SELECT * FROM personas WHERE fecha_nacimiento >= '1970-01-01';

```

---

## 44. El concepto de NULL

Al diseñar una tabla debemos decidir si una columna admite valores nulos (`NULL`).

* `NULL` significa **ausencia de valor / valor desconocido**.
* ❌ No significa `0`.
* ❌ No significa cadena vacía (`''`).
* ❌ No significa `FALSE`.

---

## 45. NOT NULL y DEFAULT

* **`NOT NULL`:** Garantiza que la columna es obligatoria en cada inserción.
* **`DEFAULT`:** Establece un valor predeterminado automático si no se especifica explícitamente.

### Ejemplo

```sql
CREATE TABLE personas (
    id INT UNSIGNED PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    telefono VARCHAR(30),
    activo BOOLEAN DEFAULT TRUE,
    pais VARCHAR(50) DEFAULT 'Argentina'
);

```

---

## 46. AUTO_INCREMENT

Para claves primarias numéricas es el estándar de la industria:

```sql
id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY

```

Al insertar registros omitiendo la columna `id`, MariaDB genera de forma concurrente y segura el identificador correlativo.

---

## 47. Ejemplo Completo: Tabla de Empleados

Analicemos un diseño robusto y profesional:

```sql
CREATE TABLE empleados (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    cuil CHAR(11) NOT NULL,
    apellido VARCHAR(100) NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    fecha_nacimiento DATE,
    edad TINYINT UNSIGNED,
    sueldo DECIMAL(12,2),
    activo BOOLEAN DEFAULT TRUE,
    telefono VARCHAR(30),
    fecha_ingreso DATE,
    observaciones TEXT,
    creado TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

```

| Columna | Tipo | Justificación técnica |
| --- | --- | --- |
| `id` | `INT UNSIGNED` | Identificador autoincremental positivo |
| `cuil` | `CHAR(11)` | Identificador fiscal de longitud fija |
| `apellido` / `nombre` | `VARCHAR(100)` | Textos variables obligatorios |
| `fecha_nacimiento` | `DATE` | Fecha calendario exacta |
| `edad` | `TINYINT UNSIGNED` | Entero pequeño positivo |
| `sueldo` | `DECIMAL(12,2)` | Importe monetario con precisión exacta |
| `activo` | `BOOLEAN` | Indicador binario de estado |
| `telefono` | `VARCHAR(30)` | Cadena flexible de contacto |
| `fecha_ingreso` | `DATE` | Fecha de alta laboral |
| `observaciones` | `TEXT` | Texto descriptivo extenso |
| `creado` | `TIMESTAMP` | Marca temporal automática del sistema |

---

## 48. Datos Derivados: ¿Almacenar la edad?

En la tabla anterior tenemos `fecha_nacimiento` y `edad`. Sin embargo, **la edad es un dato derivado** que se calcula matemáticamente a partir de la fecha de nacimiento.

> **Problema potencial:** Si almacenamos ambas, al año siguiente la `edad` estática quedará desactualizada respecto a la `fecha_nacimiento`, generando inconsistencias.

**Buena práctica:** Almacenar únicamente `fecha_nacimiento DATE` y calcular la edad en tiempo de consulta o mediante columnas virtuales calculadas (`GENERATED ALWAYS AS`).

---

## 49. Metodología Paso a Paso para Elegir un Tipo de Dato

Antes de definir cualquier columna en tu base de datos, sigue este procedimiento analítico:

1. **Paso 1: ¿Qué representa conceptualmente?** (¿Cantidad, identificador, texto, fecha, estado, booleano?)
2. **Paso 2: ¿Cuál es el rango y límites?** (¿Puede ser negativo? ¿Cuál es el valor máximo estimado? ¿Cuántos caracteres?)
3. **Paso 3: ¿Qué operaciones realizaremos?** (¿Agregaciones `SUM/AVG`, ordenamientos `ORDER BY`, cálculos de fechas?)
4. **Paso 4: ¿Admite nulos?** (Si es obligatorio, aplicar estrictamente `NOT NULL`).
5. **Paso 5: Evaluar el crecimiento futuro** (Diseñar pensando en la escalabilidad a mediano y largo plazo).

---

## 50. Tabla de Decisión Rápida

| Necesidad de almacenamiento | Tipo de dato recomendado |
| --- | --- |
| Entero pequeño | `TINYINT` |
| Entero mediano | `SMALLINT` |
| Entero de propósito general | `INT` |
| Entero masivo / contadores grandes | `BIGINT` |
| Importes monetarios / contabilidad | `DECIMAL(M,D)` |
| Medición científica o física | `FLOAT` |
| Cálculos científicos de gran precisión | `DOUBLE` |
| Texto corto y variable | `VARCHAR` |
| Texto de longitud estrictamente fija | `CHAR` |
| Textos extensos / artículos | `TEXT` |
| Fecha calendario | `DATE` |
| Hora del día / intervalo | `TIME` |
| Fecha y hora exacta de un evento | `DATETIME` |
| Marca temporal automática del sistema | `TIMESTAMP` |
| Indicador Verdadero / Falso | `BOOLEAN` |
| Selección entre opciones fijas y acotadas | `ENUM` |
| Datos binarios puros | `BINARY / VARBINARY` |
| Archivos adjuntos grandes | `BLOB` |
| Estructuras semiestructuradas | `JSON` |

---

## 51. Errores Frecuentes en el Diseño

* **Error 1:** Utilizar `VARCHAR` para almacenar números o fechas.
* **Error 2:** Utilizar `FLOAT` para valores monetarios (provoca errores de redondeo).
* **Error 3:** Utilizar `BIGINT` por defecto para cualquier ID o contador pequeño.
* **Error 4:** Almacenar teléfonos como números enteros (`BIGINT`).
* **Error 5:** Usar `TEXT` en lugar de `VARCHAR` para nombres, emails o títulos.
* **Error 6:** Almacenar datos derivados redundantes (ej. edad junto a fecha de nacimiento).

---

## 52. Buenas Prácticas de Diseño

1. **Alineación semántica:** Utiliza siempre el tipo que refleje fielmente la naturaleza del dato.
2. **Optimización de recursos:** Evita tipos sobredimensionados (ej. `BIGINT` donde `INT` sobra).
3. **Integridad de dominio:** Aplica `UNSIGNED` en magnitudes que nunca son negativas.
4. **Obligatoriedad:** Define `NOT NULL` en todas las columnas esenciales.
5. **Predeterminados lógicos:** Utiliza `DEFAULT` para estados y valores iniciales estándar.
6. **Escalabilidad:** Anticipa el crecimiento volumétrico de la aplicación al elegir tamaños numéricos y de texto.

---

## 53. Actividades Prácticas y Autoevaluación

### Ejercicio 1 — Identificar el tipo óptimo

Determina el tipo de dato ideal para cada campo:

| Campo | Tipo propuesto |
| --- | --- |
| Nombre | `VARCHAR(100)` |
| Edad | `TINYINT UNSIGNED` |
| Sueldo | `DECIMAL(12,2)` |
| Fecha de nacimiento | `DATE` |
| Teléfono | `VARCHAR(30)` |
| Activo | `BOOLEAN` |
| Código postal | `VARCHAR(10)` |
| Observaciones | `TEXT` |
| Cantidad de productos | `INT UNSIGNED` |
| Hora de entrada | `TIME` |

---

### Ejercicio 2 — Diseño de tabla `productos`

Crea una tabla profesional para gestionar productos con las siguientes especificaciones:

* Identificador único autoincremental.
* Código de producto (obligatorio, longitud fija o variable moderada).
* Nombre y descripción extensa.
* Precio y stock (con valor por defecto en cero).
* Estado activo y fecha de alta.

#### Solución sugerida:

```sql
CREATE TABLE productos (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    codigo VARCHAR(30) NOT NULL,
    nombre VARCHAR(150) NOT NULL,
    descripcion TEXT,
    precio DECIMAL(12,2) NOT NULL,
    stock INT UNSIGNED DEFAULT 0,
    activo BOOLEAN DEFAULT TRUE,
    fecha_alta DATE
);

```

---

### Ejercicio 3 — Auditoría y Detección de Errores

Analiza la siguiente estructura deficiente:

```sql
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

**Preguntas de análisis:**

1. ¿Qué tipos deberían modificarse urgentemente? (`sueldo`, `edad`, `fecha_nacimiento`, `telefono`, `activo`).
2. ¿Qué columnas deberían ser `NOT NULL`? (`nombre`, etc.).
3. ¿Cuál debería ser la clave primaria? (`id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY`).
4. ¿Dónde aplicarías `UNSIGNED`? (En identificadores y contadores).
5. ¿Qué tipo usar para sueldo? (`DECIMAL(12,2)`).
6. ¿Qué tipo usar para la fecha? (`DATE`).
7. ¿Qué tipo usar para activo? (`BOOLEAN`).
8. ¿Qué problema hay con el teléfono? Al ser `BIGINT`, descarta signos `+`, guiones y ceros iniciales.
9. ¿Es necesario almacenar la edad? No, es un dato derivado de la fecha de nacimiento.

---

### Ejercicio 4 — ¿Número o Texto?

Indica la naturaleza conceptual de cada valor:

* `12345` ➔ Número
* `0012345` ➔ **Texto** (preserva ceros iniciales, ej. legajo o código).
* `+54 11 4567-8900` ➔ Texto
* `100` ➔ Número
* `000100` ➔ Texto
* `2026` ➔ Número o Año (`YEAR`)
* `00123456` ➔ Texto

---

### Ejercicio 5 — Creación e Inserción de Datos

```sql
CREATE TABLE personas (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    apellido VARCHAR(100) NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    fecha_nacimiento DATE,
    salario DECIMAL(12,2),
    activo BOOLEAN DEFAULT TRUE
);

INSERT INTO personas (apellido, nombre, fecha_nacimiento, salario, activo) VALUES
('García', 'Juan', '1985-03-12', 850000.50, TRUE),
('Pérez', 'María', '1990-08-25', 920000.00, TRUE),
('López', 'Carlos', '1978-11-03', 1100000.75, TRUE),
('Gómez', 'Ana', '1995-06-18', 780000.00, FALSE),
('Díaz', 'Pedro', '1969-04-15', 1250000.25, TRUE);

-- Consulta general
SELECT * FROM personas;

```

---

### Ejercicio 6 — Consultas Analíticas

```sql
-- Personas activas
SELECT * FROM personas WHERE activo = TRUE;

-- Filtrado por umbral salarial
SELECT * FROM personas WHERE salario > 900000;

-- Ordenamiento descendente por salario
SELECT * FROM personas ORDER BY salario DESC;

-- Promedio salarial
SELECT AVG(salario) AS salario_promedio FROM personas;

-- Extracción del año de nacimiento
SELECT 
    nombre, 
    apellido, 
    YEAR(fecha_nacimiento) AS anio_nacimiento 
FROM personas;

```

---

## 54. Actividad Final de Consolidación

Responde y reflexiona sobre los siguientes puntos clave:

1. **¿Por qué no utilizar `VARCHAR` para almacenar un precio?**
* Porque carece de precisión matemática exacta, lo que genera errores acumulativos de redondeo en cálculos financieros.


2. **¿Por qué un número de teléfono puede ser texto?**
* Porque incluye caracteres especiales (`+`, `-`, espacios) y ceros iniciales que un tipo numérico eliminaría o interpretaría incorrectamente.


3. **¿Por qué `DATE` es preferible a `VARCHAR` para una fecha?**
* Habilita validación automática de rangos calendáricos y permite utilizar funciones temporales nativas (`YEAR`, `DATEDIFF`, comparaciones de rangos).


4. **¿Por qué una edad puede ser `TINYINT UNSIGNED`?**
* Porque cubre holgura suficiente (de 0 a 255 años) ocupando únicamente 1 byte de memoria.


5. **¿Por qué un código `001234` debe ser `CHAR` o `VARCHAR` y no `INT`?**
* Para evitar la pérdida de los ceros iniciales significativos.


6. **¿Qué diferencia conceptual existe entre `0`, `FALSE`, `NULL` y `''`?**
* `0` es un valor numérico definido. `FALSE` es un valor booleano lógico. `NULL` representa ausencia total de valor o dato desconocido. `''` es una cadena de texto vacía.


7. **¿Por qué puede ser problemático almacenar simultáneamente la fecha de nacimiento y la edad?**
* Introduce redundancia y un alto riesgo de inconsistencia de datos (anomalías de actualización) al requerir mantenimiento manual constante.
