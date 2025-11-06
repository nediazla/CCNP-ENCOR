## **Capítulo 2 – Spanning Tree Protocol (STP)**

### **1. Propósito del STP**

El **Spanning Tree Protocol (STP)** previene bucles de capa 2 en redes con caminos redundantes.  
Opera según el estándar **IEEE 802.1D**, intercambiando mensajes llamados **BPDUs (Bridge Protocol Data Units)** entre switches para identificar y bloquear enlaces redundantes, manteniendo **una sola ruta activa** entre dos dispositivos.

Si no existiera STP, los bucles provocarían:

- **Tormentas de broadcast** (broadcast storms).
    
- **Inestabilidad en la tabla MAC** (aprendizaje de direcciones por múltiples puertos).
    
- **Duplicación de tramas** y congestión.
    

---

### **2. Tipos de BPDUs**

STP utiliza tres tipos de mensajes:

- **Configuration BPDU:** informa la identidad del root bridge y las funciones de los puertos (root, designated, blocking).
    
- **Topology Change Notification (TCN):** notifica cambios en la topología (fallos o puertos que cambian de estado).
    
- **Topology Change Acknowledgement (TCA):** confirma la recepción de una notificación TCN.
    

Las BPDUs se envían cada **2 segundos** por defecto.

---

### **3. Elección del Root Bridge**

- Cada switch tiene un **Bridge ID (BID)** compuesto por:
    
    - Prioridad (por defecto **32768**)
        
    - Dirección MAC
        
    - VLAN ID (System ID Extension)
        
- El **switch con el menor Bridge ID** es elegido como **Root Bridge**.
    
- Si todos los switches tienen la misma prioridad, gana el de **menor MAC**.
    

**Comando:**

```
spanning-tree vlan 1 priority 4096
spanning-tree vlan 1 root primary
```

---

### **4. Roles de puerto**

Cada switch (excepto el root) determina qué puerto cumple cada rol:

|Rol de puerto|Función|Descripción|
|---|---|---|
|**Root Port (RP)**|Hacia el Root Bridge|Mejor camino al root (menor costo).|
|**Designated Port (DP)**|Desde el segmento|Reenvía tráfico hacia otros switches.|
|**Non-Designated Port (Blocking)**|Redundante|Bloquea tráfico para prevenir loops.|

**Criterios de desempate:**

1. Menor **costo** hacia el root.
    
2. Menor **Bridge ID** del switch remoto.
    
3. Menor **Port ID**.
    

---

### **5. Estados de los puertos**

Durante la convergencia, los puertos pasan por varios estados:

|Estado|Descripción|
|---|---|
|**Disabled**|Administrativamente apagado.|
|**Blocking**|Recibe BPDUs pero no reenvía tramas.|
|**Listening**|Evalúa si debe participar en el reenvío.|
|**Learning**|Aprende direcciones MAC pero no reenvía datos.|
|**Forwarding**|Envía y recibe tramas de datos.|

---

### **6. Timers de STP**

Configurables en el **Root Bridge**:

|Temporizador|Valor por defecto|Descripción|
|---|---|---|
|**Hello Time**|2 s|Intervalo entre BPDUs.|
|**Forward Delay**|15 s|Duración de Listening + Learning.|
|**Max Age**|20 s|Tiempo máximo antes de descartar info BPDU.|

**Comandos:**

```
spanning-tree vlan 1 hello-time 2
spanning-tree vlan 1 forward-time 15
spanning-tree vlan 1 max-age 20
```

---

### **7. Rapid Spanning Tree Protocol (RSTP – IEEE 802.1w)**

RSTP mejora la convergencia de STP tradicional (de ~50s a <2s) al introducir **roles adicionales**:

|Rol nuevo|Descripción|
|---|---|
|**Alternate Port**|Camino de respaldo al Root Bridge.|
|**Backup Port**|Respaldo de un puerto designado en el mismo segmento.|

- Estados reducidos a **Discarding**, **Learning**, **Forwarding**.
    
- Compatible con STP 802.1D.
    
- En Cisco, implementado como **Rapid-PVST+** (una instancia por VLAN).
    

**Comando:**

```
spanning-tree mode rapid-pvst
```

---

### **8. PVST+ y MST**

- **PVST+ (Per-VLAN Spanning Tree Plus):** una instancia por VLAN (Cisco propietario).
    
- **MST (Multiple Spanning Tree Protocol, IEEE 802.1s):**
    
    - Agrupa múltiples VLANs bajo una sola instancia de STP.
        
    - Reduce carga de CPU y BPDUs.
        
    - Usa tres instancias:
        
        - **IST (Instance 0):** Internal Spanning Tree.
            
        - **CST:** Common Spanning Tree (interoperabilidad).
            
        - **MSTI:** Multiple Spanning Tree Instances.
            

**Comando:**

```
spanning-tree mode mst
spanning-tree mst configuration
```

---

### **9. Optimización y protección avanzada (Spanning Tree Tuning)**

**a. PortFast**  
Permite que un puerto de acceso pase directamente a _forwarding_, evitando el retardo de los estados Listening/Learning.  
Uso: en puertos conectados a hosts.

`spanning-tree portfast`

**b. BPDU Guard**  
Desactiva un puerto si recibe una BPDU (previene loops).

`spanning-tree bpduguard enable`

**c. Root Guard**  
Evita que un puerto no autorizado se convierta en Root Port.

`spanning-tree guard root`

**d. Loop Guard**  
Previene que puertos alternativos se activen indebidamente al dejar de recibir BPDUs.

`spanning-tree guard loop`

**e. BPDU Filter**  
Bloquea el envío y recepción de BPDUs en puertos específicos (usado solo en entornos controlados).

`spanning-tree bpdufilter enable`

**f. Bridge Assurance**  
Detecta fallos unidireccionales entre switches vecinos en enlaces de capa 2 troncales.

`spanning-tree bridge assurance`

---

### **10. Verificación**

Comandos esenciales de verificación:

```
show spanning-tree
show spanning-tree vlan 10
show spanning-tree summary
show spanning-tree mst configuration
show spanning-tree mst
```