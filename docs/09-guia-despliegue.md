# 9. Guía de despliegue

[← Volver al índice](../README.md)

---

## 9.1 Requisitos previos

- VirtualBox 6.1 o superior instalado en el host Windows.
- Vagrant 2.3 o superior instalado en el host Windows.
- ~3 GB de RAM libre (5 × 512 MB = 2,5 GB + overhead del hipervisor).
- Conexión a Internet para la descarga inicial de la box `debian/bullseye64` (~400 MB).
- PowerShell con OpenSSH disponible (incluido en Windows 10/11).

---

## 9.2 Arranque del entorno

Es obligatorio arrancar `db` primero porque `web1` y `web2` montan su sistema de ficheros vía NFS desde ella:

```bash
vagrant up db
vagrant up web1 web2
vagrant up balancer
vagrant up router
```

O todos a la vez (Vagrant respetará el orden del Vagrantfile):

```bash
vagrant up
```

```bash
# Verificar que todas las VMs están corriendo
vagrant status
```

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

## 9.4 Verificar HTTPS

Una vez arrancadas todas las VMs, acceder desde el navegador del host a:

```
https://192.168.10.1
```

El navegador mostrará una advertencia de seguridad por el certificado autofirmado. Aceptar la excepción y verificar que la web carga correctamente.

---

## 9.5 Subir ficheros al servidor

Para actualizar ficheros de la aplicación desde Windows:

```powershell
# Dar permisos de escritura al usuario vagrant (desde la VM db)
sudo chown -R vagrant:vagrant /var/www/rulethegame

# Subir fichero con scp (desde PowerShell)
scp -P 2222 -i ".vagrant/machines/db/virtualbox/private_key" `
  "web/perfil/perfil.php" vagrant@127.0.0.1:/var/www/rulethegame/perfil/perfil.php

# Restaurar permisos para Apache (desde la VM db)
sudo chown -R www-data:www-data /var/www/rulethegame
sudo chmod -R 755 /var/www/rulethegame
sudo chmod -R 1777 /var/www/rulethegame/sessions
```

---

## 9.6 Lanzar la herramienta de gestión Docker

Con el entorno Docker en marcha, ejecutar desde el host:

```bash
python3 gestor_tkinter.py
```

> Requiere que el contenedor `gestor-docker` esté en ejecución y Docker accesible en el host.

---

## 9.7 Parada y destrucción

```bash
vagrant halt        # Suspender todas las VMs (se pueden volver a arrancar)
vagrant destroy -f  # Destruir completamente el entorno (se pierde todo)
```

> **Importante:** `vagrant destroy` elimina las VMs pero no los datos de MariaDB si están en un volumen persistente. Los datos de la BD sobreviven a `vagrant halt` pero no a `vagrant destroy`.

---

[← Seguridad](08-seguridad.md) | [Siguiente → Conclusiones](10-conclusiones.md)
