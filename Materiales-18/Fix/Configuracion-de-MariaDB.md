# Configuración de MariaDB (usando las variables)

## 1. Archivo de Configuración

Las variables de configuración principal de MariaDB se encuentran habitualmente en:

- **Ruta principal:** `/etc/my.cnf` o incluidos modularmente en la carpeta `/etc/my.cnf.d/server.cnf` dentro de la sección `[mysqld]`.

Manual de Configuracion y Variables de MariaDB

## 2. Variables Principales, Cálculo y Descripción

- **innodb_buffer_pool_size**
  - **Qué hace:** Define el tamaño del búfer de memoria RAM destinado a almacenar en caché los datos de las tablas y los índices de InnoDB. Es el parámetro más crítico para el rendimiento.
  - **Cómo calcularlo:** En un servidor dedicado exclusivamente a la base de datos, se recomienda asignar entre el 60% y el 80% de la memoria RAM total disponible del sistema operativo. Por ejemplo, en un servidor con 16 GB de RAM, un valor óptimo sería 10G o 12G.

- **max_connections**
  - **Qué hace:** Establece el número máximo de conexiones concurrentes permitidas hacia el servidor de base de datos.
  - **Cómo calcularlo:** Depende directamente de la capacidad del servidor (RAM y CPU) y del consumo promedio por conexión. Se calcula estimando cuántas conexiones simultáneas abiertas soportan los recursos sin agotar la memoria. Un valor común para aplicaciones medianas/grandes ronda entre 500 y 2000.

- **innodb_log_file_size**
  - **Qué hace:** Define el tamaño de cada archivo de registro de transacciones (redo log) en el grupo de logs. Archivos más grandes mejoran el rendimiento de escritura, pero incrementan el tiempo de recuperación ante un reinicio abrupto.
  - **Cómo calcularlo:** Suele dimensionarse en función de la actividad de escritura de la aplicación, asignando típicamente entre el 25% y el 100% del tamaño del innodb_buffer_pool_size dividido entre el número de archivos de log (por ejemplo, 512M o 1G por archivo).

- **innodb_flush_log_at_trx_commit**
  - **Qué hace:** Controla el equilibrio entre la seguridad/durabilidad de las transacciones (ACID) y la velocidad de escritura en disco.
    - **1:** El log se escribe y se descarga al disco en cada confirmación (commit). Máxima seguridad frente a caídas de energía.
    - **2:** El log se escribe en el archivo en cada commit, pero se descarga al disco una vez por segundo. Mayor rendimiento de escritura con un riesgo mínimo de pérdida de datos de hasta un segundo.

- **innodb_flush_method**
  - **Qué hace:** Define el método utilizado para leer y escribir datos en los archivos de almacenamiento de InnoDB, evitando la sobrecarga del sistema operativo.
  - **Cómo calcularlo:** En sistemas Linux basados en POSIX, se recomienda fuertemente utilizar O_DIRECT para evitar el doble búfer (evitando que los datos se almacenen tanto en la memoria caché de InnoDB como en la caché de páginas del sistema operativo).

- **bind-address**
  - **Qué hace:** Indica la dirección IP de red en la que MariaDB escuchará peticiones entrantes.
  - **Cómo calcularlo:** Si la base de datos debe recibir conexiones externas (por ejemplo, desde servidores de aplicación separados), se configura como 0.0.0.0 (todas las interfaces) o se especifica la IP privada exacta del nodo para mayor seguridad. Si es un entorno local estricto, se utiliza 127.0.0.1.

Manual de Configuracion y Variables de MariaDB

- **slow_query_log y long_query_time**
  - **Qué hace:** Permiten registrar automáticamente aquellas consultas SQL cuyo tiempo de ejecución supere el límite establecido, facilitando las tareas de optimización e indexación.
  - **Cómo calcularlo:** Habilitar slow_query_log = 1 y establecer long_query_time en un valor razonable (por ejemplo, 2.0 segundos o 1.0 segundo) para capturar consultas ineficientes sin saturar el disco con registros innecesarios.

- **max_allowed_packet**

