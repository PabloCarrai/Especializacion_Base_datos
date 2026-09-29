# Configurar un Galera Cluster con MariaDB en Ubuntu 24.04 a través de VirtualBox

## Configuración de red

Se asumen las siguientes direcciones IP estáticas para las máquinas virtuales en VirtualBox (es necesario adaptarse a la red local, por ejemplo usando adaptadores en modo **Red Bridged** o **Red Nat** configurada estáticamente, se puede dejar para al final para poder hacer todas las configuraciones):

* **Nodo 1 (node1):** `192.168.1.51`
* **Nodo 2 (node2):** `192.168.1.52`
* **Nodo 3 (node3):** `192.168.1.53`

---

## Paso 1: Preparación previa en VirtualBox (Todos los nodos)

Antes de tocar la consola, conviene asegurarse de configurar los adaptadores de red en VirtualBox para que las máquinas virtuales puedan verse entre sí y hacer pings mutuos.

Una vez iniciadas las máquinas virtuales con Ubuntu 24.04, se debe conectar a cada una de ellas y actualizar los repositorios del sistema:

```bash
sudo apt update && sudo apt upgrade -y
```

> **Explicación:** Garantiza que todos los paquetes base del sistema operativo estén al día y se eviten conflictos de dependencias.

---

## Paso 2: Instalación de MariaDB y Galera (Todos los nodos)

En los repositorios oficiales de Ubuntu 24.04, MariaDB viene con soporte completo para Galera Cluster integrado.

Se ejecuta el siguiente comando en cada uno de los tres nodos:

```bash
sudo apt install mariadb-server galera-4 -y
```

> **Explicación:** `mariadb-server` instala el motor de base de datos y `galera-4` instala la biblioteca del proveedor WSREP (*WriteSet Replication*) necesaria para la sincronización multi-maestro síncrona.

Una vez instalado, se procede a detener temporalmente el servicio para configurar su entorno:

```bash
sudo systemctl stop mariadb
```

---

## Paso 3: Configuración del Firewall (Todos los nodos)

Galera requiere varios puertos abiertos para comunicarse de manera interna. Si se tiene activo UFW (*Uncomplicated Firewall*), se ejecuta en todos los nodos:

```bash
sudo ufw allow 3306/tcp
sudo ufw allow 4567/tcp
sudo ufw allow 4568/tcp
sudo ufw allow 4444/tcp
```

> **Explicación de puertos:**
> * **3306:** Puerto estándar de MariaDB para consultas de clientes.
> * **4567:** Tráfico de replicación de Galera (puerto multicast/unicast principal).
> * **4568:** Sincronización de estados incrementales (IST).
> * **4444:** Transferencia de estado de snapshot (SST) para sincronizar nodos nuevos por completo.

---

## Paso 4: Configuración de MariaDB y Galera (Todos los nodos)

Se debe modificar el archivo de configuración principal de MariaDB para definir los parámetros del clúster. Se abre el archivo de configuración en cada nodo:

```bash
sudo nano /etc/mysql/mariadb.conf.d/60-galera.cnf
```

*(Si se prefiere editar el archivo general, usualmente se ubica en `/etc/mysql/mariadb.conf.d/50-server.cnf`, pero crear un archivo específico `60-galera.cnf` resulta más limpio).*

Se añade y adapta el siguiente contenido (reemplazando las IP con las de los propios nodos):

```ini
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2

# Configuración de Galera Provider
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Nombre del clúster (debe ser idéntico en todos los nodos)
wsrep_cluster_name="galera_cluster_ubuntu"

# Direcciones IP de los nodos que componen el clúster
wsrep_cluster_address="gcomm://192.168.1.51,192.168.1.52,192.168.1.53"

# Identidad propia del nodo actual (Cambiar la IP en cada servidor)
wsrep_node_address="192.168.1.51"
wsrep_node_name="node1"

# Método de transferencia de datos para unirse al clúster (por defecto rsync)
wsrep_sst_method=rsync
```

> **Nota importante:** En el **Nodo 2** se debe cambiar `wsrep_node_address="192.168.1.52"` y `wsrep_node_name="node2"`. En el **Nodo 3** se hace lo propio con su respectiva IP y nombre (`node3`).

> **Explicación de parámetros clave:**
> * `binlog_format=ROW`: Requerido por Galera para rastrear cambios a nivel de fila.
> * `innodb_autoinc_lock_mode=2`: Configuración de bloqueos interoperable para inserciones concurrentes seguras en tablas con autoincremento.
> * `wsrep_cluster_address`: Lista de nodos conocidos para que un servidor pueda unirse al clúster.

---

## Paso 5: Inicialización del Clúster (Solo en el Nodo 1)

Un clúster de Galera necesita un "nodo bootstrap" inicial para arrancar por primera vez, ya que los demás nodos buscarán un clúster activo al encenderse.

Se ejecuta **únicamente en el Nodo 1**:

```bash
sudo galera_new_cluster
```

> **Explicación:** Este comando arranca el demonio de MariaDB inicializando un clúster nuevo desde cero a partir de ese nodo.

Para verificar que el nodo 1 levantó correctamente el clúster, se entra a MariaDB:

```bash
sudo mysql -u root
```

Y se ejecuta la siguiente consulta SQL:

```sql
SHOW STATUS LIKE 'wsrep_cluster_size';
```

Debería observarse un valor de `1` (indicando que hay un nodo activo en el clúster). Se sale de la consola con `exit;`.

---

## Paso 6: Unir el resto de los nodos (Nodo 2 y Nodo 3)

Ahora que el Nodo 1 está activo, se procede a iniciar el servicio de MariaDB de forma convencional en el **Nodo 2** y en el **Nodo 3**:

```bash
sudo systemctl start mariadb
```

> **Explicación:** Al iniciar normalmente, estos nodos leen el archivo de configuración, detectan la directiva `wsrep_cluster_address` y se conectan al Nodo 1 para sincronizarse automáticamente mediante Rsync.

---

## Paso 7: Verificación final del Clúster

Se vuelve a entrar a la consola de MariaDB en cualquiera de los tres nodos:

```bash
sudo mysql -u root
```

Se ejecuta el comando de estado de Galera:

```sql
SHOW STATUS LIKE 'wsrep_%';
```

Conviene prestar especial atención a las siguientes variables devueltas en la tabla:

* **`wsrep_cluster_size`:** Debe marcar `3` (lo que confirma que los tres nodos virtuales se comunican y forman el clúster con éxito).
* **`wsrep_ready`:** Debe estar en `ON` (indica que el nodo está listo para aceptar consultas).
* **`wsrep_connected`:** Debe estar en `ON`.

### Prueba rápida de replicación multi-maestro:

1. En el **Nodo 1**, se crea una base de datos de prueba:
   ```sql
   CREATE DATABASE prueba_galera;
   ```
2. Se entra al **Nodo 3** y se comprueba que se replicó de forma instantánea ejecutando:
   ```sql
   SHOW DATABASES;
   ```
   *(Se visualizará `prueba_galera` listada automáticamente).*