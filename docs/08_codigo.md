# 08 — Documentación sobre código

## Introducción
El código de la página web que esta expuesta a internet consta de lenguaje **php, html, css y javascript**. 
El uso del lenguaje php es uno de los pilares del proyecto, y el uso de este es esencial para poder comunicar lo que el usuario de la página web introduzca como información para la base de datos.\
Todo el código de la página web se encuentra en la máquina vagrant con el servicio NFS (Es importante iniciar esta máquina como la primera del servicio).

## Cabecera de la web

### main.php
Este es el código del fichero que actúa como eje central de la pagina web, es lo primero que ve el usuario nada mas abrir la aplicación. Este va acompañado de **rulethegame.css, db.php, logout.php y tulethegame.js** 
```
<?php session_start(); ?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rule The Game</title>
    <link rel="stylesheet" href="rulethegame.css">
    <link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Nunito:wght@400;600;700&display=swap" rel="stylesheet">
</head>
<body>

    <!-- NAVBAR -->
    <header id="navbar">
        <div id="logo">Rule<span>The</span>Game</div>
        <nav>
            <a href="#inicio">Inicio</a>
            <a href="#servicios">Servicios</a>
            <a href="#coaches">Coaches</a>
        </nav>
        <div id="navbar_botones">
            <?php if (isset($_SESSION['usuario_id'])): ?>
                <button id="btn_perfil" onclick="window.location.href='perfil/perfil.php'">Mi perfil</button>
                <button id="btn_logout" onclick="window.location.href='logout.php'">Cerrar sesión</button>
            <?php else: ?>
                <a href="pag_registro/registro.php">
                    <button id="btn_registro">Registrarse</button>
                </a>
                <a href="inicio_sesion/inicio_sesion.php">
                    <button id="btn_login">Iniciar sesión</button>
                </a>
            <?php endif; ?>
        </div>
    </header>

    <!-- HERO -->
    <section id="inicio">
        <div id="hero_content">
            <p id="hero_tag">Plataforma de coaching gaming</p>
            <h1 id="hero_titulo">Entrena.<br>Mejora.<br>Domina.</h1>
            <p id="hero_desc">Conecta con los mejores coaches profesionales y lleva tu nivel al máximo.</p>
            <div id="hero_botones">
                <button class="btn_primary">Empezar ahora</button>
                <button class="btn_secondary">Ver coaches</button>
            </div>
        </div>
        <div id="carrusel">
            <div id="carrusel_track">
                <img src="imagenes/equipo.png" alt="Slide 1" class="slide">
                <img src="imagenes/coaching_directo.png" alt="Slide 2" class="slide">
                <img src="imagenes/persona_feliz.png" alt="Slide 3" class="slide">
            </div>
            <div id="carrusel_dots">
                <span class="dot activo"></span>
                <span class="dot"></span>
                <span class="dot"></span>
            </div>
            <button class="carrusel_btn" id="btn_prev">&#8249;</button>
            <button class="carrusel_btn" id="btn_next">&#8250;</button>
        </div>
    </section>

    <!-- STATS -->
    <div id="hero_stats">
        <div class="stat_card"><span class="stat_num">+500</span><span class="stat_label">Coaches activos</span></div>
        <div class="stat_card"><span class="stat_num">12K</span><span class="stat_label">Jugadores</span></div>
        <div class="stat_card"><span class="stat_num">98%</span><span class="stat_label">Satisfacción</span></div>
    </div>

    <!-- SERVICIOS -->
    <section id="servicios">
        <h2 class="section_titulo">Nuestros Servicios</h2>
        <p class="section_sub">Todo lo que necesitas para mejorar tu juego</p>
        <div id="servicios_grid">
            <div class="servicio_card">
                <div class="servicio_icono">🎯</div>
                <h3>Sesiones 1 a 1</h3>
                <p>Entrenamientos personalizados con coaches profesionales adaptados a tu nivel y objetivos.</p>
            </div>
            <div class="servicio_card destacado">
                <div class="servicio_icono">🏆</div>
                <h3>Análisis de partidas</h3>
                <p>Revisión detallada de tus replays para identificar errores y oportunidades de mejora.</p>
            </div>
            <div class="servicio_card">
                <div class="servicio_icono">📈</div>
                <h3>Plan de entrenamiento</h3>
                <p>Rutinas y ejercicios diseñados específicamente para mejorar más rápido.</p>
            </div>
            <div class="servicio_card destacado">
                <div class="servicio_icono">🤝</div>
                <h3>Comunidad</h3>
                <p>Accede a nuestra comunidad privada de jugadores y comparte tu progreso.</p>
            </div>
        </div>
    </section>

    <!-- COACHES -->
    <section id="coaches">
        <h2 class="section_titulo">Coaches Destacados</h2>
        <p class="section_sub">Aprende de los mejores jugadores</p>
        <div id="coaches_grid">
            <div class="coach_card">
                <div class="coach_avatar">JX</div>
                <h3>JaveX</h3>
                <span class="coach_juego">Valorant</span>
                <p>Radiante Top 50 EU. Especialista en duelistas y mentalidad competitiva.</p>
                <div class="coach_rating">★★★★★</div>
                <button class="btn_coach" onclick="window.location.href='perfil_coach/perfil_coach.php?id=1'">Ver perfil</button>
            </div>
            <div class="coach_card">
                <div class="coach_avatar">NV</div>
                <h3>NightViper</h3>
                <span class="coach_juego">League of Legends</span>
                <p>Challenger 500LP. Mid lane y macro game. Más de 300 alumnos formados.</p>
                <div class="coach_rating">★★★★★</div>
                <button class="btn_coach" onclick="window.location.href='perfil_coach/perfil_coach.php?id=1'">Ver perfil</button>
            </div>
            <div class="coach_card">
                <div class="coach_avatar">SR</div>
                <h3>ShadowRex</h3>
                <span class="coach_juego">CS2</span>
                <p>Faceit Level 10. AWPer profesional con experiencia en torneos internacionales.</p>
                <div class="coach_rating">★★★★☆</div>
                <button class="btn_coach" onclick="window.location.href='perfil_coach/perfil_coach.php?id=1'">Ver perfil</button>
            </div>
        </div>
    </section>

    <!-- FOOTER -->
    <footer id="footer">
        <div id="footer_logo">Rule<span>The</span>Game</div>
        <p>© 2026 RuleTheGame. Todos los derechos reservados.</p>
        <div id="footer_links">
            <a href="contacto/contacto.php">Contacto</a>
            <a href="lorem_ipsu/lorem_ipsu.php">Privacidad</a>
            <a href="lorem_ipsu/lorem_ipsu.php">Términos</a>
            <a href="lorem_ipsu/lorem_ipsu.php">Soporte</a>
        </div>
    </footer>

    <script src="rulethegame.js"></script>

    <!-- Estilos navbar logado (reutiliza los de perfil.css si está disponible) -->
    <style>
        #btn_perfil {
            padding: 9px 20px;
            border-radius: 8px;
            border: 1.5px solid var(--acento);
            background: transparent;
            color: var(--acento);
            font-weight: 700;
            font-size: 14px;
            font-family: 'Nunito', sans-serif;
            cursor: pointer;
            transition: all 0.2s;
        }
        #btn_perfil:hover { background: var(--acento); color: var(--blanco); }
        #btn_logout {
            padding: 9px 20px;
            border-radius: 8px;
            border: none;
            background: var(--negro);
            color: var(--blanco);
            font-weight: 700;
            font-size: 14px;
            font-family: 'Nunito', sans-serif;
            cursor: pointer;
            transition: opacity 0.2s;
        }
        #btn_logout:hover { opacity: 0.75; }
    </style>
</body>
</html>

```
```
<?php session_start(); ?>
```
Esta linea arranca el sistema de sesiones de PHP. Las sesiones son la forma en la que PHP recuerda a un usuario que inicio sesión entre páginas. Sin esta caracteristica $_SESSION esta siempre vacío.
```
<?php if (isset($_SESSION['usuario_id'])): ?>
    <button ...>Mi perfil</button>
    <button ...>Cerrar sesión</button>
<?php else: ?>
    <button ...>Registrarse</button>
    <button ...>Iniciar sesión</button>
<?php endif; ?>
```
Comprueba que existe la variable de sesión usuario_id. Si existe, el usuario esta logado y se muestran los botones de perfil y logout.

### db.php

Este fichero continene el codigo php necesario para poder acceder a la base de datos de la máquina virtual
```
<?php
// Configuración de la base de datos
$host = "10.0.0.100";
$user = "rtg_php";
$pass = "CambiarPasswordPHP!";
$db   = "proyecto_asir";

// Crear la conexión
$conn = new mysqli($host, $user, $pass, $db);

// Verificar la conexión
if ($conn->connect_error) {
    die("Error de conexión: " . $conn->connect_error);
}

// Establecer el conjunto de caracteres a utf8mb4
$conn->set_charset("utf8mb4");
?>
```
Este archivo es especialmente importante para la infraestructura . Con las linea
```
$conn = new mysqli($host, $user, $pass, $db);
```
se crea una conexión a la base de datos MySQL usando la extensión MySQLi ( MySQL Improved). El objeto _$conn_ es el que se usa en el resto de archivos para realizar las consultas.
```
if ($conn->connect_error) {
    die("Error de conexión: " . $conn->connect_error);
}
```
Esta condicional consta de si se produce una fallo en la conexión se ejecuta _die()_ deteniendo la completa ejecución del script y muestra un mensaje de error.
### logout.php
Este fichero contiene el codigo de acción que realiza el botón de _Cerrar Sesión_ cuando este se pulsa en la página web.
```
<?php
session_start();
session_destroy();
header('Location: main.php');
exit;
```
Primero arranca la sesión, luego destruye eliminando los datos almacenados en ella. 
```
header('Location: main.php');
```
hace una redirección al HTTP enviandolo al main.php. Exit corta la ejecución inmediatamente después, evitando ejecutar más código.

