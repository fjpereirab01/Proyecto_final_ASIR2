# 1. Introducción

[← Volver al índice](../README.md)

---

## 1.1 Descripción del proyecto

**RuleTheGame.com** es una plataforma web orientada al ámbito de los videojuegos profesionales cuyo objetivo es conectar a usuarios con coaches especializados en distintos videojuegos, facilitando la contratación de sesiones de entrenamiento personalizado.

La plataforma gestiona tres tipos de usuarios diferenciados:

- **Cliente:** usuario registrado que puede buscar coaches, contratar sesiones, valorarlos con comentarios y comunicarse con ellos mediante mensajería interna.
- **Coach:** profesional que fija su precio por hora, gestiona sus sesiones, edita su perfil y se comunica con sus clientes.
- **Administrador:** usuario con acceso completo al panel de administración para gestionar usuarios, coaches y sesiones.

---

## 1.2 Funcionalidades de la plataforma

- Registro e inicio de sesión con sistema de roles (`cliente`, `coach`, `admin`).
- Búsqueda y filtrado de coaches por juego y nombre.
- Contratación de sesiones con fecha, hora, duración y notas.
- Gestión de sesiones: el cliente puede cancelarlas, el coach puede marcarlas como completadas.
- Sistema de valoraciones y comentarios sobre coaches.
- Mensajería interna entre clientes y coaches con polling en tiempo real.
- Panel de administración con gestión completa de usuarios, coaches y sesiones.
- Edición de perfil tanto para clientes como para coaches.
- Favicon personalizado y HTTPS con certificado autofirmado.

---

## 1.3 Objetivos

- Desplegar una infraestructura virtualizada reproducible mediante Vagrant y VirtualBox.
- Implementar segmentación de red con una zona DMZ y una red de base de datos aislada.
- Configurar balanceo de carga con Nginx como proxy inverso entre dos servidores web.
- Habilitar HTTPS con certificado SSL autofirmado en el balanceador.
- Compartir los ficheros de la aplicación web mediante NFS desde el servidor de base de datos.
- Diseñar e implementar una base de datos relacional que soporte el modelo de negocio.
- Garantizar la seguridad del tráfico mediante reglas iptables en el router.
- Desarrollar una herramienta gráfica de administración del entorno Docker con tkinter y Bash.
- Desarrollar una aplicación web completa en PHP con sistema de roles, mensajería y panel de administración.

---

[Siguiente → Arquitectura de la infraestructura](02-arquitectura.md)
