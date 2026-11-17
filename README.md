# Homelab

Repositorio de configuraciones y documentación para un homelab basado en Proxmox VE. Reúne ejemplos de despliegue con Docker Compose, tareas de automatización con Ansible y guías operativas para servicios en contenedores LXC.

Los archivos que contienen valores locales se excluyen de Git. Los ejemplos usan marcadores: adapta la configuración a tu entorno antes de ejecutar los servicios.

## Contenido

| Ruta | Descripción |
| --- | --- |
| `ansible/` | Configuración de Ansible, ejemplo de inventario dinámico de Proxmox y tareas de backup. |
| `docker/` | Stacks de Docker Compose para los servicios del homelab. |
| `docs/` | Documentación de red, instalación de Vaultwarden y configuración de SSH en Alpine Linux. |
| `lxc/` | Inventario y notas sobre la distribución de servicios en contenedores LXC. |

## Stacks de Docker Compose

Ejecuta los comandos desde la raíz del repositorio. Las rutas de datos relativas a cada Compose se crean junto al archivo correspondiente.

| Stack | Servicios | Puertos publicados |
| --- | --- | --- |
| `docker/romm/` | RomM y MariaDB | `80` |
| `docker/localstack/` | LocalStack para servicios de AWS en desarrollo | `4566`, `4510-4559` |
| `docker/manga-stack/` | Suwayomi, Kavita y FlareSolverr | `4567`, `5000`, `8191` |
| `docker/open-gym/` | API, web y descarga inicial de material de ejercicios | Web publicada: `WEB_PORT` (por defecto `8080`); API: `3000` interno |
| `docker/patzer/` | Patzer | `8800` |

Por ejemplo, para iniciar RomM:

```bash
docker compose -f docker/romm/docker-compose.yml up -d
```

Sustituye `up -d` por `down` para detener el stack. Repite el comando con la ruta del Compose que quieras administrar.

### Variables de entorno

1. Copia `.env.example` a `.env` en la raíz (`cp .env.example .env` en Linux/macOS o `Copy-Item .env.example .env` en PowerShell).
2. Edita `.env` y sustituye los valores de ejemplo. RomM requiere `ROMM_DB_USER`, `ROMM_DB_PASSWORD`, `ROMM_DB_ROOT_PASSWORD` y un `ROMM_AUTH_SECRET_KEY` único y aleatorio. Las claves de IGDB y SteamGridDB son opcionales.
3. Compose carga el `.env` raíz para interpolar variables al ejecutar los comandos desde la raíz del proyecto. No publiques ese archivo.

Open Gym declara además `docker/open-gym/.env` como archivo de entorno para su API. Es independiente del `.env` raíz; créalo localmente con las variables que requiera la aplicación. No se incluye una plantilla de esas variables en este repositorio.

LocalStack monta el socket de Docker del host para poder crear recursos que lo necesitan. Ese acceso puede dar control amplio sobre el host: no expongas el servicio a redes no confiables.

## Automatización con Ansible

La configuración de `ansible/` usa el plugin dinámico `community.general.proxmox`. Instala la colección con:

```bash
ansible-galaxy collection install community.general
```

`ansible/inventory.example` es la plantilla del inventario. `ansible/ansible.cfg` espera un archivo local llamado `proxmox.proxmox.yml`; copia la plantilla con:

```bash
cp ansible/inventory.example ansible/proxmox.proxmox.yml
```

El inventario y las tareas leen del entorno `PROXMOX_HOST`, `PROXMOX_API_USER`, `PROXMOX_TOKEN_ID` y `PROXMOX_TOKEN_SECRET`. Las tareas de backup también usan `PROXMOX_BACKUP_STORAGE`, `PROXMOX_BACKUP_NOTIFY_UPTIME_KUMA` y, si se habilitan las notificaciones, `UPTIME_KUMA_PUSH_URL`. Las variables de `.env.example` son una referencia: Ansible no carga automáticamente el `.env` de Docker Compose; define esos valores en el entorno del proceso que ejecuta Ansible.

Las tareas ejecutan `vzdump` con los valores configurados para modo, compresión y almacenamiento, y conservan las dos últimas copias. La notificación a Uptime Kuma es opcional. El repositorio contiene las tareas, pero no un playbook completo que las invoque; intégralas en un playbook acorde a tu inventario y permisos.

## Documentación

- [Arquitectura de red](docs/network.md): descripción general de la red virtual y los componentes.
- [Contenedores LXC](lxc/README.md): distribución documentada de servicios.
- [Instalación de Vaultwarden](docs/manual-install-vaultwarden.md): guía para instalarlo en un LXC Debian existente.
- [SSH en Alpine Linux](docs/ssh-alpine.md): instalación y configuración de OpenSSH.
