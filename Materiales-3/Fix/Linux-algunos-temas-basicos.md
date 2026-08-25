# Linux: Algunos Temas Básicos

Una guía rápida y estructurada con los comandos y conceptos esenciales para la administración y navegación en sistemas Linux.

---

## 1. Estructura del Sistema de Archivos y Navegación

En Linux todo se organiza a partir de una **raíz única (`/`)**. No existen letras de unidad (`C:`, `D:`).

### Comandos principales

*   `pwd` (**Print Working Directory**)
    *   **Descripción:** Muestra la ruta absoluta del directorio actual.
    *   **Uso:** Saber exactamente dónde estás parado en el árbol de directorios.
    *   **Ejemplo:** `pwd` $\rightarrow$ Devuelve `/home/alumno`.

*   `ls` (**List**)
    *   **Descripción:** Lista el contenido de un directorio.
    *   **Variantes útiles:**
        *   `ls -l`: Formato largo (muestra permisos, propietario, grupo, tamaño y fecha).
        *   `ls -a`: Muestra archivos ocultos (los que empiezan con un punto `.`).
        *   `ls -lh`: Muestra tamaños en formato legible para humanos (KB, MB, GB).
    *   **Ejemplo:** `ls -lah /var/log`

*   `cd` (**Change Directory**)
    *   **Descripción:** Cambia el directorio de trabajo.
    *   **Ejemplos:**
        *   `cd /etc` $\rightarrow$ Va a una ruta absoluta.
        *   `cd ..` $\rightarrow$ Sube un nivel en la jerarquía.
        *   `cd ~` (o simplemente `cd`) $\rightarrow$ Va al directorio personal del usuario actual (`/home/<usuario>`).

*   `mkdir` (**Make Directory**)
    *   **Descripción:** Crea nuevas carpetas.
    *   **Bandera clave:** `-p` (crea directorios padres si no existen de forma anidada).
    *   **Ejemplo:** `mkdir -p proyectos/servidor/config`

*   `cp` (**Copy**)
    *   **Descripción:** Copia archivos y directorios.
    *   **Bandera clave:** `-r` (modo recursivo, obligatorio para copiar carpetas enteras).
    *   **Ejemplo:** `cp -r proyectos/ /backup/`

*   `mv` (**Move**)
    *   **Descripción:** Mueve archivos de ubicación o los renombra.
    *   **Ejemplo:** `mv archivo.txt nuevo_nombre.txt`

*   `rm` (**Remove**)
    *   **Descripción:** Elimina archivos y directorios.
    *   **Banderas clave:**
        *   `-r`: Recursivo (borra carpetas con su contenido).
        *   `-f`: Forzado (sin pedir confirmación interactiva).
    *   **Ejemplo:** `rm -rf carpeta_temporal` *(¡Usar con extrema precaución!)*

---

## 2. Usuarios, Grupos y Permisos POSIX

Cada archivo y directorio tiene 3 entidades: **Usuario** (dueño), **Grupo** y **Otros** (resto del mundo). Los permisos básicos son Lectura (`r = 4`), Escritura (`w = 2`) y Ejecución (`x = 1`).

### Comandos de gestión

*   `sudo` (**SuperUser DO**)
    *   **Descripción:** Ejecuta comandos con privilegios administrativos (`root`).
    *   **Uso:** Modificar configuraciones del sistema o instalar software.

*   `useradd` / `adduser`
    *   **Descripción:** Crea nuevos usuarios. `adduser` es un script interactivo más amigable en distribuciones basadas en Debian/Ubuntu.
    *   **Ejemplo:** `sudo adduser devops`

*   `usermod` (**User Modify**)
    *   **Descripción:** Modifica propiedades de un usuario existente.
    *   **Bandera clave:** `-aG` (agrega el usuario a un grupo secundario sin quitarlo de los actuales).
    *   **Ejemplo:** `sudo usermod -aG sudo devops` (le otorga permisos administrativos).

*   `chmod` (**Change Mode**)
    *   **Descripción:** Modifica los permisos de lectura, escritura y ejecución.
    *   **Notación numérica:** Suma de valores (`Lectura=4`, `Escritura=2`, `Ejecución=1`).
        *   `755` (`rwxr-xr-x`): Dueño tiene control total; grupo y otros pueden leer y ejecutar.
        *   `644` (`rw-r--r--`): Dueño lee y escribe; grupo y otros solo leen.
        *   `700` (`rwx------`): Acceso total exclusivo para el dueño.
    *   **Ejemplo:** `chmod 755 script.sh`

*   `chown` (**Change Owner**)
    *   **Descripción:** Cambia el usuario y/o grupo propietario de un archivo o carpeta.
    *   **Sintaxis:** `chown usuario:grupo archivo`
    *   **Ejemplo:** `sudo chown -R www-data:www-data /var/www/html`

---

## 3. Gestión de Paquetes y Software (APT)

*   `sudo apt update`
    *   **Descripción:** Actualiza la lista de repositorios y las versiones de paquetes disponibles contra los servidores remotos. *No instala ni actualiza programas*, solo actualiza el índice local.

