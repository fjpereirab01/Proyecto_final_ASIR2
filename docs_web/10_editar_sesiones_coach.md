# `perfil_coach/editar_coach.php` — Editar perfil del coach

Página privada exclusiva del coach para editar su información profesional (juego, especialidad, precio, descripción, contacto).

---

## Protección de acceso

```php
// Requiere sesión activa Y que sea un coach (tiene coach_id en sesión)
if (!isset($_SESSION['usuario_id']) || !isset($_SESSION['coach_id'])) {
    header('Location: /inicio_sesion/inicio_sesion.php'); exit;
}

$coach_id = intval($_GET['id'] ?? 0);

// Verifica que el id de la URL coincide con el coach de la sesión
// Impide que un coach edite el perfil de otro
if (!$coach_id || $coach_id !== intval($_SESSION['coach_id'])) {
    header('Location: /main.php'); exit;
}
```

---

## Guardar cambios

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $especialidad = trim($_POST['especialidad'] ?? '');
    $juego        = trim($_POST['juego'] ?? '');
    $descripcion  = trim($_POST['descripcion'] ?? '');
    $contacto     = trim($_POST['contacto'] ?? '');
    $precio       = floatval($_POST['precio'] ?? 0); // floatval para precio decimal

    if (!$juego) {
        $error = 'El juego es obligatorio.';
    } else {
        $upd = $conn->prepare(
            "UPDATE coaches SET especialidad=?, juego=?, descripcion=?, contacto=?, precio=? WHERE id=?"
        );
        // "ssssdi": string, string, string, string, double (decimal), integer
        $upd->bind_param("ssssdi", $especialidad, $juego, $descripcion, $contacto, $precio, $coach_id);

        if ($upd->execute()) {
            $success = '¡Perfil actualizado correctamente!';

            // Recarga los datos del coach para que el formulario muestre los valores actualizados
            $stmt2 = $conn->prepare("SELECT * FROM coaches WHERE id=?");
            $stmt2->bind_param("i", $coach_id);
            $stmt2->execute();
            $coach = $stmt2->get_result()->fetch_assoc();
            $stmt2->close();
        }
    }
}
```

---

---

# `perfil_coach/sesiones_coach.php` — Gestión de sesiones del coach

Página privada del coach que muestra todas sus sesiones organizadas por estado: pendientes, completadas y canceladas.

---

## Carga de sesiones

```php
// Obtiene todas las sesiones del coach con datos del cliente
$stmt = $conn->prepare("
    SELECT s.id, s.fecha, s.duracion, s.estado, s.notas,
           u.nombre AS cliente_nombre, u.apellido AS cliente_apellido,
           u.nom_usu AS cliente_usu, u.id AS cliente_id
    FROM sesiones s
    JOIN usuarios u ON s.cliente_id = u.id   -- Datos del cliente que contrató
    WHERE s.coach_id = ?
    ORDER BY s.fecha DESC
");
$stmt->bind_param("i", $coach_id);
$stmt->execute();
$todas = $stmt->get_result()->fetch_all(MYSQLI_ASSOC);
$stmt->close();

// Separa las sesiones en tres arrays según su estado
// array_filter devuelve solo los elementos que cumplen la condición (fn = función flecha)
$pendientes  = array_filter($todas, fn($s) => $s['estado'] === 'pendiente');
$completadas = array_filter($todas, fn($s) => $s['estado'] === 'completada');
$canceladas  = array_filter($todas, fn($s) => $s['estado'] === 'cancelada');
```

---

## Marcar sesión como completada

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['completar_sesion_id'])) {
    $csid = intval($_POST['completar_sesion_id']);
    $upd = $conn->prepare(
        "UPDATE sesiones SET estado='completada' WHERE id=? AND coach_id=? AND estado='pendiente'"
    );
    $upd->bind_param("ii", $csid, $coach_id);
    $upd->execute();

    // Redirige para evitar que al recargar la página se reenvíe el formulario (patrón POST-Redirect-GET)
    header('Location: /perfil_coach/sesiones_coach.php'); exit;
}
```

---

## Badges de conteo

```html
<!-- Muestra cuántas sesiones hay en cada estado -->
<div class="sesiones_bloque_titulo">
    ⏳ Pendientes
    <span class="count_badge"><?= count($pendientes) ?></span>
</div>

<div class="sesiones_bloque_titulo">
    ✅ Completadas
    <!-- Badge verde para completadas -->
    <span class="count_badge verde"><?= count($completadas) ?></span>
</div>

<div class="sesiones_bloque_titulo">
    ❌ Canceladas
    <!-- Badge gris para canceladas -->
    <span class="count_badge gris"><?= count($canceladas) ?></span>
</div>
```

---

## Enlace a conversación con el cliente

```html
<!-- Desde una sesión pendiente, el coach puede ir directamente al chat con ese cliente -->
<a href="/mensajes/conversacion.php?con=<?= $s['cliente_id'] ?>" class="cliente_link">
    <?= htmlspecialchars($s['cliente_nombre'].' '.$s['cliente_apellido']) ?>
</a>
```
