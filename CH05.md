## **Capítulo 5 – VLAN Trunks and EtherChannel Bundles**

### **1. Introducción**

Las VLANs permiten segmentar una red física en múltiples dominios de broadcast.  
Cuando se necesita transportar varias VLANs a través de un enlace entre switches, se utiliza un **trunk**.  
Cuando se requiere **mayor ancho de banda y redundancia**, varios enlaces físicos se combinan en un solo enlace lógico mediante **EtherChannel**.

---

## **VLAN Trunks**

### **2. Concepto de trunk**

Un **trunk** permite que múltiples VLANs viajen sobre un solo enlace físico, utilizando etiquetas para distinguir a qué VLAN pertenece cada trama.  
El estándar más común es **IEEE 802.1Q** (dot1q).

**Características principales:**

- Usa **etiquetado (tagging)** de tramas Ethernet con un campo VLAN ID (12 bits, rango 1–4094).
    
- Una VLAN puede designarse como **VLAN nativa**, cuyas tramas **no se etiquetan**.
    
- Los puertos trunk se utilizan entre **switches, routers y puntos de acceso.**
    

---

### **3. Configuración de trunks**

**Ejemplo:**

```
interface GigabitEthernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 10,20
```

**Verificación:**

`show interface trunk`

**Salida típica:**

```
Port     Mode   Encapsulation  Status   Native vlan
Gi0/3    on     802.1q         trunking 100
Vlans allowed on trunk: 10,20
```

---

### **4. Dynamic Trunking Protocol (DTP)**

**DTP** es un protocolo propietario de Cisco que negocia automáticamente la creación de trunks.

**Modos DTP:**

|Modo|Descripción|Resultado con otro puerto|
|---|---|---|
|**dynamic desirable**|Inicia activamente la negociación.|Forma trunk con _trunk_, _desirable_ o _auto_.|
|**dynamic auto**|Espera negociación del otro lado.|Solo forma trunk con _desirable_ o _trunk_.|
|**trunk**|Fuerza el modo trunk sin negociar.|Siempre trunk.|
|**access**|Fuerza el modo access (sin trunk).|Nunca forma trunk.|
|**nonegotiate**|Desactiva DTP.|Requiere configuración manual en ambos lados.|

**Ejemplo:**

```
interface Gi0/2
 switchport mode dynamic desirable
```

---

### **5. VLAN Trunking Protocol (VTP)**

**VTP** distribuye automáticamente la información de VLANs entre switches Cisco en el mismo dominio.

- **VTPv1/v2:** Solo distribuye VLANs estándar (1–1005).
    
- **VTPv3:** Añade soporte para VLANs extendidas y **propagación de la base MST (Multiple Spanning Tree)**.
    

**Comando clave:**

```
vtp version 3
```

---

## **EtherChannel Bundles**

### **6. Concepto**

**EtherChannel** (o _Port Channel_) combina varios enlaces físicos en un único enlace lógico, incrementando el ancho de banda y proporcionando redundancia automática.

**Beneficios:**

- **Optimiza el uso del ancho de banda.**
    
- **Acelera la convergencia.**
    
- **Reduce la dependencia de STP.**
    
- **Proporciona tolerancia a fallos físicos.**
    

**Limitación:** hasta **8 enlaces** por EtherChannel.

**Tipos de EtherChannel:**

- **Layer 2:** agrupa puertos de switch (access o trunk).
    
- **Layer 3:** agrupa puertos configurados como **routed ports** (`no switchport`).
    

---

### **7. Protocolos de negociación**

|Protocolo|Estándar|Modos|Compatibilidad|
|---|---|---|---|
|**PAgP**|Cisco propietario|_auto_, _desirable_|Solo Cisco|
|**LACP**|IEEE 802.1AX|_active_, _passive_|Multivendor|
|**ON**|Manual|No negocia|Ambos lados iguales|

---

### **8. LACP (Link Aggregation Control Protocol)**

- **Estándar IEEE 802.1AX (anteriormente 802.3ad).**
    
- Administra la creación y mantenimiento del grupo de enlaces.
    
- Verifica consistencia de configuración (velocidad, dúplex, VLAN, etc.).
    

**Modos:**

|LACP mode|Descripción|Compatibilidad|
|---|---|---|
|**active**|Inicia negociación LACP.|Con _active_ o _passive_.|
|**passive**|Espera LACP del otro extremo.|Solo con _active_.|

**Ejemplo (Layer 2):**

```
interface range Gi0/0-1
 channel-group 1 mode active
```

**Verificación:**

`show etherchannel summary`

**Salida:**

```
Group  Port-channel  Protocol  Ports
1      Po1(SU)       LACP      Gi0/0(P) Gi0/1(P)
```

---

### **9. PAgP (Port Aggregation Protocol)**

- **Propietario de Cisco.**
    
- Negocia automáticamente los enlaces EtherChannel entre switches Cisco.
    

**Modos:**

|Modo|Descripción|Compatible con|
|---|---|---|
|**desirable**|Inicia negociación.|_auto_, _desirable_|
|**auto**|Espera negociación.|_desirable_|
|**on**|Sin negociación.|_on_|

**Ejemplo:**

```
interface range Gi0/0-1
 channel-group 2 mode desirable
```

---

### **10. Configuración Layer 3 EtherChannel**

Se crea un _port-channel_ en modo enrutado:

```
interface range Gi0/2-3
 no switchport
 channel-group 3 mode active
interface Port-channel3
 ip address 10.10.10.1 255.255.255.0
```

**Verificación:**

```
show interface port-channel 3
```

---

### **11. Métodos de balanceo de carga**

El switch distribuye el tráfico en los enlaces activos usando algoritmos hash basados en dirección MAC, IP o puerto.

**Métodos disponibles:**

- `src-mac`, `dst-mac`
    
- `src-ip`, `dst-ip`
    
- `src-dst-mac`, `src-dst-ip`
    
- `src-port`, `dst-port`
    

**Configuración:**

```
port-channel load-balance src-dst-ip
```

**Verificación:**

```
show etherchannel load-balance
```

---

### **12. Comandos de verificación**

```
show etherchannel summary
show etherchannel port-channel
show interface trunk
show interfaces switchport
show interfaces port-channel
```

---

### **13. Consideraciones y mejores prácticas**

- Todos los puertos del canal deben tener la **misma configuración** (velocidad, dúplex, VLANs, modo).
    
- No mezclar protocolos (ej. LACP con PAgP).
    
- Configurar **mismo número de grupo** en ambos extremos.
    
- Activar **Spanning Tree PortFast** solo en interfaces de acceso (no en EtherChannel troncales).
    
- EtherChannel ayuda a **reducir puertos bloqueados por STP**.