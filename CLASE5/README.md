# Clase 5 - Laboratorio Redes de Computadoras 1

## Identificar el switch ROOT
Para identificar cual es el switch que tien el rol del ROOT BRIDGE, se mencionarion dos formas una con el comando:
`show spanning-tree`

![Spanning-tree-protocol](/CLASE5//assets/image1.png)

O de manera visual, ojo que de manera visual se debe de tener en cuenta que todas las interfases deben de estar activas (triangulos en verde):

![Spanning-tree](/CLASE5/assets/image2.png)

Cabe destacar que si bien ya la misma topologia elije a quien va ser su root bridge tambien se puede modificar y elejir uno de manera manual, que es quien va estar manejando el trafico de las VLANS, el comando es `spanning-tree vlan # root primary` donde se hacer referencia a las VLANs que va estar manejando

### Modos de Configuración de Spanning Tree Protocol (STP)
STP (Spanning Tree Protocol) es un protocolo utilizado en redes de switches para evitar bucles de conmutación. Existen diferentes versiones y modos de configuración de STP, entre ellos:

1. Modo por Defecto (PVST o PVST+)

El modo predeterminado en switches Cisco es Per VLAN Spanning Tree Plus (PVST+). Este modo crea una instancia independiente de STP para cada VLAN, lo que permite una mejor optimización del tráfico y la redundancia en la red.

Para la verficación usamos `show spanning-tree`

2. Modo Rapid PVST (Spanning-Tree Mode Rapid-PVST)

Este modo utiliza Rapid Per VLAN Spanning Tree (Rapid PVST+), basado en el estándar IEEE 802.1w, mejorando la velocidad de convergencia de STP y reduciendo el tiempo de recuperación en caso de fallos.

Para configurar usamos `spanning-tree mode rapid-pvst`


Recordar que para la confituracion de un switch de capa tres para los enlaces troncales la diferencia es este comando:`switchport trunk encapsulation dot1q`
las demas configuraciones son iguales


## Configuración de SW Capa 3 - Servidor

```bash
vtp domain MI_RED
vtp mode server
vtp version 2  
vtp password claveVTP 

vlan 10
 name Ventas
vlan 20
 name IT
vlan 30
 name Finanzas

interface range fa0/1 - 2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30

spanning-tree vlan 10,20,30 root primary

```

## Configuración SW Capa 3 - Cliente

```bash
vtp domain MI_RED
vtp mode client
vtp password claveVTP

interface range fa0/1-2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 10,20,30
 
 spanning-tree vlan 10,20,30
 
 interface fa0/3
 switchport mode access
 switchport access vlan 10

```