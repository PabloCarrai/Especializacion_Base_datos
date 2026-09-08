# Guía práctica de uso de `mysqladmin` en MariaDB

> Herramienta de administración desde la línea de comandos para consultar, diagnosticar y gestionar servidores MariaDB y MySQL.

---

## Índice

1. [¿Qué es `mysqladmin`?](#1-qué-es-mysqladmin)
2. [Comprobar si está instalado](#2-comprobar-si-mysqladmin-está-instalado)
3. [Consultar la versión del servidor](#3-consultar-la-versión-del-servidor)
4. [Verificar si el servidor está funcionando](#4-verificar-si-el-servidor-está-funcionando)
5. [Consultar el estado resumido](#5-consultar-el-estado-resumido-del-servidor)
6. [Consultar variables del servidor](#6-consultar-las-variables-del-servidor)
7. [Consultar estadísticas detalladas](#7-consultar-estadísticas-detalladas)
8. [Consultar procesos activos](#8-consultar-procesos-activos)
9. [Finalizar una conexión](#9-finalizar-una-conexión)
10. [Crear una base de datos](#10-crear-una-base-de-datos)
11. [Eliminar una base de datos](#11-eliminar-una-base-de-datos)
12. [Detener el servidor](#12-detener-el-servidor)
13. [Operaciones de mantenimiento](#13-operaciones-de-mantenimiento)
14. [Conectarse a un servidor remoto](#14-conectarse-a-un-servidor-remoto)
15. [Seguridad y contraseñas](#15-seguridad-y-contraseñas)
16. [Automatización mediante scripts](#16-automatización-mediante-scripts)
17. [Resumen de comandos](#17-resumen-de-comandos)

---

## 1. ¿Qué es `mysqladmin`?

`mysqladmin` es una herramienta de administración de MariaDB y MySQL que se utiliza desde la línea de comandos.

Permite realizar tareas administrativas habituales sin necesidad de ingresar al cliente SQL `mariadb` o `mysql`.

Entre sus principales usos se encuentran:

- Comprobar si el servidor está funcionando.
- Consultar información básica del servidor.
- Visualizar procesos y conexiones.
- Consultar variables y estadísticas.
- Crear y eliminar bases de datos.
- Finalizar conexiones.
- Realizar determinadas operaciones de mantenimiento.
- Detener el servidor.
- Automatizar tareas mediante scripts.

Es especialmente útil para:

- Administración rápida.
- Diagnóstico.
- Monitorización.
- Automatización.
- Comprobaciones de disponibilidad.

> **Importante:** `mysqladmin` no reemplaza al cliente SQL. Para realizar consultas, modificar datos, crear tablas, usuarios o procedimientos, normalmente se utilizan `mariadb`, `mysql` y sentencias SQL.

---

## 2. Comprobar si `mysqladmin` está instalado

Para comprobar si la herramienta está disponible en el sistema, ejecuta:

```bash
mysqladmin --version
```

Ejemplo de salida:

```text
mysqladmin  Ver 9.1 Distrib 10.6.23-MariaDB, for Linux on x86_64
```

Este comando informa de la versión de la herramienta `mysqladmin` instalada en el sistema.

> **Importante:** esta versión corresponde principalmente al cliente. No necesariamente coincide con la versión del servidor MariaDB al que posteriormente nos conectaremos.

---

## 3. Consultar la versión del servidor

Para obtener información sobre el servidor MariaDB, utiliza:

```bash
mysqladmin -u root -p version
```

El parámetro `-p` hará que se solicite la contraseña.

Ejemplo de salida:

```text
Server version          10.6.23-MariaDB
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /run/mysqld/mysqld.sock
Uptime                  2 days 4 hours 15 min
Threads                 3
Questions               1520
```

### Diferencia entre cliente y servidor

| Comando | Información obtenida |
|---|---|
| `mysqladmin --version` | Versión de la herramienta cliente |
| `mysqladmin -u root -p version` | Versión e información del servidor |

---

## 4. Verificar si el servidor está funcionando

Para comprobar rápidamente si MariaDB responde, ejecuta:

```bash
mysqladmin -u root -p ping
```

Respuesta esperada:

```text
mysqld is alive
```

Este comando es útil para:

- Scripts.
- Sistemas de monitorización.
- Comprobaciones de disponibilidad.
- Diagnósticos rápidos.

### `mysqladmin ping` frente a `systemctl`

En Ubuntu también se puede consultar el estado del servicio mediante:

```bash
systemctl status mariadb
```

La diferencia principal es la siguiente:

| Comando | Qué comprueba |
|---|---|
| `systemctl status mariadb` | Estado del servicio en el sistema operativo |
| `mysqladmin ping` | Si el servidor MariaDB acepta y responde a conexiones |

Por ejemplo, el servicio puede aparecer como iniciado en el sistema operativo, pero el servidor podría no aceptar correctamente conexiones.

---

## 5. Consultar el estado resumido del servidor

El siguiente comando muestra un resumen de la actividad del servidor:

```bash
mysqladmin -u root -p status
```

Ejemplo de salida:

```text
Uptime: 2500  Threads: 3  Questions: 1500
Slow queries: 0  Opens: 50  Flush tables: 1
Open tables: 45  Queries per second avg: 0.60
```

### Principales indicadores

| Indicador | Descripción |
|---|---|
| `Uptime` | Tiempo transcurrido desde el último inicio del servidor |
| `Threads` | Cantidad de hilos o conexiones en determinados estados |
| `Questions` | Número de consultas recibidas |
| `Slow queries` | Consultas consideradas lentas |
| `Open tables` | Cantidad de tablas abiertas |
| `Queries per second` | Promedio de consultas ejecutadas por segundo |

Este comando permite obtener una visión general del servidor sin mostrar una cantidad excesiva de información.

---

## 6. Consultar las variables del servidor

Para mostrar las variables de configuración del servidor:

```bash
mysqladmin -u root -p variables
```

Entre las variables disponibles podemos encontrar parámetros relacionados con:

- Memoria.
- Conexiones.
- Buffers.
- Archivos.
- Registros o logs.
- Red.
- Almacenamiento.
- Tiempos de espera.

También es posible consultar esta información mediante SQL:

```sql
SHOW VARIABLES;
```

Para consultar una variable concreta:

```sql
SHOW VARIABLES LIKE 'max_connections';
```

Otra alternativa es utilizar `mysqladmin` y filtrar la salida desde la terminal:

```bash
mysqladmin -u root -p variables | grep max_connections
```

> `mysqladmin` y SQL ofrecen, en muchos casos, distintas formas de acceder a la misma información administrativa.

---

## 7. Consultar estadísticas detalladas

Para obtener estadísticas más completas del funcionamiento del servidor:

```bash
mysqladmin -u root -p extended-status
```

Pueden aparecer indicadores relacionados con:

- Conexiones.
- Consultas.
- Tablas.
- Bloqueos.
- Operaciones de InnoDB.
- Actividad general del servidor.

La misma información puede consultarse mediante SQL:

```sql
SHOW GLOBAL STATUS;
```

Para evitar una salida demasiado extensa, es posible aplicar un filtro:

```sql
SHOW GLOBAL STATUS LIKE 'Threads%';
```

También se puede filtrar desde la terminal:

```bash
mysqladmin -u root -p extended-status | grep Threads
```

---

## 8. Consultar procesos activos

Para visualizar las conexiones y consultas que se están ejecutando:

```bash
mysqladmin -u root -p processlist
```

Ejemplo de salida:

```text
+----+------+-----------+----+---------+------+----------------+------------------+
| Id | User | Host      | db | Command | Time | State          | Info             |
+----+------+-----------+----+---------+------+----------------+------------------+
| 10 | root | localhost | db | Query   | 5    | Sending data   | SELECT * FROM... |
+----+------+-----------+----+---------+------+----------------+------------------+
```

La lista de procesos puede ayudar a detectar:

- Consultas de larga duración.
- Conexiones abiertas.
- Posibles bloqueos.
- Usuarios con demasiadas conexiones.
- Consultas que consumen muchos recursos.

También se puede utilizar SQL:

```sql
SHOW PROCESSLIST;
```

Para obtener información más completa:

```sql
SHOW FULL PROCESSLIST;
```

> **Consideración sobre privilegios:** un usuario sin permisos suficientes podría no visualizar todos los procesos del servidor. La información disponible depende de los privilegios asignados.

---

## 9. Finalizar una conexión

Primero consulta la lista de procesos:

```bash
mysqladmin -u root -p processlist
```

Supongamos que se encuentra la siguiente conexión:

```text
Id = 123
```

Para finalizarla:

```bash
mysqladmin -u root -p kill 123
```

También es posible hacerlo desde SQL:

```sql
KILL 123;
```

Esta operación puede resultar útil cuando una conexión:

- Ejecuta una consulta problemática.
- Permanece abierta durante demasiado tiempo.
- Está bloqueando otros procesos.
- Consume recursos excesivos.

> **Precaución:** finalizar una conexión puede interrumpir una consulta o transacción en ejecución. Antes de hacerlo, comprueba el usuario, la base de datos y la consulta afectada.

---

## 10. Crear una base de datos

Para crear una base de datos vacía:

```bash
mysqladmin -u root -p create mibasededatos
```

Este comando equivale conceptualmente a:

```sql
CREATE DATABASE mibasededatos;
```

Después, se puede comprobar su existencia mediante SQL:

```sql
SHOW DATABASES;
```

También puede utilizarse:

```bash
mysqladmin -u root -p status
```

Sin embargo, `SHOW DATABASES;` es una comprobación más directa.

---

## 11. Eliminar una base de datos

Para eliminar una base de datos:

```bash
mysqladmin -u root -p drop mibasededatos
```

El comando normalmente solicita confirmación antes de realizar la operación.

Equivale conceptualmente a:

```sql
DROP DATABASE mibasededatos;
```

> **Advertencia:** esta operación es destructiva y elimina la base de datos junto con toda la información almacenada en ella.

Antes de ejecutarla en un entorno real, comprueba:

- Qué base de datos se está eliminando.
- Si existen copias de seguridad.
- Si hay aplicaciones conectadas.
- Si la operación fue autorizada.
- Si el nombre de la base de datos es correcto.

---

## 12. Detener el servidor

Para solicitar una detención ordenada del servidor:

```bash
mysqladmin -u root -p shutdown
```

Esta operación requiere privilegios adecuados, incluido normalmente el privilegio `SHUTDOWN`.

En Ubuntu también puede utilizarse:

```bash
sudo systemctl stop mariadb
```

### Diferencias entre ambos métodos

```text
mysqladmin shutdown
        ↓
Solicita al servidor MariaDB que se detenga

systemctl stop mariadb
        ↓
Gestiona el servicio MariaDB desde el sistema operativo
```

| Comando | Nivel de administración |
|---|---|
| `mysqladmin shutdown` | Servidor MariaDB |
| `systemctl stop mariadb` | Servicio del sistema operativo |

En servidores productivos, detener MariaDB puede afectar a aplicaciones, usuarios y procesos dependientes de la base de datos.

---

## 13. Operaciones de mantenimiento

`mysqladmin` también dispone de comandos relacionados con tareas administrativas y de mantenimiento.

### Vaciar o rotar logs

```bash
mysqladmin -u root -p flush-logs
```

### Reinicializar determinadas estadísticas

```bash
mysqladmin -u root -p flush-status
```

Estas operaciones pueden utilizarse para:

- Reinicializar determinados contadores.
- Forzar acciones relacionadas con logs.
- Realizar tareas de mantenimiento.

La disponibilidad y el comportamiento exacto de algunas operaciones pueden variar según la versión de MariaDB.

Para consultar todos los comandos disponibles:

```bash
mysqladmin --help
```

También se puede consultar la documentación local:

```bash
man mysqladmin
```

---

## 14. Conectarse a un servidor remoto

`mysqladmin` puede conectarse a un servidor MariaDB remoto utilizando la opción `-h`.

Ejemplo:

```bash
mysqladmin -h 192.168.1.100 -u administrador -p status
```

El puerto habitual de MariaDB es:

```text
3306/TCP
```

### Requisitos para una conexión remota

Para que la conexión funcione deben cumplirse varias condiciones:

1. El servidor MariaDB debe estar funcionando.
2. MariaDB debe aceptar conexiones de red.
3. El puerto correspondiente debe estar accesible.
4. El firewall debe permitir la conexión.
5. El usuario debe tener permisos para conectarse desde ese origen.
6. La autenticación debe ser válida.
7. Si corresponde, deben cumplirse las condiciones de cifrado TLS.

### Ejemplo de administración remota

Supongamos la siguiente configuración:

```text
Servidor A: 192.168.1.10
Servidor B: 192.168.1.100
Puerto MariaDB: 3306
```

Desde el **Servidor A** se puede consultar el estado del servidor MariaDB ubicado en el **Servidor B**:

```bash
mysqladmin -h 192.168.1.100 -u administrador -p status
```

Si MariaDB utiliza un puerto diferente:

```bash
mysqladmin -h 192.168.1.100 -P 3307 -u administrador -p status
```

---

## 15. Seguridad y contraseñas

No es recomendable escribir la contraseña directamente en la línea de comandos.

### Forma no recomendada

```bash
mysqladmin -u root -pMiClaveSecreta status
```

La contraseña podría quedar expuesta en:

- El historial del shell.
- La lista de procesos.
- Registros del sistema.
- Herramientas de monitorización.

### Forma recomendada

```bash
mysqladmin -u root -p status
```

Con esta sintaxis, `mysqladmin` solicita la contraseña de forma interactiva.

### Uso de `~/.my.cnf`

Para automatizaciones puede utilizarse un archivo de configuración:

```ini
[client]
user=root
password=MiClaveSecreta
host=localhost
```

El archivo debe protegerse con permisos restrictivos:

```bash
chmod 600 ~/.my.cnf
```

Después, será posible ejecutar comandos sin escribir las credenciales en cada ocasión:

```bash
mysqladmin ping
```

> El archivo de configuración debe estar correctamente protegido para evitar que otros usuarios del sistema puedan leerlo.

---

## 16. Automatización mediante scripts

Una de las ventajas de `mysqladmin` es que puede utilizarse fácilmente en scripts.

### Script básico de comprobación

```bash
#!/bin/bash

if mysqladmin -u root -p ping >/dev/null 2>&1
then
    echo "MariaDB está en funcionamiento"
else
    echo "ERROR: MariaDB no responde"
fi
```

Para guardar el script:

```bash
nano comprobar_mariadb.sh
```

Después, asígnale permisos de ejecución:

```bash
chmod +x comprobar_mariadb.sh
```

Y ejecútalo:

```bash
./comprobar_mariadb.sh
```

### Esquema de funcionamiento

```text
Cron
  ↓
Ejecuta el script
  ↓
mysqladmin ping
  ↓
¿MariaDB responde?
  ├── Sí → Continuar
  └── No → Generar una alerta
```

Este tipo de comprobación puede incorporarse posteriormente a:

- `cron`.
- Sistemas de monitorización.
- Scripts de administración.
- Mecanismos de alerta.
- Procesos de recuperación automática.

### Ejemplo compatible con automatización

Cuando se utiliza un archivo `~/.my.cnf`, el script puede ejecutarse sin solicitar una contraseña:

```bash
#!/bin/bash

if mysqladmin ping >/dev/null 2>&1
then
    echo "$(date): MariaDB está funcionando"
    exit 0
else
    echo "$(date): ERROR: MariaDB no responde"
    exit 1
fi
```

El código de salida permite que otros sistemas sepan si la comprobación fue correcta:

| Código | Significado |
|---:|---|
| `0` | MariaDB responde correctamente |
| `1` | MariaDB no responde |

---

## 17. Resumen de comandos

| Objetivo | Comando |
|---|---|
| Consultar la versión del cliente | `mysqladmin --version` |
| Consultar la versión del servidor | `mysqladmin -u root -p version` |
| Comprobar disponibilidad | `mysqladmin -u root -p ping` |
| Consultar el estado resumido | `mysqladmin -u root -p status` |
| Mostrar variables | `mysqladmin -u root -p variables` |
| Mostrar estadísticas | `mysqladmin -u root -p extended-status` |
| Ver procesos activos | `mysqladmin -u root -p processlist` |
| Finalizar una conexión | `mysqladmin -u root -p kill ID` |
| Crear una base de datos | `mysqladmin -u root -p create nombre_bd` |
| Eliminar una base de datos | `mysqladmin -u root -p drop nombre_bd` |
| Detener MariaDB | `mysqladmin -u root -p shutdown` |
| Vaciar o rotar logs | `mysqladmin -u root -p flush-logs` |
| Reinicializar estadísticas | `mysqladmin -u root -p flush-status` |
| Consultar ayuda | `mysqladmin --help` |
| Consultar el manual | `man mysqladmin` |

---

## Conclusión

`mysqladmin` es una herramienta sencilla y potente para administrar MariaDB desde la terminal. Permite comprobar la disponibilidad del servidor, consultar estadísticas, revisar procesos, crear o eliminar bases de datos, finalizar conexiones y automatizar tareas de monitorización.

Su principal ventaja es la rapidez: muchas operaciones administrativas pueden realizarse con un único comando, sin necesidad de abrir una sesión interactiva en el cliente SQL.

Para trabajar de forma segura:

- Utiliza siempre el parámetro `-p` sin escribir la contraseña en la línea de comandos.
- Protege adecuadamente el archivo `~/.my.cnf`.
- Revisa los privilegios del usuario.
- Ten especial cuidado con `kill`, `drop` y `shutdown`.
- Comprueba la ayuda de tu versión de MariaDB antes de utilizar operaciones de mantenimiento en producción.
