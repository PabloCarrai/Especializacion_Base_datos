# Guía paso a paso: clonar una VM Ubuntu que forma parte de un clúster Galera en VirtualBox

Cuando clonás una máquina virtual que tiene Ubuntu, MariaDB y Galera, no alcanza con copiar la VM y cambiarle la dirección IP. La máquina clonada debe tener una identidad propia dentro del sistema operativo y del clúster, sin interferir con el nodo original.

Tomando como referencia el laboratorio con Ubuntu 24.04, VirtualBox y MariaDB/Galera, con esta configuración de dos nodos:

| VM | Nombre del nodo | IP de clúster |
| :--- | :--- | :--- |
| **Original** | `nodo1` | `192.168.56.51` |
| **Nodo existente** | `nodo2` | `192.168.56.52` |
| **Clon nuevo** | `nodo4` | `192.168.56.53` |

---

## 1. Qué cambia al clonar un nodo

* **Identidad de VirtualBox**
  La VM clonada debe tener un UUID propio y direcciones MAC distintas de las del original.
* **Identidad de Ubuntu**
  Hay que revisar el hostname, la dirección IP, la configuración de red y, si corresponde, el identificador de máquina `machine-id`.
* **Identidad de Galera**
  Cada nodo necesita su propio `wsrep_node_name` y `wsrep_node_address`. El nombre del clúster y la lista de miembros se mantienen según la topología.
* **Datos y sincronización**
  El clon no debe arrancar simultáneamente con la copia de los datos originales de MariaDB sin haber definido cómo se incorporará al clúster.

> **Importante:** Realizar la clonación con el nodo original detenido, si es posible, y no iniciar MariaDB en el clon hasta haber revisado su configuración. Si el clúster está en producción, la estrategia debe ser distinta y requiere especial cuidado con la consistencia de los datos.

---

## 2. Preparación antes de clonar

### Paso 1. Verificar el estado del nodo original
Arrancá la VM original y ejecutá:
```bash
hostnamectl
ip -br addr
ip route
```
Esto permite comprobar el nombre del equipo, las interfaces de red y las direcciones IP efectivamente asignadas.

Después verificá MariaDB:
```bash
sudo systemctl status mariadb
```
Y, si el servidor responde, consultá el estado de Galera:
```bash
sudo mariadb -e "
SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size';
SHOW GLOBAL STATUS LIKE 'wsrep_cluster_status';
SHOW GLOBAL STATUS LIKE 'wsrep_local_state_comment';
SHOW GLOBAL VARIABLES LIKE 'wsrep_node_name';
SHOW GLOBAL VARIABLES LIKE 'wsrep_node_address';
"
```

**Interpretación:**
* `wsrep_cluster_size`: cantidad de nodos integrantes del clúster.
* `wsrep_cluster_status`: debería indicar `Primary`.
* `wsrep_local_state_comment`: en un nodo sincronizado, debería indicar `Synced`.
* `wsrep_node_name` y `wsrep_node_address`: identifican al nodo.

> Si el clúster está fallando o el nodo original no está sincronizado, primero conviene diagnosticarlo. Clonar no corrige esos problemas.

### Paso 2. Realizar una copia de seguridad
Antes de modificar nada, guardá una copia de la configuración de MariaDB y Galera:
```bash
sudo cp -a /etc/mysql \
  /etc/mysql.backup-antes-clonacion
```
Guardá también la configuración de red:
```bash
sudo cp -a /etc/netplan \
  /etc/netplan.backup-antes-clonacion
```
Si existen archivos de configuración adicionales, por ejemplo en `/etc/mysql/mariadb.conf.d/`, la copia de `/etc/mysql` ya los contempla.

Si los datos son importantes, realizá además una copia de seguridad de las bases de datos mediante un procedimiento compatible con tu versión de MariaDB y con el estado del clúster. La copia de los archivos de configuración no reemplaza un backup de las bases de datos.

### Paso 3. Apagar correctamente la VM original
Si podés detener el nodo, ejecutá:
```bash
sudo systemctl stop mariadb
sudo poweroff
```
Esperá a que VirtualBox muestre que la VM está apagada.

