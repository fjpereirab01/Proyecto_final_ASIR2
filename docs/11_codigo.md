# 11. Código de la aplicación web

[← Volver al índice](../README.md)

---

## 11.1 Tecnologías utilizadas

| Tecnología | Uso |
|---|---|
| PHP 7.4+ | Backend y lógica de negocio |
| MariaDB 10.5+ | Base de datos relacional |
| HTML5 / CSS3 | Estructura y estilos |
| JavaScript (ES6) | Carrusel, polling de mensajes, toggle contraseña |
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
│   ├── perfil.php                  # Perfil del cliente (edición + sesiones)
│   └── perfil.css
├── perfil_coach/
│   ├── perfil_coach.php            # Perfil del coach (sesiones + comentarios + contratación)
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

## 11.6 Gestión de sesiones en entorno multi-servidor

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

## 11.7 Contratación de sesiones

Flujo completo de contratación:

1. El cliente visita el perfil de un coach y rellena el formulario (fecha, hora, duración, notas).
2. Se inserta un registro en `sesiones` con estado `pendiente`.
3. La sesión aparece en el perfil del cliente y en el panel del coach.
4. El cliente puede cancelarla si está en estado `pendiente`.
5. El coach puede marcarla como `completada` desde su perfil o desde `sesiones_coach.php`.
6. El admin puede cambiar cualquier estado desde el panel de administración.

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
