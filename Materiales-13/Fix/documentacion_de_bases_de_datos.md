# Documentación de Bases de Datos

## ¿Qué es la documentación en bases de datos?
Es el conjunto de archivos, descripciones y anotaciones que explican cómo está estructurada, utilizada y administrada una base de datos. Incluye desde diagramas conceptuales hasta manuales técnicos.

### Importancia:
* Facilita el mantenimiento y la evolución del sistema.
* Mejora la comunicación entre equipos.
* Reduce errores en el uso o modificación de la base de datos.
* Es fundamental en procesos de auditoría o certificaciones de calidad.

---

## Tipos de documentación

| Tipo de Documento | Contenido | ¿Quién lo usa? |
| :--- | :--- | :--- |
| **Modelo Conceptual** | Entidades, relaciones, reglas del negocio. | Analistas, diseñadores. |
| **Modelo Lógico** | Tablas, claves primarias/foráneas, tipos de datos. | Desarrolladores. |
| **Modelo Físico** | Scripts SQL, configuración del servidor. | Administradores de BD. |
| **Diccionario de Datos** | Descripción de cada campo, tipo, restricciones. | Todos los usuarios. |
| **Documentación Técnica** | Procedimientos, triggers, índices, vistas. | DBA y programadores. |
| **Manual de Usuario** | Cómo usar el sistema (consultas, reportes). | Usuarios finales. |
| **Políticas de Backup y Seguridad** | Frecuencia, responsables, procedimientos. | Administradores. |
| **Historial de Cambios (Change Log)** | Qué se cambió, cuándo y por qué. | Todos los roles. |

---

## Descripción de los distintos tipos de documentos

### 1. Modelo Conceptual
Representa la visión más abstracta del sistema. Se enfoca en entidades, atributos y relaciones sin considerar aspectos técnicos. Es útil para entender qué información se va a manejar y cómo se vinculan los distintos elementos del negocio. Se suele representar con diagramas E-R (Entidad-Relación).

### 2. Modelo Lógico
Traduce el modelo conceptual a un formato más técnico, definiendo tablas, columnas, tipos de datos, claves primarias y foráneas. Es independiente del gestor de base de datos (DBMS) y sirve de base para generar el modelo físico.

### 3. Modelo Físico
Es la implementación concreta en un motor de base de datos específico. Incluye la sintaxis SQL real, tipos de datos específicos, índices, procedimientos almacenados y configuraciones particulares (como motores de almacenamiento o particionamiento).

### 4. Diccionario de Datos
Es un documento detallado que describe cada campo de cada tabla: su nombre, tipo, longitud, restricciones y propósito. Sirve como referencia rápida y fundamental para desarrolladores, DBA y testers.

### 5. Documentación Técnica
Incluye la descripción funcional y técnica de componentes como procedimientos almacenados, funciones, triggers, vistas, índices, cursores, etc. También detalla la lógica implementada y cómo interactúa cada objeto con otros.

### 6. Manual de Usuario
Orienta al usuario final sobre cómo operar con la base de datos (por ejemplo, a través de un sistema), cómo generar reportes, cargar datos o realizar búsquedas. Debe ser claro, visual y accesible para personal no técnico.

### 7. Políticas de Backup y Seguridad
Detalla los procedimientos de respaldo y recuperación, frecuencia, responsables, almacenamiento, políticas de retención y métodos de cifrado o control de accesos. Es vital para la continuidad operativa y la seguridad de la información.

### 8. Historial de Cambios (Change Log)
Es un registro cronológico de las modificaciones realizadas en la base de datos: qué se cambió, cuándo, quién lo hizo y por qué. Facilita el mantenimiento, la trazabilidad y el control de versiones en entornos colaborativos.

---

## Ejemplos de Documentos para Base de Datos “Sistema de Ventas”

### 1. Modelo Conceptual (extracto en lenguaje natural)
* El sistema gestiona información de **Clientes**, **Productos**, **Ventas** y **Usuarios**.
* Cada cliente puede realizar múltiples ventas.
* Cada venta puede incluir uno o varios productos.
* Los usuarios del sistema deben tener roles (administrador o vendedor) que determinan su nivel de acceso.

### 2. Modelo Lógico (fragmento en formato tabla)

| Entidad | Atributo | Tipo de Dato | Clave |
| :--- | :--- | :--- | :--- |
| Clientes | id_cliente | INT | PK |
| Clientes | nombre | VARCHAR(100) | |
| Ventas | id_venta | INT | PK |
| Ventas | id_cliente | INT | FK $\rightarrow$ Clientes(id_cliente) |
| Ventas | fecha | DATE | |
| Detalle | id_venta | INT | FK $\rightarrow$ Ventas(id_venta) |
| Detalle | id_producto | INT | FK $\rightarrow$ Productos(id_producto) |
| Detalle | cantidad | INT | |

### 3. Modelo Físico (extracto SQL)
```sql
CREATE TABLE clientes (
    id_cliente INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    email VARCHAR(100),
    fecha_alta DATE DEFAULT CURRENT_DATE
);

CREATE TABLE ventas (
    id_venta INT AUTO_INCREMENT PRIMARY KEY,
    id_cliente INT NOT NULL,
    fecha DATE NOT NULL,
    FOREIGN KEY (id_cliente) REFERENCES clientes(id_cliente)
);
```

### 4. Diccionario de Datos (extracto)

| Tabla | Campo | Tipo | Restricciones | Descripción |
| :--- | :--- | :--- | :--- | :--- |
| clientes | id_cliente | INT | PK, AUTO_INCREMENT | Identificador único del cliente |
| clientes | nombre | VARCHAR(100) | NOT NULL | Nombre completo del cliente |
| ventas | fecha | DATE | NOT NULL | Fecha en la que se registró la venta |

