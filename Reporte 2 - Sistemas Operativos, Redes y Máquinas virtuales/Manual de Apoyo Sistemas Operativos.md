# Manual: Instalación y coexistencia de Windows 11 y Ubuntu 26.04 en dual boot dentro de una máquina virtual (VirtualBox)

## Índice
1. Objetivo de la práctica
2. Requisitos previos
3. Especificaciones de la máquina virtual
4. Paso 1: Creación de la máquina virtual
5. Paso 2: Instalación de Windows 11
6. Paso 3: Bypass de requisitos de Windows 11
7. Paso 4: Particionado durante la instalación de Windows
8. Paso 5: Configuración inicial de Windows
9. Paso 6: Verificación del espacio libre y snapshot
10. Paso 7: Instalación de Ubuntu 26.04
11. Paso 8: Particionado manual para Ubuntu
12. Paso 9: Finalización de la instalación de Ubuntu
13. Paso 10: Verificación del menú GRUB (dual boot funcional)
14. Resumen de particiones final
15. Comandos y valores de registro utilizados
16. Problemas encontrados y soluciones
17. Notas y recomendaciones finales

---

## Objetivo de la práctica

Configurar **una sola máquina virtual en Oracle VirtualBox** que contenga **Windows 11** y **Ubuntu 26.04** instalados en el mismo disco virtual, con arranque dual mediante **GRUB**, simulando un dual boot real (como el que se haría en una PC física con dos sistemas operativos y un gestor de arranque que permite elegir cuál iniciar).

---

## Requisitos previos

- Oracle VirtualBox instalado en el equipo anfitrión (host).
- Virtualización habilitada en la BIOS/UEFI del equipo físico (VT-x / AMD-V).
- ISO de **Windows 11** (64 bits).
- ISO de **Ubuntu 26.04 Desktop** (64 bits) — archivo `ubuntu-26.04-desktop-amd64.iso`.
- Al menos **110-120 GB de espacio libre** en el disco del equipo anfitrión.
- Conexión a internet (opcional, recomendable para actualizaciones y controladores de Ubuntu).

---

## Especificaciones de la máquina virtual

| Parámetro | Valor configurado |
|---|---|
| Nombre de la VM | Maquina Practicas |
| Tipo/Versión | Other Linux (64-bit) — genérico por tener 2 SO |
| Memoria RAM | 4096 MB (4 GB) |
| Procesadores | 2 CPUs |
| Disco duro virtual | Tipo VDI, tamaño dinámico, **110 GB** (un solo disco compartido por ambos SO) |
| Memoria de video | 128 MB |
| Chipset | PIIX3 |
| Habilitar EFI | Sí |
| I/O APIC | Habilitado |
| TPM | Versión 2.0 |
| Secure Boot | Deshabilitado (requiere UEFI marcado para poder activarse; se dejó desmarcado) |
| Adaptador de red | Intel PRO/1000 MT Desktop (NAT) |
| Controlador de almacenamiento | SATA |

### Distribución de espacio planificada en el disco de 110 GB

| Sistema | Espacio asignado |
|---|---|
| Windows 11 | ~55 GB |
| Ubuntu | ~55-56 GB (espacio libre dejado durante la instalación de Windows) |

---

## Paso 1: Creación de la máquina virtual

1. Abrir Oracle VirtualBox → **Nueva**.
2. Asignar nombre: `Maquina Practicas`.
3. Tipo: **Other Linux (64-bit)**.
4. Memoria RAM: **4096 MB**.
5. Procesadores: **2**.
6. Disco duro virtual: crear uno nuevo, tipo **VDI**, tamaño **dinámico**, **110 GB**.
7. En **Configuración → Sistema → Placa base**:
   - Marcar **I/O APIC**.
   - Chipset: **PIIX3** (o ICH9 según disponibilidad).
8. En **Configuración → Sistema → Procesador**: habilitar **PAE/NX**.
9. En **Configuración → Pantalla**: 128 MB de memoria de video.
10. En **Configuración → Almacenamiento**: montar la ISO de Windows 11 en la unidad óptica virtual.

---

## Paso 2: Instalación de Windows 11

