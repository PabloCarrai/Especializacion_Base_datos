# Puertos de red y configuración de UFW en Ubuntu

## 1. Concepto de puertos

Un puerto es un punto lógico que utiliza el sistema operativo para gestionar conexiones de red y dirigir el tráfico hacia el servicio correspondiente.

Por ejemplo, permite diferenciar si una conexión está destinada a:

- Un servidor web.
- Un servicio de correo.
- Una transferencia de archivos.
- Una conexión remota.
- Una base de datos.

Cada puerto está identificado por un número. Esto permite que varias aplicaciones funcionen simultáneamente en un mismo equipo utilizando una única conexión de red.

Los puertos trabajan principalmente con los protocolos de transporte:

- **TCP:** orientado a conexión y confiable.
- **UDP:** más rápido, pero sin garantía de entrega.

Por seguridad, los firewalls suelen bloquear la mayoría de los puertos y solo permiten el acceso a los servicios necesarios.

---

## 2. Puertos más utilizados

| Puerto | Servicio | Descripción |
|---:|---|---|
| 20 y 21 | FTP | Transferencia de archivos |
| 22 | SSH | Conexión remota segura |
| 23 | Telnet | Acceso remoto no seguro |
| 25 | SMTP | Envío de correo electrónico |
| 53 | DNS | Resolución de nombres de dominio |
| 67 y 68 | DHCP | Asignación automática de direcciones IP |
| 69 | TFTP | Transferencia simple de archivos |
| 80 | HTTP | Tráfico web |
| 110 | POP3 | Recepción de correo electrónico |
| 123 | NTP | Sincronización de hora |
| 137–139 | NetBIOS | Servicios de red de Windows |
| 143 | IMAP | Acceso a correo electrónico |
| 161 | SNMP | Monitoreo de red |
| 162 | SNMP Trap | Notificaciones SNMP |
| 194 | IRC | Chat |
| 389 | LDAP | Servicios de directorio |
| 443 | HTTPS | Tráfico web seguro |
| 445 | SMB/CIFS | Archivos compartidos en redes Windows |
| 465 | SMTPS | SMTP sobre SSL/TLS |
| 514 | Syslog | Registro de eventos |
| 515 | LPD | Impresión en red |
| 520 | RIP | Protocolo de enrutamiento |
| 587 | SMTP | Envío de correo con autenticación y cifrado |
| 636 | LDAPS | LDAP seguro |
| 989 y 990 | FTPS | FTP seguro |
| 993 | IMAPS | IMAP seguro |
| 995 | POP3S | POP3 seguro |
| 1433 | Microsoft SQL Server | Base de datos |
| 1521 | Oracle Database | Base de datos |
| 2049 | NFS | Sistema de archivos en red |
| 2082 y 2083 | cPanel | Administración web HTTP/HTTPS |
| 2375 y 2376 | Docker | API de Docker |
| 2483 y 2484 | Oracle TCPS | Conexiones seguras a Oracle |
| 3000 | Aplicaciones web | Desarrollo y servicios web |
| 3306 | MySQL/MariaDB | Base de datos |
| 3389 | RDP | Escritorio remoto de Windows |
| 4662 | eMule/P2P | Intercambio de archivos |
| 5432 | PostgreSQL | Base de datos |
| 5900 | VNC | Escritorio remoto |
| 5985 y 5986 | WinRM | Administración remota de Windows |
| 6379 | Redis | Base de datos en memoria |
| 6667 | IRC | Chat |
| 8000 y 8008 | HTTP alternativo | Servicios web |
| 8080 | HTTP alternativo/proxy | Servicios web y proxies |
| 8081 | HTTP alternativo | Servicios web |
| 8090 | HTTP alternativo | Servicios web |
| 8443 | HTTPS alternativo | Servicios web seguros |
| 9418 | Git | Protocolo de Git |
| 11211 | Memcached | Caché distribuida |
| 27017 | MongoDB | Base de datos |

> **Resumen:** los puertos organizan y dirigen el tráfico de red hacia los servicios correspondientes. El firewall controla cuáles de esos puertos pueden utilizarse.

---

# Configuración detallada del firewall UFW

## 3. ¿Qué es un firewall?

Un firewall, también llamado **cortafuegos**, es un mecanismo de seguridad que controla el tráfico de red entrante y saliente de un sistema.

Su función es permitir o bloquear conexiones según un conjunto de reglas previamente definidas. De esta forma, ayuda a proteger el equipo frente a:

- Accesos no autorizados.
- Ataques de red.
- Conexiones sospechosas.
- Usos indebidos de los servicios.

En términos simples, el firewall actúa como un filtro entre el equipo y la red, por ejemplo Internet.

---

## 4. ¿Qué es UFW?

En Ubuntu, la herramienta utilizada para administrar el firewall se llama **UFW**, que significa *Uncomplicated Firewall*.

UFW es una interfaz sencilla para administrar el firewall subyacente del sistema, como `iptables` o `nftables`.

Permite realizar tareas como:

