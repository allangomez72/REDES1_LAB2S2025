# Clase 9 - Enrutamiento Dinámico

## Introducción al Ruteo Dinámico

El **ruteo dinámico** permite a los routers compartir automáticamente información sobre redes conectadas, adaptándose a cambios en la topología sin intervención manual.

### Ventajas vs. Ruteo Estático:
- **Actualización automática** de tablas de ruteo
- **Escalabilidad** en redes grandes
- **Tolerancia a fallos** - reruteo automático
- **Menor administración**

---

## Protocolos de Ruteo Dinámico

### 1. **OSPF (Open Shortest Path First)**
- **Tipo**: Link-State, IGP
- **Métrica**: Costo (basado en ancho de banda)
- **Velocidad de Convergencia**: Rápida
- **Uso ideal**: Redes empresariales grandes

### 2. **EIGRP (Enhanced Interior Gateway Routing Protocol)**
- **Tipo**: Híbrido, IGP (propietario Cisco)
- **Métrica**: Compuesta (ancho de banda, delay, carga, confiabilidad)
- **Velocidad de Convergencia**: Muy rápida
- **Característica**: Usa DUAL algorithm

### 3. **RIP (Routing Information Protocol)**
- **Tipo**: Distance-Vector, IGP
- **Métrica**: Hop Count
- **Límite**: 15 hops máximo
- **Uso**: Redes pequeñas/simples

---

## Configuración Detallada por Protocolo

### **OSPF - Configuración Base**
```bash
! Habilitar OSPF
router ospf <process-id>
 network <network> <wildcard-mask> area <area-id>

! Ejemplo práctico
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.1.1.0 0.0.0.255 area 0
 no passive-interface gi0/0
```

### **EIGRP - Configuración Base**
```bash
! Habilitar EIGRP
router eigrp <as-number>
 network <network> <wildcard-mask>

! Ejemplo práctico
router eigrp 100
 network 192.168.1.0 0.0.0.255
 network 10.0.0.0
 no auto-summary
```

### **RIP - Configuración Base**
```bash
! Habilitar RIP
router rip
 version 2
 network <network-address>
 no auto-summary

! Ejemplo práctico
router rip
 version 2
 network 192.168.1.0
 network 10.0.0.0
 no auto-summary
```

---

SWITCH LAYER 3 - 1

```bash
enable
conf t
hostname S1
no ip domain-lookup
ip routing
vlan 10
name USERS
exit
vlan 20
name SERVERS
exit
interface vlan10
ip address 10.10.10.1 255.255.255.0
no shutdown
exit
interface vlan20
ip address 10.10.20.1 255.255.255.0
no shutdown
exit

interface gi0/1
no switchport
ip address 10.0.4.2 255.255.255.252
no shutdown
exit

!GRUPO1
interface range fa0/1-2
channel-group 1 mode active
interface port-channel 1
no switchport

!10.0.4.4 /30
ip address 10.0.4.5 255.255.255.252
no shut
exit

interface fa0/4
switchport trunk encapsulation dot1q
switchport mode trunk
exit
interface fa0/3
switchport trunk encapsulation dot1q
switchport mode trunk
exit
```

OSPF para SWITCH LAYER 3-1

```bash
router ospf 1
network 10.10.10.0 0.0.0.255 area 0
network 10.10.20.0 0.0.0.255 area 0
network 10.0.4.4 0.0.0.3 area 0
network 10.0.4.0 0.0.0.3 area 0
```

SWITCH LAYER 3-2

```bash
enable
conf t
hostname S2
no ip domain-lookup
ip routing

interface fa0/3
no switchport
ip address 10.10.30.1 255.255.255.0
no shutdown
exit

interface gi0/1
no switchport
ip address 10.0.4.9 255.255.255.252
no shutdown
exit

!GRUPO1
interface range fa0/1-2
channel-group 1 mode active
interface port-channel 1
no switchport

!10.0.4.4 /30
ip address 10.0.4.6 255.255.255.252
no shut
exit
```

SWITCH LAYER 3-2

```bash
router ospf 1
network 10.10.30.0 0.0.0.255 area 0
network 10.0.4.4 0.0.0.3 area 0
network 10.0.4.8 0.0.0.3 area 0
```

SWITCHES L2

```bash
!Primer switch
enable
conf t
interface fa0/1
switchport mode trunk
exit
interface fa0/2
switchport mode access
switchport access vlan 10
exit

!Segundo switch
enable
conf t
vlan 20
 name SERVERS
exit
interface fa0/1
switchport mode trunk
exit
interface fa0/2
switchport mode access
switchport access vlan 20
exit
```

