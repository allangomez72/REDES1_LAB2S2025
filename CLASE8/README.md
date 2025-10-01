# Clase 8 - Enrutamiento Estatico

>[!WARNING]
> El archivo PKT se va a subir luego de los 15 días.

## ¿Qué es el enrutamiento estático?

El **enrutamiento estático** es un tipo de configuración manual en los routers, donde el administrador define las rutas que debe seguir el tráfico para llegar a redes específicas. A diferencia del enrutamiento dinámico, las rutas **no se actualizan automáticamente**; cualquier cambio en la topología requiere intervención manual.

---

## Ventajas del enrutamiento estático

* Menor uso de CPU y ancho de banda.
* Mayor seguridad (no intercambia rutas con otros routers).
* Control total del camino que toma el tráfico.

---

## Desventajas

* Poco escalable en redes grandes.
* Requiere configuración manual.
* No se adapta a cambios automáticamente (fallas, nuevas rutas, etc.).

---

Configuración de los routers para asignación de IP's
```bash
enable
conf t
no ip domain-lookup
hostname ROUTER1

interface fa0/0
ip address 10.0.0.1 255.255.255.252
no shut
exit
interface fa1/0
ip address 10.0.0.2 255.255.255.252
no shut
exit
interface fa6/0
ip address 192.168.25.1 255.255.255.0
no shut
end
wr
```

## Sintaxis de enrutamiento estático

```bash
ip route <red_destino> <máscara_subred> <ip_vecina>
```

* **Red destino:** Dirección de red a la que queremos llegar.
* **Máscara de subred:** Máscara correspondiente a la red destino.
* **IP vecina:** IP del siguiente salto (router vecino) que llevará el tráfico hacia esa red.

---

## Ejemplo aplicado (Router 3)

Supongamos que el Router3 necesita llegar a dos redes:

* Red: `192.168.25.0/29` → máscara `255.255.255.248`
* Red: `192.168.25.8/29` → máscara `255.255.255.248`

Se agregan las rutas así:

```bash
ip route 192.168.25.0 255.255.255.248 10.0.0.5
ip route 192.168.25.8 255.255.255.248 10.0.0.9
```

Esto indica que para llegar a esas redes, el Router3 enviará los paquetes hacia las IPs vecinas `10.0.0.5` y `10.0.0.9`.

---

## Tabla CIDR (de /24 a /32)

| CIDR | Máscara decimal | Nº de hosts            | Rango de hosts      | Broadcast           |
| ---- | --------------- | ---------------------- | ------------------- | ------------------- |
| /24  | 255.255.255.0   | 254                    | x.x.x.1 – x.x.x.254 | x.x.x.255           |
| /25  | 255.255.255.128 | 126                    | x.x.x.1 – x.x.x.126 | x.x.x.127           |
| /26  | 255.255.255.192 | 62                     | x.x.x.1 – x.x.x.62  | x.x.x.63            |
| /27  | 255.255.255.224 | 30                     | x.x.x.1 – x.x.x.30  | x.x.x.31            |
| /28  | 255.255.255.240 | 14                     | x.x.x.1 – x.x.x.14  | x.x.x.15            |
| /29  | 255.255.255.248 | 6                      | x.x.x.1 – x.x.x.6   | x.x.x.7             |
| /30  | 255.255.255.252 | 2                      | x.x.x.1 – x.x.x.2   | x.x.x.3             |
| /31  | 255.255.255.254 | 0 (solo punto a punto) | x.x.x.0 – x.x.x.1   | No se usa broadcast |
| /32  | 255.255.255.255 | 1 (1 host)             | Solo una IP         | No aplica           |

> Nota: /31 y /32 tienen usos especiales:
>
> * **/31:** Ideal para enlaces punto a punto sin necesidad de direcciones de red y broadcast.
> * **/32:** Se usa para identificar dispositivos específicos (como loopbacks o rutas host).

---

## Recomendaciones

* Usa /30 para enlaces entre routers (punto a punto) → solo necesitas 2 IPs.
* Siempre verifica conectividad con `ping` y tabla de enrutamiento con `show ip route`.
* Guarda cambios con `wr` o `copy running-config startup-config`.


---

*Tutor: Allan Gómez*  
*Clase 8 - Enrutamiento Estático*  
*Octubre 2025*