### 5. Documentación Técnica (fragmento de stored procedure)
```sql
-- Procedimiento que devuelve el total de ventas de un cliente entre dos fechas
DELIMITER //
CREATE PROCEDURE total_ventas_cliente (
    IN p_id_cliente INT,
    IN p_fecha_desde DATE,
    IN p_fecha_hasta DATE,
    OUT p_total DECIMAL(10,2)
)
BEGIN
    SELECT SUM(total)
    INTO p_total
    FROM ventas
    WHERE id_cliente = p_id_cliente
      AND fecha BETWEEN p_fecha_desde AND p_fecha_hasta;
END //
DELIMITER ;
```

### 6. Manual de Usuario (extracto)
**Módulo de Ventas**
Para registrar una nueva venta:
1. Ingrese al menú **“Ventas” > “Nueva Venta”**.
2. Seleccione el cliente del listado.
3. Agregue los productos seleccionando nombre y cantidad.
4. Haga clic en **“Guardar”** para registrar la operación.

*El sistema calculará automáticamente el total.*

### 7. Política de Backup (resumen operativo)
* **Nombre de la BD:** `sistema_ventas`
* **Tipo de Backup:** Completo
* **Frecuencia:** Diario a las 03:00 AM
* **Herramienta:** `mysqldump` con compresión gzip
* **Ruta de destino:** `/var/backups/sistema_ventas/`
* **Retención:** 15 días
* **Verificación:** Restauración de prueba semanal los domingos
* **Responsable:** Administrador de Base de Datos

### 8. Historial de Cambios (Change Log - extracto)

| Fecha | Modificación | Autor | Motivo |
| :--- | :--- | :--- | :--- |
| 2025-06-01 | Agregado campo email a clientes | L. Llusá | Solicitud del área comercial |
| 2025-06-15 | Optimización del procedimiento `total_ventas_cliente` | M. González | Mejora de performance |
| 2025-07-01 | Cambio de tipo en `ventas.total` | L. Llusá | Precisión requerida por AFIP |

---

## Buenas prácticas para documentar
* Usar lenguaje claro y específico.
* Evitar jergas innecesarias o ambigüedad.
* Mantener los documentos actualizados.
* Incluir autores y fechas de modificación.
* Utilizar herramientas colaborativas (Google Docs, Notion, Confluence, Git).
* Si es posible, automatizar parte de la documentación (ej. generación de diagramas con herramientas como `dbdiagram.io`, MySQL Workbench, DBeaver).

## Herramientas recomendadas
* **MySQL Workbench:** para modelos E-R y documentación automática.
* **DBeaver:** exporta estructuras y diccionarios de datos. [Sitio oficial](https://dbeaver.io/)
* **SchemaSpy / Dataedo:** para generar documentación técnica. [Sitio oficial](https://schemaspy.org/)
* **Notion / Confluence:** para organizar y compartir información.
* **Git:** versionado de scripts y control de cambios.

---

## Uso de plugins en MySQL Workbench

Los plugins (o complementos) son pequeños programas que se integran dentro de MySQL Workbench para:
* Automatizar tareas repetitivas (como generación de documentación).
* Conectarse a herramientas externas (como generadores de diagramas o repositorios).
* Añadir nuevos menús, botones o acciones personalizadas.

Se escriben principalmente en Python y pueden interactuar con el modelo de base de datos, el editor de scripts o el entorno gráfico.

### Tipos de plugins que son posibles de crear o usar

| Tipo de Plugin | Ejemplo práctico |
| :--- | :--- |
| **Generadores de documentación** | Exportar diccionario de datos a HTML o Excel |
| **Analizadores de modelo** | Validar nombres de columnas o normalización |
| **Scripting personalizado** | Insertar registros de prueba automáticamente |
| **Integraciones externas** | Enviar script a Git o conectarse a un API externo |

### Dónde se encuentran los plugins en Workbench
1. Abrir MySQL Workbench.
2. En la barra de menú, ir a: `Scripting` > `Plugin Manager`.

Desde ahí se puede:
* Ver los plugins instalados.
* Habilitar/deshabilitar plugins.
* Ver los scripts existentes (en Python).
* Crear nuevos desde cero.

### Cómo crear un plugin básico
1. En `Scripting` > `Plugin Manager`, hacé clic en **New Plugin**.
2. Se abre un editor con una plantilla Python.
3. Modificá el código para hacer lo que necesitás. Por ejemplo:

```python
import grt
import mforms

# Plugin que muestra un mensaje de prueba
@ModuleInfo.plugin("com.inap.testplugin", caption="Mostrar Mensaje", input=[], pluginMenu="Tools")
@ModuleInfo.export(grt.INT, grt.classes.db_Catalog)
def mostrar_mensaje(catalog):
    mforms.Utilities.show_message("Plugin Activo", "Hola, Leonardo. Este es tu primer plugin!", "OK")
    return 0
```
4. Guardar y cerrar. El plugin aparecerá en `Tools` > `Mostrar Mensaje`.

### Dónde se guardan los plugins
* **En Linux:** `~/.mysql/workbench/modules/`
* **En Windows:** `C:\Users\TuUsuario\AppData\Roaming\MySQL\Workbench\modules\`

*Los plugins se guardan como archivos `.py`.*

### Plugins útiles disponibles en línea
Si bien Workbench no tiene una "store" de plugins oficial, se pueden encontrar muchos ejemplos en:
* GitHub
* Stack Overflow
* Sitios como `dbdiagram.io` para importar modelos

*También es posible crear propios plugins y compartirlos.*

### Requisitos
* Tener habilitado Python scripting en tu instalación de Workbench (viene por defecto).
* Conocer conceptos básicos de programación en Python.
* Tener precaución con scripts que modifiquen estructuras o datos.