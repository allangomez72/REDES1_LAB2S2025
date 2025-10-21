# Clase 11 - ACL's

## ¿QUÉ SON LAS ACLs?
Las **Listas de Control de Acceso (ACLs)** son el **firewall básico** de los routers Cisco. Son **reglas de filtrado** que deciden qué tráfico puede pasar y qué tráfico se bloquea.

---

## ACL ESTÁNDAR vs EXTENDIDA

### ACL ESTÁNDAR - "GUARDIA SIMPLE"
- **Solo mira la DIRECCIÓN DE ORIGEN**
- Como un guardia que solo pregunta "¿De dónde vienes?"
- **Menos precisa** - bloquea/permite por fuente solamente

**Ejemplo real:**
## ACL ESTÁNDAR (1-99)

### Sintaxis:
```
access-list [número] [permit|deny] [origen] [wildcard]
```

### Ejemplos prácticos:

**Permitir red específica:**
```
access-list 10 permit 192.168.10.0 0.0.0.255
```

**Permitir host específico:**
```
access-list 10 permit host 192.168.10.5
```

**Denegar red y permitir resto:**
```
access-list 10 deny 192.168.20.0 0.0.0.255
access-list 10 permit any
```

**Aplicar en interfaz:**
```
interface FastEthernet0/0
ip access-group 10 in
```
---
> "Solo deja pasar a quienes vengan del vecindario 192.168.10.x"

### ACL EXTENDIDA - "GUARDIA DETALLADO"
- **Mira ORIGEN, DESTINO y PROTOCOLO**
- Como un guardia que pregunta: "¿De dónde vienes?, ¿A dónde vas?, ¿Qué llevas?"
- **Muy precisa** - control total del tráfico

**Ejemplo real:**
## ACL EXTENDIDA (100-199)

### Sintaxis básica:
```
access-list [número] [permit|deny] [protocolo] [origen] [wildcard] [destino] [wildcard] [opciones]
```

