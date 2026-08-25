# Guía de Instalación: Ubuntu Server en una Máquina Virtual de VirtualBox

Una guía paso a paso, clara y optimizada para desplegar un entorno de pruebas o desarrollo profesional en VirtualBox.

---

## Prerrequisitos y Descarga
*   **Imagen ISO:** Descargar la versión más reciente LTS desde la [web oficial de Ubuntu Releases](https://releases.ubuntu.com/noble/).

---

## 1. Crear y Configurar la Máquina Virtual en VirtualBox

Abrir VirtualBox y hacer clic en **Nueva** para iniciar el asistente de creación.

*   **Nombre y Sistema Operativo:**
    *   **Nombre:** Asignar un identificador claro (ej. `Db_01`).
    *   **Imagen ISO:** Seleccionar el archivo `.iso` descargado de Ubuntu Server.
    *   **Importante:** Marcar la casilla **Omitir instalación desatendida** (*Skip Unattended Installation*) para mantener el control total del particionado, la red y la creación de usuarios durante la instalación clásica.

*   **Hardware:**
    *   **Memoria RAM:** Asignar al menos `2048 MB` (2 GB). *Recomendado: `4096 MB` (4 GB) si vas a correr servicios de backend o bases de datos.*
    *   **Procesadores (CPUs):** Asignar al menos `2 CPUs` virtuales.

*   **Disco Duro Virtual:**
    *   Seleccionar **Crear un disco duro virtual ahora**.
    *   **Tamaño:** Al menos `25 GB` (formato VDI, reservado dinámicamente).
    *   Hacer clic en **Terminar**.

---

## 2. Ajustar la Configuración de Red de la VM

Antes de iniciar la máquina, es recomendable configurar el acceso de red:

1. Seleccionar la máquina virtual recién creada y entrar en **Configuración > Red**.
2. En el **Adaptador 1**, cambiar el tipo de conexión de *NAT* a **Adaptador puente** (*Bridged Adapter*).
3. **Propósito:** Esto permite que el servidor tome una dirección IP directamente de tu router local, facilitando el acceso por SSH o la conexión a bases de datos directamente desde tu máquina anfitriona (host).

---

## 3. Iniciar la VM y Seleccionar el Arranque

1. Iniciar la máquina virtual.
2. En la pantalla inicial del gestor de arranque GRUB, seleccionar **Try or Install Ubuntu Server** y presionar `Enter`.
3. El instalador en modo texto comenzará a cargar el kernel de Linux y los módulos básicos en la memoria RAM.

---

## 4. Idioma y Distribución del Teclado

*   **Language:** Seleccionar el idioma preferido para el asistente de instalación (ej. *English* o *Español*).
*   **Installer update:** Si el instalador detecta una versión más reciente de sí mismo, selecciona **Continue without updating** (o actualizar si dispones de una conexión a Internet muy estable).
*   **Keyboard configuration:** Seleccionar la distribución adecuada para tu teclado físico (ej. *Spanish* o *Spanish (Latin America)*) para evitar discrepancias al tipear contraseñas complejas o comandos en la consola.

---

## 5. Tipo de Instalación y Configuración de Red

*   **Type of Install:** Elegir **Ubuntu Server** (versión estándar) en lugar de la versión *Minimized*, ya que incluye herramientas de red y diagnóstico esenciales para la administración.
*   **Network connections:** El instalador intentará obtener una IP automáticamente mediante DHCP. Puedes continuar con la IP asignada o configurarla como estática utilizando la opción de subred si requieres una IP fija en tu red local.
*   **Configure proxy / Mirror:** Si no utilizas un proxy corporativo, deja el campo en blanco. En el espejo de paquetes (*mirror*), confirma el repositorio sugerido por defecto.

---

## 6. Configuración del Almacenamiento (Particionado)

*   Seleccionar **Use an entire disk** (Utilizar todo el disco).
*   **Importante:** Desmarcar la opción **Set up this disk as an LVM group** si prefieres un esquema de particionado clásico y directo (sin la capa extra de volúmenes lógicos), o dejarla marcada si planeas redimensionar dinámicamente los volúmenes en caliente en el futuro.
*   Revisar el resumen del esquema de particiones propuesto (partición `/boot` y partición raíz `/`) y seleccionar **Done > Continue** para confirmar y aplicar los cambios en el disco.

---

## 7. Perfil de Usuario y Configuración del Host

Completar los datos requeridos para la autenticación y el sistema:

*   **Your name:** Tu nombre real o el del administrador del sistema.
*   **Your server's name:** Nombre identificador del servidor en la red local (ej. `srv-ubuntu`).
*   **Pick a username:** Nombre de usuario para el sistema operativo (*evita usar directamente el usuario root*).
*   **Choose a password:** Contraseña robusta para el usuario (se utilizará también para elevar privilegios mediante el comando `sudo`).

---

## 8. Configuración de SSH y Paquetes Adicionales

*   **Ubuntu Pro:** Selecciona **Skip for now** si no cuentas con una suscripción empresarial activa.
*   **SSH Setup:** Marcar la casilla **`[X] Install OpenSSH server`**. Esto es absolutamente fundamental para poder administrar la máquina virtual de forma remota y cómoda desde tu terminal preferida en el host.
*   **Featured Server Snaps:** No marcar ningún paquete adicional por el momento. Generalmente es mucho más limpio y seguro instalar servicios específicos (como Docker, bases de datos o servidores web) de manera individual más adelante.

---

## 9. Finalización y Primer Reinicio

1. La instalación comenzará a copiar archivos, configurar el sistema base y aplicar las actualizaciones de seguridad iniciales (*security updates*).
2. Cuando el proceso finalice, seleccionar **Reboot Now**.
3. Si la consola solicita el mensaje *Please remove the installation medium, then press ENTER*, VirtualBox suele desmontar la imagen ISO automáticamente; simplemente presiona `Enter`.
4. Tras el reinicio, se mostrará la consola de inicio de sesión (*login prompt*) lista para ingresar con tu nombre de usuario y contraseña configurados. ¡Tu servidor virtual está listo para operar!