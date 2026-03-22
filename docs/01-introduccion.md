# 1. Introducción

[← Volver al índice](../README.md)

---

## 1.1 Descripción del proyecto

**RuleTheGame.com** es una plataforma web orientada al ámbito de los videojuegos profesionales cuyo objetivo es conectar a usuarios con coaches especializados en distintos videojuegos, facilitando la contratación de sesiones de entrenamiento personalizado.

La plataforma gestiona dos tipos de usuarios diferenciados:

- **Coach:** profesional que fija su precio por hora, define la duración de cada sesión y establece su horario de disponibilidad.
- **Cliente:** usuario registrado que puede contratar sesiones con cualquier coach disponible siempre que los horarios sean compatibles.

Adicionalmente, la plataforma incorpora un módulo de gestión de incidencias: los usuarios pueden notificar problemas a través de un formulario web y los empleados, según su rol, tienen acceso a esos registros para gestionarlos y resolverlos.

---

## 1.2 Objetivos

- Desplegar una infraestructura virtualizada reproducible mediante Vagrant y VirtualBox.
- Implementar segmentación de red con una zona DMZ y una red de base de datos aislada.
- Configurar balanceo de carga con Apache como proxy inverso entre dos servidores web.
- Compartir los ficheros de la aplicación web mediante NFS desde el servidor de base de datos.
- Diseñar e implementar una base de datos relacional que soporte el modelo de negocio.
- Garantizar la seguridad del tráfico mediante reglas iptables en el router.
- Desarrollar una herramienta gráfica de administración del entorno Docker con tkinter y Bash.

---

[Siguiente → Arquitectura de la infraestructura](02-arquitectura.md)
