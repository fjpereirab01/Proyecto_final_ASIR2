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

## 4.2 Certificado SSL autofirmado

Para habilitar HTTPS se genera un certificado autofirmado con OpenSSL válido por 365 días:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/rulethegame.key \
  -out /etc/ssl/certs/rulethegame.crt \
  -subj "/C=ES/ST=Navarra/L=Pamplona/O=RuleTheGame/CN=192.168.10.1"
```

| Parámetro | Descripción |
|---|---|
| `-x509` | Genera un certificado autofirmado |
| `-nodes` | No cifra la clave privada con contraseña |
| `-days 365` | Validez del certificado |
| `-newkey rsa:2048` | Genera una clave RSA de 2048 bits |

---

## 4.3 Configuración del balanceador

Fichero `/etc/nginx/sites-enabled/balancer.conf`:

```nginx
upstream servidores_web {
    ip_hash;
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
nginx -t
systemctl reload nginx
```

---

## 4.4 ip_hash — afinidad de sesión

PHP almacena las sesiones en ficheros locales de cada servidor web. Sin afinidad de sesión, una misma petición podría ir a un servidor diferente en cada request, perdiendo la sesión del usuario.

La directiva `ip_hash` hace que todas las peticiones de una misma IP siempre vayan al mismo servidor web, resolviendo este problema.

```nginx
upstream servidores_web {
    ip_hash;   # ← garantiza que cada IP siempre va al mismo backend
    server 172.16.0.21;
    server 172.16.0.22;
}
```

---

## 4.5 Comportamiento del balanceo

- Todo el tráfico HTTP (puerto 80) se redirige automáticamente a HTTPS (puerto 443).
- Nginx usa round-robin con `ip_hash` para distribuir y mantener sesiones.
- Si uno de los servidores web no responde, Nginx deja de enviarle tráfico automáticamente.
- Añadir un tercer servidor web solo requiere un nuevo `server` en el bloque `upstream`, sin tocar el router ni la base de datos.

---

[← Configuración del router](03-router.md) | [Siguiente → Servidores web y NFS](05-nfs-servidores-web.md)