> No uses el estado guardado como sustituto de un apagado normal. Apagar MariaDB de forma ordenada permite que los archivos del servidor queden en un estado consistente antes de copiar el disco.
> Si necesitás mantener el clúster en servicio, no detengas un nodo sin evaluar antes el impacto sobre la disponibilidad.

---

## 3. Clonar la VM desde VirtualBox

### Paso 4. Crear un clon completo
En VirtualBox:
1. Seleccioná la VM original.
2. Abrí el menú contextual con el botón derecho.
3. Seleccioná **Clonar**.
4. Asigná un nombre diferente, por ejemplo `nodo3`.
5. Elegí la opción de **clonación completa**, no enlazada.
6. Si aparece la opción de generar nuevas direcciones MAC para todos los adaptadores de red, seleccioná esa opción.
7. Elegí clonar el estado actual de la máquina, sin incluir snapshots innecesarios.
8. Iniciá la clonación y esperá a que finalice.

* **¿Por qué elegir un clon completo?** Porque crea una copia independiente de los discos virtuales. No depende de que el disco de la VM original siga disponible para funcionar.
* **¿Por qué generar nuevas MAC?** Porque las interfaces clonadas deben tener identificadores de hardware virtual distintos, especialmente cuando ambas máquinas estarán conectadas a la misma red.

> No arranques todavía el clon. Primero verificá la configuración de sus adaptadores.

### Paso 5. Revisar la red virtual del clon
Con el clon apagado, entrá en **Configuración / Red** y comprobá cada adaptador.

En nuestro caso, el esquema habitual es:

| Adaptador | Función | Configuración de ejemplo |
| :--- | :--- | :--- |
| Adaptador 1 | Acceso a Internet | NAT |
| Adaptador 2 | Comunicación entre nodos | Adaptador solo-anfitrión (Host-only) |

Si el original utiliza `enp0s3` para NAT y `enp0s8` para la red del clúster, verificá que el clon conserve un esquema equivalente.

> La interfaz host-only debe estar conectada a la misma red que los demás nodos. 
> No cambies el modo de red de los nodos existentes para adaptarlos al clon. El nuevo nodo debe adaptarse a la topología que ya funciona.

---

## 4. Arrancar el clon y modificar Ubuntu

### Paso 6. Iniciar el clon con MariaDB detenido
Iniciá `nodo3` desde VirtualBox y abrí una terminal.

Si MariaDB se inicia automáticamente, detenelo antes de cambiar su identidad:
```bash
sudo systemctl stop mariadb
```
Verificá que haya quedado detenido:
```bash
sudo systemctl status mariadb
```

> No ejecutes `galera_new_cluster` ni inicies el clúster desde el clon. Esas operaciones pueden crear una partición o un clúster independiente si se usan en el contexto equivocado.

### Paso 7. Cambiar el hostname
Establecé el nuevo nombre:
```bash
sudo hostnamectl set-hostname nodo4
```
Comprobá el resultado:
```bash
hostnamectl
hostname
```
El nombre del sistema operativo debe coincidir con el que vas a utilizar para identificar el nodo en tu configuración de Galera.

Si el archivo `/etc/hosts` contiene una entrada con el hostname anterior, revisala:
```bash
cat /etc/hosts
```
Si encontrás una línea que asocia el nombre anterior con una IP fija, corregila para que refleje el nuevo nodo. Por ejemplo:
```text
127.0.0.1 localhost
192.168.56.54 nodo4
```
> No reemplaces indiscriminadamente todo el archivo: conservá las entradas necesarias para `localhost` y cualquier otra configuración válida.

### Paso 8. Cambiar la dirección IP del nodo
Este es uno de los pasos más importantes. El clon no puede usar la misma IP que el nodo original.

Primero identificá las interfaces y la configuración actual:
```bash
ip -br addr
ip route
ls -l /etc/netplan/
```

En Ubuntu 24.04, la configuración suele estar administrada mediante Netplan. Abrí el archivo que corresponda:
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```
*(El nombre del archivo es un ejemplo: utilizá el que realmente exista en tu VM).*

Si la interfaz del clúster es `enp0s8` y tu red es `192.168.56.0/24`, una configuración ilustrativa sería:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.56.53/24
```
Esta configuración supone que la interfaz NAT obtiene su dirección por DHCP y que la interfaz del clúster usa una IP fija. Si tu archivo original contiene otros parámetros, como rutas, DNS o configuración de NetworkManager, conservá los que correspondan.