- Activar o desactivar el firewall.
- Permitir o bloquear puertos.
- Restringir conexiones según direcciones IP.
- Definir políticas predeterminadas.
- Limitar intentos de conexión.
- Administrar reglas de forma sencilla.

> **Diferencia principal:** un firewall es el concepto general de protección de red, mientras que UFW es una herramienta concreta para configurar esa protección en Ubuntu.

---

## 5. Comandos básicos de UFW

### 5.1 Verificar si UFW está instalado

```bash
sudo ufw status verbose
```

Si muestra:

```text
Status: inactive
```

significa que UFW está instalado, pero actualmente se encuentra desactivado.

---

### 5.2 Habilitar UFW

Si vas a administrar el equipo mediante SSH, permití primero ese servicio:

```bash
sudo ufw allow ssh
```

Después, activá UFW:

```bash
sudo ufw enable
```

> **Importante:** si activás UFW sin permitir previamente SSH, podrías perder el acceso remoto al servidor.

---

### 5.3 Ver las reglas activas

```bash
sudo ufw status numbered
```

Este comando muestra las reglas activas junto con su número de identificación.

---

## 6. Permitir conexiones

### 6.1 Permitir puertos específicos

Permitir conexiones SSH:

```bash
sudo ufw allow 22
```

Permitir conexiones a MariaDB:

```bash
sudo ufw allow 3306
```

Permitir tráfico HTTP:

```bash
sudo ufw allow 80
```

Permitir tráfico HTTPS:

```bash
sudo ufw allow 443
```

También podés utilizar el nombre del servicio:

```bash
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
```

---

### 6.2 Permitir un rango de puertos

Para permitir un rango de puertos TCP:

```bash
sudo ufw allow 10000:20000/tcp
```

---

### 6.3 Permitir un puerto UDP

Para permitir únicamente el puerto UDP `1194`:

```bash
sudo ufw allow 1194/udp
```

---

### 6.4 Permitir el acceso desde una IP específica

El siguiente comando permite que la IP `192.168.1.100` acceda al puerto SSH:

```bash
sudo ufw allow from 192.168.1.100 to any port 22
```

---

## 7. Limitar conexiones SSH

Para limitar la cantidad de intentos de conexión al servicio SSH:

```bash
sudo ufw limit ssh
```

Esta regla ayuda a reducir los intentos repetidos de conexión y ofrece protección básica contra ataques de fuerza bruta.

---

## 8. Denegar conexiones

### Denegar un puerto

```bash
sudo ufw deny 8080
```

### Denegar una dirección IP

```bash
sudo ufw deny from 192.168.1.50
```

---

## 9. Eliminar reglas

Primero, mostrálas junto con su número:

```bash
sudo ufw status numbered
```

Después, eliminá la regla indicando su número:

```bash
sudo ufw delete 3
```

> El número de la regla puede variar. Verificá siempre la lista antes de eliminarla.

---

## 10. Desactivar UFW

Para desactivar temporalmente el firewall:

```bash
sudo ufw disable
```

---

## 11. Restablecer todas las reglas

Para eliminar todas las reglas y dejar UFW deshabilitado:

```bash
sudo ufw reset
```

> **Atención:** este comando borra todas las reglas configuradas.

---

## 12. Verificar la configuración final

Para consultar el estado actual del firewall:

```bash
sudo ufw status verbose
```

Una salida habitual puede indicar:

```text
Status: active
Logging: on
Default: deny (incoming), allow (outgoing)
```

Esto significa que:

- El firewall está activo.
- El registro de eventos está habilitado.
- Las conexiones entrantes están bloqueadas por defecto.
- Las conexiones salientes están permitidas por defecto.

---

# Puertos en escucha

## 13. Ver qué puertos están escuchando

Para consultar los puertos abiertos y los procesos asociados, ejecutá:

```bash
sudo ss -tulpn
```

### Opciones utilizadas

| Opción | Significado |
|---|---|
| `-t` | Muestra conexiones TCP |
| `-u` | Muestra conexiones UDP |
| `-l` | Muestra puertos en escucha |
| `-p` | Muestra el proceso asociado |
| `-n` | Muestra los números sin resolver nombres |

### Ejemplo

```text
LISTEN 0 80 0.0.0.0:3306 0.0.0.0:* users:(("mariadbd"...))
```

En este ejemplo, el servicio de MariaDB está escuchando en el puerto `3306`.

---

## 14. Diferencia entre `127.0.0.1` y `0.0.0.0`

### `127.0.0.1`

Indica que el servicio solo acepta conexiones desde el mismo servidor.

```text
127.0.0.1:3306
```

En otras palabras:

> Solo se aceptan conexiones locales.

### `0.0.0.0`

Indica que el servicio acepta conexiones a través de cualquiera de las interfaces de red disponibles.

```text
0.0.0.0:3306
```

En otras palabras:

> Se aceptan conexiones desde cualquier interfaz de red, siempre que el firewall y el servicio lo permitan.

> **Recomendación:** exponé un servicio únicamente en las interfaces y puertos que realmente necesite utilizar.
````*
