# `rulethegame.js` — JavaScript global

Archivo JavaScript cargado en `main.php`. Controla tres comportamientos: scroll suave, sombra de la navbar y el carrusel de imágenes.

---

## Código completo comentado

```javascript
// ===== SCROLL SUAVE =====
// Selecciona todos los enlaces <a> cuyo href empiece por "#" (anclas internas)
document.querySelectorAll('a[href^="#"]').forEach(enlace => {
    enlace.addEventListener('click', function(e) {
        e.preventDefault(); // Cancela el comportamiento de salto instantáneo

        // Busca el elemento destino con el id del href (ej: "#servicios" → busca id="servicios")
        const destino = document.querySelector(this.getAttribute('href'));

        // Si el elemento existe, hace scroll animado hasta él
        if (destino) destino.scrollIntoView({ behavior: 'smooth' });
    });
});

// ===== NAVBAR SOMBRA AL SCROLL =====
// Escucha el evento de scroll de la ventana
window.addEventListener('scroll', () => {
    const navbar = document.getElementById('navbar');

    // Si se ha bajado más de 10px, añade sombra a la navbar
    // Si está en el tope (0-10px), quita la sombra
    navbar.style.boxShadow = window.scrollY > 10
        ? '0 2px 20px rgba(0,0,0,0.1)'
        : 'none';
});

// ===== CARRUSEL =====
// Referencia al contenedor que se desplaza horizontalmente
const track = document.getElementById('carrusel_track');

// Todos los puntos indicadores (dots)
const dots = document.querySelectorAll('.dot');

// Todas las imágenes del carrusel
const slides = document.querySelectorAll('.slide');

// Índice del slide actual (empieza en 0)
let actual = 0;

// Total de slides
const total = slides.length;

function irA(index) {
    // Calcula el índice con módulo para que sea circular
    // Si index = -1, pasa al último; si index = total, vuelve al primero
    actual = (index + total) % total;

    // Mueve el track horizontalmente: 0%, 100%, 200%...
    track.style.transform = `translateX(-${actual * 100}%)`;

    // Quita la clase 'activo' de todos los dots
    dots.forEach(d => d.classList.remove('activo'));

    // Añade 'activo' al dot correspondiente al slide actual
    dots[actual].classList.add('activo');
}

// Botón siguiente: avanza un slide
document.getElementById('btn_next').addEventListener('click', () => irA(actual + 1));

// Botón anterior: retrocede un slide
document.getElementById('btn_prev').addEventListener('click', () => irA(actual - 1));

// Clic en cada dot: va directamente a ese slide
dots.forEach((dot, i) => dot.addEventListener('click', () => irA(i)));

// Avance automático cada 10 segundos
setInterval(() => irA(actual + 1), 10000);
```

---

## Resumen de funciones

| Función/evento | Qué hace |
|---|---|
| `querySelectorAll('a[href^="#"]')` | Scroll suave a secciones de la misma página |
| `scroll` listener | Añade sombra a la navbar al hacer scroll |
| `irA(index)` | Cambia el slide activo del carrusel |
| `setInterval` | Avance automático del carrusel cada 10s |
