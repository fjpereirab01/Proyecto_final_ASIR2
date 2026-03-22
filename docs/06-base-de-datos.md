# 6. Base de datos

[← Volver al índice](../README.md)

Motor: **MariaDB** · Nombre de la BD: `proyecto_asir`

---

## 6.1 Modelo entidad-relación

Relaciones principales del modelo:

- `USUARIO 1 — 0..1 COACH` — un usuario puede ser coach (subtipo)
- `USUARIO 1 — 0..1 CLIENTE` — un usuario puede ser cliente (subtipo)
- `COACH 1 — 0..N SESION` — un coach imparte muchas sesiones
- `CLIENTE 1 — 0..N SESION` — un cliente contrata muchas sesiones
- `VIDEOJUEGO 1 — 0..N SESION` — un videojuego puede estar en muchas sesiones
- `SESION 1 — 0..1 PAGO` — una sesión puede estar pendiente de pago o tener uno asociado
- `COACH 1 — 0..N DISPONIBILIDAD` — cada coach define múltiples franjas horarias

---

## 6.2 Esquema de tablas

### Usuarios

Tabla central de autenticación. El login se realiza con `email` + `contraseña`.

| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_usuario | INT | PK, AUTO_INCREMENT | Identificador único |
| nombre | VARCHAR(100) | NOT NULL | Nombre completo |
| email | VARCHAR(150) | UNIQUE, NOT NULL | Correo electrónico (login) |
| contraseña | VARCHAR(255) | NOT NULL | Contraseña cifrada (bcrypt) |
| rol | ENUM | NOT NULL | `coach` / `cliente` |
| fecha_registro | DATE | DEFAULT CURRENT_DATE | Fecha de creación de cuenta |

### Coaches

Subtipo de USUARIO con información profesional del entrenador. La PK es también FK hacia Usuarios.

| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_coach | INT | FK → Usuarios.id_usuario | Referencia al usuario coach |
| especialidad | VARCHAR(100) | NOT NULL | Videojuego principal |
| experiencia | TEXT | NULL | Años de experiencia y logros |
| tarifa_hora | DECIMAL(10,2) | NOT NULL | Precio por hora en euros |
| disponibilidad | DATETIME | NULL | Campo auxiliar de disponibilidad |

### Clientes

Subtipo de USUARIO con el perfil del jugador que contrata sesiones.

| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_cliente | INT | FK → Usuarios.id_usuario | Referencia al usuario cliente |
| nivel_actual | VARCHAR(50) | NULL | Rango en su videojuego principal |
| objetivos | TEXT | NULL | Metas de mejora con el coaching |

### Videojuegos

Catálogo centralizado de juegos disponibles en la plataforma.

| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_juego | INT | PK, AUTO_INCREMENT | Identificador único |
| nombre | VARCHAR(100) | UNIQUE, NOT NULL | Nombre del juego (ej: CS2, Valorant) |
| descripcion | TEXT | NULL | Descripción del juego |
| categoria | VARCHAR(50) | NOT NULL | FPS, MOBA, Battle Royale, etc. |

### Sesiones

Tabla central del modelo. Registra cada contratación de coaching.

| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_sesion | INT | PK, AUTO_INCREMENT | Identificador único |
| id_coach | INT | FK → Coaches | Coach que imparte |
| id_cliente | INT | FK → Clientes | Cliente que contrata |
| id_juego | INT | FK → Videojuegos | Videojuego de la sesión |
| fecha_inicio | DATETIME | NOT NULL | Inicio programado |
| duracion_min | INT | NOT NULL | Duración en minutos |
| estado | ENUM | DEFAULT 'pendiente' | `pendiente` / `completada` / `cancelada` |
| notas | TEXT | NULL | Comentarios post-sesión |

> La lógica de cancelación se implementa comprobando si `NOW() < fecha_inicio - plazo_minutos`.

### Pagos

Transacción económica asociada a cada sesión completada.

| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_pago | INT | PK, AUTO_INCREMENT | Identificador único |
| id_sesion | INT | FK → Sesiones, UNIQUE | Sesión asociada |
| monto | DECIMAL(10,2) | NOT NULL | Importe en euros |
| fecha_pago | DATE | DEFAULT CURRENT_DATE | Fecha de la transacción |
| metodo | VARCHAR(50) | NOT NULL | Tarjeta, PayPal, transferencia |
| estado | ENUM | DEFAULT 'pendiente' | `pagado` / `pendiente` |

### Disponibilidad

Franjas horarias semanales recurrentes en las que cada coach acepta reservas.

| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_disponibilidad | INT | PK, AUTO_INCREMENT | Identificador del bloque |
| id_coach | INT | FK → Coaches | Coach al que pertenece |
| dia_semana | TINYINT | NOT NULL | 0 = Lunes … 6 = Domingo |
| hora_inicio | TIME | NOT NULL | Inicio del bloque disponible |
| hora_fin | TIME | NOT NULL | Fin del bloque disponible |

