# RuleTheGame.com — Memoria Técnica TFG ASIR

**Administración de Sistemas Informáticos en Red (ASIR) · Curso 2024–2025**  
**Autor:** F.J. Pereira Benito  
**Infraestructura:** Vagrant + VirtualBox · Router · Nginx Balancer · WEB1 · WEB2 · MariaDB + NFS

---

## Índice

1. [Introducción](docs/01-introduccion.md)
2. [Arquitectura de la infraestructura](docs/02-arquitectura.md)
3. [Configuración del router](docs/03-router.md)
4. [Balanceo de carga con Nginx](docs/04-balanceo-apache.md)
5. [Servidores web y NFS](docs/05-nfs-servidores-web.md)
6. [Base de datos](docs/06-base-de-datos.md)
7. [Herramienta de gestión](docs/07-herramienta-gestion.md)
8. [Seguridad](docs/08-seguridad.md)
9. [Guía de despliegue](docs/09-guia-despliegue.md)
10. [Conclusiones](docs/10-conclusiones.md)
11. [Código de la aplicación web](docs/11-codigo-web.md)
12. [Exposición a Internet](docs/12-exposicion_internet.md)

---

## Resumen del proyecto

**RuleTheGame.com** es una plataforma web de coaching para videojuegos profesionales, desarrollada como Trabajo de Fin de Grado del ciclo ASIR. La infraestructura se despliega íntegramente con **Vagrant + VirtualBox** y está compuesta por cinco máquinas virtuales Debian organizadas en tres redes segmentadas.

| VM | Función |
|---|---|
| `router` | Puerta de enlace, NAT, iptables (HTTP + HTTPS) |
| `balancer` | Nginx proxy inverso con round-robin + SSL |
| `web1` / `web2` | Servidores web Apache + PHP (ficheros via NFS) |
| `db` | MariaDB + servidor NFS |

---

## Arquitectura de red

```
Internet
    │
  Router (192.168.10.1 / 172.16.0.1)   ← NAT + iptables (80/443)
    │
  Balancer (172.16.0.10)               ← Nginx proxy inverso + SSL
   ┌──┴──┐
 WEB1   WEB2  (172.16.0.21 / .22)     ← Apache + PHP + NFS client
   └──┬──┘
     DB (10.0.0.100)                   ← MariaDB + NFS server
```

---

## Acceso a la plataforma

| URL | Descripción |
|---|---|
| `https://192.168.10.1` | Acceso local a la plataforma |
| `https://xxxx.trycloudflare.com` | Acceso público vía Cloudflare Tunnel |

---

*RuleTheGame.com · TFG ASIR · F.J. Pereira Benito · 2024–2025*
