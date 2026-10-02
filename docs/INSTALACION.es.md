# Guía de instalación de OMEN Space (Español)

Esta guía explica cómo instalar, actualizar y desinstalar OMEN Space en Linux. Describe exactamente lo que hacen los scripts `install.sh` y `setup.sh` del repositorio.

> English: see the [README](../README.md#-quick-install).

## Requisitos

- Un portátil o equipo HP **OMEN**, **Victus** o **Transcend**.
- Linux con **systemd** y **D-Bus**.
- Distribuciones con soporte automático de dependencias: Fedora/RHEL (`dnf`), Debian/Ubuntu (`apt`), Arch Linux y derivadas como CachyOS o Manjaro (`pacman`), y openSUSE (`zypper`).
- Cabeceras del kernel (*kernel headers*) que coincidan con tu kernel en ejecución, necesarias para compilar el módulo DKMS.
- Conexión a internet y permisos de `sudo`.

## Método 1: instalador de una línea (recomendado)

Detecta tu distribución, instala las dependencias, compila todo e instala el módulo del kernel:

```bash
curl -sSL https://raw.githubusercontent.com/yunusemreyl/omen-space/main/install.sh | sudo bash
```

El instalador te pedirá elegir un canal:

| Canal | Descripción |
|-------|-------------|
| **Stable** (recomendado) | Última versión oficial etiquetada. Máxima estabilidad. |
| **Canary** | Últimos cambios de la rama `main`. Funciones nuevas, menos probadas. |

Para saltarte la pregunta y usar Canary directamente:

```bash
curl -sSL https://raw.githubusercontent.com/yunusemreyl/omen-space/main/install.sh | sudo bash -s -- --canary
```

## Método 2: clonar el repositorio

```bash
git clone https://github.com/yunusemreyl/omen-space.git
cd omen-space
sudo ./setup.sh install
```

`setup.sh` acepta tres subcomandos:

| Comando | Qué hace |
|---------|----------|
| `sudo ./setup.sh install` | Instala dependencias, compila e instala OMEN Space (limpia restos del antiguo `omenctl`). |
| `sudo ./setup.sh update` | Hace `git pull`, recompila y reinstala. |
| `sudo ./setup.sh uninstall` | Elimina OMEN Space del sistema por completo. |

### Dependencias que instala el script

| Gestor | Paquetes |
|--------|----------|
| `dnf` (Fedora) | `gcc pkgconf-pkg-config gtk4-devel libadwaita-devel systemd-devel make kernel-devel kernel-headers dbus-devel dkms hidapi-devel` |
| `apt` (Debian/Ubuntu) | `build-essential pkg-config libgtk-4-dev libadwaita-1-dev libsystemd-dev libdbus-1-dev dkms linux-headers-$(uname -r) libhidapi-dev` |
| `pacman` (Arch) | `gcc pkgconf gtk4 libadwaita systemd dbus base-devel dkms hidapi` más las cabeceras de tu kernel (`linux-headers`, `linux-zen-headers`, `linux-lts-headers`, `linux-cachyos-headers`, etc., según el kernel que uses) |
| `zypper` (openSUSE) | `gcc make pkgconfig gtk4-devel libadwaita-devel systemd-devel dbus-1-devel kernel-devel dkms libhidapi-devel` |

Si tu gestor de paquetes no es uno de estos, instala manualmente los equivalentes de `gcc`, `make`, `pkg-config`, GTK4, libadwaita y hidapi (con sus paquetes de desarrollo).

El script también necesita **Rust** (`cargo`). Si no lo encuentra, intenta instalarlo con `rustup` para tu usuario.

## Método 3: Arch Linux (PKGBUILD)

```bash
git clone https://github.com/yunusemreyl/omen-space.git
cd omen-space
makepkg -si
```

## Método 4: NixOS (Flakes)

```bash
nix profile install github:yunusemreyl/omen-space
```

## Qué hace la instalación

1. Instala las dependencias de tu distribución.
2. Elimina servicios del antiguo *OmenCtl* (`omenctl`, `hpm-*`), si existen.
3. Compila el daemon, la CLI, la GUI y la bandeja del sistema.
4. Instala los binarios `omen-cli`, `omen-gui`, `omen-tray` y `omen-overlay` en `/usr/bin`, y el daemon en `/usr/libexec/omen-space`.
5. Crea el grupo del sistema `omen-hw` y **añade a tu usuario** a ese grupo.
6. Recarga las reglas de `udev` y la configuración de D-Bus.
7. Compila e instala el módulo del kernel `hp-omen-extra` con DKMS.
8. Activa e inicia el servicio `omen-space-daemon`.
9. Prueba la conexión con `omen-cli system info`.

### Aviso sobre `power-profiles-daemon`

OMEN Space usa `power-profiles-daemon` para la integración de perfiles térmicos. Si el script detecta software de energía en conflicto (por ejemplo TLP o auto-cpufreq), te lo avisará y te preguntará si quieres continuar. Instalar `power-profiles-daemon` puede hacer que tu gestor de paquetes elimine ese otro software. Si quieres conservarlo, responde que no y resuelve el conflicto manualmente.

## Después de instalar

1. **Cierra sesión y vuelve a entrar** (o reinicia). El instalador te añade al grupo `omen-hw`, pero los grupos solo se aplican en una sesión nueva. Sin esto, la GUI y la CLI no podrán hablar con el daemon.
2. Abre la aplicación desde el menú de aplicaciones o desde la terminal:

   ```bash
   omen-gui
   ```

3. Comprueba que todo responde:

   ```bash
   omen-cli system info
   systemctl status omen-space-daemon
   ```

4. **Idioma:** en *Ajustes → Apariencia e idioma* puedes elegir *Español (ES)*. Con la opción automática, la aplicación usa español si el idioma de tu sistema (`LANG`) empieza por `es`.

## Solución de problemas

| Problema | Qué probar |
|----------|------------|
| `omen-cli` o la GUI no se conectan al daemon | Revisa `systemctl status omen-space-daemon` y `journalctl -u omen-space-daemon -b`. Confirma con `groups` que perteneces a `omen-hw` y, si no, cierra sesión y vuelve a entrar. |
| Falla la compilación del módulo DKMS | Instala las cabeceras del kernel que coincidan con `uname -r` y vuelve a ejecutar `sudo ./setup.sh install`. |
| `cargo` no se encuentra | Instala Rust con [rustup](https://rustup.rs) como usuario normal (sin `sudo`) y repite la instalación. |
| Tu modelo no aparece como compatible | Usa *Ajustes → Solución de problemas y diagnóstico* para generar un informe y adjúntalo a un *issue* en GitHub. |
| Quedan archivos de root en la carpeta clonada | Se debe a haber compilado con `sudo`. Corrige con `sudo chown -R $USER: omen-space`. |

## Actualizar

Con el instalador de una línea, vuelve a ejecutarlo. Con el repositorio clonado:

```bash
cd omen-space
sudo ./setup.sh update
```

## Desinstalar

```bash
cd omen-space
sudo ./setup.sh uninstall
```

Si borraste la carpeta, clona de nuevo el repositorio (`git clone https://github.com/yunusemreyl/omen-space.git`) y ejecuta el comando anterior desde ahí. El desinstalador detiene y desactiva el servicio, elimina los binarios, las reglas de `udev`, la configuración de D-Bus y el módulo DKMS.

---

*OMEN Space es un proyecto independiente y no está afiliado ni respaldado por HP.*
