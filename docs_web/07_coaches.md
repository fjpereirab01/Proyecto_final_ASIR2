# `coaches/coaches.php` — Listado de coaches

Página pública que muestra todos los coaches registrados, con filtros por juego y buscador por nombre o especialidad.

---

## Bloque PHP — Consultas dinámicas

```php
<?php
session_start();
require_once '../db.php';

// Lee los parámetros de la URL (?juego=Valorant&buscar=texto)
$juego  = $_GET['juego']  ?? '';
$buscar = trim($_GET['buscar'] ?? '');

// Construye la consulta SQL según los filtros activos
// Se usan 4 casos para evitar consultas innecesariamente complejas

if ($juego && $buscar) {
    // Filtro por juego Y búsqueda de texto simultáneos
    $like = "%$buscar%"; // % = cualquier texto antes/después
    $stmt = $conn->prepare(
        "SELECT ... FROM coaches WHERE juego = ?
         AND (nombre LIKE ? OR apellido LIKE ? OR especialidad LIKE ?)
         ORDER BY valoracion_media DESC"
    );
    $stmt->bind_param("ssss", $juego, $like, $like, $like);

} elseif ($juego) {
    // Solo filtro por juego
    $stmt = $conn->prepare("SELECT ... FROM coaches WHERE juego = ? ORDER BY valoracion_media DESC");
    $stmt->bind_param("s", $juego);

} elseif ($buscar) {
    // Solo búsqueda de texto (nombre, apellido o especialidad)
    $like = "%$buscar%";
    $stmt = $conn->prepare(
        "SELECT ... FROM coaches WHERE nombre LIKE ? OR apellido LIKE ? OR especialidad LIKE ?
         ORDER BY valoracion_media DESC"
    );
    $stmt->bind_param("sss", $like, $like, $like);

} else {
    // Sin filtros: devuelve todos los coaches ordenados por valoración
    $stmt = $conn->prepare("SELECT ... FROM coaches ORDER BY valoracion_media DESC");
}

$stmt->execute();
$coaches = $stmt->get_result()->fetch_all(MYSQLI_ASSOC); // Array con todos los resultados
$stmt->close();

// Consulta separada para obtener los juegos únicos para los botones de filtro
$juegos_stmt = $conn->query("SELECT DISTINCT juego FROM coaches ORDER BY juego");
$juegos = $juegos_stmt->fetch_all(MYSQLI_ASSOC);
?>
```

---

## Formulario de búsqueda

```html
<form method="GET" action="coaches.php" id="form_buscar">

    <!-- Si hay filtro de juego activo, se mantiene en el siguiente GET -->
    <?php if ($juego): ?>
        <input type="hidden" name="juego" value="<?= htmlspecialchars($juego) ?>">
    <?php endif; ?>

    <input type="text" name="buscar" value="<?= htmlspecialchars($buscar) ?>">
    <button type="submit">Buscar</button>

    <!-- Botón limpiar solo aparece si hay algún filtro activo -->
    <?php if ($buscar || $juego): ?>
        <a href="coaches.php">✕ Limpiar</a>
    <?php endif; ?>
</form>
```

---

## Botones de filtro por juego

```php
// Botón "Todos" — activo cuando no hay filtro de juego
<a href="coaches.php" class="filtro_btn <?= !$juego ? 'activo' : '' ?>">Todos</a>

// Un botón por cada juego único en BD
<?php foreach ($juegos as $j): ?>
    <a href="coaches.php?juego=<?= urlencode($j['juego']) ?>"
       class="filtro_btn <?= $juego === $j['juego'] ? 'activo' : '' ?>">
        <?= htmlspecialchars($j['juego']) ?>
    </a>
<?php endforeach; ?>
```
`urlencode()` convierte caracteres especiales de la URL (ej: espacios → `%20`).

---

## Grid de coaches

```php
<?php foreach ($coaches as $c):
    // Genera las iniciales del nombre para el avatar (ej: "John Doe" → "JD")
    $iniciales = strtoupper(substr($c['nombre'],0,1) . substr($c['apellido'],0,1));

    // Redondea la valoración para mostrar estrellas enteras
    $estrellas = round($c['valoracion_media']);
?>
<div class="coach_card_page">
    <!-- Avatar con iniciales -->
    <div class="coach_avatar_page"><?= htmlspecialchars($iniciales) ?></div>

    <div class="coach_info">
        <h3><?= htmlspecialchars($c['nombre'].' '.$c['apellido']) ?></h3>
        <span class="coach_juego_badge"><?= htmlspecialchars($c['juego']) ?></span>
        <p class="coach_especialidad">📌 <?= htmlspecialchars($c['especialidad']) ?></p>

        <!-- Genera estrellas: ★ para llenas, ☆ para vacías -->
        <div class="coach_stars">
            <?php for ($i=1; $i<=5; $i++) echo $i <= $estrellas ? '★' : '☆'; ?>
            <span>(<?= number_format($c['valoracion_media'], 1) ?>)</span>
        </div>

        <p class="coach_precio">💶 <?= number_format($c['precio'], 2) ?>€/hora</p>
    </div>

    <!-- Enlace al perfil completo del coach -->
    <a href="/perfil_coach/perfil_coach.php?id=<?= $c['id'] ?>" class="btn_ver_perfil">
        Ver perfil
    </a>
</div>
<?php endforeach; ?>
```
