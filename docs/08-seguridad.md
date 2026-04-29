# 8. Seguridad de la infraestructura

[← Volver al índice](../README.md)

---

| Capa | Medida aplicada |
|---|---|
| **Red** | Tres redes aisladas: LAN, DMZ e intnet_db. Movimiento lateral limitado entre capas. |
| **Firewall** | iptables en el router: solo se reenvía TCP/80 y TCP/443 hacia la DMZ. |
| **HTTPS** | Certificado SSL autofirmado en el balanceador Nginx. HTTP redirige automáticamente a HTTPS. |
| **MariaDB** | Sin interfaz en DMZ ni LAN. Usuario `rtg_app` con permisos mínimos restringido a `10.0.0.%`. |
| **NFS** | Exportación limitada a `10.0.0.0/24`. Opción `sync` activa. Sin `no_root_squash`. |
| **Contraseñas** | Cifradas con bcrypt mediante `password_hash()` de PHP. Nunca se almacenan en texto plano. |

---

## 8.1 Segmentación de red

La separación en tres redes independientes limita el movimiento lateral en caso de compromiso de una máquina:

- Si el **balanceador** es comprometido, el atacante no puede llegar directamente a la base de datos.
- Si uno de los **servidores web** es comprometido, el atacante solo puede alcanzar el puerto 3306 de MariaDB y el recurso NFS.
- La máquina **db** no tiene salida a Internet bajo ninguna circunstancia.

---

## 8.2 Filtrado de tráfico con iptables

El router aplica las siguientes reglas:

```bash
# HTTP y HTTPS se redirigen al balanceador
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 80  -j DNAT --to-destination 172.16.0.10:80
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 443 -j DNAT --to-destination 172.16.0.10:443

# Las VMs internas pueden acceder a Internet para actualizaciones
iptables -t nat -A POSTROUTING -j MASQUERADE
```

El resto de puertos no tienen regla de reenvío y son descartados por la política por defecto.

---

## 8.3 HTTPS con certificado autofirmado

El balanceador Nginx cifra todo el tráfico con SSL. El certificado se genera con OpenSSL:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/rulethegame.key \
  -out /etc/ssl/certs/rulethegame.crt \
  -subj "/C=ES/ST=Navarra/L=Pamplona/O=RuleTheGame/CN=192.168.10.1"
```

Nginx redirige automáticamente todo el tráfico HTTP a HTTPS:

```nginx
server {
    listen 80;
    return 301 https://$host$request_uri;
}
```

---

## 8.4 Aislamiento de MariaDB

El acceso al puerto 3306 está restringido en dos niveles:

1. **Red:** la máquina `db` no tiene interfaz en la DMZ ni en la LAN.
2. **Usuario de BD:** la restricción `@'10.0.0.%'` en el `GRANT` impide conexiones desde fuera de `intnet_db`.

```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON proyecto_asir.* TO 'rtg_app'@'10.0.0.%';
```

---

## 8.5 Seguridad del recurso NFS

```
/var/www/rulethegame  10.0.0.21(rw,sync,no_subtree_check,no_root_squash)
                      10.0.0.22(rw,sync,no_subtree_check,no_root_squash)
```

- La exportación se limita a las IPs de `web1` y `web2` exclusivamente.
- La opción `sync` garantiza la integridad de los datos escritos.

---

## 8.6 Seguridad de la aplicación web

- Las contraseñas se almacenan cifradas con **bcrypt** mediante `password_hash()` de PHP.
- Las consultas SQL usan **prepared statements** con `mysqli_prepare()` para evitar inyección SQL.
- Los datos de entrada se sanean con `htmlspecialchars()` antes de mostrarse en el HTML para evitar XSS.
- El acceso al panel de administración está restringido al usuario con `id=1`.

---

[← Herramienta de gestión](07-herramienta-gestion.md) | [Siguiente → Guía de despliegue](09-guia-despliegue.md)
