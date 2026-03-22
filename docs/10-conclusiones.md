# 10. Conclusiones

[← Volver al índice](../README.md)

---

El proyecto RuleTheGame.com ha permitido aplicar de forma integrada los principales conocimientos adquiridos durante el ciclo ASIR: virtualización con Vagrant y VirtualBox, configuración de redes segmentadas con DMZ, administración de servicios Linux (Apache, MariaDB, NFS), seguridad perimetral con iptables, diseño de bases de datos relacionales y desarrollo de herramientas de administración con Bash y Python.

## Puntos destacados

- **Arquitectura escalable:** añadir un tercer servidor web solo requiere un nuevo `BalancerMember` en Apache, sin modificar el router ni la base de datos.
- **NFS centralizado:** el código fuente es idéntico en ambos servidores web en todo momento, eliminando inconsistencias.
- **Aislamiento de BD:** sin interfaz en DMZ, usuario con permisos mínimos, acceso restringido a la red `intnet_db`.
- **Herramienta de gestión sin dependencias externas:** reutilizable desde terminal o desde la interfaz gráfica.

## Mejoras futuras

- HTTPS con Let's Encrypt en el balanceador.
- Migración a contenedores Docker para reducir el consumo de recursos y mejorar la portabilidad.
- Monitorización con Prometheus + Grafana para visualizar métricas del entorno en tiempo real.

---

*RuleTheGame.com · TFG ASIR · F.J. Pereira Benito · 2024–2025*

---

[← Guía de despliegue](09-guia-despliegue.md) | [↑ Volver al índice](../README.md)
