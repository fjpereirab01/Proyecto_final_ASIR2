# `rulethegame.css` — Estilos globales y diseño responsive

Hoja de estilos principal del proyecto. Define las variables de color, la navbar, el hero, el carrusel, las secciones de servicios y coaches, el footer, y el bloque de media queries para responsive.

---

## Variables CSS

```css
:root {
    --negro: #0d0d0d;           /* Fondo oscuro principal */
    --blanco: #ffffff;           /* Texto y fondos claros */
    --gris_claro: #f5f5f7;      /* Fondo de secciones alternadas */
    --gris_medio: #e8e8ed;      /* Bordes y separadores */
    --gris_texto: #6e6e73;      /* Color de texto secundario */
    --acento: #ff3c3c;          /* Color principal de marca (rojo) */
    --acento_hover: #e02020;    /* Acento más oscuro para hover */
    --acento_suave: #fff0f0;    /* Fondo suave para badges de juego */
    --sombra: 0 4px 24px rgba(0,0,0,0.08);       /* Sombra ligera */
    --sombra_hover: 0 8px 40px rgba(0,0,0,0.15); /* Sombra más pronunciada en hover */
    --radio: 12px;               /* Border-radius estándar de las tarjetas */
}
```

---

## Navbar

```css
#navbar {
    display: flex;                  /* Flexbox para alinear logo, nav y botones en fila */
    align-items: center;            /* Alineación vertical centrada */
    justify-content: space-between; /* Logo izquierda, nav centro, botones derecha */
    padding: 18px 60px;
    background: var(--blanco);
    border-bottom: 1px solid var(--gris_medio);
    position: sticky;               /* La navbar se queda fija al hacer scroll */
    top: 0;
    z-index: 100;                   /* Por encima de todo el contenido */
}
```

---

## Hero

```css
#inicio {
    min-height: 88vh;           /* Ocupa casi toda la pantalla */
    display: flex;
    flex-direction: row;        /* Contenido a la izquierda, carrusel a la derecha */
    justify-content: space-between;
    align-items: center;
    gap: 60px;
    padding: 80px 60px;
    background: var(--gris_claro);
}

#hero_titulo {
    font-family: 'Bebas Neue', sans-serif;
    font-size: 100px;           /* Título muy grande para impacto visual */
    line-height: 0.95;          /* Interlineado muy ajustado para bloques de texto */
    letter-spacing: 2px;
}
```

---

## Carrusel

```css
#carrusel_track {
    display: flex;              /* Las slides en fila */
    width: 100%;
    height: 100%;
    transition: transform 0.5s ease; /* Animación suave al cambiar de slide */
}

.slide {
    min-width: 100%;            /* Cada slide ocupa el 100% del carrusel */
    height: 100%;
    object-fit: cover;          /* La imagen cubre el espacio sin deformarse */
}
```
El JS mueve el `carrusel_track` con `translateX(-100%)`, `-200%`, etc. para mostrar cada slide.

---

## Grid de servicios y coaches

```css
#servicios_grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr); /* 4 columnas iguales en escritorio */
    gap: 24px;
}

#coaches_grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr); /* 3 columnas iguales en escritorio */
    gap: 28px;
}
```

---

## Responsive — Media queries

Las media queries se añaden al final del archivo y sobrescriben los estilos de escritorio para pantallas más pequeñas.

### Tablet (≤ 900px)

```css
@media (max-width: 900px) {
    #navbar {
        padding: 16px 24px; /* Menos padding lateral */
    }

    #inicio {
        flex-direction: column; /* El hero pasa a columna: contenido arriba, carrusel abajo */
        padding: 60px 24px;
        gap: 40px;
        min-height: auto;       /* Ya no necesita ocupar toda la pantalla */
    }

    #hero_titulo {
        font-size: 72px;        /* Título más pequeño */
    }

    #carrusel {
        max-width: 100%;        /* El carrusel ocupa todo el ancho disponible */
        width: 100%;
    }

    #servicios_grid {
        grid-template-columns: repeat(2, 1fr); /* 4 columnas → 2 columnas */
    }

    #coaches_grid {
        grid-template-columns: repeat(2, 1fr); /* 3 columnas → 2 columnas */
    }

    #servicios, #coaches {
        padding: 60px 24px; /* Menos padding en secciones */
    }

    #footer {
        flex-direction: column; /* Footer apilado en columna */
        gap: 16px;
        text-align: center;
    }

    #footer_links {
        flex-wrap: wrap;        /* Los links del footer se ajustan en varias líneas */
        justify-content: center;
    }
}
```

### Móvil (≤ 600px)

```css
@media (max-width: 600px) {
    #navbar {
        flex-wrap: wrap;        /* La navbar puede ocupar varias líneas */
        padding: 14px 16px;
        gap: 10px;
    }

    nav {
        order: 3;               /* El menú de navegación va debajo del logo y los botones */
        width: 100%;            /* Ocupa todo el ancho */
        justify-content: center;
        padding-top: 8px;
        border-top: 1px solid var(--gris_medio); /* Separador visual */
    }

    #hero_titulo {
        font-size: 56px;        /* Título más pequeño en móvil */
    }

    #hero_botones {
        flex-direction: column; /* Los botones del hero en columna */
        gap: 12px;
    }

    .btn_primary, .btn_secondary {
        width: 100%;            /* Botones a ancho completo en móvil */
        text-align: center;
    }

    #hero_stats {
        flex-direction: column; /* Las estadísticas en columna */
    }

    .stat_card {
        border-right: none;
        border-bottom: 1px solid var(--gris_medio); /* Separador horizontal */
        padding: 20px;
    }

    #servicios_grid {
        grid-template-columns: 1fr; /* 2 columnas → 1 columna */
    }

    #coaches_grid {
        grid-template-columns: 1fr; /* 2 columnas → 1 columna */
    }

    .section_titulo {
        font-size: 38px;        /* Títulos de sección más pequeños */
    }
}
```

---

## Resumen de breakpoints

| Breakpoint | Cambios principales |
|---|---|
| > 900px (escritorio) | Hero en fila, grids de 4/3 columnas, navbar horizontal |
| ≤ 900px (tablet) | Hero en columna, grids de 2 columnas, footer apilado |
| ≤ 600px (móvil) | Navbar apilada, título 56px, grids de 1 columna, botones ancho completo |
