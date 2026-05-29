# `main.php` — Página principal

Es la página de entrada de RuleTheGame. Contiene la navbar, el hero, el carrusel, las estadísticas, la sección de servicios, los coaches destacados y el footer.

---

## Bloque PHP inicial

```php
<?php session_start(); ?>
```
Inicia la sesión para poder leer `$_SESSION` en la navbar (saber si el usuario está logueado y cuál es su rol).

---

## Navbar

```php
<?php if (isset($_SESSION['usuario_id'])): ?>
    <!-- Si hay sesión activa, muestra botones de perfil y logout -->
    <button id="btn_perfil" onclick="...">Mi perfil</button>

    <?php if ($_SESSION['usuario_id'] === 1): ?>
        <!-- Solo el usuario con id=1 (admin) ve el botón de Admin -->
        <button onclick="...">Admin</button>
    <?php endif; ?>

    <button id="btn_logout">Cerrar sesión</button>

<?php else: ?>
    <!-- Si no hay sesión, muestra botones de registro e inicio de sesión -->
    <button id="btn_registro">Registrarse</button>
    <button id="btn_login">Iniciar sesión</button>
<?php endif; ?>
```

El botón "Mi perfil" detecta el rol del usuario:
```php
// Si el rol es 'coach', redirige al perfil de coach con su id
// Si es cliente, redirige al perfil de cliente
$_SESSION['usuario_rol'] === 'coach'
    ? '/perfil_coach/perfil_coach.php?id=' . $_SESSION['coach_id']
    : '/perfil/perfil.php'
```

---

## Sección hero

```html
<section id="inicio">
    <div id="hero_content">
        <!-- Etiqueta superior en mayúsculas -->
        <p id="hero_tag">Plataforma de coaching gaming</p>

        <!-- Título principal con saltos de línea deliberados -->
        <h1 id="hero_titulo">Entrena.<br>Mejora.<br>Domina.</h1>

        <!-- Descripción breve -->
        <p id="hero_desc">Conecta con los mejores coaches...</p>

        <!-- Dos botones de llamada a la acción -->
        <div id="hero_botones">
            <button class="btn_primary">Empezar ahora</button>
            <button class="btn_secondary">Ver coaches</button>
        </div>
    </div>

    <!-- Carrusel de imágenes a la derecha del hero -->
    <div id="carrusel">...</div>
</section>
```

---

## Carrusel

```html
<div id="carrusel">
    <div id="carrusel_track">
        <!-- Tres imágenes que se deslizan horizontalmente -->
        <img src="imagenes/equipo.png" class="slide">
        <img src="imagenes/coaching_directo.png" class="slide">
        <img src="imagenes/persona_feliz.png" class="slide">
    </div>

    <!-- Puntos indicadores de slide actual -->
    <div id="carrusel_dots">
        <span class="dot activo"></span>
        <span class="dot"></span>
        <span class="dot"></span>
    </div>

    <!-- Botones de navegación izquierda/derecha -->
    <button class="carrusel_btn" id="btn_prev">&#8249;</button>
    <button class="carrusel_btn" id="btn_next">&#8250;</button>
</div>
```
La lógica del carrusel está en `rulethegame.js`. El avance automático ocurre cada 10 segundos.

---

## Estadísticas

```html
<div id="hero_stats">
    <!-- Tres tarjetas con números destacados -->
    <div class="stat_card">
        <span class="stat_num">+500</span>
        <span class="stat_label">Coaches activos</span>
    </div>
    <!-- ... -->
</div>
```
Son datos estáticos de marketing, no se calculan desde la base de datos.

---

## Servicios

```html
<section id="servicios">
    <div id="servicios_grid">
        <!-- 4 tarjetas: las de clase 'destacado' tienen fondo negro -->
        <div class="servicio_card">...</div>
        <div class="servicio_card destacado">...</div>
        <div class="servicio_card">...</div>
        <div class="servicio_card destacado">...</div>
    </div>
</section>
```

---

## Coaches destacados

```html
<section id="coaches">
    <div id="coaches_grid">
        <!-- 3 tarjetas de coaches de ejemplo (datos estáticos) -->
        <div class="coach_card">
            <div class="coach_avatar">JX</div> <!-- Iniciales -->
            <h3>JaveX</h3>
            <span class="coach_juego">Valorant</span>
            ...
        </div>
    </div>
</section>
```
Son tarjetas de muestra con datos fijos. Los coaches reales se gestionan en `/coaches/coaches.php`.

---

## JavaScript cargado

```html
<script src="rulethegame.js"></script>
```
Carga el JS global que controla el scroll suave, la sombra de la navbar al hacer scroll y el carrusel.
