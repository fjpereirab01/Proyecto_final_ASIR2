# `admin/admin.php` — Panel de administración

Página exclusiva del administrador (usuario con `id=1`). Permite gestionar coaches, usuarios y sesiones con operaciones CRUD completas.

---

## Protección de acceso

```php
// Solo accesible si hay sesión activa
if (!isset($_SESSION['usuario_id'])) {
    header('Location: /inicio_sesion/inicio_sesion.php'); exit;
}

// El admin es el usuario con id=1 (hardcodeado)
$ADMIN_ID = 1;
if ($_SESSION['usuario_id'] !== $ADMIN_ID) {
    header('Location: /main.php'); exit; // Cualquier otro usuario es redirigido
}
```

---

## Borrar usuario

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['borrar_id'])) {
    $bid = intval($_POST['borrar_id']);

    // Impide que el admin se borre a sí mismo
    if ($bid !== $_SESSION['usuario_id']) {
        $del = $conn->prepare("DELETE FROM usuarios WHERE id=?");
        $del->bind_param("i", $bid);
        $del->execute();
        $msg = "Usuario #$bid eliminado correctamente.";
    } else {
        $msg = "No puedes eliminar tu propia cuenta de administrador.";
    }
}
```

---

## Cambiar estado de sesión

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['cambiar_estado_id'])) {
    $sid    = intval($_POST['cambiar_estado_id']);
    $estado = $_POST['nuevo_estado'] ?? '';

    // Valida que el estado sea uno de los tres permitidos (whitelist)
    // Evita que se inserte un valor arbitrario en la BD
    if (in_array($estado, ['pendiente', 'completada', 'cancelada'])) {
        $upd = $conn->prepare("UPDATE sesiones SET estado=? WHERE id=?");
        $upd->bind_param("si", $estado, $sid);
        $upd->execute();
    }
}
```

---

## Crear coach

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['crear_coach'])) {
    $cn = trim($_POST['coach_nombre'] ?? '');
    $ca = trim($_POST['coach_apellido'] ?? '');
    $cj = trim($_POST['coach_juego'] ?? '');
    // ... más campos

    if ($cn && $ca && $cj) { // Mínimo: nombre, apellido y juego
        $ins = $conn->prepare(
            "INSERT INTO coaches (nombre, apellido, especialidad, juego, descripcion, contacto, precio)
             VALUES (?,?,?,?,?,?,?)"
        );
        // "ssssssd": 6 strings + 1 double (precio)
        $ins->bind_param("ssssssd", $cn, $ca, $ce, $cj, $cd, $cc, $cp);
        $ins->execute();
    }
}
```

---

## Editar coach

```php
// El formulario de edición y creación es el mismo HTML
// Se distinguen por el campo oculto: 'crear_coach' vs 'editar_coach'
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['editar_coach'])) {
    $eid = intval($_POST['coach_id']); // ID del coach a editar

    $upd = $conn->prepare(
        "UPDATE coaches SET nombre=?,apellido=?,especialidad=?,juego=?,
         descripcion=?,contacto=?,precio=? WHERE id=?"
    );
    // "ssssssdi": 6 strings + double + integer
    $upd->bind_param("ssssssdi", $cn, $ca, $ce, $cj, $cd, $cc, $cp, $eid);
}
```

---

## Formulario doble (crear/editar)

```php
// Si hay un coach_editar (viene de ?editar_coach=ID en la URL), precarga sus datos
$coach_editar = null;
if (isset($_GET['editar_coach'])) {
    $eid = intval($_GET['editar_coach']);
    foreach ($coaches as $c) {
        if ($c['id'] === $eid) { $coach_editar = $c; break; }
    }
}
```

```html
<form method="POST" class="admin_form">
    <!-- Campo oculto que indica si es crear o editar -->
    <?php if ($coach_editar): ?>
        <input type="hidden" name="editar_coach" value="1">
        <input type="hidden" name="coach_id" value="<?= $coach_editar['id'] ?>">
    <?php else: ?>
        <input type="hidden" name="crear_coach" value="1">
    <?php endif; ?>

    <!-- Los inputs muestran los valores actuales si es edición, o vacío si es creación -->
    <input type="text" name="coach_nombre"
           value="<?= htmlspecialchars($coach_editar['nombre'] ?? '') ?>" required>
    <!-- ... -->

    <button type="submit">
        <?= $coach_editar ? 'Guardar cambios' : 'Crear coach' ?>
    </button>
</form>
```

---

## Estadísticas del panel

```php
// Todas las consultas usan fetch_all para obtener arrays completos
$usuarios = $conn->query("SELECT ... FROM usuarios ORDER BY created_at DESC")->fetch_all(MYSQLI_ASSOC);
$coaches  = $conn->query("SELECT ... FROM coaches ORDER BY id ASC")->fetch_all(MYSQLI_ASSOC);
$sesiones = $conn->query("SELECT s.*, u.nombre, c.nombre ... JOIN ... JOIN ...")->fetch_all(MYSQLI_ASSOC);

// Calcula totales para las tarjetas de estadísticas
$total_usuarios       = count($usuarios);
$total_coaches        = count($coaches);
$sesiones_pendientes  = count(array_filter($sesiones, fn($s) => $s['estado'] === 'pendiente'));
$sesiones_completadas = count(array_filter($sesiones, fn($s) => $s['estado'] === 'completada'));
```

---

## Badges de rol en la tabla de usuarios

```html
<!-- Cada rol tiene una clase CSS diferente con color distinto -->
<span class="badge_rol badge_<?= $u['rol'] ?>">
    <?= $u['rol'] ?>
</span>

<!-- badge_coach  → azul -->
<!-- badge_cliente → verde -->
<!-- badge_admin  → rojo -->
```