### ruletheegame.css
Este fichero contiene todo el codigo css para **main.php**
(:root): Todo el proyecto usa variables CSS definidas en rulethegame.css. Esto es una buena práctica porque si cambias --acento: #ff3c3c en un solo sitio, cambia en toda la web. Todas las páginas importan este archivo base.
position: sticky en el navbar: El navbar permanece fijo en la parte superior al hacer scroll sin necesidad de JavaScript gracias a position: sticky; top: 0.
```
/* ===== RESET ===== */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

/* ===== VARIABLES ===== */
:root {
    --negro: #0d0d0d;
    --blanco: #ffffff;
    --gris_claro: #f5f5f7;
    --gris_medio: #e8e8ed;
    --gris_texto: #6e6e73;
    --acento: #ff3c3c;
    --acento_hover: #e02020;
    --acento_suave: #fff0f0;
    --sombra: 0 4px 24px rgba(0,0,0,0.08);
    --sombra_hover: 0 8px 40px rgba(0,0,0,0.15);
    --radio: 12px;
}

/* ===== BASE ===== */
body {
    font-family: 'Nunito', sans-serif;
    background-color: var(--blanco);
    color: var(--negro);
    line-height: 1.6;
}

/* ===== NAVBAR ===== */
#navbar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 18px 60px;
    background: var(--blanco);
    border-bottom: 1px solid var(--gris_medio);
    position: sticky;
    top: 0;
    z-index: 100;
}

#logo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 28px;
    letter-spacing: 2px;
    color: var(--negro);
}

#logo span {
    color: var(--acento);
}

nav {
    display: flex;
    gap: 36px;
}

nav a {
    text-decoration: none;
    color: var(--gris_texto);
    font-weight: 600;
    font-size: 15px;
    transition: color 0.2s;
}

nav a:hover {
    color: var(--acento);
}

#navbar_botones {
    display: flex;
    gap: 12px;
    align-items: center;
}

#btn_registro {
    background: transparent;
    color: var(--negro);
    border: 2px solid var(--gris_medio);
    padding: 9px 20px;
    border-radius: 8px;
    font-family: 'Nunito', sans-serif;
    font-weight: 700;
    font-size: 14px;
    cursor: pointer;
    transition: all 0.2s;
}

#btn_registro:hover {
    border-color: var(--negro);
}

#btn_login {
    background: var(--negro);
    color: var(--blanco);
    border: none;
    padding: 10px 24px;
    border-radius: 8px;
    font-family: 'Nunito', sans-serif;
    font-weight: 700;
    font-size: 14px;
    cursor: pointer;
    transition: background 0.2s;
}

#btn_login:hover {
    background: var(--acento);
}

/* ===== HERO ===== */
#inicio {
    min-height: 88vh;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    gap: 60px;
    padding: 80px 60px;
    background: var(--gris_claro);
}

#hero_content {
    flex: 1;
}

#hero_tag {
    font-size: 13px;
    font-weight: 700;
    letter-spacing: 3px;
    text-transform: uppercase;
    color: var(--acento);
    margin-bottom: 20px;
}

#hero_titulo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 100px;
    line-height: 0.95;
    letter-spacing: 2px;
    color: var(--negro);
    margin-bottom: 28px;
}

#hero_desc {
    font-size: 18px;
    color: var(--gris_texto);
    max-width: 420px;
    margin-bottom: 40px;
}

#hero_botones {
    display: flex;
    gap: 16px;
}

.btn_primary {
    background: var(--acento);
    color: var(--blanco);
    border: none;
    padding: 14px 32px;
    border-radius: var(--radio);
    font-family: 'Nunito', sans-serif;
    font-weight: 700;
    font-size: 16px;
    cursor: pointer;
    transition: background 0.2s, transform 0.1s;
}

.btn_primary:hover {
    background: var(--acento_hover);
    transform: translateY(-2px);
}

.btn_secondary {
    background: transparent;
    color: var(--negro);
    border: 2px solid var(--negro);
    padding: 14px 32px;
    border-radius: var(--radio);
    font-family: 'Nunito', sans-serif;
    font-weight: 700;
    font-size: 16px;
    cursor: pointer;
    transition: all 0.2s;
}

.btn_secondary:hover {
    background: var(--negro);
    color: var(--blanco);
}

/* ===== CARRUSEL ===== */
#carrusel {
    flex: 1;
    position: relative;
    border-radius: 16px;
    overflow: hidden;
    max-width: 560px;
    aspect-ratio: 16/9;
    box-shadow: var(--sombra_hover);
    background: var(--gris_medio);
}

#carrusel_track {
    display: flex;
    width: 100%;
    height: 100%;
    transition: transform 0.5s ease;
}

.slide {
    min-width: 100%;
    height: 100%;
    object-fit: cover;
}

.carrusel_btn {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    background: rgba(0,0,0,0.5);
    color: white;
    border: none;
    font-size: 32px;
    padding: 8px 16px;
    cursor: pointer;
    border-radius: 8px;
    transition: background 0.2s;
    z-index: 10;
}

.carrusel_btn:hover {
    background: rgba(0,0,0,0.85);
}

#btn_prev { left: 12px; }
#btn_next { right: 12px; }

#carrusel_dots {
    position: absolute;
    bottom: 12px;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    gap: 8px;
    z-index: 10;
}

.dot {
    width: 10px;
    height: 10px;
    background: rgba(255,255,255,0.5);
    border-radius: 50%;
    cursor: pointer;
    transition: background 0.2s;
}

.dot.activo {
    background: white;
}

/* ===== STATS ===== */
#hero_stats {
    display: flex;
    gap: 0;
    background: var(--blanco);
    border-bottom: 1px solid var(--gris_medio);
}

.stat_card {
    flex: 1;
    padding: 28px 40px;
    display: flex;
    flex-direction: column;
    align-items: center;
    border-right: 1px solid var(--gris_medio);
}

.stat_card:last-child {
    border-right: none;
}

.stat_num {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 48px;
    color: var(--acento);
    line-height: 1;
}

.stat_label {
    font-size: 13px;
    color: var(--gris_texto);
    font-weight: 600;
    margin-top: 4px;
}

/* ===== SECCIONES GENERALES ===== */
.section_titulo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 52px;
    letter-spacing: 1px;
    text-align: center;
    margin-bottom: 10px;
}

.section_sub {
    text-align: center;
    color: var(--gris_texto);
    font-size: 16px;
    margin-bottom: 52px;
}

/* ===== SERVICIOS ===== */
#servicios {
    padding: 100px 60px;
    background: var(--blanco);
}

#servicios_grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 24px;
}

.servicio_card {
    background: var(--gris_claro);
    border-radius: var(--radio);
    padding: 36px 28px;
    transition: box-shadow 0.2s, transform 0.2s;
}

.servicio_card:hover {
    box-shadow: var(--sombra_hover);
    transform: translateY(-4px);
}

.servicio_card.destacado {
    background: var(--negro);
    color: var(--blanco);
}

.servicio_card.destacado p {
    color: #aaa;
}

.servicio_icono {
    font-size: 36px;
    margin-bottom: 18px;
}

.servicio_card h3 {
    font-size: 18px;
    font-weight: 700;
    margin-bottom: 10px;
}

.servicio_card p {
    font-size: 14px;
    color: var(--gris_texto);
    line-height: 1.6;
}

/* ===== COACHES ===== */
#coaches {
    padding: 100px 60px;
    background: var(--gris_claro);
}

#coaches_grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 28px;
}

.coach_card {
    background: var(--blanco);
    border-radius: var(--radio);
    padding: 36px 28px;
    text-align: center;
    box-shadow: var(--sombra);
    transition: box-shadow 0.2s, transform 0.2s;
}

.coach_card:hover {
    box-shadow: var(--sombra_hover);
    transform: translateY(-4px);
}

.coach_avatar {
    width: 72px;
    height: 72px;
    background: var(--negro);
    color: var(--blanco);
    font-family: 'Bebas Neue', sans-serif;
    font-size: 28px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0 auto 16px;
}

.coach_card h3 {
    font-size: 20px;
    font-weight: 700;
    margin-bottom: 6px;
}

.coach_juego {
    display: inline-block;
    background: var(--acento_suave);
    color: var(--acento);
    font-size: 12px;
    font-weight: 700;
    padding: 4px 12px;
    border-radius: 20px;
    margin-bottom: 14px;
}

.coach_card p {
    font-size: 14px;
    color: var(--gris_texto);
    margin-bottom: 14px;
}

.coach_rating {
    color: #f5a623;
    font-size: 16px;
    margin-bottom: 20px;
}

.btn_coach {
    background: transparent;
    border: 2px solid var(--negro);
    color: var(--negro);
    padding: 9px 24px;
    border-radius: 8px;
    font-family: 'Nunito', sans-serif;
    font-weight: 700;
    font-size: 14px;
    cursor: pointer;
    transition: all 0.2s;
}

.btn_coach:hover {
    background: var(--negro);
    color: var(--blanco);
}
/* ===== FOOTER ===== */
#footer {
    background: var(--negro);
    color: var(--blanco);
    padding: 40px 60px;
    display: flex;
    align-items: center;
    justify-content: space-between;
}

#footer_logo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 24px;
    letter-spacing: 2px;
}

#footer_logo span {
    color: var(--acento);
}

#footer p {
    font-size: 13px;
    color: #888;
}

#footer_links {
    display: flex;
    gap: 24px;
}

#footer_links a {
    text-decoration: none;
    color: #888;
    font-size: 13px;
    font-weight: 600;
    transition: color 0.2s;
}

#footer_links a:hover {
    color: var(--acento);
}
``` 
### rulethegame.js
Código **javascript** para main.php
```
// ===== SCROLL SUAVE =====
document.querySelectorAll('a[href^="#"]').forEach(enlace => {
    enlace.addEventListener('click', function(e) {
        e.preventDefault();
        const destino = document.querySelector(this.getAttribute('href'));
        if (destino) destino.scrollIntoView({ behavior: 'smooth' });
    });
});

// ===== NAVBAR SOMBRA AL SCROLL =====
window.addEventListener('scroll', () => {
    const navbar = document.getElementById('navbar');
    navbar.style.boxShadow = window.scrollY > 10 ? '0 2px 20px rgba(0,0,0,0.1)' : 'none';
});

// ===== CARRUSEL =====
const track = document.getElementById('carrusel_track');
const dots = document.querySelectorAll('.dot');
const slides = document.querySelectorAll('.slide');
let actual = 0;
const total = slides.length;

function irA(index) {
    actual = (index + total) % total;
    track.style.transform = `translateX(-${actual * 100}%)`;
    dots.forEach(d => d.classList.remove('activo'));
    dots[actual].classList.add('activo');
}

document.getElementById('btn_next').addEventListener('click', () => irA(actual + 1));
document.getElementById('btn_prev').addEventListener('click', () => irA(actual - 1));
dots.forEach((dot, i) => dot.addEventListener('click', () => irA(i)));

// Avance automático cada 4 segundos
setInterval(() => irA(actual + 1), 10000);
```
Scroll Suave
```
document.querySelectorAll('a[href^="#"]').forEach(enlace => {
    enlace.addEventListener('click', function(e) {
        e.preventDefault();
        const destino = document.querySelector(this.getAttribute('href'));
        if (destino) destino.scrollIntoView({ behavior: 'smooth' });
    });
});
```
Selecciona todos los enlaces cuyo href empieza por # (los del menú de navegación: Inicio, Servicios, Coaches). Al hacer clic, cancela el comportamiento por defecto del navegador (e.preventDefault()) que haría un salto brusco, y en su lugar hace un scroll animado hasta la sección correspondiente.
Sombra del navbar al hacer scroll
```
window.addEventListener('scroll', () => {
    const navbar = document.getElementById('navbar');
    navbar.style.boxShadow = window.scrollY > 10 ? '0 2px 20px rgba(0,0,0,0.1)' : 'none';
});
```
Escucha el evento de scroll de la ventana. Si el usuario ha bajado más de 10 píxeles (window.scrollY > 10), añade una sombra al navbar para que visualmente "se separe" del contenido. Si está en el top, elimina la sombra. El operador ternario ? : es un if-else en una línea. /

