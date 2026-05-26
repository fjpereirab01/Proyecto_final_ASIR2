# 14. Red Windows Server + Acceso exclusivo a MariaDB

[← Volver al índice](../README.md)

Este documento describe la integración del entorno Windows (Active Directory) con la infraestructura Vagrant de RuleTheGame, y la configuración del acceso exclusivo a MariaDB desde el equipo del jefe de atención al cliente.

---

## 14.1 Descripción del entorno Windows

El entorno Windows está compuesto por dos máquinas virtuales VirtualBox que conviven con las VMs Vagrant:

| VM | IP | Función |
|---|---|---|
| Windows Server 2016 | `192.168.50.10` | Controlador de dominio (DC), DNS del dominio |
| Windows Pro (PC-JEFE-AC) | `192.168.50.20` | Equipo del jefe de atención al cliente |

Ambas VMs están conectadas a una red interna de VirtualBox llamada `windows`, con rango `192.168.50.0/24`, independiente de las redes Vagrant.

El dominio Active Directory configurado es `rulethegame.com`, gestionado por el DC `WIN-8EF1EIM3GN7.rulethegame.com`.

---

## 14.2 Configuración de red Windows Server / Windows Pro

### Windows Server
| Campo | Valor |
|---|---|
| IP | `192.168.50.10` |
| Máscara | `255.255.255.0` |
| Gateway | (vacío) |
| DNS | `127.0.0.1` (el propio servidor) |

### Windows Pro (PC-JEFE-AC)
| Campo | Valor |
|---|---|
| IP | `192.168.50.20` |
| Máscara | `255.255.255.0` |
| Gateway | `192.168.50.10` |
| DNS | `192.168.50.10` |

### Verificación de conectividad y dominio
Desde Windows Pro, comprobar que el DC es accesible:
```cmd
nltest /dsgetdc:rulethegame.com
```
Resultado esperado: nombre del DC y su IP `192.168.50.10`.

---

## 14.3 Integración de la VM db en la red Windows

Para que el Windows Pro pueda acceder a MariaDB, la VM `db` necesita una interfaz en la red `192.168.50.x`. Se añade una segunda interfaz en el Vagrantfile conectada a la red interna `windows`:

```ruby
config.vm.define "db" do |db|
    db.vm.box      = "debian/bullseye64"
    db.vm.hostname = "database"
    db.vm.network "private_network", virtualbox__intnet: "intnet_db", ip: "10.0.0.100"
    db.vm.network "private_network", virtualbox__intnet: "windows", ip: "192.168.50.100"  # ← nueva
    db.vm.provision "shell", path: "provision_db.sh"
end
```

Tras ejecutar `vagrant reload db`, la VM db dispone de dos interfaces:

| Interfaz | IP | Red |
|---|---|---|
| `eth1` | `10.0.0.100` | `intnet_db` (web1/web2) |
| `eth2` | `192.168.50.100` | `windows` (Windows Pro/Server) |

---

## 14.4 Configuración de MariaDB

### bind-address
Por defecto MariaDB solo escuchaba en `10.0.0.100`. Se cambia a `0.0.0.0` para aceptar conexiones desde todas las interfaces:

```bash
sudo sed -i 's/bind-address = 10.0.0.100/bind-address = 0.0.0.0/' \
    /etc/mysql/mariadb.conf.d/50-server.cnf
sudo systemctl restart mariadb
```

### Usuario gestor exclusivo
Se crea un usuario MariaDB que **solo acepta conexiones desde la IP del Windows Pro**:

```sql
CREATE USER 'gestor'@'192.168.50.20' IDENTIFIED BY 'GestorRTG2024!';
GRANT ALL PRIVILEGES ON proyecto_asir.* TO 'gestor'@'192.168.50.20';
FLUSH PRIVILEGES;
```

Con este usuario, ninguna otra máquina puede autenticarse con esas credenciales aunque llegue al puerto 3306.

---

## 14.5 Firewall (iptables) en la VM db

Se aplica una doble capa de seguridad a nivel de firewall para el puerto 3306:

```bash
# Permitir web1 y web2 (red interna Vagrant)
sudo iptables -I INPUT 1 -p tcp --dport 3306 -s 10.0.0.0/24 -j ACCEPT

# Permitir solo el Windows Pro
sudo iptables -A INPUT -p tcp --dport 3306 -s 192.168.50.20 -j ACCEPT

# Bloquear cualquier otra IP
sudo iptables -A INPUT -p tcp --dport 3306 -j DROP

# Guardar reglas
sudo apt-get install -y iptables-persistent
sudo netfilter-persistent save
```

### Resultado
| Origen | Puerto 3306 | Resultado |
|---|---|---|
| `10.0.0.21` / `10.0.0.22` (web1/web2) | 3306 | ✅ ACCEPT |
| `192.168.50.20` (Windows Pro) | 3306 | ✅ ACCEPT |
| Cualquier otra IP | 3306 | ❌ DROP |

---

## 14.6 Configuración de HeidiSQL en Windows Pro

HeidiSQL es el cliente de base de datos utilizado por el jefe de atención al cliente para gestionar las incidencias.

**Parámetros de conexión:**

| Campo | Valor |
|---|---|
| Tipo | MySQL/MariaDB (TCP/IP) |
| Host | `192.168.50.100` |
| Usuario | `gestor` |
| Contraseña | `GestorRTG2024!` |
| Puerto | `3306` |
| Base de datos | (vacío — seleccionar `proyecto_asir` manualmente) |

Una vez conectado, hacer doble clic sobre `proyecto_asir` en el panel izquierdo para seleccionarla como base de datos activa.

---

## 14.7 Diagrama de acceso

```
Windows Pro (192.168.50.20)
        │
        │  TCP 3306
        ▼
VM db eth2 (192.168.50.100)
        │
        │  iptables ACCEPT solo 192.168.50.20
        ▼
MariaDB → usuario gestor@192.168.50.20
        │
        ▼
Base de datos: proyecto_asir
```

---

[← Errores encontrados](13-errores-y-soluciones.md) | [→ Sistema de incidencias](15-incidencias.md) | [↑ Volver al índice](../README.md)