Se incluye una lista adicional con 6 variables secundarias pero muy útiles en la configuración de MariaDB:

- **max_allowed_packet**
  - **Ubicación:** `/etc/my.cnf` o `/etc/my.cnf.d/server.cnf` en la sección `[mysqld]`.
  - **Qué hace:** Define el tamaño máximo permitido para un paquete de datos, una consulta SQL o un resultado enviado por la red. Es fundamental aumentarlo si se realizan inserciones masivas de datos o se almacenan objetos grandes (como archivos o imágenes BLOB).
  - **Cómo calcularlo/configurarlo:** Suele configurarse entre 64M y 512M según el tamaño de los paquetes que la aplicación necesite transmitir (por defecto suele ser muy pequeño, de 16M o menos).

- **innodb_file_per_table**
  - **Ubicación:** `/etc/my.cnf` o `/etc/my.cnf.d/server.cnf` en la sección `[mysqld]`.
  - **Qué hace:** Determina si cada tabla de InnoDB almacena sus datos y su índice en su propio archivo independiente (.ibd) dentro del directorio de la base de datos, en lugar de compartir un único archivo global (ibdata1).
  - **Cómo calcularlo/configurarlo:** Se recomienda configurarlo siempre en ON (activado). Facilita enormemente las tareas de mantenimiento, como la recuperación de espacio en disco al eliminar tablas (OPTIMIZE TABLE) y respaldos granulares.

- **wait_timeout e interactive_timeout**
  - **Ubicación:** `/etc/my.cnf` o `/etc/my.cnf.d/server.cnf` en la sección `[mysqld]`.
  - **Qué hace:** Especifican el tiempo (en segundos) que el servidor espera de inactividad en una conexión no interactiva (conexiones de aplicaciones o scripts) e interactiva (consolas o terminales) antes de cerrarla automáticamente.
  - **Cómo calcularlo/configurarlo:** En entornos web o de alta concurrencia, es conveniente reducirlos (por ejemplo, a 600 o 28800 segundos) para liberar rápidamente los recursos de conexiones fantasma o colgadas.

- **skip-name-resolve**
  - **Ubicación:** `/etc/my.cnf` o `/etc/my.cnf.d/server.cnf` en la sección `[mysqld]`.
  - **Qué hace:** Desactiva la resolución de nombres de host inversos (DNS) para las conexiones entrantes. Cuando un cliente se conecta, MariaDB por defecto intenta resolver la IP a un nombre de host.
  - **Cómo calcularlo/configurarlo:** Activar esta opción (añadiendo skip-name-resolve) acelera significativamente el tiempo de establecimiento de las conexiones y evita bloqueos si el servidor DNS externo presenta latencia o fallos. (Nota: al activarla, los permisos de usuario en la base de datos deben configurarse usando direcciones IP exactas o comodines de IP, no nombres de host).

- **back_log**
  - **Ubicación:** `/etc/my.cnf` o `/etc/my.cnf.d/server.cnf` en la sección `[mysqld]`.
  - **Qué hace:** Controla cuántas solicitudes de conexión pendientes puede acumular la pila de red de MariaDB en un breve instante mientras el servidor principal está ocupado atendiendo otras conexiones.
  - **Cómo calcularlo/configurarlo:** Si la aplicación web experimenta picos muy altos de tráfico simultáneo y errores de "Too many connections" a nivel de red, se recomienda elevar este valor (por ejemplo, a 500 o 900), siempre ajustando también el límite a nivel del sistema operativo (somaxconn).

- **character-set-server y collation-server**
  - **Ubicación:** `/etc/my.cnf` o `/etc/my.cnf.d/server.cnf` en la sección `[mysqld]`.
  - **Qué hace:** Establecen el juego de caracteres y la intercalación (cotejamiento) predeterminados a nivel de servidor para todas las bases de datos y tablas nuevas que se creen.
  - **Cómo calcularlo/configurarlo:** En la actualidad, el estándar recomendado para evitar problemas con acentos, eñes y caracteres especiales globales es configurar character-set-server = utf8mb4 y collation-server = utf8mb4_unicode_ci.
