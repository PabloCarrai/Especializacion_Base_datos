# Actualización de Repositorios e Instalación de MariaDB

Ubuntu 24.04 incluye paquetes estables de MariaDB, correspondientes a la serie **10.11 LTS**, directamente en sus repositorios oficiales.

## 1. Actualizar las listas de paquetes

Actualiza las listas de paquetes y aplica las actualizaciones disponibles:

```bash
sudo apt update && sudo apt upgrade -y
```

## 2. Instalar MariaDB en el servidor Ubuntu

Instala el servidor MariaDB mediante `apt`:

```bash
sudo apt install mariadb-server -y
```

### Verificar el estado del servicio

Comprueba que el servicio esté activo:

```bash
sudo systemctl status mariadb
```

Para habilitar MariaDB automáticamente durante el arranque del sistema:

```bash
sudo systemctl enable mariadb
```

## 3. Securización inicial

Ejecuta el script de hardening proporcionado por MariaDB para eliminar configuraciones inseguras por defecto:

```bash
sudo mariadb-secure-installation
```

Durante el proceso, utiliza las siguientes opciones recomendadas:

| Paso | Elección recomendada | Motivo |
|---|:---:|---|
| `Enter current password for root` | Presionar **Enter** | La instalación inicial no tiene una clave asignada. |
| `Switch to unix_socket authentication` | **Y** | Permite el inicio de sesión seguro de `root` únicamente mediante `sudo`. |
| `Change the root password?` | **Y** | Establece una contraseña administrativa de respaldo. |
| `Remove anonymous users?` | **Y** | Evita accesos sin autenticar. |
| `Disallow root login remotely?` | **Y** | Aplica buenas prácticas: `root` solo debe acceder localmente. |
| `Remove test database and access to it?` | **Y** | Elimina la base de datos de prueba y sus permisos asociados. |
| `Reload privilege tables now?` | **Y** | Aplica los cambios inmediatamente en memoria. |

## 4. Comprobación final

Verifica nuevamente que el servicio de MariaDB esté funcionando:

```bash
sudo systemctl status mariadb
```

Comprueba que el servicio esté habilitado durante el arranque:

```bash
sudo systemctl is-enabled mariadb
```

La salida esperada es:

```text
enabled
```
```
