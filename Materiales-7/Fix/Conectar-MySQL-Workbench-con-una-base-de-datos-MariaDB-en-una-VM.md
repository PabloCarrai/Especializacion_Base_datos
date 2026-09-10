# Conectar MySQL Workbench con una base de datos MariaDB en una VM

> **Importante:** MySQL Workbench es el cliente instalado en Windows. La base de datos está funcionando en MariaDB dentro de una máquina virtual Ubuntu (VirtualBox).

---

## 1. Verificar que MariaDB esté funcionando

Ingresar a Ubuntu y ejecutar:

```bash
sudo systemctl status mariadb

```

Si el servidor está funcionando debería aparecer algo similar a:

```text
Active: active (running)

```

Si no está iniciado:

```bash
sudo systemctl start mariadb

```

Para que MariaDB se inicie automáticamente al arrancar Ubuntu:

```bash
sudo systemctl enable mariadb

```

---

## 2. Verificar la dirección IP de Ubuntu

Desde Ubuntu ejecutar:

```bash
ip addr
# o también:
ip a
# o bien:
hostname -I

```

Por ejemplo, podríamos obtener:

```text
192.168.56.11

```

*Esta dirección será utilizada posteriormente por MySQL Workbench. La dirección IP exacta dependerá de la configuración de red de VirtualBox.*

---

## 3. Configuración de red de VirtualBox

Para que Windows pueda comunicarse con Ubuntu, ambas máquinas deben tener conectividad de red. Una configuración habitual para una máquina virtual utilizada como laboratorio es utilizar:

* **Adaptador puente (Bridged Adapter)**

---

## 4. Probar la comunicación desde Windows

Antes de intentar conectar Workbench debemos comprobar que Windows puede comunicarse con Ubuntu.

1. Abrir **CMD** o **PowerShell** en Windows.
2. Ejecutar:
```powershell
ping 192.168.56.11

```



Si existe conectividad deberíamos obtener respuestas similares a:

```text
Respuesta desde 192.168.56.11: bytes=32 tiempo<1ms TTL=64

```

> **Nota importante:** Que el servidor MariaDB esté funcionando no significa necesariamente que sea accesible desde Windows. Primero debe existir conectividad de red mediante `ping`.

---

## 5. Verificar el puerto de MariaDB

MariaDB normalmente utiliza el puerto **3306**. Desde Ubuntu podemos comprobar si está escuchando:

```bash
sudo ss -lntp | grep 3306

```

Podemos obtener algo similar a:

```text
LISTEN 0 80 0.0.0.0:3306

```

o bien:

```text
LISTEN 0 80 192.168.56.11:3306

```

---

## 6. Configuración de MariaDB para aceptar conexiones remotas

Por seguridad, MariaDB viene configurado por defecto para aceptar conexiones solamente desde el propio servidor. Debemos comprobar la configuración de `bind-address`.

1. Consultar el archivo de configuración:
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

```


2. Buscar la línea `bind-address` y modificarla para permitir conexiones de red (en entorno de laboratorio):
```text
bind-address = 0.0.0.0

```


*Esto permite que MariaDB escuche conexiones en todas las interfaces de red.*

> **Advertencia:** En un servidor real de producción no conviene habilitar indiscriminadamente conexiones desde cualquier red. En un entorno de laboratorio debemos limitar el acceso mediante la red y el firewall.

3. Después de modificar la configuración, reiniciar el servicio:
```bash
sudo systemctl restart mariadb

```


4. Comprobar nuevamente:
```bash
sudo ss -lntp | grep 3306

```



---

## 7. Crear un usuario para la conexión

No es recomendable utilizar el usuario `root` para conectarse remotamente desde Workbench.

1. Ingresar a MariaDB:
```bash
sudo mariadb

```


2. Crear un usuario para el laboratorio:
```sql
CREATE USER 'alumno'@'192.168.56.%' IDENTIFIED BY 'ClaveSegura123!';

