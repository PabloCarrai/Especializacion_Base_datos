# Replicación de MariaDB con Galera Cluster

## 1. Introducción

En una instalación tradicional de MariaDB tenemos normalmente un servidor:

```text
Aplicación
      |
      v
MariaDB
```

Si ese servidor deja de funcionar, la aplicación puede quedar sin acceso a la base de datos. Una alternativa es utilizar varios servidores MariaDB que trabajen coordinadamente formando un clúster.

```text
                +------------------+
                |    Aplicación    |
                +--------+---------+
                         |
                +------------------+
                |  Galera Cluster  |
                +--------+---------+
                        /|\
                       / | \
         +-------------+ +-------------+ +-------------+
         |   Nodo 1    | |   Nodo 2    | |   Nodo 3    |
         |   MariaDB   | |   MariaDB   | |   MariaDB   |
         +-------------+ +-------------+ +-------------+
```

La idea fundamental es que los nodos mantienen una visión coordinada de los datos y pueden participar en el servicio de la base de datos.

---

## 2. ¿Qué es Galera?

Galera es una tecnología de replicación **síncrona** y **multi-primary** para bases de datos compatibles con MySQL/MariaDB.

Esto significa que, conceptualmente:

- existen varios nodos;
- cada nodo posee una copia de los datos;
- los nodos se comunican entre sí;
- una modificación realizada en un nodo se replica hacia los demás;
- los nodos pueden aceptar operaciones de escritura, según cómo esté configurada la aplicación;
- Galera coordina las transacciones entre los miembros del clúster.

Es importante distinguir:

$$\text{Servidor MariaDB} + \text{Galera} = \text{Nodo de un clúster}$$

Galera no reemplaza a MariaDB. Es una capa de replicación y coordinación que trabaja junto con MariaDB.

---

## 3. ¿Qué problema intenta resolver?

Supongamos que tenemos:

```text
Aplicación
    |
    v
+-------------+
|   MariaDB   |
|  Servidor   |
+-------------+
```

Si el servidor falla:

```text
Aplicación
    |
    v
+-------------+
|    FALLA    |
+-------------+
```

Tenemos un punto único de falla. Con tres nodos:

```text
            Aplicación
                |
        +-------+-------+
        |       |       |
     +-----+ +-----+ +-----+
     |Nodo1| |Nodo2| |Nodo3|
     +-----+ +-----+ +-----+
```

podemos diseñar una arquitectura donde la caída de un nodo no implique necesariamente la caída del servicio. Pero hay una cuestión fundamental: **tener tres servidores no significa automáticamente tener alta disponibilidad.** Hay que comprender cómo Galera decide qué nodos forman parte del clúster válido.

---

## 4. ¿Qué es un nodo?

Un nodo es una instancia de MariaDB que participa en el clúster Galera.

Por ejemplo:

- **Nodo 1** — IP: `192.168.56.11`
- **Nodo 2** — IP: `192.168.56.12`
- **Nodo 3** — IP: `192.168.56.13`

Los tres forman: `galera_prod_cluster`

Cada nodo tiene:
- MariaDB
- Configuración Galera
- Dirección IP
- Nombre de nodo
- Acceso a los demás nodos
- Copia de los datos

---

## 5. ¿Los nodos son maestro y esclavos?

Esta es una de las primeras ideas que hay que eliminar.

En el modelo tradicional de replicación tenemos:

```text
   MASTER
     |
     +---> SLAVE 1
     +---> SLAVE 2
     +---> SLAVE 3
```

Es un modelo principalmente *source/replica*. Galera utiliza otro concepto:

```text
  Nodo 1
    ↕
  Nodo 2
    ↕
  Nodo 3
```

Los nodos son **pares** dentro del clúster. Por eso se habla de **multi-primary**.

Aunque esto no significa que sea buena práctica enviar indiscriminadamente cualquier tipo de escritura a cualquier nodo desde cualquier aplicación. La arquitectura de acceso debe diseñarse cuidadosamente.

---

## 6. Replicación síncrona

Esta es una de las ideas más importantes de Galera.

Supongamos que ejecutamos:

