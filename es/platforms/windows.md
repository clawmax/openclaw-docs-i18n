

  Resumen de plataformas

  
# Windows (WSL2)

Se recomienda OpenClaw en Windows **mediante WSL2** (se recomienda Ubuntu). La CLI + la Puerta de enlace se ejecutan dentro de Linux, lo que mantiene el entorno de ejecución consistente y hace que las herramientas sean mucho más compatibles (Node/Bun/pnpm, binarios de Linux, habilidades). Windows nativo puede ser más complicado. WSL2 te da la experiencia completa de Linux — un comando para instalar: `wsl --install`. Están planeadas aplicaciones complementarias nativas para Windows.

## Instalar (WSL2)

-   [Primeros pasos](../start/getting-started.md) (usar dentro de WSL)
-   [Instalación y actualizaciones](../install/updating.md)
-   Guía oficial de WSL2 (Microsoft): [https://learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/windows/wsl/install)

## Puerta de enlace

-   [Manual de la puerta de enlace](../gateway.md)
-   [Configuración](../gateway/configuration.md)

## Instalar el servicio de puerta de enlace (CLI)

Dentro de WSL2:

```bash
openclaw onboard --install-daemon
```

O:

```bash
openclaw gateway install
```

O:

```bash
openclaw configure
```

Selecciona **Servicio de puerta de enlace** cuando se te solicite. Reparar/migrar:

```bash
openclaw doctor
```

## Inicio automático de la puerta de enlace antes del inicio de sesión en Windows

Para configuraciones sin cabeza (headless), asegúrate de que toda la cadena de arranque se ejecute incluso cuando nadie inicie sesión en Windows.

### 1) Mantener los servicios de usuario en ejecución sin inicio de sesión

Dentro de WSL:

```bash
sudo loginctl enable-linger "$(whoami)"
```

### 2) Instalar el servicio de usuario de la puerta de enlace de OpenClaw

Dentro de WSL:

```bash
openclaw gateway install
```

### 3) Iniciar WSL automáticamente al arrancar Windows

En PowerShell como Administrador:

```bash
schtasks /create /tn "WSL Boot" /tr "wsl.exe -d Ubuntu --exec /bin/true" /sc onstart /ru SYSTEM
```

Reemplaza `Ubuntu` con el nombre de tu distribución de:

```bash
wsl --list --verbose
```

### Verificar la cadena de inicio

Después de un reinicio (antes del inicio de sesión en Windows), verifica desde WSL:

```bash
systemctl --user is-enabled openclaw-gateway
systemctl --user status openclaw-gateway --no-pager
```

## Avanzado: exponer servicios WSL en la LAN (portproxy)

WSL tiene su propia red virtual. Si otra máquina necesita acceder a un servicio que se ejecuta **dentro de WSL** (SSH, un servidor TTS local o la Puerta de enlace), debes reenviar un puerto de Windows a la IP actual de WSL. La IP de WSL cambia después de los reinicios, por lo que es posible que necesites actualizar la regla de reenvío. Ejemplo (PowerShell **como Administrador**):

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IP not found." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Permitir el puerto a través del Firewall de Windows (una vez):

```
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

Actualizar el portproxy después de reiniciar WSL:

```
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

Notas:

-   SSH desde otra máquina apunta a la **IP del host de Windows** (ejemplo: `ssh user@windows-host -p 2222`).
-   Los nodos remotos deben apuntar a una URL de Puerta de enlace **alcanzable** (no `127.0.0.1`); usa `openclaw status --all` para confirmar.
-   Usa `listenaddress=0.0.0.0` para acceso LAN; `127.0.0.1` lo mantiene solo local.
-   Si quieres que esto sea automático, registra una Tarea Programada para ejecutar el paso de actualización al iniciar sesión.

## Instalación paso a paso de WSL2

### 1) Instalar WSL2 + Ubuntu

Abrir PowerShell (Admin):

```bash
wsl --install
# O elige una distribución explícitamente:
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Reinicia si Windows lo solicita.

### 2) Habilitar systemd (requerido para la instalación de la puerta de enlace)

En tu terminal de WSL:

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

Luego desde PowerShell:

```bash
wsl --shutdown
```

Vuelve a abrir Ubuntu, luego verifica:

```bash
systemctl --user status
```

### 3) Instalar OpenClaw (dentro de WSL)

Sigue el flujo de Primeros pasos de Linux dentro de WSL:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # instala automáticamente las dependencias de la UI en la primera ejecución
pnpm build
openclaw onboard
```

Guía completa: [Primeros pasos](../start/getting-started.md)

## Aplicación complementaria para Windows

Aún no tenemos una aplicación complementaria para Windows. Las contribuciones son bienvenidas si quieres contribuciones para hacerlo realidad.

[Aplicación Linux](./linux.md)[Aplicación Android](./android.md)