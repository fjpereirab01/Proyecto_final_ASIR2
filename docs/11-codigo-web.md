# 11. Código de la aplicación web

[← Volver al índice](../README.md)

---

## 11.1 Tecnologías utilizadas

| Tecnología | Uso |
|---|---|
| PHP 7.4+ | Backend y lógica de negocio |
| MariaDB 10.5+ | Base de datos relacional |
| HTML5 / CSS3 | Estructura y estilos |
| JavaScript (ES6) | Carrusel, polling de mensajes, polling de notificaciones, toggle contraseña |
| Bebas Neue + Nunito | Tipografías (Google Fonts) |

---

## 11.2 Estructura de ficheros

```
/var/www/rulethegame/
├── main.php                        # Página principal (landing)
├── db.php                          # Conexión a MariaDB
├── logout.php                      # Cierre de sesión
├── favicon.svg                     # Favicon RTG
├── rulethegame.css                 # Estilos globales
├── rulethegame.js                  # Scripts globales (carrusel)
├── sesiones_extra.css              # Estilos extra de sesiones
├── inicio_sesion/
│   ├── inicio_sesion.php           # Login de usuarios
│   └── inicio_sesion.css
├── pag_registro/
│   ├── registro.php                # Registro de nuevos usuarios
│   ├── registro.css
│   └── registro.js
├── perfil/
│   ├── perfil.php                  # Perfil del cliente (edición + sesiones + campana notificaciones)
│   └── perfil.css
├── perfil_coach/
│   ├── perfil_coach.php            # Perfil del coach (sesiones + comentarios + contratación + campana notificaciones)
│   ├── editar_coach.php            # Edición del perfil del coach
│   ├── sesiones_coach.php          # Todas las sesiones del coach por estado
│   └── perfil_coach.css
├── coaches/
│   ├── coaches.php                 # Listado y búsqueda de coaches
│   └── coaches.css
├── mensajes/
│   ├── mensajes.php                # Bandeja de mensajes
│   ├── conversacion.php            # Chat individual con polling
│   └── mensajes.css
├── notificaciones/
│   ├── check_sesiones.php          # Genera notificaciones para sesiones próximas
│   ├── get_notificaciones.php      # Devuelve notificaciones no leídas en JSON
│   ├── marcar_leida.php            # Marca una notificación como leída
│   └── notificaciones.js           # Lógica de campana, polling y dropdown
├── admin/
│   ├── admin.php                   # Panel de administración
│   └── admin.css
├── contacto/
│   ├── contacto.php
│   └── contacto.css
├── sessions/                       # Ficheros de sesión PHP (chmod 1777)
└── imagenes/
    ├── equipo.png
    ├── coaching_directo.png
    └── persona_feliz.png
```

---

## 11.3 Sistema de roles

La aplicación distingue tres roles almacenados en el campo `rol` de la tabla `usuarios`:

| Rol | Acceso |
|---|---|
| `cliente` | Buscar coaches, contratar sesiones, valorar coaches, mensajería |
| `coach` | Ver y gestionar sus sesiones, editar su perfil, mensajería |
| `admin` | Acceso completo al panel de administración |

Al iniciar sesión, el rol se guarda en la sesión PHP:

```php
$_SESSION['usuario_id']  = $row['id'];
$_SESSION['usuario_nom'] = $row['nombre'];
$_SESSION['usuario_usr'] = $row['nom_usu'];
$_SESSION['usuario_rol'] = $row['rol'];

// Si es coach, se guarda también su coach_id
if ($row['rol'] === 'coach') {
    $sc = $conn->prepare("SELECT id FROM coaches WHERE usuario_id = ?");
    $sc->bind_param("i", $row['id']);
    $sc->execute();
    $rc = $sc->get_result()->fetch_assoc();
    if ($rc) $_SESSION['coach_id'] = $rc['id'];
}
```

El botón "Mi perfil" del navbar redirige según el rol:

```php
$url = isset($_SESSION['usuario_rol']) && $_SESSION['usuario_rol'] === 'coach'
    ? 'https://192.168.10.1/perfil_coach/perfil_coach.php?id=' . $_SESSION['coach_id']
    : 'https://192.168.10.1/perfil/perfil.php';
```

---

## 11.4 Conexión a la base de datos

Fichero `db.php`, compartido por todos los scripts PHP:

```php
<?php
$host = '10.0.0.100';
$db   = 'proyecto_asir';
$user = 'rtg_php';
$pass = 'CambiarPasswordPHP!';

$conn = new mysqli($host, $user, $pass, $db);
if ($conn->connect_error) {
    die('Error de conexión: ' . $conn->connect_error);
}
$conn->set_charset('utf8mb4');
```

La conexión siempre usa el usuario `rtg_php` con permisos mínimos, nunca `root`.

---

## 11.5 Sistema de mensajería

La mensajería interna usa **polling** en lugar de WebSockets para simplificar la implementación en el entorno Vagrant.

**Tabla `mensajes`:** `emisor_id`, `receptor_id`, `contenido`, `leido`, `created_at`.

**Flujo:**
1. El cliente envía un mensaje desde `perfil_coach.php` pulsando "Enviar mensaje".
2. El mensaje se inserta en la tabla `mensajes` vía POST.
3. `conversacion.php` muestra el historial de mensajes y ejecuta polling cada 3 segundos.
4. El polling hace una petición AJAX con `?poll=1&desde=ULTIMO_ID` y recibe los mensajes nuevos en JSON.
5. Los nuevos mensajes se añaden al DOM sin recargar la página.

