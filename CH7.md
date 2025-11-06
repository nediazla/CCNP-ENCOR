## **Capítulo 7 – Enhanced Interior Gateway Routing Protocol (EIGRP)**

### **1. Introducción**

**EIGRP** (Enhanced Interior Gateway Routing Protocol) es un **protocolo de enrutamiento vector-distancia avanzado**, desarrollado originalmente por Cisco y estandarizado en **RFC 7868**.  
Combina las ventajas de los protocolos de estado de enlace y de vector-distancia tradicionales, logrando **convergencia rápida, escalabilidad y estabilidad**.

**Ventajas clave:**

- Convergencia extremadamente rápida mediante el algoritmo **DUAL (Diffusing Update Algorithm)**.
    
- Intercambia **solo cambios** en la tabla de enrutamiento, no la tabla completa.
    
- Soporta **IPv4 e IPv6** con una sola configuración (_multi-address family_).
    
- Permite **equal y unequal load balancing**.
    
- Excelente desempeño en redes grandes y en despliegues **DMVPN**.
    

---

### **2. Conceptos fundamentales**

|Concepto|Descripción|
|---|---|
|**Successor**|Es la ruta con la métrica más baja hacia un destino (ruta activa).|
|**Feasible Distance (FD)**|Métrica del sucesor calculada localmente.|
|**Reported Distance (RD)**|Métrica que reporta un vecino hacia el mismo destino.|
|**Feasibility Condition (FC)**|Una ruta se considera de respaldo si RD < FD del sucesor.|
|**Feasible Successor**|Ruta alternativa que cumple la FC, almacenada como respaldo libre de bucles.|

Esta lógica garantiza **rutas de respaldo sin loops**, lo que permite la convergencia inmediata sin tener que esperar consultas.

---

### **3. Tablas de EIGRP**

EIGRP mantiene tres tablas principales:

1. **Neighbor Table** – lista de routers vecinos alcanzables (establecida con _hellos_).
    
2. **Topology Table** – contiene todas las rutas conocidas y sus métricas (éxito y respaldo).
    
3. **Routing Table** – solo incluye las mejores rutas (successors).
    

**Hello y Hold Timers:**

- Interfaces rápidas: **Hello = 5 s**, **Hold = 15 s**
    
- Interfaces lentas (por ejemplo, seriales): **Hello = 60 s**, **Hold = 180 s**
    

---

### **4. Reliable Transport Protocol (RTP)**

EIGRP utiliza su propio protocolo de transporte confiable, **RTP**, que garantiza la entrega ordenada de los paquetes entre vecinos.

|Tipo de paquete|Requiere ACK|Descripción|
|---|---|---|
|**Hello**|No|Descubre y mantiene vecinos|
|**ACK**|No|Reconoce paquetes recibidos|
|**Update**|Sí|Envía nuevas rutas o cambios|
|**Query**|Sí|Solicita rutas alternativas|
|**Reply**|Sí|Responde a consultas|

---

### **5. Cálculo de la métrica**

EIGRP utiliza **múltiples parámetros físicos** para calcular su métrica compuesta.  
Los factores principales son: **ancho de banda, retardo, carga, confiabilidad y MTU**.

**Fórmula clásica:**

![[Pasted image 20251104153218.png]]

**Valores por defecto:**  
K1=1, K2=0, K3=1, K4=0, K5=0

**Wide Metrics (EIGRP Named Mode):**

- Introduce **K6** (atributos extendidos: _jitter_ y _energy_).
    
- Permite escalas hasta **4.2 Tbps**.
    
- Ajuste mediante:
    
```
metric rib-scale 128
```
    

---

### **6. Modos de configuración**

#### **Modo clásico**

```
router eigrp 100
 network 172.16.0.0 0.0.0.255
 no auto-summary
```

#### **Modo nombrado (recomendado)**

```
router eigrp CCNP
 address-family ipv4 autonomous-system 100
  network 10.0.0.0 0.0.0.255
  af-interface default
   no shutdown
  exit-af-interface
 exit-address-family
```

El **modo nombrado** soporta IPv4 e IPv6 y permite configuración jerárquica.

---

### **7. Métricas y balanceo**

- **Equal-cost multipath (ECMP):** balanceo entre rutas con igual métrica (hasta 32).
    
- **Unequal-cost load balancing:** activado con el comando `variance n`.
    
    - EIGRP instala rutas adicionales con métrica ≤ FD × variance.
        

**Ejemplo:**

```
router eigrp 100
 variance 2
```

---

### **8. Tablas y verificación**

**Comandos útiles:**

```
show ip eigrp neighbors
show ip eigrp topology
show ip eigrp interfaces
show ip route eigrp
show ip protocols
```

**Código de ruta en la tabla de enrutamiento:**

`D 192.168.0.0/16 [90/130816] via 172.16.0.2, 00:08:30, GigabitEthernet0/0`

- “D” = EIGRP
    
- 90 = Administrative Distance
    
- 130816 = Métrica
    

---

### **9. Autenticación EIGRP**

Para evitar actualizaciones no autorizadas, EIGRP soporta **autenticación MD5** (y SHA en IOS 15+).

**Ejemplo:**

```
key chain EIGRP_KEYS
 key 1
  key-string CCNP2025
!
interface Gi0/0
 ip authentication mode eigrp 100 md5
 ip authentication key-chain eigrp 100 EIGRP_KEYS
```

---

### **10. Resumen de rutas**

El **resumen** mejora la escalabilidad y limita el dominio de consultas.

**Configuración manual:**

```
interface GigabitEthernet0/1
 ip summary-address eigrp 100 10.10.0.0 255.255.0.0
interface GigabitEthernet0/1
 ip summary-address eigrp 100 10.10.0.0 255.255.0.0
```

**Recomendación:** mantener `no auto-summary` activo (ya es predeterminado desde IOS 15).

---

### **11. Temporizadores y convergencia**

- **Hello timer:** cada 5 s (enlace rápido)
    
- **Hold timer:** 3 × Hello
    
- **Active timer:** tiempo que DUAL espera respuesta antes de declarar _stuck-in-active (SIA)_, normalmente 3 min.
    

DUAL permite convergencia inmediata si existe un **feasible successor**; si no, se generan _queries_ para buscar rutas alternativas.

---

### **12. Verificación rápida de configuración**

```
show ip eigrp topology
show ip route
show ip eigrp neighbors detail
```

**Salidas esperadas:**

- Vecinos activos con tiempos de hello.
    
- Rutas “Successor” y “Feasible Successor” en la tabla topológica.
    
- Prefijos agregados en la tabla de enrutamiento (código “D”).
    

---

### **13. Datos de examen importantes**

|Tema|Valor típico / concepto|
|---|---|
|Protocolo IP EIGRP|**Número 88**|
|AD interno|**90**|
|AD externo|**170**|
|Hello timer (rápido)|**5 s**|
|Hello timer (lento)|**60 s**|
|Feasibility condition|RD < FD|
|Máx. rutas ECMP|32|
|Algoritmo base|DUAL|
|Balanceo desigual|`variance`|

---

### **14. Resumen conceptual**

- EIGRP combina rapidez, estabilidad y escalabilidad.
    
- Utiliza métricas compuestas precisas y soporte multi-familia.
    
- Evita loops mediante la **feasibility condition**.
    
- Facilita diseños jerárquicos gracias al resumen manual.
    
- Es una opción idónea para entornos empresariales complejos que no requieren OSPF.