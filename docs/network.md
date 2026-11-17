
# Arquitectura de Red y Virtualización (Homelab)

Este documento describe la topología de red, la infraestructura física y el entorno virtualizado desplegado sobre Proxmox VE, utilizando redes definidas por software (VXLAN) y acceso remoto seguro mediante VPN.

## Diagrama de Arquitectura

A continuación se muestra una representación lógica y física de la infraestructura utilizando Mermaid:

```mermaid
graph TD
    WAN((☁️ WAN / Internet)) --- Router[🌐 Router]
    Router --- Switch[🖧 Switch No-Administrable]
    
    Switch --- Admin[💻 Equipo Administrador]
    Switch --- Nodo1[🖥️ Nodo Proxmox A]
    Switch --- Nodo2[🖥️ Nodo Proxmox B]

    subgraph VXLAN ["☁️ Red Virtual VXLAN (SUBRED_PRIVADA/CIDR)"]
        direction TB
        
        subgraph PVE1 ["🖥️ Nodo Proxmox"]
            direction TB
            D1["🐳 Docker Stack 1<br/>Romm · Romm BD · LocalStack<br/>Flaveresolver · Kavita · Suwayomi"]
            ADG["🛡️ AdGuard"]
            CDB["🗄️ CouchDB"]
            VW["🔐 Vaultwarden"]
            D2["🐳 Docker Stack 2<br/>Patzer · opengym-api · opengym-web"]
            SPDF["📄 Stirling-PDF"]
        end
        
        subgraph PVE2 ["🖥️ Nodo Proxmox B"]
            direction TB
            OWRT["📡 OpenWRT (VM)<br/>Router Virtualizado<br/>--advertise-routes=SUBRED_PRIVADA/CIDR"]
            ANS["⚙️ Ansible"]
            UTK["📈 Uptime Kuma"]
            HP["🏠 Homepage"]
            ITT["🛠️ IT-Tools"]
            FB["📂 FileBrowser"]
        end
    end

    Nodo1 -.- PVE1
    Nodo2 -.- PVE2
    Admin -. "🔒 Túnel VPN" .- OWRT
    OWRT --- D1
    OWRT --- D2
    OWRT --- ADG
    OWRT --- CDB
    OWRT --- VW
    OWRT --- SPDF
    OWRT --- ANS
    OWRT --- UTK
    OWRT --- HP
    OWRT --- ITT
    OWRT --- FB

    ANS -. "💾 Backup" .-> CDB
    ANS -. "💾 Backup" .-> VW

    style WAN fill:#f9f9f9,stroke:#333,stroke-width:2px
    style VXLAN fill:#eef7ff,stroke:#0066cc,stroke-width:2px,stroke-dasharray: 5 5
    style PVE1 fill:#fff5eb,stroke:#e67e22,stroke-width:1px
    style PVE2 fill:#fff5eb,stroke:#e67e22,stroke-width:1px
    class Ansible backup;

```

## Componentes de la Infraestructura

### 1. Capa Física y Acceso a Internet

* **WAN y router:** Punto de entrada y salida hacia Internet. Gestiona la conexión del proveedor de servicios y realiza el NAT inicial hacia la red local.
* **Switch No-Administrable:** Dispositivo de capa 2 que interconecta los equipos físicos (Nodos Proxmox y el equipo del Administrador) con el router principal sin necesidad de configuración de VLANs físicas en el hardware.
* **Equipo Administrador:** Estación de trabajo desde la cual se gestiona la infraestructura. Aunque está conectada físicamente al mismo switch, utiliza un túnel VPN para alcanzar directamente las subredes internas aisladas.

### 2. Capa de Virtualización (Clúster Proxmox)

La infraestructura de cómputo se divide en dos servidores físicos ejecutando Proxmox VE:

* **Nodo Proxmox A:** Aloja cargas de trabajo y almacenamiento compartido.
* **Nodo Proxmox B:** Aloja servicios de enrutamiento virtual y proporciona redundancia/distribución de recursos.

### 3. Red Virtual (SDN - VXLAN)

Para aislar el tráfico de los servicios internos de la red física doméstica (y evitar depender de un switch administrable), se utiliza una red superpuesta (*Overlay Network*):

* **Subred VXLAN (`SUBRED_PRIVADA/CIDR`):** Extiende una red de capa 2 sobre la red física de capa 3 entre ambos nodos Proxmox, permitiendo que las máquinas virtuales se comuniquen como si estuvieran en el mismo switch físico. Sustituye el marcador por la subred que configures localmente.