Carrusel de imágenes
```
let actual = 0;
const total = slides.length;

function irA(index) {
    actual = (index + total) % total;
    track.style.transform = `translateX(-${actual * 100}%)`;
    dots.forEach(d => d.classList.remove('activo'));
    dots[actual].classList.add('activo');
}
```
La función irA(index) es el núcleo del carrusel. El truco de (index + total) % total sirve para que el índice sea siempre circular: si estás en el slide 0 y pulsas "anterior", (0 + 3) % 3 = 0... espera, sería 0, pero si estás en el 0 y haces -1, (-1 + 3) % 3 = 2, que es el último slide. Funciona como un bucle sin fin.
El movimiento se consigue desplazando el #carrusel_track con translateX. Como cada slide ocupa el 100% del ancho, mover -100% muestra el segundo, -200% el tercero, etc. El CSS tiene transition: transform 0.5s ease que hace que el movimiento sea animado.
Los puntos (dots) se actualizan quitando y añadiendo la clase activo al punto correspondiente.

## Páginas auxiliares
El resto de ficheros se encuntran en ficheros _.php y .css_ diferentes que se encuentran entre carpetas. 
La disposicion de estos ficheros es la siguiente: 
![Imagen de la estructura de ficheros](imagen_estructura_ficheros.png)
### Contacto
Esta carpeta esta compuesta por los archivos que cumplen con la función de aplicación de incidencias de la página web. Consta de un formulario donde el usuario pone los detalles de su problema y este se registra en la **Base de Datos**. 
#### contacto.php
```
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Contacto - Rule The Game</title>
    <link rel="stylesheet" href="../rulethegame.css">
    <link rel="stylesheet" href="contacto.css">
    <link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Nunito:wght@400;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">
</head>
<body>

    <!-- NAVBAR -->
    <header id="navbar">
        <a href="../main.php"><div id="logo">Rule<span>The</span>Game</div></a>
        <nav>
            <a href="../main.php">Inicio</a>
            <a href="../main.php#servicios">Servicios</a>
            <a href="../main.php#coaches">Coaches</a>
        </nav>
        <div id="navbar_botones">
            <button id="btn_registro" onclick="window.location.href='../pag_registro/registro.php'">Registrarse</button>
            <button id="btn_login" onclick="window.location.href='../inicio_sesion/inicio_sesion.php'">Iniciar sesión</button>
        </div>
    </header>

    <!-- HERO CONTACTO -->
    <section id="contacto_hero">
        <p id="contacto_tag">Estamos aquí para ayudarte</p>
        <h1 id="contacto_titulo">Contacto</h1>
        <p id="contacto_desc">¿Tienes alguna duda o incidencia? Escríbenos y te responderemos lo antes posible.</p>
    </section>

    <!-- CONTENIDO PRINCIPAL -->
    <section id="contacto_main">

        <!-- INFO -->
        <div id="contacto_info">
            <h2 id="info_titulo">Encuéntranos</h2>
            <p id="info_desc">Puedes contactarnos a través de cualquiera de estos canales. Nuestro equipo de soporte está disponible de lunes a viernes de 9:00 a 20:00.</p>

            <div id="info_items">

                <div class="info_item">
                <div class="info_icono">
                    <i class="fa-solid fa-envelope"></i>
                </div>
                <div class="info_texto">
                    <span class="info_label">Email</span>
                    <span class="info_valor">soporte@rulethegame.com</span>
                </div>
            </div>

            <div class="info_item">
                <div class="info_icono">
                    <i class="fa-brands fa-instagram"></i>
                </div>
                <div class="info_texto">
                    <span class="info_label">Instagram</span>
                    <span class="info_valor">@rulethegame</span>
                </div>
            </div>

            <div class="info_item">
                <div class="info_icono">
                    <i class="fa-brands fa-x-twitter"></i>
                </div>
                <div class="info_texto">
                    <span class="info_label">Twitter / X</span>
                    <span class="info_valor">@rulethegame</span>
                </div>
            </div>

            <div class="info_item">
                <div class="info_icono">
                    <i class="fa-brands fa-facebook-f"></i>
                </div>
                <div class="info_texto">
                    <span class="info_label">Facebook</span>
                    <span class="info_valor">RuleTheGame</span>
                </div>
            </div>

            </div>

            <div id="info_horario">
                <span>🕘</span>
                <span>Lunes a Viernes · 9:00 – 20:00</span>
            </div>
        </div>

        <!-- FORMULARIO DE INCIDENCIAS -->
        <div id="incidencias_card">
            <h2 id="incidencias_titulo">Enviar incidencia</h2>
            <div id="incidencias_form">

                <div class="form_fila">
                    <div class="form_grupo">
                        <label for="inc_nombre">Nombre</label>
                        <input type="text" id="inc_nombre" name="inc_nombre" placeholder="Tu nombre" class="input_field">
                    </div>
                    <div class="form_grupo">
                        <label for="inc_email">Email</label>
                        <input type="email" id="inc_email" name="inc_email" placeholder="tucorreo@email.com" class="input_field">
                    </div>
                </div>

                <div class="form_grupo">
                    <label for="inc_tipo">Tipo de incidencia</label>
                    <select id="inc_tipo" name="inc_tipo" class="input_field">
                        <option value="" disabled selected>Selecciona una categoría</option>
                        <option value="cuenta">Problema con mi cuenta</option>
                        <option value="pago">Problema con un pago</option>
                        <option value="coach">Incidencia con un coach</option>
                        <option value="tecnico">Problema técnico</option>
                        <option value="otro">Otro</option>
                    </select>
                </div>

                <div class="form_grupo">
                    <label for="inc_asunto">Asunto</label>
                    <input type="text" id="inc_asunto" name="inc_asunto" placeholder="Resumen breve del problema" class="input_field">
                </div>

                <div class="form_grupo">
                    <label for="inc_mensaje">Descripción</label>
                    <textarea id="inc_mensaje" name="inc_mensaje" placeholder="Describe tu incidencia con el mayor detalle posible..." class="input_field" rows="5"></textarea>
                </div>

                <button class="btn_primary" id="btn_enviar">Enviar incidencia</button>

                <p id="form_nota">Al enviar este formulario aceptas nuestra <a href="../lorem_ipsu/lorem_ipsu.php">política de privacidad</a>.</p>

            </div>
        </div>

    </section>

    <!-- FOOTER -->
    <footer id="footer">
        <div id="footer_logo">Rule<span>The</span>Game</div>
        <p>© 2026 RuleTheGame. Todos los derechos reservados.</p>
        <div id="footer_links">
            <a href="contacto.php">Contacto</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Privacidad</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Términos</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Soporte</a>
        </div>
    </footer>

</body>
</html>
```
#### contacto.css