### Ejemplo - Firewall entre redes:
```
! PERMITIR comunicación BIDIRECCIONAL entre redes específicas
access-list 100 permit ip 192.168.100.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 100 permit ip 192.168.40.0 0.0.0.255 192.168.100.0 0.0.0.255

access-list 100 permit ip 192.168.110.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 100 permit ip 192.168.40.0 0.0.0.255 192.168.110.0 0.0.0.255

! DENEGAR todo lo demás (EXPLÍCITO)
access-list 100 deny ip any any

! Aplicar en interfaz de salida
interface Serial2/0
ip access-group 100 out
```
> "Solo deja pasar a la familia 192.168.100.x que quiera visitar 192.168.40.x y solo si van al puerto 80 (web)"
>[!NOTE]
> Revisar el video de ejemplo [Ejemplo ACL's](https://drive.google.com/file/d/1V4ocig3U_zhGT2FqGxP-CiIB2KeY9cnr/view?usp=sharing)

---

## LA IMPORTANCIA COMO FIREWALL

### ¿POR QUÉ SON TU PRIMERA LÍNEA DE DEFENSA?

1. **SEGURIDAD BÁSICA**: Sin ACLs, todas tus redes se comunican libremente
2. **CONTROL DE ACCESO**: Decides qué redes pueden "hablar" entre sí
3. **PREVENCIÓN DE ATAQUES**: Bloqueas tráfico malicioso entre segmentos
4. **CUMPLIMIENTO DE POLÍTICAS**: Implementas reglas de seguridad de la empresa

### EJEMPLO DE FIREWALL EN LA VIDA REAL:

**Sin ACL (PELIGROSO):**
```
! Todas las redes se comunican libremente
! Cualquier empleado puede acceder a servidores críticos
! No hay seguridad entre departamentos
```

**Con ACL (SEGURO):**
```
! Solo IT puede administrar servidores
access-list 101 permit tcp 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255 eq 22

! Solo Ventas puede acceder a CRM
access-list 101 permit tcp 192.168.20.0 0.0.0.255 host 192.168.40.50 eq 443

! Bloquear todo lo demás entre redes
access-list 101 deny ip any any
```

---

## EL GRAN ERROR: ORDEN INCORRECTO

### ANALICEMOS EL PROBLEMA:

**CONFIGURACIÓN ERRÓNEA:**
```
access-list 100 permit ip 192.168.100.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 100 deny ip any any
access-list 100 permit ip 192.168.110.0 0.0.0.255 192.168.40.0 0.0.0.255
```

### ¿QUÉ PASA PAQUETE POR PAQUETE?

1. **Paquete de 192.168.100.5 → 192.168.40.10**
   - ✅ Coincide con regla 1 → **PERMITIDO**

2. **Paquete de 192.168.50.8 → 192.168.40.20**
   - ❌ No coincide con regla 1
   - ✅ Coincide con regla 2 (`any any`) → **DENEGADO**

3. **Paquete de 192.168.110.15 → 192.168.40.30**
   - ❌ No coincide con regla 1
   - ✅ Coincide con regla 2 (`any any`) → **DENEGADO**
   - ❌ **NUNCA LLEGA** a la regla 3

### PUNTO CLAVE:
**Las ACLs se evalúan en ORDEN SECUENCIAL**. Cuando un paquete coincide con una regla, **SE DETIENE LA EVALUACIÓN**. El `deny any any` actúa como un **MURO INFRANQUEABLE** para todas las reglas posteriores.

---

## LA SOLUCIÓN CORRECTA

**CONFIGURACIÓN CORRECTA:**
```
! PRIMERO - Todas las reglas PERMIT específicas
access-list 100 permit ip 192.168.100.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 100 permit ip 192.168.110.0 0.0.0.255 192.168.40.0 0.0.0.255

! ÚLTIMO - El "deny any any" como GUARDIÁN FINAL
access-list 100 deny ip any any
```

### FLUJO CORRECTO:

1. **Paquete de 192.168.100.5 → 192.168.40.10**
   - ✅ Regla 1 → **PERMITIDO**

2. **Paquete de 192.168.110.15 → 192.168.40.30**
   - ❌ No coincide con regla 1
   - ✅ Regla 2 → **PERMITIDO**

3. **Paquete de 192.168.50.8 → 192.168.40.20**
   - ❌ No coincide con regla 1
   - ❌ No coincide con regla 2  
   - ✅ Regla 3 → **DENEGADO**

---

## ¿POR QUÉ el `deny ip any any` EXPLÍCITO?

### EL SECRETO: "DENY IMPLÍCITO"
Todas las ACLs tienen un **DENY ANY ANY INVISIBLE** al final. Es como si Cisco dijera: "Por seguridad, si no permites algo explícitamente, lo bloqueo".

**CON IMPLÍCITO (AUTOMÁTICO):**
```
access-list 100 permit ip 192.168.100.0 0.0.0.255 192.168.40.0 0.0.0.255
! deny ip any any ← INVISIBLE, no tiene contadores
```

**CON EXPLÍCITO (RECOMENDADO):**
```
access-list 100 permit ip 192.168.100.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 100 deny ip any any ← VISIBLE, con contadores
```

### VENTAJAS DEL EXPLÍCITO:

1. **VISIBILIDAD**: 
   ```
   show access-lists
   Extended IP access list 100
       10 permit ip 192.168.100.0 0.0.0.255 192.168.40.0 0.0.0.255 (5 matches)
       20 deny ip any any (150 matches) ← ¡VES EL ATAQUE!
   ```

2. **DOCUMENTACIÓN**: Otros administradores ven tu intención de seguridad

3. **DEPURACIÓN**: Sabes exactamente qué está siendo bloqueado

4. **AUDITORÍA**: Cumples con políticas de "denegar por defecto"

---

*Tutor: Allan Gómez*  
*Clase 11 - ACL's*  
*Octubre 2025*

