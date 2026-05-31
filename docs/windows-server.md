# 🖥️ Windows Server — Dominio RuleTheGame

**Proyecto:** RuleTheGame.com — Plataforma de coaching gaming  
**Titular:** Francisco Javier Pereira Benito  
**Dominio:** `rulethegame.com`  
**IP del servidor:** `192.168.50.10`  

---

## Índice

1. [Descripción](#descripción)
2. [Estructura del dominio](#estructura-del-dominio)
3. [Instalación del dominio (AD DS)](#instalación-del-dominio-ad-ds)
4. [Unidades Organizativas (OUs)](#unidades-organizativas-ous)
5. [DHCP](#dhcp)
6. [Enlazar equipos al dominio](#enlazar-equipos-al-dominio)
7. [Usuarios y grupos](#usuarios-y-grupos)
8. [Políticas de grupo (GPOs)](#políticas-de-grupo-gpos)
9. [Perfiles móviles](#perfiles-móviles)

---

## Descripción

La empresa **Rule The Game** actúa de intermediario entre coaches de videojuegos y clientes que quieren mejorar su nivel competitivo. Dispone de una página web enlazada a una base de datos que recoge los coaches disponibles, sus tarifas y disponibilidad. El Windows Server gestiona la infraestructura interna de la empresa: usuarios, equipos, permisos y políticas.

---

## Estructura del dominio

La empresa está organizada en **7 departamentos**, cada uno representado como una OU:

| Departamento | Descripción |
|---|---|
| Dirección | Directivos y CEO |
| RRHH | Recursos Humanos |
| Ventas | Equipo comercial |
| Marketing | Diseño y comunicación |
| Administración | Gestión contable y administrativa |
| Informática | Soporte técnico y sistemas |
| Atención al cliente | Soporte y atención a usuarios |

> Existe además un **grupo especial** compuesto por los directivos y los jefes de cada departamento, con permisos elevados.

---

## Instalación del dominio (AD DS)

1. Abrir **Administrador del servidor** → *Agregar roles y características*
2. En *Roles del servidor* seleccionar **Servicios de dominio de Active Directory**
3. Completar la instalación y hacer clic en el aviso amarillo → *Configuración posterior a la implementación*
4. Seleccionar **Agregar un nuevo bosque** e introducir el nombre del dominio:

```
rulethegame.com
```

5. Tras reiniciar, abrir **Usuarios y equipos de Active Directory** desde el buscador de Windows.

---

## Unidades Organizativas (OUs)

Para crear cada OU: clic derecho sobre el dominio → **Nuevo** → **Unidad organizativa**

```
rulethegame.com
├── Administracion
├── Atencion al cliente
├── Direccion
├── Informatica
├── Marketing
├── RRHH
└── Ventas
```

---

## DHCP

### Instalación del rol

1. **Administrador del servidor** → *Agregar roles y características*
2. Seleccionar **Servidor DHCP**
3. Tras instalar, completar la configuración desde el aviso amarillo

### Configuración del ámbito

| Parámetro | Valor |
|---|---|
| Nombre del ámbito | `Red_Corporativa_RTG` |
| Red | `192.168.50.0/24` |
| Rango de distribución | `192.168.50.50` – `192.168.50.200` |
| Máscara de subred | `255.255.255.0` |
| Puerta de enlace (opción 003) | `192.168.50.1` |
| DNS (opción 006) | `192.168.50.10` |
| Sufijo DNS (opción 015) | `rulethegame.com` |
| Duración de concesión | 8 días |

### Exclusiones (IPs estáticas reservadas)

| IP | Dispositivo |
|---|---|
| `192.168.50.10` | Windows Server (DC) |
| `192.168.50.20` | PC-JEFE-AC (Windows Pro) |
| `192.168.50.100` | DB VM (MariaDB) |

### Activar el ámbito por PowerShell

```powershell
Add-DhcpServerv4Scope -Name "Red_Corporativa_RTG" `
    -StartRange 192.168.50.50 `
    -EndRange 192.168.50.200 `
    -SubnetMask 255.255.255.0 `
    -State Active

Set-DhcpServerv4OptionValue `
    -ScopeId 192.168.50.0 `
    -Router 192.168.50.1 `
    -DnsServer 192.168.50.10 `
    -DnsDomain "rulethegame.com"

Add-DhcpServerv4ExclusionRange `
    -ScopeId 192.168.50.0 `
    -StartRange 192.168.50.10 `
    -EndRange 192.168.50.20

Add-DhcpServerv4ExclusionRange `
    -ScopeId 192.168.50.0 `
    -StartRange 192.168.50.100 `
    -EndRange 192.168.50.100
```

---

## Enlazar equipos al dominio

El equipo cliente debe estar en la **misma red** que el servidor y apuntar a él como DNS.

| Parámetro | Valor |
|---|---|
| IP | (dentro del rango DHCP o estática) |
| Máscara | `255.255.255.0` |
| Puerta de enlace | `192.168.50.1` |
| DNS | `192.168.50.10` |

**Pasos:**

1. Panel de control → Sistema → *Cambiar configuración*
2. Pestaña *Nombre de equipo* → **Cambiar** → marcar *Dominio*
3. Introducir: `rulethegame.com`
4. Autenticarse con credenciales de administrador del dominio
5. Reiniciar el equipo

---

## Usuarios y grupos

Cada OU contiene los empleados de su departamento. Todos los usuarios de un mismo departamento pertenecen a un **grupo** con el nombre del departamento para facilitar la gestión de permisos.

### Empleados por departamento

| Departamento | Empleados |
|---|---|
| **Dirección** | Elena Prieto, Julian Ortega, Maria Gonzalez, Ricardo Montes, Sofia Hernandez, Tomas Aguilar |
| **RRHH** | Ana Torres, Pedro Gomez |
| **Ventas** | Carlos Ramos, Lucia Benitez, Mario Silva |
| **Marketing** | Clara Ruiz, Nestor Valdez |
| **Administración** | Esteban Moya, Laura Jimenez, Pablo Muñoz, Rocio Vega |
| **Informática** | Luis Navarro |
| **Atención al cliente** | Antonio Gonzalez, Sandra Rios |

### Patrón de usuario y contraseña

- **Usuario:** `[inicial_nombre][apellido][número]` → ej: `eprieto42`
- **Contraseña temporal:** `A_123456789_[usuario]` → ej: `A_123456789_eprieto42`
- El usuario **debe cambiar la contraseña** en el primer inicio de sesión

### Script PowerShell — Crear todos los usuarios

```powershell
Import-Module ActiveDirectory

$Personal = @{
    "Direccion" = @{
        OU = "OU=Direccion,DC=rulethegame,DC=com"
        Nombres = @("Elena Prieto","Julian Ortega","Maria Gonzalez","Ricardo Montes","Sofia Hernandez","Tomas Aguilar")
    }
    "RRHH" = @{
        OU = "OU=RRHH,DC=rulethegame,DC=com"
        Nombres = @("Ana Torres","Pedro Gomez")
    }
    "Ventas" = @{
        OU = "OU=Ventas,DC=rulethegame,DC=com"
        Nombres = @("Carlos Ramos","Lucia Benitez","Mario Silva")
    }
    "Marketing" = @{
        OU = "OU=Marketing,DC=rulethegame,DC=com"
        Nombres = @("Clara Ruiz","Nestor Valdez")
    }
    "Administracion" = @{
        OU = "OU=Administracion,DC=rulethegame,DC=com"
        Nombres = @("Esteban Moya","Laura Jimenez","Pablo Muñoz","Rocio Vega")
    }
    "Informatica" = @{
        OU = "OU=Informatica,DC=rulethegame,DC=com"
        Nombres = @("Luis Navarro")
    }
    "Atencion_Cliente" = @{
        OU = "OU=AtencionCliente,DC=rulethegame,DC=com"
        Nombres = @("Antonio Gonzalez","Sandra Rios")
    }
}

foreach ($Departamento in $Personal.Keys) {
    $Info    = $Personal[$Departamento]
    $OUPath  = $Info.OU
    $Nombres = $Info.Nombres

    foreach ($NombreCompleto in $Nombres) {
        $Partes   = $NombreCompleto.Split(" ")
        $Nombre   = $Partes[0]
        $Apellido = $Partes[1]
        $Base     = "$($Nombre[0])$($Apellido)".ToLower()
        $Unico    = $false

        do {
            $Num            = Get-Random -Minimum 10 -Maximum 99
            $SamAccountName = "$Base$Num"
            try { Get-ADUser -Identity $SamAccountName -ErrorAction Stop | Out-Null }
            catch { $Unico = $true }
        } while (-not $Unico)

        $Pass = "A_123456789_$SamAccountName" | ConvertTo-SecureString -AsPlainText -Force

        New-ADUser `
            -Name            $NombreCompleto `
            -SamAccountName  $SamAccountName `
            -GivenName       $Nombre `
            -Surname         $Apellido `
            -Path            $OUPath `
            -AccountPassword $Pass `
            -Department      $Departamento `
            -UserPrincipalName "$SamAccountName@rulethegame.com" `
            -ChangePasswordAtLogon $true `
            -Enabled         $true

        Write-Host "[OK] $NombreCompleto | $SamAccountName" -ForegroundColor Green
    }
}
```

### Script PowerShell — Añadir un nuevo empleado

```powershell
$nombre   = Read-Host "Nombre"
$apellido = Read-Host "Apellido"
$ou       = Read-Host "Ruta OU (ej: OU=Ventas,DC=rulethegame,DC=com)"

$base = "$($nombre[0])$($apellido)".ToLower()
for ($i = 1; $i -le 99; $i++) {
    $username = "$base$('{0:D2}' -f $i)"
    if (-not (Get-ADUser -Filter { SamAccountName -eq $username } -ErrorAction SilentlyContinue)) { break }
}

$password = "A123456789_$username"
New-ADUser `
    -Name              "$nombre $apellido" `
    -SamAccountName    $username `
    -GivenName         $nombre `
    -Surname           $apellido `
    -Path              $ou `
    -AccountPassword   (ConvertTo-SecureString $password -AsPlainText -Force) `
    -UserPrincipalName "$username@rulethegame.com" `
    -ChangePasswordAtLogon $true `
    -Enabled           $true

Write-Host "Usuario: $username | Contraseña: $password" -ForegroundColor Green
```

---

## Políticas de grupo (GPOs)

### Vincular GPOs de dominio (script automático)

```powershell
Import-Module GroupPolicy
$domainDN = (Get-ADDomain).DistinguishedName
$gpos     = Get-GPO -All | Where-Object { $_.DisplayName -like "*DOMINIO*" }

foreach ($gpo in $gpos) {
    New-GPLink -Name $gpo.DisplayName -Target $domainDN -Enforced Yes
    Write-Host "Vinculada: $($gpo.DisplayName)" -ForegroundColor Green
}
```

### GPOs por departamento

#### Dirección
| GPO | Descripción |
|---|---|
| `Directivos_SinRestricciones` | Sin restricciones de instalación ni Panel de Control |
| `Directivos_ConfiguracionEscritorio` | Escritorio corporativo profesional |
| `Directivos_RecursosCompartidos` | Acceso a carpetas financieras y estratégicas |

#### Ventas
| GPO | Descripción |
|---|---|
| `Ventas_BloqueoUSB` | Bloqueo de USB para evitar fuga de datos de clientes |
| `Ventas_ConfiguracionCRM` | Configuración optimizada para el CRM |

#### RRHH
| GPO | Descripción |
|---|---|
| `RRHH_SeguridadAlta` | Seguridad elevada para datos de nóminas y contratos |
| `RRHH_BloqueoUSB` | Bloqueo de USB por riesgo de fuga de datos sensibles |
| `RRHH_ProteccionDatos` | Cifrado de carpetas y cumplimiento LOPD |

#### Informática
| GPO | Descripción |
|---|---|
| `IT_AdminTools` | Acceso a herramientas AD, DHCP, DNS, GPO |
| `IT_PowerShell` | Ejecución de scripts PowerShell permitida |
| `IT_RDP` | Acceso remoto a equipos del dominio |
| `IT_InstalacionSoftware` | Instalación libre de software |

#### Marketing
| GPO | Descripción |
|---|---|
| `Marketing_AccesoUSB` | USB en modo solo lectura o cifrado |
| `Marketing_SoftwareCreativo` | Permisos de GPU y software de diseño |
| `Marketing_ImpresionLibre` | Sin restricciones de impresora |

#### Atención al cliente
| GPO | Descripción |
|---|---|
| `Soporte_Lockdown` | Panel de Control bloqueado |
| `Soporte_SoloAppsAutorizadas` | Solo aplicaciones de soporte permitidas |
| `Soporte_VoIP` | Permisos de micrófono y firewall para VoIP |

#### Administración
| GPO | Descripción |
|---|---|
| `Admin_BloqueoUSB` | Bloqueo de USB no autorizados |
| `Admin_SoftwareOficial` | Solo software corporativo permitido |
| `Admin_Contabilidad` | Acceso restringido a datos contables críticos |

---

## Perfiles móviles

Un **perfil móvil** permite que un usuario inicie sesión desde cualquier equipo del dominio y su escritorio/configuración se sincronice automáticamente al cerrar sesión.

### 1. Crear y compartir la carpeta en el servidor

```
C:\PerfilesMoviles  →  Compartir como: Perfiles$
```

> El símbolo `$` oculta la carpeta en red (accesible solo por ruta directa).

### 2. Permisos NTFS sobre la carpeta

| Usuario / Grupo | Permisos |
|---|---|
| SYSTEM | Control total |
| Administradores del dominio | Control total |
| Usuarios del dominio | Crear carpetas / Adjuntar datos |
| Usuarios del dominio | Listar carpeta / Leer datos |
| Usuarios del dominio | Lectura y ejecución |

### 3. Asignar la ruta al usuario en AD

En **Usuarios y equipos de Active Directory** → propiedades del usuario → pestaña **Perfil**:

```
Ruta de acceso al perfil:  \\WIN-8EF1EIM3GN7\Perfiles$\%username%
```

> Sustituye `WIN-8EF1EIM3GN7` por el nombre real de tu servidor.

---

## Red Windows — Integración con infraestructura Vagrant

| Dispositivo | IP | Rol |
|---|---|---|
| Windows Server (DC + DHCP + DNS) | `192.168.50.10` | Controlador de dominio |
| PC-JEFE-AC (Windows Pro) | `192.168.50.20` | Cliente unido al dominio |
| DB VM (MariaDB) | `192.168.50.100` | Base de datos — red interna |
| Rango DHCP | `192.168.50.50–200` | Clientes dinámicos |

---

*Documentación generada para el TFG ASIR2 — RuleTheGame.com*
