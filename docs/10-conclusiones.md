# 10. Conclusiones

[← Volver al índice](../README.md)

---

El proyecto RuleTheGame.com ha permitido aplicar de forma integrada los principales conocimientos adquiridos durante el ciclo ASIR: virtualización con Vagrant y VirtualBox, configuración de redes segmentadas con DMZ, administración de servicios Linux (Apache, MariaDB, NFS, Nginx), seguridad perimetral con iptables, cifrado de tráfico con HTTPS, diseño de bases de datos relacionales y desarrollo de una aplicación web completa en PHP con sistema de roles, mensajería interna y panel de administración.

---

## Puntos destacados

- **Arquitectura escalable:** añadir un tercer servidor web solo requiere un nuevo `server` en el bloque `upstream` de Nginx, sin modificar el router ni la base de datos.
- **NFS centralizado:** el código fuente es idéntico en ambos servidores web en todo momento, eliminando inconsistencias.
- **HTTPS implementado:** el balanceador Nginx cifra todo el tráfico con SSL y redirige automáticamente HTTP a HTTPS, con certificado autofirmado generado con OpenSSL.
- **Afinidad de sesión con ip_hash:** resuelve el problema de pérdida de sesiones PHP en entornos con múltiples servidores web detrás de un balanceador.
- **Aislamiento de BD:** sin interfaz en DMZ, usuario con permisos mínimos, acceso restringido a la red `intnet_db`.
- **Aplicación web completa:** sistema de roles (cliente, coach, admin), contratación y gestión de sesiones, mensajería interna con polling, panel de administración, edición de perfiles y sistema de valoraciones.
- **Herramienta de gestión sin dependencias externas:** reutilizable desde terminal o desde la interfaz gráfica tkinter.
- **Exposición a Internet:** la infraestructura puede exponerse públicamente mediante Cloudflare Tunnel sin necesidad de IP estática ni apertura de puertos en el router.

---

## Mejoras futuras

- Migración del certificado autofirmado a **Let's Encrypt** para eliminar la advertencia de seguridad del navegador.
- Sistema de **pagos integrado** con pasarela de pago real.
- **Monitorización** con Prometheus + Grafana para visualizar métricas del entorno en tiempo real.
- Sistema de **disponibilidad horaria** para coaches, evitando solapamiento de sesiones.
- **Notificaciones por email** al contratar o cancelar una sesión.
- Migración de sesiones PHP a **Redis** compartido para eliminar la dependencia de `ip_hash`.

---

*RuleTheGame.com · TFG ASIR · F.J. Pereira Benito · 2024–2025*

---

[← Guía de despliegue](09-guia-despliegue.md) | [↑ Volver al índice](../README.md)
