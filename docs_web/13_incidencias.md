# Sistema de incidencias — `incidencias/incidencias.php` y `incidencias/mis_incidencias.php`

Sistema que permite a los usuarios reportar problemas al equipo de RuleTheGame. Las incidencias se almacenan en la tabla `incidencias` con estados: `abierta`, `en_curso` y `resuelta`.

---

## `incidencias/incidencias.php` — Abrir incidencia

### Estructura de la tabla `incidencias`

| Campo | Tipo | Descripción |
|---|---|---|
| id | INT | Clave primaria autoincremental |
| usuario_id | INT | FK → usuarios.id |
| rol | VARCHAR | Rol del usuario en el momento del envío |
| titulo | VARCHAR(150) | Título corto de la incidencia |
| descripcion | TEXT | Descripción detallada |
| estado | ENUM | `abierta`, `en_curso`, `resuelta` |
| asignado_a | INT | FK → usuarios.id (admin que la gestiona) |
| created_at | TIMESTAMP | Fecha de creación automática |

### Lógica PHP

```php
$usuario_id = $_SESSION['usuario_id'];
$rol        = $_SESSION['usuario_rol'] ?? 'cliente'; // Rol en el momento del envío

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $titulo      = trim($_POST['titulo'] ?? '');
    $descripcion = trim($_POST['descripcion'] ?? '');

    // Ambos campos son obligatorios
    if ($titulo === '' || $descripcion === '') {
        $mensaje_err = 'Por favor, rellena todos los campos.';
    } else {
        // Inserta la incidencia con estado 'abierta' por defecto (valor por defecto en BD)
        $stmt = $conn->prepare(
            "INSERT INTO incidencias (usuario_id, rol, titulo, descripcion) VALUES (?, ?, ?, ?)"
        );
        $stmt->bind_param("isss", $usuario_id, $rol, $titulo, $descripcion);
        // "isss": integer, string, string, string
        if ($stmt->execute()) {
            $mensaje_ok = 'Incidencia enviada correctamente.';
        }
    }
}
```

### Badge de rol

```html
<!-- Muestra el rol del usuario que abre la incidencia -->
<!-- ucfirst convierte la primera letra a mayúscula: "cliente" → "Cliente" -->
<span class="rol_badge"><?= htmlspecialchars(ucfirst($rol)) ?></span>
```

---

## `incidencias/mis_incidencias.php` — Ver mis incidencias

### Consulta de incidencias del usuario

```php
// Solo devuelve las incidencias del usuario logueado, ordenadas por fecha descendente
$stmt = $conn->prepare(
    "SELECT id, titulo, descripcion, estado, created_at
     FROM incidencias
     WHERE usuario_id = ?
     ORDER BY created_at DESC"
);
$stmt->bind_param("i", $usuario_id);
$stmt->execute();
$incidencias = $stmt->get_result()->fetch_all(MYSQLI_ASSOC);
```

### Mapa de estilos por estado

```php
// Define el color y fondo para cada estado del badge
$estado_info = [
    'abierta'  => ['color' => '#f59e0b', 'bg' => '#2a1f00', 'label' => 'Abierta'],   // Naranja
    'en_curso' => ['color' => '#60a5fa', 'bg' => '#0f1f3a', 'label' => 'En curso'],  // Azul
    'resuelta' => ['color' => '#4ade80', 'bg' => '#0a2a1a', 'label' => 'Resuelta'],  // Verde
];
```

### Badge de estado dinámico

```php
<?php foreach ($incidencias as $inc):
    // Obtiene los estilos del estado actual, con fallback a 'abierta' si el estado no existe
    $ei = $estado_info[$inc['estado']] ?? $estado_info['abierta'];
?>
<div class="inc_card">
    <div class="inc_card_top">
        <p class="inc_titulo"><?= htmlspecialchars($inc['titulo']) ?></p>

        <!-- Badge con estilos inline dinámicos según el estado -->
        <span class="estado_badge" style="
            color: <?= $ei['color'] ?>;
            background: <?= $ei['bg'] ?>;
            border: 1px solid <?= $ei['color'] ?>">
            <?= $ei['label'] ?>
        </span>
    </div>

    <!-- Descripción truncada a 2 líneas con CSS (-webkit-line-clamp) -->
    <p class="inc_desc"><?= htmlspecialchars($inc['descripcion']) ?></p>

    <div class="inc_meta">
        <!-- ID de la incidencia para referencia -->
        <span class="inc_id">#<?= $inc['id'] ?></span>
        <?= date('d/m/Y H:i', strtotime($inc['created_at'])) ?>
    </div>
</div>
<?php endforeach; ?>
```

### Estado vacío

```html
<!-- Si el usuario no tiene incidencias, se muestra un estado vacío con CTA -->
<?php if (empty($incidencias)): ?>
    <div class="inc_vacio">
        <div class="icono">📭</div>
        <p>No tienes incidencias abiertas todavía.</p>
        <a href="/incidencias/incidencias.php" class="btn_nueva">Abrir primera incidencia</a>
    </div>
<?php endif; ?>
```
