# RuleTheGame.com — Memoria Técnica TFG ASIR

**Administración de Sistemas Informáticos en Red (ASIR) · Curso 2024–2025**  
**Autor:** F.J. Pereira Benito  
**Infraestructura:** Vagrant + VirtualBox · Router · Apache Balancer · WEB1 · WEB2 · MariaDB + NFS

---

## Índice

1. [Introducción](#1-introducción)
2. [Arquitectura de la infraestructura](#2-arquitectura-de-la-infraestructura)
3. [Configuración del router](#3-configuración-del-router)
4. [Balanceo de carga con Apache](#4-balanceo-de-carga-con-apache)
5. [Servidores web y NFS](#5-servidores-web-y-compartición-nfs)
6. [Base de datos](#6-base-de-datos)
7. [Herramienta de gestión](#7-herramienta-de-gestión-del-entorno)
8. [Seguridad](#8-seguridad-de-la-infraestructura)
9. [Guía de despliegue](#9-guía-de-despliegue)
10. [Conclusiones](#10-conclusiones)

---

## 1. Introducción

### 1.1 Descripción del proyecto

**RuleTheGame.com** es una plataforma web orientada al ámbito de los videojuegos profesionales cuyo objetivo es conectar a usuarios con coaches especializados en distintos videojuegos, facilitando la contratación de sesiones de entrenamiento personalizado.

La plataforma gestiona dos tipos de usuarios diferenciados:
- **Coach:** profesional que fija su precio por hora, define la duración de cada sesión y establece su horario de disponibilidad.
- **Cliente:** usuario registrado que puede contratar sesiones con cualquier coach disponible siempre que los horarios sean compatibles.

Adicionalmente, la plataforma incorpora un módulo de gestión de incidencias: los usuarios pueden notificar problemas a través de un formulario web y los empleados, según su rol, tienen acceso a esos registros para gestionarlos y resolverlos.

### 1.2 Objetivos

- Desplegar una infraestructura virtualizada reproducible mediante Vagrant y VirtualBox.
- Implementar segmentación de red con una zona DMZ y una red de base de datos aislada.
- Configurar balanceo de carga con Apache como proxy inverso entre dos servidores web.
- Compartir los ficheros de la aplicación web mediante NFS desde el servidor de base de datos.
- Diseñar e implementar una base de datos relacional que soporte el modelo de negocio.
- Garantizar la seguridad del tráfico mediante reglas iptables en el router.
- Desarrollar una herramienta gráfica de administración del entorno Docker con tkinter y Bash.

---

## 2. Arquitectura de la infraestructura

### 2.1 Visión general

El tráfico del exterior llega al **router**, que aplica NAT y redirige las peticiones HTTP al **balanceador** en la DMZ. El balanceador distribuye entre **WEB1** y **WEB2** mediante proxy inverso. Ambos servidores montan por NFS los ficheros de la aplicación desde el servidor **db**, que también aloja MariaDB. La base de datos no tiene conectividad con el exterior.

### 2.2 Inventario de máquinas virtuales

| VM | Hostname | Box | RAM / CPU | Función |
|---|---|---|---|---|
| `router` | router-asir | debian/bullseye64 | 512 MB / 1 | Puerta de enlace, NAT, iptables |
| `balancer` | balanceador | debian/bullseye64 | 512 MB / 1 | Apache proxy inverso, round-robin |
| `web1` | web1 | debian/bullseye64 | 512 MB / 1 | Servidor web Apache (monta NFS) |
| `web2` | web2 | debian/bullseye64 | 512 MB / 1 | Servidor web Apache (monta NFS) |
| `db` | database | debian/bullseye64 | 512 MB / 1 | MariaDB + servidor NFS |

### 2.3 Segmentación de red

| Red | Tipo Vagrant | Rango IP | Máquinas |
|---|---|---|---|
| Host-Only (LAN) | `private_network` | `192.168.10.0/24` | Router (.1) |
| intnet_dmz (DMZ) | `internal network` | `172.16.0.0/24` | Router (.1), Balancer (.10), WEB1 (.21), WEB2 (.22) |
| intnet_db (DB) | `internal network` | `10.0.0.0/24` | WEB1 (.21), WEB2 (.22), DB (.100) |

---

## 3. Configuración del router

### 3.1 IP Forwarding

```bash
sysctl -w net.ipv4.ip_forward=1
```

Para hacerlo persistente se añade `net.ipv4.ip_forward=1` a `/etc/sysctl.conf`.

### 3.2 Reglas iptables

| Regla | Tabla/Cadena | Efecto |
|---|---|---|
| DNAT puerto 80 | nat / PREROUTING | Redirige el tráfico TCP entrante en el puerto 80 hacia el balanceador (172.16.0.10:80) |
| MASQUERADE | nat / POSTROUTING | Enmascara la IP origen permitiendo acceso a Internet desde las VMs internas |

```bash
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 172.16.0.10:80
iptables -t nat -A POSTROUTING -j MASQUERADE
```

> Para persistencia entre reinicios: instalar `iptables-persistent` → guarda las reglas en `/etc/iptables/rules.v4`.

---

## 4. Balanceo de carga con Apache

### 4.1 Módulos necesarios

```bash
a2enmod proxy proxy_http proxy_balancer lbmethod_byrequests status
```

### 4.2 Configuración del VirtualHost

```apache
<Proxy "balancer://mycluster">
    BalancerMember "http://172.16.0.21"
    BalancerMember "http://172.16.0.22"
</Proxy>

ProxyPass        "/" "balancer://mycluster/"
ProxyPassReverse "/" "balancer://mycluster/"
```

El algoritmo `byrequests` distribuye las peticiones en **round-robin**. Si un servidor no responde, Apache lo marca como fuera de servicio automáticamente.

---

## 5. Servidores web y compartición NFS

### 5.1 Servidor NFS en la máquina `db`

La máquina `db` exporta el directorio de la aplicación web para que WEB1 y WEB2 sirvan siempre los mismos ficheros.

**`/etc/exports`:**
```
/var/www/rulethegame  10.0.0.0/24(rw,sync,no_subtree_check)
```

| Opción | Efecto |
|---|---|
| `rw` | Lectura y escritura desde los clientes |
| `sync` | Confirma escrituras en disco antes de responder |
| `no_subtree_check` | Mejora el rendimiento deshabilitando la verificación de subdirectorios |

### 5.2 Montaje NFS en WEB1 y WEB2

**`/etc/fstab`** (en cada servidor web):
```
10.0.0.100:/var/www/rulethegame  /var/www/html  nfs  defaults  0 0
```

### 5.3 Aislamiento de la base de datos

La máquina `db` **únicamente** tiene interfaz en `intnet_db` (10.0.0.0/24). No tiene acceso a la DMZ ni al exterior. Solo WEB1 y WEB2 pueden alcanzar los puertos 3306 (MariaDB) y 2049 (NFS).

---

## 6. Base de datos

Motor: **MariaDB** · Nombre de la BD: `proyecto_asir`

### 6.1 Modelo entidad-relación

Entidades y relaciones principales:

- `USUARIO 1 — 0..1 COACH` — un usuario puede ser coach (subtipo)
- `USUARIO 1 — 0..1 CLIENTE` — un usuario puede ser cliente (subtipo)
- `COACH 1 — 0..N SESION` — un coach imparte muchas sesiones
- `CLIENTE 1 — 0..N SESION` — un cliente contrata muchas sesiones
- `VIDEOJUEGO 1 — 0..N SESION` — un juego puede estar en muchas sesiones
- `SESION 1 — 0..1 PAGO` — una sesión puede estar pendiente de pago o tener uno asociado
- `COACH 1 — 0..N DISPONIBILIDAD` — cada coach define múltiples franjas horarias

### 6.2 Esquema de tablas

#### Usuarios
| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_usuario | INT | PK, AUTO_INCREMENT | Identificador único |
| nombre | VARCHAR(100) | NOT NULL | Nombre completo |
| email | VARCHAR(150) | UNIQUE, NOT NULL | Correo electrónico (login) |
| contraseña | VARCHAR(255) | NOT NULL | Contraseña cifrada (bcrypt) |
| rol | ENUM | NOT NULL | `coach` / `cliente` |
| fecha_registro | DATE | DEFAULT CURRENT_DATE | Fecha de creación de cuenta |

#### Coaches
| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_coach | INT | FK → Usuarios | Referencia al usuario coach |
| especialidad | VARCHAR(100) | NOT NULL | Videojuego principal |
| experiencia | TEXT | NULL | Años y logros |
| tarifa_hora | DECIMAL(10,2) | NOT NULL | Precio por hora en euros |
| disponibilidad | DATETIME | NULL | Campo auxiliar de disponibilidad |

#### Clientes
| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_cliente | INT | FK → Usuarios | Referencia al usuario cliente |
| nivel_actual | VARCHAR(50) | NULL | Rango en su videojuego principal |
| objetivos | TEXT | NULL | Metas de mejora con el coaching |

#### Videojuegos
| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_juego | INT | PK, AUTO_INCREMENT | Identificador único |
| nombre | VARCHAR(100) | UNIQUE, NOT NULL | Nombre del juego (ej: CS2, Valorant) |
| descripcion | TEXT | NULL | Descripción del juego |
| categoria | VARCHAR(50) | NOT NULL | FPS, MOBA, Battle Royale, etc. |

#### Sesiones
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

#### Pagos
| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_pago | INT | PK, AUTO_INCREMENT | Identificador único |
| id_sesion | INT | FK → Sesiones, UNIQUE | Sesión asociada |
| monto | DECIMAL(10,2) | NOT NULL | Importe en euros |
| fecha_pago | DATE | DEFAULT CURRENT_DATE | Fecha de la transacción |
| metodo | VARCHAR(50) | NOT NULL | Tarjeta, PayPal, transferencia |
| estado | ENUM | DEFAULT 'pendiente' | `pagado` / `pendiente` |

#### Disponibilidad
| Campo | Tipo | Restricciones | Descripción |
|---|---|---|---|
| id_disponibilidad | INT | PK, AUTO_INCREMENT | Identificador del bloque |
| id_coach | INT | FK → Coaches | Coach al que pertenece |
| dia_semana | TINYINT | NOT NULL | 0 = Lunes … 6 = Domingo |
| hora_inicio | TIME | NOT NULL | Inicio del bloque disponible |
| hora_fin | TIME | NOT NULL | Fin del bloque disponible |

### 6.3 Script SQL de creación

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

### 6.4 Usuario de aplicación

```sql
CREATE USER 'rtg_app'@'10.0.0.%' IDENTIFIED BY 'contraseña_segura';
GRANT SELECT, INSERT, UPDATE, DELETE ON proyecto_asir.* TO 'rtg_app'@'10.0.0.%';
FLUSH PRIVILEGES;
```

> La restricción `@'10.0.0.%'` limita la conexión a la red de base de datos. La aplicación nunca usa el usuario `root`.

---

## 7. Herramienta de gestión del entorno

La herramienta está compuesta por dos ficheros que trabajan en conjunto:

| Fichero | Tecnología | Función |
|---|---|---|
| `gestor_docker.sh` | Bash | Backend: ejecuta comandos Docker dentro del contenedor gestor |
| `gestor_tkinter.py` | Python 3 + tkinter | Frontend: interfaz gráfica de escritorio sin dependencias externas |

### 7.1 Arquitectura

```
[gestor_tkinter.py]  →  docker exec gestor-docker bash /gestor/gestor_docker.sh <comando>  →  [gestor_docker.sh]
       GUI                              subprocess.run()                                           Docker API
```

La interfaz gráfica nunca llama directamente a Docker: toda la lógica pasa por el script Bash dentro del contenedor `gestor-docker`, lo que permite reutilizar el script desde cualquier terminal de forma independiente.

### 7.2 Comandos del script Bash

| Comando | Descripción |
|---|---|
| `estado` | Tabla con estado `activo`/`detenido` de cada contenedor vía `docker inspect` |
| `levantar` | `docker start` sobre los cuatro contenedores del proyecto |
| `apagar` | `docker stop` sobre los cuatro contenedores del proyecto |
| `backup` | Verifica que MariaDB esté activo y ejecuta `mariadb-dump` con timestamp |
| `listar_backups` | Lista los `.sql` del directorio de backups con fecha y tamaño |
| `cron_ver` | Muestra la tarea cron de backup activa en el contenedor |
| `cron_eliminar` | Reescribe el crontab eliminando la tarea de backup |
| `cron_programar "EXPR"` | Crea el script externo, arranca cron e instala la expresión en el crontab |

**Contenedores gestionados:** `haproxy-balanceador`, `apache-web1`, `apache-web2`, `mariadb-servidor`

**Directorio de backups:** `/gestor/backups/` — se conservan los 20 más recientes automáticamente.

#### Fragmento: detección de estado

```bash
estado_contenedor() {
    local s
    s=$(docker inspect --format='{{.State.Status}}' "$1" 2>/dev/null)
    [[ "$s" == "running" ]] && echo "activo" || echo "detenido"
}
```

#### Fragmento: backup con validación

```bash
# Nombre único con timestamp: backup_proyecto_asir_20260309_131806.sql
local f="$BACKUP_DIR/backup_${DB_NAME}_$(date +%Y%m%d_%H%M%S).sql"
docker exec "$DB_CONTAINER" mariadb-dump -u"$DB_USER" -p"$DB_PASSWORD" "$DB_NAME" > "$f" 2>/dev/null

if [[ $? -eq 0 && -s "$f" ]]; then
    echo "Backup creado: $f"
else
    echo "Error al crear backup."
    rm -f "$f"
fi
```

### 7.3 Interfaz gráfica (tkinter)

| Componente | Tipo tkinter | Función |
|---|---|---|
| Ventana principal | `Tk()` | Menú con acceso a los dos módulos |
| Ventana entorno | `Toplevel` | Estado de contenedores + Levantar / Apagar / Actualizar |
| Área de resultados | `ScrolledText` | Muestra la salida del script; en `DISABLED` para evitar edición |
| Ventana backups | `Toplevel` | Backup manual, listado, ver/eliminar/programar cron |
| Ventana programar cron | `Toplevel` | Radiobuttons con frecuencias predefinidas + campo libre |
| Diálogos de confirmación | `messagebox` | Confirmación antes de operaciones destructivas |

#### Frecuencias de backup disponibles

| Opción | Expresión cron |
|---|---|
| Cada hora | `0 * * * *` |
| Cada 6 horas | `0 */6 * * *` |
| Diario a medianoche | `0 0 * * *` |
| Semanal (lunes 2:00) | `0 2 * * 1` |
| Mensual (día 1 3:00) | `0 3 1 * *` |

#### Lanzar la interfaz

```bash
python3 gestor_tkinter.py
```

---

## 8. Seguridad de la infraestructura

| Capa | Medida |
|---|---|
| **Red** | Tres redes aisladas: LAN, DMZ e intnet_db. Movimiento lateral limitado. |
| **Firewall** | iptables en el router: solo se reenvía TCP/80 hacia la DMZ. Resto descartado. |
| **MariaDB** | Sin interfaz en DMZ ni LAN. Usuario `rtg_app` con permisos mínimos restringido a `10.0.0.%`. |
| **NFS** | Exportación limitada a `10.0.0.0/24`. Opción `sync` activa. Sin `no_root_squash`. |

---

## 9. Guía de despliegue

### 9.1 Requisitos previos

- VirtualBox 6.1+
- Vagrant 2.3+
- ~3 GB de RAM libre
- Conexión a Internet para la descarga inicial de la box `debian/bullseye64` (~400 MB)

### 9.2 Arranque

```bash
# Desde el directorio con el Vagrantfile
vagrant up

# Verificar que todas las VMs están running
vagrant status
```

### 9.3 Acceso SSH

```bash
vagrant ssh router
vagrant ssh balancer
vagrant ssh web1
vagrant ssh web2
vagrant ssh db
```

### 9.4 Lanzar la herramienta de gestión

```bash
python3 gestor_tkinter.py
```

> Requiere que el contenedor `gestor-docker` esté en ejecución y Docker accesible en el host.

### 9.5 Parada y destrucción

```bash
vagrant halt        # Suspender todas las VMs
vagrant destroy -f  # Destruir completamente el entorno
```

---

## 10. Conclusiones

El proyecto integra los principales contenidos del ciclo ASIR: virtualización con Vagrant/VirtualBox, redes segmentadas con DMZ, servicios Linux (Apache, MariaDB, NFS), seguridad con iptables, diseño de bases de datos relacionales y desarrollo de herramientas de administración con Bash y Python.

**Puntos destacados:**
- Arquitectura escalable: añadir un tercer servidor web solo requiere un nuevo `BalancerMember` en Apache.
- NFS centralizado: el código fuente es idéntico en ambos servidores web en todo momento.
- Aislamiento de BD: sin interfaz en DMZ, usuario con permisos mínimos, acceso solo desde la red `intnet_db`.
- Herramienta de gestión sin dependencias externas, reutilizable desde terminal o desde GUI.

**Mejoras futuras:**
- HTTPS con Let's Encrypt en el balanceador.
- Migración a contenedores Docker para reducir consumo de recursos.
- Monitorización con Prometheus + Grafana.

---

*RuleTheGame.com · TFG ASIR · F.J. Pereira Benito · 2024–2025*