ROUTER (R1) - RIP

```bash
enable
conf t
interface gi0/0
ip address 10.0.4.13 255.255.255.252
no shutdown
exit
interface gi0/1  
ip address 192.168.100.5 255.255.255.252
no shutdown
exit

router rip
version 2
no auto-summary
network 10.0.4.12
network 192.168.100.0
exit
```

SWITCH(L3)

```bash
enable
conf t
vlan 10
name UNO
exit
vlan 20
name DOS
exit

interface fa0/1
switchport mode access
switchport access vlan 10
exit
interface fa0/2
switchport mode access
switchport access vlan 20
exit

interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shut
exit
interface vlan 20
ip address 192.168.20.1 255.255.255.0
no shut
exit

ip routing
interface gi0/1
no switchport
ip address 192.168.100.6 255.255.255.252
no shutdown
exit

!Habilitar como tal el protocolo de RIP
router rip
version 2
no auto-summary
network 192.168.10.0
network 192.168.20.0
network 192.168.100.0
exit

```

ROUTER (R2) - EIGRP

```bash
enable
conf t
interface gi0/0
ip address 10.0.4.10 255.255.255.252
no shutdown
exit
interface gi0/1
no shut
exit
interface gi0/1.30
interface gi0/1.40

interface gi0/1.30
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
exit
interface gi0/1.40
encapsulation dot1Q 40
ip address 192.168.40.1 255.255.255.0
exit

!configurar EIGRP para la comunicación 
router eigrp 1
network 192.168.30.0
network 192.168.40.0
network 10.0.4.8
no auto-summary
exit

```

SWITCH(L2)

```bash
enable
conf t
vlan 30
name ESTUDIANTES
exit
vlan 40
name MAESTROS
exit

interface fa0/2
switchport mode access
switchport access vlan 30
exit
interface fa0/3
switchport mode access
switchport access vlan 40
exit

interface fa0/1
switchport mode trunk
switchport trunk allowed vlan 30,40
end
wr

```
---

## Redistribución de Rutas - Configuración Completa

### **Redistribución OSPF ↔ RIP**
```bash
!De OSPF a RIP, en router OSPF
enable
conf t
router ospf 1
redistribute rip metric 10 subnets
exit
! EN Router de OSPF
enable
conf t
router rip
version 2
! Esta es la ruta por la que se comunican
network 10.0.4.20
```

### **Redistribución OSPF ↔ EIGRP**
```bash
!DE OSPF a EIGRP
enable
conf t
router ospf 1
redistribute eigrp 1 metric 10 subnets
exit
router eigrp 1
network 10.0.4.8 0.0.0.3

!De EIGRP a OSPF EN R2
enable
conf t
router eigrp 1
redistribute ospf 1 metric 10000 100 255 1 1500
no auto-summary
exit
router ospf 1
network 10.0.4.8 0.0.0.3 area 0
```


---

## Comandos de Verificación

### **Verificar Estado de Protocolos**
```bash
# OSPF
show ip ospf neighbor
show ip ospf database
show ip ospf interface brief

# EIGRP
show ip eigrp neighbors
show ip eigrp topology
show ip eigrp interfaces

# RIP
show ip rip database
show ip protocols

# General
show ip route
show ip route ospf
show ip route eigrp
show ip route rip
```

---

## Mejores Prácticas

1. **Planificación de Áreas OSPF**: Usar área 0 como backbone
2. **Métricas Consistentes**: Configurar métricas apropiadas en redistribución
3. **Filtrado**: Implementar route-maps para controlar redistribución
4. **Autenticación**: Configurar autenticación MD5 entre vecinos
5. **Passive Interfaces**: Marcar interfaces hacia usuarios finales como pasivas
6. **Summarización**: Implementar summarización donde sea posible
7. **Logging**: Habilitar logging de cambios de ruteo

---

## Troubleshooting Básico

### **Problemas Comunes:**
- **Vecinos no formados**: Verificar configuraciones de red/máscara
- **Rutas no redistribuidas**: Verificar métricas y route-maps
- **Loops de ruteo**: Verificar filtros y tags
- **Convergencia lenta**: Verificar timers y hello intervals

---

*Tutor: Allan Gómez*  
*Clase 9 - Enrutamiento Dinámico*  
*Octubre 2025*

