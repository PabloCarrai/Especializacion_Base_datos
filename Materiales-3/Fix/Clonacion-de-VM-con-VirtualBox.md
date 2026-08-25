# Clonación de VM con VirtualBox
Al clonar una máquina virtual Linux (como Ubuntu) en VirtualBox, el clon hereda identificadores únicos del sistema original. Para evitar colisiones de red, problemas de resolución de nombres y conflictos de conectividad, se deben realizar los siguientes ajustes:
Regenerar la Dirección MAC (Nivel VirtualBox)
Si no seleccionaste la opción de regenerar MACs durante el asistente de clonación:
Con la VM apagada: Configuración\Red\Adaptador\Avanzadas.
Hacé clic en el ícono de las flechas circulares junto a Dirección MAC para generar una nueva dirección física virtual.
Cambiar el Nombre de Host (Hostname)
Ambas máquinas no deben compartir el mismo nombre en la red:
# 1. Establecer el nuevo hostname (ej. db02)
sudo hostnamectl set-hostname db02

# 2. Actualizar el archivo /etc/hosts para reflejar el cambio
sudo nano /etc/hosts

Reemplazá el nombre antiguo por el nuevo en la línea local:

127.0.1.1   srv-mariadb-02

Regenerar el Machine ID de Linux
El archivo machine-id es utilizado por servicios del sistema (como systemd-networkd, DHCP y logs) para identificar el equipo de forma única. Si dos VMs tienen el mismo ID, el servidor DHCP puede asignarles la misma dirección IP:
# 1. Vaciar los identificadores heredados
sudo truncate -s 0 /etc/machine-id
sudo rm /var/lib/dbus/machine-id


# 2. Generar uno nuevo
sudo systemd-machine-id-setup

# 3. Vincular con D-Bus
sudo ln -s /etc/machine-id /var/lib/dbus/machine-id

Regenerar Claves de SSH del Host
Si la máquina tiene habilitado el servicio OpenSSH Server, ambas VMs compartirán las mismas claves criptográficas de host, lo que generará alertas de seguridad (REMOTE HOST IDENTIFICATION HAS CHANGED):

# 1. Eliminar claves antiguas del servidor SSH
sudo rm -f /etc/ssh/ssh_host_*

# 2. Regenerar nuevas claves
sudo dpkg-reconfigure openssh-server

# 3. Reiniciar el servicio SSH

6. Reinicio Final y Verificación
Una vez aplicados los pasos anteriores, reiniciá la máquina para consolidar todos los servicios:

sudo reboot

Al volver a iniciar:
Verificá la IP única asignada: ip a
Verificá el nombre de host: hostname


I need to focus on formatting everything in Spanish while ensuring I use code fences and headings appropriately. There’s a line that says 127.0.1.1 srv... and I’m thinking the example should probably use db02 instead? I need to make sure that I’m maintaining the original content's integrity in the format. Since the user is asking for formatting, I shouldn't alter any of the content but could consider structuring the steps for clarity. -->
# Clonación de VM con VirtualBox

Al clonar una máquina virtual Linux, como Ubuntu, en VirtualBox, el clon hereda identificadores únicos del sistema original. Para evitar colisiones de red, problemas de resolución de nombres y conflictos de conectividad, se deben realizar los siguientes ajustes.

## 1. Regenerar la dirección MAC

**Nivel: VirtualBox**

Si no seleccionaste la opción de regenerar las direcciones MAC durante el asistente de clonación:

1. Con la VM apagada, accedé a **Configuración > Red > Adaptador > Avanzadas**.
2. Hacé clic en el ícono de las flechas circulares junto a **Dirección MAC** para generar una nueva dirección física virtual.

## 2. Cambiar el nombre de host

Ambas máquinas no deben compartir el mismo nombre en la red.

### Establecer el nuevo hostname

Por ejemplo, `db02`:

```bash
sudo hostnamectl set-hostname db02
```

### Actualizar el archivo `/etc/hosts`

Editá el archivo para reflejar el cambio:

```bash
sudo nano /etc/hosts
```

Reemplazá el nombre antiguo por el nuevo en la línea local:

```text
127.0.1.1   srv-mariadb-02
```

## 3. Regenerar el Machine ID de Linux

El archivo `machine-id` es utilizado por servicios del sistema, como `systemd-networkd`, DHCP y los registros, para identificar el equipo de forma única.

Si dos máquinas virtuales tienen el mismo ID, el servidor DHCP puede asignarles la misma dirección IP.

### Vaciar los identificadores heredados

```bash
sudo truncate -s 0 /etc/machine-id
sudo rm /var/lib/dbus/machine-id
```

### Generar un nuevo Machine ID

```bash
sudo systemd-machine-id-setup
```

### Vincular el Machine ID con D-Bus

```bash
sudo ln -s /etc/machine-id /var/lib/dbus/machine-id
```

## 4. Regenerar las claves SSH del host

Si la máquina tiene habilitado el servicio **OpenSSH Server**, ambas VMs compartirán las mismas claves criptográficas de host. Esto generará alertas de seguridad como:

```text
REMOTE HOST IDENTIFICATION HAS CHANGED
```

### Eliminar las claves antiguas del servidor SSH

```bash
sudo rm -f /etc/ssh/ssh_host_*
```

### Regenerar nuevas claves

```bash
sudo dpkg-reconfigure openssh-server
```

### Reiniciar el servicio SSH

Reiniciá el servicio SSH según la configuración de tu sistema.

## 5. Reinicio final y verificación

Una vez aplicados los pasos anteriores, reiniciá la máquina para consolidar todos los servicios:

```bash
sudo reboot
```

Al volver a iniciar, verificá:

- **La IP única asignada:**

  ```bash
  ip a
  ```

- **El nombre de host:**

  ```bash
  hostname
  ```
