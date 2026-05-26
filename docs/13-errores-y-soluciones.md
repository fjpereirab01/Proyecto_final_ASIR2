# 13. Errores encontrados y soluciones

[← Volver al índice](../README.md)

Este documento recoge los principales errores encontrados durante el desarrollo e implantación del proyecto, junto con su causa raíz y la solución aplicada.

---

## 13.1 Infraestructura y red

### Error: La tabla `sesiones` no existía en la BD

**Síntoma:** La sección "Mis sesiones" no aparecía en el perfil del cliente.

**Causa:** La tabla `sesiones` no había sido creada en MariaDB. El PHP ejecutaba una query contra una tabla inexistente y fallaba silenciosamente.

**Solución:**
```sql
CREATE TABLE sesiones (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cliente_id INT NOT NULL,
    coach_id INT NOT NULL,
    fecha DATETIME NOT NULL,
    duracion INT DEFAULT 60,
    estado ENUM('pendiente','completada','cancelada') DEFAULT 'pendiente',
    notas TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (cliente_id) REFERENCES usuarios(id),
    FOREIGN KEY (coach_id) REFERENCES coaches(id)
);
```

---

### Error: Forbidden al acceder a la web tras cambiar permisos con chown

**Síntoma:** `Forbidden — You don't have permission to access this resource. Server unable to read htaccess file`

**Causa:** Al ejecutar `sudo chown -R vagrant:vagrant /var/www/rulethegame` para poder subir ficheros con scp, Apache perdía los permisos de lectura sobre los ficheros.

**Solución:** Siempre restaurar los permisos después de subir ficheros:
```bash
sudo chown -R www-data:www-data /var/www/rulethegame
sudo chmod -R 755 /var/www/rulethegame
```

---

### Error: Pérdida de sesión PHP entre peticiones (balanceador)

**Síntoma:** Al contratar una sesión o enviar mensajes, la operación fallaba aleatoriamente porque `$_SESSION['usuario_id']` aparecía vacío.

**Causa:** El balanceador Nginx distribuía las peticiones en round-robin entre WEB1 y WEB2. Si la sesión se había creado en WEB1, una petición redirigida a WEB2 no podía leerla.

**Solución:** Añadir `ip_hash` al bloque `upstream` de Nginx para que cada IP siempre vaya al mismo servidor:
```nginx
upstream servidores_web {
    ip_hash;
    server 172.16.0.21;
    server 172.16.0.22;
}
```

---

### Error: Las reglas iptables para el puerto 443 no estaban configuradas

**Síntoma:** Al acceder a `https://192.168.10.1` el navegador no podía conectar.

**Causa:** El script de provisión del router solo añadía DNAT para el puerto 80. El puerto 443 no tenía regla de reenvío.

**Solución:**
```bash
sudo iptables -t nat -A PREROUTING -i eth1 -p tcp --dport 443 -j DNAT --to-destination 172.16.0.10:443
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j DNAT --to-destination 172.16.0.10:443
sudo netfilter-persistent save
```

---

### Error: Permission denied en la carpeta sessions

**Síntoma:** `session_start(): open(/var/www/html/sessions/sess_XXX, O_RDWR) failed: Permission denied`

**Causa:** La carpeta `sessions/` tenía permisos incorrectos que impedían a Apache crear y leer los ficheros de sesión.

**Solución:**
```bash
sudo chmod 1777 /var/www/html/sessions
sudo chmod 1777 /var/www/rulethegame/sessions
```

---

### Error: Las reglas FORWARD del router bloqueaban el tráfico HTTP/HTTPS

**Síntoma:** La web dejó de ser accesible desde el host. El curl a `192.168.10.1` daba timeout aunque el ping funcionaba y las reglas DNAT estaban correctas.

**Causa:** La política por defecto de la cadena FORWARD era `DROP`. Las reglas DNAT redirigían el tráfico correctamente pero las reglas FORWARD no permitían explícitamente el paso entre `eth1` (LAN) y `eth2` (DMZ), por lo que los paquetes eran descartados después del DNAT.

**Solución:** Añadir reglas FORWARD explícitas en el router:
```bash
sudo iptables -A FORWARD -i eth1 -o eth2 -j ACCEPT
sudo iptables -A FORWARD -i eth2 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo netfilter-persistent save
```

Estas reglas se añadieron también al script `provision_router.sh` para que persistan tras reinicios.

---

