## **Capítulo 4 – Multiple Spanning Tree Protocol (MSTP)**

### **1. Introducción**

El **Multiple Spanning Tree Protocol (MSTP)**, definido en el estándar **IEEE 802.1s**, permite agrupar varias VLANs bajo una misma instancia de spanning tree, reduciendo la cantidad de procesos STP que deben ejecutarse en redes con muchas VLANs.  
Es el único modo de STP **totalmente interoperable** entre dispositivos Cisco y no Cisco.

**Beneficios principales:**

- Reducción del número de instancias STP.
    
- Mejora de la escalabilidad en redes grandes.
    
- **Balanceo de carga** entre enlaces redundantes mediante diferentes instancias.
    
- Mayor **tolerancia a fallos**, ya que un error en una instancia no afecta a las demás.
    
- Recomendado en **entornos multivendor**.
    

---

### **2. Arquitectura de MST**

MST divide la topología en **regiones** y dentro de ellas se crean **instancias** de spanning tree.

#### **Componentes principales**

1. **MST Region**
    
    - Conjunto de switches configurados con la misma información:
        
        - **Nombre del dominio MST**
            
        - **Número de revisión**
            
        - **Asignación de VLANs a instancias**
            
    - Todos los switches dentro de la región deben coincidir exactamente en estos tres parámetros.
        
2. **Tipos de árboles dentro de MST:**
    
    - **IST (Internal Spanning Tree):**
        
        - Es la instancia **0** (por defecto).
            
        - Coordina la comunicación dentro de la región MST.
            
    - **CIST (Common and Internal Spanning Tree):**
        
        - Conecta todas las regiones MST y los switches que usan PVST+ o RSTP.
            
        - Actúa como un árbol “global” de spanning tree entre regiones.
            
    - **MSTI (Multiple Spanning Tree Instances):**
        
        - Instancias adicionales (1–4094) creadas por el administrador.
            
        - Cada MSTI puede incluir una o varias VLANs.
            

---

### **3. Funcionamiento general**

- Dentro de una **región MST**, el **IST (Instance 0)** intercambia BPDUs con los demás switches.
    
- Hacia el exterior, el **CIST Root Bridge** representa la región entera como si fuera un único switch.
    
- Cada instancia (MSTI) calcula su propio árbol libre de bucles y determina qué puertos estarán bloqueados o reenviando.
    

---

### **4. Configuración básica**

Para configurar MST en Cisco IOS:

```
Switch(config)# spanning-tree mode mst
Switch(config)# spanning-tree mst configuration
Switch(config-mst)# name ExamCram
Switch(config-mst)# revision 1
Switch(config-mst)# instance 1 vlan 10,20
Switch(config-mst)# instance 2 vlan 30,40
Switch(config-mst)# exit
```

**Verificación:**

`show spanning-tree mst configuration`

**Salida esperada:**

```
Name      [ExamCram]
Revision  1     Instances configured 3
Instance  Vlans mapped
--------  -------------------------------------
0         1-9,11-19,21-29,31-39,41-4094
1         10,20
2         30,40
-----------------------------------------------
```

---

### **5. Prioridades y roles por instancia**

Cada **instancia MST** tiene su propio proceso de elección de **Root Bridge**, independiente de las otras.

**Configuración de prioridad del Root Bridge:**

```
spanning-tree mst 1 root primary
spanning-tree mst 2 root secondary
```
**O establecer prioridad manualmente:**

```
spanning-tree mst 1 priority 4096
```

Esto permite distribuir la carga entre switches:

- Un switch puede ser **root** para la instancia 1.
    
- Otro puede ser **root** para la instancia 2, logrando **balanceo de tráfico**.
    

---

### **6. Ajuste de costo y prioridad de puerto**

Cada interfaz puede tener costos y prioridades distintos por instancia.

**Comandos:**

```
spanning-tree mst 1 port-priority 64
spanning-tree mst 1 cost 20000
```

**Rangos:**

- **Cost:** 1–200,000,000
    
- **Port priority:** 0–240 (múltiplos de 16, por defecto 128)
    

**Verificación:**

```
show spanning-tree mst 1
```

---

### **7. Temporizadores por instancia**

Al igual que STP tradicional, MST utiliza:

- **Hello Time:** 2 s (envío de BPDUs)
    
- **Forward Delay:** 15 s
    
- **Max Age:** 20 s
    

Configurables por instancia:

```
spanning-tree mst 1 hello-time 2
spanning-tree mst 1 forward-time 15
spanning-tree mst 1 max-age 20
```
---

### **8. Compatibilidad e interoperabilidad**

- **MSTP es compatible con RSTP (802.1w)** y **STP (802.1D)**.
    
- Al conectar regiones MST con otros dominios STP, el **IST (instancia 0)** actúa como puente de compatibilidad.
    
- La región completa se comporta externamente como un solo switch, simplificando la integración.
    

---

### **9. Ejemplo completo (Exam Cram Example 1.23)**

#### **Objetivo:**

Distribuir la carga entre dos switches (SW1 y SW3).

**Configuración en SW1:**

```
spanning-tree mst 1 root primary
spanning-tree mst 2 root secondary
```

**Configuración en SW3:**

```
spanning-tree mst 2 root primary
spanning-tree mst 1 root secondary
```

Esto hace que:

- SW1 sea **Root Bridge** para la instancia 1 (VLAN 10–20).
    
- SW3 sea **Root Bridge** para la instancia 2 (VLAN 30–40).
    

**Resultado:**

- Balanceo de tráfico por VLAN.
    
- Convergencia rápida.
    
- Reducción de carga CPU al limitar las instancias.
    

---

### **10. Comandos de verificación esenciales**

```
show spanning-tree mst configuration
show spanning-tree mst
show spanning-tree mst 1
show spanning-tree summary
```

---

### **11. Ventajas clave de MST**

|Característica|Descripción|
|---|---|
|**Escalabilidad**|Soporta hasta 4094 VLANs con pocas instancias STP.|
|**Balanceo de carga**|Permite distribuir VLANs entre distintas instancias.|
|**Compatibilidad**|Interopera con RSTP y STP estándar.|
|**Estabilidad**|Fallos en una instancia no afectan a las demás.|
|**Estandarización**|IEEE 802.1s, interoperable con otros fabricantes.|