```sql
INSERT INTO empleados (nombre, apellido) VALUES ('Juan', 'Pérez');
```

en Nodo 1. No debemos imaginar simplemente:

```text
Nodo 1 ---> copiar INSERT ---> Nodo 2 ---> Nodo 3
```

Galera trabaja a nivel de **transacciones** y **conjuntos de escritura** (*write sets*).

Conceptualemente:

```text
        TRANSACCIÓN
             |
          Nodo 1
             |
  Write Set / coordinación
            / \
           /   \
       Nodo 2   Nodo 3
```

Los nodos coordinan la aplicación de la transacción. Por eso Galera se diferencia de una replicación asíncrona tradicional.

---

## 7. ¿Qué significa "síncrona"?

En términos simplificados:

### Replicación asíncrona
El servidor principal puede continuar mientras las réplicas reciben posteriormente los cambios.

```text
Nodo principal --- (cambio) ---> continúa trabajando
      ... después ...
Nodo réplica (puede existir un retraso)
```

### Galera
La transacción se coordina entre los nodos participantes antes de considerarse confirmada de la forma habitual.

```text
Nodo 1 --+
Nodo 2 --+-- (coordinación) ---> COMMIT
Nodo 3 --+
```

Esto proporciona una fuerte consistencia entre los miembros, aunque introduce dependencia de la red y de la coordinación entre nodos.

---

## 8. Un concepto fundamental: Write Set

Galera no necesita enviar simplemente "el SQL original" a los otros nodos.

Una forma útil de pensarlo es:

**Write Set:** Es el conjunto de cambios producido por una transacción que Galera debe replicar.

Por ejemplo:

```sql
UPDATE empleados SET sueldo = 2000000 WHERE id = 1254;
```

Galera determina los cambios que forman parte de esa transacción y los utiliza para la replicación y certificación.

---

## 9. Certificación de transacciones

Uno de los mecanismos fundamentales de Galera es la certificación.

Supongamos:

```text
Nodo 1                         Nodo 2
  |                              |
  |-- UPDATE empleado 10 ------->|
  |                              |
```

¿Qué ocurre si dos nodos modifican simultáneamente datos que entran en conflicto? Galera necesita determinar si ambas transacciones pueden aplicarse. La certificación permite detectar conflictos.

Conceptualemente:

```text
       Transacción
            |
        Write Set
            |
      Certificación
       /         \
      /           \
(compatible)    (conflicto)
     |               |
  aplicar       abortar/reintentar
```

Por eso Galera puede funcionar como sistema *multi-primary* sin simplemente permitir que cualquier conflicto quede oculto.

---

## 10. ¿Qué pasa si dos nodos modifican el mismo registro?

Por ejemplo:

- **Nodo 1:** `UPDATE cuentas SET saldo = saldo - 100 WHERE id = 10;`
- **Nodo 2 (simultáneamente):** `UPDATE cuentas SET saldo = saldo - 200 WHERE id = 10;`

Galera debe determinar cómo se relacionan esas transacciones. Si existe conflicto de certificación, una de ellas puede ser abortada.

Esto es muy importante para entender: **Multi-primary no significa "todos pueden modificar cualquier cosa simultáneamente sin consecuencias".** La aplicación debe diseñarse teniendo en cuenta posibles conflictos.

---

## 11. Estado de los nodos

Un nodo no está simplemente `ENCENDIDO / APAGADO`. Galera mantiene diferentes estados. Uno de los más importantes es:

- **Synced:** Significa que el nodo está sincronizado con el clúster y puede participar normalmente.

Otros estados que podemos encontrar son: `Joining`, `Joined`, `Synced`, `Donor`, `Desynced`.

Y durante determinados procesos pueden aparecer otros estados relacionados con la incorporación o recuperación. Pero podemos obviar el resto de los estados inicialmente y simplificar:

```text
           Nodo
            |
      +-----+-----+
      |           |
sincronizado  sincronizando
```

---

## 12. Synced

Cuando vemos un nodo en estado `Synced`, podemos pensar: *"Este nodo está sincronizado con el estado del clúster."*

