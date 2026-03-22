# 3. Configuración del router

[← Volver al índice](../README.md)

---

## 3.1 IP Forwarding

Para que el router pueda reenviar paquetes entre interfaces es necesario activar el forwarding de IPv4 en el kernel. Esto se configura en el bloque `provision` del Vagrantfile:

```bash
sysctl -w net.ipv4.ip_forward=1
```

Para hacerlo **persistente entre reinicios**, añadir al fichero `/etc/sysctl.conf`:

```
net.ipv4.ip_forward=1
```

---

## 3.2 Reglas iptables

El router aplica dos reglas NAT fundamentales:

| Regla | Tabla/Cadena | Efecto |
|---|---|---|
| DNAT puerto 80 | nat / PREROUTING | Redirige el tráfico TCP entrante en el puerto 80 hacia el balanceador (172.16.0.10:80) |
| MASQUERADE | nat / POSTROUTING | Enmascara la IP origen permitiendo acceso a Internet desde las VMs internas |

```bash
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 172.16.0.10:80
iptables -t nat -A POSTROUTING -j MASQUERADE
```

> **Persistencia:** Las reglas iptables son volátiles por defecto. Para hacerlas persistentes instalar `iptables-persistent`:
> ```bash
> apt install iptables-persistent
> # Las reglas se guardan en /etc/iptables/rules.v4
> ```

---

[← Arquitectura](02-arquitectura.md) | [Siguiente → Balanceo de carga con Apache](04-balanceo-apache.md)
