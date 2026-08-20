# Virtualización: Creación de un entorno de trabajo basado en Linux

## ¿Qué es la Virtualización?

En informática, la virtualización es la creación a través de software de una representación (versión virtual) de algún recurso tecnológico. Por lo tanto, este software tiene la función de simular la existencia del recurso tecnológico que se quiere virtualizar.

La virtualización es la creación de una forma virtual de un recurso de computación como:
*   Una computadora.
*   Un servidor.
*   Otro componente de hardware.
*   Un recurso de software (como un sistema operativo).

> **Ejemplo común:** Partir un disco duro durante la instalación de un sistema operativo, donde el disco duro físico se divide en múltiples discos lógicos para proveer un mejor almacenamiento y recuperación de datos.

---

## Tipos de Virtualización

La virtualización se clasifica de acuerdo al recurso creado:

| Tipo | Descripción |
| :--- | :--- |
| **Servidor** | Requiere agrupar recursos de uno o más servidores físicos y partirlos en múltiples servidores virtuales. Es el más usado. |
| **Red** | Abstracción de los recursos de red. |
| **Escritorio** | Ejecución de escritorios de forma virtual. |
| **Hardware** | Emulación o partición de componentes físicos. |
| **Software** | Virtualización de aplicaciones o entornos de software. |

### Hipervisores
Para la virtualización de servidores se utiliza una herramienta especial llamada **hipervisor**.

#### Tipo 1 (Nativos / Bare-metal)
Se conectan directamente en el hardware.
*   VMware ESXi
*   Microsoft Hyper-V
*   KVM (Kernel-based Virtual Machine)
*   Xen / Xen Project
*   Oracle VM Server

#### Tipo 2 (Hospedados)
Se ejecutan sobre un sistema operativo ajeno.
*   Oracle VM VirtualBox
*   VMware Workstation / VMware Fusion (macOS)
*   Parallels Desktop (macOS)
*   QEMU

---

## Ventajas de la Virtualización

*   **Desempeño y eficiencia:** Mejor uso de los recursos de computación existentes.
*   **Seguridad:** Las máquinas virtuales (VMs) están separadas lógicamente; un ataque o *glitch* en una no afecta a las demás.
*   **Ahorro:** Menor costo y menor necesidad de hardware físico.
*   **Tranquilidad:** Mayor confiabilidad en respaldos, recuperación ante desastres y gestión de capacidades.

---

## Gestión de Máquinas Virtuales (VMs)

### Asignación de recursos
La asignación de memoria y poder de procesamiento debe ser planeada con anticipación, basándose en el crecimiento proyectado de la red. Un espacio de almacenamiento mal asignado impacta notablemente el desempeño.
> *Nota: Las VMs que no estén en uso deben ser monitoreadas y removidas para optimizar el almacenamiento.*

### Monitoreo del desempeño
La mayoría del software de monitoreo de red no cuenta con funciones para VMs, lo que requiere herramientas separadas. Una **solución de monitoreo integrada** es fundamental para obtener el mejor desempeño.

---

## Virtualización con VirtualBox

**Oracle VM VirtualBox** es un hipervisor gratuito y de código abierto desarrollado originalmente por Innotek, adquirido por Sun Microsystems (2008) y posteriormente por Oracle Corporation (2010).

*   **Sitio web oficial:** [https://www.virtualbox.org/](https://www.virtualbox.org/)

### Características principales
*   Multiplataforma (Windows, Linux).
*   Código abierto y gratuito.
*   Soporte completo de Windows (incluyendo versiones Server).
*   Soporte de instantáneas (*snapshots*).
*   Facilidad de administración de imágenes ISO.
*   Soporte de USB y escritorio remoto.
*   Portable.

### Requisitos mínimos
*   **RAM:** 2 MB (mínimo para Win 7), 4 GB (para Win 8/10). *Recomendación: El doble de lo indicado.*
*   **Espacio en disco:** Mínimo 90 MB por VM. *Recomendación: 1 GB o más para sistemas modernos.*
*   **Procesador:** Intel o AMD con soporte de virtualización (VT-x/AMD-V). *Recomendación: Más de dos núcleos.*

---

## Instalación y Configuración

### Consideraciones previas
1.  **BIOS/UEFI:** Si no se activa la virtualización por hardware, solo podrá virtualizar sistemas de 32 bits.
2.  **64 bits:** Activando la virtualización en BIOS/UEFI, se permite la virtualización de sistemas "huésped" de 64 bits.
3.  **Controladores:** Durante la instalación, es vital seleccionar la opción de controlador de bus para habilitar la conexión con puertos USB.

### VirtualBox Extension Pack
Paquete que añade compatibilidad con:
*   USB 2.0 y 3.0.
*   Controladores de host.
*   Cámaras web.
*   Discos sólidos NVMe.
*   Encriptación de disco.

#### Funciones de las "Guest Additions"
*   **Integración del puntero:** Movimiento fluido entre host e invitado.
*   **Carpetas compartidas:** Intercambio seguro de archivos.
*   **Soporte de vídeo:** Controladores optimizados y rendimiento acelerado.
*   **Ventana completa:** Integración visual de ventanas del invitado en el escritorio del host.
*   **Portapapeles compartido:** Copiar/pegar entre sistemas.
*   **Sincronización de tiempo:** Uso del protocolo NTP.