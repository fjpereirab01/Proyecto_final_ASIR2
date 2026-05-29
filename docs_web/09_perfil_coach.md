# `perfil_coach/perfil_coach.php` — Perfil del coach

Página del perfil público de un coach. Dependiendo de si el usuario logueado es el propio coach, muestra opciones diferentes. Es la página más compleja del proyecto.

---

## Detección del coach y permisos

```php
$id = intval($_GET['id'] ?? 0); // ID del coach en la URL (?id=3)

// Si no se pasa id o no existe ese coach, redirige al listado
if (!$id) { header('Location: /coaches/coaches.php'); exit; }

$stmt = $conn->prepare("SELECT * FROM coaches WHERE id = ?");
$stmt->execute(); // ...
$coach = $stmt->get_result()->fetch_assoc();

if (!$coach) { header('Location: /coaches/coaches.php'); exit; }

// Comprueba si el usuario logueado es el dueño de este perfil
// Compara el usuario_id del coach con el id de sesión
$es_mi_perfil = false;
if (isset($_SESSION['usuario_id'])) {
    $es_mi_perfil = (intval($coach['usuario_id']) === intval($_SESSION['usuario_id']));
}
```

---

## Contratar sesión (solo clientes)

```php
// Solo se procesa si: hay sesión activa Y no es el propio coach
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['contratar_sesion'])
    && isset($_SESSION['usuario_id']) && !$es_mi_perfil) {

    $fecha    = trim($_POST['fecha'] ?? '');
    $hora     = trim($_POST['hora'] ?? '');
    $dur      = intval($_POST['duracion'] ?? 60);
    $notas    = trim($_POST['notas'] ?? '');

    if (!$fecha || !$hora) {
        $ses_error = 'La fecha y la hora son obligatorias.';
    } else {
        // Combina fecha y hora en formato datetime de MySQL
        $fecha_dt = $fecha . ' ' . $hora . ':00'; // "2026-06-01 10:00:00"

        $ins = $conn->prepare(
            "INSERT INTO sesiones (cliente_id, coach_id, fecha, duracion, estado, notas)
             VALUES (?,?,?,?,'pendiente',?)"
        );
        $ins->bind_param("iisis", $_SESSION['usuario_id'], $id, $fecha_dt, $dur, $notas);
        $ins->execute();
        // ...
    }
}
```

---

## Guardar comentario

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['comentario'])
    && isset($_SESSION['usuario_id'])) {

    $texto     = trim($_POST['comentario'] ?? '');
    $valoracion = intval($_POST['valoracion'] ?? 0);

    // Validación: texto no vacío y valoración entre 1 y 5
    if (!$texto || $valoracion < 1 || $valoracion > 5) {
        $com_error = 'Escribe un comentario y selecciona una valoración.';
    } else {
        // Comprueba que el usuario no haya comentado ya este coach
        $check = $conn->prepare(
            "SELECT id FROM comentarios WHERE coach_id=? AND usuario_id=?"
        );
        // ...

        if ($check->num_rows > 0) {
            $com_error = 'Ya has dejado un comentario para este coach.';
        } else {
            // Inserta el comentario
            $ins = $conn->prepare(
                "INSERT INTO comentarios (coach_id, usuario_id, comentario, valoracion)
                 VALUES (?,?,?,?)"
            );
            $ins->execute();

            // Recalcula la valoración media del coach tras el nuevo comentario
            $conn->query(
                "UPDATE coaches SET valoracion_media =
                 (SELECT AVG(valoracion) FROM comentarios WHERE coach_id=$id)
                 WHERE id=$id"
            );
        }
    }
}
```

---

## Marcar sesión como completada (solo el coach)

```php
// Solo el propio coach puede marcar sus sesiones como completadas
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['completar_sesion_id']) && $es_mi_perfil) {
    $csid = intval($_POST['completar_sesion_id']);
    $upd = $conn->prepare(
        "UPDATE sesiones SET estado='completada' WHERE id=? AND coach_id=? AND estado='pendiente'"
    );
    $upd->bind_param("ii", $csid, $id);
    $upd->execute();
}
```

---

## Sección hero del coach

```html
<div id="coach_hero">
    <!-- Avatar con iniciales -->
    <div id="coach_hero_avatar">JD</div>

    <div id="coach_hero_info">
        <h1>Juan Doe</h1>
        <span class="coach_juego_badge">Valorant</span>

        <!-- Estrellas de valoración media -->
        <?php for ($i=1; $i<=5; $i++) echo $i <= $estrellas_media ? '★' : '☆'; ?>

        <!-- Solo visible para el propio coach -->
        <?php if ($es_mi_perfil): ?>
            <a href="/perfil_coach/editar_coach.php?id=<?= $id ?>">✏️ Editar perfil</a>
        <?php endif; ?>
    </div>

    <div id="coach_hero_precio">
        <span>25.00€</span><span>por hora</span>

        <?php if (isset($_SESSION['usuario_id']) && !$es_mi_perfil): ?>
            <!-- Un cliente puede contratar sesión o enviar mensaje -->
            <a href="#contratar_sesion">Contratar sesión</a>
            <a href="/mensajes/conversacion.php?con=<?= $coach['usuario_id'] ?>">Enviar mensaje</a>
        <?php else: ?>
            <!-- Sin sesión o es el propio coach: solo contacto por email -->
            <a href="mailto:...">Contactar</a>
        <?php endif; ?>
    </div>
</div>
```

---

## Selector de estrellas para comentarios

```html
<!-- CSS trick: inputs radio ocultos con labels de estrellas -->
<!-- El orden es de 5 a 1 para que funcione el selector inverso con CSS -->
<div id="estrellas_selector">
    <?php for ($i=5; $i>=1; $i--): ?>
        <input type="radio" id="star<?=$i?>" name="valoracion" value="<?=$i?>">
        <label for="star<?=$i?>">★</label>
    <?php endfor; ?>
</div>
```
El estilo visual de las estrellas se controla desde `perfil_coach.css` usando el selector CSS `:checked ~ label`.
