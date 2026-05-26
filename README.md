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
13. [Errores encontrados y soluciones](docs/13-errores-y-soluciones.md)
14. [Red Windows Server y acceso exclusivo a MariaDB](docs/14-red-windows-mariadb.md)
15. [Sistema de incidencias](docs/15-incidencias.md)

---

## Resumen del proyecto

**RuleTheGame.com** es una plataforma web de coaching para videojuegos profesionales, desarrollada como Trabajo de Fin de Grado del ciclo ASIR. La infraestructura se despliega íntegramente con **Vagrant + VirtualBox** y está compuesta por cinco máquinas virtuales Debian organizadas en tres redes segmentadas.

| VM | IP | Función |
|---|---|---|
| `router` | 192.168.10.1 / 172.16.0.1 | Puerta de enlace, NAT, iptables (HTTP + HTTPS) |
| `balancer` | 172.16.0.10 | Nginx proxy inverso con round-robin + ip_hash + SSL |
| `web1` | 172.16.0.21 / 10.0.0.21 | Apache + PHP (ficheros via NFS) |
| `web2` | 172.16.0.22 / 10.0.0.22 | Apache + PHP (ficheros via NFS) |
| `db` | 10.0.0.100 / 192.168.50.100 | MariaDB + servidor NFS |

---

## Arquitectura de red

```
Internet
    │
  Router (192.168.10.1 / 172.16.0.1)   ← NAT + iptables (80/443)
    │
  Balancer (172.16.0.10)               ← Nginx proxy inverso + SSL + ip_hash
   ┌──┴──┐
 WEB1   WEB2  (172.16.0.21 / .22)     ← Apache + PHP + NFS client
   └──┬──┘
     DB (10.0.0.100 / 192.168.50.100)  ← MariaDB + NFS server
      │
     Windows Pro (192.168.50.20)       ← HeidiSQL (gestión de incidencias)
```

---

## Funcionalidades de la plataforma web

- Sistema de registro e inicio de sesión con roles (`cliente`, `coach`, `admin`)
- Búsqueda y filtrado de coaches por juego y nombre
- Contratación de sesiones con fecha, hora, duración y notas
- Gestión de sesiones: cancelación por el cliente, marcado como completada por el coach
- Sistema de valoraciones y comentarios sobre coaches
- Mensajería interna entre clientes y coaches con polling en tiempo real
- Panel de administración con gestión de usuarios, coaches y sesiones
- Edición de perfil para clientes y coaches
- Sistema de incidencias: creación desde la web, gestión desde HeidiSQL
- HTTPS con certificado autofirmado

---

## Entorno Windows (Active Directory)

El proyecto incluye un entorno Windows integrado con la infraestructura Vagrant:

| VM | IP | Función |
|---|---|---|
| Windows Server 2016 | `192.168.50.10` | Controlador de dominio `rulethegame.com` |
| Windows Pro (PC-JEFE-AC) | `192.168.50.20` | Equipo del jefe de atención al cliente |

El Windows Pro tiene acceso exclusivo a MariaDB vía HeidiSQL para gestionar las incidencias de los usuarios.

---

## Acceso a la plataforma

| URL | Descripción |
|---|---|
| `https://192.168.10.1` | Acceso local desde el host Windows |
| `https://xxxx.trycloudflare.com` | Acceso público vía Cloudflare Tunnel |

---

*RuleTheGame.com · TFG ASIR · F.J. Pereira Benito · 2024–2025*
