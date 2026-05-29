# Sistema de notificaciones — `notificaciones/`

El sistema de notificaciones avisa al usuario cuando tiene una sesión que empieza en menos de 20 minutos. Consta de tres archivos PHP (endpoints) y un archivo JavaScript que hace polling periódico.

---

## Arquitectura general

```
Cada 30 segundos (JS polling)
    ↓
1. check_sesiones.php  → Detecta sesiones próximas y las inserta en tabla notificaciones
    ↓
2. get_notificaciones.php → Devuelve las notificaciones no leídas del usuario en JSON
    ↓
3. notificaciones.js   → Actualiza el badge y el dropdown en la navbar
    ↓
[Usuario hace clic en una notificación]
    ↓
4. marcar_leida.php    → Marca la notificación como leída en BD
```

---

## `check_sesiones.php` — Detectar sesiones próximas

```php
header('Content-Type: application/json');

if (!isset($_SESSION['usuario_id'])) {
    echo json_encode(['error' => 'no_auth']); exit;
}

$usuario_id = $_SESSION['usuario_id'];
$rol        = $_SESSION['usuario_rol'] ?? 'cliente';

// Busca sesiones pendientes que empiecen entre 5 y 20 minutos desde ahora
// DATE_ADD(NOW(), INTERVAL 5 MINUTE) = ahora + 5 minutos
// DATE_ADD(NOW(), INTERVAL 20 MINUTE) = ahora + 20 minutos
// BETWEEN: la fecha de la sesión debe estar en ese rango
$sql = "
    SELECT s.id, s.coach_id, s.cliente_id,
           uc.nombre  AS nombre_coach,   uc.apellido  AS apellido_coach,
           ucl.nombre AS nombre_cliente, ucl.apellido AS apellido_cliente
    FROM sesiones s
    JOIN usuarios uc  ON s.coach_id   = uc.id   -- Datos del coach
    JOIN usuarios ucl ON s.cliente_id = ucl.id  -- Datos del cliente
    WHERE s.estado = 'pendiente'
      AND s.fecha BETWEEN DATE_ADD(NOW(), INTERVAL 5 MINUTE)
                      AND DATE_ADD(NOW(), INTERVAL 20 MINUTE)
      AND (s.cliente_id = ? OR s.coach_id = ?) -- Afecta al usuario actual (sea cliente o coach)
";

while ($sesion = $result->fetch_assoc()) {
    // Construye el mensaje según el rol del usuario
    if ($rol === 'coach' && $sesion['coach_id'] == $usuario_id) {
        $contraparte = $sesion['nombre_cliente'] . ' ' . $sesion['apellido_cliente'];
        $mensaje = "Tienes una sesión con {$contraparte} en menos de 20 minutos.";
    } elseif ($sesion['cliente_id'] == $usuario_id) {
        $contraparte = $sesion['nombre_coach'] . ' ' . $sesion['apellido_coach'];
        $mensaje = "Tienes una sesión con {$contraparte} en menos de 20 minutos.";
    } else {
        continue; // Si no corresponde al usuario, se salta
    }

    // INSERT IGNORE: si ya existe una notificación para este usuario+sesión, no la duplica
    // La tabla notificaciones tiene un índice UNIQUE en (usuario_id, sesion_id)
    $ins = $conn->prepare(
        "INSERT IGNORE INTO notificaciones (usuario_id, sesion_id, mensaje) VALUES (?, ?, ?)"
    );
    $ins->bind_param("iis", $usuario_id, $sesion['id'], $mensaje);
    $ins->execute();
}
```

---

## `get_notificaciones.php` — Obtener notificaciones no leídas

```php
header('Content-Type: application/json');

if (!isset($_SESSION['usuario_id'])) {
    echo json_encode([]); exit; // Sin sesión, devuelve array vacío
}

// Devuelve solo las notificaciones NO leídas del usuario, ordenadas por fecha
$stmt = $conn->prepare("
    SELECT id, mensaje, created_at
    FROM notificaciones
    WHERE usuario_id = ? AND leida = 0
    ORDER BY created_at DESC
");
$stmt->bind_param("i", $usuario_id);
$stmt->execute();
$notifs = $stmt->get_result()->fetch_all(MYSQLI_ASSOC);

echo json_encode($notifs); // Devuelve array JSON: [{id, mensaje, created_at}, ...]
```

---

## `marcar_leida.php` — Marcar notificación como leída

```php
header('Content-Type: application/json');

// Lee el cuerpo de la petición POST en formato JSON
// file_get_contents('php://input') lee el raw body de la petición
$data = json_decode(file_get_contents('php://input'), true);
$id   = intval($data['id'] ?? 0);

if ($id > 0) {
    // Actualiza solo si: el id coincide Y pertenece al usuario actual (seguridad)
    $stmt = $conn->prepare(
        "UPDATE notificaciones SET leida = 1 WHERE id = ? AND usuario_id = ?"
    );
    $stmt->bind_param("ii", $id, $usuario_id);
    $stmt->execute();
}

echo json_encode(['ok' => true]);
```

---

## `notificaciones.js` — Lógica del cliente

```javascript
// Abre/cierra el dropdown al hacer clic en la campana
function toggleNotifDropdown() {
    document.getElementById('notifDropdown').classList.toggle('hidden');
}

// Cierra el dropdown al hacer clic fuera de él
document.addEventListener('click', function(e) {
    var wrapper = document.getElementById('notifWrapper');
    if (wrapper && !wrapper.contains(e.target)) {
        document.getElementById('notifDropdown').classList.add('hidden');
    }
});

function checkNotificaciones() {
    // Primero ejecuta check_sesiones para crear notificaciones pendientes
    fetch('/notificaciones/check_sesiones.php')
        .then(function() { getNotificaciones(); }) // Luego obtiene las notificaciones
        .catch(function() { getNotificaciones(); }); // También si falla check
}

function getNotificaciones() {
    fetch('/notificaciones/get_notificaciones.php')
        .then(function(r) { return r.json(); })
        .then(function(data) {
            var badge = document.getElementById('notifBadge');
            var list  = document.getElementById('notifList');

            if (!data.length) {
                badge.classList.add('hidden'); // Oculta el badge si no hay notificaciones
                list.innerHTML = '<li class="notif-empty">Sin notificaciones</li>';
                return;
            }

            badge.textContent = data.length; // Muestra el número de no leídas
            badge.classList.remove('hidden');
            list.innerHTML = '';

            data.forEach(function(n) {
                var li = document.createElement('li');
                li.textContent = n.mensaje;
                // Al hacer clic, la marca como leída y la elimina del DOM
                li.onclick = function() { marcarLeida(n.id, li); };
                list.appendChild(li);
            });
        });
}

function marcarLeida(id, elemento) {
    fetch('/notificaciones/marcar_leida.php', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ id: id }) // Envía el id en formato JSON
    }).then(function() {
        elemento.remove(); // Elimina el elemento del DOM

        // Cuenta las notificaciones restantes en el dropdown
        var restantes = document.querySelectorAll('#notifList li:not(.notif-empty)').length;
        var badge = document.getElementById('notifBadge');

        if (restantes === 0) {
            badge.classList.add('hidden');
            document.getElementById('notifList').innerHTML = '<li class="notif-empty">Sin notificaciones</li>';
        } else {
            badge.textContent = restantes; // Actualiza el contador
        }
    });
}

// Ejecuta al cargar la página
checkNotificaciones();

// Repite cada 30 segundos
setInterval(checkNotificaciones, 30000);
```
