# 6. Base de datos

[← Volver al índice](../README.md)

Motor: **MariaDB** · Nombre de la BD: `proyecto_asir`

---

## 6.1 Esquema de tablas

### usuarios

Tabla central de autenticación y gestión de usuarios. Incluye un campo `rol` que distingue entre clientes, coaches y el administrador.

| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id | INT | PK, AUTO_INCREMENT | Identificador único |
| nombre | VARCHAR(100) | NOT NULL | Nombre del usuario |
| apellido | VARCHAR(100) | NOT NULL | Apellido del usuario |
| nom_usu | VARCHAR(100) | UNIQUE, NOT NULL | Nombre de usuario (login) |
| email | VARCHAR(150) | UNIQUE, NOT NULL | Correo electrónico |
| password | VARCHAR(255) | NOT NULL | Contraseña cifrada (bcrypt) |
| fecha_nac | DATE | NULL | Fecha de nacimiento |
| localidad | VARCHAR(100) | NULL | Ciudad o localidad |
| rol | ENUM | DEFAULT 'cliente' | `cliente` / `coach` / `admin` |
| created_at | TIMESTAMP | DEFAULT NOW() | Fecha de registro |

### coaches

Perfil profesional del coach. Vinculado a un usuario mediante `usuario_id`.

| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id | INT | PK, AUTO_INCREMENT | Identificador único |
| usuario_id | INT | FK → usuarios.id | Usuario vinculado al coach |
| nombre | VARCHAR(100) | NOT NULL | Nombre del coach |
| apellido | VARCHAR(100) | NOT NULL | Apellido del coach |
| especialidad | VARCHAR(200) | NULL | Especialidad o rol dentro del juego |
| juego | VARCHAR(100) | NULL | Videojuego principal |
| descripcion | TEXT | NULL | Descripción del perfil |
| contacto | VARCHAR(150) | NULL | Email de contacto |
| precio | DECIMAL(8,2) | DEFAULT 0.00 | Precio por hora en euros |
| valoracion_media | DECIMAL(3,2) | DEFAULT 0.00 | Media de valoraciones recibidas |
| created_at | TIMESTAMP | DEFAULT NOW() | Fecha de alta |

### sesiones

Registra cada contratación de coaching entre un cliente y un coach.

| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id | INT | PK, AUTO_INCREMENT | Identificador único |
| cliente_id | INT | FK → usuarios.id | Cliente que contrata |
| coach_id | INT | FK → coaches.id | Coach que imparte |
| fecha | DATETIME | NOT NULL | Fecha y hora de la sesión |
| duracion | INT | DEFAULT 60 | Duración en minutos |
| estado | ENUM | DEFAULT 'pendiente' | `pendiente` / `completada` / `cancelada` |
| notas | TEXT | NULL | Notas o comentarios de la sesión |
| created_at | TIMESTAMP | DEFAULT NOW() | Fecha de creación del registro |

### comentarios

Valoraciones que los clientes dejan sobre los coaches.

| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id | INT | PK, AUTO_INCREMENT | Identificador único |
| coach_id | INT | FK → coaches.id | Coach valorado |
| usuario_id | INT | FK → usuarios.id | Usuario que valora |
| comentario | TEXT | NOT NULL | Texto del comentario |
| valoracion | INT | NOT NULL | Puntuación del 1 al 5 |
| created_at | TIMESTAMP | DEFAULT NOW() | Fecha del comentario |

---

## 6.2 Script SQL de creación

```sql
CREATE DATABASE IF NOT EXISTS proyecto_asir
  CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE proyecto_asir;

CREATE TABLE usuarios (
    id         INT          NOT NULL AUTO_INCREMENT,
    nombre     VARCHAR(100) NOT NULL,
    apellido   VARCHAR(100) NOT NULL,
    nom_usu    VARCHAR(100) NOT NULL,
    email      VARCHAR(150) NOT NULL,
    password   VARCHAR(255) NOT NULL,
    fecha_nac  DATE,
    localidad  VARCHAR(100),
    rol        ENUM('cliente','coach','admin') DEFAULT 'cliente',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uq_email (email),
    UNIQUE KEY uq_nom_usu (nom_usu)
);

CREATE TABLE coaches (
    id               INT           NOT NULL AUTO_INCREMENT,
    usuario_id       INT,
    nombre           VARCHAR(100)  NOT NULL,
    apellido         VARCHAR(100)  NOT NULL,
    especialidad     VARCHAR(200),
    juego            VARCHAR(100),
    descripcion      TEXT,
    contacto         VARCHAR(150),
    precio           DECIMAL(8,2)  DEFAULT 0.00,
    valoracion_media DECIMAL(3,2)  DEFAULT 0.00,
    created_at       TIMESTAMP     DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    CONSTRAINT fk_coach_usuario FOREIGN KEY (usuario_id)
        REFERENCES usuarios(id) ON DELETE SET NULL
);

CREATE TABLE sesiones (
    id         INT      NOT NULL AUTO_INCREMENT,
    cliente_id INT      NOT NULL,
    coach_id   INT      NOT NULL,
    fecha      DATETIME NOT NULL,
    duracion   INT      DEFAULT 60,
    estado     ENUM('pendiente','completada','cancelada') DEFAULT 'pendiente',
    notas      TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    CONSTRAINT fk_sesion_cliente FOREIGN KEY (cliente_id) REFERENCES usuarios(id),
    CONSTRAINT fk_sesion_coach   FOREIGN KEY (coach_id)   REFERENCES coaches(id)
);

CREATE TABLE comentarios (
    id         INT  NOT NULL AUTO_INCREMENT,
    coach_id   INT  NOT NULL,
    usuario_id INT  NOT NULL,
    comentario TEXT NOT NULL,
    valoracion INT  NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    CONSTRAINT fk_com_coach   FOREIGN KEY (coach_id)   REFERENCES coaches(id),
    CONSTRAINT fk_com_usuario FOREIGN KEY (usuario_id) REFERENCES usuarios(id)
);
```

---

## 6.3 Usuario de aplicación

La aplicación nunca usa el usuario `root`. Se crea un usuario con permisos mínimos restringido a la red de base de datos:

```sql
CREATE USER 'rtg_app'@'10.0.0.%' IDENTIFIED BY 'contraseña_segura';
GRANT SELECT, INSERT, UPDATE, DELETE ON proyecto_asir.* TO 'rtg_app'@'10.0.0.%';
FLUSH PRIVILEGES;
```

> La restricción `@'10.0.0.%'` impide cualquier conexión desde fuera de la red `intnet_db`, aunque las credenciales fueran comprometidas.

---

[← Servidores web y NFS](05-nfs-servidores-web.md) | [Siguiente → Herramienta de gestión](07-herramienta-gestion.md)
