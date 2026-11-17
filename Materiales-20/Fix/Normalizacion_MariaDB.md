# Normalización de Bases de Datos en MariaDB

## 1. Introducción Teórica

La normalización es un proceso formal de diseño de bases de datos relacionales cuyo propósito es organizar los datos de manera eficiente, evitando redundancias e inconsistencias.

La normalización de bases de datos surgió en 1970 de la mano de Edgar F. Codd, científico de IBM que revolucionó la informática al sentar las bases matemáticas del modelo relacional.

Antes de su propuesta, los sistemas de almacenamiento dependían de estructuras jerárquicas rígidas donde repetir información era habitual, lo que derivaba en archivos pesados y datos contradictorios.

A través de una serie de reglas progresivas denominadas formas normales (1FN, 2FN, 3FN, entre otras), Codd estableció un método formal para descomponer tablas complejas en estructuras más pequeñas, atómicas y vinculadas mediante claves.

La importancia de este proceso radica en que constituye la piedra angular de la integridad de los datos: al erradicar la redundancia innecesaria, se neutralizan las anomalías de inserción, actualización y borrado, garantizando que cada hecho del negocio se registre una única vez y que cualquier sistema transaccional pueda escalar con coherencia, seguridad y precisión a lo largo del tiempo.

En la práctica profesional, alcanzar la Tercera Forma Normal (3FN) representa el punto de equilibrio óptimo para la inmensa mayoría de los sistemas transaccionales (OLTP), ya que elimina las anomalías críticas de actualización sin fragmentar excesivamente el esquema.

Avanzar hacia formas normales superiores, como Boyce-Codd (BCNF) o 4FN, suele reservarse para escenarios con reglas de negocio muy específicas, pues subdividir aún más las tablas incrementa el costo computacional de reconstruir la información mediante múltiples uniones (`JOIN`).

Por esta misma razón, existen contextos reales donde se aplica el proceso inverso —la desnormalización controlada— para priorizar el rendimiento de lectura por sobre la pureza estructural: es el caso de los sistemas de análisis de datos (Data Warehouses), reportes masivos, paneles de control en tiempo real o catálogos de comercio electrónico con millones de consultas por segundo, donde almacenar ciertos datos calculados o redundantes evita cruces costosos y acelera drásticamente los tiempos de respuesta.

La desnormalización consiste en introducir redundancia de forma deliberada y controlada sobre un modelo que ya fue diseñado en 3FN. El objetivo nunca es desordenar la base de datos por comodidad, sino resolver un cuello de botella real de rendimiento o preservar una regla de negocio.

Tres casos clásicos en MariaDB donde desnormalizar es una buena decisión de arquitectura:

1. **Congelar datos históricos en comprobantes (Facturación / Pedidos)**
* **Por qué conviene:** Combina rendimiento con integridad histórica. El comprobante refleja exactamente las condiciones pactadas en el instante de la operación, y además se evita un `JOIN` contra productos cada vez que se imprime o consulta una factura histórica.


2. **Totales o contadores precalculados (Dashboard / Redes / Catálogos)**
* **El escenario normalizado (3FN):** En una plataforma con millones de registros (por ejemplo, una tabla `publicaciones` y otra `comentarios` o `votos`), mostrar un listado con la cantidad de respuestas de cada publicación obliga a ejecutar un `COUNT(*)` agrupado (`GROUP BY`) recorriendo miles o millones de filas en cada carga de pantalla.
* **Cómo se desnormaliza:** Se agrega una columna acumuladora (`total_comentarios` o `saldo_actual` en cuentas corrientes) directamente en la tabla maestra, manteniéndola sincronizada mediante una transacción de aplicación o un `TRIGGER` en MariaDB.
* **Por qué conviene:** Transforma una consulta costosa de agregación en una lectura directa de un único entero. Se paga un mínimo costo extra al escribir (`INSERT`), pero se gana una velocidad enorme en las lecturas masivas.


3. **Aplanar jerarquías profundas para reportes (Data Warehousing / Consultas frecuentes)**
* **El escenario normalizado (3FN):** Retomando el ejemplo universitario, para saber a qué Facultad pertenece cada Inscripción, el motor debe encadenar 3 uniones: `inscripciones -> materias -> carreras -> facultades`. En un sistema de gestión académica con millones de registros históricos de cursadas, generar reportes estadísticos por facultad penaliza la CPU y la memoria del servidor.
* **Cómo se desnormaliza:** Se incorpora directamente `facultad_id` en la tabla de hechos (`inscripciones` o una tabla de reportes `resumen_cursadas`), rompiendo la 3FN por dependencia transitiva (`cod_materia -> cod_carrera -> facultad_id`).
* **Por qué conviene:** Permite filtrar y agrupar directamente por `facultad_id` aprovechando un índice compuesto (`idx_facultad_anio`), eliminando múltiples `JOIN` intermedios en consultas analíticas.



