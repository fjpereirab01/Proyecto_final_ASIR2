# 8. Seguridad de la infraestructura

[← Volver al índice](../README.md)

---

| Capa | Medida aplicada |
|---|---|
| **Red** | Tres redes aisladas: LAN, DMZ e intnet_db. Movimiento lateral limitado entre capas. |
| **Firewall** | iptables en el router: solo se reenvía TCP/80 hacia la DMZ. El resto del tráfico se descarta. |
| **MariaDB** | Sin interfaz en DMZ ni LAN. Usuario `rtg_app` con permisos mínimos restringido a `10.0.0.%`. |
| **NFS** | Exportación limitada a `10.0.0.0/24`. Opción `sync` activa. Sin `no_root_squash`. |

---

## 8.1 Segmentación de red

La separación en tres redes independientes limita el movimiento lateral en caso de compromiso de una máquina:

- Si el **balanceador** es comprometido, el atacante no puede llegar directamente a la base de datos (no tiene interfaz en `intnet_db`).
- Si uno de los **servidores web** es comprometido, el atacante solo puede alcanzar el puerto 3306 de MariaDB y el recurso NFS, no el router ni el exterior directamente.
- La máquina **db** no tiene salida a Internet bajo ninguna circunstancia.

---

## 8.2 Filtrado de tráfico con iptables

El router aplica las siguientes reglas:

```bash
# Solo se reenvía tráfico HTTP (puerto 80) hacia el balanceador
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 172.16.0.10:80

# Las VMs internas pueden acceder a Internet para actualizaciones
iptables -t nat -A POSTROUTING -j MASQUERADE
```

El resto de puertos no tienen regla de reenvío, por lo que son descartados por la política por defecto.

---

## 8.3 Aislamiento de MariaDB

El acceso al puerto 3306 está restringido en dos niveles:

1. **Red:** la máquina `db` no tiene interfaz en la DMZ ni en la LAN.
2. **Usuario de BD:** la restricción `@'10.0.0.%'` en el `GRANT` impide conexiones desde fuera de `intnet_db`.

```sql
-- Permisos mínimos: solo operaciones DML, nunca DDL ni SUPER
GRANT SELECT, INSERT, UPDATE, DELETE ON proyecto_asir.* TO 'rtg_app'@'10.0.0.%';
```

---

## 8.4 Seguridad del recurso NFS

```
/var/www/rulethegame  10.0.0.0/24(rw,sync,no_subtree_check)
```

- La exportación se limita a `10.0.0.0/24`: ninguna máquina fuera de esa red puede montarla.
- La opción `sync` garantiza la integridad de los datos escritos.
- **No se usa `no_root_squash`**: el usuario root del cliente se mapea al usuario `nobody` del servidor, evitando escrituras privilegiadas no autorizadas.

---

[← Herramienta de gestión](07-herramienta-gestion.md) | [Siguiente → Guía de despliegue](09-guia-despliegue.md)
