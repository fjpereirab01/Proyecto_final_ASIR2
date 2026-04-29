# 3. Configuración del router

[← Volver al índice](../README.md)

---

## 3.1 IP Forwarding

Para que el router pueda reenviar paquetes entre interfaces es necesario activar el forwarding de IPv4 en el kernel:

```bash
sysctl -w net.ipv4.ip_forward=1
```

Para hacerlo **persistente entre reinicios**, añadir al fichero `/etc/sysctl.conf`:

```
net.ipv4.ip_forward=1
```

---

## 3.2 Reglas iptables

El router aplica reglas NAT para redirigir el tráfico HTTP y HTTPS hacia el balanceador:

| Regla | Tabla/Cadena | Efecto |
|---|---|---|
| DNAT puerto 80 (eth1) | nat / PREROUTING | Redirige HTTP desde la LAN al balanceador |
| DNAT puerto 80 (eth0) | nat / PREROUTING | Redirige HTTP desde VirtualBox forwarded_port al balanceador |
| DNAT puerto 443 (eth1) | nat / PREROUTING | Redirige HTTPS desde la LAN al balanceador |
| DNAT puerto 443 (eth0) | nat / PREROUTING | Redirige HTTPS desde VirtualBox forwarded_port al balanceador |
| MASQUERADE | nat / POSTROUTING | Enmascara la IP origen para acceso a Internet desde las VMs internas |

```bash
# HTTP
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80 -j DNAT --to-destination 172.16.0.10:80
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 172.16.0.10:80

# HTTPS
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 443 -j DNAT --to-destination 172.16.0.10:443
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j DNAT --to-destination 172.16.0.10:443

# NAT de salida
iptables -t nat -A POSTROUTING -j MASQUERADE
```

> **Persistencia:** Las reglas iptables son volátiles por defecto. Para hacerlas persistentes:
> ```bash
> apt install iptables-persistent
> netfilter-persistent save
> # Las reglas se guardan en /etc/iptables/rules.v4
> ```

---

## 3.3 Verificación

```bash
# Comprobar reglas activas
iptables -t nat -L -n -v

# La salida debe mostrar las reglas DNAT para los puertos 80 y 443
```

---

[← Arquitectura](02-arquitectura.md) | [Siguiente → Balanceo de carga con Nginx](04-balanceo-apache.md)