*   `sudo apt upgrade`
    *   **Descripción:** Descarga e instala las versiones más recientes de todos los paquetes del sistema que tienen actualizaciones pendientes.

*   `apt search <termino>`
    *   **Descripción:** Busca paquetes disponibles en los repositorios que coincidan con la palabra clave.
    *   **Ejemplo:** `apt search mariadb-server`

*   `sudo apt install <paquete>`
    *   **Descripción:** Descarga e instala un paquete junto a todas sus dependencias necesarias.
    *   **Ejemplo:** `sudo apt install htop curl`

*   `sudo apt remove <paquete>`
    *   **Descripción:** Desinstala el programa, pero conserva sus archivos de configuración locales.

*   `sudo apt purge <paquete>`
    *   **Descripción:** Desinstala el programa y elimina también por completo todos sus archivos de configuración.

---

## 4. Procesos y Monitoreo de Recursos

*   `ps` (**Process Status**)
    *   **Descripción:** Muestra una captura estática de los procesos en ejecución.
    *   **Variante:** `ps aux` (muestra absolutamente todos los procesos de todos los usuarios con detalle de CPU, memoria y PID).
    *   **Ejemplo con filtro:** `ps aux | grep nginx`

*   `top` / `htop`
    *   **Descripción:** 
        *   `top`: Monitor de recursos interactivo en tiempo real integrado por defecto en la mayoría de sistemas.
        *   `htop`: Versión visual interactiva y a color de `top`. Permite ver núcleos de CPU individualmente, barra de memoria, ordenar por columnas y finalizar procesos directamente de forma intuitiva.

*   `kill` y `pkill`
    *   **Descripción:** Envían señales de terminación a los procesos.
    *   **Ejemplos:**
        *   `kill 1234` (Señal `SIGTERM` / `15`: cierre ordenado y seguro).
        *   `kill -9 1234` (Señal `SIGKILL` / `9`: cierre forzado e inmediato).
        *   `sudo pkill apache2` (Mata procesos buscándolos directamente por su nombre en lugar de requerir el PID).

*   `df -h` (**Disk Free**)
    *   **Descripción:** Muestra el espacio libre y ocupado en todas las particiones montadas en el sistema en un formato legible para humanos (`-h`: GB/MB).

*   `du -sh <directorio>` (**Disk Usage**)
    *   **Descripción:** Calcula el peso real que ocupa una carpeta específica y todo su contenido en disco.
    *   **Ejemplo:** `du -sh /var/log/*`

---

## 5. Gestión de Servicios y Logs con Systemd

`systemctl` es la utilidad central para controlar el sistema de inicio y gestión de servicios (`systemd`).

*   `systemctl status <servicio>`
    *   **Descripción:** Muestra el estado actual (activo, inactivo, fallido), tiempo de actividad (*uptime*) y las últimas líneas de registro (*logs*).
    *   **Ejemplo:** `systemctl status ssh`

*   `sudo systemctl start <servicio>` $\rightarrow$ Inicia el servicio inmediatamente.
*   `sudo systemctl stop <servicio>` $\rightarrow$ Detiene el servicio en ejecución.
*   `sudo systemctl restart <servicio>` $\rightarrow$ Reinicia el servicio por completo (muy útil tras aplicar cambios de configuración).
*   `sudo systemctl reload <servicio>` $\rightarrow$ Recarga la configuración del servicio sin interrumpir ni cortar las conexiones activas.
*   `sudo systemctl enable <servicio>` $\rightarrow$ Configura el servicio para que arranque automáticamente cada vez que el sistema operativo inicia.
*   `sudo systemctl disable <servicio>` $\rightarrow$ Evita que el servicio arranque de forma automática con el sistema.

---

## 6. Redes Básicas y Diagnóstico de Conectividad

*   `ip a` (**IP Address**)
    *   **Descripción:** Muestra todas las interfaces de red disponibles en el sistema (físicas, virtuales, loopback) y las direcciones IPv4/IPv6 asignadas. Reemplaza al clásico comando `ifconfig`.

*   `ping <host>`
    *   **Descripción:** Envía paquetes ICMP para verificar si un host remoto responde a nivel de red y medir la latencia.
    *   **Ejemplo:** `ping -c 4 8.8.8.8` (envía exactamente 4 paquetes y se detiene automáticamente).

*   `ss` (**Socket Statistics**)
    *   **Descripción:** Inspecciona sockets abiertos y conexiones de red activas. Reemplaza a la utilidad `netstat`.
    *   Banderas útiles:
        *   `-t`: Conexiones TCP.
        *   `-u`: Conexiones UDP.
        *   `-l`: En escucha (*listening*).
        *   `-p`: Muestra el proceso/PID dueño del socket.
        *   `-n`: Muestra números de puerto numéricos en lugar de nombres de servicio.
    *   **Ejemplo típico:** `sudo ss -tulpn` (indispensable para verificar qué servicios están escuchando y en qué puertos del sistema).