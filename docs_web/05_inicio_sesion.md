# `inicio_sesion/inicio_sesion.php` — Inicio de sesión

Página que permite a un usuario autenticarse con su email o nombre de usuario y contraseña.

---

## Bloque PHP — Lógica de autenticación

```php
<?php
session_start();
require_once '../db.php'; // Carga la conexión a la BD

// Si ya hay sesión activa, redirige a la página principal (no tiene sentido loguear dos veces)
if (isset($_SESSION['usuario_id'])) {
    header('Location: /main.php'); exit;
}

$error = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    // Recoge los datos del formulario con valor por defecto vacío si no existen
    $login    = trim($_POST['email'] ?? '');
    $password = $_POST['password'] ?? '';

    // Validación básica: ambos campos son obligatorios
    if (!$login || !$password) {
        $error = 'Por favor, introduce tu email/usuario y contraseña.';
    } else {
        // Consulta preparada: busca el usuario por email O por nombre de usuario
        // Esto permite que el usuario pueda loguearse con cualquiera de los dos
        $stmt = $conn->prepare(
            "SELECT id, nombre, nom_usu, password, rol FROM usuarios WHERE email = ? OR nom_usu = ?"
        );
        $stmt->bind_param("ss", $login, $login); // Mismo valor para ambos parámetros
        $stmt->execute();
        $result = $stmt->get_result();

        if ($row = $result->fetch_assoc()) {
            // password_verify compara la contraseña introducida con el hash almacenado en BD
            // NUNCA se guarda la contraseña en texto plano
            if (password_verify($password, $row['password'])) {

                // Guarda los datos del usuario en la sesión
                $_SESSION['usuario_id']  = $row['id'];
                $_SESSION['usuario_nom'] = $row['nombre'];
                $_SESSION['usuario_usr'] = $row['nom_usu'];
                $_SESSION['usuario_rol'] = $row['rol'];

                // Si es coach, busca su id en la tabla coaches y lo guarda en sesión
                // Necesario para construir la URL de su perfil
                if ($row['rol'] === 'coach') {
                    $sc = $conn->prepare("SELECT id FROM coaches WHERE usuario_id = ?");
                    $sc->bind_param("i", $row['id']);
                    $sc->execute();
                    $rc = $sc->get_result()->fetch_assoc();
                    $sc->close();
                    if ($rc) $_SESSION['coach_id'] = $rc['id'];
                }

                header('Location: /main.php'); exit;

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
```

---

## Formulario HTML

```html
<form id="login_form" method="POST" action="inicio_sesion.php">

    <!-- Campo que acepta email o nombre de usuario -->
    <input type="text" name="email" placeholder="tucorreo@email.com" required>

    <!-- Campo de contraseña con botón para mostrar/ocultar -->
    <div id="password_wrapper">
        <input type="password" id="password" name="password" required>
        <button type="button" onclick="togglePassword()">👁️</button>
    </div>

    <button type="submit">Iniciar sesión</button>
</form>
```

### Función mostrar/ocultar contraseña

```javascript
function togglePassword() {
    const i = document.getElementById('password');
    const b = document.getElementById('toggle_password');

    // Alterna entre 'password' (oculto) y 'text' (visible)
    i.type = i.type === 'password' ? 'text' : 'password';

    // Cambia el icono del botón según el estado
    b.textContent = i.type === 'password' ? '👁️' : '🙈';
}
```

---

## Flujo completo

```
Usuario rellena el formulario
    ↓
PHP recibe los datos por POST
    ↓
Busca en BD por email o nom_usu
    ↓
¿Existe? → password_verify(contraseña_introducida, hash_BD)
    ↓
¿Correcto? → Guarda sesión → Redirige a main.php
    ↓
¿Incorrecto? → Muestra mensaje de error en el mismo formulario
```
