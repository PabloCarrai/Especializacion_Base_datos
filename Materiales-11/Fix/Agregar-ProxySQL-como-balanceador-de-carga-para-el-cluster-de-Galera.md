# Agregar ProxySQL como balanceador de carga para el clúster de Galera

ProxySQL es un proxy de alto rendimiento diseñado específicamente para MySQL/MariaDB que permite distribuir las consultas de lectura y escritura (Read/Write Splitting), manejar conexiones de manera eficiente y actuar como un punto de entrada único y transparente para las aplicaciones.

Para integrarlo, se procederá a levantar una cuarta máquina virtual en VirtualBox (por ejemplo, llamada `proxysql-node`) con la misma estructura de red de dos adaptadores (NAT para internet y Host-Only para la red local).

---

## Configuración de red para ProxySQL

| Componente / Nodo | Dirección IP / Puerto |
| :--- | :--- |
| **ProxySQL (`proxysql-node`)** | `192.168.56.50` (IP en la red Host-Only) |
| **Nodo 1 (`node1`)** | `192.168.56.51` |
| **Nodo 2 (`node2`)** | `192.168.56.52` |
| **Nodo 3 (`node3`)** | `192.168.56.53` |
| **Puerto de entrada (Aplicaciones)** | `6033` |
| **Puerto de administración (ProxySQL)** | `6032` |

---

## Paso 1: Preparación de la Red en la nueva VM

En VirtualBox, se debe crear o configurar la nueva VM con dos adaptadores:

* **Adaptador 1:** NAT (para descargas/internet).
* **Adaptador 2:** Adaptador solo anfitrión (con la IP estática `192.168.56.50` configurada mediante Netplan, tal como se hizo en los nodos anteriores).

Actualizar los paquetes del sistema:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Paso 2: Instalación de ProxySQL en la nueva VM

ProxySQL no siempre está en los repositorios predeterminados de Ubuntu con su versión más reciente, por lo que conviene instalarlo desde el repositorio oficial de ProxySQL.

Se ejecutan los siguientes comandos en la VM de ProxySQL:

```bash
# 1. Descargar e instalar la llave GPG del repositorio oficial
sudo apt install wget apt-transport-https lsb-release ca-certificates -y
wget -O - 'https://repo.proxysql.com/ProxySQL/proxysql-2.8.x/repo_pub_key' | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/proxysql.gpg

# 2. Agregar el repositorio de ProxySQL (asegúrate de usar una versión reciente estable, ej: 2.8.x)
echo "deb https://repo.proxysql.com/ProxySQL/proxysql-2.8.x/$(lsb_release -sc)/ ./" | sudo tee /etc/apt/sources.list.d/proxysql.list

# 3. Actualizar e instalar ProxySQL
sudo apt update
sudo apt install proxysql -y
```

> **Explicación:** Se añade el repositorio oficial del desarrollador para obtener la versión moderna y estable de ProxySQL y se procede a su instalación en el sistema.

Una vez instalado, se inicia el servicio y se habilita:

```bash
sudo systemctl start proxysql
sudo systemctl enable proxysql
```

---

## Paso 3: Configuración inicial de ProxySQL

ProxySQL se gestiona de forma interna a través de una interfaz de línea de comandos SQL (muy similar a MariaDB, pero corriendo en el puerto administrativo `6032`).

Se debe conectar a la consola administrativa de ProxySQL (la contraseña y usuario por defecto son `admin` / `admin`):

```bash
mysql -u admin -padmin -h 127.0.0.1 -P 6032
```

Una vez dentro de la consola de administración (`ProxySQL Admin>`):

### 1. Registrar los nodos del clúster de Galera en el grupo de servidores (Server Groups)
Se definirá un grupo (por ejemplo, el ID `1`) donde se incluirán los tres nodos MariaDB.

```sql
INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES 
(1, '192.168.56.51', 3306),
(1, '192.168.56.52', 3306),
(1, '192.168.56.53', 3306);

LOAD MYSQL SERVERS TO RUNTIME;
SAVE MYSQL SERVERS TO DISK;
```

> **Explicación:** Se registran las tres direcciones IP de los nodos MariaDB en el grupo de servidores 1 de ProxySQL y se aplican los cambios en memoria (`RUNTIME`) y en disco (`DISK`).

### 2. Configurar las credenciales de la base de datos
ProxySQL necesita saber con qué usuario y contraseña se conectará a los nodos backend de MariaDB para monitorear su estado y rutear las consultas de los clientes.

*(Asegúrate de cambiar `'tu_usuario'` y `'tu_contrasena'` por un usuario válido que tengas creado en el clúster MariaDB con permisos globales o suficientes).*

```sql
INSERT INTO mysql_users(username, password, default_hostgroup) VALUES 
('tu_usuario', 'tu_contrasena', 1);

LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;
```

> **Explicación:** Se le indica a ProxySQL qué usuario rutear hacia el grupo de servidores 1.

### 3. Configurar las reglas de enrutamiento básico
Para empezar de forma segura con Galera, se configurará una regla general para que todas las consultas vayan al grupo de servidores principal:

```sql
INSERT INTO mysql_query_rules (rule_id, active, match_pattern, destination_hostgroup, lexer_router_type) VALUES 
(1, 1, "^SELECT.*FOR UPDATE$", 1, 'QUERIES'),
(2, 1, "^SELECT", 1, 'QUERIES'),
(3, 1, ".*", 1, 'QUERIES');

LOAD MYSQL QUERY RULES TO RUNTIME;
SAVE MYSQL QUERY RULES TO DISK;
```

> **Explicación:** Se asegura que tanto las consultas de lectura como las de escritura pasen ordenadamente hacia los nodos del clúster a través de ProxySQL.

Se sale de la consola administrativa con `exit;`.

---

## Paso 4: Comprobación del funcionamiento de ProxySQL

Para verificar que ProxySQL está detectando correctamente a los nodos de Galera como activos y saludables, se vuelve a entrar a la consola de administración:

```bash
mysql -u admin -padmin -h 127.0.0.1 -P 6032
```

Se ejecuta la siguiente consulta para ver el estado de los backends:

```sql
SELECT * FROM runtime_mysql_servers;
```

> **Resultado esperado:** Se visualizarán las tres IPs (`192.168.56.51`, `.52`, `.53`) con el estado `ONLINE` y el estatus de monitorización funcionando.

---

## Paso 5: Cómo se conectarán las aplicaciones a partir de ahora

A partir de este momento, las aplicaciones o scripts ya no se conectarán directamente a las IPs individuales de los nodos MariaDB (`51`, `52` o `53`), sino que apuntarán de forma centralizada a la IP de ProxySQL (`192.168.56.50`) utilizando el puerto `6033`:

```bash
mysql -u tu_usuario -ptu_contrasena -h 192.168.56.50 -P 6033
```

> **Beneficio directo:** Si el Nodo 1 llega a fallar o se apaga en VirtualBox, ProxySQL detectará el fallo automáticamente, dejará de enviarle tráfico y redirigirá las consultas de manera transparente hacia el Nodo 2 o Nodo 3, logrando alta disponibilidad real para los clientes.