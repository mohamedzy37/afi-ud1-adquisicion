# Notas de trabajo · UD1 · Adquirir un equipo comprometido

| | |
|---|---|
| **Analista** | Mohammed Ghayat Zayoun |
| **Caso** | Bahía Sistemas / Almadraba Diseño · equipo del cuarto de informática (administración) |
| **Equipo investigado** | VM `FORENSIC_10` (UUID `99749a58-fc75-4699-9298-466c2f9cb39a`) · nombre de equipo `FORENSE-06` |
| **Sistema** | Windows 7 Professional SP1, 64 bits (6.1.7601) · 1 GB de RAM (1.073.274.880 bytes) |
| **Equipo del analista** | Windows 11 (10.0.26200.8653) · VirtualBox 7.2.20r175154 |
| **Fechas** | 2026-11-03 (marca, en clase) · 2026-11-04 (adquisición en vivo) · 2026-11-05 (disco) · 2026-11-10 (pendrive, búsqueda de la marca, custodia) |
| **Marca** | Texto `AFI-MohammedGhayat-20261104` · fichero `marca-mohamedghayat.txt.txt` |

> **Criterio de horas.** "Hora real" = reloj del anfitrión, comprobado contra `hora.roa.es` (Real Instituto y Observatorio de la Armada) con un desfase de −0,23 s. "Hora VM" = reloj de la máquina investigada. Las dos se indican siempre que se conocen. Las marcadas con ≈ son aproximadas (reconstruidas a partir del reloj visible en las capturas) y las que no constan se indican así.

---

# Parte 1 · Adquisición en vivo (2026-11-04)

## 0. Preparación

| Hora real | Acción | Detalle |
|---|---|---|
| No consta (antes del 2026-11-03) | Importación de `FORENSIC_10.ova` en VirtualBox 7.2.20 | VM arrancada e inicio de sesión con la cuenta `Administrador` facilitada por Bahía Sistemas |
| ≈18:15 | Preparación del kit de herramientas para Windows 7 (`USB-Herramientas-W7`) | Motivo: el equipo es Windows 7 SP1 x64 y varias herramientas del USB de respuesta original no son compatibles (ver §0.1). MD5, SHA-1 y SHA-256 de cada herramienta en `hashes/hashes-herramientas.txt` |

### 0.1 Herramientas y por qué estas

| Paso | Herramienta y versión | Por qué |
|---|---|---|
| RAM | Prevista: Magnet DumpIt 3.6.20230117 x64 (fichero 3.1.0.0). **Usada: VBoxManage 7.2.20r175154 `debugvm dumpvmcore`** | DumpIt es la herramienta que pide el enunciado; falló (§2). El volcado desde el hipervisor no necesita controlador y no ejecuta nada en el equipo |
| Volátiles | `respuesta-en-vivo-win7.bat` 1.0 (elaboración propia) | El `respuesta-en-vivo.ps1` necesita PowerShell 3+ y Windows 7 trae la 2.0. Además, el `.ps1` del USB original estaba vacío (0 bytes). El `.bat` solo usa comandos nativos de Windows 7 y escribe todo en el USB |
| Red | VBoxManage `nictrace` | Captura desde fuera, como un TAP, sin instalar nada en la VM |
| Triaje | Velociraptor 0.77.3 *legacy* (recolector offline, `Windows.Triage.Targets` / `_KapeTriage`, unidad `C:`) | Única compilación de Velociraptor compatible con Windows 7. `_KapeTriage` incluye la `$MFT` |
| Cifrado | Magnet EDD 3.10 + comandos nativos | EDD necesita .NET 4.0; si no arranca, se comprueba a mano |

### 0.2 Incidencia previa a la marca

| Hora VM | Acción | Detalle |
|---|---|---|
| 18:06-18:08 | Identificación del sistema dentro de la VM, **antes de dejar la marca** | Comandos: `ver`, `echo %PROCESSOR_ARCHITECTURE%`, `wmic os get Caption,Version,CSDVersion,OSArchitecture`. Resultado: `Microsoft Windows 7 Professional · Service Pack 1 · 64-bit · 6.1.7601`. `cmd` no guarda el historial en disco. Debió hacerse después de la marca o desde fuera (`VBoxManage showvminfo`). Captura: `capturas/00-incidencia-identificacion-so-antes-de-marca.png` |

---

## Marca (única escritura permitida en el equipo)

| Hora VM | Acción | Detalle |
|---|---|---|
| 2026-11-03 10:12:44 (VM) | Creación del fichero de la marca en el escritorio de `Administrador` (primero con el nombre del ejemplo, `marca-nombreapellido.txt`) | Según NTFS (`istat`, Parte 3). Se renombra a `marca-mohamedghayat.txt.txt` a las 10:20:36 y queda **vacío (0 bytes)**. El 3 de noviembre, en clase, el profesor prestó el pendrive que hace de "encontrado", con contenido previo que no se borró |
| 2026-11-04 ≈19:10 (real) | Bloc de notas (Notepad) abierto con la marca, **sin guardar** | Texto: `AFI-MohammedGhayat-20261104`. Sigue abierto en las capturas posteriores (y está en la RAM, Parte 3) |
| 2026-11-04 ≈19:10-19:15 | El fichero de la marca ya existía en el escritorio desde el día 3 (`marca-mohamedghayat.txt.txt`, doble extensión y "mohamed" con una sola *m*) y **no se volvió a guardar con el texto** | Se ve en la primera captura del día 4 (`00-…`, 18:08). Detectado en la Parte 3 |
| 19:15:41-19:16:46 (real, `VBox.log`) | Conexión del pendrive "encontrado" (Dispositivos → USB), copia del fichero de la marca y desconexión | Pendrive Q-CONNECT 4 GB, sin n.º de serie de hardware (ver EV-06). Fotos `14a`/`14b` |

---

## Paso 1 · Estado inicial

| Hora | Acción | Herramienta | Por qué |
|---|---|---|---|
| ≈19:18 real | Conexión del USB de respuesta a la VM (Dispositivos → USB) | VirtualBox | Windows le asigna la letra `D:` |
| 19:21:11,90 VM | Hora y zona horaria de la VM | `echo %date% %time% & tzutil /g` | Resultado: `Wed 11/04/2026 19:21:11.90` · `Romance Standard Time` (UTC+1, Madrid) |
| 19:22:11 real | Hora real | `w32tm /stripchart /computer:hora.roa.es /samples:1 /dataonly` en el anfitrión | Fuente de hora oficial. Resultado: `19:22:11, -00.2252605s` (servidor 150.214.94.5) |

- **Desfase:** los dos comandos se lanzaron con un minuto de diferencia, así que esta medición no sirve para calcularlo (incidencia). Se repitió a la vez y la VM y la hora real coincidían: **desfase ≈ 0 s** a las ≈19:28 (comprobación visual del analista; no se guardó captura de esta segunda medición).
- **Desfase posterior:** a partir de ≈19:50 el reloj de la VM aparece **unos 20 minutos atrasado** respecto a la hora real (por ejemplo, VM 8:08 PM con hora real ≈20:28; DumpIt nombró su fichero `…-190742` en UTC = 20:07:42 hora VM). Posible causa: VM en pausa o anfitrión en suspensión mientras se formateaba el USB (≈19:50-20:10). **Causa confirmada** por `VBox.log`: a las 20:04:10 VirtualBox registra `TM: Giving up catch-up attempt at a 1 164 879 718 794 ns lag`, es decir, la VM estuvo ~19 min 25 s sin tiempo de CPU y su reloj quedó atrasado esa cantidad (ver "Cronología según el registro de VirtualBox").
- **Qué se ve en pantalla:** escritorio de `Administrador` con Notepad (marca), `marca-mohammedghayat.txt` y **dos ficheros `crea_user`**, uno con icono de **script de Python**. No se han abierto. Pueden estar relacionados con el compromiso (creación de usuarios en el equipo que administra el resto).
- **Incidencia:** la primera captura de pantalla "limpia" del estado inicial desde VirtualBox (Ver → Tomar captura) no se hizo antes de interactuar. Se usan como estado inicial `capturas/00-…` y `capturas/01a-…`.
- **Incidencia:** `fsutil fsinfo volumeinfo D:` se lanzó en una `cmd` sin permisos de administrador y en la misma línea que `dir D:\` → `The FSUTIL utility requires that you have administrative privileges`. Se abrió una `cmd` como administrador (`Start → cmd → Run as administrator`) y se usó `wmic` en su lugar. Captura: `capturas/01c-incidencia-fsutil-sin-admin.png`

Capturas: `01a-estado-inicial-hora-vm.png`, `01b-estado-inicial-hora-real-w32tm.png`

### Incidencia: USB de respuesta en FAT32 y con datos ajenos

| Hora VM | Acción | Detalle |
|---|---|---|
| 19:30 | `wmic logicaldisk get DeviceID,FileSystem,FreeSpace,Size` y `dir D:\` | `D:` en **FAT32** (34.347.155.456 bytes, 16.914.513.920 libres), etiqueta `ESD-USB`, n.º de serie del volumen `A28B-F054`. Contenía un instalador de Windows y ficheros personales ajenos al caso. FAT32 no admite ficheros de más de 4 GB y el enunciado lo prohíbe |
| ≈19:50 | Desconexión del USB de la VM (Dispositivos → USB) | No se había escrito nada en él desde la VM. `wmic computersystem get TotalPhysicalMemory` → `1073274880` |
| ≈19:50-20:10 real | En el anfitrión: copia de seguridad de los datos personales, **formateo en NTFS** y copia **solo** de `USB-Herramientas-W7` | Hash MD5 de DumpIt comprobado después en la VM: coincide (captura `03b`) |
| 20:16:57 real (`VBox.log`) | Nueva conexión del USB a la VM, letra `D:`, NTFS | La etiqueta del volumen sigue apareciendo como `ESD-USB` |

Capturas: `02a-incidencia-usb-respuesta-fat32.png`, `02b-incidencia-usb-respuesta-contenido.png`, `02c-usb-desconectado-ram-total.png`

> En el registro de la VM quedará la conexión de este USB (dos veces) y la del pendrive "encontrado". Tenerlo en cuenta en la UD2.

---

## Paso 2 · Memoria RAM

### 2.1 Intentos con DumpIt (fallidos)

| Hora VM | Acción | Resultado |
|---|---|---|
| 7:55 PM | `certutil -hashfile D:\USB-Herramientas-W7\memoria\DumpIT\3.1.0-x64\DumpIt.exe MD5` | Error: ruta mal escrita (`\` en lugar de `-`, por la distribución de teclado inglesa de la VM) → `0x80070003`. Captura `03a` |
| 7:58 PM | `certutil -hashfile D:\USB-Herramientas-W7\memoria\DumpIt-3.1.0-x64\DumpIt.exe MD5` | `0f10da3a5eb49d17d73d8e195fe32f85`, que **coincide** con `hashes-herramientas.txt`. Captura `03b` |
| 8:01 PM | DumpIt lanzado con **doble clic** desde el Explorador (con elevación UAC) | DumpIt **3.6.20230117 (x64)**. Destino propuesto: la propia carpeta de la herramienta (`…\DumpIt-3.1.0-x64\FORENSE-06-20261104-190145.dmp`). Equipo: `FORENSE-06`. Se respondió `y` y la ventana se cerró sola. Captura `03c` |
| 8:08 PM | `DumpIt.exe` lanzado desde la `cmd` de administrador, **sin `/OUTPUT`** (directorio actual `C:\Windows\system32`) | Destino propuesto: `C:\Windows\system32\FORENSE-06-20261104-190742.dmp` (**disco de la VM**, error del analista). Tras `y`: `Error: Load driver failed with 0xC0000428 error code` · `ImagePath = C:\Windows\system32\Drivers\DumpIt.sys` · `Error: Can't install the driver`. No se llegó a escribir el volcado. Captura `03d` |

