# Práctica 1: Router Cisco como Servidor DHCP
## Objetivo

Configurar el router R1 para que actúe como servidor DHCP de la red LAN.

Topología
```
      PC1
       |
      SW1
       |
   G0/0 R1
```
Direccionamiento
| Dispositivo|	Interfaz|	Dirección IP|
|----|---|--|
R1	|G0/0 |	192.168.1.1/24
PC1	|DHCP |Automática
### Paso 1: Configurar la interfaz LAN
R1
```
enable
configure terminal
 
interface g0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
end
```
#### Verificar:
```
show ip interface brief
```
### Paso 2: Excluir direcciones

Reservaremos:
Gateway
Servidores futuros
```
configure terminal
 
ip dhcp excluded-address 192.168.1.1 192.168.1.20
 
end
```
### Paso 3: Crear el pool DHCP
```
configure terminal
 
ip dhcp pool AULA
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
domain-name instituto.local
lease 7
 
end
```
### Paso 4: Configurar el cliente

En PC1:

Desktop → IP Configuration

Seleccionar:
DHCP
Comprobación

PC1 debe recibir:
- IP: 192.168.1.21
- Máscara: 255.255.255.0
- Gateway: 192.168.1.1
- DNS: 8.8.8.8

#### Verificaciones
Configuración DHCP
```
show running-config | section dhcp
```
Concesiones
```
show ip dhcp binding
```

### Estadísticas  
show ip dhcp server statistics

# Solución: 
```
interface g0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
ip dhcp excluded-address 192.168.1.1 192.168.1.20
ip dhcp pool AULA
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8
domain-name instituto.local
lease 7
```

# Práctica 2: Dos Redes DHCP en el Mismo Router
## Objetivo

Crear dos pools DHCP distintos.

Topología
```
               SW1
                |
             G0/0
                |
               R1
                |
             G0/1
                |
               SW2
```
Direccionamiento
| Red	| Gateway	| 
|---------|---------|
| 192.168.10.0/24	| 192.168.10.1	|
| 192.168.20.0/24	| 192.168.20.1	|

Configuración
Interfaces
```
interface g0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
 
interface g0/1
ip address 192.168.20.1 255.255.255.0
no shutdown
```
Exclusiones
```
ip dhcp excluded-address 192.168.10.1 192.168.10.10
 
ip dhcp excluded-address 192.168.20.1 192.168.20.10
```
Pool LAN1
```
ip dhcp pool LAN1
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 8.8.8.8
domain-name aula.local
```
Pool LAN2
```
ip dhcp pool LAN2
 
network 192.168.20.0 255.255.255.0
 
default-router 192.168.20.1
 
dns-server 8.8.8.8
 
domain-name aula.local
```
### Resultado esperado
PC1
Plain Text
192.168.10.x
PC2
Plain Text
192.168.20.x


# Práctica 3: DHCP Relay (ip helper-address)

 Configuración DHCP con Relay para Server Device EndPoint

## Topología

- Red izquierda: **192.168.1.0/24**
- Enlace entre routers: **10.0.0.0/30**
- Red derecha: **192.168.2.0/24**
- Servidor DHCP: **Server0 (192.168.2.2)**

                    RED 192.168.1.0/24
┌───────┐     ┌──────────┐     ┌──────────────┐
│  PC3  │────▶│ Switch2  │────▶│ R0 G0/1      │
└───────┘     └──────────┘     │192.168.1.1   │
                               └──────┬───────┘
                                      │ G0/0
                                      │ 10.0.0.1/30
                                      │
                               ┌──────┴───────┐
                               │ Router2 G0/0 │
                               │ 10.0.0.2/30  │
                               └──────┬───────┘
                                      │ G0/1
                                      │ 192.168.2.1
                                      │
                    RED 192.168.2.0/24│
                                      │
                           ┌──────────┴──────────┐
                           │      Switch3        │
                           └───────┬───────┬─────┘
                                   │       │
                                   │       │
                         ┌─────────▼─┐   ┌─▼────────┐
                         │ Server0   │   │   PC4    │
                         │192.168.2.2│   │ DHCP     │
                         │ DHCP      │   │          │
                         └───────────┘   └──────────┘
---

## 1. Configuración de PC3 (DHCP desde Server0)

Como PC3 está en una red distinta al servidor DHCP, es necesario configurar DHCP Relay en R0.

### R0

```bash
interface g0/1
 ip address 192.168.1.1 255.255.255.0
 ip helper-address 192.168.2.2
 no shutdown
```

### PC3

Desktop → IP Configuration → DHCP.

Dirección obtenida esperada:

```text
IP Address: 192.168.1.100
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.1.1
DNS Server: 8.8.8.8
```

---

## 2. Configuración de PC4 (DHCP desde Server0)

PC4 está en la misma red que Server0, por lo que NO necesita DHCP Relay.

### Pool DHCP para PC4

Server0 → Services → DHCP → Add

```text
Pool Name: LAN2
Default Gateway: 192.168.2.1
DNS Server: 8.8.8.8
Start IP Address: 192.168.2.100
Subnet Mask: 255.255.255.0
Maximum Number of Users: 50
```

### PC4

Desktop → IP Configuration → DHCP.

Dirección obtenida esperada:

```text
IP Address: 192.168.2.100
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.2.1
DNS Server: 8.8.8.8
```

---

## Resumen

| Equipo | Obtiene IP por DHCP | Requiere DHCP Relay |
|----------|----------|----------|
| PC3 | Sí | Sí, en R0 (`ip helper-address 192.168.2.2`) |
| PC4 | Sí | No |

---

## Pruebas

Desde PC3:

```bash
ping 192.168.1.1
ping 192.168.2.2
```

Desde PC4:

```bash
ping 192.168.2.1
ping 192.168.2.2
```
