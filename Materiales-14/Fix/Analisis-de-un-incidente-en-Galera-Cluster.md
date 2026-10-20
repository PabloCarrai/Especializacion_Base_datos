# Análisis de un incidente en Galera Cluster: caída de un nodo y pérdida de comunicación

> No alcanza con comprobar si un servidor responde; hay que determinar qué nodos tienen autoridad para operar, si los datos siguen siendo consistentes y si es seguro aceptar nuevas escrituras.

---

## 1. El escenario del incidente

Supongamos que tenemos un clúster Galera de tres nodos MariaDB.

Los tres nodos tenían, antes del incidente, una base de datos sincronizada. El problema aparece cuando se interrumpe la comunicación entre ellos.

La primera pregunta que debemos plantear es:

> **¿El clúster tiene dos nodos disponibles y uno aislado, o tenemos tres servidores que podrían intentar aceptar escrituras por separado?**

La diferencia es fundamental.

Galera no toma decisiones basándose únicamente en cuántos procesos MariaDB están encendidos. Utiliza la pertenencia al componente primario (`Primary Component`), el protocolo de replicación y el quorum para determinar qué partición puede continuar operando de forma segura.

---

## 2. ¿Qué es el quorum y por qué es tan importante?

El quorum es el mecanismo que permite que Galera determine qué conjunto de nodos puede operar como componente primario.

En un clúster de tres nodos con el mismo peso de voto, la mayoría requiere **dos nodos**.

| Nodos que se comunican | ¿Hay mayoría? | Resultado esperado |
| --- | --- | --- |
| `1 + 2 + 3` | Sí | Los tres pueden operar |
| `1 + 3` | Sí | El componente formado por esos dos puede continuar |
| `1 + 2` (mientras `3` queda aislado) | Sí | El componente `1 + 2` puede continuar |
| `1` aislado de `2 + 3` | No, para el nodo 1 | El nodo 1 aislado **no debe** seguir operando como primario |

*Nota: Esto presupone una configuración normal, con tres nodos de igual peso y un componente primario establecido. La condición real debe comprobarse en el clúster; no basta con contar servidores encendidos.*

Si los tres nodos se comunican correctamente, no hay inconvenientes. Pero ¿qué sucede si se interrumpe la comunicación entre ellos?

Supongamos que el **nodo 1** deja de comunicarse con los nodos 2 y 3, aunque continúa encendido y la base de datos sigue respondiendo a las aplicaciones conectadas localmente.

Ahora tenemos dos grupos:

* **Grupo A:** nodo 1.
* **Grupo B:** nodos 2 y 3.

Si ambos grupos continuaran aceptando escrituras de manera independiente, podrían producirse modificaciones incompatibles en los datos. Por ejemplo, una misma cuenta bancaria podría recibir operaciones distintas en cada grupo.

Cuando se restableciera la comunicación, Galera tendría que resolver la situación de los cambios realizados, y algunas operaciones podrían no ser compatibles entre sí.

El quorum permite evitar este escenario: el nodo aislado, que **no tiene mayoría**, no debe continuar aceptando escrituras como si perteneciera a un componente primario válido.

Es importante aclarar que el quorum **no es un mecanismo de votación** para decidir qué dato es correcto. Su función es determinar qué componente tiene autoridad para seguir operando de forma segura.

---

### ¿Qué ocurre con el nodo aislado?

Imaginemos que el **nodo 2** pierde conectividad con los otros dos:

* El nodo 1 y el nodo 3 detectan que el nodo 2 dejó de responder al protocolo de comunicación del clúster.
* El componente formado por los nodos 1 y 3 conserva la mayoría y puede mantener el estado `Primary`.
* El nodo 2, si sigue funcionando pero no puede comunicarse con el componente primario, **debe abandonar la operación normal** como miembro aislado.
* Cuando se recupera la comunicación, el nodo 2 puede volver a incorporarse y sincronizarse con el componente válido.

Este comportamiento evita que dos grupos acepten cambios independientes y terminen con datos divergentes, situación conocida como **split-brain**.

> Un nodo puede tener MariaDB encendido, responder al ping y aceptar conexiones TCP, pero aun así **no estar habilitado** para procesar transacciones de forma segura.

---

## 3. ¿Cómo determina el administrador si puede seguir escribiendo?

No debe decidirlo mirando únicamente el balanceador, el ping o el estado del servicio `mariadb`. Hay que verificar el estado interno de Galera.

En cada nodo accesible, ejecutaría estas consultas:

```sql
SHOW GLOBAL STATUS
WHERE Variable_name IN (
    'wsrep_cluster_status',
    'wsrep_cluster_size',
    'wsrep_connected',
    'wsrep_ready',
    'wsrep_local_state_comment',
    'wsrep_cluster_state_uuid'
);

```

Estos indicadores permiten responder preguntas diferentes:

