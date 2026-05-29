# `pag_registro/registro.php` — Registro de usuarios

Página que permite crear una nueva cuenta de usuario con rol `cliente` por defecto.

---

## Bloque PHP — Lógica de registro

```php
<?php
session_start();
require_once '../db.php';

$error = ''; $success = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {

    // Recoge y limpia todos los campos del formulario
    // trim() elimina espacios al inicio y al final
    $nombre    = trim($_POST['nombre'] ?? '');
    $apellido  = trim($_POST['apellido'] ?? '');
    $nom_usu   = trim($_POST['nom_usu'] ?? '');
    $fecha_nac = trim($_POST['fecha_nac'] ?? '');
    $localidad = trim($_POST['localidad'] ?? '');
    $email     = trim($_POST['email'] ?? '');
    $password  = $_POST['password'] ?? '';

    // Validación 1: campos obligatorios vacíos
    if (!$nombre || !$apellido || !$nom_usu || !$email || !$password) {
        $error = 'Por favor, rellena todos los campos obligatorios.';

    // Validación 2: formato de email correcto
    } elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $error = 'El email no tiene un formato válido.';

    // Validación 3: contraseña mínimo 8 caracteres
    } elseif (strlen($password) < 8) {
        $error = 'La contraseña debe tener al menos 8 caracteres.';

    } else {
        // Comprueba si ya existe una cuenta con ese email o nombre de usuario
        $stmt = $conn->prepare("SELECT id FROM usuarios WHERE email = ? OR nom_usu = ?");
        $stmt->bind_param("ss", $email, $nom_usu);
        $stmt->execute();
        $stmt->store_result(); // Necesario para usar num_rows

        if ($stmt->num_rows > 0) {
            $error = 'El email o nombre de usuario ya están registrados.';
        } else {
            // Genera el hash seguro de la contraseña con bcrypt
            // NUNCA se guarda la contraseña en texto plano
            $hash = password_hash($password, PASSWORD_DEFAULT);

            // Inserta el nuevo usuario en la BD
            // El rol por defecto en la BD es 'cliente', no se especifica aquí
            $ins = $conn->prepare(
                "INSERT INTO usuarios (nombre, apellido, nom_usu, fecha_nac, localidad, email, password)
                 VALUES (?, ?, ?, ?, ?, ?, ?)"
            );
            $ins->bind_param("sssssss", $nombre, $apellido, $nom_usu, $fecha_nac, $localidad, $email, $hash);

            if ($ins->execute()) {
                $success = '¡Cuenta creada! Ya puedes iniciar sesión.';
            } else {
                $error = 'Error al crear la cuenta. Inténtalo de nuevo.';
            }
            $ins->close();
        }
        $stmt->close();
    }
}
?>
```

---

## Formulario HTML

```html
<form id="registro_form" method="POST" action="registro.php">

    <!-- Fila con dos campos en paralelo -->
    <div class="form_fila">
        <input type="text" name="nombre" required>
        <input type="text" name="apellido" required>
    </div>

    <input type="text" name="nom_usu" required>    <!-- Nombre de usuario único -->
    <input type="date" name="fecha_nac">            <!-- Opcional -->
    <input type="text" name="localidad">            <!-- Opcional -->
    <input type="email" name="email" required>
    <input type="password" name="password" required>

    <button type="submit">Crear cuenta</button>
</form>
```

Los campos `fecha_nac` y `localidad` son opcionales. Si se deja vacío, se guarda como `NULL` en la BD.

---

## Mensajes de feedback

```php
// Si hay error, se muestra en rojo antes del formulario
<?php if ($error): ?>
    <div style="background:#fee;color:#c00;">⚠️ <?= htmlspecialchars($error) ?></div>
<?php endif; ?>

// Si el registro fue exitoso, se muestra en verde con enlace para ir a login
<?php if ($success): ?>
    <div style="background:#efe;color:#060;">
        ✅ <?= htmlspecialchars($success) ?>
        <a href="/inicio_sesion/inicio_sesion.php">Ir a iniciar sesión →</a>
    </div>
<?php endif; ?>
```

`htmlspecialchars()` convierte caracteres especiales en entidades HTML para evitar inyección de código (XSS).

---

## Flujo completo

```
Usuario rellena el formulario → POST
    ↓
Validaciones: campos vacíos, email válido, contraseña ≥ 8 chars
    ↓
¿Ya existe ese email/usuario en BD? → Error
    ↓
password_hash($password) → genera hash bcrypt
    ↓
INSERT INTO usuarios → cuenta creada
    ↓
Muestra mensaje de éxito con enlace a login
```
