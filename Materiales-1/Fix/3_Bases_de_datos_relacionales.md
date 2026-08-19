# Bases de datos relacionales

Las bases de datos relacionales son las más usadas actualmente para administrar datos de forma dinámica. Permite crear todo tipo de datos y relacionarlos entre sí.

Los datos son almacenados en registros que son organizados en tablas, de esta forma pueden asociarse los elementos entre sí muy fácilmente, además se pueden cruzar sin ninguna dificultad.

## Sus principales características son:

- Son de fácil gestión.
- Se pueden acceder a los datos con rapidez.
- Garantiza la total consistencia de los datos, sin posibilidad de error.
- No son muy eficaces con datos gráficos, CAD, sistema de información geográfica ni datos en multimedia.

---

## ¿Qué es una base de datos relacional?

Una base de datos relacional es una recopilación de elementos de datos con relaciones predefinidas entre ellos.

Estos elementos se organizan como un conjunto de tablas con columnas y filas. Las tablas se utilizan para guardar información sobre las entidades que se van a representar en la base de datos.

Cada columna de una tabla guarda un determinado tipo de datos y un campo almacena el valor real de un atributo. Las filas de la tabla representan una recopilación de valores relacionados de un objeto o una entidad.

Cada fila de una tabla podría marcarse con un identificador único denominado clave principal, mientras que filas de varias tablas pueden relacionarse con claves extranjeras.

Se puede obtener acceso a estos datos de muchas formas distintas sin reorganizar las propias tablas de la base de datos.

# Características distintivas de una Base de datos relacional

## SQL

SQL o lenguaje de consulta estructurada es la interfaz principal utilizada para comunicarse con bases de datos relacionales. SQL se convirtió en un estándar del Instituto Nacional Estadounidense de Estándares (ANSI) en 1986.

Todos los motores de bases de datos relacionales populares admiten el SQL de ANSI estándar.

SQL se utiliza para agregar, actualizar o eliminar filas de datos, por lo que se recuperan subconjuntos de datos para aplicaciones de análisis y procesamiento de transacciones, y para administrar todos los aspectos de la base de datos.

## Integridad de los datos

La integridad de los datos es la totalidad, precisión y coherencia general de los datos. Las bases de datos relacionales utilizan un conjunto de restricciones para aplicar la integridad de los datos en la base de datos. Esto incluye claves principales, claves externas, restricción “Not NULL”, restricción “Unique”, restricción “Default” y restricciones “Check”.

Estas restricciones de integridad ayudan a aplicar reglas en los datos de las tablas para garantizar la precisión y fiabilidad de los datos.

Además de las anteriores, la mayoría de las bases de datos relacionales también permiten la integración de código personalizado en desencadenadores que se ejecutan en función de una acción en la base de datos.

## Transacciones

Una transacción de base de datos es una o más instrucciones SQL que se ejecutan como una secuencia de operaciones que forman una sola unidad lógica de trabajo.

Las transacciones proporcionan una proposición “todo o nada”, por lo que la transacción se debe completar como una sola unidad y se debe escribir en la base de datos; de lo contrario, ninguno de los componentes individuales de la transacción debería pasar.

En la terminología de bases de datos relacionales, una transacción genera un COMMIT o un ROLLBACK.

Cada transacción se trata de forma coherente y fiable independiente de otras transacciones.

## Conformidad con ACID

(ACID es un se de propiedades que deben cumplir las transacciones)

Todas las transacciones de base de datos deben ser conformes a ACID (atómicas, coherentes, aisladas y duraderas) para garantizar la integridad de los datos.

La atomicidad requiere que la transacción se ejecute correctamente como un todo o, si una parte de la transacción falla, que toda ella quede invalidada.

La consistencia exige que los datos escritos en la base de datos como parte de la transacción cumplan todas las reglas definidas, así como las restricciones, incluidos los desencadenadores, las limitaciones y las cascadas.

