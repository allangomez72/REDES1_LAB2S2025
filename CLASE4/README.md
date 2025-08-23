# Clase 4 - Laboratorio Redes de Computadoras 1

## Configuracion de VLANs
```sh
# Creación de VLANs
enable
configure terminal
vlan 10
name Administracion
vlan 20
name Ventas
vlan 30
name Invitados
exit
wr
```

## Configuracion de VTP

#### Modo servidor (server)
```sh
enable
configure terminal
hostname SW_SERVIDOR
vtp mode server
vtp domain redes_lab
vtp password clave123
exit
wr

```

#### Modo Cliente (client)
```sh
enable
configure terminal
hostname SW_CLIENTE
vtp mode client
vtp domain redes_lab
vtp password clave123
exit
wr

```

#### Modo Transparente (transparent)
```sh
enable
configure terminal
hostname SW_TRANSPARENTE
vtp mode transparent
exit
wr

```

## Configuracion de los enlaces troncales y los puertos de acceso

#### Modo trunk
```sh
enable
configure terminal
interface f0/1
switchport mode trunk

#Este sirve solo para dejar pasar las vlans que uno necesite en lugar de propagar todas
switchport trunk allowed vlan 10,20,30

exit
wr

```

#### Modo access
```sh
enable
configure terminal
interface f0/2
switchport mode access
switchport access vlan 10
exit
wr

```

### Ejemplo 
```sh
enable
configure terminal
hostname SW1

# VTP en modo servidor
vtp mode server
vtp domain redes_lab
vtp password clave123

# Creación de VLANs
vlan 10
name Administracion
vlan 20
name Ventas
exit

# Configuración de troncal en F0/1
interface f0/1
switchport mode trunk
exit

# Configuración de puertos en modo access
interface f0/2
switchport mode access
switchport access vlan 10
exit

interface f0/3
switchport mode access
switchport access vlan 20
exit

wr

```