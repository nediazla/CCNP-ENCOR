## **Capítulo 1: Packet Forwarding – Resumen técnico**

### **1. Concepto general**

El _packet forwarding_ es el proceso mediante el cual los dispositivos de red (routers y switches) deciden cómo reenviar los paquetes IP hacia su destino final. Cisco ha desarrollado varias arquitecturas de conmutación a lo largo del tiempo, evolucionando desde mecanismos basados en CPU hasta hardware optimizado.

---

### **2. Mecanismos de reenvío (switching mechanisms)**

Cisco utiliza **tres métodos principales** para tomar decisiones de reenvío:

#### **a. Process Switching (Software Switching)**

- Primer método de reenvío implementado.
    
- La **CPU general** del dispositivo procesa cada paquete individualmente.
    
- Es el método **más lento** porque requiere que el router inspeccione el encabezado, busque en la tabla de enrutamiento y reescriba las direcciones cada vez.
    
- Solo se usa actualmente para tráfico especial como:
    
    - Paquetes con opciones IP.
        
    - Tráfico destinado o originado en el router.
        
    - Paquetes que requieren información aún no resuelta (como ARP).
        

**Ventaja:** precisión en la decisión.  
**Desventaja:** alto consumo de CPU, baja velocidad.

---

#### **b. Fast Switching**

- Segunda generación de switching de Cisco.
    
- Introduce una **caché de rutas** (fast-switching cache).
    
- El **primer paquete** de un flujo es procesado por CPU y su resultado se almacena.
    
- Los **paquetes siguientes** usan esa entrada de caché, evitando reconsultar la tabla de enrutamiento.
    
- Se mejora la velocidad, pero **los cambios de topología invalidan la caché**, provocando reversiones a process switching.
    

**Ventaja:** mejora de rendimiento respecto a process switching.  
**Desventaja:** poco escalable ante cambios de red frecuentes.

---

#### **c. Cisco Express Forwarding (CEF)**

- **Método por defecto en equipos Cisco modernos.**
    
- Se basa en **tablas preconstruidas** que permiten reenviar paquetes directamente en hardware (ASICs, TCAM, NPUs).
    
- Se usa en dos modalidades:
    
    - **Software CEF:** manejado por la CPU.
        
    - **Hardware CEF:** usa procesadores especializados (ASICs, TCAM).
        

##### **Componentes clave de CEF**

1. **FIB (Forwarding Information Base):**
    
    - Derivada de la tabla de enrutamiento (RIB).
        
    - Contiene todas las rutas conocidas y el siguiente salto.
        
    - Optimizada para búsquedas rápidas (longest prefix match).
        
2. **Adjacency Table:**
    
    - Almacena información de capa 2 (direcciones MAC, encapsulación) para cada siguiente salto.
        

##### **Modos de operación**

- **Centralized CEF:** toda la conmutación ocurre en el route processor.
    
- **Distributed CEF (dCEF):** las _line cards_ mantienen copias sincronizadas de FIB y adjacency, realizando reenvío directamente sin involucrar la CPU.
    

**Ventajas de CEF:**

- **Alto rendimiento y escalabilidad.**
    
- **Menor uso de CPU.**
    
- **Estabilidad ante cambios de topología** (sin recálculo de caché).
    
- **Optimización de QoS y ACLs en hardware.**
    

---

### **3. Tablas de hardware: CAM y TCAM**

#### **CAM (Content Addressable Memory)**

- Usada en el _switching_ de capa 2.
    
- Guarda las asociaciones **MAC ↔ puerto/VLAN**.
    
- Permite búsquedas instantáneas de direcciones.
    
- Las entradas envejecen (por defecto, 300 segundos).
    

#### **TCAM (Ternary CAM)**

- Usada en _switching_ de capa 3 y funciones avanzadas.
    
- Permite coincidencias parciales (0, 1 o “don’t care”).
    
- Utilizada por:
    
    - **ACLs** (permit/deny).
        
    - **QoS** (clasificación, marcado, policing).
        
    - **Políticas de seguridad**.
        

---

### **4. Flujo de reenvío (Multilayer Switching)**

1. El paquete llega a la **cola de ingreso** (ingress queue).
    
2. El switch consulta simultáneamente:
    
    - **CAM** para capa 2.
        
    - **FIB** para capa 3.
        
    - **TCAM** para ACLs/QoS.
        
3. Se determina el puerto de salida y se coloca en la **cola de egreso** (egress queue).
    
4. Se **reescribe la trama**:
    
    - Se reemplaza la MAC destino con la del siguiente salto.
        
    - Se actualiza la MAC origen con la del MLS.
        
    - Se decrementa el **TTL** y se recalculan los checksums.
        

Todo este proceso se realiza **en hardware** para lograr velocidad de línea.

---

### **5. Comandos relevantes**

- `ip cef` → Habilita CEF.
    
- `no ip cef` → Desactiva CEF.
    
- `show ip cef summary` → Muestra si el router opera en modo centralizado o distribuido.
    

---

### **6. En resumen – Comparativo de métodos**

|Método|Procesamiento|Rendimiento|Escalabilidad|Uso actual|
|---|---|---|---|---|
|**Process Switching**|CPU|Bajo|Muy limitada|Casos especiales|
|**Fast Switching**|CPU + Caché|Medio|Limitada|Obsoleto|
|**CEF**|Hardware (ASIC/TCAM)|Alto|Alta|Estándar actual|