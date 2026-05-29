# Sistema de mensajería — `mensajes/mensajes.php` y `mensajes/conversacion.php`

El sistema de mensajería permite comunicación directa entre usuarios (clientes y coaches). Consta de dos páginas: el listado de conversaciones y el chat individual con polling en tiempo real.

---

## `mensajes.php` — Listado de conversaciones

### Obtener conversaciones únicas

```php
// Obtiene los IDs de todos los usuarios con los que se ha intercambiado algún mensaje
// CASE WHEN: si el usuario actual es el emisor, devuelve el receptor, y viceversa
$stmt = $conn->prepare("
    SELECT DISTINCT
        CASE WHEN emisor_id = ? THEN receptor_id
             ELSE emisor_id END AS otro_id
    FROM mensajes
    WHERE emisor_id = ? OR receptor_id = ?
");
$stmt->bind_param("iii", $id, $id, $id);
```

### Construir el array de conversaciones

```php
$conversaciones = [];
foreach ($otros as $o) {
    $otro_id = $o['otro_id'];

    // Datos del otro usuario
    $su = $conn->prepare("SELECT id, nombre, apellido, nom_usu FROM usuarios WHERE id=?");
    // ...

    // Último mensaje de esta conversación (para preview)
    $lm = $conn->prepare("
        SELECT id, emisor_id, contenido, created_at
        FROM mensajes
        WHERE (emisor_id=? AND receptor_id=?) OR (emisor_id=? AND receptor_id=?)
        ORDER BY created_at DESC LIMIT 1  -- Solo el más reciente
    ");
    // ...

    // Mensajes no leídos de este usuario hacia mí
    $nl = $conn->prepare(
        "SELECT COUNT(*) FROM mensajes WHERE emisor_id=? AND receptor_id=? AND leido=0"
    );
    // ...

    $conversaciones[] = ['otro' => $otro, 'ultimo' => $ultimo, 'no_leidos' => $nl_count];
}

// Ordena las conversaciones por el mensaje más reciente (más reciente primero)
usort($conversaciones, fn($a, $b) =>
    strtotime($b['ultimo']['created_at']) - strtotime($a['ultimo']['created_at'])
);
```

### Tarjeta de conversación

```php
<?php foreach ($conversaciones as $c):
    $preview = strlen($ultimo['contenido']) > 50
        ? substr($ultimo['contenido'], 0, 50) . '...'  // Trunca a 50 chars
        : $ultimo['contenido'];
    $es_mio = $ultimo['emisor_id'] == $id; // Para mostrar "Tú: " si el último mensaje es mío
?>
<a href="/mensajes/conversacion.php?con=<?= $otro['id'] ?>" class="conv_card">
    <div class="conv_avatar">JD</div>
    <div class="conv_info">
        <div class="conv_nombre">Juan Doe</div>
        <!-- Preview del último mensaje con "Tú: " si lo envié yo -->
        <div class="conv_preview"><?= $es_mio ? 'Tú: ' : '' ?><?= htmlspecialchars($preview) ?></div>
    </div>
    <div class="conv_meta">
        <span class="conv_fecha">20/05/2026</span>
        <!-- Badge rojo si hay mensajes no leídos -->
        <?php if ($c['no_leidos'] > 0): ?>
            <span class="conv_badge"><?= $c['no_leidos'] ?></span>
        <?php endif; ?>
    </div>
</a>
```

---

## `conversacion.php` — Chat individual

### Validaciones iniciales

```php
$mi_id  = $_SESSION['usuario_id'];
$con_id = intval($_GET['con'] ?? 0);

// Impide abrir conversación consigo mismo
if (!$con_id || $con_id === $mi_id) {
    header('Location: /mensajes/mensajes.php'); exit;
}

// Verifica que el otro usuario existe en la BD
$stmt = $conn->prepare("SELECT id, nombre, apellido, nom_usu FROM usuarios WHERE id=?");
// Si no existe, redirige
if (!$otro) { header('Location: /mensajes/mensajes.php'); exit; }
```

