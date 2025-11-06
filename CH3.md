## **Capítulo 3 – Advanced Spanning Tree Tuning**

### **1. Propósito**

El ajuste avanzado de STP (_Spanning Tree Protocol Tuning_) mejora la estabilidad, la convergencia y la seguridad de la topología de capa 2, reduciendo el riesgo de bucles y optimizando el tiempo de recuperación ante fallos.

Cisco ofrece múltiples mecanismos adicionales a STP estándar, entre ellos:  
**PortFast, BPDU Guard, BPDU Filter, Root Guard, Loop Guard, Bridge Assurance y UDLD.**

---

### **2. PortFast**

Permite que un **puerto de acceso** pase directamente al estado _forwarding_, evitando los estados _listening_ y _learning_.

**Uso recomendado:**

- Puertos conectados a **dispositivos finales (hosts, servidores, impresoras)**.
    
- Nunca en enlaces troncales o puertos hacia otros switches.
    

**Comando:**

```
interface GigabitEthernet0/3
 spanning-tree portfast
```

**Configuración global:**

`spanning-tree portfast default`

**Verificación:**

`show spanning-tree interface Gi0/3 portfast`

**Advertencia del IOS:** activar PortFast en enlaces hacia switches puede causar bucles temporales.

---

### **3. BPDU Guard**

Desactiva un puerto si recibe **BPDUs** inesperadas.  
Se utiliza junto con **PortFast** para proteger los puertos de acceso ante conexiones de switches no autorizados.

**Comandos:**

```
spanning-tree bpduguard enable
spanning-tree portfast bpduguard default
```

**Recuperación automática:**

```
errdisable recovery cause bpduguard
errdisable recovery interval 30
```

**Verificación:**

`show spanning-tree summary`

---

### **4. BPDU Filter**

Evita el envío o recepción de BPDUs, dependiendo de su modo de aplicación.

**Dos modos de configuración:**

- **Global:**  
    Solo afecta a puertos con **PortFast activo**.  
    Si recibe una BPDU, se desactiva el filtro y el puerto pierde su estado PortFast.
    
    `spanning-tree portfast bpdufilter default`
    
- **Por interfaz:**  
    Desactiva completamente el envío/recepción de BPDUs (sin depender de PortFast).
    
```
interface GigabitEthernet0/3
 spanning-tree bpdufilter enable
```
    

**Advertencia:** el uso incorrecto de BPDU Filter puede causar bucles de capa 2.

---

### **5. Root Guard**

Evita que un puerto no autorizado se convierta en **Root Port**.  
Si un puerto recibe una BPDU con mejor prioridad que el root actual, se coloca en estado **root-inconsistent**.

**Uso típico:** en **puertos troncales** hacia switches de acceso.

**Comando:**

`spanning-tree guard root`

**Verificación:**

`show spanning-tree inconsistentports`

---

### **6. Loop Guard**

Previene que un puerto en estado alternativo o raíz pase a _forwarding_ si deja de recibir BPDUs (por fallos unidireccionales).

Sin Loop Guard, el puerto podría asumir incorrectamente que la topología está libre de bucles y provocar una tormenta de broadcast.

**Funcionamiento:**

- Si no se reciben BPDUs, el puerto se coloca en **loop-inconsistent state**.
    
- Cuando el flujo de BPDUs se restablece, el puerto vuelve automáticamente a _forwarding_.
    

**Configuración global:**

`spanning-tree loopguard default`

**Configuración por interfaz:**

```
interface GigabitEthernet0/2
 spanning-tree guard loop
```

**Nota:** No debe activarse en puertos con PortFast.

---

### **7. Bridge Assurance**

Asegura la **bidireccionalidad** de los enlaces troncales entre switches.  
Si un switch deja de recibir BPDUs de su vecino, el puerto se bloquea.

**Comando:**

`spanning-tree bridge assurance`

**Verificación:**

`show spanning-tree summary`

**Importante:** se usa únicamente en **enlaces punto a punto entre switches** con STP habilitado.

---

### **8. UDLD (Unidirectional Link Detection)**

Detecta **enlaces unidireccionales físicos** que podrían causar loops.  
Funciona en la capa 2, complementando a Loop Guard.

**Modos de operación:**

- **Normal:** Detecta fallos y reporta el evento.
    
- **Aggressive:** Desactiva el puerto si no se reciben mensajes UDLD tras varios intentos.
    

**Configuración:**

```
udld enable
udld aggressive
```
**Verificación:**

```
show udld neighbor
show udld interface Gi0/0
```

**Uso recomendado:** en **enlaces troncales de fibra o uplinks críticos.**

---

### **9. Ajustes de costos y prioridades**

Se pueden ajustar los **costos de ruta** para controlar la selección del root path.

**Métodos de costo:**

- **Short (16 bits):** valores hasta 65,535 (para enlaces <10 Gbps).
    
- **Long (32 bits):** valores hasta 200,000,000 (para enlaces >10 Gbps).
    

**Comandos:**

```
spanning-tree pathcost method long
interface GigabitEthernet0/1
 spanning-tree cost 20000
```

**Verificación:**

`show spanning-tree vlan 1`

---

### **10. Recomendaciones de diseño**

- Activar **PortFast + BPDU Guard** en todos los puertos de acceso.
    
- Activar **Loop Guard** y **Bridge Assurance** en los enlaces troncales.
    
- Usar **UDLD aggressive** en enlaces de fibra o troncales críticos.
    
- Mantener **Root Bridge** definido manualmente con prioridad ajustada.
    
- Usar **MST o Rapid-PVST+** según la escala y los dominios de VLAN.
    

---

### **11. Comandos clave de verificación**

```
show spanning-tree summary
show spanning-tree inconsistentports
show spanning-tree interface detail
show udld interface
show spanning-tree mst
```