# Conceptos de Puertos y Configuración del Firewall UFW en Ubuntu

Una guía completa para entender cómo funciona el tráfico de red, los puertos estándar más utilizados y cómo blindar tu sistema operativo mediante el uso de UFW (*Uncomplicated Firewall*).

---

## 1. Concepto de Puertos

Un **puerto** es un punto lógico que utiliza el sistema operativo para gestionar conexiones de red y dirigir el tráfico hacia el servicio correcto (por ejemplo, web, correo o transferencia de archivos). Cada puerto tiene un número estándar, lo que permite que los datos lleguen a la aplicación adecuada dentro de un mismo equipo.

*   **Capa de transporte:** Los puertos trabajan en la capa de transporte (`TCP`/`UDP`), haciendo posible que múltiples servicios funcionen simultáneamente sobre una misma conexión de red.
*   **Seguridad:** Por defecto, los cortafuegos (*firewalls*) suelen bloquear la mayoría de los puertos y solo dejan abiertos aquellos estrictamente necesarios, previniendo accesos no autorizados.

### Puertos más utilizados

Para facilitar su consulta rápida, aquí tienes una tabla organizada con los puertos estándar más comunes en entornos de desarrollo, sistemas y redes:

| Puerto | Protocolo / Servicio | Descripción |
| :--- | :--- | :--- |
| **20, 21** | FTP | Transferencia de archivos |
| **22** | SSH | Conexión segura remota |
| **23** | Telnet | Acceso remoto no seguro |
| **25** | SMTP | Correo electrónico |
| **53** | DNS | Resolución de nombres de dominio |
| **67, 68** | DHCP | Asignación automática de IP |
| **69** | TFTP | Transferencia simple de archivos |
| **80** | HTTP | Tráfico web sin cifrar |
| **110** | POP3 | recepción de correo |
| **123** | NTP | Sincronización de hora |
| **137-139** | NetBIOS | Redes Windows |
| **143** | IMAP | Correo electrónico |
| **161, 162** | SNMP / Trap | Monitoreo de red y notificaciones |
| **389 / 636** | LDAP / LDAPS | Servicios de directorio (seguro) |
| **443** | HTTPS | Tráfico web seguro |
| **445** | SMB/CIFS | Archivos en red Windows |
| **465 / 587** | SMTPS / SMTP Seguro | Correo cifrado |
| **514** | Syslog | Registro de eventos |
| **515** | LPD | Impresión en red |
| **989, 990** | FTPS | FTP seguro |
| **993 / 995** | IMAPS / POP3S | Correo seguro |
| **1433** | Microsoft SQL Server | Base de datos |
| **1521 / 2483 / 2484** | Oracle Database / TCPS | Base de datos |
| **2049** | NFS | Sistema de archivos en red |
| **2082, 2083** | cPanel | Panel de control web (HTTP/HTTPS) |
| **2375, 2376** | Docker API | Gestión de contenedores |
| **3000** | Dev Web | Aplicaciones web en desarrollo |
| **3306** | MySQL / MariaDB | Base de datos |
| **3389** | RDP | Escritorio remoto de Windows |
| **5432** | PostgreSQL | Base de datos |
| **5900** | VNC | Escritorio remoto |
| **5985, 5986** | WinRM | Administración remota de Windows |
| **6379** | Redis | Base de datos en memoria / Caché |
| **6667** | IRC | Chat |
| **8000, 8080, etc.** | HTTP Alternativo | Puertos proxy y desarrollo web |
| **8443** | HTTPS Alternativo | Web segura alternativa |
| **9418** | Git | Control de versiones |
| **11211** | Memcached | Sistema de almacenamiento en caché |
| **27017** | MongoDB | Base de datos NoSQL |

> **En resumen:** Los puertos permiten organizar y dirigir el tráfico de red, mientras que los firewalls controlan cuáles de ellos pueden utilizarse para proteger los sistemas.

---

## 2. Configuración Detallada del Firewall UFW en Ubuntu

Un **firewall** (o cortafuegos) es un mecanismo de seguridad que se utiliza para controlar el tráfico de red que entra y sale de un sistema. Su función principal es permitir o bloquear conexiones según un conjunto de reglas definidas, protegiendo el equipo frente a accesos no autorizados o ataques informáticos.

En los sistemas basados en Linux (como Ubuntu), existe **UFW** (*Uncomplicated Firewall*), una interfaz sencilla para administrar el firewall subyacente (`iptables` o `nftables`), pensada para configurar reglas de seguridad de forma amigable y sin comandos complejos.

### Comandos esenciales para administrar UFW

#### 1. Verificar si UFW está instalado
```bash
sudo ufw status verbose