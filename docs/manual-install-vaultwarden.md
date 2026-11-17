
# Instalación de Vaultwarden en LXC Debian Existente

Esta guía explica los pasos necesarios para instalar y configurar **Vaultwarden** (un servidor ligero compatible con Bitwarden escrito en Rust) dentro de un contenedor LXC de Proxmox que ya cuenta con **Debian** como sistema operativo.

---

## Prerrequisitos

Antes de comenzar con la instalación, asegúrate de cumplir con los siguientes requisitos en tu contenedor LXC:

1. **Acceso Root:** Tener acceso mediante la consola de Proxmox o por SSH al contenedor LXC con privilegios de administrador (`root`).
2. **Actualización del sistema:** El contenedor debe estar actualizado.
```bash
apt update && apt upgrade -y
```

3. **Dependencias necesarias:** Asegúrate de tener instalado herramientas básicas como `curl`, `git`, `openssl` y `argon2` (necesario para el cifrado del token de administración).
```bash
apt install -y curl git openssl argon2 build-essential
```


4. **Rust (Entorno de compilación):** Dado que Vaultwarden se compila desde el código fuente para Debian, se requiere el compilador de Rust. Puedes instalarlo de forma sencilla con:
```bash
curl --proto '=https' --tlsv1.2 -sSf [https://sh.rustup.rs](https://sh.rustup.rs) | sh -s -- -y
source "$HOME/.cargo/env"
```



---

## Pasos de Instalación

Sigue estos pasos en orden para descargar, compilar y configurar Vaultwarden en tu LXC:

### 1. Clonar y compilar Vaultwarden

Descarga el código fuente oficial de la última versión y compílalo en modo release:

```bash
git clone [https://github.com/dani-garcia/vaultwarden.git](https://github.com/dani-garcia/vaultwarden.git) /tmp/vaultwarden-src
cd /tmp/vaultwarden-src
cargo build --features "sqlite,mysql,postgresql" --release
```

*Verification:* El proceso de compilación (`cargo build`) finalizará sin errores y generará el binario en `target/release/vaultwarden`.

### 2. Crear la estructura de directorios y mover el binario

Crea el directorio de la aplicación en `/opt/vaultwarden`, mueve el binario y descarga la interfaz web oficial (`web-vault`):

```bash
mkdir -p /opt/vaultwarden/bin /opt/vaultwarden/data
cp target/release/vaultwarden /opt/vaultwarden/bin/

# Descargar y descomprimir la interfaz web oficial
WV_VERSION=$(curl -s [https://api.github.com/repos/dani-garcia/bw_web_builds/releases/latest](https://api.github.com/repos/dani-garcia/bw_web_builds/releases/latest) | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')
curl -L -o /tmp/web-vault.tar.gz "[https://github.com/dani-garcia/bw_web_builds/releases/download/$](https://github.com/dani-garcia/bw_web_builds/releases/download/$){WV_VERSION}/bw_web_${WV_VERSION}.tar.gz"
tar -xzf /tmp/web-vault.tar.gz -C /opt/vaultwarden/
mv /opt/vaultwarden/bw-web-* /opt/vaultwarden/web-vault
rm /tmp/web-vault.tar.gz
```

*Verification:* Si ejecutas `ls /opt/vaultwarden/`, deberías ver las carpetas `bin/`, `data/` y `web-vault/`.

### 3. Configurar las variables de entorno

Crea un archivo de configuración `.env` en `/opt/vaultwarden/.env` con los parámetros básicos (como el puerto de escucha, por defecto el `8000`):

```env
ROCKET_PORT=8000
ROCKET_ADDRESS=0.0.0.0
WEBSOCKET_ENABLED=true
SIGNUPS_ALLOWED=false
```

### 4. Crear el servicio de systemd

Para que Vaultwarden corra en segundo plano y arranque automáticamente con el contenedor, crea el archivo de servicio:

```bash
cat <<EOF> /etc/systemd/system/vaultwarden.service
[Unit]
Description=Vaultwarden (Unofficial Bitwarden compatible server)
After=network.target

[Service]
User=root
WorkingDirectory=/opt/vaultwarden
ExecStart=/opt/vaultwarden/bin/vaultwarden
Restart=always
EnvironmentFile=/opt/vaultwarden/.env

[Install]
WantedBy=multi-user.target
EOF
```

Recarga el demonio de systemd, habilita e inicia el servicio:

```bash
systemctl daemon-reload
systemctl enable --now vaultwarden
```

*Verification:* Comprueba que el servicio está activo y sin errores ejecutando `systemctl status vaultwarden`. Debería mostrar un estado `active (running)`.

---

¡Listo! Ya puedes acceder a tu instancia de Vaultwarden abriendo el navegador en `http://<IP-DE-TU-LXC>:8000`.