### Error: MariaDB no aceptaba conexiones desde la red 192.168.50.x

**Síntoma:** HeidiSQL desde Windows Pro no podía conectar a `192.168.50.100:3306`. El ping funcionaba pero la conexión TCP al puerto 3306 era rechazada.

**Causa:** El `bind-address` de MariaDB estaba fijado a `10.0.0.100`, por lo que solo escuchaba en la interfaz de la red Vagrant y rechazaba conexiones desde la interfaz `eth2` (`192.168.50.100`).

**Solución:** Cambiar el `bind-address` a `0.0.0.0` para escuchar en todas las interfaces:
```bash
sudo sed -i 's/bind-address = 10.0.0.100/bind-address = 0.0.0.0/' \
    /etc/mysql/mariadb.conf.d/50-server.cnf
sudo systemctl restart mariadb
```

---

### Error: web1/web2 no podían conectar a MariaDB tras añadir reglas iptables

**Síntoma:** La web daba 504 Gateway Time-out. PHP se quedaba colgado al intentar conectar a la BD. `nc -zv 10.0.0.100 3306` no respondía desde web1.

**Causa:** Al añadir las reglas iptables para el acceso exclusivo desde el Windows Pro, se añadió una regla `DROP` para el puerto 3306 que bloqueaba también a web1 y web2, ya que la regla de ACCEPT para `192.168.50.20` se insertó después del DROP.

**Solución:** Insertar la regla de ACCEPT para la red interna de Vagrant en primera posición, antes del DROP:
```bash
sudo iptables -I INPUT 1 -p tcp --dport 3306 -s 10.0.0.0/24 -j ACCEPT
sudo netfilter-persistent save
```

---

### Error: Ruta por defecto incorrecta en el PC anfitrión bloqueaba el tráfico

**Síntoma:** El curl a `192.168.10.1` fallaba aunque la ruta hacia `172.16.0.0/24` estaba configurada correctamente.

**Causa:** Al añadir la interfaz host-only `192.168.50.x` a VirtualBox para la red Windows, se creó automáticamente un adaptador virtual en el PC anfitrión con una ruta por defecto incorrecta (`0.0.0.0 → 172.16.2.1`) que interfería con el tráfico.

**Solución:** Eliminar la ruta falsa desde PowerShell como administrador:
```powershell
route delete 0.0.0.0 mask 0.0.0.0 172.16.2.1
```

---

### Error: Windows Pro y Windows Server no se comunicaban

**Síntoma:** El ping entre `192.168.50.20` y `192.168.50.10` fallaba aunque ambos estaban en la misma red interna `windows` de VirtualBox.

**Causa:** El Windows Server tenía el firewall activado para todos los perfiles (Domain, Public, Private), bloqueando el tráfico ICMP y TCP entrante desde el Windows Pro.

**Solución:** Desactivar el firewall en el Windows Server (para entorno de laboratorio):
```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

---

## 13.2 Aplicación web PHP

### Error: `Call to undefined function mb_strlen()`

**Síntoma:** La página `mensajes.php` daba un Fatal Error al cargar.

**Causa:** La extensión `mbstring` de PHP no estaba instalada en los servidores web. Las funciones `mb_strlen()` y `mb_substr()` no estaban disponibles.

**Solución:** Sustituir las funciones multibyte por las estándar:
```bash
sed -i 's/mb_strlen/strlen/g; s/mb_substr/substr/g' /var/www/rulethegame/mensajes/mensajes.php
```

---

### Error: `Incorrect datetime value: '2026'` al contratar sesión

**Síntoma:** Al enviar el formulario de contratación aparecía "Error al contratar la sesión".

**Causa:** La cadena de tipo `bind_param` era `"iiiss"` cuando debería ser `"iisis"`. El campo `duracion` es entero (`i`), no string (`s`), por lo que MariaDB recibía el valor de la fecha malformado.

**Solución:**
```php
// Incorrecto
$ins->bind_param("iiiss", $_SESSION['usuario_id'], $id, $fecha_dt, $dur, $notas);

// Correcto
$ins->bind_param("iisis", $_SESSION['usuario_id'], $id, $fecha_dt, $dur, $notas);
```

---

### Error: El botón "Marcar como completada" no hacía nada

**Síntoma:** Al pulsar el botón en `sesiones_coach.php` no ocurría nada.

**Causa:** El botón tenía `onclick="return confirm('...')"`. El navegador mostraba el cuadro de confirmación, pero como el formulario hacía POST a una URL relativa en un entorno HTTPS, el confirm bloqueaba el envío.

**Solución:** Eliminar el `confirm()` del botón:
```php
// Antes
<button type="submit" class="btn_completar" onclick="return confirm('...')">

