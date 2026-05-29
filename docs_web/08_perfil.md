# `perfil/perfil.php` — Perfil del cliente

Página privada del usuario cliente. Muestra sus sesiones contratadas, permite editarlas y cancelarlas, y permite editar sus datos personales.

---

## Protección de acceso

```php
// Si no hay sesión activa, redirige al login
// Esto protege la página de accesos no autorizados
if (!isset($_SESSION['usuario_id'])) {
    header('Location: /inicio_sesion/inicio_sesion.php'); exit;
}

$id = $_SESSION['usuario_id']; // ID del usuario logueado
```

---

## Contador de mensajes no leídos

```php
// Cuenta los mensajes recibidos que aún no han sido leídos
// Se usa para mostrar el badge rojo en el botón de mensajes de la navbar
$no_leidos = 0;
$stmt_nl = $conn->prepare("SELECT COUNT(*) FROM mensajes WHERE receptor_id=? AND leido=0");
$stmt_nl->bind_param("i", $id);
$stmt_nl->execute();
$stmt_nl->bind_result($no_leidos); // bind_result para consultas con COUNT(*)
$stmt_nl->fetch();
$stmt_nl->close();
```

---

## Cancelar sesión

```php
// Se procesa antes que el formulario de edición de perfil
// isset($_POST['cancelar_sesion_id']) distingue qué formulario se envió
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['cancelar_sesion_id'])) {
    $csid = intval($_POST['cancelar_sesion_id']); // intval previene inyección SQL

    // Solo cancela si: la sesión es del usuario actual Y está en estado 'pendiente'
    // Esto impide que un usuario cancele sesiones de otro usuario
    $upd_c = $conn->prepare(
        "UPDATE sesiones SET estado='cancelada' WHERE id=? AND cliente_id=? AND estado='pendiente'"
    );
    $upd_c->bind_param("ii", $csid, $id);
    $upd_c->execute();
    $upd_c->close();
}
```

---

## Editar perfil

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && !isset($_POST['cancelar_sesion_id'])) {

    // ... validaciones de campos vacíos, email válido, contraseñas coinciden ...

    // Comprueba que el nuevo email/usuario no esté en uso por OTRA cuenta
    $check = $conn->prepare(
        "SELECT id FROM usuarios WHERE (email=? OR nom_usu=?) AND id!=?"
    );
    // id!=? excluye al propio usuario de la comprobación
    $check->bind_param("ssi", $email, $nom_usu, $id);

    if ($check->num_rows > 0) {
        $error = 'Ese email o usuario ya lo usa otra cuenta.';
    } else {
        if ($password) {
            // Si se introduce contraseña nueva, la hashea y actualiza también
            $hash = password_hash($password, PASSWORD_DEFAULT);
            $upd = $conn->prepare(
                "UPDATE usuarios SET nombre=?,apellido=?,nom_usu=?,fecha_nac=?,localidad=?,email=?,password=? WHERE id=?"
            );
            $upd->bind_param("sssssssi", $nombre,$apellido,$nom_usu,$fecha_nac,$localidad,$email,$hash,$id);
        } else {
            // Sin nueva contraseña, actualiza solo los datos personales
            $upd = $conn->prepare(
                "UPDATE usuarios SET nombre=?,apellido=?,nom_usu=?,fecha_nac=?,localidad=?,email=? WHERE id=?"
            );
            $upd->bind_param("ssssssi", $nombre,$apellido,$nom_usu,$fecha_nac,$localidad,$email,$id);
        }

        if ($upd->execute()) {
            // Actualiza también los datos de sesión para que la navbar refleje el cambio
            $_SESSION['usuario_nom'] = $nombre;
            $_SESSION['usuario_usr'] = $nom_usu;
            $success = '¡Datos actualizados correctamente!';
        }
    }
}
```

---

## Consulta de sesiones

```php
// JOIN con la tabla coaches para obtener el nombre del coach y el juego
$stmt_ses = $conn->prepare("
    SELECT s.id, s.fecha, s.duracion, s.estado, s.notas,
           c.nombre AS coach_nombre, c.apellido AS coach_apellido,
           c.juego, c.id AS coach_id
    FROM sesiones s
    JOIN coaches c ON s.coach_id = c.id
    WHERE s.cliente_id = ?    -- Solo las sesiones de este usuario
    ORDER BY s.fecha DESC     -- Las más recientes primero
");
```

---

## Tarjetas de sesión en HTML

```php
<?php foreach ($sesiones as $s):
    $estado_class = 'estado_' . $s['estado']; // Clase CSS: estado_pendiente, estado_completada...

    // Array que mapea estado BD → etiqueta visual
    $estado_label = [
        'pendiente'  => '⏳ Pendiente',
        'completada' => '✅ Completada',
        'cancelada'  => '❌ Cancelada'
    ][$s['estado']];

    $fecha_fmt = date('d/m/Y', strtotime($s['fecha'])); // Formatea fecha: 2026-05-20 → 20/05/2026
    $hora_fmt  = date('H:i',   strtotime($s['fecha'])); // Formatea hora: 10:00:00 → 10:00
?>
    <!-- Si la sesión está pendiente, muestra botón para cancelar -->
    <?php if ($s['estado'] === 'pendiente'): ?>
        <form method="POST" action="perfil.php">
            <input type="hidden" name="cancelar_sesion_id" value="<?= $s['id'] ?>">
            <button type="submit" onclick="return confirm('¿Seguro?')">Cancelar sesión</button>
        </form>
    <?php endif; ?>
<?php endforeach; ?>
```

---

## Sistema de notificaciones en navbar

```html
<!-- Campana con dropdown de notificaciones -->
<div class="notif-wrapper" id="notifWrapper">
    <button onclick="toggleNotifDropdown()">
        🔔
        <!-- Badge rojo que muestra el número de notificaciones no leídas -->
        <span class="notif-badge hidden" id="notifBadge">0</span>
    </button>
    <div class="notif-dropdown hidden" id="notifDropdown">
        <ul id="notifList">
            <li class="notif-empty">Sin notificaciones</li>
        </ul>
    </div>
</div>

<!-- Carga el JS que hace polling cada 30 segundos para obtener notificaciones -->
<script src="/notificaciones/notificaciones.js"></script>
```
