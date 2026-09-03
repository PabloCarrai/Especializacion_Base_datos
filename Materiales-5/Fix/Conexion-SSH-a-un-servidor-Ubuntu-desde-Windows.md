# 🔐 Conexión SSH a un servidor Ubuntu desde Windows

## 📘 ¿Qué es SSH?

**SSH (Secure Shell)** es un protocolo de red que permite establecer una conexión remota, segura y cifrada con otro equipo.

Es una de las herramientas fundamentales para la administración de servidores Linux, especialmente cuando el servidor no dispone de una interfaz gráfica o se encuentra en un datacenter.

Mediante SSH podemos:

- Iniciar sesión remotamente en un servidor.
- Ejecutar comandos.
- Administrar servicios.
- Instalar y actualizar software.
- Modificar archivos de configuración.
- Consultar logs.
- Administrar usuarios y permisos.
- Reiniciar o apagar el servidor.
- Transferir archivos mediante SCP o SFTP.

Por ejemplo, un administrador puede encontrarse físicamente en una PC con Windows y administrar un servidor Ubuntu ubicado en otro edificio o incluso en otro país.

## 🛡️ ¿Por qué SSH es seguro?

SSH cifra la comunicación entre el cliente y el servidor.

Esto significa que información como:

- Contraseñas.
- Comandos.
- Archivos transferidos.
- Respuestas del servidor.

no viaja en texto plano por la red.

SSH reemplazó en gran medida a protocolos antiguos como **Telnet**, que no proporcionaban este nivel de protección.

## 🏗️ Arquitectura básica de una conexión SSH

En una conexión SSH intervienen dos componentes:

```text
Cliente SSH  →  Servidor SSH
```

El cliente es el equipo desde el cual nos conectamos.

En nuestro caso:

```text
Windows  →  Ubuntu
```

El servidor SSH es un servicio que se está ejecutando en Ubuntu y espera conexiones entrantes.

Por defecto, SSH utiliza el:

```text
Puerto TCP 22
```

### Diagrama de conexión

```text
PC Windows
192.168.1.100
       |
       | TCP/22
       |
       v
Servidor Ubuntu
192.168.1.50
```

> El puerto puede modificarse por razones de seguridad o de organización, aunque cambiarlo por sí solo no constituye una medida de seguridad suficiente.

# ✅ Requisitos

## 🖥️ Servidor Ubuntu

El servidor debe tener instalado el servidor SSH.

En Ubuntu se utiliza normalmente el paquete:

```text
openssh-server
```

### 1. Actualizar la información de los paquetes

```bash
sudo apt update
```

### 2. Instalar OpenSSH Server

```bash
sudo apt install openssh-server
```

### 3. Verificar el estado del servicio

```bash
sudo systemctl status ssh
```

Deberíamos observar un estado similar a:

```text
Active: active (running)
```

Esto significa que el servicio está funcionando.

## ▶️ Iniciar el servicio

Si el servicio estuviera detenido:

```bash
sudo systemctl start ssh
```

## 🔄 Iniciar SSH automáticamente

Para que SSH se inicie automáticamente cuando arranca Ubuntu:

```bash
sudo systemctl enable ssh
```

Podemos comprobar nuevamente el estado:

```bash
sudo systemctl status ssh
```

## 🌐 Conocer la dirección IP del servidor

Para conectarnos necesitamos conocer la dirección IP del servidor Ubuntu.

Podemos utilizar:

```bash
ip a
```

También podemos consultar las direcciones asignadas mediante:

```bash
hostname -I
```

Por ejemplo:

```text
192.168.1.50
```

En este caso, esa será la dirección que utilizaremos desde Windows.

Es importante distinguir entre:

- **IP privada:** utilizada dentro de una red local.
- **IP pública:** utilizada para acceder desde Internet.

> En un entorno de producción, normalmente no se recomienda exponer directamente el puerto SSH de un servidor a Internet sin controles adicionales.

# 🪟 Windows como cliente SSH

