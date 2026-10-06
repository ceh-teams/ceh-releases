# CEH Manager — descargas

Acá se publican los **instaladores y las actualizaciones** de CEH Manager, el sistema de gestión para comités de enlace con hospitales (médicos, instituciones, casos, agenda y asistente de IA).

Este repositorio **solo tiene las descargas**. El código fuente no se publica.

> **¿Primera vez?** Bajá la versión más reciente desde **[Releases](https://github.com/ceh-teams/ceh-releases/releases/latest)**, instalala con los pasos de abajo y, al terminar el asistente, **guardá el kit de recuperación en papel**. Sin la frase maestra ni el código de recuperación, los datos no se pueden recuperar.

## Qué archivo bajar

En cada versión, elegí el que corresponde a tu equipo (`<versión>` es el número, por ejemplo `4.0.1`):

| Tu equipo | Archivo |
|---|---|
| Mac (Apple Silicon: M1, M2…) | `ceh-manager_<versión>_darwin_arm64.tar.gz` |
| Mac (Intel) | `ceh-manager_<versión>_darwin_amd64.tar.gz` |
| Windows (64 bits) | `ceh-manager_<versión>_windows_amd64.zip` |
| Windows en ARM | `ceh-manager_<versión>_windows_arm64.zip` |
| Linux Debian / Ubuntu | `ceh-manager_<versión>_amd64.deb` (o `_arm64`) |
| Linux Fedora / RHEL | `ceh-manager-<versión>-1.x86_64.rpm` (o `.aarch64`) |
| Otro Linux | `ceh-manager_<versión>_linux_amd64.tar.gz` (o `_arm64`) |
| Nube de Amazon (AWS) | `ceh-manager_<versión>_lambda_arm64.zip` |

Si no sabés qué procesador tiene tu Mac: menú  → *Acerca de esta Mac*.

Los instaladores `.pkg` (macOS) y `.msi` (Windows) se agregan a una versión cuando están listos; si no aparecen, usá el archivo comprimido, que hace lo mismo.

## Verificá la descarga

Cada versión trae `ceh-manager_<versión>_checksums.txt` con la huella SHA-256 de cada archivo. Comprobá que la de tu descarga coincida:

- macOS / Linux: `shasum -a 256 archivo`
- Windows (PowerShell): `Get-FileHash archivo -Algorithm SHA256`

Si no coincide, descartá el archivo y volvé a bajarlo.

## Instalar como servicio

El sistema queda funcionando siempre y arranca con el equipo. Solo escucha en el propio equipo (`127.0.0.1:8080`). Al terminar, el instalador muestra un **enlace** para completar el asistente en el navegador.

**macOS / otro Linux** — descomprimí, abrí una terminal en esa carpeta y corré:

```
sudo ./ceh install
```

**Windows** — descomprimí el `.zip`, abrí *Símbolo del sistema* **como administrador**, andá a la carpeta y corré:

```
ceh.exe install
```

**Linux con paquete** (no uses `ceh install` en este caso):

```
sudo apt install ./ceh-manager_<versión>_amd64.deb      # Debian, Ubuntu
sudo dnf install ./ceh-manager-<versión>-1.x86_64.rpm   # Fedora, RHEL
sudo cat /var/lib/ceh-manager/instalar.url               # para volver a ver el enlace
```

Para probarlo un rato sin instalar nada, descomprimí y corré `./ceh` (en Windows, doble clic en `ceh.exe`): aparece un enlace para abrir en el navegador y los datos quedan en tu carpeta de usuario. Si cerrás la ventana, el sistema se detiene.

**Avisos del sistema operativo.** macOS puede decir que no es de un desarrollador identificado: clic derecho sobre el programa → *Abrir*. Windows puede mostrar "Windows protegió su PC": *Más información* → *Ejecutar de todas formas*.

**Antes de cargar datos reales**, activá el **cifrado de disco** del equipo (FileVault en macOS, BitLocker o Cifrado de dispositivo en Windows, LUKS en Linux). El comando `ceh doctor` avisa si falta.

## Actualizar

1. **Generá una copia de seguridad** (*Mantenimiento → Generar copia ahora*) y confirmá que se guardó.
2. Bajá la versión nueva y verificá su huella.
3. Instalala **encima de la anterior** (mismo comando o paquete que usaste). Los datos no se tocan.
4. Desbloqueá el sistema con la frase maestra.

Para saber si hay una versión nueva sin enviar ningún dato de tu comité:

```
ceh update check      # avisa si hay versión nueva y si es de seguridad
ceh update download   # la baja y verifica firma y huella
```

## Seguridad de las actualizaciones

Cada versión incluye dos archivos: `latest.json` (la lista de archivos con su huella SHA-256) y `latest.json.sig` (su **firma digital Ed25519**). El programa solo acepta un `latest.json` firmado con la clave que lleva incorporada, y descarta cualquier archivo cuya huella no coincida. Por eso nadie puede hacerte llegar un programa alterado por este camino.

Si ves una versión marcada **de seguridad**, actualizá cuanto antes.

## En la nube de Amazon (AWS)

El archivo `ceh-manager_<versión>_lambda_arm64.zip` es el programa para AWS Lambda. Quien despliega la pila de CloudFormation necesita su **URL** (la de este repositorio, bajo `releases/download/`) y su **huella SHA-256**, que figura en `latest.json` (artefacto `lambda`). Si la huella no coincide, no se instala nada.

## Datos y privacidad

Cada comité tiene su propia instalación y sus propios datos; el proyecto no tiene acceso a ellos. Este repositorio no recibe información de quienes descargan o usan el programa más allá de lo que GitHub registra en cualquier descarga.
