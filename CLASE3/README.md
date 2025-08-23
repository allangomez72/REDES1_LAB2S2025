
# EJEMPLO CONFIGURACION BASICA DE UN SWITCH, ARP E ICMP

Comandos usados:

```SQL
#Entrar al usuario privilegiado 
enable
#Configuracion global
configure terminal
no ip domain-lookup
hostname redes1

enable password ejemplo1
enable secret ejemplosecret

#para telnet
line vty 0 4
login
password telnet1

line console 0
password consola
login

#poner un mensaje
banner motd # mensaje de ejemplo #


#comandos extra
terminal history size 15

#ver reloj
show Clock

#ver ultimas actividades

show run

#guardar la configuracion
write memory
```

> [!NOTE]
> De preferencia no estar copiando y pengando comandos mejor si los escriben, para que luego no se les vaya a estar olvidando.