**Regla de oro:** Primero siempre se normaliza hasta 3FN para comprender la estructura lógica real del negocio; recién después, con métricas de rendimiento en mano (consultas lentas detectadas con el *Slow Query Log* de MariaDB), se decide qué columna duplicar y cómo asegurar que no pierda sincronía.

### Objetivos principales:

1. Eliminar redundancia de datos.
2. Garantizar la integridad de los datos.
3. Facilitar las operaciones de actualización, inserción y eliminación sin errores.
4. Mejorar la coherencia lógica del modelo.

---

## 2. Problemas que resuelve la normalización

* **Redundancia:** el mismo dato almacenado en múltiples lugares.
* **Anomalías de actualización:** al modificar un valor en una parte y olvidarse en otra.
* **Anomalías de inserción:** dificultad para agregar un nuevo dato porque falta información obligatoria.
* **Anomalías de eliminación:** al borrar un dato, se pierde información que debería conservarse.

---

## 3. Conceptos Fundamentales

### Atributo atómico

* **Definición:** Un atributo es atómico si no puede dividirse en partes más pequeñas que tengan sentido dentro del contexto de la base de datos.
* **Ejemplo correcto:** `DNI = "32544321"` $\rightarrow$ es un único valor.
* **Ejemplo incorrecto:** `Teléfono = "011-4555-6677 / 011-4333-8899"` $\rightarrow$ tiene dos valores en una misma celda, lo cual viola la atomicidad. Lo correcto sería separar en otra tabla (`Teléfonos`) o al menos en columnas diferentes.
* **Regla:** En 1FN (Primera Forma Normal), todos los atributos deben ser atómicos.

### Dependencia funcional

* **Definición:** Un atributo $B$ depende funcionalmente de un atributo $A$ si cada valor de $A$ está asociado con un único valor de $B$.
* **Se expresa como:** $A \rightarrow B$.
* **Ejemplo:**

$$DNI \rightarrow \text{Nombre, Apellido, Fecha\_Nacimiento}$$



Si conozco el DNI, conozco de forma única el nombre, apellido y fecha de nacimiento de la persona.
* **No ejemplo:**

$$\text{Materia} \rightarrow \text{Profesor} \rightarrow \text{no siempre es cierto}$$



Una materia puede tener más de un profesor. En ese caso habría que modelar una relación aparte.
* **Importancia:** La dependencia funcional es la base matemática que sustenta la normalización (2FN y 3FN trabajan eliminando dependencias incorrectas).

### Clave primaria (Primary Key – PK)

* **Definición:** Es el atributo (o conjunto de atributos) que identifica de manera única a cada fila de una tabla.
* **Características:**
* No puede repetirse.
* No puede ser nulo.
* Generalmente es un número entero autoincremental (`id`) o un atributo natural como el `DNI`.


* **Ejemplo:**
```sql
CREATE TABLE alumnos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    dni VARCHAR(20) UNIQUE,
    nombre VARCHAR(100),
    apellido VARCHAR(100)
);

```


Aquí `id` es la PK.
* **PK compuesta:** en tablas intermedias, la clave primaria puede estar formada por más de un campo:
```sql
CREATE TABLE inscripciones (
    alumno_id INT,
    materia_id INT,
    PRIMARY KEY (alumno_id, materia_id)
);

```



### Clave foránea (Foreign Key – FK)

* **Definición:** Es un atributo en una tabla que hace referencia a la clave primaria de otra tabla, estableciendo una relación entre ambas.
* **Ejemplo:**
```sql
CREATE TABLE inscripciones (
    alumno_id INT,
    materia_id INT,
    FOREIGN KEY (alumno_id) REFERENCES alumnos(id),
    FOREIGN KEY (materia_id) REFERENCES materias(id)
);

```


* `alumno_id` es una FK que apunta a `alumnos.id`.
* `materia_id` es una FK que apunta a `materias.id`.


* **Importancia:** Garantizan la integridad referencial, es decir, que no pueda inscribirse un alumno que no exista, ni asignar una materia que no está en el catálogo.

---

## 4. Las Formas Normales

| Forma Normal | Requisito principal | Qué evita |
| --- | --- | --- |
| **1FN** | Valores atómicos (sin listas ni grupos repetidos). | Tablas con multivalores. |
| **2FN** | Cumple 1FN + todos los atributos dependen de la clave completa. | Dependencias parciales. |
| **3FN** | Cumple 2FN + no existen dependencias transitivas. | Datos que dependen de otros no-clave. |
| **BCNF** | Cumple 3FN + cada determinante es clave candidata. | Redundancias avanzadas. |