```javascript
function cargarNuevos() {
    fetch('conversacion.php?con=' + CON_ID + '&poll=1&desde=' + ultimoId)
        .then(r => r.json())
        .then(msgs => {
            msgs.forEach(m => {
                if (!document.querySelector('[data-id="' + m.id + '"]')) {
                    agregarBurbuja(m);
                }
            });
        });
}
setInterval(cargarNuevos, 3000);
```

---

## 11.6 Sistema de notificaciones

La plataforma incluye un sistema de notificaciones automáticas que alerta a clientes y coaches cuando tienen una sesión próxima a comenzar.

### Funcionamiento

- Cada 30 segundos, el frontend llama a `check_sesiones.php`, que busca sesiones en estado `pendiente` cuya fecha esté entre 5 y 20 minutos en el futuro.
- Si encuentra alguna, inserta una notificación en la tabla `notificaciones` para el usuario correspondiente. La clave única `uq_notif (usuario_id, sesion_id)` evita duplicados.
- A continuación, `get_notificaciones.php` devuelve en JSON las notificaciones no leídas del usuario.
- La campana 🔔 del navbar muestra un badge rojo con el número de notificaciones pendientes.
- Al hacer clic en una notificación, se llama a `marcar_leida.php` vía POST y desaparece del dropdown.

### Diagrama de flujo

```
Frontend (cada 30s)
    │
    ├─► check_sesiones.php
    │       └─ Busca sesiones entre NOW()+5min y NOW()+20min
    │       └─ INSERT IGNORE INTO notificaciones
    │
    └─► get_notificaciones.php
            └─ SELECT notificaciones WHERE leida=0
            └─ Devuelve JSON → actualiza campana y dropdown
```

### Endpoints

| Fichero | Método | Descripción |
|---|---|---|
| `check_sesiones.php` | GET | Genera notificaciones para sesiones próximas |
| `get_notificaciones.php` | GET | Devuelve notificaciones no leídas en JSON |
| `marcar_leida.php` | POST | Marca una notificación como leída |

### Código principal — check_sesiones.php

```php
$sql = "
    SELECT s.id, s.coach_id, s.cliente_id,
           uc.nombre AS nombre_coach, uc.apellido AS apellido_coach,
           ucl.nombre AS nombre_cliente, ucl.apellido AS apellido_cliente
    FROM sesiones s
    JOIN usuarios uc  ON s.coach_id   = uc.id
    JOIN usuarios ucl ON s.cliente_id = ucl.id
    WHERE s.estado = 'pendiente'
      AND s.fecha BETWEEN DATE_ADD(NOW(), INTERVAL 5 MINUTE)
                      AND DATE_ADD(NOW(), INTERVAL 20 MINUTE)
      AND (s.cliente_id = ? OR s.coach_id = ?)
";
```

### Código principal — notificaciones.js

```javascript
function checkNotificaciones() {
    fetch('https://192.168.10.1/notificaciones/check_sesiones.php')
        .then(function() { getNotificaciones(); });
}

function getNotificaciones() {
    fetch('https://192.168.10.1/notificaciones/get_notificaciones.php')
        .then(function(r) { return r.json(); })
        .then(function(data) {
            var badge = document.getElementById('notifBadge');
            if (!data.length) {
                badge.classList.add('hidden');
                return;
            }
            badge.textContent = data.length;
            badge.classList.remove('hidden');
            // Renderiza el dropdown con las notificaciones
        });
}

// Polling cada 30 segundos
checkNotificaciones();
setInterval(checkNotificaciones, 30000);
```

---

## 11.7 Gestión de sesiones en entorno multi-servidor

Con dos servidores web detrás del balanceador, las sesiones PHP pueden perderse si una petición va a un servidor diferente al que creó la sesión.

**Solución aplicada:** directiva `ip_hash` en Nginx para que cada IP siempre vaya al mismo servidor web.

**Configuración de la carpeta de sesiones:**
```bash
# La carpeta sessions está en el NFS compartido
sudo chmod 1777 /var/www/rulethegame/sessions
sudo chmod 1777 /var/www/html/sessions
```

El `php.ini` de cada servidor web apunta las sesiones a esta carpeta compartida:
```
session.save_path = /var/www/html/sessions
```

---

## 11.8 Contratación de sesiones

Flujo completo de contratación:

1. El cliente visita el perfil de un coach y rellena el formulario (fecha, hora, duración, notas).
2. Se inserta un registro en `sesiones` con estado `pendiente`.
3. La sesión aparece en el perfil del cliente y en el panel del coach.
4. El cliente puede cancelarla si está en estado `pendiente`.
5. El coach puede marcarla como `completada` desde su perfil o desde `sesiones_coach.php`.
6. El admin puede cambiar cualquier estado desde el panel de administración.
7. Cuando la sesión está entre 5 y 20 minutos antes de comenzar, el sistema genera automáticamente una notificación para el cliente y el coach.

```php
$ins = $conn->prepare(
    "INSERT INTO sesiones (cliente_id, coach_id, fecha, duracion, estado, notas)
     VALUES (?, ?, ?, ?, 'pendiente', ?)"
);
$ins->bind_param("iisis", $_SESSION['usuario_id'], $id, $fecha_dt, $dur, $notas);
$ins->execute();
```

---

[← Herramienta de gestión](07-herramienta-gestion.md) | [Siguiente → Exposición a Internet](12-exposicion_internet.md)
