# 4. Balanceo de carga con Apache

[← Volver al índice](../README.md)

---

## 4.1 Módulos necesarios

```bash
a2enmod proxy proxy_http proxy_balancer lbmethod_byrequests status
systemctl restart apache2
```

| Módulo | Función |
|---|---|
| `mod_proxy` | Núcleo del sistema de proxy inverso |
| `mod_proxy_http` | Soporte del protocolo HTTP sobre el proxy |
| `mod_proxy_balancer` | Gestión del cluster de servidores backend |
| `mod_lbmethod_byrequests` | Algoritmo round-robin por número de peticiones |
| `mod_status` | Panel de monitorización del balanceador (opcional) |

---

## 4.2 Configuración del VirtualHost

Fichero `/etc/apache2/sites-available/balancer.conf`:

```apache
<VirtualHost *:80>

    <Proxy "balancer://mycluster">
        BalancerMember "http://172.16.0.21"
        BalancerMember "http://172.16.0.22"
    </Proxy>

    ProxyPass        "/" "balancer://mycluster/"
    ProxyPassReverse "/" "balancer://mycluster/"

</VirtualHost>
```

```bash
a2ensite balancer.conf
systemctl reload apache2
```

---

## 4.3 Comportamiento del balanceo

- El algoritmo **byrequests** distribuye las peticiones de forma alternada (round-robin).
- Si uno de los servidores web no responde, Apache lo marca automáticamente como **fuera de servicio** y deja de enviarle tráfico hasta que vuelva a estar disponible.
- Añadir un tercer servidor web solo requiere un nuevo `BalancerMember` en este fichero, sin tocar el router ni la base de datos.

---

[← Configuración del router](03-router.md) | [Siguiente → Servidores web y NFS](05-nfs-servidores-web.md)
