# Clase 10 - Protocolos de Redundancia

## Introducción a los Protocolos de Redundancia

### **¿Qué son HSRP y VRRP?**
Son protocolos que permiten crear un **gateway virtual** compartido entre múltiples routers, proporcionando alta disponibilidad para hosts de red.

---

## HSRP (Hot Standby Router Protocol)

### **Conceptos Clave de HSRP**

| Término | Descripción |
|---------|-------------|
| **Virtual IP** | IP que comparten los routers del grupo |
| **Virtual MAC** | MAC address virtual: `0000.0c07.acXX` |
| **Active Router** | Router que actualmente maneja el tráfico |
| **Standby Router** | Router de respaldo listo para tomar el control |
| **Priority** | Valor que determina qué router es Active (por defecto 100) |
| **Preempt** | Permite retomar el rol Active si tiene mayor prioridad |

### ** Estados de HSRP**
1. **Initial** - Estado inicial
2. **Learn** - Aprendiendo Virtual IP
3. **Listen** - Escuchando mensajes Hello
4. **Speak** - Enviando mensajes Hello
5. **Standby** - Listo para tomar el control
6. **Active** - Procesando tráfico

---

## Configuración Detallada de HSRP

### **Configuración Básica HSRP**
```bash
interface <interface>
 ip address <real-ip> <mask>
 standby <group> ip <virtual-ip>
 standby <group> priority <priority>
 standby <group> preempt
```

### **Ejemplo Práctico - Router 0 (Active)**
```bash
enable
configure terminal

! Configurar interfaz física
interface gigabitEthernet0/0
 no shutdown
 exit

! Subinterfaz para VLAN 14
interface gigabitEthernet0/0.14
 description VLAN 14 - Users
 encapsulation dot1Q 14
 ip address 192.168.4.194 255.255.255.240
 no shutdown
 
 ! Configuración HSRP Grupo 1
 standby 1 ip 192.168.4.193      ! IP Virtual del Gateway
 standby 1 priority 150          ! Prioridad mayor = Active
 standby 1 preempt               ! Retomar control si es necesario
 standby 1 name HSRP-GROUP-1     ! Nombre descriptivo (opcional)
 exit

! Subinterfaz para VLAN 24
interface gigabitEthernet0/0.24
 description VLAN 24 - Servers
 encapsulation dot1Q 24
 ip address 192.168.4.130 255.255.255.192
 no shutdown
 
 ! Configuración HSRP Grupo 2
 standby 2 ip 192.168.4.129      ! IP Virtual diferente para VLAN 24
 standby 2 priority 150          ! Mismo router activo para ambas VLANs
 standby 2 preempt
 standby 2 name HSRP-GROUP-2
 exit

! Configurar interfaz de enrutamiento
interface fastEthernet0/0/0
 description Enlace al Core Network
 ip address 10.0.4.22 255.255.255.252
 no shutdown
 exit

! Configurar OSPF para anunciar las redes
router ospf 1
 network 10.0.4.20 0.0.0.3 area 0
 network 192.168.4.128 0.0.0.63 area 0    ! VLAN 24
 network 192.168.4.192 0.0.0.15 area 0     ! VLAN 14
 exit

write memory
```

### **Router 1 (Standby) - Configuración Complementaria**
```bash
enable
configure terminal

interface gigabitEthernet0/0
 no shutdown
 exit

! VLAN 14 - Configuración Standby
interface gigabitEthernet0/0.14
 encapsulation dot1Q 14
 ip address 192.168.4.195 255.255.255.240  ! IP diferente misma red
 no shutdown
 
 standby 1 ip 192.168.4.193      ! Misma IP Virtual
 standby 1 priority 100          ! Prioridad menor = Standby
 standby 1 preempt               ! También puede retomar si es necesario
 exit

! VLAN 24 - Configuración Standby
interface gigabitEthernet0/0.24
 encapsulation dot1Q 24
 ip address 192.168.4.131 255.255.255.192
 no shutdown
 
 standby 2 ip 192.168.4.129
 standby 2 priority 100
 standby 2 preempt
 exit

! Configurar interfaz de enrutamiento (diferente al Router 0)
interface fastEthernet0/0/1
 description Enlace alternativo al Core
 ip address 10.0.4.26 255.255.255.252
 no shutdown
 exit

! OSPF Configuration
router ospf 1
 network 10.0.4.24 0.0.0.3 area 0
 network 192.168.4.128 0.0.0.63 area 0
 network 192.168.4.192 0.0.0.15 area 0
 exit

write memory
```

