# 11. Código de la aplicación web

[← Volver al índice](../README.md)

---

## 11.1 Tecnologías utilizadas

| Tecnología | Versión | Uso |
|---|---|---|
| PHP | 7.4+ | Backend y lógica de negocio |
| MariaDB | 10.5+ | Base de datos relacional |
| HTML5 / CSS3 | — | Estructura y estilos |
| JavaScript | ES6 | Carrusel, toggle contraseña |
| Bebas Neue + Nunito | — | Tipografías (Google Fonts) |

---

## 11.2 Estructura de ficheros

```
/var/www/rulethegame/
├── main.php                  # Página principal (landing)
├── db.php                    # Conexión a MariaDB
├── logout.php                # Cierre de sesión
├── rulethegame.css           # Estilos globales
├── rulethegame.js            # Scripts globales (carrusel)
├── sesiones_extra.css        # Estilos de tarjetas de sesión
├── inicio_sesion/
│   ├── inicio_sesion.php     # Login de usuarios
│   └── inicio_sesion.css
├── pag_registro/
│   ├── registro.php          # Registro de nuevos usuarios
│   ├── registro.css
│   └── registro.js
├── perfil/
│   ├── perfil.php            # Perfil del cliente (edición + sesiones)
│   └── perfil.css
├── perfil_coach/
│   ├── perfil_coach.php      # Perfil del coach (sesiones + comentarios + contratación)
│   └── perfil_coach.css
├── coaches/
│   ├── coaches.php           # Listado y búsqueda de coaches
│   └── coaches.css
├── admin/
│   ├── admin.php             # Panel de administración
│   └── admin.css
├── contacto/
│   ├── contacto.php
│   └── contacto.css
└── imagenes/
    ├── equipo.png
    ├── coaching_directo.png
    └── persona_feliz.png
```

---

## 11.3 Sistema de roles

La aplicación distingue tres roles de usuario almacenados en el campo `rol` de la tabla `usuarios`:

| Rol | Acceso |
|---|---|
| `cliente` | Puede contratar sesiones y ver su perfil |
| `coach` | Puede ver sus sesiones asignadas desde su perfil de coach |
| `admin` | Acceso completo al panel de administración |

Al iniciar sesión, el rol se guarda en la sesión PHP:

```php
$_SESSION['usuario_rol'] = $row['rol'];

// Si es coach, se guarda también su coach_id
if ($row['rol'] === 'coach') {
    $sc = $conn->prepare("SELECT id FROM coaches WHERE usuario_id = ?");
    $sc->bind_param("i", $row['id']);
    $sc->execute();
    $rc = $sc->get_result()->fetch_assoc();
    if ($rc) $_SESSION['coach_id'] = $rc['id'];
}
```

El botón "Mi perfil" del navbar redirige según el rol:

```php
$url = isset($_SESSION['usuario_rol']) && $_SESSION['usuario_rol'] === 'coach'
    ? 'https://192.168.10.1/perfil_coach/perfil_coach.php?id=' . $_SESSION['coach_id']
    : 'https://192.168.10.1/perfil/perfil.php';
```

---

## 11.4 Conexión a la base de datos

Fichero `db.php`, compartido por todos los scripts PHP:

```php
<?php
$host = '10.0.0.100';
$db   = 'proyecto_asir';
$user = 'rtg_app';
$pass = 'contraseña_segura';

$conn = new mysqli($host, $user, $pass, $db);
if ($conn->connect_error) {
    die('Error de conexión: ' . $conn->connect_error);
}
$conn->set_charset('utf8mb4');
```

La conexión siempre usa el usuario `rtg_app` con permisos mínimos, nunca `root`.

---

## 11.5 Páginas principales

### Landing (`main.php`)
Página de inicio con carrusel de imágenes, sección de servicios y coaches destacados. Si el usuario está logado muestra los botones "Mi perfil" y "Cerrar sesión". Si es admin, muestra además el botón "Admin".

### Login (`inicio_sesion/inicio_sesion.php`)
Permite acceder con email o nombre de usuario. Verifica la contraseña con `password_verify()`. Guarda en sesión el id, nombre, usuario y rol.

### Registro (`pag_registro/registro.php`)
Formulario de alta de nuevos usuarios. La contraseña se cifra con `password_hash()` antes de insertar en BD. El rol por defecto es `cliente`.

### Perfil cliente (`perfil/perfil.php`)
Muestra las sesiones contratadas por el cliente con opciones de cancelación. Permite editar los datos personales y cambiar la contraseña.

### Perfil coach (`perfil_coach/perfil_coach.php`)
Página pública del coach con información, valoraciones y comentarios. Si el visitante es un cliente logado, muestra el formulario de contratación de sesión. Si el visitante es el propio coach, muestra sus sesiones asignadas.

### Coaches (`coaches/coaches.php`)
Listado de todos los coaches con filtros por juego y búsqueda por nombre o especialidad. Carga los datos dinámicamente desde MariaDB.

### Panel admin (`admin/admin.php`)
Accesible solo para el usuario con `id=1`. Permite crear, editar y eliminar coaches, eliminar usuarios y cambiar el estado de las sesiones. Muestra estadísticas generales.

---

## 11.6 Contratación de sesiones

El flujo de contratación desde el perfil del coach:

1. El cliente rellena el formulario (fecha, hora, duración, notas opcionales).
2. Se inserta un registro en la tabla `sesiones` con estado `pendiente`.
3. La sesión aparece en el perfil del cliente y en el panel del coach.
4. El cliente puede cancelarla si está en estado `pendiente`.
5. El admin puede cambiar el estado a `completada` o `cancelada` desde el panel.

```php
$ins = $conn->prepare(
    "INSERT INTO sesiones (cliente_id, coach_id, fecha, duracion, estado, notas)
     VALUES (?, ?, ?, ?, 'pendiente', ?)"
);
$ins->bind_param("iiiss", $_SESSION['usuario_id'], $id, $fecha_dt, $dur, $notas);
$ins->execute();
```

---

[← Exposición a Internet](12-exposicion_internet.md) | [↑ Volver al índice](../README.md)