```
/* ===== HERO CONTACTO ===== */
#contacto_hero {
    background: var(--gris_claro);
    padding: 80px 60px 60px;
    text-align: center;
    position: relative;
    overflow: hidden;
}

#contacto_hero::before {
    content: 'RTG';
    font-family: 'Bebas Neue', sans-serif;
    font-size: 300px;
    color: rgba(0,0,0,0.04);
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    pointer-events: none;
    line-height: 1;
}

#contacto_tag {
    font-size: 13px;
    font-weight: 700;
    letter-spacing: 3px;
    text-transform: uppercase;
    color: var(--acento);
    margin-bottom: 14px;
}

#contacto_titulo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 80px;
    letter-spacing: 2px;
    color: var(--negro);
    line-height: 1;
    margin-bottom: 16px;
}

#contacto_desc {
    font-size: 17px;
    color: var(--gris_texto);
    max-width: 500px;
    margin: 0 auto;
}

/* ===== CONTENIDO PRINCIPAL ===== */
#contacto_main {
    display: grid;
    grid-template-columns: 1fr 1.6fr;
    gap: 40px;
    padding: 80px 60px;
    background: var(--blanco);
    align-items: start;
}

/* ===== INFO ===== */
#contacto_info {
    background: var(--negro);
    border-radius: 20px;
    padding: 48px 36px;
    color: var(--blanco);
    position: sticky;
    top: 100px;
}

#info_titulo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 36px;
    letter-spacing: 1px;
    margin-bottom: 12px;
}

#info_desc {
    font-size: 14px;
    color: #aaa;
    line-height: 1.7;
    margin-bottom: 36px;
}

#info_items {
    display: flex;
    flex-direction: column;
    gap: 24px;
    margin-bottom: 36px;
}

.info_item {
    display: flex;
    align-items: center;
    gap: 16px;
}

.info_icono {
    width: 44px;
    height: 44px;
    background: rgba(255,255,255,0.08);
    border-radius: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 20px;
    flex-shrink: 0;
}

.info_texto {
    display: flex;
    flex-direction: column;
    gap: 2px;
}

.info_label {
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 1.5px;
    text-transform: uppercase;
    color: var(--acento);
}

.info_valor {
    font-size: 15px;
    color: var(--blanco);
    font-weight: 600;
}

#info_horario {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 16px 20px;
    background: rgba(255,255,255,0.06);
    border-radius: 10px;
    font-size: 14px;
    color: #ccc;
    border-left: 3px solid var(--acento);
}

/* ===== FORMULARIO INCIDENCIAS ===== */
#incidencias_card {
    background: var(--blanco);
    border: 2px solid var(--gris_medio);
    border-radius: 20px;
    padding: 48px 40px;
}

#incidencias_titulo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 40px;
    letter-spacing: 1px;
    color: var(--negro);
    margin-bottom: 8px;
}

#incidencias_sub {
    font-size: 15px;
    color: var(--gris_texto);
    margin-bottom: 36px;
}

#incidencias_form {
    display: flex;
    flex-direction: column;
    gap: 20px;
}

.form_fila {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 16px;
}

.form_grupo {
    display: flex;
    flex-direction: column;
    gap: 6px;
}

.form_grupo label {
    font-size: 13px;
    font-weight: 700;
    color: var(--negro);
    letter-spacing: 0.5px;
    text-transform: uppercase;
}

.input_field {
    width: 100%;
    padding: 13px 16px;
    border: 2px solid var(--gris_medio);
    border-radius: var(--radio);
    font-family: 'Nunito', sans-serif;
    font-size: 15px;
    color: var(--negro);
    outline: none;
    transition: border-color 0.2s;
    background: var(--blanco);
    resize: none;
}

.input_field:focus {
    border-color: var(--acento);
}

select.input_field {
    cursor: pointer;
    appearance: none;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath d='M1 1l5 5 5-5' stroke='%236e6e73' stroke-width='2' fill='none' stroke-linecap='round'/%3E%3C/svg%3E");
    background-repeat: no-repeat;
    background-position: right 16px center;
    padding-right: 40px;
}

#btn_enviar {
    width: 100%;
    padding: 16px;
    font-size: 17px;
    margin-top: 4px;
}

#form_nota {
    text-align: center;
    font-size: 13px;
    color: var(--gris_texto);
}

#form_nota a {
    color: var(--acento);
    font-weight: 700;
    text-decoration: none;
}

#form_nota a:hover {
    text-decoration: underline;
}
```
```
#contacto_hero::before {
    content: 'RTG';
    font-size: 300px;
    color: rgba(0,0,0,0.04);
    position: absolute;
    ...
}
```
Crea un texto decorativo gigante ("RTG") en el fondo de la sección de contacto usando solo CSS, sin necesidad de un elemento HTML extra.

```
select.input_field {
    appearance: none;
    background-image: url("data:image/svg+xml,...");
}
```
appearance: none elimina el estilo nativo del sistema operativo en el <select>, y luego se añade una flecha personalizada mediante una imagen SVG codificada en base64 directamente en el CSS. Es un truco habitual para tener selectores visualmente consistentes en todos los navegadores.
min-height: calc(100vh - 73px - 97px): En las páginas de login, registro y perfil, la sección principal calcula su altura restando la altura del navbar y el footer al 100% de la ventana. Esto garantiza que el contenido siempre ocupe todo el espacio visible aunque haya poco contenido, evitando que el footer "flote" a mitad de pantalla.

### imagenes
Esta carpeta guarda las imágenes que se usan en el carrusel de la página web. Dentro del codigo de main.php se encuentra la url a estas imganes
### Inicio de sesión
Guarda los archivos realcionados con el botón de main.php para iniciar sesión. Este boton accede a la información de la Base de Datos y cambia el estado de la página si el inicio de sesión es correcto.
#### inicio_sesión.php
```
<?php
// ── Lógica de inicio de sesión ──────────────────────────────
session_start();
require_once '../db.php';

// Si ya está logado, redirigir al inicio
if (isset($_SESSION['usuario_id'])) {
    header('Location: ../main.php');
    exit;
}

$error = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $login    = trim($_POST['email']    ?? '');
    $password = $_POST['password'] ?? '';

    if (!$login || !$password) {
        $error = 'Por favor, introduce tu email/usuario y contraseña.';
    } else {
        // Buscar por email O por nombre de usuario
        $stmt = $conn->prepare(
            "SELECT id, nombre, nom_usu, password FROM usuarios WHERE email = ? OR nom_usu = ?"
        );
        $stmt->bind_param("ss", $login, $login);
        $stmt->execute();
        $result = $stmt->get_result();

        if ($row = $result->fetch_assoc()) {
            if (password_verify($password, $row['password'])) {
                // Login correcto — guardar sesión
                $_SESSION['usuario_id']  = $row['id'];
                $_SESSION['usuario_nom'] = $row['nombre'];
                $_SESSION['usuario_usr'] = $row['nom_usu'];
                header('Location: ../main.php');
                exit;
            } else {
                $error = 'Contraseña incorrecta.';
            }
        } else {
            $error = 'No existe ninguna cuenta con ese email o usuario.';
        }
        $stmt->close();
    }
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Iniciar Sesión - Rule The Game</title>
    <link rel="stylesheet" href="../rulethegame.css">
    <link rel="stylesheet" href="inicio_sesion.css">
    <link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Nunito:wght@400;600;700&display=swap" rel="stylesheet">
</head>
<body>
    <header id="navbar">
        <a href="../main.php"><div id="logo">Rule<span>The</span>Game</div></a>
        <nav>
            <a href="../main.php">Inicio</a>
            <a href="../main.php#servicios">Servicios</a>
            <a href="../main.php#coaches">Coaches</a>
        </nav>
        <div id="navbar_botones">
            <button id="btn_registro" onclick="window.location.href='../pag_registro/registro.php'">Registrarse</button>
            <button id="btn_login" onclick="window.location.href='inicio_sesion.php'">Iniciar sesión</button>
        </div>
    </header>

    <section id="login_section">
        <div id="login_card">
            <div id="login_header">
                <div id="login_logo">Rule<span>The</span>Game</div>
                <h1 id="login_titulo">Bienvenido de nuevo</h1>
                <p id="login_sub">Inicia sesión para acceder a tu cuenta</p>
            </div>

            <?php if ($error): ?>
                <div style="background:#fee;color:#c00;padding:12px 16px;border-radius:8px;margin-bottom:16px;font-size:14px;">
                    ⚠️ <?= htmlspecialchars($error) ?>
                </div>
            <?php endif; ?>

            <form id="login_form" method="POST" action="inicio_sesion.php">
                <div class="form_grupo">
                    <label for="email">Email o usuario</label>
                    <input type="text" id="email" name="email" placeholder="tucorreo@email.com" class="input_field"
                           value="<?= htmlspecialchars($_POST['email'] ?? '') ?>" required>
                </div>
                <div class="form_grupo">
                    <label for="password">Contraseña</label>
                    <div id="password_wrapper">
                        <input type="password" id="password" name="password" placeholder="Tu contraseña" class="input_field" required>
                        <button type="button" id="toggle_password" onclick="togglePassword()">👁️</button>
                    </div>
                </div>
                <div id="login_opciones">
                    <label id="recordar_label">
                        <input type="checkbox" id="recordar" name="recordar">
                        Recuérdame
                    </label>
                    <a href="javascript:void(0)" id="link_olvide">¿Olvidaste tu contraseña?</a>
                </div>
                <button type="submit" class="btn_primary" id="btn_entrar">Iniciar sesión</button>
                <div id="login_separador">
                    <span></span><p>o continúa con</p><span></span>
                </div>
                <div id="login_social">
                    <button class="btn_social" type="button">
                        <img src="https://www.google.com/favicon.ico" alt="Google"> Google
                    </button>
                    <button class="btn_social" type="button">
                        <img src="https://store.steampowered.com/favicon.ico" alt="Steam"> Steam
                    </button>
                </div>
                <p id="login_registro">¿No tienes cuenta? <a href="../pag_registro/registro.php">Regístrate gratis</a></p>
            </form>
        </div>
    </section>

    <footer id="footer">
        <div id="footer_logo">Rule<span>The</span>Game</div>
        <p>© 2026 RuleTheGame. Todos los derechos reservados.</p>
        <div id="footer_links">
            <a href="../contacto/contacto.php">Contacto</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Privacidad</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Términos</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Soporte</a>
        </div>
    </footer>

    <script>
        function togglePassword() {
            const input = document.getElementById('password');
            const btn   = document.getElementById('toggle_password');
            input.type  = input.type === 'password' ? 'text' : 'password';
            btn.textContent = input.type === 'password' ? '👁️' : '🙈';
        }
    </script>
</body>
</html>
```
```
function togglePassword() {
    const input = document.getElementById('password');
    const btn   = document.getElementById('toggle_password');
    input.type  = input.type === 'password' ? 'text' : 'password';
    btn.textContent = input.type === 'password' ? '👁️' : '🙈';
}
```
Cambia el campo de tipo password a tipo text.
```
if (isset($_SESSION['usuario_id'])) {
    header('Location: ../main.php');
    exit;
}
```
Comprueba si el usuario ya esta logado antes de mostrarle el formulario. Si ya tiene sesión activa, lo manda directamente al inicio, evitando que acceda a la página de login cuando lo necesita.
```
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
```
Detecta si el formulario ha sido enciado. _$SERVER_ es una variable golbal de PHP que indica el método HTTP de la petición. Cuando el usuario rellena el formulario y plusa "Entrar", el navegador hace una petición POST.
```
$login    = trim($_POST['email']    ?? '');
$password = $_POST['password'] ?? '';
```
Recoge los datos del formulario._trim()_ elimina espacios en blanco al inicio y al final. El operador _??_ devuelve el valor de la izquierda si existe, o el de la derecha si no, evitando errores si el campo no viene en el POST.

```
$stmt = $conn->prepare(
    "SELECT id, nombre, nom_usu, password FROM usuarios WHERE email = ? OR nom_usu = ?"
);
$stmt->bind_param("ss", $login, $login);
$stmt->execute();
```
Aquí está la clave de seguridad: se usa una consulta preparada (prepared statement). En lugar de meter directamente el valor del usuario en el SQL (lo que abriría la puerta a inyección SQL), se pone ? como marcador y luego se vincula con bind_param. El "ss" indica que los dos parámetros son strings. Esto protege completamente contra SQL Injection.
```
if ($row = $result->fetch_assoc()) {
    if (password_verify($password, $row['password'])) {
```
_fetch_assoc()_ devuelve la fila encontrada como un array asociativo. Después, _password_verify()_ compara la contraseña que ha introducido el usuario con el hash guardado en la base de datos. Nunca se guarda la contraseña en texto plano, solo su hash, y esta función hace la comparación correctamente.

