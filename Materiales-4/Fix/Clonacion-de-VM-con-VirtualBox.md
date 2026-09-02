# Clonación de una máquina virtual Linux

## Preparación de la máquina clonada

Al clonar una máquina virtual Linux en VirtualBox, la copia conserva algunos identificadores de la máquina original.

Para evitar conflictos de red, nombres de host y conexiones SSH, modificá los siguientes valores.

---

## 1. Dirección MAC

> La máquina virtual debe tener una dirección MAC diferente a la original.

Con la máquina virtual **apagada**:

1. Abrí **Configuración** en VirtualBox.
2. Entrá en **Red**.
3. Seleccioná el adaptador activo.
4. Abrí **Avanzadas**.
5. Hacé clic en el botón de regeneración situado junto a **Dirección MAC**.
6. Guardá los cambios.

---

## 2. Nombre de host

Usá un nombre diferente al de la máquina original.

En este ejemplo:

```text
db02
```

Ejecutá el siguiente comando:

```bash
sudo hostnamectl set-hostname db02
```

Después, editá el archivo `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Reemplazá el nombre anterior por el nuevo:

```text
127.0.1.1   db02
```

---

## 3. Identificador del sistema

Linux utiliza el archivo `machine-id` para identificar de forma única cada equipo.

### Eliminar el identificador heredado

```bash
sudo truncate -s 0 /etc/machine-id
sudo rm -f /var/lib/dbus/machine-id
```

### Crear un identificador nuevo

```bash
sudo systemd-machine-id-setup
```

### Vincularlo con D-Bus

```bash
sudo ln -s /etc/machine-id /var/lib/dbus/machine-id
```

---

## 4. Claves SSH

Si la máquina utiliza OpenSSH, también es necesario regenerar sus claves de host.

De lo contrario, la máquina original y la clonada conservarán las mismas claves y podrían mostrar este error:

```text
REMOTE HOST IDENTIFICATION HAS CHANGED
```

### Eliminar las claves anteriores

```bash
sudo rm -f /etc/ssh/ssh_host_*
```

### Crear claves nuevas

```bash
sudo dpkg-reconfigure openssh-server
```

### Reiniciar SSH

```bash
sudo systemctl restart ssh
```

---

## 5. Reinicio

Reiniciá la máquina para aplicar todos los cambios:

```bash
sudo reboot
```

---

## 6. Verificación

Después de reiniciar, ejecutá:

### Comprobar la dirección IP

```bash
ip a
```

### Comprobar el nombre de host

```bash
hostname
```

### Comprobar el identificador del sistema

```bash
cat /etc/machine-id
```

---

## Resultado esperado

La máquina clonada debe tener:

- Una dirección MAC diferente.
- Una dirección IP propia.
- Un nombre de host diferente.
- Un `machine-id` único.
- Nuevas claves SSH de host.
````*_
