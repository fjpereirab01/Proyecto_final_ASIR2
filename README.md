# RuleTheGame.com — Memoria Técnica TFG ASIR

**Administración de Sistemas Informáticos en Red (ASIR) · Curso 2024–2025**  
**Autor:** F.J. Pereira Benito  
**Infraestructura:** Vagrant + VirtualBox · Router · Apache Balancer · WEB1 · WEB2 · MariaDB + NFS

---

## Índice

1. [Introducción](docs/01-introduccion.md)
2. [Arquitectura de la infraestructura](docs/02-arquitectura.md)
3. [Configuración del router](docs/03-router.md)
4. [Balanceo de carga con Apache](docs/04-balanceo-apache.md)
5. [Servidores web y NFS](docs/05-nfs-servidores-web.md)
6. [Base de datos](docs/06-base-de-datos.md)
7. [Herramienta de gestión](docs/07-herramienta-gestion.md)
8. [Seguridad](docs/08-seguridad.md)
9. [Guía de despliegue](docs/09-guia-despliegue.md)
10. [Conclusiones](docs/10-conclusiones.md)

---

## Resumen del proyecto

**RuleTheGame.com** es una plataforma web ficticia de coaching para videojuegos profesionales, desarrollada como Trabajo de Fin de Grado del ciclo ASIR. La infraestructura se despliega íntegramente con **Vagrant + VirtualBox** y está compuesta por cinco máquinas virtuales Debian organizadas en tres redes segmentadas.

| VM | Función |
|---|---|
| `router` | Puerta de enlace, NAT, iptables |
| `balancer` | Apache proxy inverso con round-robin |
| `web1` / `web2` | Servidores web Apache (ficheros via NFS) |
| `db` | MariaDB + servidor NFS |

---

*RuleTheGame.com · TFG ASIR · F.J. Pereira Benito · 2024–2025*