**Diagnóstico:** `0xC0000428` (STATUS_INVALID_IMAGE_HASH) indica que Windows no puede validar la firma del controlador. Windows 7 SP1 sin la actualización de soporte de firma SHA-2 (KB4474419) no carga controladores firmados solo con SHA-2. WinPmem 4.0 usa el mismo tipo de firma, así que se descarta para no repetir el intento.

**Escrituras en el equipo investigado causadas por DumpIt (no evitables una vez lanzado):**
- Clave `HKLM\System\CurrentControlSet\Services\DumpIt` (DumpIt indica que la eliminó: "We are cleaning the registry keys").
- Probable fichero `C:\Windows\system32\Drivers\DumpIt.sys`.
- Prefetch de `DUMPIT.EXE`.

### 2.2 Volcado desde el hipervisor (válido)

**Herramienta:** VBoxManage 7.2.20r175154, en el anfitrión.
**Por qué:** la RAM es lo más volátil y se pierde al apagar. Como no se puede cargar ningún controlador en la VM, se adquiere desde el hipervisor, el equivalente a un volcado por hardware: no ejecuta nada en el equipo investigado, no escribe en su disco y recoge la memoria física completa. Volatility 3 lee este formato (núcleo ELF de VirtualBox).

```
cd "C:\Program Files\Oracle\VirtualBox"
VBoxManage.exe --version
VBoxManage.exe list runningvms
mkdir C:\caso-afi\ram
VBoxManage debugvm "FORENSIC_10" dumpvmcore --filename=C:\caso-afi\ram\PC-ADMIN-ram.elf
dir C:\caso-afi\ram
certutil -hashfile C:\caso-afi\ram\PC-ADMIN-ram.elf SHA256
certutil -hashfile C:\caso-afi\ram\PC-ADMIN-ram.elf MD5
```

| Hora real | Fichero | Tamaño | SHA-256 | MD5 |
|---|---|---|---|---|
| 20:35 | `C:\caso-afi\ram\PC-ADMIN-ram.elf` | 1.094.580.064 bytes | `795858400b7b6ad4f61abea8613386a27f380c0100db356ad839ebf3474a1ad6` | `8e78f9bf4d129bee9ecde631aef476dd` |

- `dumpvmcore` no muestra salida cuando termina bien. El tamaño es algo mayor que la RAM (1.073.274.880 bytes) porque incluye las cabeceras ELF y el estado de la CPU.
- El volcado queda en el anfitrión porque el USB está conectado a la VM. Se copiará a la carpeta de evidencias verificando el hash.

Capturas: `04a-ram-dumpvmcore.png`, `04b-ram-fichero-elf.png`, `04c-ram-hashes.png`

---

## Paso 3 · Datos volátiles

**Herramienta:** `respuesta-en-vivo-win7.bat` 1.0 (MD5 `5b0820e7ed8480883b776fd8372f320f`) · **Por qué:** ver §0.1.

```
certutil -hashfile D:\USB-Herramientas-W7\volatiles\respuesta-en-vivo-win7.bat MD5
D:\USB-Herramientas-W7\volatiles\respuesta-en-vivo-win7.bat D:\USB-Herramientas-W7\salida\volatiles
```

- **Incidencia (8:27 PM VM ≈ 20:47 real):** se escribió `certuil` en lugar de `certutil` → `'certuil' is not recognized…`. Se repitió. Captura `05a-incidencia-certutil-mal-escrito.png`. En esa captura se confirma el retraso de ≈20 min de la VM.
- **8:29 PM VM (≈20:48 real):** MD5 del script `5b 08 20 e7 ed 84 80 88 3b 77 6f d8 37 2f 32 0f`, que **coincide** con `hashes-herramientas.txt`. Captura `05b-hash-script-verificado.png`.

