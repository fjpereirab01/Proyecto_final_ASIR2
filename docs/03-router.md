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

| Regla | Interfaz | Puerto | Efecto |
|---|---|---|---|
| DNAT | eth1 | 80 | Redirige HTTP desde la LAN al balanceador |
| DNAT | eth0 | 80 | Redirige HTTP desde VirtualBox forwarded_port al balanceador |
| DNAT | eth1 | 443 | Redirige HTTPS desde la LAN al balanceador |
| DNAT | eth0 | 443 | Redirige HTTPS desde VirtualBox forwarded_port al balanceador |
| MASQUERADE | * | * | Permite acceso a Internet desde las VMs internas |

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

---

## 3.3 Persistencia de reglas

Las reglas iptables son volátiles por defecto. Para hacerlas persistentes entre reinicios:

```bash
apt install iptables-persistent
netfilter-persistent save
# Las reglas se guardan en /etc/iptables/rules.v4
```

> **Nota importante:** El script de provisión del Vagrantfile limpia las reglas con `iptables -F` en cada arranque. Las reglas añadidas manualmente (especialmente las de `eth0`) deben volver a aplicarse tras cada `vagrant up` o almacenarse en el script de provisión.

---

## 3.4 Verificación

```bash
sudo iptables -t nat -L -n -v
```

La salida debe mostrar las cuatro reglas DNAT (dos para el puerto 80 y dos para el 443) y la regla MASQUERADE.

---

[← Arquitectura](02-arquitectura.md) | [Siguiente → Balanceo de carga con Nginx](04-balanceo-apache.md)
