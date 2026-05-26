# 15. Sistema de incidencias

[← Volver al índice](../README.md)

Este documento describe el sistema de gestión de incidencias implementado en RuleTheGame: la tabla en MariaDB, las páginas PHP para clientes y coaches, y la vista de gestión desde HeidiSQL para el jefe de atención al cliente.

---

## 15.1 Descripción del sistema

El sistema de incidencias permite a los usuarios de la plataforma (clientes y coaches) reportar problemas o solicitudes al equipo de atención al cliente. El flujo es el siguiente:

1. El usuario crea una incidencia desde su perfil en la web
2. La incidencia queda almacenada en MariaDB con estado `abierta`
3. El jefe de atención al cliente la visualiza desde HeidiSQL en el Windows Pro
4. El jefe asigna la incidencia a un empleado modificando el campo `asignado_a`
5. El estado puede cambiar a `en_curso` o `resuelta` según se gestione

---

## 15.2 Tabla `incidencias` en MariaDB

```sql
CREATE TABLE incidencias (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    usuario_id  INT NOT NULL,
    rol         ENUM('coach', 'cliente') NOT NULL,
    titulo      VARCHAR(150) NOT NULL,
    descripcion TEXT NOT NULL,
    estado      ENUM('abierta', 'en_curso', 'resuelta') NOT NULL DEFAULT 'abierta',
    asignado_a  INT DEFAULT NULL,
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (usuario_id) REFERENCES usuarios(id),
    FOREIGN KEY (asignado_a) REFERENCES usuarios(id)
);
```

### Campos
| Campo | Tipo | Descripción |
|---|---|---|
| `id` | INT AUTO_INCREMENT | Identificador único de la incidencia |
| `usuario_id` | INT (FK) | Usuario que abre la incidencia |
| `rol` | ENUM | Rol del usuario: `coach` o `cliente` |
| `titulo` | VARCHAR(150) | Título descriptivo de la incidencia |
| `descripcion` | TEXT | Detalle completo del problema |
| `estado` | ENUM | Estado actual: `abierta`, `en_curso`, `resuelta` |
| `asignado_a` | INT (FK, nullable) | Empleado asignado para gestionar la incidencia |
| `created_at` | DATETIME | Fecha y hora de creación |

---

## 15.3 Páginas PHP

### `incidencias/incidencias.php`
Formulario para que el usuario cree una nueva incidencia.

**Funcionalidad:**
- Redirige al login si el usuario no tiene sesión activa
- Detecta automáticamente el rol del usuario desde `$_SESSION['usuario_rol']`
- Inserta el registro en la tabla `incidencias` con estado `abierta`
- Muestra mensaje de éxito o error tras el envío

**Campos del formulario:**
- Título (max 150 caracteres)
- Descripción detallada

**Inserción en BD:**
```php
$stmt = $conn->prepare(
    "INSERT INTO incidencias (usuario_id, rol, titulo, descripcion) VALUES (?, ?, ?, ?)"
);
$stmt->bind_param("isss", $usuario_id, $rol, $titulo, $descripcion);
```

---

### `incidencias/mis_incidencias.php`
Lista de incidencias del usuario autenticado.

**Funcionalidad:**
- Muestra todas las incidencias del usuario ordenadas por fecha descendente
- Badge de color según el estado de cada incidencia:
  - 🟡 `abierta` — fondo amarillo oscuro
  - 🔵 `en_curso` — fondo azul oscuro
  - 🟢 `resuelta` — fondo verde oscuro
- Botón "Nueva incidencia" para acceder al formulario
- Mensaje vacío si no hay incidencias

**Query principal:**
```php
$stmt = $conn->prepare(
    "SELECT id, titulo, descripcion, estado, created_at
     FROM incidencias WHERE usuario_id = ? ORDER BY created_at DESC"
);
```

---

## 15.4 Acceso desde el perfil del usuario

Se añade un botón "🎫 Mis incidencias" en `perfil/perfil.php`, visible para todos los usuarios logueados, justo debajo de la fecha de registro:

```html
<a href="https://192.168.10.1/incidencias/mis_incidencias.php"
   style="display:inline-block;margin-top:12px;padding:9px 20px;
          border-radius:8px;border:1.5px solid var(--acento);
          background:transparent;color:var(--acento);font-weight:700;
          font-size:14px;font-family:'Nunito',sans-serif;text-decoration:none;">
    🎫 Mis incidencias
</a>
```

---

## 15.5 Gestión desde HeidiSQL (Windows Pro)

El jefe de atención al cliente gestiona las incidencias directamente desde HeidiSQL conectado a MariaDB con el usuario `gestor`.

### Ver incidencias abiertas
```sql
SELECT i.id, u.nom_usu, i.rol, i.titulo, i.descripcion, i.estado, i.created_at
FROM incidencias i
JOIN usuarios u ON i.usuario_id = u.id
WHERE i.estado = 'abierta'
ORDER BY i.created_at ASC;
```

### Asignar incidencia a un empleado
```sql
UPDATE incidencias
SET asignado_a = <id_empleado>, estado = 'en_curso'
WHERE id = <id_incidencia>;
```

### Marcar como resuelta
```sql
UPDATE incidencias
SET estado = 'resuelta'
WHERE id = <id_incidencia>;
```

---

## 15.6 Diagrama del flujo

```
Usuario (web)
    │
    │  POST incidencias.php
    ▼
MariaDB tabla incidencias
    │  estado = 'abierta'
    ▼
HeidiSQL (Windows Pro, usuario gestor)
    │
    ├─ Consulta incidencias abiertas
    ├─ Asigna empleado (asignado_a)
    └─ Actualiza estado → en_curso / resuelta
```

---

[← Red Windows y MariaDB](14-red-windows-mariadb.md) | [↑ Volver al índice](../README.md)