---

## 6.3 Script SQL de creación

```sql
CREATE DATABASE IF NOT EXISTS proyecto_asir
  CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE proyecto_asir;

CREATE TABLE Usuarios (
    id_usuario     INT          NOT NULL AUTO_INCREMENT,
    nombre         VARCHAR(100) NOT NULL,
    email          VARCHAR(150) NOT NULL,
    contrasena     VARCHAR(255) NOT NULL,
    rol            ENUM('coach','cliente') NOT NULL,
    fecha_registro DATE         DEFAULT (CURRENT_DATE),
    PRIMARY KEY (id_usuario),
    UNIQUE KEY uq_email (email)
);

CREATE TABLE Coaches (
    id_coach       INT           NOT NULL,
    especialidad   VARCHAR(100)  NOT NULL,
    experiencia    TEXT,
    tarifa_hora    DECIMAL(10,2) NOT NULL,
    disponibilidad DATETIME,
    PRIMARY KEY (id_coach),
    CONSTRAINT fk_coach_usuario FOREIGN KEY (id_coach)
        REFERENCES Usuarios(id_usuario) ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE Clientes (
    id_cliente   INT         NOT NULL,
    nivel_actual VARCHAR(50),
    objetivos    TEXT,
    PRIMARY KEY (id_cliente),
    CONSTRAINT fk_cliente_usuario FOREIGN KEY (id_cliente)
        REFERENCES Usuarios(id_usuario) ON DELETE CASCADE ON UPDATE CASCADE
);

CREATE TABLE Videojuegos (
    id_juego    INT          NOT NULL AUTO_INCREMENT,
    nombre      VARCHAR(100) NOT NULL,
    descripcion TEXT,
    categoria   VARCHAR(50)  NOT NULL,
    PRIMARY KEY (id_juego),
    UNIQUE KEY uq_nombre_juego (nombre)
);

CREATE TABLE Sesiones (
    id_sesion    INT      NOT NULL AUTO_INCREMENT,
    id_coach     INT      NOT NULL,
    id_cliente   INT      NOT NULL,
    id_juego     INT      NOT NULL,
    fecha_inicio DATETIME NOT NULL,
    duracion_min INT      NOT NULL,
    estado       ENUM('pendiente','completada','cancelada') DEFAULT 'pendiente',
    notas        TEXT,
    PRIMARY KEY (id_sesion),
    CONSTRAINT fk_sesion_coach   FOREIGN KEY (id_coach)   REFERENCES Coaches(id_coach),
    CONSTRAINT fk_sesion_cliente FOREIGN KEY (id_cliente) REFERENCES Clientes(id_cliente),
    CONSTRAINT fk_sesion_juego   FOREIGN KEY (id_juego)   REFERENCES Videojuegos(id_juego)
);

CREATE TABLE Pagos (
    id_pago    INT           NOT NULL AUTO_INCREMENT,
    id_sesion  INT           NOT NULL,
    monto      DECIMAL(10,2) NOT NULL,
    fecha_pago DATE          DEFAULT (CURRENT_DATE),
    metodo     VARCHAR(50)   NOT NULL,
    estado     ENUM('pagado','pendiente') DEFAULT 'pendiente',
    PRIMARY KEY (id_pago),
    UNIQUE KEY uq_pago_sesion (id_sesion),
    CONSTRAINT fk_pago_sesion FOREIGN KEY (id_sesion) REFERENCES Sesiones(id_sesion)
);

CREATE TABLE Disponibilidad (
    id_disponibilidad INT     NOT NULL AUTO_INCREMENT,
    id_coach          INT     NOT NULL,
    dia_semana        TINYINT NOT NULL COMMENT '0=Lun 1=Mar 2=Mie 3=Jue 4=Vie 5=Sab 6=Dom',
    hora_inicio       TIME    NOT NULL,
    hora_fin          TIME    NOT NULL,
    PRIMARY KEY (id_disponibilidad),
    CONSTRAINT fk_disp_coach FOREIGN KEY (id_coach)
        REFERENCES Coaches(id_coach) ON DELETE CASCADE ON UPDATE CASCADE
);
```

---

## 6.4 Usuario de aplicación

La aplicación nunca usa el usuario `root`. Se crea un usuario con permisos mínimos restringido a la red de base de datos:

```sql
CREATE USER 'rtg_app'@'10.0.0.%' IDENTIFIED BY 'contraseña_segura';
GRANT SELECT, INSERT, UPDATE, DELETE ON proyecto_asir.* TO 'rtg_app'@'10.0.0.%';
FLUSH PRIVILEGES;
```

> La restricción `@'10.0.0.%'` impide cualquier conexión desde fuera de la red `intnet_db`, aunque las credenciales fueran comprometidas.

---

[← Servidores web y NFS](05-nfs-servidores-web.md) | [Siguiente → Herramienta de gestión](07-herramienta-gestion.md)
