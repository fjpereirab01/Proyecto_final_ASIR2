# 5. Servidores web y compartición NFS

[← Volver al índice](../README.md)

---

## 5.1 Servidor NFS en la máquina `db`

La máquina `db` exporta el directorio de la aplicación web para que WEB1 y WEB2 sirvan siempre los mismos ficheros. Cualquier actualización del código fuente queda disponible al instante en ambos servidores sin sincronización manual.

**Instalación:**
```bash
apt install nfs-kernel-server
```

**`/etc/exports`:**
```
/var/www/rulethegame  10.0.0.0/24(rw,sync,no_subtree_check)
```

```bash
exportfs -ra
systemctl restart nfs-kernel-server
```

| Opción | Efecto |
|---|---|
| `rw` | Lectura y escritura desde los clientes NFS |
| `sync` | Confirma escrituras en disco antes de responder al cliente |
| `no_subtree_check` | Mejora el rendimiento deshabilitando la verificación de subdirectorios |

---

## 5.2 Montaje NFS en WEB1 y WEB2

**Instalación en cada servidor web:**
```bash
apt install nfs-common
```

**Montaje manual (verificación):**
```bash
mount 10.0.0.100:/var/www/rulethegame /var/www/html
```

**Montaje permanente — `/etc/fstab`** (en cada servidor web):
```
10.0.0.100:/var/www/rulethegame  /var/www/html  nfs  defaults  0 0
```

Con este montaje, independientemente del servidor al que el balanceador dirija cada petición, el cliente siempre recibe la misma versión de la aplicación.

---

## 5.3 Aislamiento de la base de datos

La máquina `db` **únicamente** tiene interfaz en `intnet_db` (10.0.0.0/24):

- No tiene interfaz en la DMZ → el balanceador **no puede** llegar a ella.
- No tiene interfaz en la LAN Host-Only → no tiene salida al exterior.
- Solo WEB1 y WEB2 pueden alcanzar los puertos **3306** (MariaDB) y **2049** (NFS).

---

[← Balanceo de carga con Apache](04-balanceo-apache.md) | [Siguiente → Base de datos](06-base-de-datos.md)