Por ejemplo:
- Node 1: `Synced`
- Node 2: `Synced`
- Node 3: `Synced`

Tenemos un clúster funcionando normalmente.

---

## 13. ¿Qué ocurre si un nodo se desconecta?

Supongamos:
- Nodo 1: `✓`
- Nodo 2: `✓`
- Nodo 3: `✓`

Y Nodo 3 falla:
- Nodo 1: `✓`
- Nodo 2: `✓`
- Nodo 3: `✗`

Los nodos restantes pueden continuar formando el componente primario si mantienen el quórum necesario. Cuando Nodo 3 vuelve, surge la pregunta: *¿Tengo los datos actualizados?*

Galera debe determinar cuánto le falta. Ahí aparecen dos mecanismos fundamentales: **IST** y **SST**.

---

## 14. IST — Incremental State Transfer

**IST** significa *Incremental State Transfer*.

Supongamos que Nodo 3 estuvo desconectado durante un período corto. Mientras estuvo fuera:

```text
Nodo 1 / Nodo 2
  + cambios A
  + cambios B
  + cambios C
```

Nodo 3 necesita solamente: $A + B + C$. Si esos cambios todavía están disponibles para ser enviados, puede realizarse un **IST**. Es decir, una transferencia incremental de los cambios que perdió.

Conceptualmente:

```text
Nodo 3 (faltan 500 transacciones)
  |
  v
Nodo donante --- (cambios faltantes) ---> Nodo 3
```

Es notablemente más rápido que reconstruir toda la base.

---

## 15. SST — State Snapshot Transfer

¿Qué pasa si Nodo 3 estuvo desconectado demasiado tiempo? Puede ocurrir que ya no estén disponibles todos los cambios necesarios para hacer un IST.

Entonces puede ser necesario un **SST** (*State Snapshot Transfer*). El nodo recibe una copia completa del estado necesario para ponerse al día.

Conceptualmente:

```text
Nodo donante --- (estado completo) ---> Nodo nuevo
```

Esto puede implicar una transferencia mucho más grande.

---

## 16. IST vs SST

Una comparación muy útil:

| Característica | IST | SST |
| :--- | :--- | :--- |
| **Tipo** | Incremental | Completo |
| **Transfiere** | Cambios faltantes | Estado completo |
| **Cantidad de datos** | Menor | Mayor |
| **Velocidad** | Generalmente mayor | Generalmente menor |
| **Requiere estado disponible** | Sí | No de la misma manera |
| **Uso típico** | Nodo desconectado poco tiempo | Nodo muy desactualizado/nuevo |

La idea fundamental: IST intenta enviar solamente lo que falta. SST reconstruye el estado del nodo.

---

## 17. ¿Qué es un Donor?

Cuando un nodo necesita sincronizarse, otro nodo puede proporcionar los datos. Ese nodo es denominado **Donor**.

Por ejemplo:
- Nodo 1: `Synced`
- Nodo 2: `Synced`
- Nodo 3: `Joining`

Podríamos tener:

```text
Nodo 1 (DONOR) ---> Nodo 3
```

Nodo 1 proporciona a Nodo 3 los datos necesarios para sincronizarse.

---

## 18. El concepto de Cluster

Ahora podemos entender que un clúster no es simplemente 3 servidores. Es un conjunto de nodos que:

- se conocen;
- se comunican;
- comparten estado;
- participan en un protocolo de coordinación;
- mantienen una visión común del grupo.

Por ejemplo:

```text
       GALERA CLUSTER
       +---------------+
       |    Nodo 1     |
       |    Synced     |
       +-------+-------+
               |
       +-------+-------+
       |               |
+--------------+ +--------------+
|    Nodo 2    | |    Nodo 3    |
|    Synced    | |    Synced    |
+--------------+ +--------------+
```

---

## 19. ¿Qué es el Primary Component?

Este concepto es fundamental. Galera necesita saber qué conjunto de nodos constituye el componente válido del clúster. Se denomina **Primary Component**.

Por ejemplo, con tres nodos conectados:

```text
      1
     / \
    /   \
   2-----3
```

Tenemos un componente primario de tres nodos.

---