---

## 5. Ventajas y Desventajas de la Normalización

### Ventajas:

* Datos consistentes y sin duplicados.
* Modelo más flexible y lógico.
* Menos errores al insertar, actualizar o borrar.
* Mejor control de integridad referencial.

### Desventajas:

* Mayor cantidad de tablas $\rightarrow$ más uniones (`JOIN`s) en consultas.
* En sistemas de alta carga de consultas, puede ser preferible desnormalizar (ejemplo: *Data Warehousing*).

---

## 6. Estrategia práctica

1. Modelar primero la base de datos en bruto (tabla grande).
2. Aplicar normalización paso a paso (1FN, 2FN, 3FN).
3. Evaluar si conviene llegar a BCNF o detenerse en 3FN (generalmente suficiente).
4. Ajustar en base a performance vs integridad.

---

## 7. Caso Práctico: Universidad

Ejemplo de tabla inicial (NO normalizada):

```sql
CREATE TABLE inscripciones_cruda (
    id INT AUTO_INCREMENT PRIMARY KEY,
    alumno VARCHAR(100),
    dni VARCHAR(20),
    carrera VARCHAR(100),
    materia1 VARCHAR(100),
    materia2 VARCHAR(100),
    materia3 VARCHAR(100),
    profesor1 VARCHAR(100),
    profesor2 VARCHAR(100),
    profesor3 VARCHAR(100)
);

```

---

## 8. Normalización Paso a Paso

* **Primera Forma Normal (1FN):** Se eliminan grupos repetidos (`materia1`, `materia2`, `materia3`). Cada fila representa una inscripción alumno-materia-profesor.
* **Segunda Forma Normal (2FN):** Todos los atributos dependen de la clave primaria completa.
* **Tercera Forma Normal (3FN):** Eliminamos dependencias transitivas (ejemplo: Facultad separada de Materia).
* **Forma Normal de Boyce-Codd (BCNF):** Todo determinante es clave candidata, separamos materia-profesor.

---

## 9. Diagrama E-R Final (esquema)

```text
Alumnos ───< Inscripciones >─── Materias ───< Materia_Profesor >─── Profesores
                         │
                         └─── Carreras (con Facultad)

```

---

## Glosario

* **Anomalía de actualización:** Error o inconsistencia que surge al modificar un dato redundante en un registro y omitir actualizarlo en las demás copias almacenadas.
* **Anomalía de eliminación:** Pérdida no deseada de información secundaria válida cuando se borra un registro principal (por ejemplo, perder los datos de una carrera al eliminar al único alumno inscripto en ella).
* **Anomalía de inserción:** Imposibilidad de registrar un dato nuevo en una tabla porque aún no se dispone de otros datos obligatorios asociados en la misma estructura.
* **Atributo atómico:** Dato indivisible dentro del contexto de la base de datos; cada celda debe contener un único valor y no listas ni conjuntos.
* **Clave candidata:** Atributo o conjunto mínimo de atributos capaz de identificar de forma única cada fila de una tabla, postulándose para ser elegida como clave primaria.
* **Clave foránea (Foreign Key – FK):** Campo (o conjunto de campos) en una tabla que referencia a la clave primaria de otra tabla para establecer un vínculo relacional.
* **Clave primaria (Primary Key – PK):** Atributo o combinación de atributos (PK compuesta) que identifica de manera única e irrepetible cada fila de una tabla, sin admitir valores nulos (`NULL`).
* **Dependencia funcional ($A \rightarrow B$):** Relación matemática donde el valor de un atributo $A$ (determinante) define de manera única el valor de otro atributo $B$.
* **Dependencia parcial:** Situación en la que un atributo no clave depende solo de una parte de una clave primaria compuesta, y no de la clave completa (violación de la 2FN).
* **Dependencia transitiva:** Relación indirecta donde un atributo no clave depende de otro atributo no clave, en lugar de depender directamente de la clave primaria (violación de la 3FN).
* **Desnormalización:** Proceso controlado de introducir redundancia intencional en un modelo ya normalizado para reducir la cantidad de uniones (`JOIN`) y acelerar consultas de lectura masiva.
* **Determinante:** Atributo (o conjunto de atributos) situado a la izquierda de una dependencia funcional ($A \rightarrow B$) que determina el valor de otro.
* **Forma Normal de Boyce-Codd (BCNF):** Versión estricta de la 3FN que exige que todo determinante en una tabla sea obligatoriamente una clave candidata.
* **Integridad referencial:** Regla que garantiza la coherencia entre tablas relacionadas, impidiendo crear referencias hacia registros inexistentes o eliminar registros referenciados sin control.
* **Normalización:** Proceso formal de diseño de bases de datos relacionales basado en reglas matemáticas (Formas Normales) para eliminar redundancias y evitar anomalías de manipulación.
* **OLTP (Online Transaction Processing):** Sistemas orientados al procesamiento ágil y concurrente de transacciones cotidianas (inserciones, actualizaciones y borrados), donde prima la integridad de los datos.
* **Primera Forma Normal (1FN):** Regla que exige que todos los atributos de una tabla contengan valores atómicos y que no existan grupos repetitivos de columnas.
* **Redundancia:** Almacenamiento repetido e innecesario del mismo dato en múltiples filas o tablas.
* **Segunda Forma Normal (2FN):** Regla que exige cumplir con la 1FN y que todos los atributos no clave dependan funcionalmente de la totalidad de la clave primaria.
* **Tercera Forma Normal (3FN):** Regla que exige cumplir con la 2FN y eliminar toda dependencia transitiva entre atributos no clave.

