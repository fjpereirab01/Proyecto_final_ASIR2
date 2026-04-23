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
Internet → Cloudflare Tunnel → localhost:80 (host Windows)
        → router VM (iptables DNAT) → balancer (Nginx)
        → web1 / web2 (Apache + PHP) → db (MariaDB + NFS)
```

---

## Requisitos previos

- Vagrant + VirtualBox instalados en el host Windows
- Fichero `hosts` de Windows configurado (ver paso 2)
- iPhone u otro dispositivo con datos móviles para hacer de hotspot (necesario para saltar las restricciones de red del instituto)
- Ejecutable `cloudflared-windows-amd64.exe` descargado desde [github.com/cloudflare/cloudflared/releases](https://github.com/cloudflare/cloudflared/releases)

---

## Paso 1 — Modificar el Vagrantfile

Añadir un `forwarded_port` en la sección del router para exponer el puerto 80 de la VM en el puerto 80 del host Windows:

```ruby
config.vm.define "router" do |router|
  router.vm.box      = "debian/bullseye64"
  router.vm.hostname = "router-asir"
  router.vm.network "private_network", ip: "192.168.10.1"
  router.vm.network "private_network", virtualbox__intnet: "intnet_dmz", ip: "172.16.0.1"
  router.vm.network "forwarded_port", guest: 80, host: 80   # ← AÑADIR
  router.vm.provision "shell", path: "provision_router.sh"
end
```

---

## Paso 2 — Configurar el fichero hosts de Windows

La web tiene enlaces internos que apuntan a `http://192.168.10.1` (IP del router). Para que el navegador del host pueda resolver esa IP correctamente, hay que añadirla al fichero hosts.

1. Abrir **Bloc de notas como administrador**
2. Abrir el fichero: `C:\Windows\System32\drivers\etc\hosts`
3. Cambiar el tipo de archivo a **"Todos los archivos"** si no aparece
4. Añadir al final:

```
127.0.0.1    192.168.10.1
```

5. Guardar con **Ctrl+S**

---

## Paso 3 — Arrancar las máquinas en orden

Es obligatorio arrancar `db` primero porque `web1` y `web2` montan su sistema de ficheros vía NFS desde ella:

```bash
vagrant up db
vagrant up web1 web2
vagrant up balancer
vagrant up router
```

Esperar a que cada comando finalice antes de ejecutar el siguiente.

---

## Paso 4 — Añadir regla iptables en eth0 del router

El script de provisión del router solo añade la regla DNAT para `eth1`. Como el tráfico del `forwarded_port` de VirtualBox llega por `eth0`, hay que añadir la regla manualmente:

```bash
vagrant ssh router -c "sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 172.16.0.10:80 && sudo netfilter-persistent save"
```

> **Nota:** Esta regla se debe añadir cada vez que se reinicie la VM del router, ya que `netfilter-persistent` la guarda pero el script de provisión vuelve a limpiar las reglas con `iptables -F`.

### Verificar que las reglas están activas

```bash
vagrant ssh router -c "sudo iptables -t nat -L -n -v"
```

La salida debe mostrar dos reglas DNAT: una para `eth1` y otra para `eth0`, ambas redirigiendo al balanceador (`172.16.0.10:80`).

---

## Paso 5 — Verificar funcionamiento local

Antes de exponer a internet, comprobar que la web carga correctamente desde el host:

```
http://192.168.10.1
```

Si carga el RuleTheGame, todo está correcto.

---

## Paso 6 — Conectar el iPhone como hotspot

La red del instituto bloquea los puertos necesarios para Cloudflare Tunnel (UDP para QUIC y TCP 7844 para HTTP2). Para evitarlo:

1. Activar el **Punto de Acceso Personal** en el iPhone
2. Conectar el PC Windows a esa red WiFi
3. Verificar que hay conexión a internet desde el PC

---

## Paso 7 — Lanzar Cloudflare Tunnel

Abrir PowerShell en la carpeta donde está el ejecutable y ejecutar:

```powershell
.\cloudflared-windows-amd64.exe tunnel --protocol http2 --url http://127.0.0.1:80
```

> Se usa `127.0.0.1` en lugar de `localhost` para forzar IPv4, ya que VirtualBox solo escucha en IPv4 y `localhost` en Windows puede resolver a IPv6 (`::1`).

> Se usa `--protocol http2` en lugar de QUIC porque la red del instituto bloquea el tráfico UDP.

La salida mostrará la URL pública generada:

```
+--------------------------------------------------------------------------------------------+
|  Your quick Tunnel has been created! Visit it at (it may take some time to be reachable):  |
|  https://xxxx-xxxx-xxxx-xxxx.trycloudflare.com                                             |
+--------------------------------------------------------------------------------------------+
...
INF Registered tunnel connection connIndex=0 ... location=mad05 protocol=http2
```

La línea `Registered tunnel connection` confirma que el túnel está activo.

---

## Paso 8 — Compartir la URL

La URL pública generada (formato `https://xxxx.trycloudflare.com`) es accesible desde cualquier dispositivo con conexión a internet: otros equipos del instituto, móviles, etc.

> **Importante:** La URL cambia cada vez que se reinicia el túnel. Compartirla justo antes de la demo.

> **Importante:** No cerrar la ventana de PowerShell mientras el túnel deba estar activo.

---

## Resumen — Comandos para cada sesión

```bash
# 1. Arrancar VMs en orden
vagrant up db && vagrant up web1 web2 && vagrant up balancer && vagrant up router

# 2. Añadir regla iptables en eth0 del router
vagrant ssh router -c "sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 172.16.0.10:80 && sudo netfilter-persistent save"

# 3. Verificar que la web carga en local
# Abrir en el navegador: http://192.168.10.1

# 4. Conectar el iPhone como hotspot

# 5. Lanzar el túnel (en PowerShell)
.\cloudflared-windows-amd64.exe tunnel --protocol http2 --url http://127.0.0.1:80
```

---

## Problemas encontrados y soluciones

| Problema | Causa | Solución |
|---|---|---|
| `localhost:8080` da página vacía | El HTML tiene enlaces a `192.168.10.1` que el navegador no puede resolver | Añadir `127.0.0.1 192.168.10.1` al fichero hosts y usar puerto 80 |
| Cloudflare Tunnel no conecta (timeout QUIC) | El instituto bloquea UDP | Usar `--protocol http2` |
| Cloudflare Tunnel no conecta (timeout TCP) | El instituto bloquea el puerto 7844 | Usar el iPhone como hotspot |
| 502 Bad Gateway en la URL de Cloudflare | `localhost` resuelve a IPv6 pero VirtualBox escucha en IPv4 | Usar `http://127.0.0.1:80` en lugar de `http://localhost:80` |
| DNAT solo funciona desde dentro de la red | La regla iptables solo aplica a `eth1` | Añadir regla adicional para `eth0` manualmente |