Aplicá la configuración:
```bash
sudo netplan generate
sudo netplan try
```
Si todo funciona, confirmá el cambio cuando Netplan lo solicite. Después verificá:
```bash
ip -br addr
ip route
```
La interfaz de clúster del nuevo nodo debería mostrar `192.168.56.53/24`.

### Paso 9. Regenerar el identificador de máquina si corresponde
El clon puede conservar el `machine-id` de Ubuntu. Para evitar identidades duplicadas en herramientas que dependen de este identificador, comprobá:
```bash
cat /etc/machine-id
```
Si el clon va a convivir con el original, regeneralo según el procedimiento de Ubuntu:
```bash
sudo rm -f /etc/machine-id
sudo systemd-machine-id-setup
```
Verificá el nuevo valor:
```bash
cat /etc/machine-id
```
El identificador debe ser distinto del del original.

> Si utilizás claves SSH de host, certificados o agentes de configuración que identifican a cada máquina, revisá también esas identidades. No regeneres claves o credenciales sin comprobar cómo se administran en tu entorno.

---

## 5. Adaptar la configuración de MariaDB y Galera

### Paso 10. Identificar el archivo de configuración del clúster
Buscá los parámetros de Galera:
En la instalación, es probable que estén en un archivo dentro de `/etc/mysql/mariadb.conf.d/`.

Abrilo para editarlo, por ejemplo:
```bash
sudo nano /etc/mysql/mariadb.conf.d/60-galera.cnf
```

### Paso 11. Modificar los parámetros propios del nodo
La configuración del nuevo nodo debe identificarlo de forma única. Para tu laboratorio, la parte correspondiente a Galera podría quedar así:

```ini
[mysqld]

binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2

wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

wsrep_cluster_name="galera_cluster_ubuntu"

wsrep_cluster_address="gcomm://192.168.56.51,192.168.56.52,192.168.56.53"

wsrep_node_name="nodo3"
wsrep_node_address="192.168.56.53"

wsrep_sst_method=rsync
```
Conservá también los restantes parámetros que ya tengas configurados, como `bind-address`, opciones de InnoDB y cualquier configuración de autenticación o seguridad.

### Paso 12. Revisar la configuración de red de MariaDB
Comprobá si MariaDB escucha en la interfaz adecuada.

### Paso 13. Comprobar los puertos entre nodos
En tu configuración, verificá la conectividad entre `nodo3` y los demás nodos para estos puertos:

| Puerto | Uso |
| :--- | :--- |
| `TCP 3306` | Conexiones MariaDB, incluida la administración SQL. |
| `TCP/UDP 4567` | Comunicación de Galera entre nodos. |
| `TCP 4568` | Transferencia incremental de estados, IST. |
| `TCP 4444` | Transferencia de estado completa, SST. |

Desde `nodo3`, podés realizar una primera prueba TCP:
```bash
for ip in 192.168.56.51 192.168.56.52 192.168.56.53; do
    echo "Probando $ip"
    for p in 3306 4567 4568 4444; do
        nc -zv -w3 "$ip" "$p"
    done
done
```

> Una conexión rechazada no significa necesariamente que la red esté rota: puede indicar que el servicio no está escuchando. Un timeout suele justificar revisar el firewall, las rutas y la interfaz de destino.

---

## 6. Resolver el punto crítico: los datos de MariaDB clonados

### Paso 14. Definir si vas a agregar o reemplazar un nodo
Este paso determina cómo tratar `/var/lib/mysql`.

* **Caso A — Agregar un nodo nuevo**
  El objetivo es que `nodo3` se incorpore al clúster existente y reciba los datos mediante SST o IST, según corresponda.
  Lo recomendable es evitar que el clon conserve una copia independiente y potencialmente desactualizada de la base original. Una estrategia habitual es preparar el nuevo nodo para que reciba una transferencia de estado del clúster, verificando antes los requisitos del método SST configurado.