| Variable | Qué debe verificar | Interpretación |
| --- | --- | --- |
| `wsrep_cluster_status` | `Primary` | El nodo pertenece a un componente primario |
| `wsrep_cluster_size` | `2` (en el escenario) | El componente activo reconoce dos miembros |
| `wsrep_connected` | `ON` | El proveedor Galera está conectado |
| `wsrep_ready` | `ON` | El nodo está habilitado para atender consultas compatibles con su estado |
| `wsrep_local_state_comment` | `Synced` | El nodo está sincronizado |
| `wsrep_cluster_state_uuid` | UUID coherente entre miembros | Permite comprobar que comparten la identidad del estado del clúster |

La evaluación se hace en conjunto: **ninguna variable por sí sola garantiza que todo esté bien.**

En particular, un nodo que informa `wsrep_ready = ON` **no debe considerarse automáticamente seguro** para escribir si no se verificaron también su pertenencia al componente primario y su estado de sincronización.

### Resultado esperado del incidente

* **Nodo 1:** `Primary`, `wsrep_ready = ON`, `Synced`
* **Nodo 3:** `Primary`, `wsrep_ready = ON`, `Synced`
* **Nodo 2:** Aislado, no primario o no preparado

Si los nodos 1 y 3 confirman que pertenecen al mismo componente primario, están sincronizados y mantienen la replicación, el clúster puede seguir aceptando escrituras a través de ellos.

---

## 4. ¿Qué decisiones debe tomar el administrador?

Las decisiones deben seguir un orden. El objetivo inicial es preservar la integridad de los datos y, al mismo tiempo, mantener el servicio si existen condiciones seguras para hacerlo.

### Paso 1. Confirmar el alcance de la falla

Verificar si se cayó MariaDB, si falló la red de replicación o si hay un problema de firewall, switch, interfaz de red o configuración de Galera.

> Una prueba de ping exitosa no demuestra que funcionen los puertos de Galera ni que el clúster esté saludable.

### Paso 2. Verificar el componente primario

Consultar el estado `wsrep_cluster_status` y confirmar que los nodos 1 y 3 integran el mismo componente `Primary`. Comprobar también su sincronización y disponibilidad.

### Paso 3. Mantener el tráfico en los nodos saludables

Si se utiliza HAProxy, MaxScale u otro balanceador, retirar temporalmente el nodo 2 del conjunto de destinos disponibles, según el mecanismo de comprobación de salud configurado.

> No hace falta detener todo el clúster por una falla aislada si los nodos restantes mantienen el quorum.

### Paso 4. Investigar y corregir la causa

Revisar logs de MariaDB/Galera, conectividad entre nodos, puertos requeridos y errores del sistema operativo. **Evitar cambios improvisados en la configuración de quorum.**

### Paso 5. Reincorporar el nodo 2

Restaurar la comunicación y permitir que Galera determine cómo sincronizarlo. Puede realizar una transferencia incremental de estado (**IST**) o una transferencia completa (**SST**), según los datos que falten y los requisitos de la transferencia.

> La reincorporación no debe darse por terminada hasta verificar que el nodo está `Synced`, que pertenece al componente primario y que está preparado para atender tráfico.

---

## 5. ¿Qué pasa si el nodo 1 también deja de funcionar?

Supongamos que el nodo 2 continúa aislado y el nodo 1 se cae. El **nodo 3 queda solo**.

Aunque el nodo 3 tenga la base de datos y el servicio MariaDB encendidos, **no conserva por sí solo la mayoría** de los tres nodos. En condiciones normales, perderá el estado primario y no podrá continuar aceptando escrituras como si nada hubiera pasado.

### ¿Qué debe hacer el administrador?

1. Confirmar que no existe otro componente primario operativo.
2. Investigar si los otros nodos están realmente caídos o solamente aislados.
3. **No forzar** un nuevo componente primario de manera automática o indiscriminada.
4. Si es necesario recuperar el servicio sin quorum, realizar un procedimiento controlado de recuperación, seleccionando un único componente y verificando cuál contiene el estado de datos más avanzado.

> El comando de bootstrap o la configuración `pc.bootstrap` no son herramientas para resolver cualquier problema de conectividad. Usarlos sobre una partición equivocada puede generar dos clústeres independientes con datos divergentes.

Cuando en un clúster Galera de tres nodos solamente queda un nodo funcionando, ese nodo **no tiene quorum por sí solo** en una configuración normal de tres nodos con igual peso de voto.

Pero hay que distinguir dos situaciones diferentes.

---

## 6. El nodo sigue funcionando, pero los otros dos se apagaron

Supongamos que teníamos tres nodos y se produce esta situación:

* **Nodo 1:** Activo
* **Nodo 2:** Caído
* **Nodo 3:** Caído

> **Resultado:** 1 voto de los 3 necesarios para decidir la mayoría. El nodo 1 no puede mantener el quorum por sí solo en condiciones normales.