## 20. ¿Qué es el quórum?

El quórum evita que dos grupos separados de servidores crean simultáneamente que son el clúster válido.

Supongamos tres nodos: Nodo 1, Nodo 2, Nodo 3. Si Nodo 3 se desconecta:

```text
Nodo 1 ---- Nodo 2       Nodo 3
```

Tenemos: 2 nodos vs. 1 nodo. Los dos nodos tienen mayoría. Por eso pueden continuar como componente primario.

---

## 21. ¿Por qué se recomienda una cantidad impar?

- **3 nodos:** La mayoría es 2. Si perdemos uno ($2\checkmark, 1\times$), seguimos teniendo mayoría.
- **5 nodos:** La mayoría es 3. Podemos perder dos ($3\checkmark, 2\times$) y mantener quórum.

---

## 22. ¿Qué ocurre con dos nodos?

Con dos nodos:

```text
Nodo 1     X     Nodo 2
```

Si se rompe la comunicación, cada uno tiene 1 de 2. Ninguno posee mayoría. Esto puede provocar que el clúster no pueda continuar normalmente como componente primario. Por eso una arquitectura Galera de producción suele considerar tres o más miembros, o mecanismos adicionales como un tercer voto.

---

## 23. ¿Qué es Split Brain?

*"Split brain"* significa, conceptualmente, que un sistema distribuido se divide en grupos que podrían intentar actuar independientemente.

Por ejemplo:

```text
Nodo 1 ---- Nodo 2
       X
     Nodo 3
```

Si ambos grupos pudieran seguir aceptando operaciones independientemente, podríamos terminar con estados incompatibles. Galera utiliza el mecanismo de quórum para evitar que un componente sin mayoría continúe funcionando como componente primario. Esta es una de las razones por las que la comunicación entre nodos es tan importante.

---

## 24. La red es crítica

En Galera la red no es simplemente un medio para que los administradores entren por SSH. Es parte fundamental del funcionamiento del clúster. Los nodos necesitan comunicarse para:

- intercambiar información;
- coordinar transacciones;
- detectar miembros;
- mantener el estado del grupo;
- realizar IST;
- realizar SST.

Por eso conviene separar conceptualmente:

```text
Red de usuarios --------> Aplicaciones
Red de cluster --------> Nodo 1 / Nodo 2 / Nodo 3
```

En determinados diseños puede utilizarse una red específica para el tráfico de Galera.

---

## 25. Latencia

La latencia de red es especialmente importante en Galera.

Supongamos:
- Servidor A (Argentina) -- `2 ms` --> Servidor B (Argentina)
- Servidor A (Argentina) -- `150 ms` --> Servidor B (Europa)

El segundo escenario puede tener un impacto considerablemente mayor sobre el comportamiento de las transacciones. Por eso, en un diseño Galera, la ubicación geográfica de los nodos importa.

---

## 26. Galera no es un backup

Esta distinción debe quedar muy clara. Si tenemos Nodo 1, Nodo 2 y Nodo 3, y alguien ejecuta accidentalmente:

```sql
DROP DATABASE produccion;
```

el problema puede propagarse al clúster. Por lo tanto:

$$\text{Replicación} \neq \text{Backup}$$

Galera proporciona disponibilidad + replicación + consistencia distribuida, pero necesitamos además backups + recuperación ante desastres.

---

## 27. Galera tampoco elimina la necesidad de HAProxy

Supongamos:

```text
Aplicación ---> Nodo 1
```

Aunque tengamos Nodo 1, Nodo 2 y Nodo 3, si la aplicación apunta exclusivamente a Nodo 1 y éste falla, la aplicación puede quedar sin servicio.

Por eso suele aparecer otro componente:

```text
         Aplicación
              |
           HAProxy
          /   |   \
   Nodo 1   Nodo 2   Nodo 3
```

HAProxy puede distribuir o dirigir las conexiones hacia los nodos disponibles según la arquitectura. Esto permite separar dos problemas: Galera (mantener los datos sincronizados) y HAProxy (dirigir las conexiones).

---

## 28. ¿Qué pasa cuando un nodo falla?