| Hora inicio (VM / real) | Hora fin (VM / real) | Carpeta de salida | Registro del script |
|---|---|---|---|
| 20:35:58 VM (≈20:55 real) | 8:36 PM VM (≈20:56 real) | `D:\USB-Herramientas-W7\salida\volatiles\FORENSE-06-20261104-203558\` | `_registro.log`, `_hashes-sha256.txt` |

- **Incidencia:** primer intento de escribir la ruta con `D;` (punto y coma, por el teclado inglés) → el Tab no completaba. Corregido a `D:` (Shift + Ñ). Captura `05c-incidencia-punto-y-coma.png`.
- El script terminó con `Terminado.` y generó la carpeta `FORENSE-06-20261104-203558` (hora de la VM) con un `.txt` por comando (00 a 33), `_registro.log` y `_hashes-sha256.txt`. Captura `05d-volatiles-terminado.png`.
- USB de respuesta tras el formateo: etiqueta `ESD-USB`, n.º de serie del volumen **`EA4E-856E`**, 34.177.363.968 bytes libres.

Lo más relevante que se ha visto (revisado en el anfitrión el 2026-11-10):
- **Conexiones extrañas** (`01-netstat-anob.txt`): `192.168.18.53:49335 → 10.28.5.1:8081 SYN_SENT` del proceso **`QkryuzzwVu.exe`** (PID 1976) y `192.168.18.53:49336 → 10.28.5.1:53 SYN_SENT` de **`KzcmVNSNkYkueQf.exe`** (PID 2328). Los dos intentan conectar con la misma IP externa.
- **Procesos** (`13-procesos-wmic.txt`): esos ejecutables corren desde `C:\Users\ADMINI~1\AppData\Local\Temp\radC8421.tmp\` y `…\radAB2F4.tmp\` (creados a las 20:33:26 hora VM), hijos de dos `wscript.exe` (PID 2508 y 2500) que ejecutan `C:\Windows\Temp\HoVgcPUXNBk.vbs` y `C:\Windows\Temp\jnQGgUrmj.vbs` desde el inicio de sesión (19:01:39 hora VM).
- **Persistencia** (`25-run-hklm.txt`): claves `HKLM\…\Run` `GZARMViUQp` → `C:\Windows\TEMP\jnQGgUrmj.vbs` e `ijgWsjRRs` → `C:\Windows\TEMP\HoVgcPUXNBk.vbs` (nombres aleatorios).
- **Usuarios:** `Administrador` (sesión de consola desde 11/4/2026 7:01 PM), `Administrator` y `Guest`; `Administrador` y `Administrator` son administradores. En el escritorio están `crea_user.py` y `crea_user.txt`.
- Servicios a la escucha: RDP (3389), SMB (445), HTTP (80), MSMQ (`mqsvc.exe`).
- Para analizar en la UD2 (memoria, triaje y disco).

---

## Paso 4 · Tráfico de red

**Herramienta:** VBoxManage `nictrace` en el anfitrión · **Por qué:** se captura desde fuera, como un TAP, sin instalar nada en la VM.

```
mkdir C:\caso-afi\red
VBoxManage controlvm "FORENSIC_10" nictracefile1 C:\caso-afi\red\red.pcap
VBoxManage controlvm "FORENSIC_10" nictrace1 on
VBoxManage showvminfo "FORENSIC_10" | findstr /i "NIC"
(al terminar)
VBoxManage controlvm "FORENSIC_10" nictrace1 off
certutil -hashfile C:\caso-afi\red\red.pcap SHA256
certutil -hashfile C:\caso-afi\red\red.pcap MD5
```

- **≈20:58 real:** captura activada. `showvminfo` confirma: `NIC 1: MAC 080027B0E9DF, Attachment: Bridged Interface 'Realtek RTL8822CE 802.11ac PCIe Adapter', Cable connected: on, Trace: on (file: C:\caso-afi\red\red.pcap), Type: 82545EM`. NIC 2 a 8 desactivadas. Captura `06a-red-nictrace-on.png`.
- **Observación:** la VM está en modo **puente** sobre la wifi del anfitrión, es decir, el equipo comprometido está conectado a la red local real. No se cambia la configuración en vivo (se alterarían sus conexiones y lo que se captura). La captura incluirá también tráfico de difusión de otros equipos de la red. En un caso real se valoraría aislarlo después de la captura.
- La captura se deja activa durante el triaje y la comprobación del cifrado, y se para antes del apagado.

| Hora on (real) | Hora off (real) | Fichero | Tamaño | SHA-256 |
|---|---|---|---|---|
| ≈20:58 (carpeta creada a las 20:58) | ≈21:55 (última escritura del fichero) | `C:\caso-afi\red\red.pcap` | 6.023.020 bytes | `90c41195f1455a4cdebccada449cfd11a5fe511f83453c58a25b43654fca65f6` |

- MD5: `eeecceb27e87c7729e0a90c8bf85d568`. Captura `09b-red-nictrace-off-hashes.png`.
- **Incidencia menor:** primer `certutil` con la ruta `C:\caso-afi\red.pcap` (sin `\red`) → `0x80070002`. Se repitió con la ruta correcta.
- La captura abarca el triaje, la reconexión del USB y la comprobación del cifrado.

---

## Paso 5 · Triaje

**Herramienta:** Velociraptor 0.77.3 *legacy*, recolector offline (`Windows.Triage.Targets`, `_KapeTriage`, unidad `C:`) · **Por qué:** ver §0.1.

- **8:43-8:44 PM VM (≈21:03 real):** MD5 del recolector `e9 83 fb 6f 46 95 10 f2 53 b1 34 a1 37 ce 91 7d`, que **coincide** con `hashes-herramientas.txt`. Captura `07a-hash-recolector-verificado.png`.
- `TMP` y `TEMP` se redirigen al USB para que Velociraptor no escriba ficheros temporales en el disco investigado.
- **≈8:53 PM VM (≈21:13 real):** carpetas creadas y variables asignadas. **Incidencia:** el primer `set TMP=\USB/Herramientas\salida\tmp` se escribió mal (sin `D:`, con `/`); se corrigió con un segundo `set TMP=D:\USB-Herramientas-W7\salida\tmp`, que lo sobrescribe. Captura `07b-triaje-preparacion.png`.
- **8:55 PM VM (≈21:15 real) · Incidencia:** el recolector 0.77.3 *legacy* no arranca: `The program can't start because api-ms-win-crt-stdio-l1-1-0.dll is missing from your computer`. La compilación legacy importa el **Universal C Runtime** (`api-ms-win-crt-*.dll`, comprobado con `objdump -p` en el equipo del analista), y este Windows 7 no tiene instalado KB2999226. No llegó a ejecutarse nada (fallo del cargador de Windows). Captura `07c-incidencia-velociraptor-legacy-ucrt.png`.
- **Decisión:** se usa **Velociraptor 0.6.9**, versión oficial compilada con Go 1.19.9 (con soporte para Windows 7) y enlazada con `msvcrt.dll`, que sí existe en Windows 7 (comprobado también con `objdump -p`). Incluye el artefacto `Windows.KapeFiles.Targets`, predecesor de `Windows.Triage.Targets`, con el mismo target `_KapeTriage`. Se lanza con un `.bat` propio (`triaje-velociraptor-069.bat`) que guarda el comando exacto, las horas y los hashes del ZIP. Hashes en `hashes/hashes-herramientas.txt`.
- **Incidencia de custodia:** para añadir la herramienta al USB hubo que desconectarlo de la VM, copiarla en el anfitrión y volver a conectarlo. Horas (según `VBox.log`): desconexión 21:21:38 · conexión 21:24:19, desconexión 21:25:03 y conexión definitiva 21:25:08. Antes de desconectarlo: `cd /d C:\`, cierre del Explorador y expulsión segura desde Windows. En el anfitrión se comprobó el MD5 de los dos ficheros copiados. Tras reconectar, `dir D:\USB-Herramientas-W7\triaje` muestra `Velociraptor-0.6.9` con fecha **9:22 PM** (hora del anfitrión, la real) mientras la VM marca **9:09 PM**: otra prueba del retraso de ≈20 min. Captura `07d-usb-reconectado-velociraptor-069.png`.
- **9:16 PM VM (≈21:36 real):** MD5 de Velociraptor 0.6.9 `74 ce a9 f1 e8 97 9c 87 e4 07 e0 d4 99 86 70 3c`, que **coincide**. Captura `07e-hash-velociraptor-069-verificado.png`. (Incidencia menor: se escribió `Windows\System32` como comando por error.)

Comando ejecutado (lo hace el `.bat`):

```
velociraptor-v0.6.9-windows-amd64.exe --nobanner --tempdir <salida>\tmp --logfile <salida>\Collection-FORENSE-06-<fecha>.log artifacts collect Windows.KapeFiles.Targets --args Device=C: --args _KapeTriage=Y --require_admin --output <salida>\Collection-FORENSE-06-<fecha>.zip -v
```

```
certutil -hashfile D:\USB-Herramientas-W7\triaje\Velociraptor-0.77.3-legacy\Collector_velociraptor-v0.77.3-windows-amd64-legacy.exe MD5
set TMP=D:\USB-Herramientas-W7\salida\tmp
set TEMP=D:\USB-Herramientas-W7\salida\tmp
mkdir D:\USB-Herramientas-W7\salida\tmp
mkdir D:\USB-Herramientas-W7\salida\triaje
cd /d D:\USB-Herramientas-W7\salida\triaje
D:\USB-Herramientas-W7\triaje\Velociraptor-0.77.3-legacy\Collector_velociraptor-v0.77.3-windows-amd64-legacy.exe
```

| Hora inicio | Hora fin | Fichero | Tamaño | SHA-256 |
|---|---|---|---|---|
| 21:18:20 VM (≈21:38 real) | 21:20:10 VM (≈21:40 real) · código 0 | `D:\USB-Herramientas-W7\salida\triaje\Collection-FORENSE-06-20261104-211820.zip` | 46.293.848 bytes | `fa6b129cba315e830a80a3cd66a15ced366b718a2dbae1822042aa9a72f0f341` |

- MD5 del ZIP: `4ab86ec6669246d52aa3e135603fe2d8`.
- Ficheros generados junto al ZIP: `.log` (registro de Velociraptor), `.resultado.json` e `.info.txt` (comando, horas y hashes).
- Comando real registrado en el `.info.txt`: `"D:\USB-Herramientas-W7\triaje\Velociraptor-0.6.9\velociraptor-v0.6.9-windows-amd64.exe" --nobanner --tempdir "D:\USB-Herramientas-W7\salida\triaje\tmp" --logfile "D:\USB-Herramientas-W7\salida\triaje\Collection-FORENSE-06-20261104-211820.log" artifacts collect Windows.KapeFiles.Targets --args Device=C: --args _KapeTriage=Y --require_admin --output "D:\USB-Herramientas-W7\salida\triaje\Collection-FORENSE-06-20261104-211820.zip" -v`
- Captura `07f-triaje-terminado-hashes.png`.
- `dir` de la salida: `.info.txt` 979 B · `.log` 298.313 B · `.resultado.json` 250.432 B · `.zip` 46.293.848 B · carpeta `tmp`. Captura `07g-triaje-dir.png`.
- **9:26 PM VM (≈21:46 real) · Comprobación de la `$MFT`** (necesaria para buscar la marca en la Parte 3): `findstr /i "MFT" D:\USB-Herramientas-W7\salida\triaje\Collection-FORENSE-06-20261104-211820.log`. El registro muestra `Found \\.\C:\$MFT` y `Collecting file \\.\C:\$MFT ... (69206016 bytes)`, copiada con el analizador NTFS de Velociraptor (horas del registro en UTC: 20:18-20:19Z = 21:18-21:19 hora VM). La `$MFT` (66 MB) está en el triaje; el ZIP pesa 46 MB por la compresión. Capturas `07h-triaje-mft-log-1.png` y `07i-triaje-mft-log-2.png`.

---

## Paso 6 · Cifrado

**Comprobado con:** Magnet Encrypted Disk Detector (EDD) 3.10, ejecutado desde el USB a las **9:29 PM VM (≈21:49 real)**. MD5 de la herramienta comprobado al preparar el kit (coincide con USB-RESPUESTA).

```
D:\USB-Herramientas-W7\cifrado\EDD-3.10\EDDv310.exe
```

Resultado (resumen de la salida):

```
Drive C: (PhysicalDrive0), Drive Type: Fixed, Filesystem: NTFS, Size: 34 GB, Free Space: 23 GB
Drive D: [Label: ESD-USB] (PhysicalDrive1), Drive Type: Removable, Filesystem: NTFS, Size: 34 GB, Free Space: 34 GB
* Running Secondary Bitlocker Check... *
* Checking for running processes... *
*** No TrueCrypt, PGP, Bitlocker, VeraCrypt, SafeBoot, BestCrypt, Checkpoint, Sophos, or Symantec encrypted volumes detectable by EDD were found. ***
```

- **Qué se ha visto:** ningún volumen cifrado (ni en el disco físico ni en los volúmenes lógicos) y ningún proceso de software de cifrado en ejecución. Además, la edición Windows 7 Professional no incluye BitLocker.
- **Qué se ha hecho en consecuencia:** no hace falta extraer claves de recuperación ni hacer una imagen lógica en vivo antes de apagar. El disco será legible en la adquisición post-mortem (Parte 2), así que se puede apagar.

Captura: `08a-cifrado-edd.png`

---

## Paso 7 · Apagado

| Hora real | Acción |
|---|---|
| ≈21:53 real (9:34 PM VM) | **Incidencia:** `VBoxManage controlvm "FORENSIC_10" nictrace1 off` se escribió por error en la `cmd` de la **VM** (dos veces, la primera como `VBoxMange`) → `'VBoxManage' is not recognized…`. Sin efecto en la VM, salvo que queda en la consola. Se repitió en el anfitrión. Captura `09a-incidencia-vboxmanage-en-vm.png` |
| ≈21:55 real | Fin de la captura de red (`VBoxManage controlvm "FORENSIC_10" nictrace1 off`, en el anfitrión) y hash (ver paso 4) |
| 9:45 PM VM (≈22:05 real) | `cd /d C:\` en la `cmd` de la VM para no dejar el USB en uso. Expulsión segura y desconexión del USB: **22:01:48** (según `VBox.log`). Captura `09c-vm-antes-del-apagado.png` (en ella ya no se ve el Notepad de la marca en la barra de tareas; no consta cuándo se cerró, pero fue después del volcado de RAM, que lo recoge) |
| ≈22:06 real | **Apagado por corte de corriente** desde el anfitrión: `VBoxManage controlvm "FORENSIC_10" poweroff` → `0%...100%`. Comprobación: `VBoxManage list runningvms` no devuelve nada. **Incidencia menor:** primer intento escrito como `controlvm "FORENSIC_10" poweroff` (sin `VBoxManage`) → no reconocido. Captura `09d-apagado-poweroff.png` |

**Cómo:** `VBoxManage controlvm "FORENSIC_10" poweroff`, el equivalente a desconectar el cable de alimentación. No se apagó desde Windows (Start → Shut down) ni con "Apagado ACPI", y no se guardó el estado ni se hizo ninguna instantánea.

**Por qué:**
- Todo lo volátil ya estaba adquirido (RAM, datos volátiles, red y triaje) y no hay cifrado (paso 6), así que no se pierde nada necesario.
- Un apagado ordenado escribe en el disco (registros de eventos, ficheros temporales, actualización del registro, posible vaciado del pagefile) y ejecutaría los scripts o tareas de apagado que hubiera dejado el atacante, que podrían borrar rastros.
- El corte de corriente congela el disco tal como estaba en ese momento. Lo único que puede perderse son escrituras en caché aún no volcadas, cuyo contenido está en el volcado de RAM. NTFS tiene registro de transacciones y la imagen se monta en solo lectura, así que la posible inconsistencia es mínima.
- La VM **no se vuelve a arrancar**. La Parte 2 se hace sobre sus ficheros (`.vmdk`, `.vbox`, `Logs/`) con la máquina apagada.

---

## Verificación tras el apagado

| Hora real | Acción | Resultado |
|---|---|---|
| 22:09 | USB de respuesta conectado al anfitrión (letra `D:`). `Get-FileHash -Algorithm SHA256 "D:\USB-Herramientas-W7\salida\triaje\Collection-FORENSE-06-20261104-211820.zip" \| Format-List` | `FA6B129CBA315E830A80A3CD66A15CED366B718A2DBAE1822042AA9A72F0F341`, que **coincide** con el hash calculado en la VM al terminar el triaje. Captura `10a-triaje-hash-verificado-anfitrion.png` |

### Comprobación del USB de respuesta en el anfitrión (2026-11-05, ≈20:30)

Comprobación de solo lectura del contenido de `D:\USB-Herramientas-W7\`:
- **Volátiles:** los 35 ficheros de `FORENSE-06-20261104-203558` (34 `.txt` y `_registro.log`) coinciden con su `_hashes-sha256.txt`.
- **Triaje:** el ZIP sigue dando SHA-256 `fa6b129c…f0f341` y MD5 `4ab86ec6…3fe2d8`, igual que en la VM. El `.info.txt` contiene el comando, las horas (21:18:20,72 a 21:20:10,28, código 0) y los hashes.
- **Herramientas:** el MD5 de las 9 herramientas del USB coincide con `hashes-herramientas.txt`.
- `salida\ram\` no llegó a crearse (DumpIt falló). DumpIt no dejó ningún `.dmp` parcial en el USB.
- **Vista previa de la Parte 3:** `strings -el '$MFT' | grep -i "marca-"` sobre la `$MFT` del triaje (69.206.016 bytes) devuelve `marca-mohamedghayat.txt.txt`, `MARCA-~1.TXT`, `MARCA-~1.LNK` y `marca-nombreapellido.txt.lnk`.
  - **Incidencia:** el fichero de la marca se guardó como **`marca-mohamedghayat.txt.txt`**, con doble extensión (Notepad añadió `.txt` porque Windows oculta las extensiones) y con **"mohamed"** escrito con una sola *m*, mientras que el texto de la marca es `AFI-MohammedGhayat-20261104`. Se deja tal cual: es el nombre real que aparecerá en todas las evidencias.
  - `marca-nombreapellido.txt.lnk` es un acceso directo de "Recientes" a un fichero llamado literalmente `marca-nombreapellido.txt`. Según `istat` (Parte 3), el fichero de la marca se creó el 2026-11-03 y se renombró desde ese nombre de ejemplo.

## Unificación de las evidencias en el USB (2026-11-05)

Para tener todas las evidencias de la adquisición en vivo en un solo sitio, se copian al USB de respuesta las dos que se adquirieron desde el anfitrión. Se **copian, no se mueven**: los originales de `C:\caso-afi\` se conservan como copia de seguridad hasta la entrega.

| Hora real | Acción | Comprobación |
|---|---|---|
| ≈20:58 | `C:\caso-afi\red\red.pcap` → `D:\USB-Herramientas-W7\salida\red\red.pcap` | SHA-256 de la copia `90c41195…fca65f6`: coincide |
| ≈21:00 | `C:\caso-afi\ram\PC-ADMIN-ram.elf` → `D:\USB-Herramientas-W7\salida\ram\PC-ADMIN-ram.elf` (copia del analista) | 21:02 · `Get-FileHash -Algorithm SHA256 … \| Format-List` → `795858400B7B6AD4F61ABEA8613386A27F380C0100DB356AD839EBF3474A1AD6`: coincide. Captura `11a-ram-copia-usb-hash-verificado.png` |
| ≈20:58 | Se crea `D:\USB-Herramientas-W7\salida\hashes-evidencias.txt` con los datos y hashes de EV-01 a EV-04 (copia también en `hashes/`) | — |

Estructura de `salida\`: `hashes-evidencias.txt`, `ram\`, `volatiles\`, `red\`, `triaje\` (y `tmp\`, vacía).

# Parte 2 · Adquisición post-mortem (2026-11-05)

## Disco de la máquina (EV-05)

### 1. Documentación de la máquina (≈21:06-21:17, VM apagada desde el 2026-11-04 22:05:26)

```
VBoxManage showvminfo "FORENSIC_10" --machinereadable | findstr /i "CfgFile SnapFldr LogFldr vmdk Controller"
VBoxManage snapshot "FORENSIC_10" list
VBoxManage showmediuminfo disk "C:\Users\Windows\VirtualBox VMs\FORENSIC_10\FORENSIC_10_disk0.vdi"
```

- **Carpeta:** `C:\Users\Windows\VirtualBox VMs\FORENSIC_10\` (configuración `FORENSIC_10.vbox`, registros en `Logs\`, instantáneas en `Snapshots\`).
- **Instantáneas:** ninguna (`This machine does not have any snapshots`; la carpeta `Snapshots\` está vacía).
- **Controladores:** IDE (PIIX4) y SAS (LsiLogicSas). Único disco: SAS puerto 0.
- **Disco:** `FORENSIC_10_disk0.vdi` · UUID `4fdf748a-fd7e-4b05-98ff-0eca49b38f27` · tipo **normal (base)**, sin padre · formato **VDI dinámico** · capacidad **32.768 MB** (34.359.738.368 bytes) · ocupa 12.742 MB (13.360.955.392 bytes) · sin cifrar.
- **Nota:** el enunciado habla de `.vmdk`, pero al importar el `.ova` VirtualBox guardó el disco como `.vdi`. FTK Imager no admite VDI (ver paso 3).
- Capturas `12a-disco-showmediuminfo.png`.

### 2. Hashes de los ficheros de la carpeta, antes de la imagen

| Fichero | Tamaño (bytes) | SHA-256 | MD5 |
|---|---|---|---|
| `FORENSIC_10_disk0.vdi` | 13.360.955.392 | `ec721351bb7e7e06a02f3e32ec0680f08109173192c0bb2f296ccc28d3702114` | `983f6136be1119213be2ebcff503995c` |
| `FORENSIC_10.vbox` | 2.563 | `f0677852b1e257ec4ca2e652fc3d8fc7216e16ab36f7f9f827a3361e5db15945` | `b67274972d078ac8654a9126478b410e` |
| `FORENSIC_10.vbox-prev` | 3.258 | `53934f85ecb964d1b26cb92a6c04d2a57f08bfbbf3ff8b8d474c349fbeab3ef8` | `ef85168c8ec015a0bbb369bad6686efb` |
| `Logs\VBox.log` | 380.856 | `0b39c0ed65a013150ee1816b0fd9b98eea3c66bb4e92613545ff14615c9ccabf` | `597703d8b92adfb383a711a697a13b00` |
| `Logs\VBox.log.1` | 278.915 | `99a71652c476869b1671f91f57ddbc14df187dbe2d77c477c599448b38df2c81` | `718ced49cd732ab6020d8832d7abb2ed` |
| `Logs\VBox.log.2` | 372.985 | `03b0693c6e015eb261aa736c01e13216e9b0369a49d7fbd47edbd926641ef872` | `467a49a8d3a4b7eec73c58f7f430b41a` |
| `Logs\VBox.log.3` | 373.648 | `071a3894a42b71b28165517eeb413ad290d9fdc0e7a9e39a4ae4a8d7ac2950b5` | `dcbb2bd4f369d6f1ef8c0ddb0ab237ed` |
| `Logs\VBoxHardening.log` | 508.992 | `ad31817a8d2aae3e8900eececaf13683e664de94caad0c380f0507052a841e6b` | `f722c4601d4b24a7770a6db242e7cb63` |

- SHA-256 del `.vdi` con `Get-FileHash -Algorithm SHA256 … | Format-List` a las ≈21:18 (captura `12b-disco-vdi-hash-antes.png`). Su MD5 se calculó a las ≈21:35, ya después de la conversión del paso 3 (solo lectura).
- Los ficheros pequeños se leyeron en solo lectura desde la carpeta de la VM y se calcularon sus hashes con `sha256sum` y `md5sum`.

### 3. Imagen del disco

**Problema:** FTK Imager (la herramienta del enunciado para Windows) no abre discos VDI. **Solución:** convertir el contenido del disco a RAW con `VBoxManage clonemedium`, equivalente al `qemu-img convert -f … -O raw` del enunciado, que solo lee el original. Con FTK Imager se genera después el E01 verificado a partir del RAW.

**Fase A · VDI → RAW** (VBoxManage 7.2.20r175154, `cmd` del anfitrión):

```
mkdir C:\caso-afi\disco
VBoxManage clonemedium disk "C:\Users\Windows\VirtualBox VMs\FORENSIC_10\FORENSIC_10_disk0.vdi" "C:\caso-afi\disco\PC-ADMIN.raw" --format RAW
VBoxManage closemedium disk "C:\caso-afi\disco\PC-ADMIN.raw"
```

- ≈21:30-21:32 real: `0%...100%` → `Clone medium created in format 'RAW'. UUID: f6b6f376-af94-4e71-99cf-ae9690209c11`. Captura `12c-disco-clonemedium-raw.png`.
- `closemedium` (sin `--delete`) quita el RAW de la lista de discos de VirtualBox sin borrarlo. ≈21:39: el primer `closemedium` terminó sin mensajes (correcto). Al repetirlo, da `Could not get the storage format of the medium … (VERR_NOT_SUPPORTED)`, que confirma que el RAW ya no está registrado. Captura `12e-disco-closemedium.png`.
- `PC-ADMIN.raw`: **34.359.738.368 bytes** (= capacidad del disco).
  - MD5 `7dc45b11965b2b27f90a9f8eaafe7e30`
  - SHA-256 `a2a5a640b0171ee32914a1adfe16c70b0817b1464c1c27310e55feee7578910d`
  - Captura `12d-disco-raw-hashes.png`.
- Como el hash del `.vdi` es el del contenedor y el del RAW es el del contenido del disco, **no coinciden, y es lo normal**.

**Fase B · RAW → E01 con FTK Imager** (Exterro FTK Imager 8.3.0.27, instalado en el anfitrión):

- File → Create Disk Image → **Image File** → origen `C:\caso-afi\disco\PC-ADMIN.raw` → tipo **E01** → datos de la evidencia (Case `AFI-UD1-Almadraba`, Evidence `EV-05`, examinador Mohammed Ghayat Zayoun) → destino `D:\USB-Herramientas-W7\salida\disco\`, nombre **`Imagen-Disco`**, sin fragmentar → marcado **Verify images after they are created**. Captura `13a-ftk-create-image.png`.
- **≈21:48:** inicio. **Incidencia:** aviso `Low Disk Space Warning` (FTK Imager reserva 32.768 MB, el disco sin comprimir, y en el USB hay 31.447 MB disponibles). Se responde **No** (continuar en el mismo destino): el E01 comprimido ocupa mucho menos porque el disco solo tiene ~12,7 GB asignados. Captura `13b-ftk-aviso-espacio.png`.
- Registro de FTK Imager (`Imagen-Disco.E01.txt`): adquisición **21:50:37-21:56:38**, verificación **21:56:38-22:00:13** (2026-11-05). En los datos del caso quedó `Case Number: Caso-01`, `Evidence Number: EV-05`, `Examiner: Mohammed Ghayat`, sin descripción ni notas.
- **Resultado:** `Image created successfully` (6 min 01 s). Verificación: **MD5 y SHA1 → Match** (computed = stored = report), `No bad blocks found in image`. Sector count 67.108.864 × 512 = **34.359.738.368 bytes**, igual que el RAW.
  - MD5 **`7dc45b11965b2b27f90a9f8eaafe7e30`**, **idéntico al MD5 del `PC-ADMIN.raw`** calculado con `Get-FileHash`: la imagen contiene exactamente el disco convertido.
  - SHA1 `c0bb182fbb35d55720f6e2c2414ed5ae267c7460`
  - Capturas `13c-ftk-verificacion-disco.png` y `13d-ftk-imagen-creada.png`.
- Ficheros: `D:\USB-Herramientas-W7\salida\disco\Imagen-Disco.E01` (4.979.261.033 bytes) e `Imagen-Disco.E01.txt` (2.215 bytes, SHA-256 `54b2f897195e86c02a82d75357d5499b36891ed60f70a35dacfa5e32924bd74d`).

### 4. Hash del `.vdi` después de la imagen

- **≈22:02:** `Get-FileHash -Algorithm SHA256 "C:\Users\Windows\VirtualBox VMs\FORENSIC_10\FORENSIC_10_disk0.vdi" | Format-List` → `EC721351BB7E7E06A02F3E32EC0680F08109173192C0BB2F296CCC28D3702114`, **idéntico al del paso 2**. La conversión y la imagen no han escrito en el original. MD5 también igual: `983F6136BE1119213BE2EBCFF503995C`. Captura `13e-disco-vdi-hash-despues.png`.

### 5. Copia de la configuración y los registros junto a la imagen

- **22:01:** copia de `FORENSIC_10.vbox`, `FORENSIC_10.vbox-prev` y `Logs\*` (5 ficheros) a `D:\USB-Herramientas-W7\salida\disco\config\`. Antes se comprobó que los originales no habían cambiado (mismos tamaños y fechas de modificación que en el paso 2).
- Comprobación: `sha256sum -c` de las 7 copias del USB contra los originales → **7/7 OK**. Coinciden con los hashes del paso 2.

## Pendrive encontrado (EV-06)

### Identificación (2026-11-10, ≈19:40), antes de conectarlo

| Dato | Valor |
|---|---|
| Marca | **Q-CONNECT** |
| Modelo | Memoria USB de tapa giratoria (*swivel*), cuerpo negro y tapa metálica plateada, sin referencia de modelo visible |
| Capacidad | **4 GB** (impreso en la tapa) |
| N.º de serie | No impreso ni de hardware (`SerialNumber` vacío). Identificador de Windows: `USBSTOR\DISK&VEN_GENERAL&PROD_UDISK&REV_5.00\7&8BEBF63&0&_&0` |
| Otras marcas | Reverso: `Interaction-Connect S.A., 296-298 Route de Longwy, L-1940 Luxembourg LU`, marcados CE y UKCA y símbolo RAEE. Tapa con desgaste y manchas |
| Identificador USB | En `VBox.log` aparece como `proxy abcd:1234` (VID:PID genérico) |
| Fotos | `14a-pendrive-foto-reverso.jpg`, `14b-pendrive-foto-anverso.jpg` |

Contiene `marca-mohamedghayat.txt.txt`, copiado el 2026-11-04 entre las 19:15:41 y las 19:16:46 (según `VBox.log`).

### Bloqueo de escritura

**Herramienta:** `bloqueo-usb1.ps1` (script de la asignatura, `C:\\Users\\Windows\\Downloads\\bloqueo-usb1.ps1`, 9.700 bytes, UTF-8 con BOM) · SHA-256 `150adbc3bee0ee8f8662bee8758e4791fb090c20a48807ac24ae9bea531eef5a` (comprobado en el anfitrión con `Get-FileHash` el 2026-11-10 ≈19:46) · MD5 `5a4cb8014c67d1b95eb5c1662e995cf2` · SHA-1 `f7029cb9c54158147c6b00045341a036b964acd2`.
  - Nota: la copia que se revisó en el análisis llegó sin los 3 bytes del BOM (UTF-8), por eso su hash era distinto (`71c60f42…`). El contenido es idéntico: con el BOM añadido da exactamente el hash del anfitrión.

Funcionamiento (revisado antes de usarlo): "Activar para todos los USB" crea `HKLM\SYSTEM\CurrentControlSet\Control\StorageDevicePolicies\WriteProtect=1`, que pone en solo lectura los USB **que se conecten después**. "Comprobar" intenta escribir un fichero temporal en cada USB con letra. El registro se guarda en `Documentos\bloqueo-usb.log`.

**Precauciones:**
- La política afecta a **todos** los USB, también al de respuesta. Por eso el USB de respuesta se desconecta antes y el E01 del pendrive se guarda en `C:\caso-afi\pendrive\`; después se copia al USB verificando el hash.
- "Comprobar" escribe si el USB no está protegido. Solo se pulsa con la política ya activa y el pendrive conectado después de activarla.

| Hora real | Acción | Resultado |
|---|---|---|
| ≈19:45 | USB de respuesta desconectado; hash del script en el anfitrión | SHA-256 `150ADBC3…31EEF5A` (captura `15a`) |
| ≈19:48 | Script abierto (`powershell -ExecutionPolicy Bypass -File …\bloqueo-usb1.ps1`) | Política desactivada, ningún USB conectado (captura `15b`) |
| 19:49:36 | **Activar para todos los USB** | `Politica global WriteProtect=1`, recuadro verde (captura `15c`) |
| ≈19:51-19:54 | Conexión del pendrive Q-CONNECT | **Windows no lo detecta**: no aparece en `Get-Disk` ni en `Get-PnpDevice` (ni como USB). Se prueba otro puerto |
| ≈19:55-20:05 | Se desconecta el pendrive y se **reinicia el anfitrión** (la política está en el registro y se mantiene) | Tras el reinicio Windows detecta el pendrive. Se conectó **con la política desactivada** (ver la causa abajo) |
| 20:06:53 | Script reabierto → **Activar para todos los USB** otra vez | `Politica global WriteProtect=1`. Lista: `Disco 1 · General UDisk · 3.8 GB · Serie 7 · D: · SOLO LECTURA` (captura `15d`) |
| ≈20:08 | `Get-PnpDevice … USBSTOR` | `USBSTOR\DISK&VEN_GENERAL&PROD_UDISK&REV_5.00\7&8BEBF63&0&_&0`. El pendrive **no tiene número de serie de hardware**: `7&8BEBF63&0` es un identificador generado por Windows (captura `15e`) |
| ≈20:09 | `Get-Disk -Number 1` | `General UDisk`, `SerialNumber` vacío, **4.026.531.840 bytes** (7.864.320 sectores), **`IsReadOnly : True`**, `BusType USB` (captura `15f`) |
| **20:09:47** | **Comprobar** | **`Comprobacion: Disco 1 D: : ESCRIBE -> NO PROTEGIDO`** (captura `15g`) |

> **INCIDENCIA GRAVE — el bloqueo por software no funcionó.** Aunque la política `WriteProtect=1` estaba activa y `Get-Disk` mostraba `IsReadOnly : True`, la comprobación del script pudo escribir en el volumen `D:` del pendrive. **La propia comprobación modificó la evidencia:** creó en la raíz un fichero `.prueba-bloqueo-<guid>.tmp` con el contenido `x` y lo borró a continuación (líneas 74-76 del script), es decir, dejó una entrada de directorio borrada y posiblemente un clúster escrito.
>
> **Causa (confirmada por `bloqueo-usb.log`):** el registro muestra `2026-11-10 19:55:22 Politica global WriteProtect=0`, es decir, **la política se desactivó a las 19:55:22, antes del reinicio**, y no se volvió a activar hasta las 20:06:53. El pendrive se conectó y Windows lo montó **con escritura** en ese intervalo (tras el reinicio, ≈20:05). Al reactivar la política a las 20:06:53 con el pendrive ya conectado, no se aplicó a ese volumen (el propio script avisa: "La politica se aplica al conectar"), así que la comprobación de las 20:09:47 escribió. Se desactivó a propósito antes de reiniciar el anfitrión, porque el pendrive no se detectaba; fue un error de procedimiento, porque había que mantenerla activa durante el reinicio.
>
> **Alcance posible del cambio:** además del `.prueba-bloqueo-*.tmp` creado y borrado por la comprobación, al montar un volumen extraíble con escritura Windows 11 puede crear `System Volume Information\` (por ejemplo `IndexerVolumeGuid` o `WPSettings.dat`) y actualizar fechas de último acceso. En la Parte 3 se buscarán en la imagen las entradas con fecha 2026-11-10 para delimitar exactamente qué cambió.
>
> El registro también contiene una prueba anterior del script, del 2026-11-03 09:07:35-09:07:42 (activar y desactivar sin ningún USB), hecha al prepararlo.
>
> **Corrección:** se desconecta el pendrive **sin expulsarlo** (la expulsión vacía la caché y podría escribir más) y, con la política activa (recuadro verde), se vuelve a conectar para que Windows lo monte ya protegido. (≈20:10-20:18)
>
> En la Parte 3 se localizará en la imagen la entrada borrada `.prueba-bloqueo-*.tmp` para documentar el cambio exacto. La marca (`marca-mohamedghayat.txt.txt`) no se ha tocado.

| Hora real | Acción | Resultado |
|---|---|---|
| **20:18:34** | **Comprobar**, tras la reconexión | **`Comprobacion: Disco 1 D: : no deja escribir -> PROTEGIDO`** ✔ · Disco 1 General UDisk 3.8 GB, D:, SOLO LECTURA (captura `15h-bloqueo-protegido.png`) |

**Bloqueo de escritura comprobado antes de la adquisición: PROTEGIDO (20:18:34).**

### Imagen E01 del pendrive

**Herramienta:** Exterro FTK Imager 8.3.0.27 (anfitrión, como administrador) → File → Create Disk Image → **Physical Drive** → `\\.\PHYSICALDRIVE1` (General UDisk USB Device) → E01 → verificación activada.

Del registro `Imagen_USB.E01.txt`:
- Datos del caso: `Case Number: Caso-01`, `Evidence Number: Ev-06`, `Unique description: Pendrive`, `Examiner: Mohamed Ghayat`.
- Origen: `Source Type: Physical`, `Drive Model: General UDisk USB Device`, `Drive Serial Number:` (vacío), `Drive Interface Type: USB`, `Removable drive: True`, 512 bytes/sector, **7.864.320 sectores** (3.840 MB = 4.026.531.840 bytes).
- **Adquisición: 2026-11-10 20:23:26-20:26:15 · Verificación: 20:26:15-20:26:31** (pendrive protegido desde 20:18:34).
- **MD5 `25ea4b1382ec48f2374087294bdcc084`** y **SHA1 `a8bcaf97c52a681295fe2cacd726ab9670fa0302`**: computed = stored = report → **verified / Match**. `No bad blocks found in image`. Captura `16a-ftk-verificacion-pendrive.png`.
- **Segmentos:** con el *Image Fragment Size* por defecto, FTK Imager partió la imagen en **2 segmentos**, `Imagen_USB.E01` (~1,6 GB) e `Imagen_USB.E02` (~764 MB), verificados en conjunto. Forman una sola imagen y deben conservarse juntos.
- **Incidencia menor:** la imagen se guardó en el escritorio (`C:\Users\Windows\Desktop\Imagen_USB`) en lugar de en `C:\caso-afi\pendrive\`. Se mueve después a la carpeta del caso (mismo disco: el contenido no cambia; se comprueba con el hash).
- FTK Imager generó también en el escritorio un `bitlocker_debug_log.txt` con los metadatos de BitLocker **del disco del anfitrión** (`DESKTOP-G3SK6SL C:`), no del pendrive. Es un fichero auxiliar del equipo del analista, ajeno a las evidencias: **no se incluye** en las evidencias ni en el repositorio.

- **20:29:24:** `Politica global WriteProtect=0` (política desactivada al terminar, según `bloqueo-usb.log`), después de expulsar y retirar el pendrive.
- **≈20:34:** `Move-Item` de `Imagen_USB.E01`, `Imagen_USB.E02` e `Imagen_USB.E01.txt` del escritorio a `C:\caso-afi\pendrive\` (mismo volumen) y copia de `Documentos\bloqueo-usb.log`. Captura `16b-pendrive-hashes-caso.png`.

| Fichero en `C:\caso-afi\pendrive\` | Tamaño (bytes) | SHA-256 |
|---|---|---|
| `Imagen_USB.E01` | 1.572.691.608 | `233cf444e098abe3062e1bf5b6432315fe003aebe7d663ac4101ed8206efb304` |
| `Imagen_USB.E02` | 763.693.798 | `20c75e48c1505d5a2b647e5f9967148f600b38b9f3f5741a785db8a2b62ddcb1` |
| `Imagen_USB.E01.txt` | 2.439 | `b825f85396adf61e77397f96e19830f9350f20bd97aef9db4e91264e1320fc4e` |
| `bloqueo-usb.log` | 941 | `dfb2a025fdb0723800ea7f944a970d7506707ee92a5463a84aa040ca549bbe42` |

- **≈20:36-20:38:** USB de respuesta conectado (`D:`) y copia `Copy-Item C:\caso-afi\pendrive D:\USB-Herramientas-W7\salida\ -Recurse`.
- **≈20:39:** SHA-256 completos de las dos copias guardados con `Get-FileHash … | Format-List Hash, Path | Out-File` en `C:\caso-afi\hashes-pendrive-C.txt` y `C:\caso-afi\hashes-pendrive-USB.txt`. Comparados: **4/4 coinciden** (mismos tamaños y mismos SHA-256). Captura `16c-pendrive-copia-usb-hashes.png`.



# Parte 3 · Búsqueda de la marca (2026-11-10)

Estación de análisis: VM Kali `kali-linux-2026.2-virtualbox-amd64`, con las copias de trabajo en `/mnt/trabajo` (carpeta compartida **de solo lectura**). Distribución de teclado cambiada a español en la Kali (`setxkbmap es`). Las horas de `ls` en la Kali salen en UTC. Captura `19a-kali-ls-trabajo.png`.

| Evidencia | Comando | Resultado | Captura |
|---|---|---|---|
| RAM (`PC-ADMIN-ram.elf`) | `strings -el /mnt/trabajo/PC-ADMIN-ram.elf \| grep "AFI-MohammedGhayat" \| sort \| uniq -c` | `1 AFI-MohammedGhayat-20261104` → **marca encontrada** (texto UTF-16 del Bloc de notas abierto) | `19b-marca-en-ram.png` |
| Triaje (`$MFT`) | `unzip -o /mnt/trabajo/Collection-FORENSE-06-20261104-211820.zip 'uploads/ntfs/*$MFT' -d .` (en `~/afi/triaje`) y `strings -el uploads/ntfs/*/'$MFT' \| grep -i "marca-"` | `marca-mohamedghayat.txt.txt` y su nombre corto `MARCA-~1.TXT` → **marca encontrada**. También `marca-nombreapellido.txt.lnk` / `MARCA-~1.LNK` (acceso directo de Recientes) | `19c-marca-en-triaje-mft.png` |
| Disco (`Imagen-Disco.E01`) | `sudo ewfmount /mnt/trabajo/Imagen-Disco.E01 /mnt/ewf_disco` · `sudo mmls /mnt/ewf_disco/ewf1` (particiones NTFS en los sectores 2048 y **206848**) · `sudo mount -t ntfs-3g -o ro,loop,offset=105906176 /mnt/ewf_disco/ewf1 /mnt/disco` · `ls -la "/mnt/disco/Users/Administrador/Desktop/"` | `marca-mohamedghayat.txt.txt` en el escritorio de `Administrador` (junto a `crea_user.py`, `crea_user.txt` y `desktop.ini`) → **fichero de la marca localizado**. **Tamaño: 0 bytes** (el `cat` no muestra nada) | `19d-disco-ewfmount-mmls.png`, `19e-marca-en-disco.png` |
| Pendrive (`Imagen_USB.E01` + `.E02`) | `sudo ewfmount /mnt/trabajo/Imagen_USB.E01 /mnt/ewf_usb` · `sudo mmls /mnt/ewf_usb/ewf1` (FAT32 en el sector **2048**) · `sudo mount -t vfat -o ro,loop,offset=1048576 /mnt/ewf_usb/ewf1 /mnt/usb` · `ls -la /mnt/usb` | `marca-mohamedghayat.txt.txt` (0 bytes) en la raíz del pendrive → **fichero de la marca localizado**. Además: `.fseventsd`, `.Spotlight-V100` y `.__prueba` (restos de un Mac), la carpeta `_prueba` y `System Volume Information`, todos fechados el 3 de noviembre: es contenido previo del pendrive que prestó el profesor, que no se borró. (Las horas que muestra la Kali están desplazadas por su zona horaria, America/New_York.) | `19h-pendrive-ewfmount-mmls.png`, `19i-marca-en-pendrive.png` |

**Resultado de la Parte 3: la marca aparece en las cuatro adquisiciones**: su texto en la RAM y el nombre del fichero en el triaje (`$MFT`), en la imagen del disco y en la imagen del pendrive.

### Hallazgo: el fichero de la marca se creó el 3 de noviembre y está vacío

`sudo fls -o 206848 -p -r /mnt/ewf_disco/ewf1 | grep -i "marca-"` → entradas MFT **65374** (`Users/Administrador/Desktop/marca-mohamedghayat.txt.txt`) y **67287** (`Users/Administrador/AppData/Roaming/Microsoft/Windows/Recent/marca-nombreapellido.txt.lnk`). `istat -z Europe/Madrid` (horas según el reloj de la VM en ese momento):

| Entrada | Created | Modified | MFT Modified | Tamaño | Observación |
|---|---|---|---|---|---|
| 65374 `marca-mohamedghayat.txt.txt` (8.3: `MARCA~1.TXT`) | **2026-11-03 10:12:44** | 2026-11-03 10:12:44 | $FN 10:20:36 · $SI 10:21:02 | **0** | Propietario SID `…-1000` (Administrador) |
| 67287 `marca-nombreapellido.txt.lnk` (8.3: `MARCA~1.LNK`) | 2026-11-03 10:20:36 | 10:20:36 | 10:20:36 | 622 | Acceso directo de "Recientes": se abrió un fichero llamado `marca-nombreapellido.txt` |

**Interpretación:** el fichero de la marca se creó el **3 de noviembre a las 10:12:44**, al parecer con el nombre del ejemplo del enunciado (`marca-nombreapellido.txt`), se abrió y **se renombró a `marca-mohamedghayat.txt.txt` a las 10:20:36** (a esa hora cambian los `$FILE_NAME` y se crea el `.lnk`), y **nunca se guardó con texto** (0 bytes). El 2026-11-04, durante la adquisición, la marca se escribió en el Bloc de notas (por eso está en la RAM) pero no se volvió a guardar en el fichero, que ya existía: en la primera captura del día 4 (18:08, `capturas/00-…`) ya se ve el icono `marca-…` en el escritorio. El 3 de noviembre, en clase, se creó la marca y se usó el pendrive que prestó el profesor (no se borró su contenido previo: `_prueba`, restos de Mac y `System Volume Information`).

Capturas `19f-disco-istat-marca.png` y `19g-disco-istat-lnk.png`.

# Parte 4 · Cadena de custodia

## Copias de trabajo (imagen maestra y copia de trabajo)

Se hicieron antes de la Parte 3, las copias de trabajo para no tocar nunca las maestras del USB de respuesta.

- **20:46-20:54:** `Copy-Item` de `Imagen-Disco.E01`, `Imagen_USB.E01`, `Imagen_USB.E02`, `PC-ADMIN-ram.elf` y `Collection-FORENSE-06-20261104-211820.zip` desde `D:\USB-Herramientas-W7\salida\` (maestras) a `C:\caso-afi\trabajo\` (copias de trabajo). Tamaños idénticos.
- **Incidencia, ≈20:55-20:58:** al arrancar la VM de análisis Kali (`kali-linux-2026.2-virtualbox-amd64`), VirtualBox **capturó el USB de respuesta** con un filtro USB de esa VM, y Windows lo perdió en mitad del cálculo de hashes (`El volumen para un archivo ha sido alterado externamente` / `No se encuentra la ruta`). Se desconectó de la Kali desde Dispositivos → USB y volvió al anfitrión. La Kali **no llegó a montarlo**: no apareció en su escritorio y se desconectó enseguida.. Comprobaciones:
  - `fsutil dirty query D:` → `El volumen - D: no contiene errores` (NTFS no quedó marcado como sucio). Captura `18a-incidencia-usb-kali-fsutil-dirty.png`.
  - **Integridad de las maestras:** las copias de trabajo se terminaron (20:54) **antes** de la captura. Se recalculan los SHA-256 de maestras y copias (`C:\caso-afi\hashes-trabajo.txt`, ≈21:01):

| Evidencia | SHA-256 maestra (USB) = copia de trabajo | Igual que el hash de adquisición |
|---|---|---|
| `Imagen-Disco.E01` | `4eb3f7b25c5f7577c585bde6857d7bb846ffd78d145aac3c4b813dcfd93d7cf0` ✔ | (primer SHA-256 del fichero; su contenido se verifica con FTK, MD5 `7dc45b11…`) |
| `Imagen_USB.E01` | `233cf444e098abe3062e1bf5b6432315fe003aebe7d663ac4101ed8206efb304` ✔ | ✔ |
| `Imagen_USB.E02` | `20c75e48c1505d5a2b647e5f9967148f600b38b9f3f5741a785db8a2b62ddcb1` ✔ | ✔ |
| `PC-ADMIN-ram.elf` | `795858400b7b6ad4f61abea8613386a27f380c0100db356ad839ebf3474a1ad6` ✔ | ✔ |
| `Collection-…-211820.zip` | `fa6b129cba315e830a80a3cd66a15ced366b718a2dbae1822042aa9a72f0f341` ✔ | ✔ |

  **Conclusión:** las 5 maestras y sus 5 copias de trabajo son idénticas, y coinciden con los hashes tomados al adquirir. La captura del USB por la Kali no alteró ninguna evidencia. Captura `18b-hashes-maestras-copias.png`.
- **Medida:** antes de volver a arrancar la Kali, se desconecta el USB de respuesta del anfitrión (o se quita el filtro USB de la VM). La Kali accede a las copias solo por la carpeta compartida `trabajo` → `C:\caso-afi\trabajo`, en **solo lectura**, automontada en `/mnt/trabajo` (captura `17a-kali-carpeta-compartida-solo-lectura.png`).

## Fichero de hashes, acta, registro y entrega

- **`hashes.sha256`** (2026-11-10 ≈21:55): fichero de hashes de **las 57 evidencias** de `D:\USB-Herramientas-W7\salida\` en formato `sha256sum` (rutas relativas `./…`), guardado en la propia carpeta y copiado en `hashes/hashes.sha256`. Se generó en Linux con `sha256sum` sobre los ficheros leídos del USB; para los 6 ficheros grandes (las imágenes E01/E02, la RAM, el ZIP del triaje y el `.pcap`) se usaron los SHA-256 ya verificados (maestra = copia de trabajo = hash de adquisición). Se comprueba desde la carpeta con:
  ```
  sha256sum -c hashes.sha256
  ```
- **Acta de adquisición** y **registro de custodia:** `acta-adquisicion.pdf` y `registro-custodia.pdf`, pendientes de firmar.
- **Entrega al compañero y devolución (2026-11-10, en este mismo PC, PowerShell en `D:\USB-Herramientas-W7\salida`):**
  - **22:00 · Entrega:** Mohammed Ghayat Zayoun entrega el USB de respuesta (EV-01 a EV-06) a **Eduardo Moreno Rodríguez**.
  - **22:12:19-22:14:04 · Comprobación del receptor:** como Windows no trae `sha256sum`, se recorre `hashes.sha256` con `Get-FileHash -Algorithm SHA256` (solo lectura) comparando cada hash → **`OK: 57   FALLOS: 0`**. Capturas `20a-entrega-companero-hashes-inicio.png` y `20b-entrega-companero-hashes-final.png`.
  - **22:15 · Devolución:** Eduardo Moreno Rodríguez devuelve el USB a Mohammed Ghayat Zayoun.
  - **22:17:57-22:20:45 · Comprobación al recibirlo de vuelta:** mismo procedimiento → **`OK: 57   FALLOS: 0`**. Capturas `20c-devolucion-hashes-inicio.png` y `20d-devolucion-hashes-final.png`.
  - Ambas transferencias quedan anotadas en `registro-custodia.pdf`, para firmar por los dos. Comando usado:
    ```powershell
    Get-Content .\hashes.sha256 | ForEach-Object {
      if ($_ -match '^([0-9a-f]{64}) [ *](.+)$') {
        $h = $matches[1]; $p = $matches[2] -replace '^\./',''
        $c = (Get-FileHash -LiteralPath $p -Algorithm SHA256).Hash.ToLower()
        if ($c -eq $h) { $ok++; "OK     $p" } else { $bad++; "FALLO  $p" }
      }
    }
    ```
- **No se incluye** en el repositorio el `bitlocker_debug_log.txt` que generó FTK Imager, porque contiene metadatos de BitLocker del equipo del analista y no tiene relación con el caso.

## Anexo · Cronología según el registro de VirtualBox (`Logs\VBox.log*`)

Los registros de VirtualBox de la VM (revisados el 2026-11-05 en solo lectura) dan horas **exactas e independientes del reloj de la VM**. Hora local = hora de apertura del registro (UTC+1) + tiempo transcurrido de cada línea.

**Arranques de la VM el 2026-11-04** (cada arranque abre un registro nuevo):

| Registro | Encendido | Apagado | Observaciones |
|---|---|---|---|
| `VBox.log.3` | 17:48:06 | 18:46:37 (apagado desde pausa) | USB Kingston (0951:1666) conectado de 17:50:35 a 18:44:45. Aquí se hicieron `ver` y `wmic os` (18:06-18:08) |
| `VBox.log.2` | 18:48:41 | 18:51:08 (apagado desde pausa) | — |
| `VBox.log.1` | 18:52:33 | No consta (el registro se cortó) | — |
| `VBox.log` | **19:00:28** | **22:05:26** (`poweroff`) | **Sesión de la adquisición** |

> **Incidencia importante:** la VM se encendió y se apagó **tres veces antes** de la sesión en la que se adquirió. Cada arranque y cada apagado escribe en el disco (registros de eventos, Prefetch, pagefile…). Fueron arranques de prueba del analista antes de empezar la adquisición.. La marca y todas las evidencias son de la cuarta sesión (19:00:28-22:05:26).

**Eventos de la sesión de adquisición (`VBox.log`):**

| Hora real | Evento |
|---|---|
| 19:00:28 | Encendido de la VM |
| 19:02:02 | Conexión del USB de respuesta Kingston (USB 0951:1666, entonces en FAT32) |
| 19:15:41 | Conexión del **pendrive encontrado** (USB abcd:1234) para copiar la marca |
| 19:16:46 | Desconexión del pendrive encontrado |
| 19:33:26 / 19:37:58 | Desconexión y reconexión del USB de respuesta |
| **20:04:10** | `TM: Giving up catch-up attempt at a 1 164 879 718 794 ns lag` → **causa del desfase**: la VM estuvo ~19 min 25 s sin recibir tiempo de CPU del anfitrión (VirtualBox detenido o anfitrión suspendido mientras se formateaba el USB) y VirtualBox renunció a recuperar el retraso. A partir de aquí, **reloj de la VM = hora real − 19 min 25 s** |
| 20:04:11 | Desconexión del USB de respuesta (para formatearlo en NTFS) |
| 20:16:57 | Conexión del USB de respuesta ya en NTFS |
| 20:35:56 | `DBGFCoreWrite: Successfully wrote guest core dump 'C:\caso-afi\ram\PC-ADMIN-ram.elf'` (EV-01) |
| 20:58:22 | `NetSniffer: Sniffing to 'C:\caso-afi\red\red.pcap'` (inicio de EV-03) |
| 21:21:38 | Desconexión del USB de respuesta (para añadir Velociraptor 0.6.9) |
| 21:24:19 - 21:25:08 | Reconexión del USB (se conecta, se desconecta y se vuelve a conectar) |
| 22:01:48 | **Desconexión del USB de respuesta antes del apagado** ✔ |
| 22:05:26 | `Console::powerDown()` → `POWERING_OFF` → `OFF` (apagado por corte de corriente) |

Con estas horas se completan los huecos anteriores: la marca se copió al pendrive entre 19:15:41 y 19:16:46; el USB se desconectó para añadir Velociraptor a las 21:21:38 y se reconectó a las 21:25:08; antes del apagado se desconectó a las 22:01:48.

## Anexo · Escrituras inevitables en el equipo investigado

- La marca (Notepad abierto, `marca-mohammedghayat.txt` en el escritorio): excepción del ejercicio.
- Conexión de tres dispositivos USB (pendrive encontrado, USB de respuesta en FAT32 y el mismo USB en NTFS): quedan en el registro (USBSTOR, MountedDevices…).
- Prefetch de los programas ejecutados: `cmd.exe`, `wmic.exe`, `certutil.exe`, `fsutil.exe`, `notepad.exe`, `DumpIt.exe`, `explorer` (navegación por `D:`)…
- DumpIt: clave de servicio y controlador `DumpIt.sys` (§2.1).

## Anexo · Resumen de incidencias

1. Identificación del sistema operativo dentro de la VM antes de dejar la marca (§0.2).
2. Primera medición de hora con un minuto de diferencia entre la VM y el anfitrión (§1).
3. `fsutil` sin permisos de administrador (§1).
4. USB de respuesta en FAT32 y con datos ajenos; formateado en NTFS en el anfitrión (§1).
5. Desfase de ≈20 min de la VM aparecido durante la intervención (§1).
6. Ruta mal escrita por la distribución de teclado (§2.1).
7. DumpIt lanzado con doble clic y luego sin `/OUTPUT`; fallo `0xC0000428` del controlador (§2.1).
8. RAM adquirida desde el hipervisor en lugar de con DumpIt (§2.2).
9. Velociraptor 0.77.3 legacy no arranca por falta del Universal C Runtime; se usa Velociraptor 0.6.9 y hay que desconectar y reconectar el USB para añadirlo (§5).
10. Pequeños errores de escritura por la distribución de teclado (`certuil`, `D;`, `set TMP` mal escrito, `VBoxManage` tecleado en la VM, `controlvm` sin `VBoxManage`, ruta del `.pcap`), todos corregidos (§3, §5, §7).
11. El Notepad con la marca no aparece abierto en la última captura antes del apagado; se cerró en algún momento después del volcado de RAM (20:35), que sí lo recoge (§7).
12. El fichero de la marca se creó el 3 de noviembre y está vacío (0 bytes); el texto de la marca solo está en la RAM (Parte 3).
13. El bloqueo de escritura del pendrive falló en la primera comprobación (20:09:47) y la comprobación escribió un `.tmp`; corregido y comprobado PROTEGIDO a las 20:18:34 (Parte 2).
14. La VM Kali capturó el USB de respuesta al arrancar; no lo montó y las maestras se comprobaron intactas (Parte 4).