// Después
<button type="submit" class="btn_completar">
```

---

### Error: Los coaches duplicados al insertar nuevos

**Síntoma:** Al insertar coaches con `INSERT INTO coaches`, aparecían duplicados en la lista porque ya existían registros anteriores con los mismos nombres.

**Causa:** Los coaches originales (Nicolás Vera y Samuel Reyes) ya estaban en la BD desde una inserción anterior. Al insertar nuevos registros con los mismos nombres, se duplicaron.

**Solución:** Borrar los registros duplicados por ID y reorganizar los IDs:
```sql
DELETE FROM coaches WHERE id IN (2, 3);
SET FOREIGN_KEY_CHECKS = 0;
SET @i = 0;
UPDATE coaches SET id = (@i := @i + 1) ORDER BY id;
ALTER TABLE coaches AUTO_INCREMENT = 8;
SET FOREIGN_KEY_CHECKS = 1;
```

---

### Error: El balanceador usaba Apache en la documentación pero Nginx en producción

**Síntoma:** La documentación describía módulos de Apache (`mod_proxy`, `mod_proxy_balancer`) pero el balanceador real usaba Nginx.

**Causa:** El Vagrantfile original configuraba Apache como balanceador, pero durante el desarrollo se migró a Nginx por su mayor simplicidad de configuración para SSL.

**Solución:** Actualizar toda la documentación para reflejar Nginx como balanceador real.

---

### Error: `command not found` al ejecutar `a2enmod` en el balanceador

**Síntoma:** Al intentar habilitar módulos de Apache en el balanceador, el comando `a2enmod` no existía.

**Causa:** El balanceador tenía Nginx instalado, no Apache. Los comandos de Apache no estaban disponibles.

**Solución:** Usar los comandos de Nginx (`nginx -t`, `systemctl reload nginx`) en lugar de los de Apache.

---

### Error: La mensajería no mostraba conversaciones en la bandeja

**Síntoma:** La página `mensajes.php` mostraba "No tienes conversaciones" aunque existían mensajes en la BD.

**Causa:** La query original usaba `CASE WHEN` dentro de un subquery correlacionado que no funcionaba correctamente en la versión de MariaDB instalada.

**Solución:** Reescribir la query de forma más simple, obteniendo primero los IDs de los interlocutores y luego consultando cada conversación por separado en un bucle PHP.

---

### Error: `mysql: command not found` en WEB1

**Síntoma:** Al intentar ejecutar comandos MySQL desde WEB1 para depurar, el cliente MySQL no estaba instalado.

**Causa:** Los servidores web solo tienen Apache y PHP instalados, no el cliente MySQL.

**Solución:** Ejecutar los comandos SQL desde la VM `db`, que sí tiene MariaDB instalado, o instalar el cliente:
```bash
sudo apt install default-mysql-client -y
```

---

## 13.3 Exposición a Internet

### Error: Cloudflare Tunnel no conecta desde la red del instituto

**Síntoma:** El túnel intentaba conectar pero daba timeout.

**Causa:** La red del instituto bloquea el tráfico UDP (necesario para el protocolo QUIC de Cloudflare) y el puerto TCP 7844.

**Solución:** Usar el iPhone como hotspot para saltarse las restricciones de red, y forzar HTTP2 en el tunnel:
```powershell
.\cloudflared-windows-amd64.exe tunnel --protocol http2 --url http://127.0.0.1:80
```

---

### Error: 502 Bad Gateway en la URL de Cloudflare

**Síntoma:** La URL pública de Cloudflare daba error 502.

**Causa:** Al usar `localhost` en la URL del túnel, Windows resolvía a IPv6 (`::1`) pero VirtualBox solo escuchaba en IPv4.

**Solución:** Usar `127.0.0.1` en lugar de `localhost`:
```powershell
# Incorrecto
.\cloudflared-windows-amd64.exe tunnel --url http://localhost:80

# Correcto
.\cloudflared-windows-amd64.exe tunnel --url http://127.0.0.1:80
```

---

[← Exposición a Internet](12-exposicion_internet.md) | [→ Red Windows y MariaDB](14-red-windows-mariadb.md) | [↑ Volver al índice](../README.md)