Supongamos: Nodo 1 (`✓`), Nodo 2 (`✓`), Nodo 3 (`✓`). Nodo 2 falla (`✗`).

El clúster queda como Primary Component con Nodo 1 y Nodo 3. Nodo 2 queda fuera. Cuando vuelve:

```text
Nodo 2 ---> Joining ---> ¿IST? ---> (sí: IST / no: SST) ---> Synced
```

Este flujo es fundamental para entender la recuperación.

---

## 29. ¿Qué sucede con una transacción mientras un nodo está caído?

Supongamos que Nodo 1 (`✓`) y Nodo 2 (`✓`) continúan trabajando como componente primario mientras Nodo 3 (`✗`) está caído. Se ejecutan: `INSERT A`, `UPDATE B`, `DELETE C`, `INSERT D`.

Nodo 3 no recibe esos cambios mientras está desconectado. Cuando vuelve, debe recuperar los cambios:

```text
Nodo 3 ---> "¿Qué me falta?" ---> IST ---> Synced
```

Esto explica por qué el nodo desconectado no necesariamente necesita recibir una copia completa de la base.

---

## 30. ¿Todos los nodos tienen los mismos datos?

La intención del clúster es que los nodos sincronizados mantengan un estado de datos consistente.

Por ejemplo:
- Nodo 1: Clientes = 1.000.000
- Nodo 2: Clientes = 1.000.000
- Nodo 3: Clientes = 1.000.000

Pero durante procesos como `Joining`, `IST` o `SST`, puede existir temporalmente un nodo que todavía no esté sincronizado. Por eso, `Synced` es un estado particularmente importante.

---

## 31. El flujo completo

Podemos resumir una operación de escritura así:

```text
             CLIENTE
                |
         +--------------+
         |    Nodo 1    |
         +--------------+
                |
           Transacción
                |
            Write Set
                |
          Certificación
                |
         +------+------+
         |             |
      Nodo 2        Nodo 3
         |             |
         +------+------+
                |
     Transacción confirmada
```

Este esquema es una simplificación didáctica, pero ayuda a entender la arquitectura.

---

## 32. Los conceptos que debemos dominar

- **Nodo:** Una instancia MariaDB participante del clúster.
- **Cluster:** Conjunto coordinado de nodos.
- **Multi-primary:** Varios nodos pueden actuar como primarios para operaciones de escritura, sujeto a las reglas y al diseño de la aplicación.
- **Write Set:** Conjunto de cambios que representa una transacción para su replicación.
- **Certificación:** Mecanismo utilizado para detectar conflictos entre transacciones.
- **Synced:** Nodo sincronizado con el clúster.
- **Donor:** Nodo que proporciona estado a otro nodo durante una transferencia.
- **IST:** Transferencia incremental de los cambios que faltan.
- **SST:** Transferencia completa del estado necesario para sincronizar un nodo.
- **Primary Component:** Componente del clúster que posee el estado primario válido.
- **Quórum:** Cantidad suficiente de miembros/votos para mantener el componente primario.

---

## Conclusiones

Galera no es "tres bases independientes". No debemos imaginar:

```text
Base A    Base B    Base C
```

como tres bases independientes. La idea es:

```text
        CLUSTER
           |
   +-------+-------+
   |       |       |
Nodo 1  Nodo 2  Nodo 3
```

Los tres forman parte de un sistema distribuido. Esto introduce ventajas, pero también complejidad.

### Ventajas y costos

#### Ventajas
- Alta disponibilidad.
- Múltiples nodos.
- Replicación síncrona.
- Consistencia fuerte entre nodos sincronizados.
- Recuperación de nodos.
- Posibilidad de múltiples primarios.
- Escenario favorable para evitar un único servidor como punto de falla.

#### Costos / consideraciones
- Mayor complejidad.
- Dependencia de la red.
- Latencia.
- Necesidad de comprender quórum.
- Posibles conflictos de certificación.
- Necesidad de diseñar correctamente las escrituras.
- Requiere monitoreo.
- No reemplaza los backups.
- SST puede ser costoso en tiempo y tráfico.
- La arquitectura de conexión de las aplicaciones es importante.