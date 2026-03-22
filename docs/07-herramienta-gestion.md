# 7. Herramienta de gestión del entorno

[← Volver al índice](../README.md)

La herramienta está compuesta por dos ficheros que trabajan en conjunto:

| Fichero | Tecnología | Función |
|---|---|---|
| `gestor_docker.sh` | Bash | Backend: ejecuta comandos Docker dentro del contenedor gestor |
| `gestor_tkinter.py` | Python 3 + tkinter | Frontend: interfaz gráfica de escritorio sin dependencias externas |

---

## 7.1 Arquitectura

```
[gestor_tkinter.py]
        │
        │  subprocess.run()
        ▼
docker exec gestor-docker bash /gestor/gestor_docker.sh <comando>
        │
        ▼
[gestor_docker.sh]  →  Docker API  →  contenedores del proyecto
```

La interfaz gráfica **nunca llama directamente a Docker**: toda la lógica pasa por el script Bash dentro del contenedor `gestor-docker`. Esto permite reutilizar el script desde cualquier terminal de forma independiente.

**Contenedores gestionados:** `haproxy-balanceador`, `apache-web1`, `apache-web2`, `mariadb-servidor`

---

## 7.2 Script Bash: `gestor_docker.sh`

### Comandos disponibles

| Comando | Descripción |
|---|---|
| `estado` | Tabla con estado `activo`/`detenido` de cada contenedor vía `docker inspect` |
| `levantar` | `docker start` sobre los cuatro contenedores del proyecto |
| `apagar` | `docker stop` sobre los cuatro contenedores del proyecto |
| `backup` | Verifica que MariaDB esté activo y ejecuta `mariadb-dump` con timestamp |
| `listar_backups` | Lista los `.sql` del directorio de backups con fecha y tamaño (`ls -lh`) |
| `cron_ver` | Muestra la tarea cron de backup activa en el contenedor |
| `cron_eliminar` | Reescribe el crontab eliminando la tarea de backup |
| `cron_programar "EXPR"` | Crea el script externo, arranca cron e instala la expresión en el crontab |

**Directorio de backups:** `/gestor/backups/`  
Los backups se nombran con timestamp (`backup_proyecto_asir_YYYYMMDD_HHMMSS.sql`) y se conservan **los 20 más recientes** automáticamente.

### Fragmento: detección de estado

```bash
estado_contenedor() {
    local s
    s=$(docker inspect --format='{{.State.Status}}' "$1" 2>/dev/null)
    [[ "$s" == "running" ]] && echo "activo" || echo "detenido"
}
```

### Fragmento: backup con validación

```bash
local f="$BACKUP_DIR/backup_${DB_NAME}_$(date +%Y%m%d_%H%M%S).sql"
docker exec "$DB_CONTAINER" mariadb-dump -u"$DB_USER" -p"$DB_PASSWORD" "$DB_NAME" > "$f" 2>/dev/null

if [[ $? -eq 0 && -s "$f" ]]; then
    echo "Backup creado: $f"
else
    echo "Error al crear backup."
    rm -f "$f"
fi
```

---

## 7.3 Interfaz gráfica: `gestor_tkinter.py`

### Componentes

| Componente | Tipo tkinter | Función |
|---|---|---|
| Ventana principal | `Tk()` | Menú con acceso a los dos módulos |
| Ventana entorno | `Toplevel` | Estado de contenedores + Levantar / Apagar / Actualizar |
| Área de resultados | `ScrolledText` | Muestra la salida del script; en `DISABLED` para evitar edición |
| Ventana backups | `Toplevel` | Backup manual, listado, ver/eliminar/programar cron |
| Ventana programar cron | `Toplevel` | Radiobuttons con frecuencias predefinidas + campo libre |
| Diálogos de confirmación | `messagebox` | Confirmación antes de operaciones destructivas |

### Frecuencias de backup disponibles

| Opción | Expresión cron |
|---|---|
| Cada hora | `0 * * * *` |
| Cada 6 horas | `0 */6 * * *` |
| Diario a medianoche | `0 0 * * *` |
| Semanal (lunes 2:00) | `0 2 * * 1` |
| Mensual (día 1 3:00) | `0 3 1 * *` |

### Lanzar la interfaz

```bash
python3 gestor_tkinter.py
```

> Requiere que el contenedor `gestor-docker` esté en ejecución y Docker accesible en el host.

---

[← Base de datos](06-base-de-datos.md) | [Siguiente → Seguridad](08-seguridad.md)