---

## 10. Script para aplicar el ejemplo en MariaDB

```sql
CREATE DATABASE universidad_normalizada;
USE universidad_normalizada;

-- Alumnos
CREATE TABLE alumnos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100),
    dni VARCHAR(20) UNIQUE
);

-- Carreras
CREATE TABLE carreras (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100),
    facultad VARCHAR(100)
);

-- Materias
CREATE TABLE materias (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100),
    carrera_id INT,
    FOREIGN KEY (carrera_id) REFERENCES carreras(id)
);

-- Profesores
CREATE TABLE profesores (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100)
);

-- Relación materia-profesor (BCNF)
CREATE TABLE materia_profesor (
    materia_id INT,
    profesor_id INT,
    PRIMARY KEY (materia_id, profesor_id),
    FOREIGN KEY (materia_id) REFERENCES materias(id),
    FOREIGN KEY (profesor_id) REFERENCES profesores(id)
);

-- Inscripciones (alumno ↔ materia)
CREATE TABLE inscripciones (
    alumno_id INT,
    materia_id INT,
    PRIMARY KEY (alumno_id, materia_id),
    FOREIGN KEY (alumno_id) REFERENCES alumnos(id),
    FOREIGN KEY (materia_id) REFERENCES materias(id)
);

```

---

## 11. Conclusiones

* La normalización hasta 3FN es suficiente en la mayoría de sistemas OLTP.
* BCNF se aplica en casos específicos con reglas de negocio más estrictas.
* El diseño resultante reduce redundancia, evita inconsistencias y mejora la integridad referencial.
* En Data Warehousing a veces se desnormaliza para mejorar rendimiento en consultas masivas.

---

