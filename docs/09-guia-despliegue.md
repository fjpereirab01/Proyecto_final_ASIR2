# 9. Guía de despliegue

[← Volver al índice](../README.md)

---

## 9.1 Requisitos previos

- VirtualBox 6.1 o superior instalado en el host.
- Vagrant 2.3 o superior instalado en el host.
- ~3 GB de RAM libre (5 × 512 MB = 2,5 GB + overhead del hipervisor).
- Conexión a Internet para la descarga inicial de la box `debian/bullseye64` (~400 MB).

---

## 9.2 Arranque del entorno

```bash
# Desde el directorio que contiene el Vagrantfile
vagrant up

# Verificar que todas las VMs están corriendo
vagrant status
```

Vagrant descargará automáticamente la box si no está disponible localmente. El primer arranque tarda más por la descarga y el aprovisionamiento.

---

## 9.3 Acceso SSH a las máquinas

```bash
vagrant ssh router
vagrant ssh balancer
vagrant ssh web1
vagrant ssh web2
vagrant ssh db
```

---

## 9.4 Lanzar la herramienta de gestión

Con el entorno en marcha, ejecutar desde el host:

```bash
python3 gestor_tkinter.py
```

> Requiere que el contenedor `gestor-docker` esté en ejecución y Docker accesible en el host.

---

## 9.5 Parada y destrucción

```bash
vagrant halt        # Suspender todas las VMs (se pueden volver a arrancar)
vagrant destroy -f  # Destruir completamente el entorno (se pierde todo)
```

---

[← Seguridad](08-seguridad.md) | [Siguiente → Conclusiones](10-conclusiones.md)