1. Iniciar la VM con la ISO de Windows 11 montada.
2. Pantalla **"Seleccionar opción de configuración"** → elegir **"Instalar Windows 11"**.
3. Marcar la casilla de aceptación (advertencia genérica de borrado, sin riesgo en disco vacío).
4. **Clave de producto** → "No tengo clave de producto".
5. Seleccionar edición de Windows (Home/Pro).
6. Aceptar términos de licencia.
7. Tipo de instalación → **"Personalizada: instalar solo Windows (avanzado)"**.

---

## Paso 3: Bypass de requisitos de Windows 11 (TPM/Secure Boot)

Al llegar al particionado, el instalador mostró el error:

> **"Este equipo no cumple actualmente los requisitos del sistema de Windows 11"**
> - El equipo debe admitir TPM 2.0.
> - El equipo debe admitir el arranque seguro.

### Solución aplicada: bypass por registro

Desde la pantalla de error, sin cerrar el instalador:

1. Presionar **Shift + F10** → abre una consola CMD.
2. Ejecutar:
   ```
   regedit
   ```
3. Navegar a:
   ```
   HKEY_LOCAL_MACHINE\SYSTEM\Setup
   ```
4. Click derecho en `Setup` → **Nuevo → Clave** → nombrarla `LabConfig`.
5. Dentro de `LabConfig`, crear 4 valores **DWORD (32 bits)**, todos con dato `1`:

| Nombre del valor | Tipo | Dato |
|---|---|---|
| `BypassTPMCheck` | REG_DWORD | 0x00000001 (1) |
| `BypassSecureBootCheck` | REG_DWORD | 0x00000001 (1) |
| `BypassRAMCheck` | REG_DWORD | 0x00000001 (1) |
| `BypassStorageCheck` | REG_DWORD | 0x00000001 (1) |

6. Cerrar el Editor del Registro y la consola CMD (`exit`).
7. En la pantalla de error, click en **"Atrás"** y luego **"Siguiente"** de nuevo → el instalador ya no bloquea el avance.

> **Importante:** el nombre del valor debe escribirse solo, sin el signo `=` ni el número (ej. correcto: `BypassTPMCheck`; incorrecto: `BypassTPMCheck = 1`). El dato numérico se coloca aparte en el campo "Información del valor".

---

## Paso 4: Particionado durante la instalación de Windows

En la pantalla **"¿Dónde quieres instalar Windows?"**:

1. Se mostró el disco de 110 GB como "Espacio sin asignar".
2. Click en **"Nuevo"** → tamaño: **55000 MB** (~55 GB).
3. Click en **"Aplicar"** → Windows crea automáticamente:
   - Partición del sistema **EFI** (~100 MB, FAT32/NTFS).
   - Partición reservada de Microsoft (MSR, ~16 MB).
   - Partición principal de Windows (C:, el resto de los 55 GB).
4. Se seleccionó la **partición principal de Windows** (no el espacio sin asignar) y se dio clic en **"Siguiente"**.
5. Quedaron **~56 GB de "Espacio sin asignar"**, reservados para Ubuntu — **sin tocar**.
6. Pantalla "Listo para instalar" confirmó:
   - Instalar Windows 11 Home.
   - No mantener nada.
7. Click en **"Instalar"** → proceso de copia de archivos y reinicios automáticos (10-30 min).

---

## Paso 5: Configuración inicial de Windows (OOBE)

1. Región: **Guatemala**.
2. Distribución de teclado: **Latinoamericano**.
3. Nombre del dispositivo: se omitió (o se asignó uno personalizado, ej. `WIN11-DUALBOOT`).
4. **Cuenta**: se recomendó una cuenta local para evitar dependencias de internet/Microsoft, mediante el comando:

   Desde la pantalla de red o de inicio de sesión con Microsoft, presionar **Shift + F10** y ejecutar:
   ```
   oobe\bypassnro
   ```
   Esto reinicia el asistente OOBE (sin borrar la instalación ya copiada en disco) y habilita la opción "No tengo internet" / "Continuar con configuración limitada" en la pantalla de red.

   > En la práctica realizada, finalmente se optó por iniciar sesión con una **cuenta Microsoft** normal, ya que no afecta el particionado ni el dual boot.

5. Se completó el resto del asistente (privacidad, aplicaciones, etc.) hasta llegar al escritorio de Windows.

### Advertencia sobre reinicio y montaje de ISO

Durante el proceso, al reiniciar la VM con la ISO de Windows aún montada y con el orden de arranque priorizando la unidad óptica, la VM volvió a arrancar el **instalador** en vez de continuar con Windows ya instalado. Solución:

1. Apagar la VM.
2. **Configuración → Almacenamiento** → seleccionar la unidad óptica con la ISO de Windows.
3. Click derecho / ícono de disco → **"Remove Disk From Virtual Drive"** (quitar disco).
4. Aceptar y volver a iniciar la VM → arranca correctamente desde el disco duro.

---

## Paso 6: Verificación del espacio libre y snapshot

1. Dentro de Windows ya instalado: **Inicio → Administración de discos**.
2. Verificar visualmente que existan:
   - Partición del sistema EFI (~100 MB).
   - Partición C: de Windows (~53-54 GB).
   - **Espacio sin asignar** (~56 GB) — marcado en negro/rayado, sin formato.
3. Apagar la VM correctamente desde dentro de Windows (**Inicio → Apagar**).
4. En el Administrador de VirtualBox, con la VM apagada:
   - Ícono de instantáneas (cámara) → **"Tomar instantánea"**.
   - Nombre sugerido: `Windows 11 instalado - listo para Ubuntu`.

---

## Paso 7: Instalación de Ubuntu 26.04

1. Con la VM apagada, ir a **Configuración → Almacenamiento**.
2. Seleccionar la unidad óptica (vacía) → ícono de disco → **"Seleccionar/crear un disco óptico virtual"**.
3. Elegir el archivo `ubuntu-26.04-desktop-amd64.iso`.
4. Verificar en **Configuración → Sistema → Placa base** que el orden de arranque incluya la unidad óptica antes que el disco duro (o al menos habilitada).
5. Iniciar la VM → arranca el instalador de Ubuntu (logo Ubuntu → pantalla de bienvenida "Welcome to Ubuntu").
6. **Choose your language** → Español.
7. Pantalla de **Accesibilidad** → se omitió (Siguiente sin marcar nada).
8. **Disposición del teclado** → Español (latinoamericano).
9. **Conectarse a una red** → se seleccionó **"Utilizar conexión por cable"** (Ubuntu sí tiene internet vía NAT).
10. **Probar o instalar Ubuntu** → se seleccionó **"Instalar Ubuntu"**.
11. **Tipo de instalación (método)** → se dejó **"Instalación interactiva"**.
12. **Aplicaciones** → se dejó **"Selección predeterminada"** (solo lo esencial).
13. **Optimice el equipo (programas privativos)** → se marcaron ambas casillas:
    - Instalar software de terceros para gráficos y dispositivos Wi-Fi.
    - Descargar e instalar compatibilidad para más formatos multimedia.

---

## Paso 8: Particionado manual para Ubuntu

En la pantalla **"Configuración del disco" / "¿Cómo quieres instalar Ubuntu?"**, se presentaron 3 opciones:

- Instalar Ubuntu junto a Windows 11 (automático).
- Borrar disco e instalar Ubuntu.
- **Instalación manual** ← opción elegida, para tener control total del particionado y no alterar las particiones ya creadas por Windows.

### Tabla de particiones antes de particionar Ubuntu

| Dispositivo | Tipo | Tamaño | Sistema |
|---|---|---|---|
| sda1 | NTFS | 104.86 MB | Windows 11 (EFI) |
| sda2 | NTFS | 56.84 GB | Windows 11 (C:) |
| sda3 | NTFS | 831.52 MB | Recuperación de Windows |
| Espacio libre | — | 60.34 GB | (para Ubuntu) |

### Intento inicial con partición SWAP (revertido)

Se intentó crear primero una partición **swap** de 4096 MB, pero el disco (con tabla de particiones **MBR**) alcanzó el **límite de 4 particiones primarias**, impidiendo crear la partición raíz. 

**Solución:** se eliminó la partición swap creada (seleccionar `sda4` → botón **"−"**), liberando el espacio nuevamente a "Espacio libre" (60.34 GB), y se usó **todo el espacio en una sola partición raíz**, ya que Ubuntu moderno gestiona la memoria de intercambio mediante un **archivo swap (swapfile)** dentro del sistema de archivos, sin necesitar partición dedicada.

### Partición final creada

1. Seleccionar **"Espacio libre"** (60.34 GB) → botón **"+"**.
2. En la ventana "Crear partición":
   - **Tamaño**: 60335 MB (todo el espacio disponible).
   - **Utilizada como**: **Ext4**.
   - **Punto de montaje**: `/`
