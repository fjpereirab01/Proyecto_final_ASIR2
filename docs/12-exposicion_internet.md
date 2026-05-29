# Exposición a Internet del Proyecto Final ASIR — RuleTheGame

## Descripción del proyecto

**RuleTheGame** es una plataforma de coaching gaming desarrollada como proyecto final de ASIR. La infraestructura está desplegada con Vagrant + VirtualBox y consta de 5 máquinas virtuales:

| Máquina | IP | Rol |
|---|---|---|
| router | 192.168.10.1 / 172.16.0.1 | Gateway + DNAT con iptables |
| balancer | 172.16.0.10 | Balanceador de carga (Nginx) |
| web1 | 172.16.0.21 / 10.0.0.21 | Servidor web Apache + PHP |
| web2 | 172.16.0.22 / 10.0.0.22 | Servidor web Apache + PHP |
| db | 10.0.0.100 | Base de datos MariaDB + servidor NFS |

La web se sirve desde la máquina `db` vía NFS y es montada por `web1` y `web2`. El tráfico externo entra por el router, que lo redirige al balanceador, y este distribuye las peticiones entre los dos servidores web.

```
Internet → ngrok → localhost:8443 (host Windows)
        → balancer VM (Nginx, puerto 443)
        → web1 / web2 (Apache + PHP) → db (MariaDB + NFS)
```

---

## Requisitos previos

- Vagrant + VirtualBox instalados en el host Windows
- Cuenta gratuita en [ngrok.com](https://ngrok.com) con authtoken configurado
- ngrok instalado en el host Windows ([ngrok.com/download](https://ngrok.com/download))

---

## Paso 1 — Modificar el Vagrantfile

Añadir un `forwarded_port` en la sección del balancer para exponer el puerto 443 de la VM en el puerto 8443 del host Windows:

```ruby
config.vm.define "balancer" do |lb|
  lb.vm.box      = "debian/bullseye64"
  lb.vm.hostname = "balanceador"
  lb.vm.network "private_network", virtualbox__intnet: "intnet_dmz", ip: "172.16.0.10"
  lb.vm.network "forwarded_port", guest: 443, host: 8443   # ← AÑADIR
  lb.vm.provision "shell", path: "provision_balancer.sh"
end
```

Recargar la VM del balancer para aplicar el cambio:

```bash
vagrant reload balancer
```

---

## Paso 2 — Corregir las URLs internas de la web

Las URLs del código PHP apuntaban a `https://192.168.10.1/...`, una IP solo accesible en la red local de Vagrant. Para que la web funcione desde cualquier dominio externo (ngrok u otro), se convierten a **rutas relativas** eliminando el dominio:

```bash
vagrant ssh db -c "sudo grep -rl 'https://192.168.10.1' /var/www/rulethegame/ | xargs sudo sed -i 's|https://192.168.10.1||g'"
```

Esto convierte por ejemplo:
- `https://192.168.10.1/coaches/coaches.php` → `/coaches/coaches.php`
- `https://192.168.10.1/logout.php` → `/logout.php`

Con rutas relativas, los enlaces funcionan independientemente del dominio desde el que se acceda.

---

## Paso 3 — Configurar ngrok

### 3.1 Registrar el authtoken

Tras crear la cuenta en ngrok.com, obtener el authtoken desde [dashboard.ngrok.com/get-started/your-authtoken](https://dashboard.ngrok.com/get-started/your-authtoken) y registrarlo en el sistema:

```powershell
ngrok config add-authtoken TU_AUTHTOKEN
```

El token se guarda en `C:\Users\<usuario>\AppData\Local\ngrok\ngrok.yml`.

### 3.2 Lanzar el túnel

Con las VMs corriendo, abrir PowerShell y ejecutar:

```powershell
ngrok http https://localhost:8443 --host-header=rewrite
```

La salida mostrará la URL pública generada:

```
Session Status    online
Account           usuario@email.com (Plan: Free)
Region            Europe (eu)
Forwarding        https://xxxx-xxxx-xxxx.ngrok-free.app -> https://localhost:8443
```

La línea `Forwarding` confirma que el túnel está activo y muestra la URL pública.

---

## Paso 4 — Arrancar las máquinas en orden

Es obligatorio arrancar `db` primero porque `web1` y `web2` montan su sistema de ficheros vía NFS desde ella:

```bash
vagrant up db
vagrant up web1 web2
vagrant up balancer
vagrant up router
```

Esperar a que cada comando finalice antes de ejecutar el siguiente.

---

## Paso 5 — Verificar funcionamiento local

Antes de exponer a internet, comprobar que la web carga correctamente desde el host accediendo al puerto forwardeado:

```
https://localhost:8443
```

Si carga RuleTheGame, el port forwarding está correcto.

---

## Paso 6 — Compartir la URL

La URL pública generada (formato `https://xxxx.ngrok-free.app`) es accesible desde cualquier dispositivo con conexión a internet. Al acceder por primera vez, ngrok muestra una pantalla de aviso indicando que el sitio se sirve gratuitamente a través de ngrok. El visitante debe hacer clic en **"Visit Site"** para continuar. Esta pantalla solo aparece una vez por sesión.

> **Importante:** La URL cambia cada vez que se reinicia ngrok. Compartirla justo antes de la demo.

> **Importante:** No cerrar la ventana de PowerShell mientras el túnel deba estar activo.

---

## Resumen — Comandos para cada sesión

```bash
# 1. Arrancar VMs en orden
vagrant up db && vagrant up web1 web2 && vagrant up balancer && vagrant up router

# 2. Verificar que la web carga en local
# Abrir en el navegador: https://localhost:8443

# 3. Lanzar ngrok (en PowerShell)
ngrok http https://localhost:8443 --host-header=rewrite

# 4. Compartir la URL pública que aparece en la terminal
```

---

## Problemas encontrados y soluciones

| Problema | Causa | Solución |
|---|---|---|
| `ERR_NGROK_3200` — endpoint offline | Las VMs no estaban corriendo o ngrok apuntaba a una IP inaccesible | Verificar `vagrant status` y usar `localhost:8443` en lugar de `192.168.10.1` |
| Links de la web no funcionan desde ngrok | Las URLs del PHP apuntaban a `192.168.10.1` (IP local de Vagrant) | Reemplazar todas las URLs absolutas por rutas relativas con `sed` |
| Pantalla de aviso de ngrok al acceder | ngrok muestra advertencia en plan gratuito la primera vez | Es normal, hacer clic en "Visit Site" para continuar |
| Puerto 443 no accesible desde el host | El Vagrantfile no tenía `forwarded_port` para el balancer | Añadir `lb.vm.network "forwarded_port", guest: 443, host: 8443` y recargar |
| Autenticación fallida al lanzar ngrok | No se había configurado el authtoken | Ejecutar `ngrok config add-authtoken TU_AUTHTOKEN` |