```
$_SESSION['usuario_id']  = $row['id'];
$_SESSION['usuario_nom'] = $row['nombre'];
$_SESSION['usuario_usr'] = $row['nom_usu'];
header('Location: ../main.php');
exit;
```
Si todo es correcto, guarda los datos del usuario en la sesión y redirige al inicio. Esos datos de sesión son los que _main.php_ comprueba para mostrar el navbar correcto.


#### inicio_sesion.css
```
/* ===== LOGIN SECTION ===== */
#login_section {
    min-height: calc(100vh - 73px - 97px);
    background: var(--gris_claro);
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 60px 20px;
}

#login_card {
    background: var(--blanco);
    border-radius: 20px;
    box-shadow: var(--sombra_hover);
    padding: 52px 48px;
    width: 100%;
    max-width: 460px;
}

/* ===== HEADER ===== */
#login_header {
    text-align: center;
    margin-bottom: 40px;
}

#login_logo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 32px;
    letter-spacing: 2px;
    color: var(--negro);
    margin-bottom: 20px;
}

#login_logo span {
    color: var(--acento);
}

#login_titulo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 42px;
    letter-spacing: 1px;
    color: var(--negro);
    line-height: 1;
    margin-bottom: 8px;
}

#login_sub {
    color: var(--gris_texto);
    font-size: 15px;
}

/* ===== FORM ===== */
#login_form {
    display: flex;
    flex-direction: column;
    gap: 20px;
}

.form_grupo {
    display: flex;
    flex-direction: column;
    gap: 6px;
}

.form_grupo label {
    font-size: 13px;
    font-weight: 700;
    color: var(--negro);
    letter-spacing: 0.5px;
    text-transform: uppercase;
}

.input_field {
    width: 100%;
    padding: 13px 16px;
    border: 2px solid var(--gris_medio);
    border-radius: var(--radio);
    font-family: 'Nunito', sans-serif;
    font-size: 15px;
    color: var(--negro);
    outline: none;
    transition: border-color 0.2s;
    background: var(--blanco);
}

.input_field:focus {
    border-color: var(--acento);
}

/* ===== PASSWORD WRAPPER ===== */
#password_wrapper {
    position: relative;
    display: flex;
    align-items: center;
}

#password_wrapper .input_field {
    padding-right: 50px;
}

#toggle_password {
    position: absolute;
    right: 14px;
    background: none;
    border: none;
    cursor: pointer;
    font-size: 18px;
    line-height: 1;
    padding: 0;
}

/* ===== OPCIONES ===== */
#login_opciones {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: -6px;
}

#recordar_label {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 14px;
    color: var(--gris_texto);
    cursor: pointer;
    font-weight: 600;
    text-transform: none;
    letter-spacing: 0;
}

#recordar_label input[type="checkbox"] {
    accent-color: var(--acento);
    width: 16px;
    height: 16px;
    cursor: pointer;
}

#link_olvide {
    font-size: 14px;
    color: var(--acento);
    font-weight: 700;
    text-decoration: none;
}

#link_olvide:hover {
    text-decoration: underline;
}

/* ===== BOTÓN PRINCIPAL ===== */
#btn_entrar {
    width: 100%;
    padding: 16px;
    font-size: 17px;
}

/* ===== SEPARADOR ===== */
#login_separador {
    display: flex;
    align-items: center;
    gap: 12px;
}

#login_separador span {
    flex: 1;
    height: 1px;
    background: var(--gris_medio);
}

#login_separador p {
    font-size: 13px;
    color: var(--gris_texto);
    font-weight: 600;
    white-space: nowrap;
}

/* ===== BOTONES SOCIALES ===== */
#login_social {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 12px;
}

.btn_social {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    padding: 12px 16px;
    border: 2px solid var(--gris_medio);
    border-radius: var(--radio);
    background: var(--blanco);
    font-family: 'Nunito', sans-serif;
    font-size: 14px;
    font-weight: 700;
    color: var(--negro);
    cursor: pointer;
    transition: border-color 0.2s, background 0.2s;
}

.btn_social:hover {
    border-color: var(--negro);
    background: var(--gris_claro);
}

.btn_social img {
    width: 18px;
    height: 18px;
    object-fit: contain;
}

/* ===== LINK REGISTRO ===== */
#login_registro {
    text-align: center;
    font-size: 14px;
    color: var(--gris_texto);
}

#login_registro a {
    color: var(--acento);
    font-weight: 700;
    text-decoration: none;
}

#login_registro a:hover {
    text-decoration: underline;
}
```
### Lorem_ipsu
Esta carpeta posee un archivo con el contenido de lorem ipsu su única función es la de rellanar espacio en algunas zonas com mucho texto o en los términos de políticas y privacidad.

### Página de registro
A diferencia de la página de inicio de sesión esta página da una información que se guarda en la Base de Datos. Aquí el usuario rellena la información necesaria para que este pueda acceder a funciones especificas que el usuario sin autenticar no puede hacer.
#### registro.php
```
<?php
// ── Lógica de registro ──────────────────────────────────────
session_start();
require_once '../db.php';

$error   = '';
$success = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $nombre    = trim($_POST['nombre']    ?? '');
    $apellido  = trim($_POST['apellido']  ?? '');
    $nom_usu   = trim($_POST['nom_usu']   ?? '');
    $fecha_nac = trim($_POST['fecha_nac'] ?? '');
    $localidad = trim($_POST['localidad'] ?? '');
    $email     = trim($_POST['email']     ?? '');
    $password  = $_POST['password']       ?? '';

    if (!$nombre || !$apellido || !$nom_usu || !$email || !$password) {
        $error = 'Por favor, rellena todos los campos obligatorios.';
    } elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $error = 'El email no tiene un formato válido.';
    } elseif (strlen($password) < 8) {
        $error = 'La contraseña debe tener al menos 8 caracteres.';
    } else {
        $stmt = $conn->prepare("SELECT id FROM usuarios WHERE email = ? OR nom_usu = ?");
        $stmt->bind_param("ss", $email, $nom_usu);
        $stmt->execute();
        $stmt->store_result();

        if ($stmt->num_rows > 0) {
            $error = 'El email o nombre de usuario ya están registrados.';
        } else {
            $hash = password_hash($password, PASSWORD_DEFAULT);
            $ins  = $conn->prepare(
                "INSERT INTO usuarios (nombre, apellido, nom_usu, fecha_nac, localidad, email, password)
                 VALUES (?, ?, ?, ?, ?, ?, ?)"
            );
            $ins->bind_param("sssssss", $nombre, $apellido, $nom_usu, $fecha_nac, $localidad, $email, $hash);

            if ($ins->execute()) {
                $success = '¡Cuenta creada correctamente! Ya puedes iniciar sesión.';
            } else {
                $error = 'Error al crear la cuenta. Inténtalo de nuevo.';
            }
            $ins->close();
        }
        $stmt->close();
    }
}
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Registro - Rule The Game</title>
    <link rel="stylesheet" href="../rulethegame.css">
    <link rel="stylesheet" href="registro.css">
    <link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Nunito:wght@400;600;700&display=swap" rel="stylesheet">
</head>
<body>
    <header id="navbar">
        <a href="../main.php"><div id="logo">Rule<span>The</span>Game</div></a>
        <nav>
            <a href="../main.php">Inicio</a>
            <a href="../main.php#servicios">Servicios</a>
            <a href="../main.php#coaches">Coaches</a>
        </nav>
        <div id="navbar_botones">
            <button id="btn_registro" onclick="window.location.href='registro.php'">Registrarse</button>
            <button id="btn_login" onclick="window.location.href='../inicio_sesion/inicio_sesion.php'">Iniciar sesión</button>
        </div>
    </header>

    <section id="registro_section">
        <div id="registro_card">
            <div id="registro_header">
                <h1 id="registro_titulo">Crear cuenta</h1>
                <p id="registro_sub">Únete a la comunidad de Rule The Game</p>
            </div>

            <?php if ($error): ?>
                <div style="background:#fee;color:#c00;padding:12px 16px;border-radius:8px;margin-bottom:16px;font-size:14px;">
                    ⚠️ <?= htmlspecialchars($error) ?>
                </div>
            <?php endif; ?>
            <?php if ($success): ?>
                <div style="background:#efe;color:#060;padding:12px 16px;border-radius:8px;margin-bottom:16px;font-size:14px;">
                    ✅ <?= htmlspecialchars($success) ?>
                    <br><a href="../inicio_sesion/inicio_sesion.php">Ir a iniciar sesión →</a>
                </div>
            <?php endif; ?>

            <form id="registro_form" method="POST" action="registro.php">
                <div class="form_fila">
                    <div class="form_grupo">
                        <label for="nombre">Nombre</label>
                        <input type="text" id="nombre" name="nombre" placeholder="Tu nombre" class="input_field"
                               value="<?= htmlspecialchars($_POST['nombre'] ?? '') ?>" required>
                    </div>
                    <div class="form_grupo">
                        <label for="apellido">Apellido</label>
                        <input type="text" id="apellido" name="apellido" placeholder="Tu apellido" class="input_field"
                               value="<?= htmlspecialchars($_POST['apellido'] ?? '') ?>" required>
                    </div>
                </div>
                <div class="form_grupo">
                    <label for="nom_usu">Nombre de usuario</label>
                    <input type="text" id="nom_usu" name="nom_usu" placeholder="Ej: ProGamer99" class="input_field"
                           value="<?= htmlspecialchars($_POST['nom_usu'] ?? '') ?>" required>
                </div>
                <div class="form_grupo">
                    <label for="fecha_nac">Fecha de nacimiento</label>
                    <input type="date" id="fecha_nac" name="fecha_nac" class="input_field"
                           value="<?= htmlspecialchars($_POST['fecha_nac'] ?? '') ?>">
                </div>
                <div class="form_grupo">
                    <label for="localidad">Localidad</label>
                    <input type="text" id="localidad" name="localidad" placeholder="Tu ciudad" class="input_field"
                           value="<?= htmlspecialchars($_POST['localidad'] ?? '') ?>">
                </div>
                <div class="form_grupo">
                    <label for="email">Email</label>
                    <input type="email" id="email" name="email" placeholder="tucorreo@email.com" class="input_field"
                           value="<?= htmlspecialchars($_POST['email'] ?? '') ?>" required>
                </div>
                <div class="form_grupo">
                    <label for="password">Contraseña</label>
                    <input type="password" id="password" name="password" placeholder="Mínimo 8 caracteres" class="input_field" required>
                </div>
                <button type="submit" class="btn_primary" id="btn_registrar">Crear cuenta</button>
                <p id="registro_login">¿Ya tienes cuenta? <a href="../inicio_sesion/inicio_sesion.php">Inicia sesión</a></p>
            </form>
        </div>
    </section>

    <footer id="footer">
        <div id="footer_logo">Rule<span>The</span>Game</div>
        <p>© 2026 RuleTheGame. Todos los derechos reservados.</p>
        <div id="footer_links">
            <a href="../contacto/contacto.php">Contacto</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Privacidad</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Términos</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Soporte</a>
        </div>
    </footer>
</body>
</html>
```
```
if (!$nombre || !$apellido || !$nom_usu || !$email || !$password) {
    $error = 'Por favor, rellena todos los campos obligatorios.';
} elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    $error = 'El email no tiene un formato válido.';
} elseif (strlen($password) < 8) {
    $error = 'La contraseña debe tener al menos 8 caracteres.';
```
Validación en capas: primero comprueba que los campos no estén vacíos, luego que el email tenga formato válido con _filter-var()_ y luego que la contraseña tenga 8 caracteres.
```
$stmt = $conn->prepare("SELECT id FROM usuarios WHERE email = ? OR nom_usu = ?");
$stmt->store_result();
if ($stmt->num_rows > 0) {
    $error = 'El email o nombre de usuario ya están registrados.';
```
Antes de insertar, comprueba si ya existe alguien con ese email o nombre de usuario. _num-rows_ devuelve cúantas filas encontró la consulta.

