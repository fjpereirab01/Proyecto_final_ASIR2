# 2. Arquitectura de la infraestructura

[← Volver al índice](../README.md)

---

## 2.1 Visión general

El tráfico del exterior llega al **router**, que aplica NAT y redirige las peticiones HTTP al **balanceador** en la DMZ. El balanceador distribuye entre **WEB1** y **WEB2** mediante proxy inverso. Ambos servidores montan por NFS los ficheros de la aplicación desde el servidor **db**, que también aloja MariaDB. La base de datos no tiene conectividad con el exterior.

```
Internet
    │
  Router (192.168.10.1 / 172.16.0.1)   ← NAT + iptables
    │
  Balancer (172.16.0.10)               ← Apache proxy inverso
   ┌──┴──┐
 WEB1   WEB2  (172.16.0.21 / .22)     ← Apache + NFS client
   └──┬──┘
     DB (10.0.0.100)                   ← MariaDB + NFS server
```

---

## 2.2 Inventario de máquinas virtuales

| VM | Hostname | Box | RAM / CPU | Función |
|---|---|---|---|---|
| `router` | router-asir | debian/bullseye64 | 512 MB / 1 | Puerta de enlace, NAT, iptables |
| `balancer` | balanceador | debian/bullseye64 | 512 MB / 1 | Apache proxy inverso, round-robin |
| `web1` | web1 | debian/bullseye64 | 512 MB / 1 | Servidor web Apache (monta NFS) |
| `web2` | web2 | debian/bullseye64 | 512 MB / 1 | Servidor web Apache (monta NFS) |
| `db` | database | debian/bullseye64 | 512 MB / 1 | MariaDB + servidor NFS |

---

## 2.3 Segmentación de red

| Red | Tipo Vagrant | Rango IP | Máquinas |
|---|---|---|---|
| Host-Only (LAN) | `private_network` | `192.168.10.0/24` | Router (.1) |
| intnet_dmz (DMZ) | `internal network` | `172.16.0.0/24` | Router (.1), Balancer (.10), WEB1 (.21), WEB2 (.22) |
| intnet_db (DB) | `internal network` | `10.0.0.0/24` | WEB1 (.21), WEB2 (.22), DB (.100) |

La separación en tres redes garantiza que:
- El balanceador **no puede** acceder directamente a la base de datos.
- La base de datos **no tiene** salida a Internet bajo ninguna circunstancia.
- Solo WEB1 y WEB2 tienen acceso simultáneo a la DMZ y a la red de base de datos.

---

[← Introducción](01-introduccion.md) | [Siguiente → Configuración del router](03-router.md)