El aislamiento es fundamental para lograr el control de concurrencia y asegurarse de que cada transacción sea independiente por sí misma.

La durabilidad requiere que todos los cambios realizados en la base de datos sean permanentes luego de que la transacción se haya completado de forma correcta.

# Motores de bases de datos relacionales

Los siguientes motores de BD son los mas utilizados en la actualidad:

- MySQL
- PostgreSQL
- MSSQL
- SQLite
- Oracle

## MySQL

Es el sistema gestor de bases de datos relacional por excelencia.

Es un SGBD multihilo y multiusuario utilizado en la gran parte de las páginas web actuales.

Además, es el más usado en aplicaciones creadas con software libre.

Se ofrece bajo la GNU GPL aunque también es posible adquirir una licencia para empresas que quieran incorporarlo en productos privativos (Desde la compra por parte de Oracle se está orientando a este ámbito empresarial).

### Las principales ventajas de este Sistema Gestor de Bases de datos son:

- Facilidad de uso y gran rendimiento.
- Facilidad para instalar y configurar.
- Soporte multiplataforma.
- Soporte SSL.

## MariaDB

Este SGBD es una derivación de MySQL que cuenta con la mayoría de características de este e incluye varias extensiones.

Nace a partir de la adquisición de MySQL por parte de Oracle para seguir la filosofía Open Source y tiene la ventaja de que es totalmente compatible con MySQL.

Entre las principales características de este Sistema Gestor de Bases de datos se encuentran:

- Gran escalabilidad.
- Seguridad y rapidez en transacciones.
- Extensiones y nuevas características relacionadas con su aplicación para Bases de datos NoSQL.

Presenta algunas pequeñas incompatibilidades en la migración de MariaDB y MySQL o pequeños atrasos en la liberación de versiones estables.

## SQLite

Más que un Sistema Gestor de bases de datos como tal, SQLite es una biblioteca escrita en C que implementa un SGBD y que permite transacciones sin necesidad de un servidor ni configuraciones.

Es una biblioteca utilizada en multitud de aplicaciones actuales ya que es open source y las consultas son muy eficientes.

Las principales características de SQLite son:

- El tamaño, al tratarse de una biblioteca, es mucho menor que cualquier SGBD.
- Reúne los cuatro criterios ACID (Atomicidad, Consistencia, Aislamiento y Durabilidad) logrando gran estabilidad.
- Gran portabilidad y rendimiento.

No soporta bases de datos que sean muy grandes.

## PostgreSQL

Este sistema gestor de base de datos relacional está orientado a objetos y es libre, publicado bajo la licencia BSD.

Sus principales características son:

- Control de Concurrencias multiversión (MVCC).
- Flexibilidad en cuanto a lenguajes de programación.
- Multiplataforma.
- Dispone de una herramienta (pgAdmin, https://www.pgadmin.org/) muy fácil e intuitiva para la administración de las bases de datos.
- Robustez, Eficiencia y Estabilidad.

## Microsoft SQL Server

Es un sistema gestor de bases de datos relacionales basado en el lenguaje Transact-SQL, capaz de poner a disposición de muchos usuarios grandes cantidades de datos de manera simultánea.

Es un sistema propietario de Microsoft.

Sus principales características son:

- Soporte exclusivo por parte de Microsoft.
- Escalabilidad, estabilidad y seguridad.
- Posibilidad de cancelar consultas.
- Potente entorno gráfico de administración que permite utilizar comandos DDL y DML.
- Aunque es nativo para Windows puede utilizarse desde hace ya un tiempo en otras plataformas como Linux o Docker.

## Oracle

Tradicionamente, Oracle ha sido el SGBD por excelencia para el mundo empresarial, considerado siempre como el más completo y robusto, destacando por:

- Soporte de transacciones.
- Estabilidad.
- Escalabilidad.
- Multiplataforma.

La principal desventaja, al igual que SQL Server, es el costo del software ya que, aunque cuenta con una versión gratuita (Express Edition o XE), sus principales opciones son de pago.