### Enviar mensaje

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['contenido'])) {
    $contenido = trim($_POST['contenido'] ?? '');
    if ($contenido) {
        $ins = $conn->prepare("INSERT INTO mensajes (emisor_id, receptor_id, contenido) VALUES (?,?,?)");
        $ins->bind_param("iis", $mi_id, $con_id, $contenido);
        $ins->execute();
    }

    // Si la petición viene por AJAX (X-Requested-With header), devuelve 'ok' en texto
    // Si es un POST normal (sin JS), redirige para evitar reenvío del formulario
    if (!empty($_SERVER['HTTP_X_REQUESTED_WITH'])) {
        echo 'ok'; exit;
    }
    header('Location: conversacion.php?con=' . $con_id); exit;
}
```

### Polling AJAX para mensajes nuevos

```php
// Endpoint especial que devuelve mensajes nuevos desde un ID dado
if (isset($_GET['poll'])) {
    $desde = intval($_GET['desde'] ?? 0);
    $stmt2 = $conn->prepare("
        SELECT id, emisor_id, contenido, created_at
        FROM mensajes
        WHERE ((emisor_id=? AND receptor_id=?) OR (emisor_id=? AND receptor_id=?))
          AND id > ?          -- Solo mensajes posteriores al último recibido
        ORDER BY created_at ASC
    ");
    $stmt2->bind_param("iiiii", $mi_id, $con_id, $con_id, $mi_id, $desde);
    // ...
    header('Content-Type: application/json');
    echo json_encode($rows); // Devuelve array JSON de mensajes nuevos
    exit;
}
```

### Marcar mensajes como leídos

```php
// Al cargar la conversación, marca todos los mensajes del otro usuario como leídos
$upd = $conn->prepare(
    "UPDATE mensajes SET leido=1 WHERE emisor_id=? AND receptor_id=? AND leido=0"
);
$upd->bind_param("ii", $con_id, $mi_id);
$upd->execute();
```

### JavaScript — Envío y polling

```javascript
const MI_ID  = <?= $mi_id ?>;   // ID del usuario actual (inyectado desde PHP)
const CON_ID = <?= $con_id ?>;  // ID del interlocutor
let ultimoId = <?= $ultimo_id ?>; // ID del último mensaje cargado

// Enviar con Enter (Shift+Enter hace salto de línea)
document.getElementById('conv_input').addEventListener('keydown', function(e) {
    if (e.key === 'Enter' && !e.shiftKey) {
        e.preventDefault();
        enviarMensaje();
    }
});

function enviarMensaje() {
    const texto = document.getElementById('conv_input').value.trim();
    if (!texto) return;

    // Envía el mensaje por AJAX sin recargar la página
    fetch('conversacion.php?con=' + CON_ID, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'X-Requested-With': 'XMLHttpRequest' // Indica que es AJAX
        },
        body: 'contenido=' + encodeURIComponent(texto)
    }).then(() => {
        document.getElementById('conv_input').value = '';
        cargarNuevos(); // Carga inmediatamente el mensaje enviado
    });
}

function agregarBurbuja(m) {
    const clase = m.emisor_id == MI_ID ? 'mio' : 'suyo'; // Clase CSS según quién envió
    const hora  = new Date(m.created_at).toLocaleTimeString('es-ES', {hour:'2-digit', minute:'2-digit'});
    const div   = document.createElement('div');
    div.className  = 'msg_burbuja ' + clase;
    div.dataset.id = m.id; // data-id para identificar burbujas ya renderizadas
    div.innerHTML  = `<div class="msg_texto">${m.contenido.replace(/\n/g,'<br>')}</div>
                      <span class="msg_hora">${hora}</span>`;
    caja.appendChild(div);
    caja.scrollTop = caja.scrollHeight; // Scroll automático al fondo
    ultimoId = m.id; // Actualiza el último ID conocido
}

function cargarNuevos() {
    // Consulta al servidor solo los mensajes posteriores al último conocido
    fetch('conversacion.php?con=' + CON_ID + '&poll=1&desde=' + ultimoId)
        .then(r => r.json())
        .then(msgs => {
            msgs.forEach(m => {
                // Comprueba que la burbuja no existe ya (evita duplicados)
                if (!document.querySelector('[data-id="' + m.id + '"]')) {
                    agregarBurbuja(m);
                }
            });
        });
}

// Polling cada 3 segundos para recibir mensajes nuevos en tiempo real
setInterval(cargarNuevos, 3000);
```