```


3. Otorgarle permisos sobre nuestra base de datos de práctica:
```sql
GRANT ALL PRIVILEGES ON empresa_indices.* TO 'alumno'@'192.168.56.%';

```


4. Aplicar los cambios y salir:
```sql
FLUSH PRIVILEGES;
EXIT;

```



> **¿Qué significa `'alumno'@'192.168.56.%'`?**
> * **Usuario:** `alumno`
> * **Origen permitido:** Cualquier dirección IP que comience con `192.168.56.x` (por ejemplo: `192.168.56.1`, `192.168.56.10`, `192.168.56.11`, `192.168.56.20`).
> * Esto es preferible en laboratorios frente a utilizar `'alumno'@'%'`, ya que este último permitiría conexiones desde cualquier dirección de internet o red externa que llegue al servidor.
> 
> 

---

## 8. Verificar el usuario

Podemos comprobar que el usuario se creó correctamente ejecutando:

```sql
SELECT User, Host FROM mysql.user WHERE User = 'alumno';

```

Deberíamos obtener un resultado similar a:

| User | Host |
| --- | --- |
| alumno | 192.168.56.% |

---

## 9. Configuración del firewall de Ubuntu

Si tenemos habilitado **UFW**, debemos permitir el acceso al puerto `3306` desde nuestra red de laboratorio.

1. Comprobar el estado:
```bash
sudo ufw status

```


2. Permitir tráfico a MariaDB desde la red `192.168.56.0/24`:
```bash
sudo ufw allow from 192.168.56.0/24 to any port 3306 proto tcp

```


3. Comprobar nuevamente:
```bash
sudo ufw status

```


Deberíamos encontrar una regla similar a:
```text
3306/tcp ALLOW 192.168.56.0/24

```



---

## 10. Probar el puerto desde Windows

Antes de utilizar Workbench podemos comprobar si Windows puede alcanzar el puerto `3306`.

1. Abrir **PowerShell** en Windows y ejecutar:
```powershell
Test-NetConnection 192.168.56.11 -Port 3306

```


2. Si todo funciona correctamente, verás:
```text
TcpTestSucceeded : True

```


*Si aparece `False`, todavía existe un problema de red, de firewall o de configuración en MariaDB.*

---

## 11. Crear la conexión en MySQL Workbench

1. Abrir **MySQL Workbench** en Windows.
2. En la pantalla principal seleccionar **MySQL Connections** y hacer clic en el botón **`+`**.
3. Se abrirá la ventana **Setup New Connection**. Completar los siguientes campos:
* **Connection Name:** `MariaDB Ubuntu - Laboratorio`
* **Connection Method:** `Standard (TCP/IP)`
* **Hostname:** `192.168.56.11`
* **Port:** `3306`
* **Username:** `alumno`



---

## 12. Guardar la contraseña

1. Hacer clic en **Store in Vault...**.
2. Ingresar la contraseña: `ClaveSegura123!`
3. Hacer clic en **OK**.

---

## 13. Probar la conexión

1. Presionar **Test Connection**.
2. Si todo está correctamente configurado, aparecerá un mensaje indicando que la conexión fue exitosa (*Successfully connected to the MySQL server*).
3. Seleccionar **OK** y abrir la conexión.

---

## 14. Verificar la conexión desde Workbench

Una vez dentro del editor SQL, podemos ejecutar comandos de prueba:

```sql
SELECT VERSION();
SELECT USER();

```

---

## 15. Seleccionar nuestra base de datos

1. Ejecutar la selección de la base de datos de práctica:
```sql
USE empresa_indices;

```


2. Comprobar las tablas disponibles:
```sql
SHOW TABLES;

```


Deberíamos obtener la lista de tablas:
* `articulos`
* `clientes`
* `detalle_ventas`
* `productos`
* `ventas`


3. Probar una consulta de datos:
```sql
SELECT * FROM clientes;

```
