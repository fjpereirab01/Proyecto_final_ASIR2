# 4. Balanceo de carga con Nginx

[← Volver al índice](../README.md)

---

## 4.1 Instalación

```bash
apt install nginx -y
systemctl enable nginx
systemctl start nginx
```

---

## 4.2 Configuración del balanceador

Fichero `/etc/nginx/sites-available/balancer.conf`:

```nginx
upstream servidores_web {
    server 172.16.0.21;
    server 172.16.0.22;
}

server {
    listen 80;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    ssl_certificate     /etc/ssl/certs/rulethegame.crt;
    ssl_certificate_key /etc/ssl/private/rulethegame.key;

    location / {
        proxy_pass http://servidores_web;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

```bash
ln -s /etc/nginx/sites-available/balancer.conf /etc/nginx/sites-enabled/balancer.conf
nginx -t
systemctl reload nginx
```

---

## 4.3 Certificado SSL autofirmado

Para habilitar HTTPS se genera un certificado autofirmado con OpenSSL válido por 365 días:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/rulethegame.key \
  -out /etc/ssl/certs/rulethegame.crt \
  -subj "/C=ES/ST=Navarra/L=Pamplona/O=RuleTheGame/CN=192.168.10.1"
```

| Parámetro | Descripción |
|---|---|
| `-x509` | Genera un certificado autofirmado en lugar de una solicitud CSR |
| `-nodes` | No cifra la clave privada con contraseña |
| `-days 365` | Validez del certificado |
| `-newkey rsa:2048` | Genera una clave RSA de 2048 bits |

> El navegador mostrará una advertencia de seguridad al ser autofirmado. En un entorno de producción se utilizaría Let's Encrypt.

---

## 4.4 Comportamiento del balanceo

- Nginx usa el algoritmo **round-robin** por defecto, distribuyendo las peticiones de forma alternada entre `web1` y `web2`.
- Todo el tráfico HTTP entrante en el puerto 80 es redirigido automáticamente a HTTPS (puerto 443).
- Si uno de los servidores web no responde, Nginx lo marca como no disponible y deja de enviarle tráfico.
- Añadir un tercer servidor web solo requiere un nuevo `server` en el bloque `upstream`, sin tocar el router ni la base de datos.

---

## 4.5 Reglas iptables en el router para HTTPS

Para que el tráfico HTTPS llegue al balanceador, el router aplica DNAT también en el puerto 443:

```bash
iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 443 -j DNAT --to-destination 172.16.0.10:443
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j DNAT --to-destination 172.16.0.10:443
```

---

[← Configuración del router](03-router.md) | [Siguiente → Servidores web y NFS](05-nfs-servidores-web.md)