```
$hash = password_hash($password, PASSWORD_DEFAULT);
```
Aquí se hashea la contraseña antes de guardarla. _PASSWORD-DEFAULT_ usa el algoritmo más seguro disponible en esa versión de PHP (actualmente bcrypt). Nunca se guarda la contraseña original.

```
$ins = $conn->prepare(
    "INSERT INTO usuarios (nombre, apellido, nom_usu, fecha_nac, localidad, email, password)
     VALUES (?, ?, ?, ?, ?, ?, ?)"
);
$ins->bind_param("sssssss", $nombre, $apellido, $nom_usu, $fecha_nac, $localidad, $email, $hash);
```
Inserta el nuevo usuario con otra consulta preparada. Los siete ? corresponden a los siete s del bind_param (todos son strings).

#### registro.css

```
/* ===== REGISTRO SECTION ===== */
#registro_section {
    min-height: calc(100vh - 73px - 97px);
    background: var(--gris_claro);
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 60px 20px;
}

#registro_card {
    background: var(--blanco);
    border-radius: 20px;
    box-shadow: var(--sombra_hover);
    padding: 52px 48px;
    width: 100%;
    max-width: 580px;
}

#registro_header {
    text-align: center;
    margin-bottom: 40px;
}

#registro_titulo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 52px;
    letter-spacing: 2px;
    color: var(--negro);
    line-height: 1;
    margin-bottom: 8px;
}

#registro_sub {
    color: var(--gris_texto);
    font-size: 15px;
}

/* ===== FORM ===== */
#registro_form {
    display: flex;
    flex-direction: column;
    gap: 20px;
}

.form_fila {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 16px;
}

.form_grupo {
    display: flex;
    flex-direction: column;
    gap: 6px;
}

.form_grupo label {
    font-size: 13px;
    font-weight: 700;
    color: var(--negro);
    letter-spacing: 0.5px;
    text-transform: uppercase;
}

.input_field {
    width: 100%;
    padding: 13px 16px;
    border: 2px solid var(--gris_medio);
    border-radius: var(--radio);
    font-family: 'Nunito', sans-serif;
    font-size: 15px;
    color: var(--negro);
    outline: none;
    transition: border-color 0.2s;
    background: var(--blanco);
}

.input_field:focus {
    border-color: var(--acento);
}

/* Estilo del selector de fecha */
input[type="date"].input_field {
    cursor: pointer;
}

input[type="date"].input_field::-webkit-calendar-picker-indicator {
    cursor: pointer;
    opacity: 0.6;
    transition: opacity 0.2s;
}

input[type="date"].input_field::-webkit-calendar-picker-indicator:hover {
    opacity: 1;
}

#btn_registrar {
    margin-top: 8px;
    width: 100%;
    padding: 16px;
    font-size: 17px;
}

#registro_login {
    text-align: center;
    font-size: 14px;
    color: var(--gris_texto);
}

#registro_login a {
    color: var(--acento);
    font-weight: 700;
    text-decoration: none;
}

#registro_login a:hover {
    text-decoration: underline;
}
```
### perfil
Dentro de main.php y tras iniciar sesión aparece el botón _mi perfil_ donde aparecen los datos del usuario y donde este puede cambiar dicha información. 
#### perfil.php
```
<?php
session_start();
require_once '../db.php';

if (!isset($_SESSION['usuario_id'])) {
    header('Location: ../inicio_sesion/inicio_sesion.php');
    exit;
}

$id      = $_SESSION['usuario_id'];
$error   = '';
$success = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $nombre    = trim($_POST['nombre']    ?? '');
    $apellido  = trim($_POST['apellido']  ?? '');
    $nom_usu   = trim($_POST['nom_usu']   ?? '');
    $fecha_nac = trim($_POST['fecha_nac'] ?? '');
    $localidad = trim($_POST['localidad'] ?? '');
    $email     = trim($_POST['email']     ?? '');
    $password  = $_POST['password']       ?? '';
    $password2 = $_POST['password2']      ?? '';

    if (!$nombre || !$apellido || !$nom_usu || !$email) {
        $error = 'Los campos nombre, apellido, usuario y email son obligatorios.';
    } elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $error = 'El email no tiene un formato válido.';
    } elseif ($password && $password !== $password2) {
        $error = 'Las contraseñas no coinciden.';
    } elseif ($password && strlen($password) < 8) {
        $error = 'La contraseña debe tener al menos 8 caracteres.';
    } else {
        $check = $conn->prepare("SELECT id FROM usuarios WHERE (email = ? OR nom_usu = ?) AND id != ?");
        $check->bind_param("ssi", $email, $nom_usu, $id);
        $check->execute();
        $check->store_result();

        if ($check->num_rows > 0) {
            $error = 'Ese email o nombre de usuario ya lo usa otra cuenta.';
        } else {
            if ($password) {
                $hash = password_hash($password, PASSWORD_DEFAULT);
                $upd  = $conn->prepare("UPDATE usuarios SET nombre=?, apellido=?, nom_usu=?, fecha_nac=?, localidad=?, email=?, password=? WHERE id=?");
                $upd->bind_param("sssssssi", $nombre, $apellido, $nom_usu, $fecha_nac, $localidad, $email, $hash, $id);
            } else {
                $upd = $conn->prepare("UPDATE usuarios SET nombre=?, apellido=?, nom_usu=?, fecha_nac=?, localidad=?, email=? WHERE id=?");
                $upd->bind_param("ssssssi", $nombre, $apellido, $nom_usu, $fecha_nac, $localidad, $email, $id);
            }
            if ($upd->execute()) {
                $_SESSION['usuario_nom'] = $nombre;
                $_SESSION['usuario_usr'] = $nom_usu;
                $success = '¡Datos actualizados correctamente!';
            } else {
                $error = 'Error al guardar los cambios. Inténtalo de nuevo.';
            }
            $upd->close();
        }
        $check->close();
    }
}

$stmt = $conn->prepare("SELECT nombre, apellido, nom_usu, fecha_nac, localidad, email, created_at FROM usuarios WHERE id = ?");
$stmt->bind_param("i", $id);
$stmt->execute();
$u = $stmt->get_result()->fetch_assoc();
$stmt->close();

$iniciales = strtoupper(substr($u['nombre'], 0, 1) . substr($u['apellido'], 0, 1));
?>
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mi Perfil - Rule The Game</title>
    <link rel="stylesheet" href="../rulethegame.css">
    <link rel="stylesheet" href="perfil.css">
    <link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Nunito:wght@400;600;700&display=swap" rel="stylesheet">
</head>
<body>
    <header id="navbar">
        <a href="../main.php"><div id="logo">Rule<span>The</span>Game</div></a>
        <nav>
            <a href="../main.php">Inicio</a>
            <a href="../main.php#servicios">Servicios</a>
            <a href="../main.php#coaches">Coaches</a>
        </nav>
        <div id="navbar_botones">
            <button id="btn_perfil" onclick="window.location.href='perfil.php'">Mi perfil</button>
            <button id="btn_logout" onclick="window.location.href='../logout.php'">Cerrar sesión</button>
        </div>
    </header>

    <section id="perfil_section">
        <div id="perfil_header">
            <div id="perfil_avatar"><?= htmlspecialchars($iniciales) ?></div>
            <div id="perfil_info">
                <h1 id="perfil_nombre"><?= htmlspecialchars($u['nombre'] . ' ' . $u['apellido']) ?></h1>
                <p id="perfil_usuario">@<?= htmlspecialchars($u['nom_usu']) ?></p>
                <p id="perfil_fecha_reg">Miembro desde <?= date('d/m/Y', strtotime($u['created_at'])) ?></p>
            </div>
        </div>

        <?php if ($error): ?>
            <div class="perfil_msg perfil_msg_error">⚠️ <?= htmlspecialchars($error) ?></div>
        <?php endif; ?>
        <?php if ($success): ?>
            <div class="perfil_msg perfil_msg_ok">✅ <?= htmlspecialchars($success) ?></div>
        <?php endif; ?>

        <div id="perfil_card">
            <h2 id="perfil_card_titulo">Editar información</h2>
            <form method="POST" action="perfil.php">
                <div class="form_fila">
                    <div class="form_grupo">
                        <label for="nombre">Nombre</label>
                        <input type="text" id="nombre" name="nombre" class="input_field" value="<?= htmlspecialchars($u['nombre']) ?>" required>
                    </div>
                    <div class="form_grupo">
                        <label for="apellido">Apellido</label>
                        <input type="text" id="apellido" name="apellido" class="input_field" value="<?= htmlspecialchars($u['apellido']) ?>" required>
                    </div>
                </div>
                <div class="form_grupo">
                    <label for="nom_usu">Nombre de usuario</label>
                    <input type="text" id="nom_usu" name="nom_usu" class="input_field" value="<?= htmlspecialchars($u['nom_usu']) ?>" required>
                </div>
                <div class="form_grupo">
                    <label for="fecha_nac">Fecha de nacimiento</label>
                    <input type="date" id="fecha_nac" name="fecha_nac" class="input_field" value="<?= htmlspecialchars($u['fecha_nac'] ?? '') ?>">
                </div>
                <div class="form_grupo">
                    <label for="localidad">Localidad</label>
                    <input type="text" id="localidad" name="localidad" class="input_field" value="<?= htmlspecialchars($u['localidad'] ?? '') ?>">
                </div>
                <div class="form_grupo">
                    <label for="email">Email</label>
                    <input type="email" id="email" name="email" class="input_field" value="<?= htmlspecialchars($u['email']) ?>" required>
                </div>
                <div id="perfil_pass_separador">
                    <span></span><p>Cambiar contraseña <em>(opcional)</em></p><span></span>
                </div>
                <div class="form_fila">
                    <div class="form_grupo">
                        <label for="password">Nueva contraseña</label>
                        <input type="password" id="password" name="password" class="input_field" placeholder="Mínimo 8 caracteres">
                    </div>
                    <div class="form_grupo">
                        <label for="password2">Repetir contraseña</label>
                        <input type="password" id="password2" name="password2" class="input_field" placeholder="Repite la contraseña">
                    </div>
                </div>
                <button type="submit" class="btn_primary" id="btn_guardar">Guardar cambios</button>
            </form>
        </div>
    </section>

    <footer id="footer">
        <div id="footer_logo">Rule<span>The</span>Game</div>
        <p>© 2026 RuleTheGame. Todos los derechos reservados.</p>
        <div id="footer_links">
            <a href="../contacto/contacto.php">Contacto</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Privacidad</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Términos</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Soporte</a>
        </div>
    </footer>
</body>
</html>

```
```
if (!isset($_SESSION['usuario_id'])) {
    header('Location: ../inicio_sesion/inicio_sesion.php');
    exit;
}
```
Protege la página: si alguien intenta entrar a _perfil.php_ sin estar logado, lo redirige al login. Esto es lo que se llama una "guard".
La lógica de actualización tiene dos ramas porque la contraseña es opcional:
```
if ($password) {
    $hash = password_hash($password, PASSWORD_DEFAULT);
    $upd  = $conn->prepare("UPDATE usuarios SET nombre=?, ..., password=? WHERE id=?");
    $upd->bind_param("sssssssi", ...);  // 7 strings + 1 integer
} else {
    $upd = $conn->prepare("UPDATE usuarios SET nombre=?, ..., email=? WHERE id=?");
    $upd->bind_param("ssssssi", ...);   // 6 strings + 1 integer
}
```
Si el usuario dejó la contraseña en blanco, hace el UPDATE sin tocarla. Si la rellenó, la hashea y la incluye en el UPDATE. La i al final del bind_param indica que el id es un entero (integer).
```
$iniciales = strtoupper(substr($u['nombre'], 0, 1) . substr($u['apellido'], 0, 1));
```
Genera las iniciales del usuario (por ejemplo "JG" para "Juan García") para mostrarlas en el avatar circular. substr() recorta una cadena, strtoupper() la pone en mayúsculas.
```
$iniciales = strtoupper(substr($u['nombre'], 0, 1) . substr($u['apellido'], 0, 1));
```
```
<p id="perfil_fecha_reg">Miembro desde <?= date('d/m/Y', strtotime($u['created_at'])) ?></p>
```
_strtotime()_ convierte la fecha de MySQL (formato 2024-01-15 10:30:00) en un timestamp Unix, y date() la formatea al estilo español 15/01/2024.