Las versiones actuales de **Windows 10** y **Windows 11** incluyen un cliente SSH basado en OpenSSH.

Por lo tanto, normalmente no es necesario instalar ningún programa adicional.

Podemos utilizar:

- PowerShell.
- Símbolo del sistema (CMD).
- Windows Terminal.

Para comprobar si SSH está disponible:

```bash
ssh
```

Si el comando está instalado, Windows mostrará la ayuda y las opciones disponibles.

# 🚀 Primera conexión

La sintaxis básica es:

```bash
ssh usuario@direccion_ip
```

Por ejemplo:

```bash
ssh user@192.168.1.50
```

En este ejemplo:

- `ssh` → programa cliente.
- `user` → usuario existente en Ubuntu.
- `192.168.1.50` → dirección IP del servidor.

También podemos especificar un puerto diferente al 22 utilizando `-p`:

```bash
ssh -p 2222 user@192.168.1.50
```

## ❓ ¿Qué ocurre durante la primera conexión?

Cuando nos conectamos por primera vez, el cliente SSH todavía no conoce al servidor.

Por eso puede aparecer un mensaje similar a:

```text
The authenticity of host '192.168.1.50' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

SSH muestra una huella digital (*fingerprint*) de la clave pública del servidor.

Si verificamos que la huella corresponde al servidor correcto, podemos responder:

```text
yes
```

La clave del servidor queda registrada en el equipo cliente.

En conexiones posteriores, SSH puede detectar si la clave del servidor cambió. Esto es importante porque permite detectar situaciones en las que alguien intenta hacerse pasar por el servidor, conocidas como ataques **Man-in-the-Middle (MITM)**.

# 🔑 Autenticación

Una vez establecida la conexión con el servidor, SSH necesita comprobar nuestra identidad.

Los dos mecanismos más habituales son:

## 👤 Usuario y contraseña

Es el método más sencillo.

```bash
ssh user@192.168.1.50
```

El servidor solicitará:

```text
user@192.168.1.50's password:
```

Al escribir la contraseña, normalmente no se muestran caracteres en pantalla. Esto es normal.

## 🔐 Autenticación mediante claves SSH

Para ambientes administrados profesionalmente es preferible utilizar claves SSH.

El sistema utiliza un par de claves:

### Clave privada

- Permanece en el equipo del usuario.
- Nunca debe copiarse al servidor.
- No debe compartirse con otras personas.

### Clave pública

- Se instala en el servidor.
- Puede compartirse para configurar la autenticación.

### Relación entre las claves

```text
PC Windows
   |
   | Clave privada
   |
   v
[Cliente SSH]
      |
      | Conexión
      v
[Servidor Ubuntu]
   |
   | Clave pública