### ¿Qué ocurre entonces?

* El proceso MariaDB del nodo 1 podría continuar ejecutándose.
* El nodo puede perder el estado `Primary` y pasar a `Non-Primary`.
* Galera normalmente impedirá que procese nuevas transacciones como miembro válido del clúster.
* Las aplicaciones podrían seguir conectándose al servidor, pero eso no significa que puedan realizar escrituras correctamente.

Es decir, que MariaDB esté activo **no significa** que Galera permita seguir trabajando con normalidad.

---

## 7. ¿Y si necesito que ese único nodo siga atendiendo?

Hay una posibilidad: el administrador puede forzar manualmente al nodo superviviente a formar un nuevo componente primario mediante un procedimiento de recuperación controlada.

Por ejemplo, si el nodo 1 es el único disponible, se puede evaluar la operación de recuperación mediante:

```bash
sudo galera_new_cluster

```

Este comando es un ejemplo de recuperación de un componente nuevo en instalaciones que utilizan el script de servicio habitual de MariaDB. **No debe ejecutarse automáticamente** ante cualquier pérdida de quorum.

> ⚠️ **¡Atención!** Antes de hacerlo, el administrador debe comprobar que los otros nodos no estén operando como primarios y determinar cuál contiene el estado de datos más reciente. Si los nodos restantes todavía están funcionando en otro componente, forzar un nuevo primario puede crear dos componentes independientes y provocar **divergencias o pérdida de datos**.

Además, aunque el nodo se recupere como `Primary`, estará funcionando temporalmente como un clúster de un solo nodo. **No habrá redundancia:** si ese servidor falla, el servicio quedará interrumpido.

---

## 8. ¿Cómo verifica el administrador la situación?

En el nodo superviviente, ejecutá:

```sql
SHOW GLOBAL STATUS LIKE 'wsrep_cluster_status';
SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size';
SHOW GLOBAL STATUS LIKE 'wsrep_ready';
SHOW GLOBAL STATUS LIKE 'wsrep_local_state_comment';

```

* Si informa `Non-Primary`, tamaño `1` y `wsrep_ready = OFF`, el nodo **no está listo** para procesar consultas como miembro operativo del clúster.
* Si después de una recuperación controlada informa `Primary`, tamaño `1` y está sincronizado y listo, podrá operar, pero **sin protección frente a otra falla**.

---

## 9. La decisión del administrador

| Situación | Decisión recomendada |
| --- | --- |
| Los otros nodos se reiniciarán en breve | Investigar la falla y recuperar el clúster original. |
| Los otros nodos están aislados, pero siguen activos | **No forzar** un nuevo primario sin confirmar el estado de los otros componentes. |
| Los otros nodos se perdieron definitivamente | Evaluar una recuperación controlada desde el nodo con los datos más recientes. |
| El único nodo también podría fallar | Priorizar una copia de seguridad y un plan de recuperación antes de continuar, cuando las circunstancias lo permitan. |

> Galera prioriza la consistencia antes que permitir escrituras en cualquier nodo disponible. Si queda uno solo, no alcanza con que esté encendido: hay que recuperar el estado primario de forma segura o restablecer la mayoría.

---

## 10. Actividad

**1. Los nodos 1 y 3 informan Primary, size 2, Synced y wsrep_ready ON. El nodo 2 está aislado.**

* [ ] Continuar con 1 y 3, retirar 2 del balanceador e investigar la red
* [ ] Detener los tres nodos inmediatamente
* [ ] Forzar bootstrap en el nodo 2

**2. El nodo 3 está encendido, pero informa wsrep_cluster_status = NON_PRIMARY y wsrep_ready = OFF.**

* [ ] Habilitar escrituras manualmente
* [ ] Tratarlo como nodo no operativo para escrituras y diagnosticar el quorum
* [ ] Agregarlo al balanceador porque responde al ping

**3. Se recupera la red del nodo 2, que estuvo aislado durante varios minutos.**

* [ ] Declararlo saludable inmediatamente
* [ ] Forzar bootstrap en los tres nodos
* [ ] Verificar que se reincorpore, sincronice y quede listo antes de habilitar tráfico

---

## 11. La conclusión que deberían llevarse

Un administrador de bases de datos no debe preguntarse solamente: *«¿Cuántos servidores están encendidos?»*.

Debe responder estas cuatro preguntas:

* 🌐 **Quorum:** ¿qué componente tiene autoridad para operar?
* 🔗 **Consistencia:** ¿los nodos que siguen activos comparten un estado coherente?
* ⚡ **Disponibilidad:** ¿podemos seguir atendiendo las aplicaciones sin comprometer los datos?
* 🔄 **Recuperación:** ¿cómo reincorporamos los nodos sin perder transacciones ni crear componentes independientes?