* **Caso B — Reemplazar un nodo existente**
  El tratamiento depende de qué nodo se reemplaza, de si conserva su identidad lógica y de cuál es el estado de los datos. No borres su directorio de datos ni reutilices su identidad sin planificar el reemplazo y comprobar que el clúster esté sano.

Para el **Caso A**, antes de modificar los datos:
1. Mantené MariaDB detenido en el clon.
2. Verificá que los otros nodos formen un clúster `Primary` y estén sincronizados.
3. Comprobá que el método `rsync` esté configurado correctamente y que los puertos de SST estén disponibles.
4. Respaldá cualquier dato que necesites conservar exclusivamente en el clon.
5. Prepará el directorio de datos según el procedimiento compatible con tu versión de Galera y MariaDB.

> No te recomiendo ejecutar directamente `rm -rf /var/lib/mysql/*` como parte de una receta genérica. Es una operación destructiva y no hace falta para clonar una VM en sí misma. Si el objetivo es unirse mediante SST, conviene preparar un directorio de destino vacío de forma controlada, conservando una copia recuperable y respetando los permisos del usuario `mysql`.
> Si no estás seguro de si corresponde limpiar el directorio de datos, detenete en este paso: se puede comprobar primero el estado de los nodos y decidir la estrategia sin arriesgar el clúster.

---

## 7. Incorporar el clon al clúster

### Paso 15. Revisar la configuración antes del arranque
Desde el clon, comprobá:
```bash
hostname
ip -br addr
sudo mariadbd --print-defaults
```
Revisá que no queden parámetros propios del nodo original.

Podés validar además que no haya errores de sintaxis evidentes en los archivos de configuración, aunque la validación definitiva se realiza observando el arranque y los registros.

### Paso 16. Arrancar MariaDB en el clon
Solo después de completar las verificaciones anteriores, iniciá el servicio:
```bash
sudo systemctl start mariadb
```
Consultá su estado:
```bash
sudo systemctl status mariadb --no-pager
```
Si falla:
```bash
sudo journalctl -u mariadb -n 100 --no-pager
```

> No intentes resolver un fallo ejecutando `galera_new_cluster` en el clon. Ese comando se utiliza para inicializar un clúster y no para incorporar normalmente un nodo a un clúster existente.

Con una configuración correcta, el nuevo nodo debería contactar con el clúster y, cuando sea necesario, recibir una transferencia de estado. Si no se incorpora, revisá el log para distinguir errores de configuración, conectividad, autenticación SST o incompatibilidad de datos.

### Paso 17. Verificar que el nuevo nodo se haya sincronizado
En `nodo3`:
```bash
sudo mariadb -e "
SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size';
SHOW GLOBAL STATUS LIKE 'wsrep_cluster_status';
SHOW GLOBAL STATUS LIKE 'wsrep_local_state_comment';
SHOW GLOBAL VARIABLES LIKE 'wsrep_node_name';
SHOW GLOBAL VARIABLES LIKE 'wsrep_node_address';
"
```
Si efectivamente agregaste un cuarto nodo, los resultados esperados son:
* `wsrep_cluster_size`: `3` (o el número correspondiente al total de nodos).
* `wsrep_cluster_status`: `Primary`.
* `wsrep_local_state_comment`: `Synced`.
* `wsrep_node_name`: `nodo4` (o el nombre configurado).
* `wsrep_node_address`: `192.168.56.53`.

> Repetí la consulta de tamaño de clúster en los nodos existentes. No des por concluida la operación solo porque MariaDB indique que está activo: tiene que estar integrado y sincronizado con los demás miembros.

### Paso 18. Hacer una prueba funcional
Una vez que todos los nodos estén sincronizados, realizá una prueba con una tabla de laboratorio y un registro identificable, siempre en una base de datos de pruebas.
1. Creá una tabla de prueba desde un nodo.
2. Insertá un registro.
3. Consultá el registro desde otro nodo.
4. Verificá que los cambios se hayan replicado.
5. Eliminá los objetos de prueba cuando termines.

Esto permite comprobar la replicación real y non solamente el estado de conexión.