---

### **1. Múltiples Grupos HSRP (Load Balancing)**
```bash
! Router A - Active para Grupo 1, Standby para Grupo 2
interface vlan10
 ip address 192.168.10.2 255.255.255.0
 standby 1 ip 192.168.10.1
 standby 1 priority 150
 standby 1 preempt
 
 standby 2 ip 192.168.10.254
 standby 2 priority 100
 standby 2 preempt

! Router B - Active para Grupo 2, Standby para Grupo 1  
interface vlan10
 ip address 192.168.10.3 255.255.255.0
 standby 1 ip 192.168.10.1
 standby 1 priority 100
 standby 1 preempt
 
 standby 2 ip 192.168.10.254
 standby 2 priority 150
 standby 2 preempt
```

---

## VRRP (Virtual Router Redundancy Protocol)

### **Comparación HSRP vs VRRP**

| Característica | HSRP | VRRP |
|----------------|------|------|
| **Estándar** | Propietario Cisco | Estándar abierto (RFC 5798) |
| **Virtual MAC** | 0000.0c07.acXX | 0000.5e00.01XX |
| **Grupos** | 0-255 | 1-255 |
| **Prioridad** | 0-255 | 1-254 |
| **Preempt** | Configurable | Habilitado por defecto |

### **Configuración Básica VRRP**
```bash
interface <interface>
 ip address <real-ip> <mask>
 vrrp <group> ip <virtual-ip>
 vrrp <group> priority <priority>
```

### **Ejemplo VRRP - Router Master**
```bash
enable
configure terminal

interface gigabitEthernet0/0.14
 encapsulation dot1Q 14
 ip address 192.168.4.194 255.255.255.240
 
 ! Configuración VRRP
 vrrp 1 ip 192.168.4.193        ! IP Virtual
 vrrp 1 priority 150            ! Master Router
 vrrp 1 preempt                 ! Retomar control (default enabled)
 vrrp 1 authentication text MyVRRPPass
 exit

interface gigabitEthernet0/0.24
 encapsulation dot1Q 24
 ip address 192.168.4.130 255.255.255.192
 
 vrrp 2 ip 192.168.4.129
 vrrp 2 priority 150
 vrrp 2 preempt
 exit
```

### **Router Backup - VRRP**
```bash
interface gigabitEthernet0/0.14
 encapsulation dot1Q 14
 ip address 192.168.4.195 255.255.255.240
 
 vrrp 1 ip 192.168.4.193
 vrrp 1 priority 100            ! Backup Router
 vrrp 1 preempt
 exit
```

---

## Comandos de Verificación y Troubleshooting

### **Comandos HSRP**
```bash
# Ver estado de grupos HSRP
show standby
show standby brief
show standby all

# Ver detalles específicos
show standby interface gigabitEthernet0/0.14
show standby capabilities

# Debugging (usar con cuidado)
debug standby events
debug standby packets
```

### **Comandos VRRP**
```bash
# Ver estado VRRP
show vrrp
show vrrp brief
show vrrp all

# Detalles por interfaz
show vrrp interface gigabitEthernet0/0.14
```

---

## Troubleshooting Guiado

### **Verificaciones**

1. **Verificar configuraciones HSRP**
   ```bash
   show standby
   show running-config | section standby
   ```

2. **Verificar que IP virtual esté en misma subred**
   ```bash
   show ip interface brief
   ```

3. **Verificar preempt y prioridades**
   ```bash
   show standby detail
   ```

---

*Tutor: Allan Gómez*  
*Clase 10 - Protocolos de Redundancia HSRP*  
*Octubre 2025*

