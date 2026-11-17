# Contenedores LXC

Este directorio documenta la arquitectura y distribución de los contenedores LXC desplegados en los nodos Proxmox del homelab.

### Nodo A

| VM/LXC ID | Nombre / Hostname | Servicios | Tipo de Ejecución |
| --- | --- | --- | --- |
| **ID local** | `lxc-servicio-1` | Manga, ROMM, LocalStack | Docker en LXC |
| **ID local** | `lxc-servicio-2` | AdGuard Home | Nativo |
| **ID local** | `lxc-servicio-3` | CouchDB | Nativo |
| **ID local** | `lxc-servicio-4` | Vaultwarden | Nativo |
| **ID local** | `lxc-servicio-5` | OpenGym, Patzer | Docker en LXC |
| **ID local** | `lxc-servicio-6` | Stirling-PDF | Nativo |

### Nodo B

| VM/LXC ID | Nombre / Hostname | Servicios | Tipo de Ejecución |
| --- | --- | --- | --- |
| **ID local** | `lxc-servicio-7` | Ansible | Nativo |
| **ID local** | `lxc-servicio-8` | Uptime Kuma | Nativo |
| **ID local** | `lxc-servicio-9` | Homepage | Nativo |
| **ID local** | `lxc-servicio-10` | IT-Tools | Nativo |
| **ID local** | `lxc-servicio-11` | File Browser | Nativo |

---

## Notas de Configuración

- **Contenedores con Docker (Docker-in-LXC):** Los contenedores que ejecutan múltiples servicios mediante Docker tienen habilitado el soporte de *Nesting* (Anidamiento) y *KeyCTL* en su configuración de Proxmox.
- **Red:** Todos los contenedores se comunican a través de la red interna gestionada mediante la topología VXLAN y Tailscale (detallada en [docs/network.md](../docs/network.md)).
