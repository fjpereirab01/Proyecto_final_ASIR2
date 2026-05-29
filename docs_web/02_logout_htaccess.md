# `logout.php` y `.htaccess`

---

## `logout.php` — Cierre de sesión

Archivo que destruye la sesión del usuario y lo redirige a la página principal.

```php
<?php
// Inicia la sesión para poder acceder a los datos de sesión actuales
session_start();

// Destruye todos los datos almacenados en la sesión del usuario
// (usuario_id, usuario_rol, coach_id, etc.)
session_destroy();

// Redirige al usuario a la página principal tras cerrar sesión
header('Location: /main.php');
exit; // Detiene la ejecución para que no se procese nada más
?>
```

### Flujo
1. El usuario hace clic en "Cerrar sesión"
2. PHP elimina todos los datos de sesión del servidor
3. El usuario es redirigido a `main.php` sin sesión activa

---

## `.htaccess` — Página de error personalizada

Archivo de configuración de Apache que redirige los errores 404 a una página personalizada.

```apache
# Cuando Apache no encuentra una página (error 404),
# en lugar de mostrar la página genérica del servidor,
# redirige al archivo 404.php del proyecto
ErrorDocument 404 /404.php
```

### ¿Por qué es necesario?
Sin este archivo, si un usuario accede a una URL inexistente (por ejemplo `/pagina-que-no-existe`), Apache mostraría su página de error genérica con el estilo del servidor. Con `.htaccess` se muestra la página `404.php` del proyecto, con el diseño de RuleTheGame.