#### perfil.css
```
/* ===== PERFIL SECTION ===== */
#perfil_section {
    min-height: calc(100vh - 73px - 97px);
    background: var(--gris_claro);
    padding: 60px 20px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 32px;
}

/* ── Cabecera avatar ── */
#perfil_header {
    display: flex;
    align-items: center;
    gap: 28px;
    background: var(--blanco);
    border-radius: 20px;
    box-shadow: var(--sombra);
    padding: 36px 48px;
    width: 100%;
    max-width: 680px;
}

#perfil_avatar {
    width: 90px;
    height: 90px;
    border-radius: 50%;
    background: var(--acento);
    color: var(--blanco);
    font-family: 'Bebas Neue', sans-serif;
    font-size: 36px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    box-shadow: 0 4px 16px rgba(255,60,60,0.3);
}

#perfil_nombre {
    font-size: 24px;
    font-weight: 700;
    color: var(--negro);
    margin-bottom: 4px;
}

#perfil_usuario {
    color: var(--acento);
    font-weight: 600;
    font-size: 15px;
    margin-bottom: 4px;
}

#perfil_fecha_reg {
    color: var(--gris_texto);
    font-size: 13px;
}

/* ── Mensajes ── */
.perfil_msg {
    width: 100%;
    max-width: 680px;
    padding: 14px 18px;
    border-radius: 10px;
    font-size: 14px;
    font-weight: 600;
}

.perfil_msg_error { background: #fee; color: #c00; }
.perfil_msg_ok    { background: #efe; color: #060; }

/* ── Card edición ── */
#perfil_card {
    background: var(--blanco);
    border-radius: 20px;
    box-shadow: var(--sombra);
    padding: 48px;
    width: 100%;
    max-width: 680px;
}

#perfil_card_titulo {
    font-size: 20px;
    font-weight: 700;
    margin-bottom: 28px;
    color: var(--negro);
}

/* ── Reutilizar clases del registro ── */
.form_fila {
    display: flex;
    gap: 20px;
}

.form_grupo {
    display: flex;
    flex-direction: column;
    gap: 6px;
    margin-bottom: 20px;
    flex: 1;
}

.form_grupo label {
    font-size: 14px;
    font-weight: 600;
    color: var(--negro);
}

.input_field {
    padding: 12px 16px;
    border: 1.5px solid var(--gris_medio);
    border-radius: var(--radio);
    font-size: 15px;
    font-family: 'Nunito', sans-serif;
    transition: border-color 0.2s;
    outline: none;
    width: 100%;
}

.input_field:focus {
    border-color: var(--acento);
}

/* ── Separador contraseña ── */
#perfil_pass_separador {
    display: flex;
    align-items: center;
    gap: 12px;
    margin: 8px 0 20px;
    color: var(--gris_texto);
    font-size: 13px;
}

#perfil_pass_separador span {
    flex: 1;
    height: 1px;
    background: var(--gris_medio);
}

#perfil_pass_separador em {
    color: var(--gris_texto);
    font-style: normal;
}

/* ── Botón guardar ── */
#btn_guardar {
    width: 100%;
    padding: 14px;
    background: var(--acento);
    color: var(--blanco);
    border: none;
    border-radius: var(--radio);
    font-size: 16px;
    font-weight: 700;
    font-family: 'Nunito', sans-serif;
    cursor: pointer;
    margin-top: 8px;
    transition: background 0.2s;
}

#btn_guardar:hover {
    background: var(--acento_hover);
}

/* ── Navbar logado ── */
#btn_perfil {
    padding: 9px 20px;
    border-radius: 8px;
    border: 1.5px solid var(--acento);
    background: transparent;
    color: var(--acento);
    font-weight: 700;
    font-size: 14px;
    font-family: 'Nunito', sans-serif;
    cursor: pointer;
    transition: all 0.2s;
}

#btn_perfil:hover {
    background: var(--acento);
    color: var(--blanco);
}

#btn_logout {
    padding: 9px 20px;
    border-radius: 8px;
    border: none;
    background: var(--negro);
    color: var(--blanco);
    font-weight: 700;
    font-size: 14px;
    font-family: 'Nunito', sans-serif;
    cursor: pointer;
    transition: opacity 0.2s;
}

#btn_logout:hover { opacity: 0.75; }
```
### perfil_coach
Los usuarios pueden acceder al apartado de coachs donde estos pueden ver el perfil de cada coach con información relevante para su contratación. Este perfil es el mismo para todos los coach excepto su información ya que se hace una consulta a la Base de datos y se remplaza dicha información por la de cada coach.
#### perfil.coach.php
```
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Perfil Coach - Rule The Game</title>
    <link rel="stylesheet" href="../rulethegame.css">
    <link rel="stylesheet" href="perfil_coach.css">
    <link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Nunito:wght@400;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0/css/all.min.css">
</head>
<body>

    <!-- NAVBAR -->
    <header id="navbar">
        <a href="../main.php"><div id="logo">Rule<span>The</span>Game</div></a>
        <nav>
            <a href="../main.php">Inicio</a>
            <a href="../main.php#servicios">Servicios</a>
            <a href="../main.php#coaches">Coaches</a>
        </nav>
        <div id="navbar_botones">
            <button id="btn_registro" onclick="window.location.href='../pag_registro/registro.php'">Registrarse</button>
            <button id="btn_login" onclick="window.location.href='../inicio_sesion/inicio_sesion.php'">Iniciar sesión</button>
        </div>
    </header>

    <?php
    $coach = [
        "nombre"        => "JaveX",
        "juego"         => "Valorant",
        "rango"         => "Radiante Top 50 EU",
        "descripcion"   => "Especialista en duelistas y mentalidad competitiva. Llevo más de 3 años entrenando jugadores de todos los niveles, desde Plata hasta Immortal. Mi metodología se basa en el análisis de replays, control mental y mecánicas avanzadas.",
        "rating"        => 5,
        "sesiones"      => 128,
        "alumnos"       => 74,
        "precio"        => 25,
        "juegos"        => ["Valorant"],
        "especialidades"=> ["Duelistas", "Aim training", "Mentalidad competitiva", "Análisis de replays"],
        "idiomas"       => ["Español", "Inglés"],
        "avatar"        => "JX"
    ];
    ?>

    <!-- HERO PERFIL -->
    <section id="perfil_hero">
        <div id="perfil_hero_inner">

            <div id="perfil_avatar">
                <?= $coach['avatar'] ?>
            </div>

            <div id="perfil_info">
                <span id="perfil_juego_tag"><?= $coach['juego'] ?></span>
                <h1 id="perfil_nombre"><?= $coach['nombre'] ?></h1>
                <p id="perfil_rango"><i class="fa-solid fa-trophy"></i> <?= $coach['rango'] ?></p>

                <div id="perfil_rating">
                    <?php for ($i = 1; $i <= 5; $i++): ?>
                        <i class="fa-star <?= $i <= $coach['rating'] ? 'fa-solid' : 'fa-regular' ?>"></i>
                    <?php endfor; ?>
                    <span><?= $coach['rating'] ?>.0 · <?= $coach['sesiones'] ?> sesiones</span>
                </div>

                <div id="perfil_stats">
                    <div class="perfil_stat">
                        <span class="perfil_stat_num"><?= $coach['sesiones'] ?></span>
                        <span class="perfil_stat_label">Sesiones</span>
                    </div>
                    <div class="perfil_stat">
                        <span class="perfil_stat_num"><?= $coach['alumnos'] ?></span>
                        <span class="perfil_stat_label">Alumnos</span>
                    </div>
                    <div class="perfil_stat">
                        <span class="perfil_stat_num"><?= $coach['precio'] ?>€</span>
                        <span class="perfil_stat_label">/ hora</span>
                    </div>
                </div>
            </div>

            <div id="perfil_cta">
                <button class="btn_primary" id="btn_reservar">Reservar sesión</button>
                <button class="btn_secondary" id="btn_mensaje">Enviar mensaje</button>
            </div>

        </div>
    </section>

    <!-- CONTENIDO -->
    <section id="perfil_main">

        <!-- COLUMNA IZQUIERDA -->
        <div id="perfil_columna_izq">

            <div class="perfil_card">
                <h2 class="perfil_card_titulo">Sobre mí</h2>
                <p class="perfil_card_texto"><?= $coach['descripcion'] ?></p>
            </div>

            <div class="perfil_card">
                <h2 class="perfil_card_titulo">Especialidades</h2>
                <div id="perfil_tags">
                    <?php foreach ($coach['especialidades'] as $esp): ?>
                        <span class="perfil_tag"><?= $esp ?></span>
                    <?php endforeach; ?>
                </div>
            </div>

            <div class="perfil_card">
                <h2 class="perfil_card_titulo">Reseñas</h2>
                <!-- PRÓXIMAMENTE: cargar reseñas desde BD -->
                <div class="resena">
                    <div class="resena_header">
                        <div class="resena_avatar">MG</div>
                        <div>
                            <span class="resena_nombre">MidGamer99</span>
                            <div class="resena_estrellas">
                                <i class="fa-solid fa-star"></i>
                                <i class="fa-solid fa-star"></i>
                                <i class="fa-solid fa-star"></i>
                                <i class="fa-solid fa-star"></i>
                                <i class="fa-solid fa-star"></i>
                            </div>
                        </div>
                    </div>
                    <p class="resena_texto">Increíble coach, en 3 semanas subí de Oro a Platino. Muy claro explicando y siempre puntual.</p>
                </div>
                <div class="resena">
                    <div class="resena_header">
                        <div class="resena_avatar">PX</div>
                        <div>
                            <span class="resena_nombre">ProXenon</span>
                            <div class="resena_estrellas">
                                <i class="fa-solid fa-star"></i>
                                <i class="fa-solid fa-star"></i>
                                <i class="fa-solid fa-star"></i>
                                <i class="fa-solid fa-star"></i>
                                <i class="fa-regular fa-star"></i>
                            </div>
                        </div>
                    </div>
                    <p class="resena_texto">Muy buen análisis de replays, me ayudó a identificar mis errores más comunes. Lo recomiendo.</p>
                </div>
            </div>

        </div>

        <!-- COLUMNA DERECHA -->
        <div id="perfil_columna_der">

            <div class="perfil_card">
                <h2 class="perfil_card_titulo">Juegos</h2>
                <div id="perfil_tags">
                    <?php foreach ($coach['juegos'] as $juego): ?>
                        <span class="perfil_tag destacado"><?= $juego ?></span>
                    <?php endforeach; ?>
                </div>
            </div>

            <div class="perfil_card">
                <h2 class="perfil_card_titulo">Idiomas</h2>
                <div id="perfil_tags">
                    <?php foreach ($coach['idiomas'] as $idioma): ?>
                        <span class="perfil_tag"><?= $idioma ?></span>
                    <?php endforeach; ?>
                </div>
            </div>

            <div class="perfil_card" id="card_precio">
                <h2 class="perfil_card_titulo">Precio</h2>
                <div id="precio_display">
                    <span id="precio_num"><?= $coach['precio'] ?>€</span>
                    <span id="precio_label">por hora</span>
                </div>
                <button class="btn_primary" id="btn_reservar2">Reservar ahora</button>
            </div>

        </div>

    </section>

    <!-- FOOTER -->
    <footer id="footer">
        <div id="footer_logo">Rule<span>The</span>Game</div>
        <p>© 2026 RuleTheGame. Todos los derechos reservados.</p>
        <div id="footer_links">
            <a href="../contacto/contacto.php">Contacto</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Privacidad</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Términos</a>
            <a href="../lorem_ipsu/lorem_ipsu.php">Soporte</a>
        </div>
    </footer>

</body>
</html>
```
#### perfil_coach.css
```
/* ===== HERO PERFIL ===== */
#perfil_hero {
    background: var(--negro);
    padding: 60px;
    color: var(--blanco);
}

#perfil_hero_inner {
    display: flex;
    align-items: center;
    gap: 40px;
    flex-wrap: wrap;
}

#perfil_avatar {
    width: 110px;
    height: 110px;
    background: var(--acento);
    color: var(--blanco);
    font-family: 'Bebas Neue', sans-serif;
    font-size: 40px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
}

#perfil_info {
    flex: 1;
}

#perfil_juego_tag {
    display: inline-block;
    background: rgba(255,60,60,0.2);
    color: var(--acento);
    font-size: 12px;
    font-weight: 700;
    padding: 4px 12px;
    border-radius: 20px;
    margin-bottom: 10px;
    letter-spacing: 1px;
    text-transform: uppercase;
}

#perfil_nombre {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 56px;
    letter-spacing: 2px;
    line-height: 1;
    margin-bottom: 8px;
}

#perfil_rango {
    font-size: 15px;
    color: #aaa;
    margin-bottom: 12px;
}

#perfil_rango i {
    color: #f5a623;
    margin-right: 6px;
}

#perfil_rating {
    display: flex;
    align-items: center;
    gap: 6px;
    margin-bottom: 24px;
}

#perfil_rating i {
    color: #f5a623;
    font-size: 16px;
}

#perfil_rating span {
    font-size: 14px;
    color: #aaa;
    margin-left: 4px;
}

#perfil_stats {
    display: flex;
    gap: 32px;
}

.perfil_stat {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.perfil_stat_num {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 36px;
    color: var(--blanco);
    line-height: 1;
}

.perfil_stat_label {
    font-size: 12px;
    color: #888;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 1px;
}

#perfil_cta {
    display: flex;
    flex-direction: column;
    gap: 12px;
    min-width: 200px;
}

#perfil_cta .btn_secondary {
    border-color: rgba(255,255,255,0.3);
    color: var(--blanco);
}

#perfil_cta .btn_secondary:hover {
    background: rgba(255,255,255,0.1);
    color: var(--blanco);
}

/* ===== CONTENIDO PRINCIPAL ===== */
#perfil_main {
    display: grid;
    grid-template-columns: 1.6fr 1fr;
    gap: 32px;
    padding: 60px;
    background: var(--gris_claro);
    align-items: start;
}

/* ===== COLUMNAS ===== */
#perfil_columna_izq,
#perfil_columna_der {
    display: flex;
    flex-direction: column;
    gap: 24px;
}

/* ===== CARDS ===== */
.perfil_card {
    background: var(--blanco);
    border-radius: var(--radio);
    padding: 32px;
    box-shadow: var(--sombra);
}

.perfil_card_titulo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 28px;
    letter-spacing: 1px;
    color: var(--negro);
    margin-bottom: 16px;
}

.perfil_card_texto {
    font-size: 15px;
    color: var(--gris_texto);
    line-height: 1.8;
}

/* ===== TAGS ===== */
#perfil_tags {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
}

.perfil_tag {
    background: var(--gris_claro);
    color: var(--negro);
    font-size: 13px;
    font-weight: 700;
    padding: 6px 14px;
    border-radius: 20px;
}

.perfil_tag.destacado {
    background: var(--acento_suave);
    color: var(--acento);
}

/* ===== RESEÑAS ===== */
.resena {
    padding: 20px 0;
    border-bottom: 1px solid var(--gris_medio);
}

.resena:last-child {
    border-bottom: none;
    padding-bottom: 0;
}

.resena_header {
    display: flex;
    align-items: center;
    gap: 14px;
    margin-bottom: 10px;
}

.resena_avatar {
    width: 44px;
    height: 44px;
    background: var(--negro);
    color: var(--blanco);
    font-family: 'Bebas Neue', sans-serif;
    font-size: 16px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
}

.resena_nombre {
    font-weight: 700;
    font-size: 15px;
    display: block;
    margin-bottom: 4px;
}

.resena_estrellas i {
    color: #f5a623;
    font-size: 13px;
}

.resena_texto {
    font-size: 14px;
    color: var(--gris_texto);
    line-height: 1.7;
}

/* ===== PRECIO ===== */
#card_precio {
    position: sticky;
    top: 100px;
    text-align: center;
}

#precio_display {
    display: flex;
    align-items: baseline;
    justify-content: center;
    gap: 8px;
    margin: 16px 0 24px;
}

#precio_num {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 64px;
    color: var(--acento);
    line-height: 1;
}

#precio_label {
    font-size: 16px;
    color: var(--gris_texto);
    font-weight: 600;
}

#btn_reservar2 {
    width: 100%;
    padding: 16px;
    font-size: 16px;
}
```