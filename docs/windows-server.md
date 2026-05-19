# Windows Server — Integración y Seguridad

## Descripción

Se ha añadido una máquina virtual Windows Server a la infraestructura del proyecto. El objetivo es que tenga salida a internet a través del router Vagrant, pero que no sea accesible desde el exterior ni desde la DMZ, protegiéndola de posibles ataques dirigidos a la web o al balanceador.

---

## Topología

```
Internet
   |
 [Router Vagrant]
   |          |
 [DMZ]      [LAN 192.168.10.0/24]
   |              |
[Balanceador]  [Windows Server 192.168.10.50]
[Web1 / Web2]
[Base de datos]
```

El Windows Server se conecta a la red LAN interna (`192.168.10.0/24`) mediante un adaptador **Host-Only** de VirtualBox (`VirtualBox Host-Only Ethernet Adapter #4`), la misma red donde el router tiene la IP `192.168.10.1`.

---

## Configuración de red del Windows Server

| Parámetro | Valor |
|-----------|-------|
| IP | `192.168.10.50` |
| Máscara | `255.255.255.0` |
| Gateway | `192.168.10.1` |
| DNS | `8.8.8.8` |

El tráfico de salida a internet pasa por el router Vagrant, que aplica NAT hacia `eth0`.

---

## Reglas iptables en el Router

Se detectó que la chain FORWARD tenía **policy ACCEPT** y estaba vacía, lo que permitía tráfico libre entre todas las redes. Se aplicaron las siguientes reglas para restringir el tráfico:

```bash
# Vaciar reglas anteriores
sudo iptables -F FORWARD

# Política por defecto: denegar todo tráfico entre redes
sudo iptables -P FORWARD DROP

# Permitir tráfico de conexiones ya establecidas o relacionadas
sudo iptables -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT

# Permitir DMZ → internet
sudo iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT

# Permitir LAN → internet
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
```

### ¿Por qué así?

- La política `DROP` por defecto bloquea cualquier tráfico entre redes que no esté explícitamente permitido.
- La regla `ESTABLISHED,RELATED` permite que las respuestas a conexiones iniciadas desde la LAN o DMZ hacia internet vuelvan correctamente.
- No se añade ninguna regla que permita DMZ → LAN, por lo que el balanceador o los servidores web no pueden iniciar conexiones hacia el Windows Server.

---

## Reglas de entrada en Windows Defender Firewall

Se descubrió que aunque las iptables del router bloqueaban el tráfico entre redes, el balanceador podía seguir llegando al Windows Server a través de la red NAT interna de VirtualBox (`10.0.2.x`), que es compartida por todas las VMs de Vagrant y no pasa por el router.

Para solucionar esto se configuraron dos reglas en **Windows Defender Firewall con seguridad avanzada → Reglas de entrada**:

### Regla 1: Permitir LAN interna

| Campo | Valor |
|-------|-------|
| Tipo | Personalizada |
| Programa | Todos |
| Protocolo | Cualquiera |
| IP remota | `192.168.10.0/24` |
| Acción | Permitir la conexión |
| Perfil | Dominio, Privado, Público |
| Nombre | `Permitir LAN interna` |

### Regla 2: Bloquear todo lo demás

| Campo | Valor |
|-------|-------|
| Tipo | Personalizada |
| Programa | Todos |
| Protocolo | Cualquiera |
| IP remota | `0.0.0.0/1` y `128.0.0.0/1` |
| Acción | Bloquear la conexión |
| Perfil | Dominio, Privado, Público |
| Nombre | `Bloquear todo externo` |

> **Nota:** Windows Firewall no acepta la notación `0.0.0.0/0`. Se usan `0.0.0.0/1` y `128.0.0.0/1` que juntas cubren todo el rango de IPs posibles.

### ¿Por qué así?

- Windows Firewall aplica primero las reglas de **permitir** antes que las de **bloquear**, por lo que la LAN siempre tendrá acceso aunque exista la regla de bloqueo global.
- Esto garantiza que aunque el tráfico llegue por cualquier interfaz (incluida la red NAT de VirtualBox), solo las IPs de la LAN interna podrán comunicarse con el Windows Server.

---

## Resultado final

| Origen | Destino | Resultado |
|--------|---------|-----------|
| Windows Server | Internet | ✅ Permitido |
| LAN `192.168.10.x` | Windows Server | ✅ Permitido |
| DMZ `172.16.0.x` | Windows Server | ❌ Bloqueado |
| Internet | Windows Server | ❌ Bloqueado |
