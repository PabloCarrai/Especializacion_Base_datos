# Configuración de IPs de las VMs

Para lograr que las máquinas virtuales (VMs) tengan salida a internet (para descargar paquetes) y a la vez puedas conectarte a ellas por SSH desde tu ordenador anfitrión (host), la estrategia más recomendada y profesional en VirtualBox es utilizar dos adaptadores de red por cada máquina virtual:

* **Adaptador 1:** Configurado en modo **NAT** (Se encarga exclusivamente de proveer internet a la VM).
* **Adaptador 2:** Configurado en modo **Adaptador solo anfitrión (Host-Only)** (Crea una red interna privada entre tu PC y las VMs para el tráfico local, las IPs estáticas y el acceso SSH).

---

## Paso 1: Configurar las interfaces de red en VirtualBox (Interfaz gráfica)

Antes de encender las máquinas (o con ellas apagadas), se deben configurar los adaptadores en la configuración de VirtualBox para cada nodo (`node1`, `node2`, `node3`):

1. Abrir **VirtualBox**, seleccionar la máquina virtual y hacer clic en **Configuración** $\rightarrow$ **Red**.
2. **Adaptador 1:**
   * Marcar la casilla *"Habilitar adaptador de red"*.
   * Conectado a: **NAT**. (Esto les da internet automáticamente).
3. **Adaptador 2:**
   * Marcar la casilla *"Habilitar adaptador de red"*.
   * Conectado a: **Adaptador sólo anfitrión (Host-Only Adapter)**.

> **Nota:** Asegurarse de verificar en el menú superior de VirtualBox en **Archivo** $\rightarrow$ **Herramientas** $\rightarrow$ **Administrador de redes host** (o *Host Network Manager*) qué rango de IP utiliza esta red (por ejemplo, suele ser `192.168.56.x`).

---

## Paso 2: Configurar IPs estáticas en Ubuntu 24.04 usando Netplan

Ubuntu 24.04 utiliza **Netplan** para administrar las redes. Se deben encender las VMs y configurar las direcciones IP estáticas para el Adaptador 2 (el de la red Solo Anfitrión), permitiendo que el Adaptador 1 (NAT) tome su IP dinámica habitual para el internet.

1. Conectarse a la terminal del nodo (por ejemplo, `node1`).
2. Buscar el archivo de configuración de Netplan dentro de `/etc/netplan/`:
   ```bash
   ls /etc/netplan/
   ```
   Generalmente se llama `01-netcfg.yaml` o `50-cloud-init.yaml`.

3. Editar el archivo con privilegios de superusuario (reemplazar el nombre del archivo por el que tengas):
   ```bash
   sudo nano /etc/netplan/01-netcfg.yaml
   ```

4. Modificar o añadir la estructura para que sea similar a esto (ajustar las interfaces según corresponda, generalmente `enp0s3` es NAT y `enp0s8` es Host-Only):
   ```yaml
   network:
     version: 2
     renderer: networkd
     ethernets:
       enp0s3:
         dhcp4: true  # Adaptador 1: NAT (Internet automático)
       
       enp0s8:
         dhcp4: false
         addresses:
           - 192.168.56.51/24  # IP estática para el Nodo 1 (Usa .52 para nodo2 y .53 para nodo3)
         nameservers:
           addresses: [8.8.8.8, 1.1.1.1]
   ```

> **Nota importante:** Para saber los nombres exactos de las tarjetas de red (`enp0s3`, `enp0s8`, etc.), se puede ejecutar previamente el comando `ip a`. Asegurarse de respetar la identación de los espacios en archivos de configuración (YAML).

5. Guardar los cambios (en Nano: `Ctrl + O`, `Enter` y luego `Ctrl + X`).
6. Aplicar la configuración de red con Netplan:
   ```bash
   sudo netplan apply
   ```

---

## Paso 3: Comprobar el acceso a Internet y la red local

Una vez aplicado Netplan en cada nodo:

* **Probar Internet (Adaptador NAT):**
  Ejecutar un ping a un servidor externo para verificar que las actualizaciones funcionarán:
  ```bash
  ping -c 3 google.com
  ```
  Si responde correctamente, los nodos ya tienen salida a internet para correr `sudo apt update`.

* **Probar comunicación entre nodos y el Anfitrión:**
  Desde el ordenador anfitrión (la máquina física), abrir la terminal (PowerShell, CMD o terminal de Linux/Mac) e intentar hacer ping a la IP estática que configuraste:
  ```bash
  ping 192.168.56.51
  ```

---

## Paso 4: Conectarse por SSH desde el Anfitrión

Para poder administrar los nodos de MariaDB/Galera cómodamente desde la terminal de tu máquina anfitriona:

1. Instala el servidor SSH en cada nodo (si no viene preinstalado):
   ```bash
   sudo apt install openssh-server -y
   sudo systemctl enable --now ssh
   ```

2. Desde la terminal de tu computadora anfitriona, conéctate vía SSH utilizando el usuario de tu máquina virtual y la IP estática asignada:
   ```bash
   ssh tu_usuario@192.168.56.51
   ```
   *(Introduce la contraseña de la VM cuando te la solicite).*

De esta manera, se mantiene una red completamente aislada y estable para el clúster de base de datos (`192.168.56.x`) y, al mismo tiempo, las máquinas virtuales se valen del adaptador NAT para salir a internet a descargar paquetes o actualizaciones.