```sql
-- =======================================================================
-- PREPARACIÓN DEL ENTORNO
-- =======================================================================
DROP DATABASE IF EXISTS lab_normalizacion;
CREATE DATABASE lab_normalizacion DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
USE lab_normalizacion;

-- =======================================================================
-- ETAPA 1: PRIMERA FORMA NORMAL (1FN)
-- Regla: Valores atómicos y clave primaria definida (dni + cod_materia).
-- Problema: Persisten dependencias parciales (datos repetidos de alumno y materia).
-- =======================================================================
CREATE TABLE inscripciones_1fn (
    dni VARCHAR(15) NOT NULL,
    cod_materia VARCHAR(10) NOT NULL,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    materia VARCHAR(100) NOT NULL,
    carrera VARCHAR(100) NOT NULL,
    facultad VARCHAR(100) NOT NULL,
    PRIMARY KEY (dni, cod_materia)
) ENGINE=InnoDB;

INSERT INTO inscripciones_1fn (dni, cod_materia, nombre, apellido, materia, carrera, facultad) VALUES
('30111222', 'MAT-01', 'Ana', 'Gómez', 'Base de Datos', 'Tec. en Programación', 'Ingeniería'),
('30111222', 'MAT-02', 'Ana', 'Gómez', 'Redes', 'Tec. en Programación', 'Ingeniería'),
('32444555', 'MAT-01', 'Luis', 'Pérez', 'Base de Datos', 'Tec. en Programación', 'Ingeniería');

-- =======================================================================
-- ETAPA 2: SEGUNDA FORMA NORMAL (2FN)
-- Regla: Eliminar dependencias parciales separando alumnos y materias.
-- Problema: Persiste una dependencia transitiva en materias_2fn
--           (cod_materia -> carrera -> facultad).
-- =======================================================================
CREATE TABLE alumnos_2fn (
    dni VARCHAR(15) NOT NULL,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    PRIMARY KEY (dni)
) ENGINE=InnoDB;

CREATE TABLE materias_2fn (
    cod_materia VARCHAR(10) NOT NULL,
    materia VARCHAR(100) NOT NULL,
    carrera VARCHAR(100) NOT NULL,
    facultad VARCHAR(100) NOT NULL,
    PRIMARY KEY (cod_materia)
) ENGINE=InnoDB;

CREATE TABLE inscripciones_2fn (
    dni VARCHAR(15) NOT NULL,
    cod_materia VARCHAR(10) NOT NULL,
    PRIMARY KEY (dni, cod_materia),
    CONSTRAINT fk_2fn_alumno FOREIGN KEY (dni) REFERENCES alumnos_2fn(dni),
    CONSTRAINT fk_2fn_materia FOREIGN KEY (cod_materia) REFERENCES materias_2fn(cod_materia)
) ENGINE=InnoDB;

-- Migración de datos desde 1FN hacia 2FN
INSERT INTO alumnos_2fn (dni, nombre, apellido)
SELECT DISTINCT dni, nombre, apellido
FROM inscripciones_1fn;

INSERT INTO materias_2fn (cod_materia, materia, carrera, facultad)
SELECT DISTINCT cod_materia, materia, carrera, facultad
FROM inscripciones_1fn;

INSERT INTO inscripciones_2fn (dni, cod_materia)
SELECT dni, cod_materia
FROM inscripciones_1fn;

-- =======================================================================
-- ETAPA 3: TERCERA FORMA NORMAL (3FN)
-- Regla: Eliminar dependencias transitivas extrayendo carreras a su propia tabla.
-- Resultado: Modelo relacional limpio, sin redundancias y con integridad plena.
-- =======================================================================
CREATE TABLE alumnos (
    dni VARCHAR(15) NOT NULL,
    nombre VARCHAR(50) NOT NULL,
    apellido VARCHAR(50) NOT NULL,
    PRIMARY KEY (dni)
) ENGINE=InnoDB;

CREATE TABLE carreras (
    cod_carrera VARCHAR(10) NOT NULL,
    carrera VARCHAR(100) NOT NULL,
    facultad VARCHAR(100) NOT NULL,
    PRIMARY KEY (cod_carrera)
) ENGINE=InnoDB;

CREATE TABLE materias (
    cod_materia VARCHAR(10) NOT NULL,
    materia VARCHAR(100) NOT NULL,
    cod_carrera VARCHAR(10) NOT NULL,
    PRIMARY KEY (cod_materia),
    CONSTRAINT fk_materias_carrera FOREIGN KEY (cod_carrera) REFERENCES carreras(cod_carrera)
) ENGINE=InnoDB;

CREATE TABLE inscripciones (
    dni VARCHAR(15) NOT NULL,
    cod_materia VARCHAR(10) NOT NULL,
    PRIMARY KEY (dni, cod_materia),
    CONSTRAINT fk_inscripciones_alumno FOREIGN KEY (dni) REFERENCES alumnos(dni),
    CONSTRAINT fk_inscripciones_materia FOREIGN KEY (cod_materia) REFERENCES materias(cod_materia)
) ENGINE=InnoDB;

-- Carga de datos en el modelo final (3FN)
INSERT INTO alumnos (dni, nombre, apellido)
SELECT dni, nombre, apellido
FROM alumnos_2fn;

INSERT INTO carreras (cod_carrera, carrera, facultad) VALUES
('CAR-01', 'Tec. en Programación', 'Ingeniería');

INSERT INTO materias (cod_materia, materia, cod_carrera)
SELECT m.cod_materia, m.materia, c.cod_carrera
FROM materias_2fn m
INNER JOIN carreras c ON m.carrera = c.carrera AND m.facultad = c.facultad;

INSERT INTO inscripciones (dni, cod_materia)
SELECT dni, cod_materia
FROM inscripciones_2fn;

-- =======================================================================
-- CONSULTA DE VERIFICACIÓN FINAL (Reconstruyendo la vista original mediante JOIN)
-- =======================================================================
SELECT 
    a.dni,
    a.nombre,
    a.apellido,
    m.cod_materia,
    m.materia,
    c.carrera,
    c.facultad
FROM inscripciones i
INNER JOIN alumnos a ON i.dni = a.dni
INNER JOIN materias m ON i.cod_materia = m.cod_materia
INNER JOIN carreras c ON m.cod_carrera = c.cod_carrera
ORDER BY a.dni, m.cod_materia;

```