```

La clave privada demuestra que el usuario posee la credencial correspondiente a la clave pública instalada en el servidor.

# 🛠️ Generar una clave SSH en Windows

Desde PowerShell podemos ejecutar:

```bash
ssh-keygen
```

El programa preguntará dónde guardar la clave. Normalmente podemos aceptar la ubicación propuesta.

También solicitará una **passphrase**.

> Es recomendable utilizar una passphrase para proteger la clave privada.

Se generarán dos archivos:

```text
id_ed25519
id_ed25519.pub
```

- `id_ed25519` → clave privada.
- `id_ed25519.pub` → clave pública.

> ⚠️ La clave privada debe mantenerse protegida.

## 📤 Copiar la clave pública al servidor

La clave pública debe agregarse al archivo:

```text
~/.ssh/authorized_keys
```

En Linux existe una herramienta llamada:

```bash
ssh-copy-id
```

que permite realizar esta operación fácilmente.

Sin embargo, `ssh-copy-id` no siempre está disponible de forma predeterminada en Windows.

Una alternativa es copiar el contenido de la clave pública y agregarlo manualmente al archivo `authorized_keys` del usuario en Ubuntu.

### 1. Crear el directorio `.ssh`

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

### 2. Editar el archivo de claves autorizadas

```bash
nano ~/.ssh/authorized_keys
```

Cada clave pública debe ocupar una línea.

### 3. Establecer los permisos correctos

```bash
chmod 600 ~/.ssh/authorized_keys
```

## 🔗 Conectarse utilizando la clave

Una vez instalada la clave pública:

```bash
ssh user@192.168.1.50
```

SSH utilizará la clave privada disponible en el equipo cliente.

Si la clave está protegida mediante passphrase, solicitará esa passphrase.

# 🖥️ ¿Qué ocurre cuando iniciamos sesión?

Una vez autenticados, obtenemos una shell remota.

Por ejemplo:

```text
user@ubuntu-server:~$
```

A partir de ese momento, los comandos que escribimos se ejecutan en el servidor Ubuntu, no en Windows.

# ⚡ Ejecutar un único comando remotamente

No es necesario iniciar una sesión interactiva para ejecutar un comando.

Por ejemplo:

```bash
ssh user@192.168.1.50 "hostname"
```

El servidor ejecutará:

```bash
hostname
```

y devolverá el resultado a Windows.

Esto es especialmente útil para automatización y administración remota.

Por ejemplo:

```bash
ssh user@192.168.1.50 "df -h"
```

permite consultar el espacio en disco remotamente.

# 📁 Transferencia de archivos mediante SCP

SSH también permite transferir archivos utilizando **SCP (Secure Copy Protocol)**.

## Copiar un archivo desde Windows hacia Ubuntu

```bash
scp archivo.txt user@192.168.1.50:/home/user/
```

## Copiar un archivo desde Ubuntu hacia Windows

```bash
scp user@192.168.1.50:/home/user/archivo.txt .
```

SCP utiliza la conexión SSH y, por lo tanto, la transferencia se realiza de forma cifrada.

# 📂 SFTP

Otra alternativa es **SFTP (SSH File Transfer Protocol)**.

SFTP permite administrar archivos remotamente mediante una conexión SSH.

Desde Windows podemos utilizar clientes gráficos como **WinSCP**, que permiten:

- Subir archivos.
- Descargar archivos.
- Crear directorios.
- Modificar permisos.
- Navegar por el sistema de archivos.

SFTP resulta especialmente cómodo cuando no queremos trabajar exclusivamente desde la línea de comandos.

# 🧱 Firewall y SSH

Ubuntu puede utilizar **UFW (Uncomplicated Firewall)** para administrar el firewall.

## Consultar el estado del firewall

```bash
sudo ufw status
```

## Permitir SSH

```bash
sudo ufw allow ssh
```

Esto normalmente habilita el puerto TCP 22.

También podemos especificar explícitamente el puerto:

```bash
sudo ufw allow 22/tcp
```

## Habilitar UFW

```bash
sudo ufw enable
```

> ⚠️ **Advertencia importante:** si estamos conectados remotamente al servidor mediante SSH, debemos tener especial cuidado al modificar el firewall. Una configuración incorrecta puede bloquear nuestra propia conexión.

Antes de habilitar UFW en un servidor remoto, debemos asegurarnos de permitir el acceso SSH.

# 🐞 Problemas comunes

## `Connection refused`

Ejemplo:

```text
ssh: connect to host 192.168.1.50 port 22: Connection refused
```

Puede indicar:

- El servicio SSH no está funcionando.
- SSH está escuchando en otro puerto.
- Un firewall está rechazando la conexión.

En Ubuntu:

```bash
sudo systemctl status ssh
```

También podemos verificar qué puerto está escuchando:

```bash
sudo ss -lntp
```

## `Connection timed out`

Ejemplo:

```text
Connection timed out
```

Suele indicar un problema de conectividad o filtrado.

Debemos comprobar:

- Dirección IP.
- Conectividad de red.
- Rutas.
- Firewall.
- VLAN.
- Reglas de seguridad.
- Accesibilidad del puerto TCP 22.

Desde Windows podemos probar:

```powershell
Test-NetConnection 192.168.1.50 -Port 22
```

Este comando resulta muy útil para determinar si Windows puede alcanzar el puerto SSH.

## `Permission denied`

Ejemplo:

```text
Permission denied
```

Puede deberse a:

- Usuario incorrecto.
- Contraseña incorrecta.
- Clave SSH incorrecta.
- Clave pública no instalada correctamente.
- Permisos incorrectos en `~/.ssh`.
- El servidor no permite el método de autenticación utilizado.

# ⚙️ Archivo de configuración del servidor SSH

La configuración principal del servidor se encuentra normalmente en:

```text
/etc/ssh/sshd_config
```

Podemos consultar o editar el archivo:

```bash
sudo nano /etc/ssh/sshd_config
```

Algunos parámetros importantes son:

```text
Port 22
PermitRootLogin no
PasswordAuthentication yes
PubkeyAuthentication yes
```

Después de modificar la configuración debemos reiniciar o recargar el servicio:

```bash
sudo systemctl restart ssh
```

> Antes de realizar cambios importantes conviene validar cuidadosamente la configuración, ya que un error puede impedir nuevas conexiones SSH.

# 🔒 Recomendaciones básicas de seguridad

En un servidor de producción se recomienda:

## 1. No permitir acceso SSH directo al usuario root

```text
PermitRootLogin no
```

Es preferible conectarse con un usuario normal y utilizar `sudo` cuando se necesiten privilegios administrativos.

## 2. Utilizar claves SSH

Las claves permiten evitar depender exclusivamente de contraseñas.

## 3. Proteger la clave privada

La clave privada debe permanecer protegida en el equipo del administrador.

## 4. Limitar quién puede conectarse

Puede restringirse el acceso SSH mediante:

- Firewall.
- Redes administrativas.
- VPN.
- Listas de IP permitidas.
- Reglas de seguridad.

## 5. Mantener Ubuntu actualizado

```bash
sudo apt update
sudo apt upgrade
```

Las actualizaciones incluyen correcciones de seguridad.

## 6. No exponer SSH innecesariamente a Internet

Siempre que sea posible, el acceso administrativo debería realizarse desde una red de administración o mediante una VPN.

# 🔀 Cambiar el puerto SSH

SSH utiliza por defecto:

```text
TCP/22
```

Es posible utilizar otro puerto modificando:

```text
/etc/ssh/sshd_config
```

Por ejemplo:

```text
Port 2222
```

Después reiniciamos el servicio:

```bash
sudo systemctl restart ssh
```

Y desde Windows nos conectamos utilizando:

```bash
ssh -p 2222 user@192.168.1.50
```

> ⚠️ **Importante:** cambiar el puerto puede reducir ciertos intentos automatizados contra el puerto 22, pero no debe considerarse una medida de seguridad suficiente.

La seguridad debe basarse principalmente en:

- Autenticación robusta.
- Restricciones de red.
- Firewall.
- Actualizaciones.
- Controles de acceso.

# 🧰 Herramientas disponibles en Windows

Además del cliente SSH incluido en Windows, existen herramientas gráficas.

| Herramienta | Descripción |
|---|---|
| **Windows Terminal** | Permite trabajar con PowerShell, CMD y otros shells desde una única aplicación. |
| **PuTTY** | Cliente SSH muy utilizado, especialmente en entornos Windows. |
| **WinSCP** | Cliente gráfico orientado principalmente a la transferencia de archivos mediante SFTP/SCP. |

# ✅ Resumen

SSH permite administrar de forma remota y segura un servidor Ubuntu desde Windows.

Las operaciones más habituales son:

- Conectarse mediante usuario y contraseña.
- Utilizar claves SSH.
- Ejecutar comandos remotamente.
- Transferir archivos mediante SCP o SFTP.
- Administrar servicios y configuraciones.
- Controlar el acceso mediante firewall.
- Aplicar buenas prácticas de seguridad.
```