3. Click en **"Aceptar"**.

### Tabla de particiones final

| Dispositivo | Tipo | Punto de montaje | Tamaño | Sistema |
|---|---|---|---|---|
| sda1 | NTFS | — | 104.86 MB | Windows 11 (EFI, no modificada) |
| sda2 | NTFS | — | 56.84 GB | Windows 11 C: (no modificada) |
| sda3 | NTFS | — | 831.52 MB | Recuperación de Windows (no modificada) |
| **sda4** | **Ext4** | **/** | **60.34 GB** | **Ubuntu (nueva)** |

### Dispositivo del gestor de arranque (bootloader)

En el desplegable **"Dispositivo donde instalar el cargador de arranque"** se dejó seleccionado:
```
sda VBOX HARDDISK (118.11 GB)
```
Es decir, el **disco completo**, no una partición específica — esto es necesario para que **GRUB** se instale correctamente y pueda detectar ambos sistemas operativos.

### Pantalla de resumen ("Listo para instalar")

Confirmó:
- Tipo de instalación: Instalación manual.
- Disco de instalación: VBOX HARDDISK sda.
- Aplicaciones: Selección predeterminada.
- Cifrado del disco: Ninguno.
- Software propietario: Códecs y controladores.
- Particiones:
  - Windows 11 (sda1): No modificado.
  - sda2: No modificado.
  - sda3: No modificado.
  - Ubuntu 26.04 (sda4): Creado y formateado como ext4, utilizado para `/`.

---

## Paso 9: Finalización de la instalación de Ubuntu

1. Click en **"Instalar"** (botón verde).
2. Confirmar escritura de cambios en el disco si se solicita.
3. Copia de archivos y configuración del sistema (proceso "Configurando el sistema...", duración variable, ~30-40 min en esta práctica).
4. **Creación de cuenta de usuario** ("Cree su cuenta"):
   - Su nombre: (ej. Ana).
   - Nombre del equipo.
   - Nombre de usuario (minúsculas, sin espacios).
   - Contraseña y confirmación.
   - Mantener marcado **"Solicitar mi contraseña para acceder"**.
   - No marcar "Utilizar Active Directory".
5. **Selección de huso horario**: detectado automáticamente como `America/Guatemala` (Guatemala City).
6. Pantalla **"Finalizó la instalación"** → **"Ubuntu 26.04 está instalado y listo para usarse"**.

### Retiro de la ISO antes de reiniciar

1. **Antes de reiniciar**, ir al menú **Dispositivos → Unidades ópticas** (o Configuración → Almacenamiento) y quitar la ISO de Ubuntu de la unidad virtual.
2. Si VirtualBox muestra el aviso *"No se pudo extraer el disco óptico virtual..."*, elegir **"Forzar desmontaje"**.
3. Click en **"Reiniciar ahora"**.

---

## Paso 10: Verificación del menú GRUB (dual boot funcional)

Al reiniciar, el sistema mostró el log de arranque de systemd (normal, primera vez que arrancan los servicios de Linux) y, tras un **reinicio manual** desde **Máquina → Reiniciar** manteniendo presionada la flecha de dirección (↓) para evitar que el temporizador de GRUB avanzara automáticamente, se visualizó correctamente el menú:

```
GNU GRUB version 2.14

Ubuntu
Advanced options for Ubuntu
Memory test (mt86+x64)
Memory test (mt86+x64, serial console)
*Windows 11 (on /dev/sda1)
```

Esto confirmó que:
- **GRUB detectó e integró ambos sistemas operativos.**
- El dual boot dentro de la máquina virtual quedó **completamente funcional**.

Después de verificar el menú, se tomó una segunda instantánea (snapshot) con la VM apagada, nombrada:
```
Dual boot completo - Windows + Ubuntu
```

---

## Resumen de particiones final

| Partición | Sistema de archivos | Tamaño | Punto de montaje | Contenido |
|---|---|---|---|---|
| sda1 | FAT32/NTFS (EFI) | 104.86 MB | `/boot/efi` (compartido) | Partición EFI del sistema (usada por ambos SO) |
| sda2 | NTFS | 56.84 GB | — | Windows 11 (C:) |
| sda3 | NTFS | 831.52 MB | — | Partición de recuperación de Windows |
| sda4 | Ext4 | 60.34 GB | `/` | Ubuntu 26.04 (raíz del sistema) |

**Disco virtual total:** 110-118.11 GB (VDI, tamaño dinámico)

---

## Comandos y valores de registro utilizados

### Bypass de requisitos de Windows 11 (durante instalación)

```
regedit
```
Ruta creada: `HKEY_LOCAL_MACHINE\SYSTEM\Setup\LabConfig`

Valores DWORD creados (todos en `1`):
```
BypassTPMCheck
BypassSecureBootCheck
BypassRAMCheck
BypassStorageCheck
```

### Bypass de cuenta Microsoft obligatoria (OOBE)

```
oobe\bypassnro
```

### Comando alternativo probado para forzar cuenta local (pantalla "Desbloquea tu experiencia de Microsoft")

```
start ms-cxh:localonly
```

### Atajo de teclado usado repetidamente durante la instalación

```
Shift + F10   → abre consola CMD dentro del instalador de Windows
```

---

## Problemas encontrados y soluciones

| Problema | Causa | Solución aplicada |
|---|---|---|
| Windows 11 no cumple requisitos (TPM 2.0 / Secure Boot) | VirtualBox no emula TPM/Secure Boot por defecto | Bypass mediante clave de registro `LabConfig` (ver arriba) |
| Valor de registro mal creado (`BypassTPMCheck = 1` como nombre) | Se escribió el signo `=` y el número dentro del campo "Nombre" | Editar el valor: dejar el nombre limpio y el dato `1` en el campo correspondiente |
| Al reiniciar, volvió a aparecer el instalador de Windows | La ISO seguía montada y el orden de arranque priorizaba la unidad óptica | Apagar la VM, quitar el disco óptico virtual (Configuración → Almacenamiento → Remove Disk), reiniciar |
| Pantalla de cuenta Microsoft obligatoria en OOBE | Windows 11 fuerza cuenta online por defecto | Uso de `oobe\bypassnro` (o, alternativamente, se optó por iniciar sesión con cuenta Microsoft real sin mayor inconveniente) |
| Límite de particiones alcanzado al crear swap + raíz | Tabla de particiones **MBR** solo permite 4 particiones primarias, y ya estaban ocupadas por Windows (3) + swap (1) | Se eliminó la partición swap y se usó todo el espacio libre en una sola partición Ext4 para `/`; Ubuntu gestiona el intercambio con swapfile automático |
| Pantalla en negro tras reiniciar después de instalar Ubuntu | El menú de GRUB tiene un temporizador corto y a veces no se alcanza a capturar/interactuar a tiempo | Reiniciar desde **Máquina → Reiniciar** y mantener presionada la flecha de dirección (↓) para pausar el temporizador de GRUB |
| No extraía el disco óptico automáticamente al reiniciar tras la instalación de Ubuntu | El sistema aún tenía el archivo ISO "en uso" | Usar la opción **"Forzar desmontaje"** en el cuadro de diálogo de VirtualBox |

---

## Notas y recomendaciones finales

- **Orden de instalación:** siempre instalar primero Windows y después Ubuntu. Si se hace al revés, el instalador de Windows sobrescribe el bootloader y Ubuntu deja de aparecer en el menú de arranque.
- **Snapshots:** se recomienda tomar una instantánea después de cada hito importante (Windows instalado, Ubuntu instalado, dual boot verificado) para poder revertir cambios sin repetir toda la instalación.
- **Aislamiento de la VM:** todo lo realizado dentro de la máquina virtual (incluyendo inicios de sesión con cuentas Microsoft) queda contenido en el archivo `.vdi` y **no afecta al sistema operativo físico (host)** ni a la cuenta personal más allá de un registro de "dispositivo conectado" que puede eliminarse desde `account.microsoft.com → Dispositivos`.
- **Tamaño en disco real:** al usar un disco virtual de tipo dinámico, el archivo `.vdi` no ocupa los 110 GB completos desde el inicio; crece gradualmente según el uso real dentro de la VM.
- **Secure Boot:** se mantuvo deshabilitado (ligado a que UEFI estaba desmarcado en la configuración de Sistema) para evitar conflictos con el bootloader de Ubuntu (shim/GRUB no firmado).
- **Reutilización de partición EFI:** es fundamental **no crear una nueva partición EFI** durante la instalación de Ubuntu; se debe usar la que ya creó Windows, para que ambos sistemas compartan el mismo punto de arranque UEFI.

---
